# Tickets Freshdesk — API, rotas e regras de negócio

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.35
**Data:** 2026-10-20
**Status:** vigente

## 1. Autenticação

A integração usa HTTP Basic Auth:

- **username:** valor de `FRESHDESK_API_KEY`;
- **password:** `X`;
- **base URL:** `https://{FRESHDESK_DOMAIN}/api/v2`.

Configuração no `.env`:

```env
FRESHDESK_DOMAIN=seu_subdominio.freshdesk.com
FRESHDESK_API_KEY=sua_chave_api
FRESHDESK_NIVEL_FIELD=cf_nvel_de_atendimento
```

A API Key real existe somente no `.env` do servidor. O campo visual de API Key no HTML não pode vir preenchido (um valor preenchido foi removido na v0.1.4.0). Credenciais reais não entram em documentos nem em repositório.

## 2. Operações do Freshdesk usadas

### Listagem leve

```http
GET /api/v2/tickets?per_page=10
```

Recupera informações resumidas sem carregar antecipadamente toda a conversa.

### Detalhe do ticket

```http
GET /api/v2/tickets/{ticket_id}?include=conversations
```

Destinada a carregar descrição, anexos e conversas. **Ainda não está integrada** ao painel (ver seção 7). O contrato efetivo deve ser validado na conta real.

### Busca avançada

```http
GET /api/v2/search/tickets?query="..."
```

Exemplos de cláusulas:

```text
status:2
priority:4
group_id:68000004919
(group_id:68000004918 OR group_id:68000004924)
cf_nvel_de_atendimento:'N1'
fr_due_by:>'2026-09-27' AND fr_due_by:<'2026-09-29'
```

Regras validadas na conta real (v0.1.4.9 a v0.1.4.35):

- a consulta completa vai entre aspas duplas no parâmetro `query` e deve ser codificada na URL;
- datas em `fr_due_by` usam somente `YYYY-MM-DD` (`fr_due_by:<'2026-09-28'`, `fr_due_by:'2026-09-28'`, intervalo com `>` e `<`);
- **`fr_escalated` não é pesquisável**: aparece no JSON do ticket, mas a Search API responde `invalid_field` (`Unexpected/invalid field in request`). Não usar `fr_escalated:true` nem `fr_escalated:false`;
- a Search API trabalha com páginas de até 30 candidatos e aceita `page`; **`per_page` não vale em `/search/tickets`** e gera HTTP 400 (removido na v0.1.4.21);
- a Search API devolve candidatos (entre outros: `id`, `updated_at`, `status`, `group_id`, `fr_due_by`, `fr_escalated`, `responder_id`), mas **não devolve `stats.first_responded_at`**; para saber se o ticket já recebeu a primeira resposta é preciso obter `stats` (seções abaixo).

### Listagem em lote com `stats` (usada pelas buscas com prazo de primeira resposta)

```http
GET /api/v2/tickets
  ?updated_since=<menor updated_at dos candidatos − margem>
  &include=stats
  &per_page=100
  &page=N
  &order_by=updated_at
  &order_type=asc
```

Cada ticket da página traz `stats` (`first_responded_at`, `agent_responded_at`, `requester_responded_at`, `resolved_at`, `closed_at`). O backend cruza por ID com os candidatos da Search API e para quando todos forem encontrados. `updated_since` usa a data de atualização, não `created_at`; a margem de segurança padrão é de 5 s. O parâmetro `?id=8021,8022,...` **é ignorado** pela listagem (devolve a página normal): não usar.

Fluxo completo, cache e cobertura: `filtros-e-buscas.md`, seção 5.2, e `adds-validados/Projeto4_ADD_v004_...`.

### Detalhe individual com `stats` (não usar nas buscas com `frDueBy`)

```http
GET /api/v2/tickets/{ticket_id}?include=stats
```

Foi o fluxo da v0.1.4.10 à v0.1.4.32: uma chamada por candidato. Custa 2 créditos cada, e chegou a 49 créditos numa execução (1 + 24 × 2), causando HTTP 429. Desde a v0.1.4.35 nenhuma busca com `frDueBy` o utiliza (os logs não devem mostrar `operation = ticket_details_stats` nelas).

## 3. Rotas locais do backend

