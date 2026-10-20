# Decisões registradas — Projeto4

**Assunto:** transversal (todos)
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

## Preparação e infraestrutura `[preparacao]`
1. Raiz do projeto: `parte4_cloudflare`.
2. Servidor local em `127.0.0.1`, porta `65004`.
3. Nenhuma dependência acidental das portas usadas pelo Projeto3 (3003/3004); Projeto3 e Projeto4 devem poder rodar ao mesmo tempo sem compartilhar sessão, cookie ou estado.
4. Sem abertura de porta no roteador e sem port forwarding — o acesso externo é só pelo túnel Cloudflare.
5. Versões V2, V3, V4 e evoluções V4.x podem coexistir em hostnames e portas separados.
6. O Projeto3 real só será conectado ao Projeto4 depois das revisões de autenticação, autorização, sessão e auditoria.
7. A Hostinger deixou de fazer parte do fluxo de rede do projeto (nameservers já apontam para a Cloudflare); pode voltar a ser usada como servidor de arquivos e imagens.

## Acesso e segurança `[acesso-seguranca]`
8. Acesso externo protegido por Cloudflare Access; primeiro método de login validado é e-mail autorizado com One-Time PIN.
9. Cada novo usuário deve ter e-mail individual; o e-mail operacional do projeto não deve virar conta compartilhada.
10. Credenciais reais nunca entram em documentos, patches ou repositório.
11. Tunnel Token é secreto e deve ser rotacionado se exposto.
12. API Token da Cloudflare não foi necessário para a instalação inicial.
13. Login do Portal é opcional dentro do Projeto4; ignorá-lo não impede o uso geral da aplicação, mas desabilita toda função de busca no Portal enquanto não houver sessão.
14. O nome exibido no topo tenta a identidade confiável do Cloudflare Access primeiro; sem isso, usa `Usuário Projeto4` ou outro nome genérico aprovado — nunca o login do Portal como identidade principal do Projeto4.
15. A aplicação é preparada inicialmente para até 5 usuários simultâneos, com sessão, estado e auditoria separados por usuário.

## Layout e frontend `[layout]`
16. `principal.html` único é o padrão do Projeto4 — não se mantêm dois sistemas de HTML (desktop/mobile) separados como no Projeto3.
17. A consolidação do layout é a primeira pendência funcional da migração, antes de conectar qualquer funcionalidade.
18. O layout consolidado deve se aproximar estruturalmente do Projeto3, sem copiar a aparência antiga.
19. CSS, botões, cores, proporções e organização do layout só mudam mediante solicitação específica.
20. Desktop é consolidado e validado antes da implementação mobile compartilhada.
21. Login pode continuar como página separada do shell principal.

## Tickets Freshdesk `[tickets]`
22. Valores especiais de filtro: `Qualquer` (ignora o filtro, não selecionável, só placeholder), `Nenhum` (campo nulo, selecionável) e `Não atribuído` (grupo nulo, selecionável) — ver `03-tickets-freshdesk/filtros-e-buscas.md`.
23. Apenas Abertos Portal N1 e 1ª Resposta são buscas de sistema; Personalizada é busca normal.
24. `data/saved-searches.json` é a única fonte de verdade das buscas salvas; o projeto nunca reinjeta, reordena ou regrava o conteúdo por conta própria.
25. O detalhe do ticket deve sempre ser renderizado visualmente, nunca como JSON bruto.
26. Nenhum controle envia conteúdo do editor de Resposta/Anotação automaticamente.

## Agentes de IA `[agentes-ia]`
27. A tela de downloads de tickets do Projeto3 (Detectar/Baixar/Continuar/Parar) fica fora do escopo do Projeto4; usam-se apenas os dados já minerados por ela.
28. A mineração por tag (ex.: `acess_senha`), com classificação de frases em palavras fortes, médias, fracas e excludentes, é a base para as automações de IA e já funciona no Projeto3.

## Padrão de patch `[padrao-patch]`
29. Todo patch é entregue como um único ZIP `Projeto4_patch_vX.Y.Z.N_AAAA-MM-DD_descricao.zip`, com versão-base declarada, backup em `_backup_patch/vX.Y.Z.N/`, documentação (`UPDATE`, `FLOW`, `README_PATCH`, `manifest.json`) e validação de sintaxe/hash antes da entrega.
30. Correção de um patch já entregue usa o próximo número de versão — nunca se reutiliza um número já entregue para conteúdo diferente.
31. O ADD não é gerado a cada patch: registra a **conclusão validada de uma tarefa** e só é criado depois da validação do usuário (ver decisão 46, que esclarece esta regra). O documento original `Projeto4_padrao_ADD_arquivos.md` não foi localizado; o padrão foi reconstruído em `11-padrao-de-patch/Projeto4_padrao_ADD_arquivos.md` (P4-1207, concluída em 2026-10-20).
32. `.env`, credenciais reais, cookies, sessões, logs reais e dados de usuário em `saved-searches.json` nunca entram em um patch comum.

