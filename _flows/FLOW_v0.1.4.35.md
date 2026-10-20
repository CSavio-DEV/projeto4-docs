# FLOW v0.1.4.35

```text
/api/tickets
  -> ler frDueBy

frDueBy vazio
  -> fluxo normal da busca

frDueBy possui Vencidos, Hoje ou Amanhã
  -> Search API com todos os filtros atuais
  -> candidatos por ID + updated_at
  -> cache persistente
  -> páginas com stats somente para cache misses
  -> cruzamento por ID
  -> first_responded_at null
  -> nenhuma consulta individual por ticket
```
