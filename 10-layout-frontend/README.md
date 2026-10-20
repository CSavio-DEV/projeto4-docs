# Layout, frontend e mobile

**Assunto:** Layout, frontend e mobile
**Versão do código de referência:** v0.1.4.8 (frontend consolidado a partir da v0.1.3.0)
**Data:** 2026-09-29
**Status:** vigente, com ressalva — o frontend atual é um pré-projeto

## Aviso importante
O frontend consolidado (`principal.html`/`.css`/`.js`) instalado na v0.1.3.0 é um **pré-projeto**: ainda há itens a revisar nele antes de considerá-lo definitivo. O que está registrado abaixo descreve o estado atual e as lições já aprendidas, não uma versão final aprovada.

## Escopo
Consolidação dos HTML separados em um único layout (`principal.*`), sistema de design herdado do Projeto3, padrões mobile e as lições da fusão.

## Arquivo único
Decisão vigente: `principal.html` único é o padrão do Projeto4 — não se mantêm dois sistemas de HTML (desktop/mobile) separados como no Projeto3. As versões `*-mobile.html` do Projeto3 servem só como referência visual (ver `legado-projeto3`).

## Estrutura do shell
```text
.app-shell
  -> .sidebar (72px, ícones)
  -> .main
       -> .topbar
       -> .page-host
            -> uma .app-page visível por vez, controlada por data-page
```
Navegação interna por `data-page`, URL com âncora (`#dashboard`, `#tickets`, `#automacoes`, `#portal`, `#ia`, `#config`, `#logs`).

## Sistema de design (herdado do Projeto3, vigente enquanto não revisado)

### Paleta
| Token | Valor | Uso |
|---|---|---|
| `--bg` | `#14161c` | Fundo geral |
| `--surface` | `#1c1f27` | Cards, painéis, sidebar |
| `--surface-2` | `#22262f` | Campos de formulário |
| `--border` | `#33383f` | Bordas e divisores |
| `--text` | `#eceef1` | Texto principal |
| `--text-muted` | `#9aa0aa` | Texto secundário |
| `--text-faint` | `#6b7078` | Placeholder, texto terciário |
| `--accent` | `#d6a13c` | Botão primário, foco, destaque único |
| `--success` | `#4f9d6e` | Estado de sucesso/ativo |
| `--danger` | `#e0524a` | Estado de alerta/erro |

Grafite quente com um único acento âmbar para ações primárias; verde e vermelho só para estado real, nunca decoração.

### Tipografia e espaçamento
- Space Grotesk para títulos/rótulos (`var(--font-display)`); Inter para corpo e tabelas (`var(--font-body)`); nunca uma terceira família;
- escala de espaçamento em variáveis `--sp-1` a `--sp-9` (4px a 56px), sem valores soltos.

### Componentes reutilizáveis
`.card`, `.field`/`.input`, `.input-wrap`, `.checkbox-row`, `.btn`/`.btn-primary`/`.btn-secondary`/`.btn-block`, `.status-dot` (`is-idle`/`is-active`/`is-alert`), `.switch`, `.badge` (`-success`/`-danger`/`-accent`/`-neutral`), `.data-table`, `.record-card`, `.empty-state`, `.config-card`, `.alert-row`, `.stepper`, `.channel-row`, `.automation-grid`, `.notice-panel`/`.notice-list`/`.notice-item`.

### Convenções de escrita
- minúsculas normais, sem caixa alta em rótulos;
- evitar títulos compostos com ponto médio; título simples + subtítulo curto;
- mensagens de estado vazio explicam o que fazer, não só "nada encontrado".

## Padrões mobile
- inputs com `font-size` mínimo de 16px (evita zoom automático no iOS);
- alvos de toque com no mínimo 40–44px de altura;
- barra lateral vira menu hambúrguer no topo;
- tabelas largas viram lista de cartões empilhados em vez de rolagem horizontal;
- respeitar `env(safe-area-inset-top)`/`env(safe-area-inset-bottom)`;
- painéis laterais fixos (como Contexto geral) viram folha (sheet) que sobe de baixo para cima;
- breakpoint único do sistema: `860px`.

