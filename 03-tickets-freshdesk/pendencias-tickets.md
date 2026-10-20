# Pendências consolidadas - Tickets Freshdesk

**Assunto:** Tickets Freshdesk  
**Versão de referência:** Projeto4 `v0.1.4.35` instalada; última versão integralmente validada `v0.1.4.15` (ver `onde_paramos.md`)  
**Data:** 2026-10-20  
**Status:** vigente  
**Próxima tarefa funcional:** ver `onde_paramos.md`, seção 5 (começa pelos ajustes da aba Logs); nesta lista: TCK-0112 (validar o fluxo híbrido), TCK-0110, TCK-0111 e TCK-0102

## 1. Objetivo

Manter uma lista única, curta e verificável das pendências da aba Tickets Freshdesk.

Esta lista substitui pendências genéricas antigas como "criar a tela de tickets", "criar filtros" ou "criar buscas salvas", pois essas partes já possuem implementação no Projeto4.

Cada item deve distinguir:

- o que já foi implementado;
- o que já foi validado;
- o que aguarda teste real;
- o que ainda precisa ser desenvolvido;
- o que precisará ser adaptado para multiusuário.

## 2. Base que deve ser preservada

Antes de qualquer novo patch, coletar os arquivos completos atuais da versão instalada (`v0.1.4.35` em 2026-10-20). Não partir de arquivos enviados antes dessa versão. Além dos arquivos abaixo, incluir os que o patch for alterar entre `src/api/freshdesk/`, `src/services/tickets/`, `src/lib/env.js` e o serviço de logs (caminhos conforme o UPDATE/FLOW da `v0.1.4.29`):

```text
src/frontend/principal.html
src/frontend/js/principal.js
src/frontend/css/principal.css
src/lib/freshdesk.js
src/lib/savedSearches.js
src/server.js
data/saved-searches.json
```

O `principal.html` instalado contém a correção manual posterior à `v0.1.4.8` (remoção do `\n` literal) e as alterações das versões seguintes. Não reconstruir esse arquivo a partir do pacote original de nenhuma versão.

## 3. Itens já confirmados

Os itens desta seção não devem voltar ao backlog como tarefas genéricas.

### TCK-0001 - Estado inicial simplificado

**Estado:** validado  
**Resultado:** ao abrir Tickets sem resultados carregados, existe apenas um botão de carregamento e o estado `Nenhum ticket carregado`.

### TCK-0002 - Nome da busca Abertos Portal N1

**Estado:** confirmado  
**Resultado:** o nome visual é `Abertos Portal N1`; o identificador interno permanece `padrao_n1`.

### TCK-0003 - Valores especiais de filtros

**Estado:** confirmado

```text
Qualquer      = ausência de filtro; não selecionar, enviar ou salvar
Nenhum        = opção real; representa valor nulo
Não atribuído = opção real; representa grupo nulo
```

### TCK-0004 - JSON como fonte das buscas salvas

**Estado:** validado no comportamento de leitura  
**Resultado:** o backend respeita o conteúdo e a ordem de `data/saved-searches.json`; uma busca apagada manualmente não é recriada ao reiniciar.

### TCK-0005 - Correções locais de frontend

**Estado:** corrigido

- erro `$(...).forEach is not a function` (na `v0.1.4.8`, em `markSearchAsUnsavedCustom()`, e de novo na `v0.1.4.13`, na atualização dos botões de tipo de busca; ambas trocaram `$()` por `$$()`);
- requisição `/favicon.ico` com 404;
- texto literal `\n` no topo do HTML.

### TCK-0101 - Regra de 1ª Resposta

**Estado:** validado pelo usuário em 2026-09-28 (`v0.1.4.13`)  
**Documento conclusivo:** `adds-validados/Projeto4_ADD_v001_2026-09-28_busca_primeira_resposta_validada.md`

