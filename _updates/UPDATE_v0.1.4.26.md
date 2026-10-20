# UPDATE v0.1.4.26

- Adiciona `FRESHDESK_REQUEST_CONCURRENCY`, com valor padrão 1 e faixa de 1 a 10.
- Substitui a fila estritamente serial por um governador com fila pendente e limite configurável de chamadas ativas.
- Mantém o intervalo mínimo configurado entre inícios de chamadas, inclusive quando a concorrência for maior que 1.
- Com `FRESHDESK_REQUEST_MIN_INTERVAL_MS=0`, até o número configurado de chamadas pode executar simultaneamente.
- Mantém cache de 60 segundos, deduplicação de chamadas idênticas e contexto por ticket mais `updated_at`.
- Mantém bloqueio mínimo de 60 segundos depois de HTTP 429 e respeito ao `Retry-After` maior.
- O instalador adiciona a nova variável ao `.env` sem substituir as configurações existentes.
- `.env.example` passa a ser opcional: se não existir, será criado; se existir, será atualizado.
