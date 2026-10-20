# Tickets Freshdesk — Filtros, tipos de busca e buscas salvas

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

## 1. Regra de combinação

- valores dentro do mesmo campo são unidos por **OU**;
- campos diferentes são combinados por **E**.

```text
(status:2 OR status:3)
AND
(group_id:GRUPO_A OR group_id:GRUPO_B)
AND
priority:4
```

## 2. Campos do formulário

| Campo | Forma | Situação |
|---|---|---|
| Tags | texto separado por vírgula | Implementado |
| É usuário do Portal | múltipla seleção | Interface pronta, sem mapeamento Freshdesk |
| Tipo de usuário | múltipla seleção | Interface pronta, sem mapeamento Freshdesk |
| Departamento regional | múltipla seleção | Interface pronta, sem mapeamento Freshdesk |
| Unidade escolar | texto com vários valores separados por vírgula | Interface pronta, sem mapeamento Freshdesk |
| Tipo | múltipla seleção | Interface pronta, sem mapeamento Freshdesk |
| Nível | múltipla seleção | Implementado; valor literal a validar |
| Grupo | múltipla seleção | Implementado |
| Agente | múltipla seleção | Implementado (cláusula `responder_id` a confirmar) |
| Status | múltipla seleção | Implementado |
| Prioridade | múltipla seleção | Implementado |

Os cinco campos sem mapeamento não são aplicados: o backend devolve aviso em `ignoredFilters`. Se somente eles falharem, é falta de mapeamento, não regressão de aba.

## 3. Componente de múltipla seleção

Os dropdowns de checkboxes são construídos sobre `<select multiple>`, que permanece no DOM como fonte de opções e valores.

Para adicionar outro campo múltiplo:

1. incluir o ID em `MULTI_SELECT_IDS`;
2. ler com `getMultiValues(id)`;
3. escrever com `setMultiValues(id, valores)`;
4. chamar `refreshMultiSelectOptions(...)` depois de atualizar opções dinamicamente;
5. não usar `select.value` para representar múltiplas escolhas.

Seletores auxiliares em `principal.js`: `$` retorna **um** elemento; `$$` retorna uma coleção. Usar `$$` para `forEach`.

## 4. Valores especiais

| Valor | Significado | Selecionável | Salvo no JSON | Enviado à API |
|---|---|---|---|---|
| `Qualquer` | ignorar o filtro | não | não | não |
| `Nenhum` | valor nulo do campo | sim | sim | sim |
| `Não atribuído` | grupo não atribuído | sim | sim | sim |

- **Qualquer** é somente o placeholder do botão fechado do multisseletor, em cinza (`color: var(--text-faint); font-weight: 400;`). Não aparece no menu aberto e sua representação interna é `[]`. Com seleção real, o texto volta à cor normal.
- **Nenhum** no nível vira `cf_nvel_de_atendimento:null`.
- **Não atribuído** vira `group_id:null`.
- O valor `--` não tem significado de filtro: é ignorado pelo backend e removido ao salvar.

Regra do nível: com `"level": ["N1", "Nenhum"]` a consulta deve ser:

```text
(cf_nvel_de_atendimento:'N1' OR cf_nvel_de_atendimento:null)
```

## 5. Tipos de busca

Existem três. O nome da busca ativa aparece **uma única vez**, no seletor principal (botão `#searchTypeBtn`); o indicador "Busca atual" do topo e o do cabeçalho dos filtros foram removidos na v0.1.4.15 (ADD v003). As explicações dos tipos e dos prazos ficam no painel do botão `i`.

### 5.0 Seletor de buscas

*(v0.1.4.16 a v0.1.4.20; aguardando validação, salvo as cores, aceitas pelo usuário.)*

Ordem do menu do botão: Abertos Portal N1, 1ª Resposta, até três buscas adicionais na ordem do JSON e, por último, `Personalizada (não salva)`.

- até **cinco** buscas salvas visíveis; as duas de sistema contam no limite; as demais continuam disponíveis na área completa de buscas salvas;
- clicar numa busca salva do menu carrega seus filtros e atualiza o seletor da área de buscas salvas;
- o nome `Personalizada` é **reservado**, junto de `Personalizada não salva` e `Personalizada (não salva)`; a comparação ignora caixa e espaços externos; a validação existe no frontend e no backend. Mensagem: "O nome “Personalizada” é reservado pelo sistema. Escolha outro nome para a busca.";
- ao entrar em `Personalizada (não salva)` a partir de uma busca salva, os filtros são limpos **uma única vez**; novo clique, já nesse estado, não apaga os filtros preenchidos;
- visual: no botão principal, `Personalizada` em dourado, `(não salva)` em cinza e a seta dourada; no menu, item inativo na cor normal e item ativo com `Personalizada` em destaque e `(não salva)` em cinza; buscas salvas comuns em dourado.

