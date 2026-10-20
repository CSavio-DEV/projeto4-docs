# Erros e lições — Projeto4

**Assunto:** transversal (todos)
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

Erros já corrigidos e a lição reaproveitável de cada um. Detalhes de reprodução ficam nos originais arquivados; aqui fica o que evita repetir o erro em outro projeto.

## Fusão de páginas HTML em um arquivo único `[layout]`

**Causa raiz geral:** ao compilar `dashboard.html`, `portal.html`, `tickets-freshdesk.html`, `automacoes.html`, `configuracoes.html` e `logs.html` (cada um com uma versão `-mobile.html`) em `principal.html` + `principal.css` + `principal.js`, o `styles.css` compartilhado foi mantido, mas **cada página original tinha um `<style>` próprio no `<head>`** com ajustes (padding de cartões, alvos de toque maiores, faixa de usuário mobile) que nunca foram promovidos ao CSS compartilhado. Quase todos os bugs das duas rodadas de correção vieram dessa mesma causa.

**Lição:** ao fundir várias páginas com CSS "local" por página em um arquivo único, migrar também o `<style>` embutido de cada uma para o CSS compartilhado — não apenas o CSS já compartilhado.

### Erros específicos e correção
- **Menu mobile não abria:** faltava `display:flex` em `.menu-open .sidebar`; havia também dois breakpoints diferentes (760px e 860px) — unificados em `860px`.
- **Switches de Automações não respondiam fora do topbar:** o `<input type="checkbox">` ficava coberto por `.track`/`.thumb`; corrigido com `pointer-events: none` em `.switch .track`, deixando o clique passar para o input.
- **Cartões sem padding** (`.overview-card`, `.results-card`, `.list-card`): existiam só no `<style>` de cada página original; restaurados no CSS compartilhado.
- **Contador "próximo disparo" e toggle global sumidos do topbar (desktop):** o novo topbar havia trocado esse bloco por um título dinâmico; os dois elementos foram restaurados.
- **Lista de tickets no mobile virava tabela com rolagem horizontal:** a marcação de cartão (`.ticket-card`) do HTML mobile original não existia no arquivo único. **Padrão reaproveitável:** manter os dois blocos (tabela e cartões) sempre no DOM e alternar por CSS/breakpoint, em vez de alternar por JS — evita sincronizar estado entre duas representações da mesma lista.
- **Steppers de intervalo só visuais:** caíam no handler genérico de placeholder por não estarem na lista de exclusão. **Padrão reaproveitável:** guardar o estado num `data-*` do próprio elemento (`data-value`) deixa o componente autocontido e fácil de duplicar.
- **Scroll deslocando a página, definitivo:** a causa real não era o `.content` (que já escondia sua própria barra), e sim `html`/`body`, porque `.app-shell{ min-height:100vh }` permite a página crescer além da tela. **Padrão reaproveitável:** ao esconder uma barra de rolagem, checar qual elemento realmente rola — `min-height` num container flex é sinal de alerta, não `height` fixo.
- **Contador e nome do usuário sumidos só no mobile:** o `.topbar` inteiro vira `display:none` no mobile, e a faixa de usuário/contador de cada página original vivia só no `<style>` local, nunca no CSS compartilhado. Restauradas uma única vez no `principal.html`, reaproveitando os mesmos atributos `data-*` que `principal.js` já preenche — nenhuma mudança de JS foi necessária.

## HTTP 400 e regra errada em "1ª Resposta" `[tickets]`
Causas somadas: regra baseada só em `fr_due_by`, ausência do filtro visual de prazo, busca fixa permanecendo ativa após alterar filtros, consultas personalizadas enviadas com `searchType=primeira_resposta` e, na correção da v0.1.4.2, o uso de `fr_escalated` como campo de busca. A Search API da conta rejeita esse campo (`invalid_field`), e datas em formato diferente de `YYYY-MM-DD` geravam `Validation Failed`. Consultas diferentes chegaram a devolver IDs repetidos e totais inesperados. Resolvido nas v0.1.4.9 a v0.1.4.13: sem `fr_escalated`, datas `YYYY-MM-DD` e classificação final no backend com `stats.first_responded_at`. **Lição:** um campo que aparece no JSON do ticket não é necessariamente pesquisável; testar cada campo na Search API da conta real antes de basear uma regra nele, e validar o resultado contra tickets conhecidos e a visão oficial do Freshdesk.

## HTTP 429 por uso simultâneo da mesma conta `[tickets]`
Durante a validação da 1ª Resposta ocorreu HTTP 429 porque Projeto3 (alertas ativos), Projeto4, scripts de inspeção e buscas repetidas usavam a mesma cota da conta Freshdesk; a busca faz uma chamada `include=stats` por candidato, o que aumenta o consumo. Após cerca de cinco segundos voltou a responder. **Lição:** o limite é da conta, não do usuário nem do processo; controle de intervalo, cache e controle global são pré-requisito do teste multiusuário (TCK-0105).

