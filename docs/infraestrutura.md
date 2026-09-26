# Plano de Infraestrutura

## Checklist antes do primeiro deploy

- [x] Confirmar que o plano Hostgator é **VPS** (acesso root/SSH, Docker instalável) e não hospedagem
      compartilhada (cPanel comum, sem Docker/root) — ver seção "Como pegar o SSH" abaixo.
- [ ] Cloudflare Tunnel nomeado configurado, com `layssafaria.com` apontando para ele (ver seção abaixo).
- [ ] Conta Supabase criada (tier free serve para começar).
- [ ] Chave de API da OpenAI, com billing ativo.
- [ ] Número de WhatsApp dedicado, com o app instalado em um celular para escanear o QR Code do WAHA.
- [ ] Ler o `hostgator-setup-kit/install.sh` do DeskcommCRM linha a linha antes de rodar em produção.

## Domínio: layssafaria.com via Cloudflare Tunnel (decisões D005 e D007)

O domínio oficial **`layssafaria.com`** já foi comprado pela Dra. Layssa na Hostgator (1 ano, confirmado
em 2026-09-26 — decisão D007). Continuamos usando o **Cloudflare Tunnel** (`cloudflared`) para expor a
VPS pela internet com HTTPS válido de graça, sem abrir porta nenhuma no firewall — só que agora direto
com **túnel nomeado**, já que existe domínio fixo para apontar (não precisa mais do modo rápido
`trycloudflare.com`):

1. Criar uma conta gratuita no Cloudflare e adicionar o domínio `layssafaria.com` (o Cloudflare vai
   indicar dois nameservers, tipo `xxx.ns.cloudflare.com`).
2. No painel da Hostgator, trocar os nameservers do domínio para os que o Cloudflare indicou (propagação
   pode levar de minutos a algumas horas).
3. Instalar o `cloudflared` na VPS (dentro do próprio Docker Compose do projeto ou como serviço do
   sistema) e autenticar (`cloudflared tunnel login`).
4. Criar o túnel nomeado: `cloudflared tunnel create clinica-layssa`.
5. Criar o registro DNS (CNAME) apontando `app.layssafaria.com` para o túnel:
   `cloudflared tunnel route dns clinica-layssa app.layssafaria.com`.
6. Usar `https://app.layssafaria.com` como `NEXT_PUBLIC_APP_URL` no `.env` e como URL de callback/webhook
   onde for pedido (ex.: OAuth do Google Calendar, que exige URL estável — agora já dá pra configurar,
   porque o domínio é fixo).

**Decidido (2026-09-26):** a Dra. Layssa quer um site institucional em `layssafaria.com` futuramente (fora
do escopo deste projeto). Por isso o DeskcommCRM usa o subdomínio **`app.layssafaria.com`** (passo 5) —
subdomínio é só mais um registro DNS, sem custo extra no plano free do Cloudflare, e não conflita com o
site que vai ocupar a raiz do domínio.

### Jeito mais simples: criar o túnel pelo dashboard (sem `cloudflared tunnel login`)

Em vez dos comandos `cloudflared tunnel login`/`create`/`route dns` acima (que pedem login via navegador
na própria VPS), dá pra fazer tudo pelo painel web, o que só precisa rodar **um comando** na VPS:

1. Painel do Cloudflare → **Zero Trust → Networks → Tunnels → Create a tunnel** → tipo "Cloudflared" →
   dar um nome (ex. `clinica-layssa`).
2. O painel mostra um comando de instalação com um token embutido (ex. para Ubuntu:
   `curl -fsSL ... | sudo bash` seguido de `sudo cloudflared service install <TOKEN>`) — rodar esse
   comando uma única vez na VPS via SSH.
3. Na aba **Public Hostname** do túnel, adicionar `app.layssafaria.com` apontando para
   `http://localhost:<porta-do-deskcommcrm>` — isso já cria o registro DNS automaticamente, não precisa
   mexer na aba de DNS separadamente.

Essa via evita o fluxo de login interativo do `cloudflared` e deixa só um comando pra rodar na VPS.

## Como pegar o acesso SSH na Hostgator

1. Acesse o painel do cliente da Hostgator: https://cliente.hostgator.com.br (login com o e-mail/senha
   da conta que contratou o serviço).
