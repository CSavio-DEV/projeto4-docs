# Padrão de patch, versionamento e definição de pronto

**Assunto:** Padrão de patch, versionamento e definição de pronto
**Versão do código de referência:** documento de padrão v1.1, 29/09/2026
**Data:** 2026-09-29
**Status:** vigente

## Escopo
Como um patch do Projeto4 é criado, nomeado, entregue, instalado, validado e versionado, além da definição de pronto de uma página migrada. Fonte principal: `Projeto4_padrao_oficial_patches.md` (v1.0).

## Princípio geral
Um patch representa uma alteração limitada, rastreável e reversível sobre uma versão conhecida do projeto: declara a versão-base, altera só os arquivos necessários, cria backup antes de substituir, valida a integridade instalada, gera `UPDATE`/`FLOW`, e falha com mensagem clara se a base local divergir da esperada.

## Formato de entrega
```text
Projeto4_patch_vX.Y.Z.N_AAAA-MM-DD_descricao.zip
```
Exemplo: `Projeto4_patch_v0.1.4.8_2026-09-27_nivel_nenhum_dirty_state_favicon.zip`.

Regras do nome: nome do projeto, palavra `patch`, versão completa, data `AAAA-MM-DD`, descrição curta em minúsculas separada por `_`, extensão `.zip`. Evitar espaço, acento na descrição, nomes genéricos (`correcao.zip`), reaproveitar número de versão para conteúdo diferente, e palavras como "final"/"novo"/"certo"/"ultima_versao". Um único ZIP oficial por versão; se estiver incorreto, gera-se a próxima versão de correção, nunca substitui o mesmo número.

## O que a mensagem de entrega deve informar
Link do ZIP, versão-base exigida, resumo da alteração, arquivos alterados, comando de instalação, necessidade de reiniciar o servidor e de `Ctrl+F5` quando houver mudança de frontend, local do backup, validações realizadas, itens não alterados e um roteiro curto de teste.

## Estrutura interna do ZIP
O nome externo do arquivo continua completo e descritivo:

```text
Projeto4_patch_vX.Y.Z.N_AAAA-MM-DD_descricao.zip
```

Dentro do ZIP deve existir uma única pasta de primeiro nível, nomeada somente com a versão:

```text
Projeto4_patch_vX.Y.Z.N_AAAA-MM-DD_descricao.zip
└── vX.Y.Z.N/
    ├── apply_patch_vX.Y.Z.N.mjs
    ├── manifest.json
    ├── README_PATCH.md
    ├── docs/
    │   ├── UPDATE_vX.Y.Z.N.md
    │   └── FLOW_vX.Y.Z.N.md
    ├── files/
    │   └── arquivos completos a instalar, no caminho de destino real
    └── tools/          (somente quando necessário)
```

Exemplo:

```text
Projeto4_patch_v0.1.4.14_2026-09-29_limite_pesquisas_api.zip
└── v0.1.4.14/
    ├── apply_patch_v0.1.4.14.mjs
    ├── manifest.json
    ├── README_PATCH.md
    ├── docs/
    │   ├── UPDATE_v0.1.4.14.md
    │   └── FLOW_v0.1.4.14.md
    ├── files/
    └── tools/
```

Não repetir o nome completo do ZIP como pasta interna. A pasta curta `vX.Y.Z.N/` facilita a extração em `_patches/` e a execução do aplicador.

### Destino permanente de UPDATE e FLOW
Os arquivos em `vX.Y.Z.N/docs/` são o payload documental do patch. Durante a instalação, o aplicador deve copiá-los para:

```text
vX.Y.Z.N/docs/UPDATE_vX.Y.Z.N.md
  -> parte4_cloudflare/docs/_updates/UPDATE_vX.Y.Z.N.md

vX.Y.Z.N/docs/FLOW_vX.Y.Z.N.md
  -> parte4_cloudflare/docs/_flows/FLOW_vX.Y.Z.N.md
```

Estrutura permanente:

