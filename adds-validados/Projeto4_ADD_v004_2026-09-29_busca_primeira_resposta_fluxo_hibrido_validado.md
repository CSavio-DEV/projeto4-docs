# Projeto4 ADD v004 - Busca 1ª Resposta, proteção da API e fluxo híbrido

**Data:** 29/09/2026  
**Projeto:** `parte4_cloudflare`  
**Versão de referência:** `v0.1.4.35`  
**Tipo:** ADD evolutivo e atualizável  
**Situação:** fluxo da busca `1ª Resposta` validado como nova base de arquitetura; resultados funcionais, paginação, cache e classificações continuam sujeitos a refinamentos documentados.

---

## 1. Objetivo deste ADD

Registrar a evolução da busca `1ª Resposta`, desde o modelo inicial de consultas individuais até o fluxo híbrido com pré-seleção, listagem paginada com `stats`, cruzamento por ID, cache persistente e logs técnicos.

Este ADD pode ser atualizado futuramente. A validação registrada aqui não encerra a evolução da busca. Novos ajustes poderão complementar:

- paginação;
- cobertura dos candidatos;
- classificação de `Vencidos`, `Hoje` e `Amanhã`;
- cache persistente;
- auditoria dos grupos;
- proteção preventiva contra `HTTP 429`;
- apresentação de resultados parciais ou incompletos.

---

## 2. Regra funcional da busca 1ª Resposta

A busca deve retornar somente tickets que atendam simultaneamente aos critérios:

```text
Grupo permitido
AND
Status Aberto ou Pendente
AND
Prazo de primeira resposta correspondente à seleção
AND
stats.first_responded_at == null
```

### 2.1 Status permitidos

```text
2 = Aberto
3 = Pendente
```

A busca fixa passou a preencher e enviar explicitamente:

```text
status=Aberto
status=Pendente
```

### 2.2 Grupos permitidos

```text
RDS
PSGE
Portal SESI - Pedagógico
Portal SESI - Técnico/Funcionamento
Não atribuído
```

Os nomes são resolvidos para os IDs configurados no `.env` por meio de `FRESHDESK_GROUP_MAP`.

### 2.3 Prazos

A busca pode trabalhar com:

```text
Vencidos
Hoje
Amanhã
```

A confirmação definitiva de que a primeira resposta ainda não ocorreu é:

```javascript
ticket.stats?.first_responded_at == null
```

Campos como `responder_id`, igualdade entre `created_at` e `updated_at` ou ausência de alteração posterior não substituem essa confirmação.

---

## 3. Problema do fluxo anterior

O fluxo anterior executava:

```text
1 chamada à Search API
+
1 chamada GET /tickets/{id}?include=stats por candidato
```

Os logs comprovaram o custo real informado pelo Freshdesk:

```text
Search API:
X-RateLimit-Used-CurrentRequest = 1

Detalhe individual com include=stats:
X-RateLimit-Used-CurrentRequest = 2
```

Em uma execução foram consumidos:

```text
1 crédito da Search API
+
24 detalhes × 2 créditos
=
49 créditos
```

Quando restava apenas um crédito, o sistema iniciou outro detalhe que exigia dois créditos e recebeu:

```text
HTTP 429 Too Many Requests
```

O problema não era apenas concorrência. Mesmo com chamadas sequenciais, a quantidade total de créditos ultrapassava a janela disponível.

---

## 4. Ativação dos logs técnicos

A aba Logs foi ativada no novo layout do Projeto4 com três modos:

```text
Desligado
Resumido
Completo
```

Configuração inicial:

```dotenv
PROJETO4_LOG_LEVEL=summary
PROJETO4_LOG_MAX_ENTRIES=500
PROJETO4_LOG_REFRESH_MS=2000
PROJETO4_LOG_TIMEZONE=America/Fortaleza
PROJETO4_LOG_FILE_ENABLED=true
```

Os logs passaram a registrar:

- tipo e rótulo da busca;
- operação executada;
- duração;
- status HTTP;
- limite total;
- limite restante;
- créditos usados na requisição;
- cache hit e miss;
- candidatos;
- páginas consultadas;
- IDs encontrados e ausentes;
- interrupção por `HTTP 429`;
- resumo da busca.

