# Onde paramos - Projeto4

**Arquivo único de estado.** Sobrescreva este arquivo a cada sessão; não crie `onde_paramos_01`, `_02` etc. O histórico fica em `registros/changelog.md`, em `_updates/`, `_flows/` e em `adds-validados/`.

**Última atualização:** 2026-10-20 (consolida o estado de 29/09/2026)
**Projeto:** `parte4_cloudflare`  |  **Host:** `127.0.0.1`  |  **Porta:** `65004`
**Entrada:** `src/frontend/principal.html`  |  **Servidor:** Node.js  |  **Publicação:** Cloudflare Tunnel + Cloudflare Access

## 1. Versões

```text
Última versão integralmente validada: v0.1.4.15   (ADD v001 a v003: .13, .14, .15)
Base instalada mais recente:          v0.1.4.35   (ADD v004 valida o fluxo híbrido como nova base,
                                                   com validações ainda abertas: ver seção 4)
Não aplicadas:                        v0.1.4.17 (base divergente, nada alterado)
                                      v0.1.4.19 (regra mal interpretada; substituída pela .20)
Experimentais, não validadas:         v0.1.4.23, v0.1.4.24
Sem registro em nenhum documento:     v0.1.4.28
Ciclo v0.1.4.16 a v0.1.4.27:          não validado como ciclo (UPDATE/FLOW existem; sem ADD)
```

**A confirmar com o usuário:** se a v0.1.4.35 deve ser tratada como a versão validada atual. Enquanto não houver confirmação, tratar a **v0.1.4.15** como última validada e a **v0.1.4.35** como instalada.

Todo novo patch parte dos **arquivos completos atualmente instalados** (v0.1.4.35), nunca de arquivos anteriores. O `principal.html` instalado contém correções manuais posteriores à v0.1.4.8 (remoção de `\n` literal) e as alterações seguintes; não reconstruí-lo de nenhum pacote.

## 2. Situação em poucas linhas

- Busca `1ª Resposta` (Vencidos, Hoje, Amanhã) usa o **fluxo híbrido**: Search API para candidatos, listagem com `include=stats` em páginas, cruzamento por ID, cache persistente e logs técnicos. Validado em teste real (26 candidatos, 9 créditos, sem 429), com refinamentos pendentes.
- Aba **Logs** ativa (modos Desligado, Resumido, Completo), com ajustes visuais e de dados pendentes.
- Aba Tickets: limitador de pesquisas (ADD v002) e painel de informações `i` (ADD v003) validados.
- Portal, Automações, Configurações, IA, Dashboard e auditoria **ainda não migrados**: existe só a referência do Projeto3.
- Sessão principal e sessão do Portal (24 h): decidida, **não implementada** (P4-0207 a P4-0209).

## 3. Regras permanentes (resumo; detalhes nos arquivos indicados)

1. Implementado não é validado; ADD só depois da validação do usuário (`00-indice/CONVENCOES.md`).
2. Um objetivo por patch; layout validado não muda sem pedido específico (`11-padrao-de-patch`, `10-layout-frontend`).
3. Versão nunca é reutilizada; correção usa o próximo número (`11-padrao-de-patch`).
4. Sem credenciais, chaves, cookies, e-mails reais ou dados reais em documentos, patches, logs ou exemplos; usar placeholders (`02-acesso-seguranca`, `CONVENCOES` seção 5).
5. Identidade principal = Cloudflare Access; login do Portal é capacidade opcional da mesma sessão (decisões 38 a 42).
6. `data/saved-searches.json` é a única fonte das buscas salvas (decisão 24).
7. Detalhe do ticket sempre visual, nunca JSON bruto (decisão 25).
8. `stats.first_responded_at == null` é o critério de primeira resposta; `fr_escalated` não é usado (decisão 43).
9. Toda busca com `frDueBy` usa o fluxo híbrido e o cache persistente; nunca consulta individual de stats por ticket (ADD v004).
10. Não alterar o fluxo híbrido sem analisar os logs e o código completo atual.
11. Resultado parcial nunca é apresentado como completo (`complete: false`).
12. Nome do arquivo diário de log usa a data local do fuso configurado, nunca `toISOString()`.

