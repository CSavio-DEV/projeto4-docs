# Tickets Freshdesk — Painel de contexto e interface da aba

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

## 1. Estado inicial da aba

A lista fica visível desde a primeira abertura. Existe um único botão **Carregar tickets**, e a área de resultados mostra apenas o ícone e **Nenhum ticket carregado**. Não há tela inicial separada nem botão duplicado. (Validado pelo usuário na v0.1.4.1.)

## 2. Contexto geral

Três abas.

### Filtros

Formulário completo de busca (Tags, É usuário do Portal, Tipo de usuário, Departamento regional, Unidade escolar, Tipo, Nível, Grupo, Agente, Status, Prioridade), botão Buscar e, abaixo dele, o bloco **Buscas salvas**. A aba Filtros existe apenas no Contexto geral. Detalhes dos filtros em `filtros-e-buscas.md`.

### Propriedades

Resumo da última busca: quantidade de tickets carregados, quantidade de abertos, grupo, nível e prioridade mais frequentes e última sincronização.

### Automações

Preservada sem alteração funcional.

## 3. Contexto do ticket

Com um ticket aberto, o painel mantém apenas **Propriedades** e **Automações**. A aba Filtros não aparece.

## 4. Controle de abas

O controle considera simultaneamente:

- contexto ativo: `geral` ou `ticket`;
- aba ativa dentro do contexto;
- `data-context` e `data-panel` do painel;
- `data-context-only` do botão, quando existir.

```javascript
let activeContext = 'geral';
const activeTabByContext = {
  geral: 'filtros',
  ticket: 'propriedades'
};
```

Cada contexto preserva a própria aba selecionada.

### Regra para novas abas

Cada botão precisa de um painel correspondente:

```html
<button class="tab-btn" data-tab="X">
<div class="tab-panel" data-context="geral" data-panel="X">
```

Para limitar uma aba a um contexto, usar no botão `data-context-only="geral"` ou `data-context-only="ticket"`.

O atributo do painel é `data-panel` (não `data-tab-panel`): comparar com `dataset.tabPanel` deixa o valor indefinido e faz qualquer clique ocultar todos os painéis.

## 5. Elementos de interface relevantes


| Elemento | Função |
|---|---|
| `#filterFeedback` | mensagens e avisos de filtro/busca (inclui `warnings` do backend) |
| `#searchTypeBtn` | seletor de buscas; o nome da busca ativa aparece só aqui (o indicador "Busca atual" foi removido na v0.1.4.15) |
| Botão `i` e seu painel | explicações e informações de uso; ao lado do título Tickets Freshdesk (ver seção 6) |
| `ticketListMeta` | mensagem de carregamento; visível apenas durante a busca |
| Prefixo `tf-` | IDs dos campos do formulário de Tickets (evita colisão com `f-*` do Portal) |
| Favicon vazio embutido no `<head>` | evita `GET /favicon.ico` 404 |

## 6. Painel de informações (botão `i`)

*(v0.1.4.15, validado, ADD v003.)* Botão circular `i` ao lado do título **Tickets Freshdesk**. Abre um painel compacto com: funcionamento geral da busca, significado de Vencidos, Hoje e Amanhã, intervalo entre pesquisas, limite da API e abertura de um ticket da lista.

- fecha por `×`, por clique fora do painel e por `Esc`; tem comportamento adaptado a telas menores;
- as explicações permanentes saíram do cabeçalho e da linha de ações; o warning técnico sobre `fr_due_by` e `America/Sao_Paulo` não vai mais ao frontend (a regra permanece comentada em `src/lib/freshdesk.js`);
- continuam visíveis fora do painel: carregamento, erros de consulta, falhas de conexão, aviso de HTTP 429, contagem de bloqueio entre pesquisas, confirmação ou erro de busca salva e mensagens que exigem ação;
- pendente (TCK-0113): brilho branco suave atrás do painel, sem exagero e legível no tema escuro.
