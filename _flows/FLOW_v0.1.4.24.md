# FLOW v0.1.4.24

```text
Search API retorna candidatos
  -> status diferente de 2/3: excluir
  -> sem fr_due_by: excluir

Vencidos selecionado
  -> fr_escalated=true: incluir

Hoje/Amanhã selecionado
  -> fr_escalated=true: não classificar como futuro
  -> responder_id preenchido: excluir do teste
  -> responder_id null: comparar fr_due_by localmente
  -> data corresponde à janela: incluir

Chamadas adicionais de detalhes: 0
```
