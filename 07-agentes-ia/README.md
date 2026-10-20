# Agentes de IA

**Assunto:** Agentes de IA
**Versão do código de referência:** Projeto3 v0.8.0.7.0 (dados de mineração a portar; tela de IA ainda por decidir no Projeto4)
**Data:** 2026-09-29
**Status:** reservado no Projeto4 — referência do Projeto3 completa, escopo ainda em decisão

## Escopo
Definição de escopo antes de qualquer implementação nova no Projeto4. O que já existe no Projeto3 não é propriamente "IA" (modelo de linguagem) — é classificação de texto por regras e mineração de frases a partir de tickets reais, usada hoje só para reset de senha.

## O que existe hoje no Projeto3

### O serviço `ia.service.js` é um stub
```js
export async function sugerirRespostaMock(ticket) {
  return { modo: 'mock', sugestao: `Sugestão para ticket ${ticket?.id || 'sem id'}` };
}
```
Não há integração com nenhum provedor de IA generativa. Toda a "inteligência" real do projeto, hoje, é o classificador por regras abaixo.

### Classificador de intenção — reset de senha
`src/freshdesk/resete-senha-rules.service.js` define, em código, as listas de termos por força:

| Categoria | O que é | Exemplo |
|---|---|---|
| `identificadores_fortes` | par de termos que praticamente garante a intenção | `resetar` + `senha` |
| `identificadores_medios` | termos mais genéricos, sozinhos ou em par | `senha` isolado; `não consigo` + `entrar` |
| `identificadores_fracos` | termos de contexto, fracos isoladamente | `acesso`, `portal`, `login` |
| `invalidadores_fortes` | presença descarta a intenção | `teams`, `outlook`, `corporativo` |
| `invalidadores_fracos` | reduz a confiança sem descartar | `gmail`, `wifi`, `notebook` |

A função `avaliarReseteSenhaTexto()` roda essas listas contra o texto normalizado do ticket e devolve `status` (`validado`/`indeterminado`/`não_validado`) e `nivel` (`forte`/`medio`/`fraco`/`invalidador_forte`). Se qualquer invalidador forte aparecer, o ticket é descartado mesmo com termos fortes presentes.

Funções complementares no mesmo arquivo: `extrairCpfReseteSenha()` (regex de CPF), `extrairDataNascimentoReseteSenha()` (regex de data), `extrairPossivelUsuarioReseteSenha()` (heurística de nome próprio a partir de padrões como "senha de", "usuário:", "aluno:"). `gerarMensagemConfirmacaoReseteSenha()` monta a mensagem que seria enviada ao requerente pedindo confirmação — hoje usada só na simulação (ver `05-automacoes`).

### Base de treino já minerada
`freshdesk/db/resete-senha-treino-palavras.json` registra o resultado de rodar esse classificador sobre um lote real de tickets (filtrados pela tag interna de reset de senha): de 6.449 tickets varridos, 726 tinham a tag e foram analisados. O arquivo guarda as mesmas listas de regras usadas na mineração, contadores agregados e, por ticket, quais validadores/invalidadores bateram — sem reproduzir aqui o conteúdo de nenhum ticket individual. Esse arquivo é a base de dados que já "funciona no Projeto3", mencionada como ponto de partida para o Projeto4 (decisão #28 em `registros/decisoes.md`).

Existe ainda um arquivo mais simples e desatualizado, `src/classificacao/palavras-chave.reset-senha.json`, com uma lista plana de frases sem classificação por força — parece anterior ao `resete-senha-rules.service.js` e não é a fonte a portar.

### Onde essa mineração se conecta ao restante
`analisarReseteSenhaTicket()` (mesmo arquivo) junta classificação + CPF + data + nome numa única avaliação por ticket, usada pelo workflow de reset de senha (`05-automacoes`) e por `resete-senha-integracao-real.service.js`/`resete-senha-note.service.js` (que preparam a nota e a mensagem de confirmação a publicar no Freshdesk).

## Decisões já tomadas para o Projeto4
- a tela de download de tickets (Detectar/Baixar/Continuar/Parar) fica fora do Projeto4 — só os dados minerados são usados (`P4-0801`, parcial);
- portar a lógica de mineração por tag (fortes/médias/fracas/excludentes) para o Projeto4, sem a tela de download (`P4-0802`, pendente).

## Ainda em aberto
- se o Projeto4 vai só reaproveitar essa base já minerada (estática) ou vai rodar o classificador continuamente sobre tickets novos;
- se outras categorias além de reset de senha vão usar o mesmo padrão de classificação por regras;
- provedor, anonimização, auditoria e limites de uso, caso o Projeto4 decida futuramente incorporar IA generativa de fato (não existe hoje no Projeto3).

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0801` (parcial), `P4-0802`, e a lista de decisões futuras em aberto.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
