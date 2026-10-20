# UPDATE v0.1.4.34

- Cria cache persistente em `data/cache/freshdesk-ticket-stats.json`.
- Chaveia cada registro por `ticket_id + updated_at`.
- Reutiliza stats quando o candidato não mudou.
- Invalida versões anteriores do mesmo ticket quando `updated_at` mudar.
- Consulta páginas com stats somente para candidatos ausentes do cache.
- Grava apenas campos técnicos mínimos; não grava descrição, conversa, e-mail, CPF ou anexo.
- Usa escrita atômica com arquivo temporário e renomeação.
- Configura TTL padrão de 168 horas e máximo de 2000 entradas.
