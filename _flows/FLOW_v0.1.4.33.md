# FLOW v0.1.4.33

```text
Search API
  -> grupos + Aberto/Pendente + prazo
  -> coletar candidatos e menor updated_at

List Tickets com stats
  -> updated_since = menor updated_at - margem
  -> 100 por página, updated_at crescente
  -> cruzar IDs com candidatos
  -> parar quando todos forem encontrados

Resultado
  -> first_responded_at null
  -> Vencidos/Hoje/Amanhã
  -> nenhuma consulta individual
```
