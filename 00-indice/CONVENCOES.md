# Convenções da documentação

**Data:** 2026-09-29
**Status:** vigente (estados documentais, UPDATE/FLOW/ADD e `adds-validados/` incorporados em 2026-09-29)

## 1. Estrutura

- Uma pasta por assunto. Cada pasta tem um `README.md` com o estado atual e uma subpasta `_a-incorporar/`.
- `registros/` guarda o que cruza assuntos (changelog, erros e lições, decisões, pendências). Cada entrada leva a tag do assunto.
- `legado-projeto3/` é histórico congelado, só para consulta.
- `adds-validados/` guarda os ADDs: um por tarefa concluída e validada pelo usuário (ver seção 8).
- `onde_paramos.md`, na raiz de `docs/`, é o ponto de entrada de quem retoma o trabalho: versão validada, próximas tarefas e regras permanentes. Atualizar sempre que mudar a versão validada, a próxima tarefa ou uma decisão importante.

## 2. Cabeçalho padrão

Todo documento começa com:

```text
Assunto:
Versão do código de referência:
Data:
Status: vigente | rascunho | superado
```

Documento `superado` não fica ao lado do vigente: é descartado ou movido para `_arquivo/` na raiz da documentação.

## 3. Precedência em caso de conflito

```text
1. código da versão
2. entradas em _a-incorporar/ mais recentes que o estado atual do assunto
3. README.md do assunto (estado atual)
4. registros/ (changelog, decisões, erros e lições)
5. legado-projeto3/
```

## 4. Pasta `_a-incorporar/`

Guarda etapas novas que já aconteceram, mas ainda não foram compiladas no README do assunto.

- Nome do arquivo: `AAAA-MM-DD_vX.Y.Z.W_descricao-curta.md`, usando o modelo `modelos/modelo-add.md`.
- Ao incorporar no README do assunto, o resumo vai para `registros/changelog` e o arquivo sai da pasta. A mesma informação não fica em dois lugares.
- Pasta vazia significa que o assunto está totalmente consolidado.
- `registros/` e `legado-projeto3/` não têm `_a-incorporar/`: o primeiro já é acumulativo e o segundo é congelado.

## 5. Conteúdo proibido

Nenhum documento contém tokens, senhas, API keys, PINs, IDs de conta, dados de cartão ou e-mails reais de usuários. Usar placeholders. A política sobre credenciais fica em `02-acesso-seguranca`.

## 6. O que não entra nos documentos de referência

Comentários sobre o próprio documento ("registro antigo superado", "tarefas repetidas", "histórico desta consolidação"). Isso vive no changelog ou no controle de versões, não no texto técnico.

## 7. Pendências

Cada pendência tem um ID único, no padrão já existente `P4-xxxx`, e a tag do assunto. Existe um único lugar para elas: `registros/pendencias`.

## 8. UPDATE, FLOW e ADD

```text
UPDATE = mudanças de uma versão
FLOW   = funcionamento resultante da versão
ADD    = conclusão validada de uma tarefa
```

O ADD só é criado depois da validação do usuário; uma tarefa pode ter vários patches antes dele. Nome: `Projeto4_ADD_vNNN_AAAA-MM-DD_descricao.md`, na pasta `adds-validados/`. Ao ser incorporado, o resumo vai para `registros/changelog`.

## 9. Estados documentais

```text
Planejado
Pendente
Em implementação
Parcialmente implementado
Implementado, aguardando validação
Parcialmente validado
Validado
Superado por versão posterior
Descartado como base de código
Legado funcional do Projeto3
Regressão conhecida
```

- uma função visível no frontend não é funcional sem conexão e teste do backend;
- uma alteração implementada não é validada apenas por passar em verificação sintática;
- itens do Projeto3 são separados entre legado validado, pendência histórica e requisito de migração;
- itens já resolvidos não continuam como pendências genéricas.
