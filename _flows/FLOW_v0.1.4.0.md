# FLOW v0.1.4.0

```text
GET /
  -> principal.html

Tickets Freshdesk
  -> tipo de busca
  -> filtros múltiplos
  -> GET /api/tickets
  -> freshdesk.buildSearchQuery()
  -> Search API Freshdesk
  -> lista normalizada

Buscas salvas
  -> GET/POST/DELETE /api/saved-searches
  -> savedSearches.js
  -> data/saved-searches.json (criado em runtime, fora do patch)

Configuração de filtros
  -> GET /api/config/filters
  -> mapas de grupos e agentes do .env

Segurança
  -> FRESHDESK_API_KEY apenas no .env
  -> frontend não recebe nem exibe a chave
```
