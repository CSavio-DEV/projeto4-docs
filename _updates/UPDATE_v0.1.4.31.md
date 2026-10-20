# UPDATE v0.1.4.31

- Altera somente `src/frontend/js/principal.js`.
- A busca fixa `1ª Resposta` passa a preencher e enviar explicitamente `Aberto` e `Pendente`.
- Preserva grupos fixos e os prazos `Vencidos` e `Hoje`.
- Não altera o backend, o cache, a concorrência, os logs ou a validação por `stats.first_responded_at`.
- Objetivo do teste: verificar se a quantidade inicial de candidatos diminui.
