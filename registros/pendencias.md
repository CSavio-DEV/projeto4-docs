# Pendências — Projeto4

**Assunto:** transversal (todos)
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

Lista única de pendências. IDs no padrão `P4-xxxx`, com a etapa de origem e a tag de assunto. Itens de Tickets têm lista própria, com IDs `TCK-xxxx`, em `03-tickets-freshdesk/pendencias-tickets.md`, e não são repetidos aqui item a item — apenas referenciados.

## Legenda de estado
`pendente` · `em andamento` · `concluída (vX.Y.Z.N)` · `parcial`

## Etapa 0 — Baseline e proteção da base `[preparacao]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0001 | Criar cópia controlada do Projeto3 para o Projeto4 | pendente — legado do Projeto3 completo ainda não recebido |
| P4-0002 | Configuração independente do Projeto4 (raiz, host, porta, sem depender de portas do Projeto3) | parcial — raiz `parte4_cloudflare`, host `127.0.0.1` e porta `65004` confirmados; isolamento de cookies/logs/estado ainda não verificado |
| P4-0003 *(novo)* | Aplicar a página amigável de "localhost desligado" também quando o acesso é feito pela URL pública da Cloudflare (hoje só o launcher local cobre esse caso; o acesso via túnel ainda cai na página de erro padrão da Cloudflare) | pendente |

## Etapa 1 — Consolidação do novo layout `[layout]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0101 | Consolidar os HTML separados em um único layout | concluída (v0.1.0.0–v0.1.3.0) |
| P4-0102 | Criar mapa de componentes e estilos do novo layout | pendente |
| P4-0103 | Criar página ausente de IA Freshdesk (contêiner honesto "planejado/não implementado") | concluída (v0.1.0.0) — escopo funcional ainda em aberto, ver P4-0801/P4-0802 |

## Etapa 2 — Login, identidade e sessão `[acesso-seguranca]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0201 | Escolha de login no Portal (Entrar / Não logar) | concluída (v0.1.1.0) |
| P4-0202 | Desabilitar recursos do Portal quando o login for ignorado | concluída (v0.1.1.0), a reconfirmar após a migração real do Portal (etapa 7); o bloqueio deve valer também no backend (decisão 40) |
| P4-0203 | Definir nome exibido no topo, com ordem de fallback | em andamento — regra definida (decisão 14 em `decisoes.md`); formato exato do nome genérico ainda pendente |
| P4-0204 | Integrar identidade do Cloudflare Access com segurança (validar origem do cabeçalho/token) | pendente |
| P4-0205 | Preparar até 5 usuários simultâneos (sessão, cookies, ticket aberto, editor, auditoria por usuário) | pendente |
| P4-0207 *(novo)* | Localizar e analisar o mecanismo de persistência da sessão do Portal do Projeto2 e definir o que é aproveitável (isolamento, cookies, web, auditoria) | pendente |
| P4-0208 *(novo)* | Implementar a sessão principal individual no backend, vinculada à identidade do Cloudflare Access, e o armazenamento seguro e isolado da sessão do Portal (24 h, expiração, logout, invalidação, renovação) | pendente — depende de decidir se o logout completo encerra o Portal e a política de renovação |
| P4-0209 *(novo)* | Proteger as rotas do Portal no frontend e no backend, auditar por usuário e testar com até cinco usuários simultâneos | pendente |

## Etapa 3 — Estrutura global e Dashboard `[dashboards]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0301 | Conectar o shell consolidado à sessão (usuário no topo, estado do Portal, menu, sessão expirada) | pendente |
| P4-0302 | Migrar Dashboard (saúde do backend, automações, indicadores, avisos, Portal, Telegram) | pendente |
| P4-0303 | Padronizar rota de heartbeat/presença (`/api/frontend/heartbeat` vs `/heartbeat`) | pendente |

