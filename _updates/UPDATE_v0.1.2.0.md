# UPDATE v0.1.2.0

- Cria `package.json` próprio do Projeto4, sem copiar módulos do Projeto3.
- Cria servidor HTTP Node sem dependências externas.
- A raiz `/` entrega `src/frontend/principal.html`.
- `/login` entrega `login.html`.
- `/health` e `/api/health` informam a saúde do servidor.
- Cria aviso independente `servidor-desligado.html`.
- Cria `abrir_projeto4.cmd`, que tenta iniciar o servidor e abre o aviso local se falhar.
