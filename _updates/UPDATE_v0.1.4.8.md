# UPDATE v0.1.4.8

## Correções

- Corrige `$(...).forEach is not a function` em `markSearchAsUnsavedCustom()`, trocando o seletor unitário `$` pelo seletor de coleção `$$`.
- Restaura a opção real `Nenhum` no filtro `Nível de Atendimento`.
- Preserva o mapeamento já existente: `N1` ou `Nenhum` gera `cf_nvel_de_atendimento:'N1' OR cf_nvel_de_atendimento:null`.
- Adiciona favicon vazio embutido para impedir a requisição automática a `/favicon.ico` e o respectivo 404.

Nenhuma regra de buscas salvas foi alterada.