## Padrão para alternar tabela ↔ cartões (lição reaproveitável)
Manter os dois blocos sempre no DOM (`.data-table-wrap` e `.ticket-cards`/`.ticket-card`) e deixar só o CSS por `@media (max-width: 860px)` decidir qual mostrar, em vez de alternar por JavaScript. Evita sincronizar estado entre duas representações da mesma lista. Ver aplicação em `03-tickets-freshdesk/painel-contexto.md`.

## Checklist ao entregar ou revisar uma tela
- usa só o CSS compartilhado + estilos específicos da própria página, sem duplicar tokens;
- nenhum dado real (nome completo, CPF, e-mail, IP, número de ticket) — tudo fictício;
- título da aba do navegador no padrão `Projeto4 — [Nome da tela]`;
- versão desktop validada visualmente antes da adaptação mobile;
- links de navegação (sidebar e menu mobile) atualizados em todas as telas;
- foco de teclado visível em todos os campos e botões interativos;
- texto revisado: minúsculas normais, sem ponto médio em título, estado vazio explicativo;
- qualquer `<style>` específico de uma tela é promovido ao CSS compartilhado antes de considerar a tela pronta (causa raiz da maioria dos bugs já corrigidos — ver `registros/erros-e-licoes`).

## Referência rápida de seletores centrais
| Elemento/conceito | Onde vive |
|---|---|
| Layout raiz | `.app-shell`, `.sidebar`, `.main`, `.topbar`, `.content` |
| Menu mobile | `.mobile-topbar`, `.mobile-nav-overlay`, `body.menu-open` |
| Faixa de usuário/contador mobile | `.mobile-user-strip`, `.mobile-run-row` |
| Navegação entre páginas | `[data-nav]`, `[data-page]`, `showPage()` em `principal.js` |
| Identidade do usuário | `session` (`sessionStorage p4.session.v1`), `[data-user-name]`, `[data-user-origin]`, `[data-user-initials]`, `[data-portal-status]` |
| Gate do Portal | `configurePortalGate()`, `.portal-session-notice`, `.portal-controls-disabled` |
| Tickets (tabela + cartões) | `#ticketList`, `.data-table-wrap`, `.ticket-cards`/`.ticket-card`, `openTicket()`/`closeTicket()` |
| Switches ligado/desligado | `.switch`, `.switch .track` (`pointer-events:none`) |
| Steppers de intervalo | `.stepper[data-min][data-value]`, `.stepper-value`, `adjustStepper()`, `stepFor()` |
| Contexto lateral | `.context-panel`, `.context-fab`, `setContext()` |
| Placeholder genérico | `toast()`, handler `document.addEventListener('click', …)` em `principal.js` |

Botões dentro de `[data-page]` sem integração ligada mostram `toast('Controle visual preservado...')` — mensagem proposital avisando que a função ainda não foi conectada (não é bug).

## Telas ainda não migradas (especificação herdada do Projeto3)
Ver `06-configuracoes/README.md` e `09-logs-auditoria/README.md` para os campos e o layout de referência de cada uma.

## Ajustes de tela já solicitados (Tickets, 2026-09-29)

Primeiros itens concretos da revisão do pré-projeto, todos registrados em `03-tickets-freshdesk/pendencias-tickets.md`: botão discreto `(i)` que reúne explicações e detalhes técnicos hoje fixos na tela (TCK-0106); nome da busca ativa exibido uma única vez (TCK-0107); botão `← Voltar` com seta amarela (TCK-0108). A lista completa de itens a revisar no frontend ainda não foi informada.

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0102` (mapa de componentes), `P4-1101`–`P4-1103` (mobile por página), e a ressalva de pré-projeto no topo deste documento.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
