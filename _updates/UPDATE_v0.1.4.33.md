# UPDATE v0.1.4.33

- Substitui a listagem genérica da v0.1.4.32 pelo fluxo híbrido.
- Search API pré-seleciona por grupos, status e prazo.
- Usa o menor `updated_at` dos candidatos menos uma margem configurável.
- Lista tickets com `include=stats`, `order_by=updated_at` e `order_type=asc`.
- Cruza localmente os IDs e para quando todos forem encontrados.
- Registra páginas, candidatos, IDs ausentes, primeira resposta e consumo real.
- Mantém zero consultas individuais `/tickets/{id}?include=stats`.
