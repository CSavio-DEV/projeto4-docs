# UPDATE v0.1.4.23

- Remove todas as consultas individuais `GET /tickets/{id}?include=stats` da busca `1ª Resposta`.
- Remove a dependência de `stats.first_responded_at` nesse fluxo.
- Classifica Vencidos com `fr_escalated === true` e status Aberto/Pendente.
- Classifica Hoje e Amanhã comparando localmente o ISO completo de `fr_due_by` no fuso `America/Sao_Paulo`, com `fr_escalated !== true` e status Aberto/Pendente.
- Usa somente os prazos selecionados na query.
- Evita repetir a cláusula dos grupos na busca fixa `1ª Resposta`.
- Registra `detailRequests: 0` e `mode: deadline_only` no diagnóstico da resposta.
- Mantém `page=1`, seletor de buscas, nomes reservados e regras visuais anteriores.
- Limitação conhecida: sem `first_responded_at`, Hoje/Amanhã são classificados pelos indicadores de SLA presentes na resposta inicial.
