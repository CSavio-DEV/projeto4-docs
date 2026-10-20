# FLOW v0.1.4.32

```text
1ª Resposta
  -> GET /tickets com include=stats, updated_since, per_page e page=1
  -> uma resposta já contém first_responded_at de todos os tickets da página
  -> filtrar status 2/3
  -> filtrar grupos permitidos
  -> filtrar first_responded_at null
  -> filtrar fr_due_by e prazo selecionado
  -> devolver lista
  -> zero consultas individuais de stats
```