**Atualização (v0.1.4.14 a v0.1.4.35):** a causa deste 429 foi medida nos logs e está em "Créditos da API, não concorrência" mais abaixo.

## Busca salva apagada retornava ao reiniciar `[tickets]`
`savedSearches.js` reinjetava buscas conhecidas a cada leitura. Corrigido na v0.1.4.6: o JSON passou a ser a única fonte de verdade, sem inserção, reordenação ou regravação automática.

## `Qualquer` aparecia como opção selecionável `[tickets]`
O placeholder tinha sido incluído como `<option>` e virava checkbox no multisseletor. Corrigido: removido do menu aberto, mantido só no botão fechado, com `text-faint`.

## `$(...).forEach is not a function` `[tickets]`
Uso do seletor `$` (retorna um elemento) como se retornasse uma coleção. Corrigido trocando por `$$` em `markSearchAsUnsavedCustom()`. **Lição:** conferir sempre se o seletor usado é o unitário ou o de coleção antes de encadear `.forEach`. O mesmo erro reapareceu na v0.1.4.13, na atualização dos botões de tipo de busca, e foi corrigido do mesmo modo: ao criar ou alterar qualquer função que percorra elementos, revisar todos os usos de `$` no arquivo.

## `favicon.ico` 404 `[tickets] [layout]`
Corrigido com favicon vazio embutido no `<head>`, sem depender de arquivo adicional.

## Texto literal `"\n"` no topo da página `[layout]`
Inserção inadequada durante o patch do favicon; corrigida manualmente. **Lição, válida para todos os próximos patches:** partir sempre do `principal.html` atual do projeto como base (nunca reconstruir a partir de uma versão anterior) e validar `view-source` e o DOM após qualquer alteração no `<head>`.

## Valor de exemplo em formato de chave de API citado em documentação `[acesso-seguranca]`
O UPDATE da versão que introduziu a integração de Tickets citou, ao registrar a remoção do campo visual de API Key do HTML, um valor no formato de chave de API. Confirmado: era um valor de exemplo, não a credencial real — a chave real está protegida apenas no `.env`. **Lição, mesmo assim:** não citar nenhum valor no formato de credencial (real ou de exemplo plausível) em UPDATE, FLOW, changelog ou qualquer documento; usar um placeholder óbvio (`<chave-exemplo>`) evita a dúvida de segurança que esse caso gerou.

## Página de erro do localhost desligado ainda incompleta `[preparacao]`
O launcher (`abrir_projeto4.cmd`) já abre um `servidor-desligado.html` local quando o servidor Node falha ao iniciar, mas o acesso pela URL pública do Cloudflare continua caindo na página de erro genérica da Cloudflare quando o serviço local está fora do ar — situação diferente da que o launcher cobre. Ver pendência em `registros/pendencias`.

# Erros e lições de 2026-09-29 (v0.1.4.14 a v0.1.4.35)

## `per_page` na Search API gerava HTTP 400 `[tickets]`
A v0.1.4.18 passou `per_page=30` a `/search/tickets` para limitar a primeira página; a Search API rejeitou o parâmetro. Corrigido na v0.1.4.21 (removido `per_page`, mantido `page=1`). A Search API trabalha com páginas de até 30 candidatos; a paginação configurável fica para a aba Configurações. **Lição:** parâmetros que existem na listagem não valem na Search API; testar cada parâmetro no endpoint certo antes de montar o patch.

## Patch aplicado sobre base divergente (v0.1.4.17) `[padrao-patch]`
O patch falhou por base divergente e não alterou nenhum arquivo, que é o comportamento esperado do aplicador. A correção (v0.1.4.18) foi refeita diretamente sobre os arquivos reais da v0.1.4.16. **Lição:** nunca montar patch sobre arquivos reconstruídos; pedir os arquivos completos instalados. Uma falha de base não gera ADD nem entra na lista de versões válidas.

## Regra visual interpretada errado (v0.1.4.19) `[layout]`
A correção de cor de `Personalizada (não salva)` foi proposta com a regra mal entendida e não foi aplicada; a v0.1.4.20 refez com a regra certa (`Personalizada` dourada, `(não salva)` cinza, seta dourada). **Lição:** quando a regra visual tem duas partes (texto e estado), descrever a interpretação por escrito, item ativo e inativo, e confirmar antes de gerar o patch.

