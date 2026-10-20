# UPDATE v0.1.4.2

- Renomeia Padrão N1 para Abertos Portal N1, preservando o ID interno `padrao_n1`.
- Preenche Contexto Geral > Filtros ao selecionar as duas buscas fixas.
- Acrescenta o filtro Prazo para primeira resposta: Vencidos e Hoje.
- Acrescenta Não atribuído ao filtro manual de Grupo.
- Interpreta Nível Nenhum como campo nulo.
- Corrige 1ª Resposta: vencidos usam `fr_escalated:true AND status:2`; hoje usa a janela do dia em `America/Sao_Paulo`, convertida para UTC, com `fr_escalated:false`.
- A seleção não executa a busca automaticamente.