Os parâmetros das buscas fixas são carregados nos filtros do Contexto Geral. O usuário pode preencher filtros adicionais, combinados por **E**.

### 5.1 Abertos Portal N1

`searchType` interno: `padrao_n1` (nome anterior: Padrão N1).

| Filtro | Valores |
|---|---|
| Grupo | RDS, PSGE, Portal SESI - Pedagógico, Portal SESI - Técnico/Funcionamento, Não atribuído |
| Status | Aberto, Pendente |
| Nível | N1, Nenhum |

Implementação: função compartilhada `buildGroupsClause()` e constante `GRUPOS_PADRAO_N1` (os 5 grupos), reaproveitadas pelo tipo 1ª Resposta.

### 5.2 1ª Resposta

`searchType` interno: `primeira_resposta`. Regra **validada pelo usuário na v0.1.4.13** (ver `adds-validados/Projeto4_ADD_v001_2026-09-28_busca_primeira_resposta_validada.md`); fluxo técnico atualizado nas v0.1.4.33 a v0.1.4.35 (ADD v004).

| Filtro | Valores |
|---|---|
| Grupo | os mesmos 5 grupos de Abertos Portal N1 |
| Status | Aberto e Pendente (a busca fixa preenche e envia os dois explicitamente desde a v0.1.4.31) |
| Prazo para primeira resposta | Vencidos, Hoje, Amanhã |

Ao carregar a busca salva de sistema, vêm selecionados **Vencidos** e **Hoje**. **Amanhã** aparece no campo, fica desmarcado por padrão e pode ser marcado manualmente.

#### Indicador de "já respondido"

```text
stats.first_responded_at preenchido -> ticket já recebeu a 1ª resposta -> excluir
stats.first_responded_at null       -> ainda sem 1ª resposta -> avaliar fr_due_by
```

`fr_escalated` **não é usado**: a Search API da conta o rejeita (`invalid_field`). A regra anterior da v0.1.4.2 (`fr_escalated:true/false`) está superada.

#### Classificação dos prazos

Feita pelo backend com a data e hora completas de `fr_due_by`, com os limites dos dias calculados em `America/Sao_Paulo`:

```text
Vencidos: first_responded_at == null AND fr_due_by < agora
          AND status Aberto ou Pendente
Hoje:     first_responded_at == null AND fr_due_by >= agora
          AND fr_due_by < 00h de amanhã AND status Aberto ou Pendente
Amanhã:   first_responded_at == null AND fr_due_by >= 00h de amanhã
          AND fr_due_by < 00h do dia seguinte AND status Aberto ou Pendente
```

As três classes não se sobrepõem depois da filtragem final.

#### Fluxo técnico (híbrido, v0.1.4.33 a v0.1.4.35)

Ativado por **`frDueBy` preenchido**, qualquer que seja o tipo ou nome da busca (fixa, personalizada ou salva).

```text
Search API pré-seleciona candidatos (grupos, status, fr_due_by em YYYY-MM-DD)
  -> coletar id, updated_at, status, group_id, fr_due_by
  -> consultar o cache persistente por ticket_id + updated_at
       cache hit: reutilizar stats
       cache miss: seguir
  -> updated_since = menor updated_at dos ausentes − 5 s
  -> GET /tickets?include=stats&per_page=100&page=N&order_by=updated_at&order_type=asc
  -> cruzar por ID; salvar no cache; parar quando todos forem encontrados
  -> first_responded_at preenchido: excluir
  -> sem primeira resposta: comparar fr_due_by e classificar
  -> devolver os tickets válidos e o diagnóstico
```

Operações esperadas nos logs: `search_first_response_candidates` e, havendo cache misses, `list_candidate_stats`. **Nunca** `ticket_details_stats`.

Limites e configuração (`.env`):

```text
FRESHDESK_FIRST_RESPONSE_SEARCH_MAX_PAGES=10            (Search API, 30 candidatos por página)
FRESHDESK_FIRST_RESPONSE_PER_PAGE=100
FRESHDESK_FIRST_RESPONSE_STATS_MAX_PAGES=10
FRESHDESK_FIRST_RESPONSE_UPDATED_SAFETY_SECONDS=5
FRESHDESK_FIRST_RESPONSE_CACHE_TTL_HOURS=168
FRESHDESK_FIRST_RESPONSE_CACHE_MAX_ENTRIES=2000
```

**Cache persistente** (`data/cache/freshdesk-ticket-stats.json`): mesma chave `ticket_id + updated_at` reutiliza stats; se `updated_at` mudar, a versão anterior é invalidada e substituída; sobrevive a reinício; guarda só campos técnicos mínimos (nunca descrição, conversa, e-mail, CPF, anexo, cookie, token ou API key); escrita atômica; compartilhado por qualquer busca que precise de stats.

