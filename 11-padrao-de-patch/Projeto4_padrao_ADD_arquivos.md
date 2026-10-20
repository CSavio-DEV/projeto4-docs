# Padrão de ADD — Projeto4

**Assunto:** Padrão de patch, versionamento e definição de pronto
**Versão do código de referência:** padrão reconstruído dos ADDs v001 a v004 (v0.1.4.13 a v0.1.4.35)
**Data:** 2026-10-20
**Status:** vigente

O documento original (`Projeto4_padrao_ADD_arquivos.md`) nunca foi localizado (P4-1207). Este padrão foi reconstruído dos quatro ADDs existentes em `adds-validados/` e das regras de `00-indice/CONVENCOES.md` (seção 8) e `registros/decisoes.md` (decisões 31, 46 e 60).

## 1. O que é um ADD

```text
UPDATE = mudanças de uma versão
FLOW   = funcionamento resultante da versão
ADD    = conclusão validada de uma tarefa
```

O ADD registra que **uma tarefa foi concluída e validada pelo usuário**. Não significa que um arquivo novo foi criado, e não é gerado a cada patch: uma tarefa pode ter vários patches antes do ADD.

```text
Tarefa -> Patch 1 -> teste -> Patch 2 -> ajuste -> validação do usuário -> ADD
```

Não geram ADD: patch não aplicado ou com falha, versão experimental descartada, ciclo ainda não validado.

## 2. Nome e local

```text
docs/adds-validados/Projeto4_ADD_vNNN_AAAA-MM-DD_descricao.md
```

`NNN` é sequencial com três dígitos e **nunca é reutilizado**; a data é a da validação; a descrição é curta, em minúsculas, separada por `_`, sem acento, terminando em `_validado` ou `_validada`.

Exemplos existentes: `Projeto4_ADD_v001_2026-09-28_busca_primeira_resposta_validada.md`, `Projeto4_ADD_v002_2026-09-29_limitador_pesquisas_api_validado.md`.

## 3. Cabeçalho

```text
# Projeto4 - ADD vNNN

**Tarefa concluída:** <nome da tarefa>
**Data de validação:** AAAA-MM-DD
**Versão final validada:** `vX.Y.Z.N`
**Versão-base:** `vX.Y.Z.N`          (ou **Versões relacionadas:** quando a tarefa teve vários patches)
**Status:** validado pelo usuário
```

O ADD evolutivo (ver seção 6) acrescenta **Tipo** e **Situação** e usa os estados documentais oficiais.

## 4. Seções

Todo ADD segue esta sequência, numerada. As seções 1, 2, 5 a 7 e 9 a 10 são obrigatórias; as demais entram quando se aplicam.

| Nº | Seção | Conteúdo | Obrigatória |
|---|---|---|---|
| 1 | Objetivo concluído | o que a tarefa resolveu e o que **não** foi alterado | sim |
| 2 | Comportamento ou regra validada | a regra em blocos de texto, com fluxos e valores | sim |
| 3 | Fluxo técnico | passos de implementação, endpoints, configuração | quando houver backend |
| 4 | Correções por versão | o que cada patch da tarefa alterou | quando houve mais de um patch |
| 5 | Arquivos de código alterados | caminhos reais | sim |
| 6 | Documentação da versão | `UPDATE`, `FLOW` e destino (`docs/_updates/`, `docs/_flows/`); pacote do patch; backup em `_backup_patch/vX.Y.Z.N/` | sim |
| 7 | Validações realizadas | o que foi testado e confirmado pelo usuário | sim |
| 8 | Limites e itens não alterados | o que a conclusão **não** cobre; pendência separada, com ID | quando houver |
| 9 | Critério de conclusão | por que a tarefa foi considerada concluída | sim |
| 10 | Onde incorporar no manual técnico | lista de arquivos de `docs/` a atualizar | sim |
| 11 | Próxima tarefa ou pendência | a etapa seguinte planejada | recomendada |

Os ADDs v001 a v004 variam de 11 a 17 seções; a sequência acima é a espinha comum.

## 5. Regras de conteúdo

- só fatos validados; o que não foi validado vai para "Limites" com ID de pendência (`P4-xxxx` ou `TCK-xxxx`);
- não confundir implementado com validado (estados em `CONVENCOES.md`, seção 9);
- nenhum token, senha, API key, cookie, e-mail real, CPF ou dado de ticket; usar placeholders;
- cada item de "Validações" é algo que o usuário viu funcionar, não só passou em verificação sintática;
- depois de criado, o ADD é incorporado ao manual: o resumo vai para `registros/changelog.md` e as regras para o README do assunto; atualizar `onde_paramos.md`.

## 6. ADD evolutivo

Quando a validação é de uma **arquitetura** e o refinamento continua (caso do ADD v004), o ADD pode ser atualizado depois. Nesse caso:

- declarar no cabeçalho `Tipo: ADD evolutivo e atualizável` e `Situação` com o estado oficial;
- listar, numa seção própria, **o que está validado** e **o que permanece sem validação continuada**;
- cada atualização futura indica a data e a versão, sem apagar o que já constava;
- ciclos não validados, como v0.1.4.16 a v0.1.4.27 isolados, não geram ADD próprio.

## 7. Modelo mínimo

```text
# Projeto4 - ADD vNNN

**Tarefa concluída:**
**Data de validação:**
**Versão final validada:**
**Versão-base:**
**Status:** validado pelo usuário

## 1. Objetivo concluído
## 2. Comportamento validado
## 3. Fluxo técnico
## 4. Correções por versão
## 5. Arquivos de código alterados
## 6. Documentação da versão
## 7. Validações realizadas
## 8. Limites e itens não alterados
## 9. Critério de conclusão
## 10. Onde incorporar no manual técnico
## 11. Próxima tarefa
```
