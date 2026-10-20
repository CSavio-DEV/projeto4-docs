# Logs e auditoria

**Assunto:** Logs e auditoria
**Versão do código de referência:** Projeto4 v0.1.4.35 (aba Logs técnica) e Projeto3 v0.8.0.7.0 (referência; auditoria por usuário ainda não existe)
**Data:** 2026-10-20
**Status:** parcialmente implementado no Projeto4 — aba Logs técnica ativa desde a v0.1.4.29, aguardando validação; auditoria individual ainda pendente; referência do Projeto3 completa; requisitos de auditoria do Portal já decididos

## Escopo
Visualizador do log diário, auditoria individual, filtros, retenção e remoção de dados sensíveis dos logs.

## Requisitos de auditoria já decididos (2026-09-28)
Toda ação e auditoria usa a identidade principal do Cloudflare Access. Para o Portal, registrar tentativa e resultado do login, ativação, expiração, invalidação e logout da sessão do Portal, e as operações auditáveis feitas com ela — nunca senha, cookie completo, token, cabeçalhos de autenticação ou credenciais reutilizáveis. Auditoria das buscas salvas (criação, atualização, exclusão) também é requisito (`TCK-0403` em `03-tickets-freshdesk/pendencias-tickets.md`). Detalhes em `02-acesso-seguranca/README.md`.

## Estado atual no Projeto4 (v0.1.4.29 a v0.1.4.35)

*(Implementado, aguardando validação. A arquitetura de logs apareceu no ADD v004.)* É um log **técnico** do processo, não uma trilha de auditoria por usuário.

### Modos e configuração

Modos da aba: `Desligado` (`off`), `Resumido` (`summary`) e `Completo` (`full`). Configuração inicial no `.env`:

```dotenv
PROJETO4_LOG_LEVEL=summary
PROJETO4_LOG_MAX_ENTRIES=500
PROJETO4_LOG_REFRESH_MS=2000
PROJETO4_LOG_TIMEZONE=America/Fortaleza
PROJETO4_LOG_FILE_ENABLED=true
```

O instalador mescla as opções de log sem substituir o conteúdo local do `.env.example`.

### Componentes

memória circular com limite configurável; arquivos diários; sanitização antes de armazenar; API incremental. Rotas: `GET /api/logs`, `GET /api/logs/status`, `GET /api/logs/today`, `DELETE /api/logs` (limpar) e `PUT /api/logs/settings` (modo).

### O que é registrado

Instrumentação inicial do cliente Freshdesk (v0.1.4.29): cache, deduplicação, conclusão e HTTP 429. Depois (v0.1.4.30 e v0.1.4.33 a v0.1.4.35): tipo e rótulo da busca; operação (`search_tickets`, `search_first_response_candidates`, `list_candidate_stats`; `ticket_details_stats` não deve aparecer em buscas com `frDueBy`); duração; HTTP; `X-RateLimit-Total`, `X-RateLimit-Remaining` e `X-RateLimit-Used-CurrentRequest`; cache hit e miss; candidatos, páginas, IDs encontrados e ausentes; interrupção por rate limit; resumo da busca com o consumo real (`apiUnitsConsumed`). O `ticketId` só entra em contexto técnico já sanitizado.

### Regras obrigatórias

nunca registrar API key, `Authorization`, cookie completo, token reutilizável nem senha; evitar descrições e dados pessoais dos tickets (preferir `ticket_id`, status, horário e motivo técnico); permitir copiar ou exportar logs técnicos no futuro; separar logs por sessão e usuário quando o multiusuário estiver ativo.

### Erros conhecidos e ajustes pendentes

