# Tickets Freshdesk — API, rotas e regras de negócio

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.13
**Data:** 2026-09-29
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

Regras validadas na conta real (v0.1.4.9 a v0.1.4.13):

- a consulta completa vai entre aspas duplas no parâmetro `query` e deve ser codificada na URL;
- datas em `fr_due_by` usam somente `YYYY-MM-DD` (`fr_due_by:<'2026-09-28'`, `fr_due_by:'2026-09-28'`, intervalo com `>` e `<`);
- **`fr_escalated` não é pesquisável**: aparece no JSON do ticket, mas a Search API responde `invalid_field` (`Unexpected/invalid field in request`). Não usar `fr_escalated:true` nem `fr_escalated:false`;
- a Search API devolve candidatos; para saber se o ticket já recebeu a primeira resposta é preciso consultar o detalhe com `include=stats` (seção abaixo).

### Detalhe com `stats` (usado pela busca 1ª Resposta)

```http
GET /api/v2/tickets/{ticket_id}?include=stats
```

O backend consulta cada candidato e lê `stats.first_responded_at`. Preenchido = já respondido (excluído da busca 1ª Resposta). Ver `filtros-e-buscas.md`, seção 5.2.

## 3. Rotas locais do backend

| Rota | Método | Uso |
|---|---|---|
| `/api/tickets` | GET | Busca de tickets. Filtros chegam como parâmetros repetidos (ex.: `group=...&group=...`) mais `searchType` |
| `/api/config/filters` | GET | Nomes de grupos e agentes configurados no `.env`; nunca revela a API Key. O frontend usa para preencher os seletores |
| `/api/saved-searches` | GET | Lista as buscas salvas |
| `/api/saved-searches` | POST | Salva a busca atual ou atualiza uma existente com o mesmo nome |
| `/api/saved-searches?name=...` | DELETE | Exclui uma busca salva |

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

Retry automático e fila de espera no frontend não estão implementados.

### Limite de chamadas da conta (HTTP 429)

O limite de requisições é **da conta Freshdesk**, compartilhado por tudo que a consulta. Nos testes da v0.1.4.13 ocorreu HTTP 429 por uso simultâneo de: Projeto3 com alertas ativos, Projeto4, scripts de inspeção e buscas repetidas em sequência. Após cerca de cinco segundos a busca voltou a responder.

Como a busca 1ª Resposta faz uma chamada `include=stats` por candidato, ela consome mais cota que as demais. O controle de limite ainda **não existe** (cache de `stats`, controle global entre usuários, `Retry-After`, bloqueio de buscas simultâneas, retorno parcial); está em `pendencias-tickets.md`, TCK-0105.

## 10. Suposições provisórias (dependem de teste real)

Já resolvido: a sintaxe de `fr_due_by` e a inexistência de `fr_escalated` como campo pesquisável (seção 2). O que segue continua sem confirmação:

- o filtro de agente usa `responder_id:{id}`; falta confirmar se a Search API aceita essa cláusula;
- o filtro de tags usa `tag:'valor'` e combina tags com `OR`; falta confirmar se a conta exige `tag`, `tags` ou outra sintaxe;
- os status personalizados “Aguardando o cliente” e “Aguardando terceiros” não têm códigos confirmados;
- filtros customizados sem chave conhecida retornam aviso em `ignoredFilters`, em vez de serem aplicados silenciosamente.
