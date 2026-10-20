# FLOW v0.1.4.27

```text
Freshdesk responde HTTP 429
  -> freshdesk-client cria erro controlado
  -> Promise original rejeita
  -> deduplicador remove a entrada em andamento
  -> servidor captura o erro e responde HTTP 429 ao frontend
  -> processo Node.js continua ativo
  -> governador bloqueia novas chamadas pelo período configurado
```