```text
parte4_cloudflare/
└── docs/
    ├── _updates/
    │   └── UPDATE_vX.Y.Z.N.md
    ├── _flows/
    │   └── FLOW_vX.Y.Z.N.md
    └── adds-validados/
        └── Projeto4_ADD_vNNN_AAAA-MM-DD_descricao_validada.md
```

Regras:

1. `_updates/` guarda um UPDATE por versão aplicada.
2. `_flows/` guarda um FLOW por versão aplicada.
3. O aplicador cria as duas pastas quando ainda não existirem.
4. Se já existir um documento com o mesmo nome e conteúdo diferente, o aplicador cria backup antes de substituir.
5. Reexecução idempotente não duplica documentos.
6. UPDATE e FLOW não ficam espalhados na raiz nem nas pastas de assunto.
7. ADD continua separado e só é criado depois da validação da tarefa.

## Nunca entram num patch comum
`.env`, credenciais reais, API Keys, cookies, sessões, logs reais, `data/saved-searches.json` com dados do usuário, `node_modules/`, backups locais, arquivos temporários. Se um arquivo de dados precisar ser migrado, o patch altera o arquivo já existente no projeto, com backup antes — nunca substitui silenciosamente dado real por exemplo.

## Instalação
Depois da extração, a estrutura deve ficar:

```text
parte4_cloudflare/
└── _patches/
    └── vX.Y.Z.N/
        ├── apply_patch_vX.Y.Z.N.mjs
        ├── manifest.json
        ├── README_PATCH.md
        ├── docs/
        ├── files/
        └── tools/
```

Procedimento:

```text
parar o servidor
  -> extrair a pasta vX.Y.Z.N em _patches/
  -> executar node _patches/vX.Y.Z.N/apply_patch_vX.Y.Z.N.mjs
  -> conferir a saída
  -> npm start
  -> Ctrl+F5 se houve mudança de frontend
  -> executar os testes recomendados
```

O aplicador também copia:

```text
UPDATE -> docs/_updates/
FLOW   -> docs/_flows/
```

Backups sempre em `_backup_patch/vX.Y.Z.N/<caminhos originais>`, nunca soltos na raiz.

## Versionamento
Formato `vMAJOR.MINOR.FEATURE.PATCH` (`vX.Y.Z.N`):
- 1º número: mudança estrutural ampla ou nova geração (`v1.0.0.0`);
- 2º número: novo ciclo funcional relevante (`v0.2.0.0`);
- 3º número: conjunto funcional ou etapa maior do ciclo (`v0.1.4.0`);
- 4º número: correções e evoluções incrementais da mesma etapa (`v0.1.4.1`, `v0.1.4.2`...).

Regras: qualquer conteúdo diferente exige nova versão; não substituir ZIP já entregue mantendo o número; correção de patch entregue usa o próximo número; alteração manual deve ser registrada e incorporada à próxima base; versão com erro/rollback não vira base validada; a documentação declara a base real usada. A **versão lógica** é o último patch aplicado; havendo correção manual depois, o estado real é "versão lógica + correção local registrada", e essa correção deve ser coletada antes do próximo patch.

## Tipos de patch
- **Substituição integral** — arquivos completos conhecidos e controlados; resultado previsível, hash verificável.
- **Transformação controlada** (`tools/`) — quando o arquivo tem correção manual válida, substituir o arquivo inteiro apagaria trabalho local, ou é preciso migrar um JSON existente. O script deve procurar uma estrutura conhecida, falhar se não achar, ser idempotente quando possível e validar o resultado.
- **Registro** — registra uma base atual recebida (código + documentação consolidada + UPDATE/FLOW), sem necessariamente mudança funcional grande.
- **Dados** — migra dados existentes; backup obrigatório, nunca apaga dado personalizado silenciosamente, nunca recria item removido manualmente sem regra explícita, declara a fonte de verdade, testa com cópia do arquivo real.

