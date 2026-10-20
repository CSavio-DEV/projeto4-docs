# UPDATE v0.1.4.16

- Torna dinamico o menu do botao seletor de buscas.
- Exibe ate cinco buscas salvas, respeitando a ordem retornada pelo backend e registrada no JSON.
- O limite inclui as duas buscas de sistema; portanto, podem aparecer ate tres buscas salvas adicionais.
- Mantem `Personalizada nao salva` sempre como ultimo item.
- Ao clicar numa busca salva do menu, carrega seus filtros e atualiza o seletor de buscas salvas.
- Preserva o ajuste visual manual da opcao 2 no CSS; este patch nao substitui o CSS.