| Rota | Método | Uso |
|---|---|---|
| `/api/tickets` | GET | Busca de tickets. Filtros chegam como parâmetros repetidos (ex.: `group=...&group=...`) mais `searchType` |
| `/api/config/filters` | GET | Nomes de grupos e agentes configurados no `.env`; nunca revela a API Key. O frontend usa para preencher os seletores |
| `/api/saved-searches` | GET | Lista as buscas salvas |
| `/api/saved-searches` | POST | Salva a busca atual ou atualiza uma existente com o mesmo nome |
| `/api/saved-searches?name=...` | DELETE | Exclui uma busca salva |
| `/api/logs` | GET / DELETE | Lê os logs técnicos de forma incremental / limpa a memória de logs (v0.1.4.29) |
| `/api/logs/status` | GET | Estado do serviço de logs |
| `/api/logs/today` | GET | Arquivo diário do dia |
| `/api/logs/settings` | PUT | Altera o modo de log (`off`, `summary`, `full`) |

## 4. Prioridade, nível e status

### Prioridade nativa

| Valor | Prioridade |
|---:|---|
| 1 | Baixa |
| 2 | Média |
| 3 | Alta |
| 4 | Urgente |

### Nível de atendimento

Prioridade e nível são campos diferentes. O nível é um campo personalizado da operação:

```text
custom_fields.cf_nvel_de_atendimento
```

Valores esperados: `N1`, `N2`, `N3` e vazio ou nulo. O valor literal aceito pela conta real (`N1` ou `Nível 1`) e a grafia exata da chave ainda precisam ser confirmados e padronizados.

### Status

| Valor | Status |
|---:|---|
| 2 | Aberto |
| 3 | Pendente |
| 4 | Resolvido |
| 5 | Fechado |

A interface também exibe **Aguardando o cliente** e **Aguardando terceiros**. Seus códigos reais não estão registrados e precisam ser mapeados antes de qualquer regra depender deles.

## 5. Grupos e agentes

Grupos e agentes são **fixos nos mapas do `.env`** (por exemplo `FRESHDESK_GROUP_MAP`). A rota `/api/config/filters` expõe apenas os nomes.

Dois conceitos diferentes:

1. **preenchimento dinâmico da interface**: já realizado, a partir de `/api/config/filters`;
2. **sincronização dinâmica com a conta Freshdesk** (`GET /api/v2/groups` e `GET /api/v2/agents`): não implementada.

O ID mais recente documentado para o grupo PSGE é `68000013925`. O ID deve ser validado na conta real e usado por uma única fonte de configuração (`.env`, cache, filtros e as buscas fixas).

O grupo **Não atribuído** é tratado internamente como `group_id:null` e funciona mesmo que essa chave não exista em `FRESHDESK_GROUP_MAP`.

## 6. Campos retornados e estrutura interna

| Campo interno | Origem Freshdesk | Observação |
|---|---|---|
| `prioridade` | `priority` | inteiro de 1 a 4 |
| `grupo` | `group_id` | ID do grupo |
| `status` | `status` | código do estado |
| `nivel` | `custom_fields.cf_nvel_de_atendimento` | campo personalizado |
| `tag` | `tags` | lista de strings |
| `ticket_id` | `id` | identificador único |
| `titulo` | `subject` | assunto do chamado |
| `nome_requester` | dados do requester | confirmar contrato real do detalhe |
| `email_requester` | dados do requester | confirmar contrato real do detalhe |
| `data_criacao_ticket` | `created_at` | ISO 8601 |
| `data_atualizacao` | `updated_at` | ISO 8601 |
| `data_resolucao_fechamento` | `stats.resolved_at` / `stats.closed_at` | exige presença de `stats` |
| `data_vencimento` | `due_by` | prazo de resolução |
| `data_vencimento_primeira_resposta` | `fr_due_by` | prazo de primeira resposta |

`requester` (solicitante) e `responder_id` (agente) representam papéis diferentes. A Search API fornece `requester_id`, mas não necessariamente nome e e-mail completos; esses dados podem exigir `/contacts/{id}` ou o endpoint de detalhe.

## 7. Estado do detalhe do ticket

