# Automações

**Assunto:** Automações
**Versão do código de referência:** Projeto3 v0.8.0.7.0 (ainda não migrado ao Projeto4)
**Data:** 2026-09-29
**Status:** reservado no Projeto4 — referência do Projeto3 completa

## Escopo
Scheduler, alertas (inclusive repetidos), intervenção manual, isolamento global versus individual e o fluxo de reset de senha com trava de segurança. Nada disto foi conectado no Projeto4 ainda (etapa 6, `P4-0601` a `P4-0605`).

## Como funciona no Projeto3

### Estado e liga/desliga
`src/automacoes/automacoes-state.service.js` guarda o estado global: automação geral ligada/desligada (`getAutomacoesState`, `setAutomacoesLigado`), quais das três automações estão ativas (Aviso de vencimento, Ticket aberto, Sem tickets) e os canais de aviso (tela, Telegram grupo, Telegram privado). É estado único do processo — hoje não há separação por usuário (ver `P4-0604`).

### Scheduler
`src/automacoes/automacoes-scheduler.service.js` roda o ciclo automaticamente em intervalo aleatório entre `AUTOMACOES_INTERVALO_MIN_MS` e `AUTOMACOES_INTERVALO_MAX_MS`; expõe status (`getAutomacoesSchedulerStatus`) e permite reiniciar (`restartAutomacoesScheduler`). O mesmo ciclo pode ser disparado manualmente por `POST /api/automacoes/executar-ciclo`, que busca os tickets (`listarTicketsFreshdesk`), roda as automações ativas e manda avisos por Telegram se houver.

### As três automações
| Automação | Serviço | O que verifica |
|---|---|---|
| Aviso de vencimento | `aviso-vencimento.service.js` | tickets perto do prazo (`fr_due_by`/vencimento) |
| Ticket aberto | `ticket-aberto.service.js` | tickets abertos/pendentes sem tratamento (`isAbertoOuPendente`) |
| Sem tickets | `no-tickets.service.js` | ausência de tickets no filtro monitorado |

Cada uma devolve uma lista de avisos, que passa pelo controle de repetição antes de virar alerta na tela ou no Telegram.

### Controle de repetição de alerta
`src/automacoes/alert-repeat-control.service.js` decide se um alerta já emitido deve repetir (`shouldEmitRepeatAlert`, `markRepeatAlert`, `filterAvisosByRepeat`) e permite configurar isso por seção (`getAlertRepeatConfig`/`setAlertRepeatConfig`, rota `/api/automacoes/repeat-config`). Persistido em `data/alert-repeat-control.json`.

### Intervenção manual
`src/automacoes/intervencao-manual.service.js` permite que um atendente assuma manualmente um ticket, pausando avisos de tela/áudio enquanto isso (`iniciarIntervencao`, `heartbeatIntervencao`, `finalizarIntervencao`, `isPausaManualAtiva`). Depende de heartbeat periódico do frontend; expira sozinho se o heartbeat parar (idade máxima configurável por `INTERVENCAO_MANUAL_HEARTBEAT_MAX_AGE_MS`).

### Canais de aviso
`src/telegram/telegram.service.js` envia os alertas (`sendTelegramAlerts`) para até três destinos configuráveis: grupo, chat privado geral e chat privado de "sem tickets"/"alertas" (variáveis `TELEGRAM_CHAT_ID*` no `.env`). Também expõe status (`/api/telegram/status`) e teste (`/api/telegram/testar`) sem nunca devolver o token completo.

### Reset de senha — hoje só simulado
`src/automacoes/reset-senha.workflow.js` (`simularResetSenhaWorkflow`) descreve os passos do fluxo (receber ticket → classificar por palavras-chave → aplicar tag → extrair nome/e-mail/CPF → consultar Portal → pedir confirmação → resetar somente em produção → responder ticket → registrar log) mas **não executa nenhuma ação real** — `AUTOMACOES_MODO_SIMULACAO=true` no `.env.example`, e não há indicação no código de um caminho de produção habilitado. A parte de classificação por palavras-chave e extração de dados é real e está detalhada em `07-agentes-ia`; a parte de reset propriamente dita continua simulada.

## Isolamento global vs. individual (pendência a revisar, `P4-0604`)
No Projeto3, quase tudo é estado global do processo: automação ligada/desligada, scheduler, controle de repetição de alerta. Só a intervenção manual já nasce por execução (`execucao_id`/`ticket_id`). Ao migrar para o Projeto4 (até 5 usuários simultâneos), cada um desses estados precisa ser classificado como realmente global, por sessão, por usuário ou por ticket — nenhuma dessas decisões foi tomada ainda.

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0601` a `P4-0605`.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
