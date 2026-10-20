# UPDATE v0.1.4.32

- Busca `1ª Resposta` passa a usar `/api/v2/tickets?include=stats&updated_since=...&per_page=100&page=1`.
- Remove consultas individuais `/tickets/{id}?include=stats` dessa busca fixa.
- Filtra localmente grupos, status Aberto/Pendente, `stats.first_responded_at == null`, `fr_due_by` e prazos selecionados.
- Registra quantidade listada, candidatos por status e grupo, quantidade sem primeira resposta e quantidade final.
- Registra tickets com `created_at == updated_at` como diagnóstico; eles não exigem chamada adicional porque `stats` já veio na listagem.
- Mantém Search API e o fluxo anterior para outras buscas.
- Intervalo inicial configurável em 7 dias e até 100 tickets na primeira página.
- `complete` será falso quando a página vier cheia, indicando que poderá existir uma página seguinte.
