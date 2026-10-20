# UPDATE v0.1.4.30

- Registra tipo e rótulo da busca em eventos de início, conclusão, cache, detalhe e HTTP 429.
- Distingue `search_tickets` de `ticket_details_stats`.
- Registra `ticketId` somente no contexto técnico já sanitizado.
- Registra `X-RateLimit-Used-CurrentRequest`.
- Gera resumo da busca com candidatos, analisados, retornados, context hits, detail requests e estimativa de unidades consumidas.
- Registra busca interrompida por rate limit.
- Não altera a lógica da busca `1ª Resposta`.
