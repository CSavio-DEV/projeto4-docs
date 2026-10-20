# UPDATE v0.1.4.27

- Corrige a rejeição não tratada criada por `promise.finally()` no deduplicador de requisições.
- Substitui a limpeza por `promise.then(cleanup, cleanup)`, cujo ramo derivado não permanece rejeitado.
- Preserva a rejeição da Promise original para o fluxo normal de tratamento no servidor.
- Mantém cache, deduplicação, concorrência configurável, fila global e bloqueio por HTTP 429.
- O HTTP 429 passa a ser uma resposta controlada e não encerra mais o processo Node.js.