## Etapas 4 e 5 — Tickets Freshdesk e painel do ticket `[tickets]`
Ver `03-tickets-freshdesk/pendencias-tickets.md` (IDs `TCK-xxxx`) e `03-tickets-freshdesk/README.md`. Concluído desde a última versão deste arquivo: busca 1ª Resposta (TCK-0101, v0.1.4.13, ADD v001); limitador de pesquisas e bloqueio visual de HTTP 429 no frontend (TCK-0105, parte do frontend, v0.1.4.14, ADD v002); botão `i` e nome da busca sem repetição (TCK-0106 e TCK-0107, v0.1.4.15, ADD v003). Implementado, aguardando validação: fluxo híbrido, cache persistente e governador de API no backend (v0.1.4.25 a v0.1.4.35; arquitetura validada como base no ADD v004). Próximas e novas pendências de Tickets (TCK-0108 a TCK-0113, TCK-0102): ver `pendencias-tickets.md` e `onde_paramos.md`. IDs originais P4-0401 a P4-0406 e P4-0501 a P4-0504 seguem pendentes, exceto a lista de tickets, os filtros, os tipos de busca e as buscas salvas, já migrados e detalhados no assunto (ver "Implementado"/"Ainda não implementado" no README).

## Etapa 6 — Automações `[automacoes]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0601 | Conectar controles visuais ao estado real (ligar/desligar, scheduler, intervalos) | pendente |
| P4-0602 | Migrar alertas (ticket aberto, vencimento, ausência de tickets, repetição, histórico) | pendente |
| P4-0603 | Migrar intervenção manual (iniciar, heartbeat, finalizar, pausa/retorno) | pendente |
| P4-0604 | Revisar isolamento global vs. individual de cada estado de automação | pendente |
| P4-0605 | Migrar fluxo de reset de senha com trava de segurança (simulação padrão, confirmação humana, dependência do Portal, auditoria) | pendente |

## Etapa 7 — Portal `[portal]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0701 | Migrar a página de busca manual do Portal | pendente |
| P4-0702 | Aplicar bloqueio por ausência de sessão do Portal | depende de P4-0202 |
| P4-0703 | Migrar busca contextual a partir do ticket | pendente |
| P4-0704 | Preservar regras validadas de busca de usuário (prioridade de requerentes, reclassificação de rodapé, rótulo "Não encontrado(s):") | pendente |

## Etapa 8 — Agentes de IA `[agentes-ia]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0801 | Definir escopo antes de implementar | parcial — decidido que a tela de download (Detectar/Baixar/Continuar/Parar) fica fora do Projeto4; demais decisões (provedor, anonimização, auditoria, limites) seguem pendentes |
| P4-0802 *(novo)* | Portar para o Projeto4 a lógica de mineração de frases por tag do Projeto3 (classificação em palavras fortes/médias/fracas/excludentes), sem a tela de download | pendente |

## Etapa 9 — Configurações `[configuracoes]`
| ID | Descrição | Estado |
|---|---|---|
| P4-0901 | Mapear controles do layout para configurações reais (editável/leitura/`.env`/global/por usuário) | pendente |
| P4-0902 | Migrar configurações de Freshdesk, downloader, áudios e Telegram | pendente |
| P4-0903 | Proteger segredos (nunca completos no frontend, logs, UPDATE, FLOW ou patch) | em andamento — reforçada pela lição em `registros/erros-e-licoes` (chave exposta) |
| P4-0904 | Classificar todo controle visual sem backend como funcional, planejado ou removido | pendente |
| P4-0905 *(novo)* | Definir como o navegador mobile deve tratar os áudios de aviso (no Projeto3 funciona normal; comportamento mobile ainda em decisão) | pendente |
| — | Seção "Áudios dos avisos" na tela de Configurações do Projeto3 nunca foi totalmente especificada (upload vs. lista pré-definida, player de teste) | pendente — confirmar antes de portar a tela |