## 4. Estado funcional

**Validado:** estado inicial simplificado de Tickets; Abertos Portal N1 (`padrao_n1`); valores `Qualquer`, `Nenhum`, `Não atribuído`; JSON como fonte única das buscas salvas; busca 1ª Resposta com Vencidos, Hoje e Amanhã (v0.1.4.13, ADD v001); limitador de 5 s entre pesquisas e bloqueio visual após HTTP 429 (v0.1.4.14, ADD v002); painel `i` e redução de textos (v0.1.4.15, ADD v003).

**Implementado, aguardando validação:** seletor com até 5 buscas salvas e `Personalizada (não salva)` sempre no fim, nome reservado, limpeza de filtros e aviso de busca vazia (v0.1.4.16 a .20); cache, deduplicação, contexto e governador de API no backend (v0.1.4.25 a .27); tratamento de 429 sem derrubar o servidor (v0.1.4.27); aba Logs (v0.1.4.29, .30); Aberto e Pendente explícitos na busca fixa (v0.1.4.31); fluxo híbrido, cache persistente e aplicação a toda busca com `frDueBy` (v0.1.4.33 a .35); nível `N1 OR null` (TCK-0102).

**Ainda sem validação continuada (ADD v004):** cache em segunda execução e após reinício; busca personalizada com Amanhã depois da v0.1.4.35; auditoria de grupos nos logs; resultado funcional esperado de Vencidos, Hoje e Amanhã; apresentação de resultado parcial; proteção preventiva por crédito restante.

## 5. Próximas tarefas (ordem)

Uma tarefa por patch. Antes de cada uma, pedir os arquivos completos atuais.

IDs em `registros/pendencias.md` (P4-xxxx) e `03-tickets-freshdesk/pendencias-tickets.md` (TCK-xxxx).

1. **Logs: data do arquivo diário (P4-1005).** Em 29/09 à noite o sistema criou o arquivo com data 30/09 (provável uso de `toISOString()`, UTC). Usar a data local de `PROJETO4_LOG_TIMEZONE`.
2. **Logs: aparência e rolagem (P4-1006).** Cores discretas para DEBUG, INFO, WARNING, ERROR, SYSTEM, TICKETS, FRESHDESK, CACHE, RATE-LIMIT; não rolar para o fim automaticamente, só se o usuário já estiver perto do rodapé; colunas legíveis (horário, nível, escopo, operação, mensagem, detalhes); sem rolagem horizontal no documento inteiro.
3. **Logs: mais detalhe das buscas Freshdesk (P4-1007).** Tipo e rótulo da busca, nome da busca salva, filtros, grupos com nome e ID, status com código, prazos, query resumida, página, candidatos, distribuição por grupo e status, menor e maior `updated_at`, `updated_since` e margem, páginas de stats, IDs encontrados e ausentes, cache hits e misses, classificação, motivo de descarte, créditos reais, cabeçalhos `X-RateLimit-*` e `Retry-After`, `complete`, duração. Resumo final usa `apiUnitsConsumed` real (remover a estimativa antiga).
4. **Validar** cache persistente (segunda execução e reinício) e a busca com Amanhã após a v0.1.4.35 (TCK-0112).
5. **Corrigir N2-Abertos (TCK-0110):** selecionar a busca salva não pode mudar para "Personalizada (usar filtros)". Preencher filtros por código não é edição manual; só alteração do usuário muda para Personalizada.
6. **Visual geral (TCK-0113):** reduzir textos (tooltip, botão de informação, texto recolhível); barra de botões fixa; remover rolagem horizontal do `body` corrigindo a origem; scrollbars discretas em cinza.
7. **Pesquisa por número do ticket (TCK-0111)** (único, lista, intervalo). Não presumir que `id=1,2,3` funcione. Registrar nos logs critério, intervalo, datas, páginas, IDs achados e ausentes, créditos.
8. **Ticket aberto:** validar o detalhe visual (TCK-0201, TCK-0202); botão `← Voltar` com seta amarela (TCK-0108); depois conversas, interações, Resposta, Anotação e anexos sob demanda (TCK-0203 a TCK-0205).
9. **Concluir filtros:** revisar nomes, IDs e valores, combinações AND/OR, filtros múltiplos, query nos logs (TCK-0102, TCK-0301 a TCK-0303).
10. **Proteção preventiva global (TCK-0109)** por crédito restante e resultado parcial visível no frontend.
11. Depois da aba Tickets: teste multiusuário (2 sessões, até 5), sessão do Portal (P4-0207 a P4-0209) e migração do Portal. Lista completa em `registros/pendencias.md` e `03-tickets-freshdesk/pendencias-tickets.md`.

