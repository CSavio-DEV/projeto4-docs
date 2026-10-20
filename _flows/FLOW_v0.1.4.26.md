# FLOW v0.1.4.26

```text
Nova chamada Freshdesk
  -> cache idêntico válido: reutilizar
  -> chamada idêntica em andamento: compartilhar Promise
  -> caso contrário: adicionar à fila global

Governador
  -> se bloqueado por 429: recusar localmente
  -> se chamadas ativas >= concorrência: aguardar
  -> respeitar intervalo mínimo entre inícios
  -> iniciar chamada
  -> ao concluir: liberar vaga e processar próxima

MIN_INTERVAL_MS=0 + CONCURRENCY=1
  -> chamadas rápidas, porém sequenciais

MIN_INTERVAL_MS=0 + CONCURRENCY=4
  -> até quatro chamadas simultâneas
```
