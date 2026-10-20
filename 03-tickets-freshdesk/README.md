# Tickets Freshdesk

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.13 (validada pelo usuário em 2026-09-28; inclui a correção manual anterior em `src/frontend/principal.html`)
**Data:** 2026-09-29
**Status:** vigente

## Escopo

Interface personalizada para localizar, carregar e visualizar chamados do Freshdesk pela API v2, com autenticação por API Key.

## Visão geral

O módulo contempla:

- listagem leve de tickets;
- filtros combináveis, com múltiplos valores no mesmo filtro;
- tipos de busca predefinidos e pesquisas personalizadas;
- buscas salvas em JSON;
- grupos e agentes carregados a partir dos mapas do `.env`;
- painel flutuante com **Contexto geral** e **Contexto do ticket**.

## Estado atual

### Implementado

- servidor Node.js e rotas de tickets, de opções de filtros e de buscas salvas;
- tradutor de filtros para consultas da Search API;
- filtros múltiplos (OU dentro do campo, E entre campos);
- tipos de busca **Abertos Portal N1**, **1ª Resposta** (Vencidos, Hoje e Amanhã; **validada na conta real**, ver `adds-validados/`) e **Personalizada**;
- buscas salvas com `data/saved-searches.json` como fonte única;
- painel de contexto com abas independentes por contexto;
- estado inicial simplificado da aba Tickets (validado pelo usuário);
- tratamento de erros da API (400, 429, 502);
- exclusão, na busca 1ª Resposta, dos tickets que já receberam primeira resposta (`stats.first_responded_at`).

### Validado pelo usuário

Estado inicial simplificado; JSON como fonte única das buscas salvas; `Qualquer`/`Nenhum`/`Não atribuído`; correções de `forEach` e favicon; busca 1ª Resposta com Vencidos, Hoje e Amanhã (v0.1.4.13); `Abertos Portal N1` preservada.

### Implementado, aguardando validação completa

- transformação de busca salva alterada em **Personalizada não salva** (roteiro completo pendente, TCK-0103);
- nível `N1 OR null` (TCK-0102);
- apresentação detalhada dos erros de busca (parcial, TCK-0104).

### Regressões a prevenir

Detalhe do ticket renderizado como JSON bruto (TCK-0202) e texto literal `\n` no topo do HTML.

### Ainda não implementado

- detalhe completo do ticket sob demanda (descrição, conversas, anexos, `stats`, campos personalizados). Hoje o clique em um ticket usa apenas os dados da listagem;
- envio de **Resposta** e **Anotação** (os controles existem, mas ainda não chamam as rotas do Freshdesk);
- mapeamento dos cinco filtros personalizados (Portal, Tipo de usuário, Regional, Unidade escolar, Tipo);
- sincronização dinâmica de grupos e agentes com a conta Freshdesk;
- controle do limite de chamadas: intervalo entre buscas, bloqueio após `HTTP 429`, respeito a `Retry-After`, cache de `stats` (TCK-0105). O limite é da conta e é compartilhado com o Projeto3;
- buscas salvas por usuário, com permissões e auditoria.

As pendências detalhadas, com IDs `TCK-xxxx` e a ordem de execução, ficam em [pendencias-tickets.md](pendencias-tickets.md). `registros/pendencias` referencia esse arquivo em vez de repeti-lo.

## Arquivos envolvidos

| Arquivo | Responsabilidade |
|---|---|
| `src/server.js` | Rotas da integração; leitura dos filtros como arrays (`getAll`); rotas de buscas salvas |
| `src/lib/freshdesk.js` | Comunicação com o Freshdesk, Basic Auth, normalização de erros, montagem das consultas, mapeamentos, combinação de valores múltiplos e tipos de busca |
| `src/lib/savedSearches.js` | Persistência das buscas salvas (listar, criar, atualizar, excluir) |
| `src/lib/env.js` | Loader manual do `.env` com `node:fs`; expõe domínio, API Key, campo de nível, mapas de grupos/agentes e se a integração está configurada |
| `src/frontend/principal.html` | Painel de contexto, formulários, seletores, componentes de busca e buscas salvas |
| `src/frontend/js/principal.js` | Alternância de contextos e abas, componente de múltipla seleção, estado do tipo de busca, buscas salvas, opções dinâmicas |
| `src/frontend/css/principal.css` | Componente `.multiselect` e demais elementos visuais |
| `data/saved-searches.json` | Buscas salvas; fonte de verdade (ver `filtros-e-buscas.md`) |
| `.env` | Domínio e API Key do Freshdesk, chave do campo de nível, mapas de grupos e agentes |
| `.env.example` | Modelo sem credenciais reais |

## Documentos deste assunto

| Documento | Conteúdo |
|---|---|
| [api-e-regras.md](api-e-regras.md) | API do Freshdesk, rotas locais, mapeamentos, regras de negócio, erros |
| [filtros-e-buscas.md](filtros-e-buscas.md) | Filtros, valores especiais, tipos de busca, buscas salvas |
| [painel-contexto.md](painel-contexto.md) | Contexto geral e do ticket, abas, estado inicial da lista |
| [validacao.md](validacao.md) | Roteiro de regressão e critérios de aceite |
| [pendencias-tickets.md](pendencias-tickets.md) | Pendências da aba (TCK-xxxx), critérios de aceite, ordem de execução, teste multiusuário |

## Registros de tarefas concluídas

- `adds-validados/Projeto4_ADD_v001_2026-09-28_busca_primeira_resposta_validada.md` — busca 1ª Resposta (v0.1.4.9 a v0.1.4.13).

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/` (vazia: tudo consolidado até a v0.1.4.13).