## Design do front-end (herdado do Projeto3, vigente para telas ainda não migradas) `[layout]`
33. Paleta grafite quente com acento âmbar único para ações primárias; verde e vermelho só para estado real (sucesso/alerta), nunca decoração.
34. Tipografia: Space Grotesk para títulos/rótulos, Inter para corpo e dados de tabela; sem uma terceira família.
35. Escala de espaçamento em variáveis de 4 px (`--sp-1` a `--sp-9`), sem valores soltos.
36. Mobile: inputs com `font-size` mínimo de 16px, alvos de toque de 40–44px, tabelas largas viram cartões empilhados, painéis laterais fixos viram folha (sheet) de baixo para cima, e uso de `env(safe-area-inset-*)`.
37. Nunca usar dado real (nome completo, CPF, e-mail, IP, número de ticket) em exemplo de tela — sempre fictício.

## Decisões posteriores (2026-09-28 e 2026-09-29)

### Acesso e sessão `[acesso-seguranca]`
38. O Projeto4 tem uma única identidade principal por usuário: a do Cloudflare Access. O login no Portal é uma capacidade opcional da mesma sessão principal e nunca cria outra identidade nem altera a autoria.
39. A sessão do Portal dura inicialmente 24 horas a partir do login bem-sucedido, mas a validade real é verificada antes de cada operação protegida. Decisão adotada; implementação e teste pendentes.
40. Sem sessão válida do Portal, as rotas protegidas são bloqueadas no backend, e não só sinalizadas no frontend.
41. O mecanismo de persistência de sessão do Portal do Projeto2 será analisado e adaptado (isolamento por usuário, web, cookies seguros, auditoria); arquivos de sessão do Projeto2 não são copiados diretamente.
42. Os logs do Portal não registram senha, cookie completo, token, cabeçalhos de autenticação nem credenciais reutilizáveis; o nome usado no login do Portal nunca vira identidade principal.

### Tickets Freshdesk `[tickets]`
43. Na busca 1ª Resposta, `fr_escalated` não é usado (a Search API o rejeita). O indicador de ticket já respondido é `stats.first_responded_at`, obtido com `include=stats`; Vencidos, Hoje e Amanhã são classificados pelo backend com `fr_due_by` completo em `America/Sao_Paulo` (validado na v0.1.4.13).
44. Na busca salva 1ª Resposta vêm marcados Vencidos e Hoje; Amanhã fica disponível e desmarcado.
45. O controle do limite de chamadas da API (intervalo mínimo de 5 s entre pesquisas; bloqueio de 10 s após HTTP 429, respeitando `Retry-After` maior) precede o teste multiusuário. *(Camada de frontend, implementada e validada na v0.1.4.14, ADD v002. O backend tem governador próprio, com valores diferentes — ver decisão 52.)*

### Documentação e método de trabalho `[padrao-patch]`
46. UPDATE registra as mudanças de uma versão; FLOW registra o funcionamento resultante; ADD registra a conclusão validada da tarefa. Uma tarefa pode exigir vários patches antes do ADD, que não significa necessariamente que um arquivo novo foi criado. A pasta `_a-incorporar/` não substitui nenhum dos três. Os ADDs validados ficam em `adds-validados/`.
47. Estados documentais: Planejado, Pendente, Em implementação, Parcialmente implementado, Implementado (aguardando validação), Parcialmente validado, Validado, Superado por versão posterior, Descartado como base de código, Legado funcional do Projeto3, Regressão conhecida. Função visível no frontend não é funcional sem conexão e teste do backend; alteração implementada não é validada só por passar em verificação sintática.
48. Base de código do Projeto3 V2.1: `v0.8.0.6.3`. As versões `v0.8.0.6.4` a `v0.8.0.6.7` contêm erros, regressões e rollbacks e não são base de código; servem para erros e lições, regras preventivas e procedimentos de rollback.
49. Todo novo patch parte dos arquivos completos atualmente instalados (a versão instalada indicada em `onde_paramos.md`; em 2026-10-20, a `v0.1.4.35`), nunca de arquivos anteriores à última validação; trabalho em partes pequenas, com objetivo limitado e layout validado preservado.

## Decisões de 2026-09-29 (v0.1.4.14 a v0.1.4.35)

Fontes: UPDATE de cada versão, ADD v002, v003 e v004 e o estado de 2026-09-29. Cada decisão indica se está validada ou aguardando validação.

### Tickets, interface `[tickets] [layout]`
50. Explicações longas e detalhes técnicos saem da área permanente da aba Tickets e ficam no painel do botão `i` (fecha por `×`, clique fora e `Esc`). Permanecem visíveis só carregamento, erros de consulta, falhas de conexão, aviso de HTTP 429, contagem de bloqueio, confirmação ou erro de busca salva e mensagens que exigem ação. O nome da busca ativa aparece uma única vez, no seletor principal. *(Validada, v0.1.4.15, ADD v003.)*
51. Seletor de buscas: até cinco buscas salvas visíveis (as duas de sistema contam; até três adicionais, na ordem do JSON); `Personalizada (não salva)` sempre por último; o nome `Personalizada` é reservado, com as variantes `Personalizada não salva` e `Personalizada (não salva)`, comparados sem diferenciar caixa nem espaços externos, no frontend e no backend; ao entrar em Personalizada a partir de uma busca salva os filtros são limpos uma única vez; busca sem filtros pede confirmação. Visual: `Personalizada` dourada e `(não salva)` cinza no botão e no item ativo do menu. *(Implementada nas v0.1.4.16 a v0.1.4.20, aguardando validação; as cores foram aceitas pelo usuário.)*

