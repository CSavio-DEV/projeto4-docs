# FLOW v0.1.4.23

```text
Busca 1ª Resposta
  -> uma chamada à Search API, page=1
  -> receber candidatos com status, fr_escalated e fr_due_by
  -> não abrir detalhes e não solicitar stats
  -> Vencidos: status 2/3 + fr_escalated=true
  -> Hoje: status 2/3 + fr_escalated!=true + fr_due_by no dia atual
  -> Amanhã: status 2/3 + fr_escalated!=true + fr_due_by no dia seguinte
  -> mapear e devolver resultados

Consumo esperado:
  -> 1 chamada à API por pesquisa
  -> 0 chamadas include=stats
```
