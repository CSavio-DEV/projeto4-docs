# UPDATE v0.1.4.24

- Adiciona `responder_id == null` como filtro local experimental para Hoje e Amanhã.
- Não aplica `responder_id` aos Vencidos, que continuam usando `fr_escalated === true` e status Aberto/Pendente.
- Continua sem solicitar detalhes, `include=stats` ou `first_responded_at`.
- Mantém a comparação local do ISO completo de `fr_due_by` em `America/Sao_Paulo`.
- Mantém consumo esperado de uma única chamada à Search API.
- Registra a limitação: um ticket pode estar atribuído antes da primeira resposta; nesses casos, `responder_id` pode gerar falso negativo.
