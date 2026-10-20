# Tickets Freshdesk — Roteiro de validação

**Assunto:** Tickets Freshdesk
**Versão do código de referência:** v0.1.4.13
**Data:** 2026-09-29
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

**1ª Resposta** (regra validada pelo usuário na v0.1.4.13; este roteiro serve de regressão)

1. conferir grupos, status Aberto/Pendente e o filtro de prazo com Vencidos, Hoje e Amanhã;
2. ao carregar a busca salva, confirmar Vencidos e Hoje marcados e Amanhã disponível e desmarcado;
3. executar e confirmar que não há `Validation Failed` nem `invalid_field` (nenhuma consulta usa `fr_escalated`);
4. confirmar que tickets com `stats.first_responded_at` preenchido não aparecem e que a resposta traz `candidateCount` e `returnedCount` coerentes;
5. conferir que Vencidos, Hoje e Amanhã não se sobrepõem;
6. alterar um filtro e confirmar **Personalizada não salva**, sem erro de JavaScript, mantendo a verificação de primeira resposta enquanto houver filtro de prazo;
7. se ocorrer HTTP 429, aguardar alguns segundos antes de repetir (ver TCK-0105).

**Personalizada**

1. executar sem filtros e confirmar o alerta do frontend e o `warnings` do backend;
2. verificar paginação e volume retornado.

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
- erros da API, especialmente `429`, são tratados de modo previsível;
- os testes foram executados também em cenário multiusuário.
