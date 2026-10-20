# Portal

**Assunto:** Portal
**Versão do código de referência:** Projeto3 v0.8.0.7.0 (ainda não migrado ao Projeto4)
**Data:** 2026-09-29
**Status:** reservado no Projeto4 — referência do Projeto3 completa; regra de acesso ao Portal já decidida (implementação pendente)

## Escopo
Login real no Portal, busca manual de usuário, busca contextual a partir do ticket, bloqueio por ausência de sessão do Portal e as regras já validadas de busca de usuário. Nada disto foi migrado para o Projeto4 ainda (etapa 7, `P4-0701` a `P4-0704`); este documento registra como funciona no Projeto3, para servir de contrato na migração.

## Regra de acesso já decidida (2026-09-28)
O acesso ao Portal é uma capacidade opcional da **sessão principal** do usuário — a identidade vem do Cloudflare Access, não do login do Portal. Duração inicial da sessão do Portal: 24 horas a partir do login bem-sucedido. Sem sessão válida do Portal, a busca manual e a busca contextual ficam indisponíveis, com bloqueio também no backend (não só sinalização visual). Decisão adotada; implementação e teste ainda pendentes. Detalhes completos em `02-acesso-seguranca/README.md`, seção "Sessão principal e acesso opcional ao Portal".

## Como funciona no Projeto3

### Login e contexto
`src/portal/authService.js` faz o login real no Portal (ou simulado, se `AUTH_MOCK_ENABLED=true`); a sessão retorna um token e, com ele, `src/portal/contextoService.js` lê o contexto do usuário (`lerContexto()`), permite trocar de contexto (`trocarContexto()`), personificar outro usuário (`personificar()`/`despersonificar()`) e buscar plataformas do estudante e unidades por nome. A sessão do Portal fica associada à sessão do Projeto3 via `src/auth/session.service.js` (`createPortalSession`), que também calcula as permissões pelo perfil (`permissoes.service.js` — ver `legado-projeto3`).

### Busca manual de usuário
Rota `POST /api/portal/usuarios/buscar` (autenticada, `src/routes/portal.routes.js`) chama `executarBuscaManualPortal()` em `src/portal/portal-search.service.js`. O resultado fica em cache em `data/portal-last-search.json` e pode ser recuperado (`GET /api/portal/usuarios/ultima`) ou limpo (`POST /api/portal/usuarios/limpar`) sem repetir a busca. Em modo mock, a busca usa `buscarUsuarioPortalMock()` (`usuarios.service.js`) e devolve um resultado simulado.

### Busca contextual a partir do ticket (mineração + Portal)
`src/freshdesk/busca-usuario-portal-stable.service.js` (1500+ linhas) é o motor mais complexo do Projeto3: a partir do texto do ticket, tenta detectar candidatos a usuário (nome, e-mail, RA) e cruza com o Portal. Trabalha com um lock (`adquirirPortalBuscaLock`/`liberarPortalBuscaLock`) para não rodar duas buscas ao mesmo tempo, tem configuração e status próprios (`getBuscaUsuarioPortalStableConfig`/`Status`) e é usada tanto pela pré-visualização de automação (`POST /api/automacoes/busca-usuario-portal/preview`) quanto pelo fluxo de reset de senha. O estado por ticket é registrado por `src/freshdesk/busca-usuario-portal-estado.service.js`.

Auxiliar: `src/freshdesk/usuario-detector-local.service.js` detecta possíveis usuários diretamente no texto do ticket, sem depender do Portal — usado como primeira triagem.

### Domínios de e-mail (apoio à busca)
`freshdesk/db/dominios_corporativos.json`, `dominios_pessoais.json`, `dominios_erros.json` e `dominios_insights_regionais.json` guardam listas de domínios de e-mail para ajudar a classificar e-mails encontrados no texto do ticket (corporativo vs. pessoal vs. erro de digitação). Conteúdo não copiado para a documentação (são listas, não código, mas ficam fora por precaução e porque não mudam a lógica).

## Regras já validadas a preservar na migração (`P4-0704`)
- prioridade e classificação de requerentes (quem pediu o ticket tem prioridade sobre nomes citados no corpo);
- não repetir o requerente como um "usuário encontrado" separado;
- reclassificar nome de rodapé de e-mail semelhante ao do requerente;
- não usar RA como critério de busca quando essa regra estiver vigente;
- rótulo `Não encontrado(s):` para quem não foi localizado;
- marcadores internos de controle só no JSON baixado, nunca na interface.

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0701` a `P4-0704`, `P4-0207` a `P4-0209` (implementação da sessão principal e da sessão do Portal). Depende também de `P4-0202` (bloqueio de recursos do Portal sem sessão) — o Projeto4 substitui a autenticação direta do Projeto3 por login opcional sobre a mesma sessão do Cloudflare Access (ver `02-acesso-seguranca`).

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
