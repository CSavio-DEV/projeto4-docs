# UPDATE v0.1.4.22

- Altera a busca `1ª Resposta` de quatro consultas simultâneas de detalhes para uma consulta por vez.
- Cada chamada `GET /tickets/{id}?include=stats` aguarda a resposta da anterior.
- Mantém a Search API, `stats.first_responded_at`, prazos, filtros e resultados sem outras mudanças.
- Não adiciona cache nesta versão para permitir comparar somente o efeito da retirada do paralelismo.
- Mantém o tratamento atual de HTTP 429 e `retryAfter`.
- Registra como hipóteses: aumento da quantidade de candidatos e regressão causada pelo paralelismo.
