# UPDATE v0.1.4.14

- Impede mais de uma pesquisa simultanea no frontend.
- Aplica intervalo minimo de 5 segundos depois de uma pesquisa concluida com sucesso.
- Trata HTTP 429 com bloqueio minimo de 10 segundos.
- Respeita `retryAfter` retornado pelo backend quando for maior que 10 segundos.
- Mostra contagem regressiva nos botoes `Carregar tickets` e `Buscar`.
- Mantem a mensagem curta no feedback e em toast.
- Nao altera filtros, consultas Freshdesk, stats, busca salva ou layout estrutural.
