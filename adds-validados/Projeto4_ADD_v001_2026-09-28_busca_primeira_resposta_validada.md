# Projeto4 - ADD v001

**Tarefa concluída:** Busca `1ª Resposta` com prazos Vencidos, Hoje e Amanhã  
**Data de validação:** 2026-09-28  
**Versão final validada:** `v0.1.4.13`  
**Versões relacionadas:** `v0.1.4.9`, `v0.1.4.10`, `v0.1.4.11`, `v0.1.4.12` e `v0.1.4.13`  
**Status:** validado pelo usuário

## 1. Objetivo concluído

Corrigir e concluir a busca `1ª Resposta` da aba Tickets Freshdesk, permitindo localizar tickets que ainda não receberam a primeira resposta e classificá-los pelos prazos:

- Vencidos;
- Hoje;
- Amanhã.

A tarefa também preserva:

- os grupos definidos para a busca;
- os status Aberto e Pendente;
- a busca `Abertos Portal N1`;
- a transformação de uma busca salva alterada em `Personalizada não salva`;
- o layout atual da aba Tickets.

## 2. Regra funcional validada

O indicador definitivo para saber se o ticket já recebeu a primeira resposta é:

```text
stats.first_responded_at
```

Interpretação:

```text
first_responded_at preenchido
  -> ticket já recebeu a primeira resposta
  -> excluir do resultado

first_responded_at null
  -> ticket ainda não recebeu a primeira resposta
  -> avaliar o prazo fr_due_by
```

O campo `fr_escalated` não é usado na pesquisa, pois a Search API da conta o rejeitou como campo inválido.

## 3. Classificação dos prazos

A classificação final utiliza a data e hora completa de `fr_due_by`, com os limites dos dias calculados no fuso:

```text
America/Sao_Paulo
```

### Vencidos

```text
stats.first_responded_at == null
AND fr_due_by < agora
AND status Aberto ou Pendente
```

### Hoje

```text
stats.first_responded_at == null
AND fr_due_by >= agora
AND fr_due_by < 00h de amanhã
AND status Aberto ou Pendente
```

### Amanhã

```text
stats.first_responded_at == null
AND fr_due_by >= 00h de amanhã
AND fr_due_by < 00h do dia seguinte
AND status Aberto ou Pendente
```

As três classificações não devem se sobrepor depois da filtragem final realizada pelo backend.

## 4. Fluxo técnico validado

```text
Usuário executa uma busca com Prazo para primeira resposta
  -> Search API localiza candidatos por grupos, status e fr_due_by
  -> backend carrega cada candidato com ?include=stats
  -> verifica stats.first_responded_at
  -> ticket já respondido: excluir
  -> ticket sem primeira resposta: comparar fr_due_by
  -> classificar como Vencidos, Hoje ou Amanhã
  -> devolver somente os tickets válidos ao frontend
```

A resposta interna apresenta metadados como:

```text
candidateCount
returnedCount
criterion: stats.first_responded_at == null
scope: página atual da Search API
```

## 5. Busca salva `1ª Resposta`

Ao carregar a busca salva de sistema `1ª Resposta`, devem vir selecionados por padrão:

```text
Vencidos
Hoje
```

A opção `Amanhã` deve:

- aparecer no campo `Prazo para primeira resposta`;
- permanecer desmarcada por padrão;
- poder ser selecionada manualmente.

Ao alterar manualmente os prazos ou qualquer outro filtro:

```text
Busca salva
  -> Personalizada não salva
```

Mesmo após a mudança para `Personalizada não salva`, a presença de qualquer filtro `frDueBy` mantém ativa a verificação de:

```text
stats.first_responded_at == null
```

## 6. Exemplo validado de Amanhã

Na validação prática, a Search API retornou seis candidatos. Depois da consulta de `stats`, quatro tickets que já haviam recebido primeira resposta foram excluídos.

Resultado final:

```text
Candidatos: 6
Retornados: 2
```

Tickets mantidos:

```text
8038
8034
```

Tickets já respondidos que deixaram de aparecer:

