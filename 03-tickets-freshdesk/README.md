# Tickets Freshdesk

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.35 instalada (última versão integralmente validada: v0.1.4.15; a arquitetura do fluxo híbrido foi validada como base no ADD v004; inclui a correção manual anterior em `src/frontend/principal.html`)
**Data:** 2026-10-20
**Status:** vigente

## Escopo

Interface personalizada para localizar, carregar e visualizar chamados do Freshdesk pela API v2, com autenticação por API Key.

## Visão geral

O módulo contempla:

- listagem leve de tickets;
- filtros combináveis, com múltiplos valores no mesmo filtro;
- tipos de busca predefinidos e pesquisas personalizadas;
- buscas salvas em JSON;
- grupos e agentes carregados a partir dos mapas do `.env`;
- painel flutuante com **Contexto geral** e **Contexto do ticket**.

## Estado atual

### Implementado

- servidor Node.js e rotas de tickets, de opções de filtros e de buscas salvas;
- tradutor de filtros para consultas da Search API;
- filtros múltiplos (OU dentro do campo, E entre campos);
- tipos de busca **Abertos Portal N1**, **1ª Resposta** (Vencidos, Hoje e Amanhã; regra **validada na conta real**, ver `adds-validados/`) e **Personalizada**;
- seletor de buscas dinâmico: até cinco buscas salvas (as duas de sistema contam) e `Personalizada (não salva)` sempre por último; nome `Personalizada` reservado;
- botão `i` com painel de informações; textos técnicos fora da área permanente;
- limitador de pesquisas no frontend: sem pesquisa simultânea, 5 s entre pesquisas e bloqueio após HTTP 429, com contagem regressiva;
- **fluxo híbrido** para toda busca com filtro de prazo de primeira resposta (`frDueBy`): Search API + listagem com `include=stats` + cruzamento por ID + cache persistente de stats;
- camada de controle de chamadas no backend: governador (fila, intervalo, concorrência), cache de chamadas idênticas, deduplicação e bloqueio após HTTP 429, sem derrubar o servidor;
- aba **Logs** técnica (ver `09-logs-auditoria`);
- buscas salvas com `data/saved-searches.json` como fonte única;
- painel de contexto com abas independentes por contexto;
- estado inicial simplificado da aba Tickets (validado pelo usuário);
- tratamento de erros da API (400, 429, 502);
- exclusão, nas buscas com prazo de primeira resposta, dos tickets que já receberam primeira resposta (`stats.first_responded_at`).

### Validado pelo usuário

Estado inicial simplificado; JSON como fonte única das buscas salvas; `Qualquer`/`Nenhum`/`Não atribuído`; correções de `forEach` e favicon; busca 1ª Resposta com Vencidos, Hoje e Amanhã (v0.1.4.13); limitador de pesquisas e bloqueio visual após HTTP 429 (v0.1.4.14); botão `i` e menos texto permanente (v0.1.4.15); `Abertos Portal N1` preservada.

Validada como **nova base de arquitetura** (ADD v004, v0.1.4.33 a v0.1.4.35): o fluxo híbrido concluiu 26 candidatos sem IDs ausentes, com 9 créditos e sem HTTP 429. As validações de uso continuado estão em TCK-0112.

### Implementado, aguardando validação completa

- transformação de busca salva alterada em **Personalizada não salva** (roteiro completo pendente, TCK-0103);
- nível `N1 OR null` (TCK-0102);
- apresentação detalhada dos erros de busca (parcial, TCK-0104);
- seletor de buscas com até cinco salvas, nome reservado, limpeza de filtros e aviso de busca vazia (v0.1.4.16 a v0.1.4.20);
- governador de chamadas, cache, deduplicação e tratamento de HTTP 429 no backend (v0.1.4.25 a v0.1.4.27);
- aba Logs (v0.1.4.29 e v0.1.4.30) e Aberto/Pendente explícitos na busca fixa (v0.1.4.31);
- cache persistente e aplicação do fluxo híbrido a toda busca com `frDueBy` (v0.1.4.34 e v0.1.4.35); uso continuado em TCK-0112.

### Erros conhecidos

- selecionar a busca salva `N2-Abertos` a transforma em `Personalizada (usar filtros)` (TCK-0110);
- o arquivo diário de log pode receber a data do dia seguinte (P4-1005).

### Regressões a prevenir

Detalhe do ticket renderizado como JSON bruto (TCK-0202), texto literal `\n` no topo do HTML, consulta individual `ticket_details_stats` em busca com `frDueBy` e uso de `fr_escalated` ou `responder_id` como critério de primeira resposta.

### Ainda não implementado