## Validações obrigatórias antes da entrega
ZIP íntegro (`unzip -t`), sintaxe JavaScript válida (`node --check`) no aplicador, nos scripts de `tools/`, no JS e no servidor alterados, hashes do `manifest.json` conferidos, estrutura completa e sem credenciais nem arquivo temporário, pelo menos um teste funcional direcionado ao problema tratado, e uma verificação secundária extraindo o ZIP numa pasta temporária e simulando a instalação.

## UPDATE, FLOW e ADD

```text
UPDATE = mudanças de uma versão
FLOW   = funcionamento resultante da versão
ADD    = conclusão validada de uma tarefa
```

O ADD **não é gerado a cada patch**. Uma tarefa pode exigir vários patches antes de seu ADD:

```text
Tarefa -> Patch 1 -> teste -> Patch 2 -> ajuste -> validação do usuário -> ADD da tarefa concluída
```

O ADD só é criado quando o resultado estiver concluído e validado pelo usuário. Ele registra a conclusão da tarefa e não significa necessariamente que um arquivo novo foi criado. Os ADDs validados ficam em `adds-validados/`; o modelo concreto é o `Projeto4_ADD_v001_2026-09-28_busca_primeira_resposta_validada.md` (objetivo, regra validada, fluxo técnico, correções por versão, arquivos relacionados, validações, limitações e pendência separada, critério de conclusão e onde incorporar no manual).

O padrão de ADD foi reconstruído a partir dos ADDs v001 a v004 e está em [Projeto4_padrao_ADD_arquivos.md](Projeto4_padrao_ADD_arquivos.md) (pendência `P4-1207`, concluída em 2026-10-20). O documento original nunca foi localizado. A pasta `_a-incorporar/` de cada assunto guarda o conteúdo novo ainda não compilado e não substitui UPDATE, FLOW nem ADD.

## Alterações do padrão v1.1, 29/09/2026

- criadas as pastas permanentes `docs/_updates/` e `docs/_flows/`;
- definido que o aplicador copia automaticamente UPDATE e FLOW para essas pastas;
- confirmado que o ZIP tem nome externo completo, mas contém internamente apenas `vX.Y.Z.N/`;
- confirmado o comando de instalação a partir de `_patches/vX.Y.Z.N/`;
- mantida a separação entre UPDATE/FLOW por versão e ADD por tarefa validada.

## Regra final
Um patch não é só um conjunto de arquivos: é código (ou transformação) + versão + validação da base + backup + instalação + integridade + documentação + teste + possibilidade de continuidade.

## Definição de pronto de uma página migrada
Uma página só é considerada migrada quando:
1. visualmente fiel ao novo layout;
2. usa o shell consolidado;
3. tem contrato de API documentado;
4. preserva as funções validadas do Projeto3;
5. respeita sessão e permissões individuais;
6. trata carregamento, vazio, erro e sessão expirada;
7. não exibe JSON bruto como substituto do componente visual;
8. possui teste mínimo;
9. foi validada no desktop;
10. tem adaptação mobile planejada ou concluída;
11. tem `UPDATE` e `FLOW` correspondentes;
12. não expõe credenciais ou dados operacionais.

## Checklist de criação de um patch
**Antes de editar:** identificar a versão-base real; coletar os arquivos atuais; verificar correções manuais posteriores; delimitar o escopo; listar arquivos que não podem ser alterados.
**Durante:** usar o próximo número de versão; criar `manifest.json`, aplicador, `UPDATE`, `FLOW`, `README`; configurar backup em `_backup_patch/versão/`; preservar `.env` e dados reais.
**Antes de entregar:** validar ZIP, sintaxe e hashes; testar instalação e o comportamento alterado; verificar ausência de credenciais; confirmar nome do ZIP; informar base e comando de instalação.
**Depois da validação do usuário:** registrar a versão como validada; atualizar a documentação consolidada; registrar regressões encontradas; gerar o ADD da tarefa só quando ela estiver concluída e validada.

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-1203`–`P4-1206` e `P4-1208` (lacunas de UPDATE/FLOW). `P4-1207` (padrão de ADD) está concluída.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