2. Em "Meus Produtos e Serviços" (ou "Hospedagem"), localize o serviço de **VPS** (importante: se só
   aparecer um serviço de "Hospedagem Compartilhada" com acesso a cPanel, **não é uma VPS** — não vai
   ter Docker nem root, e o instalador do DeskcommCRM não vai funcionar; nesse caso a VPS precisaria ser
   contratada à parte).
3. Dentro do gerenciamento da VPS deve aparecer o **IP do servidor** e uma opção para ver/resetar a
   **senha de root** (às vezes ela também é enviada por e-mail no momento da contratação).
4. Com IP e senha em mãos, conecte pelo terminal (no seu Windows, o PowerShell já tem `ssh` embutido):
   ```powershell
   ssh root@SEU_IP_AQUI
   ```
   Digite "yes" se perguntar sobre confirmar a identidade do servidor (primeira conexão), depois cole a
   senha de root.
5. **Assim que conectar pela primeira vez, faça isso antes de qualquer outra coisa** (VPS confirmada como
   Ubuntu 22.04 em 2026-09-26 — os comandos abaixo são para essa distro):
   - Troque a senha de root: `passwd`
   - Crie um usuário próprio com sudo (evite trabalhar como root no dia a dia):
     `adduser deskcomm && usermod -aG sudo deskcomm`
   - Siga a seção seguinte para configurar acesso por chave SSH.
   - Ative um firewall básico liberando só o necessário: `ufw allow 22 && ufw allow 80 && ufw allow 443 && ufw enable`

Não me passe a senha de root nem a chave SSH pelo chat — isso deve ficar só entre você e a VPS. Se
precisar que eu rode comandos na VPS futuramente, o caminho mais seguro é você mesmo abrir uma sessão
SSH e ir me colando as saídas dos comandos que eu pedir, ou usar uma ferramenta de acesso remoto que
você controle.

## Onde guardar IP, porta e usuário da VPS com segurança

**Não** colocar isso no `infra/.env` do projeto — esse arquivo é para segredos que a *aplicação* usa
(chaves de API), e IP/porta/usuário/senha de acesso root são muito mais sensíveis (dão controle total do
servidor), então merecem um lugar separado que nem sequer fica dentro da pasta do projeto.

**A forma correta é nunca "escrever" a senha em lugar nenhum a longo prazo — usar chave SSH:**

1. No seu PC (PowerShell), gerar um par de chaves só para esta VPS:
   ```powershell
   ssh-keygen -t ed25519 -C "layssafaria-vps" -f "$env:USERPROFILE\.ssh\layssafaria_vps"
   ```
   Vai pedir uma frase secreta (passphrase) — coloque uma, é uma camada extra de proteção da própria
   chave. Isso cria dois arquivos em `C:\Users\Samue\.ssh\` (fora do OneDrive, fica só nesse PC):
   `layssafaria_vps` (privada, nunca compartilhar) e `layssafaria_vps.pub` (pública).

2. Copiar a chave pública para a VPS (única vez que ainda vai usar a senha):
   ```powershell
   type "$env:USERPROFILE\.ssh\layssafaria_vps.pub" | ssh deskcomm@SEU_IP -p SUA_PORTA "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
   ```

3. Criar (ou editar) o arquivo `C:\Users\Samue\.ssh\config` com um "apelido" para a VPS:
   ```
   Host layssafaria-vps
       HostName SEU_IP
       Port SUA_PORTA
       User deskcomm
       IdentityFile ~/.ssh/layssafaria_vps
   ```
   A partir daqui, conectar é só `ssh layssafaria-vps` — o IP/porta/usuário ficam guardados nesse
   arquivo de configuração do SSH (que vive em `C:\Users\Samue\.ssh`, **fora** de qualquer pasta
   sincronizada pelo OneDrive), não em nenhum arquivo do projeto.

4. Testar `ssh layssafaria-vps` — deve conectar pedindo só a passphrase da chave (não mais a senha da
   VPS). Depois de confirmar que funciona, **desative login por senha** no servidor: editar
   `/etc/ssh/sshd_config`, colocar `PasswordAuthentication no` e `PermitRootLogin no`, depois
   `sudo systemctl restart ssh`. A partir daí não existe mais senha nenhuma pra vazar — só a chave
   privada no seu PC (protegida por passphrase).

Se quiser um lembrete simples por escrito além disso (ex.: "a VPS da clínica é a que uso pra tal coisa"),
use um gerenciador de senhas (Bitwarden, 1Password) em vez de um arquivo de texto — é o padrão correto
pra esse tipo de credencial, e continua acessível de qualquer dispositivo sem depender de sync de pasta.

## Domínio: qualquer um serve?

Sim — **praticamente qualquer domínio, de qualquer registrador, funciona** com essa arquitetura
(Cloudflare Tunnel/DNS + Let's Encrypt), incluindo `.com.br` (registro.br permite trocar os
"nameservers" para os do Cloudflare sem problema — é uma prática comum). A única exigência real é que
você (ou a clínica) **controle o DNS** desse domínio, ou seja, consiga apontar os nameservers para o
Cloudflare ou pelo menos criar registros nele. Isso vale tanto para um domínio novo quanto para um
subdomínio de um domínio que vocês já têm. Se em algum momento tiverem um domínio específico em mente,
me diga qual que eu confirmo se há alguma particularidade antes de configurar.

## Passo a passo (alto nível)

1. Confirmar que é uma VPS de verdade e conseguir o acesso SSH (seção acima).
2. Configurar o Cloudflare Tunnel para ter uma URL pública (seção acima).
3. Rodar o instalador guiado:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/melgarafael/DeskcommCRM/main/hostgator-setup-kit/comecar.sh | bash
   ```
   (revisar o script antes — ver nota de segurança abaixo)
