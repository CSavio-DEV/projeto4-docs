# Tickets Freshdesk — Painel de contexto e interface da aba

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.13
**Data:** 2026-09-29
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
| Indicador **Busca atual** | tipo de busca no topo da página, fora do Contexto |
| Prefixo `tf-` | IDs dos campos do formulário de Tickets (evita colisão com `f-*` do Portal) |
| Favicon vazio embutido no `<head>` | evita `GET /favicon.ico` 404 |
