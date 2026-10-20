# Configurações

**Assunto:** Configurações
**Versão do código de referência:** Projeto3 v0.8.0.7.0 (ainda não migrado ao Projeto4)
**Data:** 2026-09-29
**Status:** reservado no Projeto4 — referência do Projeto3 completa

## Escopo
Mapa entre controles da tela e configurações reais (`.env`, arquivos de dados), Freshdesk, downloader, áudios, Telegram e proteção de segredos.

## Como funciona no Projeto3

### Configurações do Freshdesk (editáveis pela interface)
`GET`/`POST /api/config/freshdesk` (`freshdesk-settings.service.js`), persistidas em `data/freshdesk.settings.json`, com validação e limites em cada campo:

| Grupo | Campos | Limite |
|---|---|---|
| `tickets` | status, grupos, níveis exibidos por padrão | só valores de uma lista fechada (`STATUS_OPTIONS`, `GROUP_OPTIONS`, `NIVEL_OPTIONS`) |
| `api` | tempo de disparo do scheduler (s), paginação, tempo limite de retorno (s) | 5–3600s, 1–100 itens, 3–120s |
| `sla` | prazo de 1ª resposta e de resolução por prioridade, expediente (dias, manhã/tarde, feriados) | 0,25–240h (1ª resposta), 0,25–720h (resolução) |

As listas fechadas do Projeto3 já estão desatualizadas frente aos grupos reais usados em `03-tickets-freshdesk` (ali são 5 grupos + Não atribuído, aqui só 3 + Não atribuído) — ajustar ao migrar.

### Variáveis de ambiente (não editáveis pela interface)
Confirmadas em `.env.example` do Projeto3 — a lista completa de nomes de variável (sem valores) é a referência para a tela "Configurações avançadas" do Projeto4, mantendo os mesmos princípios de proteção de `02-acesso-seguranca`:

`PORT`, `NODE_ENV`, `AUTH_MOCK_ENABLED`, `AUTH_SECRET`, `FRESHDESK_DOMAIN`, `FRESHDESK_API_KEY`, `FRESHDESK_OCULTAR_TICKETS_BAIXADOS`, `FRESHDESK_GROUP_MAP`, `FRESHDESK_AGENTS_MAP`, `FRESHDESK_NIVEL_FIELD`, `PORTAL_DOMAIN_PRINCIPAL`, `PORTAL_DOMAIN_PAINEL`, `RESET_SENHA_ENABLED`, `RESET_SENHA_MAX`, `RESET_SENHA_TIMEOUT_MS`, `RESET_SENHA_DELAY_MS`, `PORTAL_AVALIADOR_ID`, `DOMINIOX_ADMIN_ID`, `BUSCA_USUARIO_PORTAL_ATIVA`, `BUSCA_USUARIO_PORTAL_LOG_NIVEL`, `BUSCA_USUARIO_PORTAL_LIMITE_USUARIOS`, `LOG_MODO`, `AUTOMACOES_MODO_SIMULACAO`, `AUTOMACOES_INTERVALO_MIN_MS`, `AUTOMACOES_INTERVALO_MAX_MS`, `TELEGRAM_BOT_NAME`, `TELEGRAM_BOT_USERNAME`, `TELEGRAM_CHANNEL_ID`, `TELEGRAM_CHAT_ID`, `TELEGRAM_CHAT_ID_PRIVATE_NO_TICKETS`, `TELEGRAM_CHAT_ID_PRIVATE_ALERTS`, `TELEGRAM_BOT_TOKEN`, `ALERTAS_PAUSA_INTERVENCAO_MANUAL`, `ALERTAS_RETORNO_APOS_INTERVENCAO_MS`, `INTERVENCAO_MANUAL_HEARTBEAT_MAX_AGE_MS`.

### Downloader do Freshdesk
`freshdesk-download-db.service.js` tem configuração própria (`getFreshdeskDownloadConfig`/`saveFreshdeskDownloadConfig`, rota `/api/freshdesk/download-config`), independente das configurações gerais acima — controla o download automático de tickets para treino/mineração (ver `07-agentes-ia`). **Decisão já tomada para o Projeto4** (`decisoes.md` #27): a tela de acionamento desse downloader fica fora do Projeto4; só os dados já minerados são usados.

### Áudios
`GET /api/audios` (`audios.routes.js`) lista os arquivos `.mp3`/`.wav` existentes em `public/audios/` (hoje: `audio_alerta01.mp3`, `audio_success.mp3`) e devolve nome + URL de cada um. **Não há endpoint de upload nem de exclusão** — a lista é somente leitura, montada dinamicamente a partir da pasta. Isso é exatamente o ponto da seção "Áudios dos avisos" que ficou por especificar (ver pendência `P4-0905` em `registros/pendencias.md`): decidir se o Projeto4 mantém só-leitura de pasta, adiciona upload, e como tratar a reprodução no navegador mobile.

### Telegram
Configuração vem só do `.env` (não editável pela tela); `GET /api/telegram/status` devolve nomes/booleanos de "configurado" sem nunca expor o token; `POST /api/telegram/testar` dispara um envio de teste.

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0901` a `P4-0905`, e a decisão pendente sobre upload de áudio.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
