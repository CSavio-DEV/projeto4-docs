# Projeto4 - ADD v003

**Tarefa concluída:** Redução da poluição visual e painel de informações da aba Tickets  
**Data de validação:** 2026-09-29  
**Versão final validada:** `v0.1.4.15`  
**Versão-base:** `v0.1.4.14`  
**Status:** validado pelo usuário

## 1. Objetivo concluído

Reduzir a quantidade de textos explicativos e informações técnicas exibidas permanentemente na aba Tickets Freshdesk, preservando as mensagens operacionais importantes e concentrando as orientações em um painel discreto de informações.

## 2. Resultado visual validado

Foi adicionado um botão circular de informações:

```text
i
```

O botão fica próximo ao título `Tickets Freshdesk` e abre um painel compacto com orientações sobre:

- funcionamento geral da busca;
- significado de Vencidos;
- significado de Hoje;
- significado de Amanhã;
- intervalo entre pesquisas;
- limite da API;
- abertura de um ticket da lista.

## 3. Formas de fechar o painel

O painel pode ser fechado por:

- botão `×`;
- clique fora do painel;
- tecla `Esc`.

O painel também possui comportamento adaptado para telas menores.

## 4. Informações removidas da área permanente

Foram removidos da visualização principal:

- texto explicativo longo abaixo do título da página;
- indicação duplicada de `Busca atual` no cabeçalho;
- indicação duplicada de `Busca atual` dentro dos filtros;
- instrução permanente sobre clicar em um ticket;
- warning técnico sobre `fr_due_by` e `America/Sao_Paulo`.

O nome da busca ativa permanece visível somente no seletor principal.

## 5. Informações que continuam visíveis

Permanecem visíveis quando necessárias:

- mensagem de carregamento;
- erros de consulta;
- falhas de conexão;
- aviso de HTTP 429;
- contagem de bloqueio entre pesquisas;
- confirmação ou erro de busca salva;
- mensagens que exigem ação do usuário.

## 6. Tratamento da informação técnica

O texto técnico abaixo deixou de ser enviado ao frontend:

```text
Tipo de busca "1a Resposta" usa fr_due_by por dia no fuso America/Sao_Paulo,
limitado a tickets Abertos ou Pendentes e aos mesmos grupos de Abertos Portal N1.
```

A regra permanece documentada internamente em `src/lib/freshdesk.js`, sem ocupar espaço na interface.

## 7. Mensagem de carregamento

A mensagem:

```text
Buscando tickets no Freshdesk…
```

continua aparecendo durante a pesquisa e é ocultada quando a chamada termina.

## 8. Arquivos relacionados

```text
src/frontend/principal.html
src/frontend/js/principal.js
src/frontend/css/principal.css
src/lib/freshdesk.js
```

## 9. Documentação da versão

```text
docs/_updates/UPDATE_v0.1.4.15.md
docs/_flows/FLOW_v0.1.4.15.md
```

Pacote relacionado:

```text
Projeto4_patch_v0.1.4.15_2026-09-29_informacoes_tickets.zip
└── v0.1.4.15/
```

## 10. Validações realizadas

- botão `i` exibido corretamente;
- painel aberto pelo botão de informações;
- painel fechado pelo `×`;
- painel fechado por clique externo;
- painel fechado com `Esc`;
- textos permanentes reduzidos;
- nome da busca exibido somente no seletor principal;
- warning técnico retirado do frontend;
- mensagens operacionais preservadas;
- buscas `Abertos Portal N1` e `1ª Resposta` preservadas;
- busca personalizada preservada;
- usuário confirmou a validação da versão.

## 11. Itens não alterados

Esta tarefa não alterou:

- regras de consulta do Freshdesk;
- filtros de Vencidos, Hoje e Amanhã;
- `stats.first_responded_at`;
- limitador de pesquisas da v0.1.4.14;
- buscas salvas;
- detalhe do ticket;
- botões do ticket aberto;
- botão Voltar.

## 12. Critério de conclusão

A tarefa foi considerada concluída porque:

- a tela ficou mais limpa;
- as explicações continuam acessíveis;
- informações técnicas deixaram de ocupar a área principal;
- erros e mensagens operacionais continuam visíveis;
- o comportamento foi validado pelo usuário;
- não houve regressão funcional nas buscas já validadas.

## 13. Onde incorporar no Manual Técnico

Atualizar, quando necessário:

```text
docs/03-tickets-freshdesk/README.md
docs/03-tickets-freshdesk/validacao.md
docs/10-layout-frontend/README.md
docs/registros/changelog.md
docs/registros/decisoes.md
docs/registros/pendencias.md
docs/00-indice/PROGRESSO-COMPILACAO.md
onde_paramos.md
```

## 14. Próxima tarefa

A próxima etapa planejada é revisar a abertura do ticket e sua interface:

```text
- validar o detalhe visual do ticket;
- impedir regressão para JSON bruto;
- identificar botões funcionais e apenas visuais;
- revisar informações repetidas;
- preparar o botão ← Voltar com seta amarela.
```
