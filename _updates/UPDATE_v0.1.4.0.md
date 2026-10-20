# UPDATE v0.1.4.0

## Finalidade

Registrar e instalar integralmente o código atual recebido para a integração de Tickets Freshdesk, tipos de busca, filtros múltiplos e buscas salvas.

## Código registrado

- `src/server.js`;
- `src/lib/env.js`;
- `src/lib/freshdesk.js`;
- `src/lib/savedSearches.js`;
- frontend atual completo em `src/frontend/`.

## Documentos incorporados

- `CHANGELOG_TIPOS_BUSCA_FILTROS.md`;
- `DOCUMENTACAO_CONSOLIDADA_TICKETS_FRESHDESK.md`.

## Precedência documental

Quando houver divergência, considerar nesta ordem:

1. código desta versão;
2. `CHANGELOG_TIPOS_BUSCA_FILTROS.md`, de 27/09/2026;
3. `DOCUMENTACAO_CONSOLIDADA_TICKETS_FRESHDESK.md`, como consolidação histórica.

## Adaptação de segurança

Foi removido do HTML o valor preenchido no campo visual `#apiKey` (`fd_sk_2b91a6c7d84e4f1a9c30f7e6b5a4d2c1`). O campo passou a ser somente leitura e informa que `FRESHDESK_API_KEY` deve existir apenas no `.env` do servidor.

Nenhuma outra regra funcional do código recebido foi reescrita.

## Dados deliberadamente não incluídos

- `.env`;
- credenciais reais;
- `data/saved-searches.json`;
- sessões e registros de auditoria reais.

## Pendências preservadas

- validar a cláusula `fr_due_by` e o fuso aceito pela conta real;
- confirmar se “Não atribuído” também será um filtro manual;
- mapear campos personalizados ainda sem correspondência;
- revisar buscas salvas para uso multiusuário, sessão e auditoria;
- validar detalhe completo, conversas, anexos, respostas e anotações.
