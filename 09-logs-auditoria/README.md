# Logs e auditoria

**Assunto:** Logs e auditoria
**Versão do código de referência:** Projeto3 v0.8.0.7.0 (visualizador e auditoria por usuário ainda não migrados ao Projeto4)
**Data:** 2026-09-29
**Status:** reservado no Projeto4 — referência do Projeto3 completa; requisitos de auditoria do Portal já decididos

## Escopo
Visualizador do log diário, auditoria individual, filtros, retenção e remoção de dados sensíveis dos logs.

## Requisitos de auditoria já decididos (2026-09-28)
Toda ação e auditoria usa a identidade principal do Cloudflare Access. Para o Portal, registrar tentativa e resultado do login, ativação, expiração, invalidação e logout da sessão do Portal, e as operações auditáveis feitas com ela — nunca senha, cookie completo, token, cabeçalhos de autenticação ou credenciais reutilizáveis. Auditoria das buscas salvas (criação, atualização, exclusão) também é requisito (`TCK-0403` em `03-tickets-freshdesk/pendencias-tickets.md`). Detalhes em `02-acesso-seguranca/README.md`.

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
- **Visualizador** (`P4-1001`): página própria, não só texto puro — reaproveitando o padrão visual de `10-layout-frontend`;
- **Auditoria individual** (`P4-1002`): novo registro, por usuário, dos eventos citados na pendência (login, escolha do Portal, ticket aberto, resposta, alteração, download, automação, busca, reset de senha);
- **Filtros e retenção** (`P4-1003`): política de quanto tempo manter, e filtro mínimo por escopo/data;
- **Remoção de dados sensíveis** (`P4-1004`): o mascaramento do Projeto3 (`maskSensitive`) é um bom ponto de partida, mas cobre só um conjunto fixo de padrões — revisar se cobre os dados sensíveis do Projeto4 (ex.: dados extraídos pelo classificador de reset de senha em `07-agentes-ia`, como CPF e data de nascimento, que hoje não estão na lista de mascaramento).

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-1001` a `P4-1004`, `P4-0207` a `P4-0209` (implementação da auditoria de sessão do Portal) e `TCK-0403` (auditoria de buscas salvas) em `03-tickets-freshdesk/pendencias-tickets.md`.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