Dados sensíveis não devem ser registrados.

---

## 5. Descoberta sobre include=stats em lote

Foi confirmado em teste real que:

```text
GET /api/v2/tickets?include=stats
```

retorna vários tickets na mesma página, cada um contendo:

```json
{
  "stats": {
    "first_responded_at": null,
    "agent_responded_at": null,
    "requester_responded_at": null,
    "resolved_at": null,
    "closed_at": null
  }
}
```

Na conta testada, uma página com `include=stats` consumiu:

```text
X-RateLimit-Used-CurrentRequest = 2
```

Isso permitiu eliminar dezenas de chamadas individuais.

Também foi confirmado que o parâmetro não documentado:

```text
?id=8021,8022,8023
```

foi ignorado pela listagem. O endpoint retornou a página normal de tickets, não somente os IDs informados.

---

## 6. Primeira solução em lote

A primeira solução em lote usou:

```text
GET /api/v2/tickets
  ?include=stats
  &updated_since=<últimos dias>
  &per_page=100
  &page=1
```

Depois, o Projeto4 filtrava localmente:

```text
status 2 ou 3
grupo permitido
first_responded_at nulo
fr_due_by preenchido
prazo selecionado
```

Resultado observado:

```text
Tempo aproximado: 1 segundo
Chamadas externas: 1
Créditos consumidos: 2
Detalhes individuais: 0
HTTP 429: não
```

### 6.1 Limitação encontrada

A página continha 100 tickets genéricos, mas não representava diretamente os candidatos da busca.

Exemplo do funil:

```text
100 tickets listados
20 Abertos ou Pendentes
10 nos grupos permitidos
1 sem primeira resposta
0 em Vencidos ou Hoje
```

A resposta foi rápida e econômica, mas a cobertura podia ser incompleta porque candidatos relevantes poderiam estar nas páginas seguintes ou fora do intervalo de `updated_since` escolhido.

---

## 7. Fluxo híbrido validado como nova base

Para combinar precisão e economia, foi implementado o fluxo híbrido.

### 7.1 Etapa 1: Search API

A Search API pré-seleciona com os filtros reais:

```text
Grupos permitidos
AND
Status Aberto ou Pendente
AND
Prazo selecionado
```

A Search API retorna os candidatos com:

```text
id
updated_at
status
group_id
fr_due_by
fr_escalated
```

### 7.2 Etapa 2: menor updated_at

O sistema calcula:

```text
menor updated_at entre os candidatos
```

Depois aplica margem de segurança:

```text
updated_since = menor updated_at - 5 segundos
```

O `updated_at` é usado porque o parâmetro `updated_since` trabalha com a data de atualização, não com `created_at`.

### 7.3 Etapa 3: listagem paginada com stats

```text
GET /api/v2/tickets
  ?updated_since=<menor updated_at - margem>
  &include=stats
  &per_page=100
  &page=N
  &order_by=updated_at
  &order_type=asc
```

### 7.4 Etapa 4: cruzamento por ID

O sistema mantém:

```text
IDs candidatos da Search API
IDs localizados nas páginas com stats
IDs ainda ausentes
```

As páginas seguintes são carregadas até:

```text
todos os candidatos serem encontrados
OU
chegar ao fim da listagem
OU
atingir o máximo configurado
```

### 7.5 Etapa 5: confirmação final

```text
stats.first_responded_at == null
```

Depois, os tickets são classificados em:

```text
Vencidos
Hoje
Amanhã
```

---

## 8. Resultado validado do fluxo híbrido

Em um teste real:

```text
Candidatos da Search API: 26
Status dos candidatos: 26 Abertos, 0 Pendentes, 0 Outros
Páginas da Search API: 1
Páginas com stats: 4
Tickets examinados nas páginas com stats: 309
Candidatos cruzados: 26
IDs ausentes: 0
Sem primeira resposta: 2
Chamadas externas: 5
Créditos consumidos: 9
Tempo total: aproximadamente 2,6 segundos
HTTP 429: não
Cobertura: completa
```

Os 26 candidatos coincidiram com a quantidade observada na busca manual com grupos, Aberto e Pendente.

