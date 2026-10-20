# Projeto4 - ADD v002

**Tarefa concluída:** Limitador de pesquisas e tratamento visual de HTTP 429  
**Data de validação:** 2026-09-29  
**Versão final validada:** `v0.1.4.14`  
**Versão-base:** `v0.1.4.13`  
**Status:** validado pelo usuário

## 1. Objetivo concluído

Proteger a aba Tickets Freshdesk contra pesquisas repetidas em curto intervalo e melhorar o comportamento visual quando a API do Freshdesk atingir o limite de chamadas.

A tarefa foi concluída sem alterar:

- consultas do Freshdesk;
- regras de `stats.first_responded_at`;
- filtros de Vencidos, Hoje e Amanhã;
- buscas salvas;
- HTML e CSS;
- layout estrutural;
- backend.

## 2. Comportamento validado

### Durante uma pesquisa

```text
Usuário inicia uma pesquisa
  -> bloquear Carregar tickets
  -> bloquear Buscar
  -> mostrar Buscando…
  -> impedir uma segunda chamada simultânea
```

### Depois de uma pesquisa concluída com sucesso

```text
Pesquisa concluída
  -> bloquear novas pesquisas por 5 segundos
  -> mostrar contagem regressiva nos botões
  -> liberar automaticamente ao final
```

### Quando a API retorna HTTP 429

```text
HTTP 429
  -> mostrar aviso curto ao usuário
  -> bloquear novas pesquisas por no mínimo 10 segundos
  -> mostrar contagem regressiva
  -> respeitar retryAfter quando o valor informado for maior
  -> não iniciar novas chamadas durante o bloqueio
```

## 3. Regras implementadas

```text
Intervalo normal após pesquisa: 5 segundos
Bloqueio mínimo após HTTP 429: 10 segundos
Bloqueio efetivo no HTTP 429: maior valor entre 10 segundos e retryAfter
Pesquisas simultâneas pela mesma tela: não permitidas
Liberação dos botões: automática
```

## 4. Arquivo de código alterado

```text
src/frontend/js/principal.js
```

## 5. Documentação da versão

O patch inaugurou o novo destino permanente dos documentos por versão:

```text
docs/_updates/UPDATE_v0.1.4.14.md
docs/_flows/FLOW_v0.1.4.14.md
```

Estrutura interna validada do pacote:

```text
Projeto4_patch_v0.1.4.14_2026-09-29_limitador_pesquisas_api.zip
└── v0.1.4.14/
    ├── apply_patch_v0.1.4.14.mjs
    ├── manifest.json
    ├── README_PATCH.md
    ├── docs/
    │   ├── UPDATE_v0.1.4.14.md
    │   └── FLOW_v0.1.4.14.md
    └── files/
        └── src/frontend/js/principal.js
```

## 6. Backup

O aplicador cria backup em:

```text
_backup_patch/v0.1.4.14/
```

Se já existirem documentos diferentes com o mesmo nome, o aplicador também preserva suas versões anteriores dentro do backup da versão.

## 7. Validações realizadas

- o patch foi aplicado sobre a base atual da `v0.1.4.13`;
- os botões ficam bloqueados enquanto a pesquisa está ativa;
- os botões mostram `Buscando…` durante a chamada;
- após o retorno, a contagem de 5 segundos funciona;
- novas pesquisas ficam impedidas durante a contagem;
- os botões são liberados automaticamente;
- a busca `Abertos Portal N1` permaneceu funcional;
- a busca `1ª Resposta` permaneceu funcional;
- os documentos UPDATE e FLOW foram direcionados às novas pastas permanentes;
- o usuário confirmou que o ajuste funcionou.

## 8. Limites desta conclusão

Este ADD conclui a proteção local da interface contra chamadas repetidas. Ele não representa ainda um controle global de consumo da API para vários usuários ou vários projetos.

Continuam como pendências futuras, especialmente antes do teste multiusuário:

- cache temporário de `stats` por ticket;
- reutilização de detalhes já carregados;
- controle centralizado das chamadas de todos os usuários;
- coordenação com o Projeto3 enquanto ele consultar a mesma conta;
- estratégia de fila no backend;
- observação dos cabeçalhos reais de limite da conta;
- tratamento de resultados parciais quando o limite for atingido durante uma operação composta.

## 9. Critério de conclusão

A tarefa foi considerada concluída porque:

- evita pesquisas simultâneas na mesma interface;
- aplica intervalo mínimo de 5 segundos depois de uma pesquisa válida;
- possui tratamento visual de HTTP 429;
- aplica bloqueio mínimo de 10 segundos em caso de limite;
- respeita `retryAfter` quando maior;
- libera os controles automaticamente;
- não causou regressão nas buscas já validadas;
- foi validada pelo usuário.

## 10. Onde incorporar no Manual Técnico

Atualizar, quando necessário:

```text
docs/03-tickets-freshdesk/README.md
docs/03-tickets-freshdesk/validacao.md
docs/03-tickets-freshdesk/pendencias-tickets.md
docs/09-logs-auditoria/README.md
docs/11-padrao-de-patch/README.md
docs/registros/changelog.md
docs/registros/decisoes.md
docs/registros/erros-e-licoes.md
docs/registros/pendencias.md
docs/00-indice/PROGRESSO-COMPILACAO.md
onde_paramos.md
```

## 11. Próxima tarefa

A próxima pequena etapa planejada é:

```text
Reduzir a poluição visual da aba Tickets
  -> mover explicações técnicas para um botão de informação (i) ou (?)
  -> manter visíveis apenas mensagens operacionais importantes
  -> preservar o layout e os estilos já validados
```
