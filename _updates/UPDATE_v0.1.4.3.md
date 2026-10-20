# UPDATE v0.1.4.3

- Registra no mesmo `data/saved-searches.json` as buscas fixas Abertos Portal N1, 1ª Resposta e Personalizada.
- Mantém essa ordem no início da lista; buscas criadas pelo usuário aparecem depois, por atualização mais recente.
- Remove `--`, `Qualquer`, valores vazios e duplicados antes de salvar.
- Troca a opção visual `---` por `Qualquer`, desabilitada e apenas informativa.
- Preserva as opções reais `Nenhum` e `Não atribuído`.
- Protege as três buscas fixas contra sobrescrita e exclusão.
- Migra o JSON existente sem apagar buscas personalizadas.
