# Acesso, login e segurança

**Assunto:** Acesso, login e segurança
**Versão do código de referência:** v0.1.1.0 (login/Portal opcional) + configuração Cloudflare vigente; sessão principal e sessão do Portal definidas no planejamento e ainda não implementadas (base atual v0.1.4.13)
**Data:** 2026-09-29
**Status:** vigente (a sessão principal/Portal é decisão adotada, com implementação e validação pendentes)

## Escopo
Cloudflare Access, login do Portal (opcional), identidade e sessão, usuários e perfis, política de credenciais e riscos. Este documento guarda apenas a política — nunca valores reais de credencial.

## Estado atual

### Camada de acesso externo — Cloudflare Access
- acesso externo protegido por Cloudflare Access, na frente do túnel descrito em `01-preparacao`;
- primeiro método de login validado: e-mail autorizado com One-Time PIN;
- cada novo usuário deve ter e-mail individual; o e-mail operacional do projeto não deve virar conta compartilhada por todos;
- plano em uso: Zero Trust Free (a confirmar que permanece — ver pendências);
- API Token da Cloudflare não foi necessário para a instalação inicial;
- Tunnel Token é secreto e deve ser rotacionado se houver exposição.

### DNS e domínio
- nameservers do domínio migrados da Hostinger para a Cloudflare;
- regras permanentes: não alterar os nameservers novamente, não ativar DNSSEC, não excluir registros DNS, não mudar registros de e-mail, não abrir a porta 65004 no roteador, não criar port forwarding.

### Identidade dentro do Projeto4
Ordem de resolução do nome exibido no topo (decisão registrada, formato exato ainda pendente):
1. nome confiável fornecido pelo Cloudflare Access;
2. nome da sessão local do Projeto4, se houver;
3. identificador ou e-mail autenticado, quando permitido;
4. nome genérico `Usuário Projeto4`.

Regras:
- não usar o login do Portal como identidade principal do Projeto4;
- não confiar em cabeçalho enviado diretamente pelo navegador sem validar que a requisição passou pelo túnel/Access esperado;
- registrar a origem da identidade na sessão e na auditoria.

A validação de que o cabeçalho/token realmente vem do Cloudflare Access (e não de acesso direto local) ainda não foi implementada (pendência `P4-0204`).

### Login do Portal (opcional)
Desde a v0.1.1.0, após a autenticação de acesso ao Projeto4, a aplicação oferece:
- **Entrar no Portal**;
- **Não logar no Portal**.

A decisão pertence à sessão individual do usuário e pode ser alterada depois, sem afetar outras sessões. Sem sessão do Portal, ficam desabilitados: busca manual do Portal, busca de usuário a partir do ticket, automações que dependem do Portal e o fluxo real de reset de senha dependente do Portal — com mensagem objetiva na interface e opção de logar depois. Senha do Portal não é persistida.

### Sessão principal e acesso opcional ao Portal (decisão de 2026-09-28)

**Estado:** decisão funcional adotada; implementação e teste pendentes.

O Projeto4 tem **uma única identidade principal por usuário**: a autenticada pelo Cloudflare Access. O login no Portal não cria um segundo usuário e não substitui a identidade principal.

```text
Usuário autenticado pelo Cloudflare Access
  -> sessão principal do Projeto4
  -> Portal desativado ou ativado (capacidade opcional da mesma sessão)
```

Regras da sessão principal:

- todas as ações, logs e auditorias usam a identidade principal;
- o login no Portal só acrescenta a capacidade de acessar o Portal; não altera nome, identificador nem autoria;
- tickets abertos, filtros, rascunhos, contexto e auditoria continuam ligados ao usuário principal;
- não existem duas identidades concorrentes para o mesmo usuário.

Sem sessão válida do Portal:

- a página de busca do Portal fica indisponível para operações reais;
- a busca contextual de usuários do Portal fica desabilitada;
- automações que dependem do Portal não executam a etapa correspondente;
- o reset de senha dependente do Portal não executa ações reais;
- o frontend informa que o acesso ao Portal não está ativo **e o backend também bloqueia as rotas protegidas**, sem depender só do estado visual.