- o nome do arquivo diário pode receber a data do dia seguinte (UTC em vez da data local do fuso configurado): P4-1005;
- cores por nível e escopo, rolagem que não pula para o fim e colunas legíveis (horário, nível, escopo, operação, mensagem, detalhes): P4-1006;
- mais detalhes das buscas (filtros, grupos com ID, distribuição por grupo e status, `updated_since`, motivo de descarte, `Retry-After`, `complete`, duração): P4-1007;
- fuso: os logs usam `America/Fortaleza` (configuração) e o SLA usa `America/Sao_Paulo`; o Projeto3 gravava em `America/Sao_Paulo` (P4-1210).

## Como funciona no Projeto3

### Formato do log
`src/logs/text-log.service.js` grava em texto simples, um arquivo por dia (`logs/AAAA-MM-DD.log`, fuso `America/Sao_Paulo`), com o formato:
```text
[DD/MM/AAAA HH:MM:SS] [ESCOPO] mensagem
{dados em JSON, quando houver}
```
`writeTextLog(scope, message, data)` é a função central; `logFreshdesk()` e `logError()` são atalhos para escopos específicos. `src/logs/console-log-mirror.service.js` espelha todo `console.log`/`console.error` do processo para o mesmo arquivo, então erros não tratados também aparecem ali.

### Modo de log
Controlado por `LOG_MODO` no `.env` (`resumido`/`summary`/`compacto` vs. `completo`, padrão completo). No modo resumido, textos são cortados em 1200 caracteres e erros perdem o `stack`; no completo, tudo é gravado.

### Mascaramento de dados sensíveis (já existe, vale preservar)
Antes de gravar, `maskSensitive()` substitui por `***`: `FRESHDESK_API_KEY=...`, `Authorization: Basic ...`, e os campos JSON `apiKey`, `password` e `senha`. Essa é a mesma prática que a lição em `registros/erros-e-licoes` recomenda também para a documentação.

### Acesso ao log
Hoje só existe **um** endpoint: `GET /api/logs/today`, que devolve o arquivo do dia inteiro como texto puro (`logs.routes.js` → `readTodayLog()`). Não há:
- filtro por escopo, período ou palavra-chave;
- paginação (arquivos grandes vêm inteiros);
- separação por usuário — o log é do processo inteiro, não por sessão;
- política de retenção — os arquivos `logs/*.log` se acumulam sem limpeza automática;
- rota para listar ou baixar dias anteriores, só o dia atual.

### Auditoria por usuário — não existe hoje
Não há, no Projeto3, nenhum registro estruturado de "quem fez o quê" (login, busca, resposta, automação disparada) associado a um usuário identificado — o log de texto registra eventos do sistema, não uma trilha de auditoria por pessoa. Isso é inteiramente uma construção nova para o Projeto4 (`P4-1002`), motivada pelo suporte a múltiplos usuários simultâneos.

## O que o Projeto4 precisa decidir/migrar
- **Visualizador** (`P4-1001`): parcialmente atendido pela aba Logs técnica do Projeto4 (ver acima); falta validá-la e ajustá-la (P4-1005 a P4-1007);
- **Auditoria individual** (`P4-1002`): novo registro, por usuário, dos eventos citados na pendência (login, escolha do Portal, ticket aberto, resposta, alteração, download, automação, busca, reset de senha);
- **Filtros e retenção** (`P4-1003`): há memória circular com limite e um arquivo por dia; falta a política de quanto tempo manter os arquivos e um filtro mínimo por escopo e data;
- **Remoção de dados sensíveis** (`P4-1004`): a sanitização do Projeto4 (v0.1.4.29) está implementada e aguarda validação; o mascaramento do Projeto3 (`maskSensitive`) é um bom ponto de partida, mas cobre só um conjunto fixo de padrões — revisar se cobre os dados sensíveis do Projeto4 (ex.: dados extraídos pelo classificador de reset de senha em `07-agentes-ia`, como CPF e data de nascimento, que hoje não estão na lista de mascaramento).

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-1001` a `P4-1007`, `P4-0207` a `P4-0209` (implementação da auditoria de sessão do Portal) e `TCK-0403` (auditoria de buscas salvas) em `03-tickets-freshdesk/pendencias-tickets.md`.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
