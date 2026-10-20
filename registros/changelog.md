# Changelog do Projeto4

**Assunto:** transversal (todos)
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

Fonte oficial: os arquivos UPDATE e FLOW de cada versão (as entradas v0.1.4.9 a v0.1.4.13 foram resumidas a partir do ADD v001, pois os UPDATE/FLOW dessas versões não foram enviados; as de v0.1.4.14 a v0.1.4.35 vêm dos UPDATE oficiais, exceto v0.1.4.17, v0.1.4.19 e v0.1.4.28, que não têm UPDATE/FLOW) (ver `04_padrão_patch`/entrega de patches). Este changelog resume os mesmos fatos; em caso de dúvida, o UPDATE da versão prevalece.

## v0.1.0.0 — Layout consolidado inicial `[layout]`
Consolida Dashboard, Tickets, Automações, Portal, IA Freshdesk, Configurações e Logs em um único `principal.html`, com navegação interna por `data-page` e URL com âncora (`#dashboard`, `#tickets`, ...). Sem conexão com APIs reais.

## v0.1.0.1 — Correção da casca do layout `[layout]`
Corrige a hierarquia `.app-shell > .sidebar (72px) + .main > .topbar + .page-host`, restaurando a barra lateral compacta e removendo largura/margem conflitantes do patch anterior.

## v0.1.0.2 — Contexto de Tickets e ajustes mobile `[layout] [tickets]`
Restaura Contexto geral/Contexto do ticket em Tickets Freshdesk; no mobile o painel de contexto vira folha inferior e o menu passa a exibir o nome das 7 abas; remove `history.replaceState` ao abrir via `file://`; corrige a marca lateral de P3 para P4.

## v0.1.1.0 — Login e Portal opcional `[acesso-seguranca]`
Login do Projeto4 sobre a base v0.1.0.2. Oferece **Entrar no Portal** ou **Não logar no Portal**; tenta ler identidade do Cloudflare Access via HTTP/HTTPS; usa `Usuário Projeto4` na prévia local; bloqueia a página Portal sem sessão confirmada; não persiste senha do Portal.

## v0.1.2.0 — Servidor inicial do Projeto4 `[preparacao]`
Cria `package.json` e servidor HTTP Node próprios, sem dependências externas e sem reaproveitar módulos do Projeto3. `/` entrega `principal.html`, `/login` entrega `login.html`, `/health` e `/api/health` informam saúde do servidor. Cria `servidor-desligado.html` e `abrir_projeto4.cmd` (tenta iniciar o servidor; se falhar, abre o aviso local — um servidor desligado não consegue entregar sua própria página de erro).

## v0.1.3.0 — Aplicação integral do frontend ajustado `[layout]`
Substituição integral de `principal.html`, `principal.css` e `principal.js` a partir de frontend ajustado externamente (acompanhado do `registro_tecnico_projeto4`, ver `registros/erros-e-licoes`). `login.html`, `login.js` e `servidor-desligado.html` mantidos intactos. Validado antes: ZIP íntegro, JS com sintaxe válida, 7 páginas e 7 itens de navegação presentes, sem IDs duplicados, caminhos relativos compatíveis com `src/frontend/`.

## v0.1.4.0 — Registro da integração Tickets Freshdesk `[tickets]`
Instala o código completo da integração: `server.js`, `lib/env.js`, `lib/freshdesk.js`, `lib/savedSearches.js` e frontend correspondente. Incorpora `CHANGELOG_TIPOS_BUSCA_FILTROS` e a `DOCUMENTACAO_CONSOLIDADA_TICKETS_FRESHDESK` como referência (nessa ordem de precedência, atrás do código). **Correção de segurança:** removido do HTML o valor preenchido no campo visual da API Key; campo passou a somente leitura, instruindo que `FRESHDESK_API_KEY` mora só no `.env`. `.env`, credenciais reais, `data/saved-searches.json` e auditoria real não entraram no patch.

