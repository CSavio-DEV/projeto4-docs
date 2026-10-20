# UPDATE v0.1.4.35

- O fluxo híbrido passa a ser ativado por `frDueBy`, não apenas por `searchType=primeira_resposta`.
- Buscas fixas, personalizadas e salvas com Vencidos, Hoje ou Amanhã passam a usar a mesma lógica.
- Preserva todos os filtros da query montada pela Search API.
- Impede consultas individuais `ticket_details_stats` nas buscas que exigem confirmação de primeira resposta.
- Reutiliza o cache persistente por `ticket_id + updated_at` em todos esses casos.
- Corrige o resumo final para registrar `apiUnitsConsumed` real do fluxo híbrido, removendo a estimativa incorreta.
- Não altera buscas sem `frDueBy`.