O fluxo híbrido foi mais caro que uma única página genérica, mas garantiu cobertura completa dos candidatos e permaneceu muito abaixo do fluxo individual que consumia aproximadamente 49 créditos.

---

## 9. Paginação

### 9.1 Search API

A Search API trabalha com páginas de até 30 candidatos.

Configuração:

```dotenv
FRESHDESK_FIRST_RESPONSE_SEARCH_MAX_PAGES=10
```

### 9.2 Listagem com stats

```dotenv
FRESHDESK_FIRST_RESPONSE_PER_PAGE=100
FRESHDESK_FIRST_RESPONSE_STATS_MAX_PAGES=10
FRESHDESK_FIRST_RESPONSE_UPDATED_SAFETY_SECONDS=5
```

### 9.3 Parada antecipada

A listagem com stats não precisa percorrer todas as páginas quando todos os candidatos já foram encontrados.

```text
Página 1 encontra parte dos candidatos
Página 2 encontra novos candidatos
...
Todos os IDs encontrados
Parar imediatamente
```

### 9.4 Resultado incompleto

Se o máximo de páginas for atingido e ainda existirem IDs ausentes:

```json
{
  "complete": false,
  "missingCandidateCount": 2,
  "missingCandidateIds": [8071, 8066]
}
```

Resultados incompletos não devem ser apresentados como cobertura total.

---

## 10. Cache persistente

Foi criado o cache:

```text
data/cache/freshdesk-ticket-stats.json
```

Chave lógica:

```text
ticket_id + updated_at
```

### 10.1 Cache hit

```text
Mesmo ID
+
Mesmo updated_at
=
reutilizar stats sem nova página
```

### 10.2 Cache miss

```text
Ticket novo
OU
updated_at alterado
=
procurar stats nas páginas
```

### 10.3 Invalidação

Quando o `updated_at` muda, a versão anterior do mesmo ticket deixa de ser reutilizada e é substituída após nova obtenção.

### 10.4 Persistência após reinício

O cache continua existindo depois de:

```text
Ctrl + C
npm start
```

Configurações:

```dotenv
FRESHDESK_FIRST_RESPONSE_CACHE_TTL_HOURS=168
FRESHDESK_FIRST_RESPONSE_CACHE_MAX_ENTRIES=2000
```

### 10.5 Dados armazenados

Somente dados técnicos necessários, sem descrição, conversa, e-mail, CPF, anexos, cookies, tokens ou API key.

---

## 11. Regra unificada para buscas que necessitam stats

Foi identificado que adicionar `Amanhã` ou alterar filtros pode transformar a busca fixa em:

```text
Personalizada
```

Anteriormente, buscas personalizadas com `frDueBy` voltavam ao fluxo antigo individual e podiam gerar `HTTP 429`.

A regra corrigida é:

```text
Se frDueBy estiver preenchido,
usar fluxo híbrido e cache persistente,
independentemente do nome ou tipo da busca.
```

Isso vale para:

- busca fixa `1ª Resposta`;
- busca personalizada;
- busca salva;
- Vencidos;
- Hoje;
- Amanhã;
- qualquer combinação desses prazos.

Buscas com `frDueBy` não devem produzir:

```text
operation = ticket_details_stats
```

Devem produzir:

```text
operation = search_first_response_candidates
```

E, quando houver cache misses:

```text
operation = list_candidate_stats
```

---

## 12. Proteção da API Freshdesk

Os cabeçalhos usados para diagnóstico são:

```text
X-RateLimit-Total
X-RateLimit-Remaining
X-RateLimit-Used-CurrentRequest
Retry-After
```

A arquitetura atual reduz o risco de limite por:

- pré-seleção com Search API;
- listagem com stats em lote;
- paginação controlada;
- parada antecipada;
- cache persistente;
- ausência de consultas individuais nas buscas com `frDueBy`;
- logs completos de consumo.

Ainda permanece recomendada uma proteção global preventiva para não iniciar chamadas quando o limite restante não comportar o custo da próxima operação.

---

## 13. Desafios e pontos que permanecem abertos

### 13.1 Classificação de prazo

Precisamos continuar validando:

```text
Vencidos
Hoje
Amanhã
```

Especialmente:

- fuso horário;
- limite entre dias;
- comportamento de `fr_escalated`;
- diferença entre a granularidade da Search API e a hora completa de `fr_due_by`;
- tickets que entram na Search API mas são rejeitados no classificador local.

### 13.2 Auditoria de grupos

Os logs devem evoluir para mostrar:

- nome do grupo;
- ID resolvido;
- quantidade de candidatos por grupo;
- quantidade excluída por grupo;
- tratamento de Não atribuído.

### 13.3 Consumo real

O resumo final deve sempre usar:

```text
apiUnitsConsumed
```

obtido do fluxo, sem estimativas antigas conflitantes.

### 13.4 Cache e buscas diferentes

O cache deve permanecer compartilhado por qualquer busca que necessite de `stats`, desde que a chave `ticket_id + updated_at` corresponda.

### 13.5 Paginação extensa

Se os candidatos estiverem espalhados por muitas páginas, o custo cresce em duas unidades por página com stats. O limite máximo configurável e o estado `complete` devem continuar obrigatórios.

### 13.6 Resultado parcial

Se nem todos os IDs forem encontrados, o frontend deverá futuramente informar claramente:

```text
Resultado parcial
X candidatos analisados
Y candidatos ainda pendentes
```

---

## 14. Versões principais deste ciclo

```text
v0.1.4.21
  removeu per_page indevido da Search API

v0.1.4.22
  tornou detalhes sequenciais

v0.1.4.25
  criou cache em memória, deduplicação, contexto e governador

v0.1.4.26
  tornou concorrência configurável

v0.1.4.27
  impediu queda do Node.js em HTTP 429

v0.1.4.29
  ativou a aba Logs com instalador corrigido

v0.1.4.30
  adicionou contexto da busca e consumo por operação nos logs

v0.1.4.31
  explicitou Aberto e Pendente na busca fixa

v0.1.4.32
  testou listagem em lote com stats

v0.1.4.33
  implementou fluxo híbrido com cruzamento por ID

v0.1.4.34
  adicionou cache persistente de stats

v0.1.4.35
  aplicou fluxo híbrido a toda busca com frDueBy
```

---

## 15. Critérios de validação registrados

Considera-se validado como nova base:

- a Search API reduz corretamente os candidatos por filtros;
- `include=stats` em listagem retorna stats de vários tickets;
- a listagem em lote consome poucos créditos por página;
- o cruzamento por ID encontra os candidatos sem consulta individual;
- a paginação pode parar quando todos os IDs forem encontrados;
- o fluxo híbrido concluiu 26 candidatos sem IDs ausentes;
- o fluxo híbrido evitou `HTTP 429` no teste completo;
- o cache persistente foi criado para reduzir chamadas futuras;
- buscas com `frDueBy` devem compartilhar a mesma lógica híbrida.

Permanece pendente de validação continuada:

- comportamento do cache em segunda execução e após reinício;
- busca personalizada com `Amanhã` depois da `v0.1.4.35`;
- distribuição e auditoria dos grupos nos logs;
- resultado funcional esperado de Vencidos, Hoje e Amanhã;
- apresentação de resultados parciais;
- proteção preventiva global por crédito restante.

---

## 16. Fluxo consolidado

```text
Usuário executa busca com frDueBy
  -> Search API aplica grupos, status e prazo
  -> coletar candidatos e updated_at
  -> verificar cache persistente por ticket_id + updated_at
  -> separar cache hits e misses
  -> se houver misses:
       calcular menor updated_at dos ausentes
       aplicar margem de segurança
       listar páginas com include=stats
       cruzar IDs
       salvar encontrados no cache
       parar quando todos forem encontrados
  -> manter first_responded_at null
  -> classificar Vencidos, Hoje e Amanhã
  -> devolver resultado e diagnóstico
  -> registrar créditos, páginas, cache e cobertura
```

---

## 17. Decisão para continuidade

A arquitetura recomendada para buscas que precisam de `stats` é:

```text
Search API para precisão
+
List Tickets com stats para eficiência
+
Cruzamento por ID para integridade
+
Cache persistente para economia recorrente
+
Logs para auditoria
```

Não retornar ao modelo de uma consulta individual por ticket como fluxo padrão.

Este ADD poderá ser revisado e ampliado quando as próximas validações forem concluídas.
