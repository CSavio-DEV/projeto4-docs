# Tickets Freshdesk — Roteiro de validação

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

Roteiro único de regressão. Executar em desktop e mobile, e conferir console e rede ao final de cada bloco.

## 1. Interface

1. abrir Tickets sem tickets carregados;
2. confirmar um único botão, o ícone e **Nenhum ticket carregado**;
3. abrir e fechar o Contexto geral;
4. alternar repetidamente entre Filtros, Propriedades e Automações e confirmar que nenhum conteúdo desaparece;
5. mudar para o Contexto do ticket e confirmar que Filtros não aparece;
6. fechar o ticket e confirmar que a aba anterior do Contexto geral foi preservada.

## 2. Filtros

1. confirmar `Qualquer` em cinza e ausente do dropdown;
2. confirmar `Nenhum` em Nível e `Não atribuído` em Grupo;
3. selecionar dois status, dois grupos e duas tags separadas por vírgula;
4. executar e inspecionar a consulta gerada: OU dentro do campo, E entre campos;
5. repetir com agente, prioridade e nível;
6. testar os campos ainda não mapeados e confirmar o aviso;
7. confirmar que qualquer mudança de filtro ativa **Personalizada não salva**;
8. mudar filtros e verificar `#filterFeedback` quando o resultado não variar.

## 3. Tipos de busca

**Abertos Portal N1**

1. abrir a busca e conferir os 5 grupos, os status Aberto e Pendente e os níveis N1 e Nenhum;
2. executar e inspecionar a consulta (`N1 OR null`);
3. confirmar no log a chave real `cf_nvel_de_atendimento`.

**1ª Resposta** (regra validada pelo usuário na v0.1.4.13; fluxo híbrido a partir da v0.1.4.33; este roteiro serve de regressão)

1. conferir grupos, status Aberto/Pendente e o filtro de prazo com Vencidos, Hoje e Amanhã;
2. ao carregar a busca salva, confirmar Vencidos e Hoje marcados e Amanhã disponível e desmarcado;
3. executar e confirmar que não há `Validation Failed` nem `invalid_field` (nenhuma consulta usa `fr_escalated`);
4. confirmar que tickets com `stats.first_responded_at` preenchido não aparecem e que a resposta traz `candidateCount` e `returnedCount` coerentes;
5. conferir que Vencidos, Hoje e Amanhã não se sobrepõem;
6. alterar um filtro e confirmar **Personalizada não salva**, sem erro de JavaScript, mantendo a verificação de primeira resposta enquanto houver filtro de prazo;
7. se ocorrer HTTP 429, aguardar a contagem regressiva dos botões antes de repetir (ver seção 3.1 e TCK-0105);
8. abrir a aba Logs e conferir que a busca gerou `search_first_response_candidates` (e `list_candidate_stats` quando houve cache miss) e **nenhum** `ticket_details_stats`;
9. conferir no resumo: candidatos, páginas, cache hits e misses, `apiUnitsConsumed` real, `complete: true` e nenhum ID ausente; sem `HTTP 429`;
10. repetir a mesma busca e confirmar cache hits (segunda execução); reiniciar o servidor, repetir e confirmar que o cache persistiu (`data/cache/freshdesk-ticket-stats.json`);
11. marcar `Amanhã` numa busca personalizada e confirmar que usa o mesmo fluxo (sem `ticket_details_stats`);
12. conferir que o cache não guarda descrição, conversa, e-mail, CPF, anexo, cookie, token ou API key.

**Personalizada**

1. executar sem filtros e confirmar o alerta do frontend (primeiros 30 tickets, com confirmação) e o `warnings` do backend;
2. verificar paginação e volume retornado;
3. tentar salvar com o nome `Personalizada`, `Personalizada não salva` ou `Personalizada (não salva)` (com variações de caixa) e confirmar a mensagem de nome reservado, no frontend e no backend;
4. a partir de uma busca salva, entrar em `Personalizada (não salva)` e confirmar que os filtros são limpos uma única vez; clicar de novo e confirmar que filtros preenchidos permanecem.

### 3.1 Limitador de pesquisas (v0.1.4.14)