Resultado: `fr_escalated` deixou de ser usado (a Search API o rejeita); `stats.first_responded_at` é o indicador de ticket já respondido; Vencidos, Hoje e Amanhã são classificados pelo backend com `fr_due_by` completo em `America/Sao_Paulo`; Vencidos e Hoje vêm marcados por padrão e Amanhã fica disponível e desmarcado. Regra completa em `filtros-e-buscas.md`, seção 5.2.

## 4. Pendências prioritárias

### TCK-0102 - Validar Nível N1 ou Nenhum

**Prioridade:** alta  
**Estado:** implementado, aguardando validação real

Resultado esperado:

```text
(
  cf_nvel_de_atendimento:'N1'
  OR
  cf_nvel_de_atendimento:null
)
```

#### Validações obrigatórias

- confirmar a chave real do campo personalizado;
- confirmar o valor literal aceito pela conta, como `N1` ou outra representação;
- inspecionar a consulta enviada;
- comparar o resultado com tickets conhecidos;
- manter `Nenhum` como opção real, não como placeholder.

### TCK-0103 - Validar a desvinculação da busca salva

**Prioridade:** alta  
**Estado:** implementado, aguardando roteiro completo

Fluxo esperado:

```text
carregar busca salva
  -> preservar nome e searchType
alterar qualquer filtro
  -> searchType=personalizada
  -> rótulo=Personalizada não salva
  -> remover seleção da busca salva
voltar manualmente aos filtros anteriores
  -> continuar Personalizada não salva
recarregar explicitamente a busca
  -> restaurar nome e searchType originais
```

#### Critério de aceite

Nenhuma alteração manual deve continuar enviando silenciosamente o `searchType` da busca fixa anterior.

### TCK-0104 - Melhorar erros de busca

**Prioridade:** alta  
**Estado:** parcialmente implementado

O frontend deve apresentar, quando seguro:

- mensagem do backend;
- filtro ou cláusula inválida;
- consulta aplicada;
- resposta resumida do Freshdesk;
- `retry-after` em HTTP 429;
- distinção entre erro de configuração, validação, rede e limite da API.

Segredos, cookies, tokens e API keys não podem aparecer.

### TCK-0105 - Controlar o limite de chamadas da API (HTTP 429)

**Prioridade:** imediata; obrigatória antes do teste multiusuário  
**Estado:** parcial. **Frontend concluído e validado** na `v0.1.4.14` (ADD v002): sem pesquisa simultânea, 5 s entre pesquisas, bloqueio de no mínimo 10 s após 429 (ou `retryAfter` maior) com contagem regressiva. **Cache de stats e governador no backend implementados** (`v0.1.4.25` a `v0.1.4.35`), aguardando validação continuada. **Pendente:** proteção preventiva por crédito restante e controle entre usuários (TCK-0109).  
**Origem:** HTTP 429 observado nos testes da `v0.1.4.13` (ver `api-e-regras.md`, seção 9)

Comportamento desejado:

```text
Usuário inicia uma pesquisa
  -> bloquear nova pesquisa por 5 segundos
  -> mostrar contagem ou indicação discreta
  -> liberar o botão depois do intervalo

HTTP 429
  -> aviso claro e curto
  -> impedir novas pesquisas por 10 segundos, com contagem regressiva
  -> respeitar Retry-After quando o Freshdesk informar valor maior
  -> não disparar novas chamadas durante o bloqueio
```

Avaliar também:

- impedir duplo clique em Buscar e manter uma pesquisa ativa por usuário;
- cache temporário de `stats` por ticket e reaproveitamento de detalhes já carregados;
- controle global de chamadas entre usuários;
- retorno parcial ou mensagem controlada em caso de 429;
- convivência com o Projeto3, que consulta a mesma conta enquanto continuar ativo.

### TCK-0106 - Reduzir informações permanentes na tela e criar botão de ajuda

**Prioridade:** média  
**Estado:** concluído e validado na `v0.1.4.15` (ADD v003)