```text
8023
8017
8009
7997
```

## 7. Correções realizadas por versão

### v0.1.4.9

- removeu `fr_escalated:true` e `fr_escalated:false`;
- corrigiu a sintaxe de datas de `fr_due_by` para `YYYY-MM-DD`;
- eliminou o erro `Validation Failed` causado pela consulta anterior.

### v0.1.4.10

- adicionou consulta individual com `include=stats`;
- adotou `stats.first_responded_at` como indicador definitivo;
- passou a comparar a data e hora real de `fr_due_by`.

### v0.1.4.11

- adicionou a opção `Amanhã` ao seletor de prazo.

### v0.1.4.12

- deixou `Amanhã` desmarcado por padrão na busca salva;
- manteve `Vencidos` e `Hoje` como seleção padrão;
- aplicou o filtro de primeira resposta também quando a busca passou a `Personalizada não salva`.

### v0.1.4.13

- corrigiu o erro:

```text
$(...).forEach is not a function
```

- substituiu o seletor de elemento único `$()` pelo seletor de coleção `$$()` ao atualizar os botões de tipo de busca.

## 8. Arquivos relacionados

```text
src/lib/freshdesk.js
src/server.js
src/frontend/principal.html
src/frontend/js/principal.js
data/saved-searches.json
```

## 9. Validações realizadas

- busca `Abertos Portal N1` permaneceu funcional;
- busca `1ª Resposta` deixou de retornar `Validation Failed`;
- `Vencidos` e `Hoje` permanecem selecionados por padrão;
- `Amanhã` aparece disponível e desmarcado por padrão;
- alteração manual converte a busca em `Personalizada não salva`;
- não ocorre mais erro de `forEach`;
- tickets com `first_responded_at` preenchido são excluídos;
- tickets sem primeira resposta são classificados pelo prazo;
- a busca com `Amanhã` retornou corretamente após a recuperação do limite da API.

## 10. Limitação conhecida e pendência separada

Durante os testes ocorreu HTTP 429 por uso simultâneo da mesma conta do Freshdesk por:

- Projeto3 com alertas ativos;
- Projeto4;
- scripts de inspeção;
- buscas repetidas em sequência.

Após aguardar aproximadamente cinco segundos, a pesquisa retornou corretamente.

O controle de limite não invalida esta tarefa funcional, mas permanece como pendência técnica antes dos testes multiusuário:

- cache temporário de `stats` por ticket;
- reutilização de detalhes já carregados;
- controle global de chamadas entre usuários;
- tratamento do cabeçalho `Retry-After`;
- bloqueio de buscas repetidas enquanto uma consulta estiver em andamento;
- retorno parcial ou mensagem controlada em caso de HTTP 429;
- análise da convivência com o Projeto3 enquanto ele continuar consultando a mesma conta.

## 11. Critério de conclusão

A tarefa foi considerada concluída porque:

- a consulta é aceita pela API;
- os tickets respondidos são excluídos;
- os prazos são classificados corretamente;
- Amanhã funciona como opção manual;
- a busca salva mantém apenas Vencidos e Hoje por padrão;
- a transição para `Personalizada não salva` funciona sem erro JavaScript;
- o usuário validou o retorno após o intervalo de recuperação da API.

## 12. Onde incorporar no Manual Técnico

Atualizar:

```text
docs/03-tickets-freshdesk/README.md
docs/03-tickets-freshdesk/filtros-e-buscas.md
docs/03-tickets-freshdesk/validacao.md
docs/03-tickets-freshdesk/pendencias-tickets.md
docs/registros/changelog.md
docs/registros/decisoes.md
docs/registros/erros-e-licoes.md
docs/registros/pendencias.md
docs/00-indice/PROGRESSO-COMPILACAO.md
onde_paramos.md
```

## 13. Próxima pendência de Tickets

Após esta conclusão, a próxima pendência funcional documentada é:

```text
TCK-0102 - Validar Nível N1 ou Nenhum
```

Antes do teste multiusuário da aba Tickets, também será obrigatório implementar ou validar o controle centralizado do limite da API.