## 6. Conflitos e pontos abertos (decidir antes de mexer)

| Ponto | Situação |
|---|---|
| Bloqueio após HTTP 429 | frontend: no mínimo 10 s (ADD v002, decisão 45); backend: no mínimo 60 s e autorregulação a 10 s quando restam 5 créditos ou menos (UPDATE v0.1.4.25/.26). São duas camadas; confirmar nos UPDATE/FLOW os valores vigentes e documentar |
| Fuso | SLA em `America/Sao_Paulo`; logs em `PROJETO4_LOG_TIMEZONE=America/Fortaleza`. Decidir um fuso único ou registrar o motivo de serem dois |
| Intervalo mínimo 0 ms | funcionou uma vez e voltou a dar 429; não considerar seguro |
| Referência dos documentos | `registros/*` e `03-tickets-freshdesk/pendencias-tickets.md` já incorporam v0.1.4.14 a .35 (2026-10-20, P4-1212). Pendente (P4-1213): README, `api-e-regras`, `filtros-e-buscas`, `validacao` e `painel-contexto` de `03-tickets-freshdesk`, e `10-layout-frontend`, que ainda declaram v0.1.4.13 ou v0.1.4.8 |
| `Projeto4_padrao_ADD_arquivos.md` | não existe (P4-1207); usar o ADD v002/v003 como modelo |

## 7. Armadilhas conhecidas

- Search API: `per_page` dá HTTP 400; `fr_escalated` dá `invalid_field`; datas em `YYYY-MM-DD`.
- `GET /tickets?id=1,2,3` é ignorado pela listagem.
- Custo observado: Search 1 crédito; detalhe individual com stats 2; página de listagem com stats 2. O limite é da conta, compartilhado com o Projeto3.
- `$` devolve um elemento e `$$` uma coleção; conferir antes de `.forEach`.
- Alterar `<head>` exige conferir `view-source` e o DOM.
- Não gerar ADD para o ciclo v0.1.4.16 a .27 isoladamente.

## 8. Arquivos para enviar na próxima sessão

```text
este onde_paramos.md
ADD v004 (adds-validados/)
código completo instalado da v0.1.4.35 (arquivos que a tarefa vai alterar)
arquivo de log mais recente (sem credenciais)
.env.example (sem credenciais)
```

Não enviar: API key, Authorization, cookies, tokens, senhas, `.env` real.

## 9. Mensagem de retomada

```text
Leia primeiro onde_paramos.md e use somente fatos documentados.
Última versão validada: v0.1.4.15. Instalada: v0.1.4.35.
Não confunda implementado com validado.
Comece pela tarefa 1 da seção 5 (data do arquivo diário de logs).
Preserve o layout validado e não altere o fluxo híbrido sem analisar logs e código atual.
Peça os arquivos completos atuais antes de criar patch.
Gere UPDATE e FLOW por versão; o ADD só depois da minha validação.
```