1. iniciar uma pesquisa e confirmar `Buscando…` e botões `Carregar tickets` e `Buscar` bloqueados;
2. ao concluir, confirmar a contagem de 5 s nos botões e a liberação automática;
3. em HTTP 429, confirmar o aviso curto, o bloqueio de no mínimo 10 s (ou `retryAfter`, se maior), a contagem regressiva e a ausência de novas chamadas no período;
4. forçar um 429 e confirmar que o servidor Node.js continua ativo (v0.1.4.27).

### 3.2 Seletor de buscas e botão `i` (v0.1.4.15 e v0.1.4.16)

1. abrir o menu e conferir a ordem: Abertos Portal N1, 1ª Resposta, até três adicionais, `Personalizada (não salva)` por último, no máximo cinco buscas salvas;
2. conferir as cores de `Personalizada (não salva)` no botão e no item ativo do menu;
3. selecionar **N2-Abertos** e confirmar que continua como busca ativa (erro conhecido, TCK-0110);
4. conferir que o nome da busca aparece só no seletor principal;
5. abrir o painel do botão `i` e fechar por `×`, por clique fora e por `Esc`; conferir os textos de Vencidos, Hoje, Amanhã, limite da API e abertura de ticket;
6. conferir que permanecem visíveis: carregamento, erros, falhas de conexão, aviso de HTTP 429, contagem de bloqueio e confirmação ou erro de busca salva.

## 4. Buscas salvas

1. salvar uma busca, recarregar a página e carregá-la; conferir nome, tipo e arrays;
2. sobrescrever usando o mesmo nome e excluir;
3. carregar Abertos Portal N1, alterar um filtro e confirmar **Personalizada não salva** e `searchType=personalizada`;
4. voltar os filtros ao valor anterior: deve continuar personalizada não salva;
5. recarregar explicitamente a busca salva e confirmar a restauração do tipo original;
6. editar o JSON manualmente e reiniciar: buscas apagadas não voltam e a ordem do arquivo é mantida;
7. repetir com duas sessões simultâneas para observar concorrência.

## 5. Detalhe do ticket

1. abrir o ticket pela lista leve;
2. confirmar renderização visual do detalhe (nunca JSON bruto);
3. validar requester, descrição, campos personalizados, conversas, anexos e `stats`;
4. fechar e reabrir;
5. verificar erros e warnings do backend e do navegador.

## 5.1 Aba Logs

1. conferir os modos `Desligado`, `Resumido` e `Completo`;
2. executar uma busca e conferir tipo e rótulo da busca, operação, duração, HTTP, `X-RateLimit-*`, cache hit e miss, candidatos e resumo;
3. conferir que nenhum log contém API key, `Authorization`, cookie, token ou senha;
4. conferir o nome do arquivo diário: deve usar a data local (erro conhecido, P4-1005);
5. limpar os logs e conferir o resultado.

## 6. Console e rede

Confirmar ausência de:

```text
$(...).forEach is not a function
/favicon.ico 404
texto literal "\\n" no topo da página
```

Verificar as respostas HTTP 400 e as mensagens apresentadas.

## 7. Critérios de aceite

A entrega é considerada validada quando:

- as abas funcionam repetidamente sem ocultar todos os painéis;
- Contexto geral e Contexto do ticket mantêm estados independentes;
- filtros múltiplos geram a lógica esperada;
- grupos e agentes são carregados corretamente;
- Abertos Portal N1 retorna o conjunto operacional esperado;
- 1ª Resposta tem regra confirmada com a conta real (**atendido na v0.1.4.13**);
- os campos personalizados sem efeito estão mapeados ou explicitamente desabilitados;
- a busca vazia tem proteção confirmada no frontend e no backend;
- buscas salvas persistem sem expor dados inadequados;
- o detalhe do ticket continua visual e completo;
- erros da API, especialmente `429`, são tratados de modo previsível (frontend validado na v0.1.4.14; backend aguardando validação);
- buscas com prazo de primeira resposta usam o fluxo híbrido e o cache, com cobertura completa ou marcada como parcial;
- os testes foram executados também em cenário multiusuário.
