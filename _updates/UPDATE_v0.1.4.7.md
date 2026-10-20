# UPDATE v0.1.4.7

## Fluxo corrigido

1. o usuário carrega uma busca salva;
2. a busca permanece identificada enquanto nenhum filtro for alterado;
3. a primeira alteração manual transforma o estado em `Personalizada não salva`;
4. a busca salva é desmarcada;
5. mesmo que os filtros voltem aos valores anteriores, o estado continua personalizado não salvo;
6. a próxima requisição usa `searchType=personalizada` e envia apenas os filtros atualmente selecionados.

## Erros do backend

O frontend passa a mostrar a mensagem e os detalhes reais retornados em respostas HTTP 400, em vez de informar apenas indisponibilidade do backend.
