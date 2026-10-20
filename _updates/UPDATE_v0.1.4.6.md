# UPDATE v0.1.4.6

- Remove a injeção automática de Abertos Portal N1, 1ª Resposta e Personalizada no JSON.
- Remove a reordenação e regravação automática durante a leitura.
- A API devolve exatamente as buscas e a ordem existentes no arquivo.
- Se uma busca for apagada manualmente do JSON, ela não volta ao reiniciar.
- Um arquivo inexistente é criado somente como `[]`.
- O salvamento pela interface ainda limpa `--`, `Qualquer`, vazios e duplicados.
- Buscas marcadas no próprio JSON com `system:true` continuam protegidas contra alteração pela interface, mas podem ser removidas pela edição manual do arquivo.
