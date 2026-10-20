# Legado do Projeto3

**Data:** 2026-09-29
**Status:** referência congelada (código completo recebido em 2026-09-29, `projeto3_final.zip`)

Consulta sobre o que o Projeto3 já tinha. Serve de referência quando um assunto for iniciado no Projeto4; **não** descreve o comportamento do Projeto4. Nenhum dado real (ticket, usuário, e-mail, CPF, log) foi copiado para cá — apenas estrutura, rotas e regras de código.

## Nota sobre versões do Projeto3
O Projeto4 foi iniciado a partir da base `v0.8.0.6.3` do Projeto3 V2.1 (decisão registrada em `registros/decisoes.md`, #48); as versões `v0.8.0.6.4` a `v0.8.0.6.7` são histórico de erros e rollbacks do Projeto3, não base de código. O código completo analisado neste documento (`projeto3_final.zip`) é um snapshot posterior e mais atual do Projeto3, com o `README.md` interno indicando `v0.8.0.7.0` como última versão funcional validada — reflete a evolução do Projeto3 depois do ponto em que o Projeto4 foi criado a partir dele. Ao migrar uma função específica, comparar se ela já existia no ponto do fork (`v0.8.0.6.3`) ou se foi adicionada depois, no Projeto3, sem chegar ao Projeto4.

## Identificação do projeto
- raiz: `parte3_automacao_v2.1`;
- porta: `3004`;
- última versão funcional validada: `v0.8.0.7.0`;
- patch em validação no momento da entrega: `v0.8.0.7.1.1`;
- stack: Node.js (ESM), Express, `cheerio`, `node-fetch`, `fetch-cookie`, `tough-cookie`, `dotenv`.

## Mapa de pastas
```text
parte3_automacao_v2.1/
├── src/
│   ├── app.js, server.js        (montagem do Express e bootstrap)
│   ├── auth/                    (sessão do Projeto3 e permissões do Portal)
│   ├── automacoes/               (scheduler, alertas, intervenção manual)
│   ├── config/                  (env, automações, configurações Freshdesk)
│   ├── classificacao/           (lista simples de palavras-chave)
│   ├── freshdesk/                (cliente, tickets, download, reset de senha, busca de usuário)
│   ├── frontend/                 (HTML/CSS/JS servidos)
│   ├── ia/                       (stub)
│   ├── logs/                     (log em texto + espelho do console)
│   ├── portal/                   (login/contexto real do Portal, busca de usuário)
│   ├── routes/                   (uma rota por domínio, ver abaixo)
│   ├── telegram/                 (envio de alertas)
│   └── utils/
├── data/                         (estado em runtime: última busca do Portal, settings do Freshdesk, controle de repetição de alerta)
├── freshdesk/db/, freshdesk/tickets/, freshdesk/usuario_unico/, freshdesk/attachments/, freshdesk/logs/   (cache local de tickets baixados e mineração)
├── logs/                         (log diário em texto)
├── public/audios/                (arquivos de áudio dos avisos)
├── images/icones/                (ícones usados pelo Portal)
└── .env.example
```

## Rotas HTTP (montagem em `src/app.js`)
| Prefixo | Arquivo de rota | Domínio |
|---|---|---|
| `/api` | `health.routes.js` | saúde do backend |
| `/api/auth` | `auth.routes.js` | login/logout, sessão |
| `/api/automacoes` (+ `/repeat-config`) | `automacoes.routes.js`, `alert-repeat.routes.js` | automações e controle de repetição |
| `/api/config` | `config.routes.js` | configurações do Freshdesk |
| `/api/freshdesk` (+ `/last-tickets`) | `freshdesk.routes.js`, `freshdesk-last-tickets.routes.js` | tickets, download, respostas prontas, anexos |
| `/api/logs` | `logs.routes.js` | log do dia |
| `/api/audios` | `audios.routes.js` | lista de áudios disponíveis |
| `/api/telegram` | `telegram.routes.js` | status e teste do Telegram |
| `/api/frontend` e `/` | `frontend.routes.js` | heartbeat/presença do frontend |
| `/api/portal` | `portal.routes.js` | busca de usuário no Portal |
| `/`, `/principal`, `/login` | (direto em `app.js`) | HTML servido |

Detalhe de cada rota fica nos assuntos correspondentes (`04` a `09`).

## Permissões do Portal (usadas para liberar o Projeto3)
`src/auth/permissoes.service.js` mapeia o código de perfil do Portal (ex.: `B2`, `C3`, `D4`...) para acesso: perfis A/B/C têm acesso completo (tickets, automações, Portal, configurações, escopo global); D/E têm acesso limitado ao Portal, escopo de unidade; os demais não têm acesso ao projeto. Esse esquema é específico do Portal do Projeto3 — no Projeto4 a identidade primária vem do Cloudflare Access (ver `02-acesso-seguranca`), e este mapeamento só voltaria a valer se o Projeto4 also precisar decidir o que cada perfil do Portal pode fazer depois de logado.

## Segurança observada no código (positivo, vale preservar)
- credenciais do Portal ficam em `src/portal/credentials.enc.json`, **criptografadas** (não texto plano);
- o log em texto (`text-log.service.js`) tem uma função `maskSensitive()` que mascara `FRESHDESK_API_KEY`, `Authorization: Basic`, `apiKey`, `password` e `senha` antes de gravar;
- `.env.example` não traz nenhuma credencial real, só placeholders e um `FRESHDESK_GROUP_MAP`/`FRESHDESK_AGENTS_MAP` de exemplo.

## Dados de runtime (existem no projeto, não foram copiados para a documentação)
`data/*.json`, `freshdesk/db/*.json`, `freshdesk/tickets/*.json`, `freshdesk/usuario_unico/*.json`, `freshdesk/attachments/`, `logs/*.log` — todos contêm ou podem conter dados operacionais reais (tickets, anexos, IDs de usuário). Ficam fora de qualquer documento; a regra de sanitização antes de empacotar (`P4-1201`) se aplica a eles.

## Assuntos derivados deste legado
| Assunto | Onde está o código de referência |
|---|---|
| `04-portal` | `src/portal/`, `src/auth/permissoes.service.js`, rota `/api/portal` |
| `05-automacoes` | `src/automacoes/`, `src/telegram/`, rota `/api/automacoes` |
| `06-configuracoes` | `src/config/`, rota `/api/config`, `/api/audios`, `.env.example` |
| `07-agentes-ia` | `src/ia/`, `src/classificacao/`, `src/freshdesk/resete-senha-*.service.js` |
| `08-dashboards` | `src/routes/health.routes.js`, `src/automacoes/automacoes-state.service.js`, `src/automacoes/front-presence.service.js`, `src/telegram/telegram.service.js` |
| `09-logs-auditoria` | `src/logs/` |
| `03-tickets-freshdesk` (já compilado) | `src/freshdesk/tickets.service.js`, `freshdesk.routes.js`, `freshdesk.client.js` |

## Pendências deste assunto
`P4-0001` (cópia controlada), `P4-1201` (sanitizar dados reais antes de qualquer distribuição).
