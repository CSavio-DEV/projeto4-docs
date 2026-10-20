# FLOW v0.1.4.21

```text
Usuário executa busca
  -> backend monta a query
  -> Search API recebe query + page=1
  -> endpoint aplica sua paginação padrão de até 30 resultados
  -> backend processa os tickets
  -> frontend exibe a lista
```