## Etapa 10 — Logs e auditoria `[logs-auditoria]`
| ID | Descrição | Estado |
|---|---|---|
| P4-1001 | Migrar visualizador do log diário | parcial — aba Logs ativa na v0.1.4.29 (modos `off`, `summary`, `full`; memória circular; arquivos diários; rotas `/api/logs`); aguardando validação; ajustes em P4-1005 a P4-1007 |
| P4-1002 | Criar auditoria individual (login, escolha do Portal, ticket aberto, resposta, alteração, download, automação, busca, reset de senha) | pendente — os logs atuais são técnicos e ainda não separados por sessão e usuário |
| P4-1003 | Adicionar filtros e retenção ao log | parcial — retenção por arquivo diário e limite de entradas existem; filtros de exibição pendentes |
| P4-1004 | Remover dados sensíveis dos logs (senhas, chaves, tokens, cookies, payload de credencial) | parcial — sanitização implementada na v0.1.4.29, aguardando validação; manter a regra para toda instrumentação nova |
| P4-1005 *(novo)* | Corrigir o nome do arquivo diário de log: usar a data local de `PROJETO4_LOG_TIMEZONE`, não a data UTC (em 29/09 o sistema criou o arquivo de 30/09 ainda no horário local; investigar `toISOString()` e a função que gera `AAAA-MM-DD.log`) | pendente — primeira tarefa de `onde_paramos.md` |
| P4-1006 *(novo)* | Aba Logs: cores discretas por nível e escopo (DEBUG, INFO, WARNING, ERROR, SYSTEM, TICKETS, FRESHDESK, CACHE, RATE-LIMIT); não rolar sozinha para o fim (só acompanhar se o usuário estiver perto do rodapé); colunas legíveis (horário, nível, escopo, operação, mensagem, detalhes); sem rolagem horizontal do documento | pendente |
| P4-1007 *(novo)* | Logs das buscas Freshdesk com mais detalhe: tipo, rótulo e nome da busca salva; filtros; grupos com nome e ID; status com código; prazos; query resumida; página; distribuição por grupo e status; menor e maior `updated_at`; `updated_since` e margem; páginas de stats; IDs encontrados e ausentes; cache hits e misses; motivo de descarte; créditos reais; cabeçalhos `X-RateLimit-*` e `Retry-After`; `complete`; duração. O resumo usa `apiUnitsConsumed` real (corrigido na v0.1.4.35; validar) | pendente |

## Etapa 11 — Mobile e responsividade `[layout]`
| ID | Descrição | Estado |
|---|---|---|
| P4-1101 | Não manter dois sistemas independentes de HTML (usar os `-mobile.html` do Projeto3 só como referência visual) | concluída como princípio (v0.1.0.0 em diante); execução completa por página ainda pendente |
| P4-1102 | Adaptar cada página ao mobile após validação desktop, na ordem: Login, Dashboard, Tickets e painel, Automações, Portal, Configurações, Logs, IA Freshdesk | em andamento — Tickets já tem cartões mobile (v0.1.0.2 e correção na fusão) |
| P4-1103 | Planejar JS adaptado da versão desktop sem duplicar regra de negócio no navegador | pendente |