Com sessão válida, o mesmo usuário principal acessa as funções do Portal, as ações continuam registradas em nome do usuário do Cloudflare Access e a sessão fica isolada das de outros usuários.

#### Duração da sessão do Portal

```text
Duração inicial: 24 horas, contadas a partir da autenticação bem-sucedida no Portal
Estado: adotada, aguardando implementação e teste
```

A sessão termina antes de 24 horas por logout explícito do Portal, logout completo do Projeto4 (se a política implementada encerrar também o Portal), invalidação pelo próprio Portal, falha de renovação ou validação, revogação administrativa ou limpeza segura. Antes de uma operação protegida, a validade real deve ser verificada: o Projeto4 não mantém artificialmente uma sessão inválida.

#### Referência do Projeto2

O Projeto2 tinha persistência de sessão do Portal que permanecia ativa; no Projeto3 essa continuidade se perdeu. O Projeto4 deve analisar o mecanismo do Projeto2 e aproveitar só o que for compatível com isolamento por usuário, publicação web, segurança de cookies, Cloudflare Access, limite de cinco usuários simultâneos e auditoria individual. Não copiar arquivos de sessão do Projeto2 sem mapear formato, validade, segurança, escopo e dependências.

#### O que registrar e o que nunca registrar

Registrar: identidade principal do Cloudflare Access; data e hora; resultado do login no Portal; ativação, expiração, invalidação ou logout da sessão do Portal; operação do Portal executada, quando necessária para auditoria, e seu resultado.

Nunca registrar: senha do Portal, valor completo de cookie, token de sessão, cabeçalhos de autenticação, credenciais reutilizáveis. O nome usado no login do Portal não vira identidade principal; se for necessário para diagnóstico, fica restrito aos logs protegidos, com política de retenção.

#### Como validar (quando implementado)

1. autenticar dois usuários diferentes pelo Cloudflare Access;
2. ativar o Portal só para o primeiro e confirmar que só ele acessa as buscas do Portal;
3. confirmar que os dois continuam identificados pelas identidades principais;
4. confirmar que o segundo não recebe cookies, resultados ou estado do primeiro;
5. recarregar a interface e verificar a persistência esperada;
6. simular expiração e confirmar o bloqueio das rotas;
7. confirmar que nenhum segredo aparece nos logs;
8. testar logout do Portal sem trocar a identidade principal;
9. testar o limite temporal de 24 horas.

### Multiusuário
O Projeto4 é preparado inicialmente para até 5 usuários simultâneos, com sessão, cookies, ticket aberto, editor/rascunho, intervenção manual e auditoria individuais por usuário; estado global só para serviços realmente globais. A implementação completa (sessão simultânea testada, limites e expiração) ainda está pendente (`P4-0205`).

### Política de credenciais
- credenciais reais nunca entram em documentos, patches ou repositório;
- tokens e chaves nunca retornam completos ao frontend e nunca entram em logs, UPDATE, FLOW ou pacote de patch;
- em qualquer documento, um valor no formato de credencial — real ou de exemplo — deve usar um placeholder óbvio (ex.: `<chave-exemplo>`), nunca uma string que pareça uma chave real (ver `registros/erros-e-licoes`);
- a chave de API do Freshdesk fica somente no `.env` do servidor; o campo visual correspondente na interface é somente leitura.

### Riscos e continuidade
- o Projeto3 real só será conectado ao Projeto4 depois das revisões de autenticação, autorização, sessão e auditoria;
- versões V2, V3, V4 e evoluções V4.x podem coexistir em hostnames e portas separados, sem interferir umas nas outras.

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0204`, `P4-0205`, `P4-0207` a `P4-0209` (sessão principal e sessão do Portal), além das de decisão futura (cabeçalho/token do Cloudflare a usar como identidade, tempo de expiração de sessão, ativação de MFA na conta administrativa, logout completo do Projeto4 encerrar ou não a sessão do Portal, política de renovação dentro das 24 horas, teste do bloqueio de e-mail não autorizado, procedimento de inclusão/remoção de usuários).

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
