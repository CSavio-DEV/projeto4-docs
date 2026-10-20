# FLOW v0.1.4.34

```text
Search API retorna candidatos
  -> para cada ticket_id + updated_at
     -> cache hit: reutilizar stats
     -> cache miss: colocar na lista de ausentes
  -> se não houver ausentes: terminar com apenas a Search API
  -> se houver ausentes: percorrer páginas com stats
  -> salvar cada candidato encontrado no cache persistente
  -> reinício do servidor preserva os registros
```
