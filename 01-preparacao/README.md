# Preparação inicial do projeto

**Assunto:** Preparação inicial
**Versão do código de referência:** v0.1.2.0 (servidor) / túnel Cloudflare instalado antes do código
**Data:** 2026-09-29
**Status:** vigente

## Escopo
Infraestrutura mínima para o Projeto4 existir de forma isolada: túnel Cloudflare, servidor local, estrutura de pastas e a cópia controlada do Projeto3 como ponto de partida.

## Estado atual

### Arquitetura
```text
navegador
  -> https://<subdominio>.<dominio>/         (Cloudflare)
  -> túnel nomeado (serviço cloudflared, Windows 11)
  -> http://127.0.0.1:65004                  (servidor Node do Projeto4)
```

O túnel liga o PC Windows (via serviço `cloudflared`) à rede da Cloudflare; não há porta aberta no roteador nem port forwarding. A Hostinger não participa mais dessa cadeia — só cedeu os nameservers do domínio para a Cloudflare (ver `02-acesso-seguranca`) e pode voltar a ser usada só como servidor de arquivos e imagens.

### Configuração do túnel
| Item | Valor |
|---|---|
| Application | `parte4-cloudflare-v4` |
| Protocol | HTTP |
| Internal hostname | `127.0.0.1` |
| Port | `65004` |
| Subdomain | `<subdominio>` |
| Domain | `<dominio>` |
| Nome do Tunnel | `parte4-cloudflare` |

Valores reais de subdomínio, domínio e conta ficam fora da documentação (ver `02-acesso-seguranca`, seção de credenciais).

### Serviço do túnel (Windows)
```powershell
cloudflared.exe service install
Get-Service cloudflared   # status
Start-Service cloudflared # inicia
```

Acesso: `https://<subdominio>.<dominio>/`.

### Servidor local
- raiz do projeto: `parte4_cloudflare`;
- `package.json` e servidor HTTP em Node próprios do Projeto4, sem reaproveitar módulos do Projeto3 (v0.1.2.0);
- `GET /` entrega `src/frontend/principal.html`;
- `GET /login` entrega `src/frontend/login.html`;
- `GET /health` e `GET /api/health` informam a saúde do servidor;
- host `127.0.0.1`, porta `65004`.

### Launcher local (`abrir_projeto4.cmd`)
```text
abrir_projeto4.cmd
  -> consulta /health
  -> se ligado: abre http://127.0.0.1:65004/
  -> se desligado: tenta node src/server.js
       -> sucesso: abre principal.html pela porta
       -> falha: abre servidor-desligado.html localmente
```

Um servidor desligado não consegue entregar sua própria página de erro — por isso o aviso é aberto localmente pelo launcher. **Essa cobertura vale só para o acesso local.** Pelo endereço público da Cloudflare, com o servidor local fora do ar, ainda aparece a página de erro padrão da Cloudflare, não uma página própria do projeto (pendência `P4-0003`).

### Estrutura inicial de pastas
```text
parte4_cloudflare/
├── src/
│   ├── server.js
│   ├── lib/
│   └── frontend/
├── data/
├── _patches/
└── _backup_patch/
```

### Cópia do Projeto3 para o Projeto4
Regra vigente (ver `registros/decisoes`): trabalhar sempre sobre cópia controlada da última base validada do Projeto3, excluindo credenciais, cookies e sessões reais do pacote de desenvolvimento, com `.env.example` só com valores fictícios. A cópia efetiva ainda não foi realizada (pendência `P4-0001`) — o Projeto3 completo ainda será enviado para compor `legado-projeto3`.

### Independência de portas e estado
Regra vigente: Projeto3 e Projeto4 devem poder rodar ao mesmo tempo sem compartilhar sessão, cookie ou estado operacional, e sem depender das portas do Projeto3 (3003/3004). A porta e o host do Projeto4 já estão fixados; o isolamento completo de cookies, arquivos de estado e logs ainda não foi verificado (pendência `P4-0002`, estado parcial).

## Pendências deste assunto
Ver `registros/pendencias.md`: `P4-0001`, `P4-0002`, `P4-0003`.

## Novidades ainda não incorporadas
Ver a pasta `_a-incorporar/`.
