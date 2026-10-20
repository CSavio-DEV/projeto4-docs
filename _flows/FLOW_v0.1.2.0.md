# FLOW v0.1.2.0

```text
abrir_projeto4.cmd
  -> consulta /health
  -> se ligado: abre http://127.0.0.1:65004/
  -> se desligado: tenta node src/server.js
       -> sucesso: abre principal.html pela porta
       -> falha: abre servidor-desligado.html local

GET / -> src/frontend/principal.html
GET /login -> src/frontend/login.html
GET /health -> JSON de saúde
```

Observação: um servidor desligado não consegue entregar sua própria página de erro. Por isso o aviso é aberto localmente pelo launcher.