Mover explicações longas e detalhes técnicos para um botão discreto `(i)` (ou `(?)`, se ficar melhor), que abre modal, popover ou painel pequeno com: explicação da busca ativa, significado dos filtros especiais, avisos de limite da API, detalhes de diagnóstico e o significado de Vencidos, Hoje e Amanhã. Mensagens de erro importantes continuam visíveis. Preservar o layout e o CSS validados; não criar blocos grandes permanentes; não duplicar informação.

### TCK-0107 - Exibir o nome da busca ativa uma única vez

**Prioridade:** média  
**Estado:** concluído e validado na `v0.1.4.15` (ADD v003): o nome permanece só no seletor principal

O nome da busca aparece mais de uma vez em alguns casos. Revisar título da busca, indicador do tipo, nome dentro dos filtros, seleção em buscas salvas e feedback abaixo dos controles; manter só o local mais claro. Os demais só permanecem se mostrarem informação diferente.

### TCK-0108 - Botão Voltar do ticket aberto

**Prioridade:** média  
**Estado:** pendente (depois da validação do detalhe, TCK-0201 e TCK-0202)

`← Voltar`, com seta amarela apontando para a esquerda, antes do texto ou acima dele conforme ficar melhor no layout. Preservar o padrão visual geral e validar no desktop antes de adaptar ao mobile. Não alterar estilos não solicitados.

### TCK-0109 - Proteção preventiva global da API

**Prioridade:** alta; antes do teste multiusuário  
**Estado:** pendente  
**Origem:** ADD v002 (seção 8) e ADD v004 (seções 12 e 13)

- não iniciar uma chamada quando o crédito restante (`X-RateLimit-Remaining`) não comportar o custo da próxima operação (Search API 1 crédito; página com stats 2 créditos);
- reservar um mínimo de créditos e relacionar o restante com os candidatos pendentes;
- processamento progressivo e reaproveitamento do que foi obtido antes de um bloqueio;
- resultado parcial apresentado como parcial no frontend ("X candidatos analisados, Y pendentes"), nunca como completo;
- controle centralizado das chamadas de todos os usuários e coordenação com o Projeto3, que consulta a mesma conta;
- reconciliar o bloqueio de 10 s (frontend) com o de 60 s (backend) — P4-1209.

### TCK-0110 - Busca salva N2-Abertos vira Personalizada sozinha

**Prioridade:** alta  
**Estado:** pendente (erro observado em 2026-09-29)

Selecionar a busca salva `N2-Abertos` muda automaticamente para `Personalizada (usar filtros)`. Investigar a identificação entre busca salva e personalizada, a alteração de `currentSearchType` e `currentSearchLabel`, o preenchimento dos filtros pelo JSON, eventos que tratam o preenchimento programático como edição manual e a sincronização do seletor.

Critério de aceite: carregar `N2-Abertos` mantém `N2-Abertos` como busca ativa; só uma alteração manual de filtro muda para Personalizada (TCK-0103).

### TCK-0111 - Pesquisa por número do ticket

**Prioridade:** média  
**Estado:** pendente

Modos: ticket único (`8071`), lista (`8071, 8066, 8063`) e intervalo (`8000 até 8071`). A listagem não tem filtro documentado por vários IDs e `id=1,2,3` é ignorado: não presumir esse parâmetro. Estratégias a avaliar: `/tickets/{id}` para ticket único; mineração por intervalo de datas; Search API para reduzir candidatos; cruzamento local por ID; listagem ordenada e paginada; datas de criação do mais antigo e do mais novo como limites. Registrar nos logs critério, intervalo, datas calculadas, páginas, IDs encontrados e ausentes e créditos.

### TCK-0112 - Validar o fluxo híbrido e o cache persistente

**Prioridade:** alta  
**Estado:** implementado (`v0.1.4.33` a `v0.1.4.35`); arquitetura validada no ADD v004; uso continuado aguardando validação  
**Origem:** ADD v004, seção 15