## v0.1.4.1 — Estado inicial simplificado de Tickets `[tickets]`
A aba abre direto na lista; antes da primeira busca mostra só ícone + "Nenhum ticket carregado"; um único botão **Carregar tickets**.

## v0.1.4.2 — Buscas fixas, Não atribuído e correção de 1ª Resposta `[tickets]`
Renomeia Padrão N1 para **Abertos Portal N1** (ID interno `padrao_n1` preservado); ao selecionar uma busca fixa, os filtros aparecem em Contexto Geral > Filtros; adiciona o filtro **Prazo para primeira resposta** (Vencidos/Hoje); adiciona **Não atribuído** ao filtro manual de Grupo; nível **Nenhum** interpretado como campo nulo; corrige a regra de 1ª Resposta: Vencidos = `fr_escalated:true AND status:2`; Hoje = janela do dia em `America/Sao_Paulo` convertida para UTC, com `fr_escalated:false`. **Superada nas v0.1.4.9 e v0.1.4.10:** a Search API rejeita `fr_escalated`. Selecionar a busca fixa não executa a busca automaticamente.

## v0.1.4.3 — Buscas fixas no JSON (revertido na v0.1.4.6) `[tickets]`
Passou a registrar Abertos Portal N1, 1ª Resposta e Personalizada no `data/saved-searches.json`, protegendo-as contra sobrescrita/exclusão; trocou `---` pelo `Qualquer` desabilitado. **Superado:** a injeção automática dessas três buscas no JSON foi removida na v0.1.4.6.

## v0.1.4.4 — Qualquer some do dropdown; Personalizada deixa de ser sistema `[tickets]`
`Qualquer` deixa de aparecer como opção selecionável, ficando só no botão fechado como placeholder. Apenas Abertos Portal N1 e 1ª Resposta continuam como buscas de sistema; Personalizada passa a busca normal, atualizável.

## v0.1.4.5 — Qualquer em cinza `[tickets]`
`Qualquer`, sem seleção, usa `var(--text-faint)` e peso normal; com seleção real, volta à cor normal. Nenhuma regra de busca, API ou persistência alterada.

## v0.1.4.6 — JSON como fonte única das buscas salvas `[tickets]`
Remove a injeção automática das 3 buscas fixas e a reordenação/regravação automática na leitura. A API devolve exatamente o conteúdo e a ordem do arquivo; busca apagada manualmente não retorna ao reiniciar; arquivo inexistente é criado só como `[]`. `system:true` no próprio JSON continua protegendo contra alteração pela interface, mas não contra edição manual do arquivo.

## v0.1.4.7 — Busca salva vira "Personalizada não salva" ao editar `[tickets]`
A primeira alteração manual em qualquer filtro, após carregar uma busca salva, transforma o estado em **Personalizada não salva**, desmarca a busca salva e não tenta reconhecer automaticamente os filtros anteriores — mesmo que voltem aos valores originais. A requisição seguinte usa `searchType=personalizada`. Erros HTTP 400 do backend passam a exibir mensagem e detalhes reais, não apenas "indisponibilidade do backend".

## v0.1.4.8 — Nível Nenhum, correção de forEach e favicon `[tickets] [layout]`
Corrige `$(...).forEach is not a function` em `markSearchAsUnsavedCustom()` (troca `$` por `$$`); restaura a opção real **Nenhum** no filtro de Nível (`cf_nvel_de_atendimento:'N1' OR cf_nvel_de_atendimento:null`); adiciona favicon vazio embutido para evitar a requisição a `/favicon.ico` e o 404 correspondente. Nenhuma regra de buscas salvas alterada.

## v0.1.4.9 — Sintaxe de `fr_due_by` e fim de `fr_escalated` `[tickets]`
Remove `fr_escalated:true` e `fr_escalated:false` da busca 1ª Resposta (a Search API responde `invalid_field`); corrige as datas de `fr_due_by` para `YYYY-MM-DD`; elimina o erro `Validation Failed` da consulta anterior.