- detalhe completo do ticket sob demanda (descrição, conversas, anexos, `stats`, campos personalizados). Hoje o clique em um ticket usa apenas os dados da listagem;
- envio de **Resposta** e **Anotação** (os controles existem, mas ainda não chamam as rotas do Freshdesk);
- mapeamento dos cinco filtros personalizados (Portal, Tipo de usuário, Regional, Unidade escolar, Tipo);
- sincronização dinâmica de grupos e agentes com a conta Freshdesk;
- proteção preventiva por crédito restante, resultado parcial visível no frontend e controle entre usuários (TCK-0109). O limite de chamadas é da conta e é compartilhado com o Projeto3;
- pesquisa por número, lista e intervalo de tickets (TCK-0111);
- buscas salvas por usuário, com permissões e auditoria.

As pendências detalhadas, com IDs `TCK-xxxx` e a ordem de execução, ficam em [pendencias-tickets.md](pendencias-tickets.md). `registros/pendencias` referencia esse arquivo em vez de repeti-lo.

## Arquivos envolvidos

| Arquivo | Responsabilidade |
|---|---|
| `src/server.js` | Rotas da integração; leitura dos filtros como arrays (`getAll`); rotas de buscas salvas |
| `src/lib/freshdesk.js` | Comunicação com o Freshdesk, Basic Auth, normalização de erros, montagem das consultas, mapeamentos, combinação de valores múltiplos e tipos de busca |
| `src/lib/savedSearches.js` | Persistência das buscas salvas (listar, criar, atualizar, excluir) |
| `src/lib/env.js` | Loader manual do `.env` com `node:fs`; expõe domínio, API Key, campo de nível, mapas de grupos/agentes, configurações de chamadas, cache e logs, e se a integração está configurada |
| `src/api/freshdesk/` | Criada na v0.1.4.25: `freshdesk-client.js` (chamadas HTTP), `api-governor.js` (fila, intervalo, concorrência), `rate-limit-state.js` (cabeçalhos, bloqueio, `Retry-After`), `request-cache.js` e `request-deduplicator.js` |
| `src/services/tickets/` | Criada na v0.1.4.25: `first-response-classifier.js` (Vencidos, Hoje, Amanhã), `first-response.service.js` (coordena o fluxo da 1ª Resposta) e `ticket-details-context.js` (contexto por `ticket_id + updated_at`). O fluxo híbrido e o cache persistente das v0.1.4.33 a v0.1.4.35 seguem os UPDATE/FLOW dessas versões, que não listam os caminhos dos arquivos |
| `data/cache/freshdesk-ticket-stats.json` | Cache persistente de stats (chave `ticket_id + updated_at`; somente campos técnicos; TTL 168 h; até 2000 entradas); nunca versionar nem enviar em patch |
| `src/frontend/principal.html` | Painel de contexto, formulários, seletores, componentes de busca e buscas salvas |
| `src/frontend/js/principal.js` | Alternância de contextos e abas, componente de múltipla seleção, estado do tipo de busca, buscas salvas, opções dinâmicas |
| `src/frontend/css/principal.css` | Componente `.multiselect` e demais elementos visuais |
| `data/saved-searches.json` | Buscas salvas; fonte de verdade (ver `filtros-e-buscas.md`) |
| `.env` | Domínio e API Key do Freshdesk, chave do campo de nível, mapas de grupos e agentes |
| `.env.example` | Modelo sem credenciais reais |

## Documentos deste assunto

| Documento | Conteúdo |
|---|---|
| [api-e-regras.md](api-e-regras.md) | API do Freshdesk, rotas locais, mapeamentos, regras de negócio, erros |
| [filtros-e-buscas.md](filtros-e-buscas.md) | Filtros, valores especiais, tipos de busca, buscas salvas |
| [painel-contexto.md](painel-contexto.md) | Contexto geral e do ticket, abas, estado inicial da lista |
| [validacao.md](validacao.md) | Roteiro de regressão e critérios de aceite |
| [pendencias-tickets.md](pendencias-tickets.md) | Pendências da aba (TCK-xxxx), critérios de aceite, ordem de execução, teste multiusuário |

## Registros de tarefas concluídas

- `adds-validados/Projeto4_ADD_v001_2026-09-28_busca_primeira_resposta_validada.md` — busca 1ª Resposta (v0.1.4.9 a v0.1.4.13);
- `adds-validados/Projeto4_ADD_v002_2026-09-29_limitador_pesquisas_api_validado.md` — limitador de pesquisas e bloqueio visual de HTTP 429 (v0.1.4.14);
- `adds-validados/Projeto4_ADD_v003_2026-09-29_informacoes_tickets_validado.md` — botão `i` e redução de textos (v0.1.4.15);
- `adds-validados/Projeto4_ADD_v004_2026-09-29_busca_primeira_resposta_fluxo_hibrido_validado.md` — fluxo híbrido, cache persistente e logs (v0.1.4.33 a v0.1.4.35; ADD evolutivo).

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/` (vazia: consolidado até a v0.1.4.35 em 2026-10-20; o que ainda não foi validado consta em `pendencias-tickets.md`).