- segunda execução usando o cache (cache hits) e comportamento após reiniciar o servidor;
- busca personalizada com `Amanhã` depois da `v0.1.4.35` (sem `ticket_details_stats` nos logs);
- distribuição por grupo e auditoria dos grupos nos logs (inclui `Não atribuído`);
- resultado funcional esperado de Vencidos, Hoje e Amanhã: fuso, limite entre dias, tickets que entram na Search API e são rejeitados no classificador local;
- apresentação de resultado parcial.

### TCK-0113 - Ajustes visuais gerais e do botão `i`

**Prioridade:** média  
**Estado:** pendente

Reduzir textos longos em todas as abas (tooltip, botão de informação, texto recolhível); barra de botões fixa na rolagem; remover a rolagem horizontal do `body` corrigindo a origem do transbordamento, sem `overflow-x: hidden` como atalho; barras de rolagem discretas em cinza (página, listas, console de logs, janelas, painéis); brilho branco suave atrás do painel aberto pelo `i`, sem exagero e legível no tema escuro. Preservar o layout validado.

## 5. Detalhe e operação do ticket

### TCK-0201 - Carregar detalhe completo sob demanda

**Prioridade:** alta  
**Estado:** pendente

Implementar ou confirmar o contrato de:

```text
GET /tickets/{id}?include=conversations
```

Validar:

- descrição;
- requester;
- conversas;
- anexos;
- `stats`;
- campos personalizados;
- estados de carregamento e erro.

### TCK-0202 - Preservar o renderizador visual

**Prioridade:** bloqueadora para considerar o detalhe concluído  
**Estado:** regressão conhecida a prevenir

O detalhe não pode ser considerado validado apenas porque o backend retorna JSON correto. O frontend deve renderizar visualmente o ticket completo e nunca regredir para JSON bruto.

### TCK-0203 - Conectar Resposta

**Prioridade:** alta  
**Estado:** pendente

Conectar o editor à operação real de resposta, com envio explícito, carregamento, erro, confirmação e auditoria.

Conteúdo carregado no editor, inclusive resposta pronta, não deve ser enviado automaticamente.

### TCK-0204 - Conectar Anotação

**Prioridade:** alta  
**Estado:** pendente

Conectar a aba ativa à criação de anotação, mantendo separação clara entre Resposta e Anotação.

### TCK-0205 - Anexos e download sob demanda

**Prioridade:** alta  
**Estado:** pendente de migração e validação

O download deve funcionar na sessão atual sem solicitar autorização local inesperada, sem expor caminhos internos e com autoria registrada na auditoria.

## 6. Filtros e mapeamentos

### TCK-0301 - Mapear cinco campos personalizados

**Prioridade:** média-alta  
**Estado:** pendente

Campos:

- É usuário do Portal;
- Tipo de usuário;
- Departamento regional;
- Unidade escolar;
- Tipo.

Após confirmar as chaves e valores reais:

- implementar as cláusulas;
- remover os campos da lista de não mapeados;
- validar resultados na conta real;
- desabilitar ou sinalizar claramente qualquer campo ainda sem efeito.

### TCK-0302 - Confirmar agente, tags e status personalizados

**Prioridade:** média-alta  
**Estado:** pendente de validação

Confirmar:

- `responder_id` para pesquisa por agente;
- sintaxe `tag:` ou `tags:`;
- códigos de `Aguardando o cliente`;
- códigos de `Aguardando terceiros`;
- ID real do PSGE;
- centralização dos IDs e aliases na configuração.

### TCK-0303 - Avaliar grupos e agentes dinâmicos

**Prioridade:** média  
**Estado:** decisão futura

Hoje a interface recebe mapas configurados. Avaliar se haverá sincronização com `/groups` e `/agents`, incluindo cache, aliases e comportamento quando um item for desativado no Freshdesk.