## v0.1.4.10 — `stats.first_responded_at` como indicador `[tickets]`
Adiciona consulta individual com `include=stats`; adota `stats.first_responded_at` como indicador definitivo de primeira resposta; passa a comparar a data e hora real de `fr_due_by`.

## v0.1.4.11 — Opção Amanhã `[tickets]`
Adiciona **Amanhã** ao seletor de prazo para primeira resposta.

## v0.1.4.12 — Padrões da busca salva 1ª Resposta `[tickets]`
Na busca salva, **Amanhã** fica desmarcado por padrão e **Vencidos** e **Hoje** permanecem marcados; o filtro de primeira resposta passa a valer também quando a busca vira **Personalizada não salva**.

## v0.1.4.13 — Correção de `forEach` na atualização dos botões de busca `[tickets]`
Corrige `$(...).forEach is not a function` ao atualizar os botões de tipo de busca (troca `$()` por `$$()`). **Validada pelo usuário em 2026-09-28**, concluindo a tarefa da busca 1ª Resposta (ver `adds-validados/Projeto4_ADD_v001_2026-09-28_busca_primeira_resposta_validada.md`).

## v0.1.4.14 — Limitador de pesquisas e bloqueio visual em HTTP 429 `[tickets]`
Frontend: impede pesquisas simultâneas; intervalo mínimo de 5 s depois de uma pesquisa concluída; em HTTP 429, bloqueio mínimo de 10 s (ou `retryAfter`, se maior) com contagem regressiva em `Carregar tickets` e `Buscar`. Não altera filtros, consultas, stats, busca salva nem layout estrutural. Inaugura o destino permanente `docs/_updates/` e `docs/_flows/`. **Validada pelo usuário em 2026-09-29** (ADD v002). Não é controle global de consumo da API.

## v0.1.4.15 — Botão de informações `i` e menos texto permanente `[tickets] [layout]`
Adiciona o botão circular `i` ao lado do título; o painel (fecha por `×`, clique fora e `Esc`) reúne as explicações de Vencidos, Hoje, Amanhã, limite da API e abertura de ticket. Remove textos permanentes, as duas exibições duplicadas de "Busca atual" e o warning técnico sobre `fr_due_by` (a regra segue comentada no código). **Validada pelo usuário em 2026-09-29** (ADD v003).

## v0.1.4.16 — Seletor de buscas dinâmico `[tickets]`
Menu do botão seletor com até cinco buscas salvas (incluindo as duas de sistema, ou seja, até três adicionais), na ordem do JSON; `Personalizada não salva` sempre por último; clicar numa busca salva carrega os filtros e atualiza o seletor da área de buscas salvas. Implementada, aguardando validação.

## v0.1.4.17 — Não aplicada `[padrao-patch]`
O patch falhou por base divergente e nenhum arquivo foi alterado. Sem UPDATE/FLOW. Não gera ADD. Corrigida pela v0.1.4.18.

## v0.1.4.18 — Nome reservado, limpeza de filtros e aviso de busca vazia `[tickets]`
Criada sobre os arquivos reais da v0.1.4.16. Reserva `Personalizada`, `Personalizada não salva` e `Personalizada (não salva)` no frontend e no backend; ao entrar em Personalizada a partir de uma busca salva, limpa os filtros uma única vez; busca sem filtros pede confirmação (primeiros 30 tickets); definia `per_page=30` na Search API (**erro, removido na v0.1.4.21**). Implementada, aguardando validação.

## v0.1.4.19 — Não aplicada `[layout]`
Correção visual de `Personalizada (não salva)` proposta com a regra interpretada incorretamente; substituída pela v0.1.4.20. Sem UPDATE/FLOW. Registro a partir do estado de 2026-09-29.

