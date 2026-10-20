# FLOW v0.1.4.25

```text
Busca inicial
  -> chave idêntica com cache válido: reutilizar
  -> chamada idêntica em andamento: aguardar a mesma Promise
  -> sem cache: entrar na fila global
  -> respeitar intervalo mínimo
  -> consultar Freshdesk

Detalhes de primeira resposta
  -> procurar contexto ticket_id + updated_at
  -> encontrado: reutilizar
  -> ausente: fila global -> include=stats
  -> filtrar first_responded_at == null
  -> classificar pelo fr_due_by

HTTP 429
  -> bloquear novas chamadas
  -> mínimo 60 segundos ou Retry-After maior
```