Clicar em um ticket preenche apenas cabeçalho e propriedades com os dados já recebidos pela listagem. Não há segunda chamada para descrição completa, conversas ou anexos, e o bloco visual de mensagem é fixo no HTML.

Devem ser tratados separadamente:

- **abertura visual do painel**: existente;
- **detalhe completo sob demanda**: pendente;
- **responder ticket e criar anotação**: os campos e botões existem, mas ainda não chamam `POST /tickets/{id}/reply` nem `POST /tickets/{id}/notes`. Regra a manter: nunca enviar automaticamente conteúdo carregado no editor.

Regra para a integração do detalhe: preservar o renderizador visual completo e não regredir para JSON bruto.

## 8. Identificadores do formulário

O formulário de Tickets Freshdesk usa o prefixo `tf-` nos IDs dos campos (`tf-tags`, `tf-group`, ...) para não colidir com os IDs `f-*` da aba Portal. O prefixo deve ser preservado ao alterar ou integrar os dois formulários.

## 9. Tratamento de erros

| Situação | Resposta |
|---|---|
| `.env` ausente ou incompleto | `400` |
| Filtro ou consulta inválidos | `400` com detalhes; o frontend tenta exibir o detalhe real do erro |
| `HTTP 429` do Freshdesk | status e `retryAfter` repassados ao cliente (ver limite de chamadas abaixo) |
| Outros erros do Freshdesk | corpo original repassado quando possível |
| Erro de rede ou domínio inalcançável | `502` |

Retry automático com laço de repetição não existe e não deve existir: o 429 é devolvido ao frontend como resposta controlada.

### Limite de chamadas da conta (HTTP 429)

O limite de requisições é **da conta Freshdesk**, compartilhado por tudo que a consulta (Projeto3 com alertas ativos, Projeto4, scripts de inspeção e buscas repetidas).

**Custo observado na conta testada (2026-09-29):**

| Chamada | Créditos |
|---|---:|
| Search API (`/search/tickets`) | 1 |
| Detalhe individual com `include=stats` | 2 |
| Página da listagem com `include=stats` | 2 |

Cabeçalhos para diagnóstico: `X-RateLimit-Total` (50 no teste), `X-RateLimit-Remaining`, `X-RateLimit-Used-CurrentRequest` e `Retry-After`. Confirmar nos logs, porque o plano da conta pode mudar.

**Controle em duas camadas:**

| Camada | Regra | Situação |
|---|---|---|
| Frontend (v0.1.4.14) | sem pesquisa simultânea; 5 s entre pesquisas; após HTTP 429, bloqueio de no mínimo 10 s ou `retryAfter` se maior, com contagem regressiva | validado (ADD v002) |
| Backend (v0.1.4.25 a v0.1.4.27) | fila global com intervalo mínimo (padrão 5 s) e concorrência configurável (padrão 1, faixa de 1 a 10); cache de chamadas idênticas (60 s); deduplicação; bloqueio de no mínimo 60 s ou `Retry-After` maior após 429; intervalo de 10 s quando `X-RateLimit-Remaining <= 5`; o 429 não derruba o processo | aguardando validação |

A diferença entre 10 s e 60 s está em aberto (P4-1209). Um intervalo mínimo de 0 ms não é considerado seguro. A **proteção preventiva por crédito restante**, o retorno parcial no frontend e o controle entre usuários seguem em `pendencias-tickets.md`, TCK-0109.

Com o fluxo híbrido, a busca `1ª Resposta` do teste real consumiu 9 créditos (contra cerca de 49 do fluxo individual) e não gerou 429.

## 10. Suposições provisórias (dependem de teste real)

Já resolvido: a sintaxe de `fr_due_by` e a inexistência de `fr_escalated` como campo pesquisável (seção 2). O que segue continua sem confirmação:

- o filtro de agente usa `responder_id:{id}`; falta confirmar se a Search API aceita essa cláusula;
- o filtro de tags usa `tag:'valor'` e combina tags com `OR`; falta confirmar se a conta exige `tag`, `tags` ou outra sintaxe;
- os status personalizados “Aguardando o cliente” e “Aguardando terceiros” não têm códigos confirmados;
- filtros customizados sem chave conhecida retornam aviso em `ignoredFilters`, em vez de serem aplicados silenciosamente.