## v0.1.4.20 — Cores de `Personalizada (não salva)` `[layout]`
No botão principal, `Personalizada` em dourado e `(não salva)` em cinza, com a seta dourada; no menu, item inativo na cor normal e item ativo com `Personalizada` em destaque e `(não salva)` em cinza. Regra visual aceita pelo usuário.

## v0.1.4.21 — Fim do `per_page` na Search API `[tickets]`
Remove `per_page=30` da URL da Search API e mantém `page=1`. Corrige o HTTP 400 causado por `per_page` em `/search/tickets`. A paginação configurável fica para a aba Configurações.

## v0.1.4.22 — Detalhes de 1ª Resposta em sequência `[tickets]`
Troca quatro consultas simultâneas de `GET /tickets/{id}?include=stats` por uma por vez. Reduz o pico, mas não a quantidade total de chamadas; HTTP 429 continuou. **Superada** pelo fluxo em lote (v0.1.4.32 e seguintes).

## v0.1.4.23 — Teste sem stats (`deadline_only`) `[tickets]`
Remove as consultas individuais e a dependência de `stats.first_responded_at`; classifica por `fr_escalated`, status e `fr_due_by` local. Fez uma chamada e não travou, mas trouxe tickets a mais. Experimental, não validada, **descartada** na v0.1.4.25.

## v0.1.4.24 — Teste com `responder_id == null` `[tickets]`
Filtro local experimental para Hoje e Amanhã. Continuou trazendo muitos resultados e pode gerar falso negativo (ticket atribuído antes da primeira resposta). Experimental, não validada, **descartada** na v0.1.4.25.

## v0.1.4.25 — Governador de chamadas, cache e contexto `[tickets] [logs-auditoria]`
Restaura a confirmação por `stats.first_responded_at` e remove `responder_id`. Cria cache global de chamadas idênticas (60 s), deduplicação de requisições simultâneas, fila global com intervalo mínimo de 5 s, bloqueio mínimo de 60 s (ou `Retry-After` maior) em HTTP 429, autorregulação a 10 s quando `X-RateLimit-Remaining <= 5` e contexto por `ticket_id + updated_at`. Separa cliente, governador, cache, deduplicador, estado de limite, contexto e classificador em pastas (`src/api/freshdesk/`, `src/services/tickets/`). Primeira execução ficou lenta. Implementada, aguardando validação.

## v0.1.4.26 — Concorrência configurável `[tickets]`
Adiciona `FRESHDESK_REQUEST_CONCURRENCY` (padrão 1, faixa de 1 a 10) e substitui a fila estritamente serial por governador com fila pendente e limite de chamadas ativas; o intervalo mínimo continua valendo entre inícios de chamada. O instalador acrescenta a variável ao `.env` sem substituir o existente; `.env.example` passa a ser opcional. Com `FRESHDESK_REQUEST_MIN_INTERVAL_MS=0`, funcionou uma vez e o HTTP 429 voltou: intervalo zero não é considerado seguro. O servidor caiu por rejeição não tratada (corrigido na v0.1.4.27).

## v0.1.4.27 — Servidor não cai mais em HTTP 429 `[tickets]`
Corrige a rejeição não tratada criada por `promise.finally()` no deduplicador; a limpeza passa a usar `promise.then(cleanup, cleanup)` e a rejeição original segue para o tratamento normal. O HTTP 429 é uma resposta controlada e não encerra o processo Node.js. Implementada, aguardando validação.

## v0.1.4.28 — Sem registro `[padrao-patch]`
Nenhum UPDATE, FLOW ou menção em outro documento. O UPDATE da v0.1.4.29 cita instalador corrigido, o que sugere falha de instalação, mas isso não está documentado. Confirmar com o usuário.