4. Fornecer ao instalador: URL do Cloudflare Tunnel (no lugar do domínio), `OPENAI_API_KEY`, credenciais
   do Supabase, senha do admin.
5. Acessar a aplicação pela URL do túnel.
6. Conectar o WhatsApp escaneando o QR Code gerado pelo WAHA dentro da interface do DeskcommCRM.
7. Configurar o cron do `backup.sh` (Supabase free tier não faz backup automático).
8. Configurar integração com Google Calendar (OAuth) para agendamento — atenção: o Google exige uma URL
   de callback estável, então isso só faz sentido configurar depois de ter o túnel nomeado (ou o
   domínio definitivo), não com a URL aleatória do modo rápido.

## Nota de segurança sobre "curl | bash"

Rodar `curl | bash` direto de um script de terceiros em uma VPS de produção é uma prática que merece
cautela, mesmo quando o projeto é confiável (e este parece ser, ver
[`../memoria/02-pesquisa-deskcommcrm.md`](../memoria/02-pesquisa-deskcommcrm.md)). Antes do deploy real:

1. Baixar o script localmente primeiro (`curl -o comecar.sh ...`), ler o conteúdo, só depois rodar.
2. O mesmo vale para `install.sh`, que é quem de fato mexe no sistema (Docker, Postgres, HTTPS, cron).
3. Preferir rodar como usuário com sudo, não como root direto, quando possível.
4. Fazer isso com um snapshot/backup da VPS tirado antes (se a Hostgator oferecer), para poder reverter.

## Variáveis de ambiente que vamos precisar preencher

Já existe o arquivo real em [`../infra/.env`](../infra/.env) (ignorado pelo Git — pode preencher direto
nele, nunca precisa colar chave nenhuma aqui no chat) e o template versionado em
[`../infra/.env.example`](../infra/.env.example). Lista completa das variáveis do projeto original em
[`../memoria/02-pesquisa-deskcommcrm.md`](../memoria/02-pesquisa-deskcommcrm.md).

## Orçamento recorrente estimado

| Item | Custo estimado | Observação |
|---|---|---|
| VPS Hostgator | já contratada pelo usuário | confirmar plano/specs (RAM mínima recomendada: 4GB) e que é VPS de verdade (root/Docker), não hospedagem compartilhada |
| Domínio | já pago pela Dra. Layssa | `layssafaria.com`, 1 ano, Hostgator (decisão D007) |
| Cloudflare | R$0 (plano free cobre Tunnel + DNS) | |
| Supabase | R$0 (tier free) para começar | pode precisar upgrade se crescer muito |
| OpenAI API | variável, por uso (tokens) | monitorar via `AI_BUDGET_ENFORCEMENT` |
| WAHA | a partir de **US$ 5/mês** (tier "Community", inclui o que antes era "Plus": múltiplas sessões, mídia) | confirmado em 2026-09-25, ver `memoria/02-pesquisa-deskcommcrm.md` |
| Upstash Redis | R$0 (tier free) para começar | opcional |