## Etapa 12 — Segurança, testes e documentação `[acesso-seguranca] [padrao-patch]`
| ID | Descrição | Estado |
|---|---|---|
| P4-1201 | Sanitizar a distribuição (remover tickets reais, anexos, logs, `credentials.enc.json`, `auth_store.json`) | pendente |
| P4-1202 | Revisar persistência JSON para concorrência (escrita atômica, locks, corrupção, banco de dados futuro) | parcial — o cache de stats (`data/cache/freshdesk-ticket-stats.json`, v0.1.4.34) já usa arquivo temporário e renomeação; `saved-searches.json` e demais JSON ainda pendentes |
| P4-1203 | Criar testes mínimos automatizados | pendente |
| P4-1204 | Criar matriz de validação por página | pendente |
| P4-1205 | Corrigir o script de verificação (sintaxe, inicialização, rotas principais) | pendente |
| P4-1206 | Manter documentação por versão (`UPDATE`, `FLOW`, `SECURITY`, ADD após validação, backup em `_backup_patch/`) | em andamento — `UPDATE`/`FLOW` seguidos de v0.1.0.0 a v0.1.4.35, com as lacunas de P4-1208; `SECURITY` ainda não formalizado; existem ADD v001 a v004 |
| P4-1207 *(novo)* | Reconstruir `Projeto4_padrao_ADD_arquivos.md` a partir dos adds já produzidos (documento original não localizado) | concluída (2026-10-20) — padrão reconstruído dos ADDs v001 a v004 em `11-padrao-de-patch/Projeto4_padrao_ADD_arquivos.md` |
| P4-1208 *(novo)* | Fechar as lacunas de UPDATE/FLOW: v0.1.4.9 a v0.1.4.13 (changelog resumido a partir do ADD v001), v0.1.4.17 e v0.1.4.19 (não aplicadas) e v0.1.4.28 (sem nenhum registro) | pendente — confirmar com o usuário o que ocorreu na v0.1.4.28 |
| P4-1209 *(novo)* | Conciliar o bloqueio de HTTP 429 entre frontend (mínimo 10 s) e backend (mínimo 60 s) e documentar qual valor vale em cada camada (decisão 52) | pendente |
| P4-1210 *(novo)* | Unificar o fuso do projeto (SLA em `America/Sao_Paulo`; logs em `America/Fortaleza`) ou registrar por que são dois | pendente — decisão do usuário |
| P4-1211 *(novo)* | Manter um único arquivo de estado `docs/onde_paramos.md` | concluída (2026-10-20) — consolida `onde_paramos_00`, `onde_estamos_01` e `onde_paramos_02`; decisão 62 |
| P4-1212 *(novo)* | Incorporar as versões v0.1.4.14 a v0.1.4.35 ao changelog, decisões, erros e lições e pendências | concluída (2026-10-20) nos quatro registros e em `pendencias-tickets.md` |
| P4-1213 *(novo)* | Incorporar v0.1.4.14 a v0.1.4.35 aos documentos de assunto: README, `api-e-regras`, `filtros-e-buscas`, `validacao` e `painel-contexto` de `03-tickets-freshdesk`, `10-layout-frontend`, `09-logs-auditoria` e `00-indice/README` | concluída (2026-10-20); `01-preparacao` e `02-acesso-seguranca` não mudaram nesse intervalo, exceto novas variáveis do `.env` descritas em `api-e-regras` |

## Pendências de decisão futura (sem ID de execução ainda)
- Cabeçalho/token específico do Cloudflare Access a usar como identidade;
- tempo de expiração das sessões do Projeto4 (a do Portal está definida em 24 horas, aguardando implementação), se o logout completo do Projeto4 encerra o Portal e a política de renovação dentro das 24 horas;
- publicação de V2/V3 em outros subdomínios e portas;
- banco de dados futuro para sessões, auditoria e concorrência;
- configurações globais que poderão ser alteradas por usuários não administradores;
- proteção preventiva global: não iniciar chamada quando o crédito restante não comportar o custo da próxima operação; apresentar resultado parcial no frontend; controle centralizado entre usuários e coordenação com o Projeto3 enquanto ele consultar a mesma conta (detalhes em `03-tickets-freshdesk/pendencias-tickets.md`, TCK-0105 e TCK-0109);
- configuração do tamanho de página das buscas (hoje fixo; ficou para a aba Configurações);
- validar na conta real: `responder_id` para agente, sintaxe `tag:`/`tags:`, valor literal do nível, ID do grupo PSGE, códigos de "Aguardando o cliente"/"Aguardando terceiros" (ver `03-tickets-freshdesk/api-e-regras.md`, seção 10).
