# UPDATE v0.1.4.29

- Ativa a aba Logs no novo layout do Projeto4.
- Cria modos `off`, `summary` e `full`.
- Adiciona memória circular, arquivos diários, sanitização e API incremental.
- Adiciona rotas GET `/api/logs`, GET `/api/logs/status`, GET `/api/logs/today`, DELETE `/api/logs` e PUT `/api/logs/settings`.
- Instrumenta inicialmente o cliente Freshdesk para cache, deduplicação, conclusão e HTTP 429.
- Mantém a busca `1ª Resposta` sem alteração funcional.
- Remove dados sensíveis conhecidos antes de armazenar detalhes.

- Corrige o instalador para não exigir hash exato de `.env.example`; as opções de log são mescladas sem substituir conteúdo local.
