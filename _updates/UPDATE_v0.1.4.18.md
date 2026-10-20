# UPDATE v0.1.4.18

- Corrige a v0.1.4.17 usando diretamente os arquivos atuais da v0.1.4.16.
- Reserva `Personalizada`, `Personalizada não salva` e `Personalizada (não salva)` no frontend e backend.
- Ao entrar em `Personalizada (não salva)` a partir de uma busca salva, limpa todos os filtros uma única vez.
- Novo clique enquanto já está nesse estado não apaga filtros preenchidos.
- Antes de pesquisar sem filtros, avisa sobre o volume e pede confirmação para carregar os primeiros 30 tickets.
- Define `page=1` e `per_page=30` na Search API.
- Destaca `Personalizada` em dourado e `(não salva)` em cinza.
- Preserva o estilo dourado validado do botão principal e todas as funções da v0.1.4.16.
- A configuração futura do tamanho da página permanece pendente para a aba Configurações.
