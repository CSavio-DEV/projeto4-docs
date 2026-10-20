# FLOW v0.1.4.30

```text
/api/tickets recebe searchType
  -> log tickets.search.started
  -> search_tickets recebe contexto da busca
  -> cada detalhe recebe o mesmo contexto e ticketId
  -> cabeçalhos de limite são registrados
  -> sucesso: tickets.search.completed
  -> erro/429: tickets.search.failed
```
