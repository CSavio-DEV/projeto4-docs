# UPDATE v0.1.4.25

- Restaura a confirmação exata por `stats.first_responded_at`.
- Remove `responder_id` como critério da busca.
- Cria cache global de chamadas idênticas, padrão 60 segundos.
- Deduplica requisições idênticas simultâneas.
- Cria fila global com intervalo mínimo padrão de 5 segundos.
- Em HTTP 429, bloqueia chamadas por no mínimo 60 segundos ou pelo `Retry-After` maior.
- Autorregula para intervalo de 10 segundos quando `X-RateLimit-Remaining <= 5`.
- Cria contexto de detalhes por `ticket_id + updated_at`.
- Separa cliente, governador, cache, deduplicador, estado de limite, contexto e classificador em pastas.
- Adiciona configurações correspondentes ao `.env.example`.