## v0.1.4.29 — Aba Logs ativada `[logs-auditoria]`
Ativa a aba Logs com modos `off`, `summary` e `full`; memória circular, arquivos diários, sanitização e API incremental (rotas `GET /api/logs`, `GET /api/logs/status`, `GET /api/logs/today`, `DELETE /api/logs`, `PUT /api/logs/settings`). Instrumenta o cliente Freshdesk (cache, deduplicação, conclusão, HTTP 429). Instalador corrigido para não exigir hash exato do `.env.example`; as opções de log são mescladas sem substituir conteúdo local. Implementada, aguardando validação.

## v0.1.4.30 — Contexto da busca nos logs `[logs-auditoria] [tickets]`
Registra tipo e rótulo da busca, distingue `search_tickets` de `ticket_details_stats`, registra `ticketId` apenas em contexto sanitizado e `X-RateLimit-Used-CurrentRequest`, e gera resumo (candidatos, analisados, retornados, context hits, detail requests, unidades estimadas) e a interrupção por rate limit. Os logs mostraram o custo real: 1 crédito na Search API e 2 em cada detalhe com stats. Não altera a regra da busca.

## v0.1.4.31 — Aberto e Pendente explícitos na busca fixa `[tickets]`
Só `principal.js`: a busca fixa `1ª Resposta` passa a preencher e enviar `Aberto` e `Pendente`, mantendo grupos e prazos. Teste para ver se a quantidade inicial de candidatos diminui.

## v0.1.4.32 — Listagem em lote com stats (teste) `[tickets]`
A busca `1ª Resposta` usa `/api/v2/tickets?include=stats&updated_since=...&per_page=100&page=1` e filtra localmente; zero consultas individuais; uma chamada, 2 créditos, cerca de 1 s. A página genérica nem sempre contém os candidatos da busca (cobertura incompleta). **Superada** pela v0.1.4.33.

## v0.1.4.33 — Fluxo híbrido `[tickets]`
Search API pré-seleciona por grupos, status e prazo; usa o menor `updated_at` dos candidatos menos margem configurável (padrão 5 s); lista tickets com `include=stats`, `order_by=updated_at` e `order_type=asc`; cruza por ID e para quando todos forem encontrados. Registra páginas, candidatos, IDs ausentes e consumo real. Teste real: 26 candidatos, 4 páginas com stats, 309 tickets examinados, 9 créditos, 2,6 s, sem 429, cobertura completa. ADD v004.

## v0.1.4.34 — Cache persistente de stats `[tickets]`
Cria `data/cache/freshdesk-ticket-stats.json`, chaveado por `ticket_id + updated_at`; reutiliza stats quando o candidato não mudou, invalida versões anteriores, consulta páginas só para ausentes do cache, grava apenas campos técnicos mínimos, com escrita atômica, TTL padrão de 168 h e máximo de 2000 entradas.

## v0.1.4.35 — Fluxo híbrido por `frDueBy` `[tickets]`
O fluxo híbrido passa a ser ativado por `frDueBy` preenchido, não só por `searchType=primeira_resposta`; busca fixa, personalizada e salva com Vencidos, Hoje ou Amanhã usam a mesma lógica e o mesmo cache; nenhuma delas faz consulta individual `ticket_details_stats`. O resumo final passa a registrar `apiUnitsConsumed` real, sem a estimativa antiga. Não altera buscas sem `frDueBy`. **Validada como nova base de arquitetura (ADD v004)**, com validações de uso continuado ainda abertas.

## Pendente de registro formal
- UPDATE/FLOW das v0.1.4.9–v0.1.4.13, v0.1.4.17, v0.1.4.19 e v0.1.4.28 (ver `registros/pendencias`, P4-1208).
- Fusão do frontend em `principal.*` (duas rodadas de correção anteriores/paralelas à v0.1.3.0) — ver `registros/erros-e-licoes` e `10-layout-frontend`.
- Preparação inicial do túnel Cloudflare e da estrutura de pastas — ver `01-preparacao`.
