# FLOW v0.1.4.22

```text
Busca 1ª Resposta
  -> Search API retorna candidatos
  -> candidato 1: consultar include=stats
  -> aguardar resposta
  -> candidato 2: consultar include=stats
  -> aguardar resposta
  -> repetir até o último candidato
  -> filtrar first_responded_at preenchido
  -> classificar prazos
  -> retornar tickets válidos

Se ocorrer HTTP 429
  -> a chamada atual falha
  -> nenhuma outra chamada paralela já está em andamento
  -> backend retorna retryAfter ao frontend
```