**Cobertura.** Se o máximo de páginas for atingido com candidatos ainda ausentes, a resposta traz `complete: false`, `missingCandidateCount` e `missingCandidateIds`. Resultado incompleto nunca é apresentado como cobertura total; o aviso visual de resultado parcial no frontend ainda está pendente (TCK-0109). O resumo usa `apiUnitsConsumed`, o consumo real.

**Resultado do teste real (ADD v004):** 26 candidatos da Search API (26 Abertos), 1 página da Search API, 4 páginas com stats, 309 tickets examinados, 26 cruzados, 0 ausentes, 2 sem primeira resposta, 5 chamadas, 9 créditos, cerca de 2,6 s, sem HTTP 429. Os 26 candidatos coincidiram com a busca manual de grupos, Aberto e Pendente.

A resposta interna traz `candidateCount`, `returnedCount` e `criterion: stats.first_responded_at == null`.

#### Regras que permanecem

- a verificação de `stats.first_responded_at == null` e o fluxo híbrido continuam ativos quando a busca vira **Personalizada não salva**, desde que exista qualquer filtro de prazo (`frDueBy`);
- `responder_id` e `fr_escalated` não são critério de primeira resposta;
- não voltar à consulta individual `/tickets/{id}?include=stats` como fluxo padrão;
- não alterar o fluxo sem analisar os logs e o código completo atual;
- validações de uso continuado (cache em segunda execução e após reinício, `Amanhã` na busca personalizada, auditoria de grupos, fuso e limite entre dias): TCK-0112.

### 5.3 Personalizada

Usa somente os filtros preenchidos no formulário. Uma busca é personalizada quando é criada assim, quando um filtro de uma busca salva é alterado manualmente ou quando o usuário monta diretamente um conjunto de filtros. Estado não salvo: **Personalizada não salva**.

Se nenhum filtro for preenchido:

- o frontend avisa o volume e pede confirmação para carregar os primeiros 30 tickets (a Search API devolve 30 por página; `per_page` não é aceito);
- o backend aplica `created_at:>'2000-01-01'`;
- o backend devolve aviso em `warnings`, apresentado em `#filterFeedback`.

Abertos Portal N1 e 1ª Resposta só são aplicadas quando selecionadas; a busca vazia não aplica nenhuma delas.

## 6. Fluxo ao editar uma busca salva

```text
selecionar uma busca salva
  -> manter nome e tipo

alterar qualquer filtro
  -> currentSearchType = personalizada
  -> rótulo = Personalizada não salva
  -> limpar seleção da busca salva
  -> não reconhecer automaticamente os filtros anteriores

executar a busca
  -> enviar searchType=personalizada
  -> enviar os parâmetros atualmente selecionados
```

Mesmo que o usuário volte os filtros aos valores anteriores, a busca permanece **Personalizada não salva**. Para restaurar a busca salva, é preciso carregá-la novamente.

**Preencher os filtros por código não é edição manual.** Carregar uma busca salva preenche os filtros programaticamente; só a alteração feita pelo usuário muda o estado para Personalizada. Erro conhecido: carregar `N2-Abertos` muda para `Personalizada (usar filtros)` (TCK-0110).

## 7. Buscas salvas

O bloco de buscas salvas fica na aba **Filtros**, logo abaixo do botão Buscar.

### Operações

- listar; carregar nos filtros; salvar a busca atual; atualizar usando o mesmo nome; excluir.

### Fonte de verdade

```text
data/saved-searches.json
```

O projeto depende do JSON; o JSON não depende do projeto.

- a ordem do frontend segue a ordem do arquivo;
- apagar manualmente uma busca é definitivo (não é recriada ao reiniciar);
- o backend lê o arquivo e devolve exatamente conteúdo e ordem; não injeta, não reordena e não regrava durante a leitura;
- se o arquivo não existir, é criado apenas com `[]`;
- buscas marcadas com `system:true` no arquivo são protegidas na interface. Somente Abertos Portal N1 e 1ª Resposta são de sistema; Personalizada é busca normal;
- a edição manual do JSON continua soberana.

### Limpeza no salvamento explícito

Ao salvar pela interface são removidos `--`, `Qualquer`, valores vazios e valores duplicados.

### Formato

```json
{
  "name": "N1 urgente sem agente",
  "searchType": "personalizada",
  "filters": {
    "status": ["Aberto"],
    "priority": ["Urgente"],
    "tags": []
  },
  "createdAt": "2026-09-27T17:08:34.645Z",
  "updatedAt": "2026-09-27T17:08:34.645Z"
}
```

### Limitações atuais

- arquivo único, compartilhado por todos os usuários da instância;
- sem propriedade nem permissão por usuário;
- sem paginação;
- risco de concorrência ou sobrescrita com vários usuários simultâneos;
- o conteúdo real deve ser avaliado quanto à sensibilidade antes de ser versionado.

O Projeto4 deve suportar inicialmente até cinco usuários simultâneos, com sessões e auditoria individuais; o modelo compartilhado atual precisa ser revisto antes de ser considerado definitivo.
