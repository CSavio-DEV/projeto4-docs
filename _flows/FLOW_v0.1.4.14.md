# FLOW v0.1.4.14

```text
Usuario inicia pesquisa
  -> bloquear os dois botoes enquanto a chamada esta ativa
  -> sucesso
      -> bloquear novas pesquisas por 5 segundos
      -> mostrar contagem nos botoes
      -> liberar automaticamente
  -> HTTP 429
      -> ler retryAfter
      -> bloquear por no minimo 10 segundos
      -> usar retryAfter se for maior
      -> mostrar aviso curto e contagem
      -> liberar automaticamente
```