## Créditos da API, não concorrência, causavam o HTTP 429 `[tickets]`
A busca `1ª Resposta` fazia 1 chamada à Search API mais 1 `GET /tickets/{id}?include=stats` por candidato. Os logs (v0.1.4.30) mostraram o custo real: **1 crédito** na Search API e **2 créditos** em cada detalhe com stats. Numa execução: 1 + 24 × 2 = **49 créditos**; com 1 crédito restante, o sistema iniciou outro detalhe de 2 créditos e recebeu 429. Reduzir a concorrência de 4 para 1 (v0.1.4.22) baixou o pico, mas não o total. **Lição:** o gargalo é o total de créditos da janela, não o paralelismo. Medir o custo por tipo de chamada nos cabeçalhos (`X-RateLimit-Used-CurrentRequest`, `X-RateLimit-Remaining`) e comparar o crédito restante com o custo da próxima operação antes de iniciá-la. O resumo deve usar o consumo real, não estimativa.

## Indicador "parecido" no lugar do definitivo (v0.1.4.23 e v0.1.4.24) `[tickets]`
Sem `stats`, a classificação por `fr_escalated`/status/`fr_due_by` trouxe tickets a mais; acrescentar `responder_id == null` ainda trouxe muitos e pode gerar falso negativo, porque um ticket pode ser atribuído antes da primeira resposta. Ambos foram descartados. **Lição:** o campo definitivo (`stats.first_responded_at`) não é substituível por indicadores aproximados; economizar chamadas muda a forma de obtê-lo (lote, cache), não o critério.

## Queda do servidor Node.js em HTTP 429 `[tickets]`
A limpeza do deduplicador usava `promise.finally()`; o ramo derivado herdava a rejeição da Promise original, e a rejeição não tratada encerrava o processo. Corrigido na v0.1.4.27 com `promise.then(cleanup, cleanup)`, preservando a rejeição original para o fluxo normal. **Lição:** `finally()` devolve uma nova Promise que continua rejeitada; ao encadear limpeza, tratar o ramo derivado e testar o caminho de erro (429 forçado), não só o de sucesso.

## Intervalo mínimo de 0 ms parecia seguro `[tickets]`
Com `FRESHDESK_REQUEST_MIN_INTERVAL_MS=0` e concorrência 1, um teste passou e outro voltou a dar 429. **Lição:** um teste verde não prova margem de segurança; só se considera seguro um intervalo validado em várias execuções com os logs de consumo.

## Listagem genérica com stats: rápida, mas incompleta `[tickets]`
A primeira versão em lote (v0.1.4.32) leu uma página de 100 tickets genéricos (1 chamada, 2 créditos, cerca de 1 s). A página não representava os candidatos da busca: 100 listados → 20 Abertos ou Pendentes → 10 nos grupos permitidos → 1 sem primeira resposta → 0 em Vencidos ou Hoje. Também foi confirmado que `?id=8021,8022,...` é ignorado pela listagem. Resolvido com o fluxo híbrido (v0.1.4.33). **Lição:** economia sem garantia de cobertura gera resultado errado e silencioso; pré-selecionar com a Search API e conferir por ID que todos os candidatos foram encontrados, marcando `complete: false` quando não forem.

## Fluxo decidido pelo rótulo da busca, não pelo filtro `[tickets]`
Ao incluir `Amanhã` ou mudar um filtro, a busca fixa virava Personalizada e, como o fluxo em lote valia só para `searchType=primeira_resposta`, voltava ao fluxo individual e ao 429. Corrigido na v0.1.4.35: o fluxo é ativado por `frDueBy` preenchido. **Lição:** escolher o caminho de execução pela característica da consulta (o que ela precisa), nunca pelo nome ou tipo que a interface lhe dá.

## Instalador exigindo hash exato de arquivo de configuração local (v0.1.4.29) `[padrao-patch]`
O instalador da aba Logs exigia o hash exato de `.env.example` (arquivo de configuração local). Corrigido na v0.1.4.29: as opções de log são mescladas sem substituir conteúdo local (a v0.1.4.26 já tornara `.env.example` opcional e acrescentava variáveis ao `.env` sem sobrescrever). **Lição:** arquivos de configuração do usuário não entram na verificação de base por hash exato; o aplicador mescla, nunca substitui, e cria o arquivo só se não existir.

## Observação em aberto: arquivo de log com a data do dia seguinte `[logs-auditoria]`
Em 29/09/2026, ainda no horário local, o sistema criou o arquivo diário com data 30/09. Suspeita: `new Date().toISOString()` (UTC) para o nome do arquivo, enquanto o conteúdo usa o fuso de `PROJETO4_LOG_TIMEZONE`. Ainda **não corrigido** (ver `registros/pendencias`, P4-1005); ao corrigir, mover esta entrada para o formato de erro corrigido, com a lição.
