# Dashboards

**Assunto:** Dashboards
**Versão do código de referência:** Projeto3 v0.8.0.7.0 (Dashboard ainda não migrado ao Projeto4)
**Data:** 2026-09-29
**Status:** reservado no Projeto4 — referência do Projeto3 completa

## Escopo
Dashboard inicial, heartbeat/presença e indicadores. O Projeto3 não tem uma página de dashboard dedicada com layout próprio — o que existe são as fontes de dado que uma tela de dashboard consumiria. No Projeto4, a página existe visualmente desde a v0.1.0.0 (parte do `principal.html` consolidado) mas sem nenhuma dessas fontes conectada ainda (`P4-0302`).

## Fontes de dado já existentes no Projeto3 (para conectar no Projeto4)

| Indicador | Fonte | Rota |
|---|---|---|
| Saúde do backend | `health.routes.js` — projeto, versão, porta, modo simulação, data/hora | `GET /api/health` |
| Automação ligada/desligada e por tipo | `automacoes-state.service.js` | `GET /api/automacoes/status`, `/state` |
| Status do scheduler | `automacoes-scheduler.service.js` | `GET /api/automacoes/scheduler-status` |
| Presença do frontend (aba ativa/inativa) | `front-presence.service.js` | `POST /api/frontend/heartbeat`, `GET /api/frontend/presence` |
| Intervenção manual em andamento | `intervencao-manual.service.js` | `GET /api/automacoes/intervencao/status` |
| Status do Telegram | `telegram.service.js` | `GET /api/telegram/status` |
| Últimos tickets carregados (cache) | `last-tickets-cache.service.js`, persistido em `data/freshdesk-last-tickets.json` | `GET /api/freshdesk/last-tickets` |
| Log do dia (para um resumo/contador de eventos) | `text-log.service.js` | `GET /api/logs/today` |

Não existe, no Projeto3, uma rota que já agregue tudo isso em um único payload de dashboard — cada indicador é lido separadamente. Ao montar o Dashboard do Projeto4, decidir se cada card consulta sua própria rota ou se vale criar um endpoint agregador.

## O que falta decidir/migrar (`P4-0301`, `P4-0302`, `P4-0303`)
- conectar o shell do Projeto4 à sessão real (nome no topo, status do Portal, menu) antes do Dashboard em si;
- desenhar os cards do Dashboard a partir da lista de fontes acima;
- padronizar a rota de heartbeat: o Projeto3 usa `/api/frontend/heartbeat`; o Projeto4 precisa decidir se mantém esse caminho ou usa `/heartbeat` direto, conforme a pendência já registrada.

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0301`, `P4-0302`, `P4-0303`.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