## 7. Buscas salvas e multiusuário

### TCK-0401 - Definir propriedade das buscas salvas

**Prioridade:** bloqueadora antes do teste operacional multiusuário  
**Estado:** pendente de decisão e implementação

Definir se cada busca será:

```text
individual do usuário
compartilhada com todos
compartilhada somente mediante marcação explícita
```

Não assumir que o arquivo JSON compartilhado atual é adequado para multiusuário.

### TCK-0402 - Proteger gravações simultâneas

**Prioridade:** bloqueadora para multiusuário  
**Estado:** pendente

Avaliar:

- escrita atômica;
- lock ou fila de gravação;
- prevenção de sobrescrita;
- recuperação após interrupção;
- necessidade futura de banco de dados.

### TCK-0403 - Auditoria das buscas salvas

**Prioridade:** alta  
**Estado:** pendente

Registrar criação, atualização e exclusão com:

- identidade principal do Cloudflare Access;
- data e hora;
- nome ou identificador da busca;
- resultado;
- sem registrar credenciais ou dados desnecessários.

## 8. Teste multiusuário da aba Tickets

**Momento:** depois de concluir e validar as pendências funcionais imediatas da aba Tickets.

Usar duas sessões independentes inicialmente e ampliar até cinco.

### Cenários mínimos

```text
Usuário A abre o ticket 1
Usuário B abre o ticket 2
  -> um ticket não substitui o outro

Usuário A usa Contexto do Ticket
Usuário B usa Contexto Geral
  -> abas e estados permanecem isolados

Usuário A monta filtros
Usuário B monta filtros diferentes
  -> parâmetros e resultados não se misturam

Usuário A escreve um rascunho
Usuário B escreve outro
  -> conteúdo não se mistura

Usuário A ativa o Portal
Usuário B permanece sem Portal
  -> somente A acessa funções dependentes do Portal

Usuário A executa download ou ação
  -> auditoria identifica corretamente A
```

### Critério de aceite

Nenhum usuário recebe ticket aberto, filtros, rascunho, cookie do Portal, resultado, intervenção ou autoria pertencente a outro usuário.

## 9. Ordem recomendada de execução

```text
1. ajustes da aba Logs (P4-1005, P4-1006, P4-1007 em registros/pendencias)
2. TCK-0112 - validar o fluxo híbrido e o cache persistente
3. TCK-0110 - N2-Abertos vira Personalizada
4. TCK-0113 - ajustes visuais gerais e do botão i
5. TCK-0111 - pesquisa por número do ticket
6. TCK-0201 e TCK-0202 - detalhe completo e renderizador visual (validar o ticket aberto)
7. TCK-0108 - botão Voltar
8. regressão de Abertos Portal N1 e 1ª Resposta
9. TCK-0102 - validar N1 ou Nenhum
10. TCK-0103 - validar desvinculação de busca salva
11. TCK-0104 - melhorar erros de busca
12. TCK-0203 e TCK-0204 - Resposta e Anotação
13. TCK-0205 - anexos
14. TCK-0301 a TCK-0303 - mapeamentos
15. TCK-0109 - proteção preventiva global da API (antes do teste multiusuário)
16. TCK-0401 a TCK-0403 - buscas salvas multiusuário
17. executar bateria multiusuário da aba Tickets
```

(TCK-0101, TCK-0106 e TCK-0107 concluídas, e a parte de frontend de TCK-0105, saíram da fila. A ordem segue `onde_paramos.md`, seção 5.)

## 10. Regra para fechamento

Ao concluir cada tarefa:

1. gerar UPDATE e FLOW da versão;
2. executar o roteiro de validação;
3. registrar resultado e regressões;
4. gerar ADD somente após validação do usuário;
5. atualizar este arquivo;
6. atualizar `docs/03-tickets-freshdesk/README.md`;
7. atualizar registros e `docs/onde_paramos.md`.