### Limite da API e fluxo da 1ª Resposta `[tickets]`
52. O controle de chamadas ao Freshdesk tem duas camadas. **Frontend** (decisão 45): 5 s entre pesquisas, sem pesquisas simultâneas, bloqueio de no mínimo 10 s após 429. **Backend** (governador, v0.1.4.25 e v0.1.4.26): fila global, intervalo mínimo padrão de 5 s, concorrência configurável (padrão 1, faixa de 1 a 10), cache de chamadas idênticas por 60 s, deduplicação, bloqueio de no mínimo 60 s (ou `Retry-After` maior) após 429 e intervalo de 10 s quando `X-RateLimit-Remaining <= 5`. Intervalo mínimo de 0 ms não é considerado seguro. *(Backend aguardando validação. A diferença entre 10 s e 60 s ainda precisa ser conciliada — ver `registros/pendencias`.)*
53. HTTP 429 é resposta controlada, nunca motivo para encerrar o processo Node.js. *(v0.1.4.27, aguardando validação.)*
54. `responder_id` não é critério de primeira resposta (um ticket pode ser atribuído antes de ser respondido), e classificar sem `stats` deixa tickets a mais. Complementa a decisão 43: o indicador continua sendo `stats.first_responded_at`. *(Testes das v0.1.4.23 e v0.1.4.24, descartados na v0.1.4.25.)*
55. Toda busca com `frDueBy` preenchido (Vencidos, Hoje, Amanhã ou combinação) usa o fluxo híbrido e o cache persistente, qualquer que seja o tipo ou nome da busca (fixa, personalizada ou salva). Fluxo: Search API pré-seleciona por grupos, status e prazo → cache por `ticket_id + updated_at` → para os ausentes, listagem com `include=stats` a partir do menor `updated_at` menos margem de 5 s → cruzamento por ID → confirmação por `first_responded_at` nulo → classificação. Não voltar à consulta individual por ticket como fluxo padrão, e não alterar o fluxo sem analisar logs e código completo atual. *(ADD v004, v0.1.4.33 a v0.1.4.35: validado como nova base de arquitetura; validações de uso continuado ainda abertas.)*
56. Cache persistente de stats em `data/cache/freshdesk-ticket-stats.json`: chave `ticket_id + updated_at`, somente campos técnicos mínimos (nunca descrição, conversa, e-mail, CPF, anexo, cookie, token ou API key), escrita atômica, TTL padrão de 168 h e máximo de 2000 entradas, compartilhado por qualquer busca que precise de stats. *(v0.1.4.34, aguardando validação continuada.)*
57. Cobertura sempre explícita: se o limite de páginas for atingido com candidatos ausentes, a resposta traz `complete: false` e `missingCandidateIds`; resultado incompleto nunca é apresentado como completo. Limites configuráveis: até 10 páginas na Search API (30 candidatos por página), até 10 páginas com stats (100 tickets por página). O resumo final usa o consumo real (`apiUnitsConsumed`). *(Apresentação visual do resultado parcial no frontend ainda pendente.)*

### Logs `[logs-auditoria]`
58. A aba Logs tem os modos `off`, `summary` e `full`, memória circular (500 entradas por padrão), arquivos diários e sanitização antes de gravar. Os logs técnicos nunca registram API key, `Authorization`, cookie completo, token, senha nem dados pessoais de tickets; preferem `ticket_id`, status, horário e motivo técnico. O fuso vem de `PROJETO4_LOG_TIMEZONE` e o nome do arquivo diário deve usar a data local desse fuso, não a data UTC. *(Aba implementada na v0.1.4.29, aguardando validação; a data do arquivo diário tem erro conhecido, ver `registros/pendencias`.)*

### Método e documentação `[padrao-patch]`
59. Alteração de logs e alteração da regra de uma busca vão em patches separados, para não misturar causa e efeito.
60. O ADD pode ser evolutivo e atualizável quando declara no topo o que ainda não foi validado (prática do ADD v004). Ciclos não validados, como v0.1.4.16 a v0.1.4.27 isolados, não geram ADD próprio. *(Prática adotada no ADD v004; confirmar como regra.)*
61. Padrão de patch v1.1: UPDATE e FLOW de cada versão ficam em `docs/_updates/` e `docs/_flows/`, e o aplicador os copia e cria as pastas. *(Inaugurado na v0.1.4.14.)*
62. Existe um único arquivo de estado, `docs/onde_paramos.md`, sobrescrito a cada sessão. Não se criam `onde_paramos_01`, `onde_estamos_01` etc. *(Definido em 2026-10-20; substitui `onde_paramos_00`, `onde_estamos_01` e `onde_paramos_02`.)*
