# FLOW v0.1.4.2

```text
Selecionar Abertos Portal N1
  -> grupos fixos + Não atribuído
  -> status Aberto/Pendente
  -> nível N1/Nenhum
  -> Contexto Geral > Filtros

Selecionar 1ª Resposta
  -> grupos fixos + Não atribuído
  -> prazo Vencidos/Hoje
  -> Vencidos: fr_escalated:true AND status:2
  -> Hoje: limites do dia no Brasil convertidos para UTC + fr_escalated:false
  -> Contexto Geral > Filtros
```
