# Diário do Projeto

Registro cronológico de tudo que foi feito, decidido e descoberto. Toda sessão de trabalho deve
adicionar uma entrada nova no topo (mais recente primeiro).

> **Nota de localização (2026-09-26):** o projeto vive agora em `D:\Projetos\layssafaria` (antes estava
> em `OneDrive\Área de Trabalho\layssafaria`). Ver decisão D006.

---

## 2026-09-27 — Configuração dos agentes começada: WhatsApp, funil e follow-ups

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário confirmou o WhatsApp da clínica (**+55 64 99626-2769**, número que a
clínica já usa) e entrou pela primeira vez no painel do DeskcommCRM pra configurar os agentes. Achado o
guia oficial do próprio produto pra isso — skill `deskcomm-cliente-novo` dentro do repositório
(`.agents/skills/deskcomm-cliente-novo/`), que existe exatamente pra montar um cliente novo por nicho.

Como o guia exige triagem (não dá pra inventar regra de negócio), criado
`clinica/questionario-dra-layssa.md` — e depois, a pedido do usuário, uma versão em PDF
(`clinica/questionario-dra-layssa.pdf`, gerado com reportlab, **não versionado no Git** por pedido do
usuário) — cobrindo tanto a base de conhecimento (preços, procedimentos, FAQ) quanto as regras de
comportamento do agente (o que pode decidir sozinho, quando chama humano, regras de reengajamento).
Enviado pra Dra. Layssa responder; ainda aguardando.

Enquanto isso, avançado tudo que **não** depende das respostas dela, seguindo `pela-tela.md` do guia:
1. **Conexões:** WhatsApp conectado (status Conectado), IA em modo de teste (proposital).
2. **IA › Credenciais:** já existia uma chave OpenAI validada, criada no onboarding inicial.
3. **IA › Provedores:** mantido "Modelo padrão" (GPT-5.6 Terra) pra tudo — simples e já funcional;
   afinar por ponto de uso fica pra depois, com dado real de custo. Confirmado que "Jev" segue desligado
   (decisão D003).
4. **Funil "Agendamentos":** reconfigurado o funil padrão (que veio com etapas de e-commenrce, tipo
   "Carrinho abandonado" — modelo errado que o onboarding cria por padrão) pro pacote oficial de
   clínica: 7 etapas (Novo contato → Já respondi → Entendendo o caso → Quer agendar → Escolhendo
   horário → Consulta marcada [ganho] → Não vai marcar [perdido]), vocabulário
   paciente/consulta/marcada/não marcou. Achado detalhe da interface: pra mudar qual etapa é "ganho",
   primeiro escolhe a NOVA etapa como ganho (a marcação se move sozinha) — tentar remover da antiga
   primeiro dá erro.
5. **IA › Follow-ups:** instalados (rascunho, não publicados) os modelos prontos de clínica **Falta**
   (remarcar quem não veio) e **Consulta** (retomar quem sumiu na marcação). **Não instalados**: Exame e
   Cirurgia — pedem uma etapa do funil como gatilho, e nosso funil não tem etapa de "aguardando
   exame"/"decidindo cirurgia" — decisão de processo real da clínica, perguntar à Dra. Layssa.

**Pendente para a próxima sessão:** aguardar resposta do questionário; quando vier, preencher
`clinica/base-conhecimento.md`, escrever o prompt final do agente de Recepção (esqueleto já existe no
pacote de clínica do guia), publicar os follow-ups, testar, e só depois publicar o agente de verdade.

**Arquivos alterados:** `clinica/questionario-dra-layssa.md` (novo), `.gitignore` (`*.pdf` ignorado),
`memoria/03-pendencias.md`, `memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 10) — DeskcommCRM instalado e no ar

**Participantes:** Samue + Claude Code

**O que aconteceu:** rodado `ubuntu-production-installer.sh --domain app.layssafaria.com` na VPS. Três
problemas reais encontrados e corrigidos ao longo do processo (não foi tentativa e erro sem rumo — cada
falha apontava pra uma causa específica, diagnosticada antes de mexer em qualquer coisa):

1. **`sudo -v` exigia senha mesmo com NOPASSWD** configurado — corrigido com
   `Defaults:deskcomm !authenticate` em `/etc/sudoers.d/deskcomm-nopasswd` (ajuste só na VPS, não no
   repositório).
2. **Validador de chave do Supabase (`v_sb_key` em `hostgator-setup-kit/install.sh`) testava a chave pela
   URL pública**, que só existe depois do Caddy subir — um validador vizinho (`v_sb_url`) já tinha o
   desvio certo pra usar a URL interna em modo single-server, mas esse não. Corrigido replicando o mesmo
   padrão (commit `9f76ea6`).
3. **Caddy não conseguia emitir certificado Let's Encrypt** porque, atrás de Cloudflare Tunnel, o domínio
   nunca aponta pro IP real da VPS — os desafios `tls-alpn-01` e `http-01` do ACME falham porque quem
   responde no lugar da origem é a própria Cloudflare. Trocado por certificado autoassinado de longa
   duração (`Caddyfile.single-server`, commit `d0f5121`) — o "No TLS Verify" do túnel cobre esse salto.
   Isso por sua vez quebrou a chamada interna que o app/worker fazem pro domínio público (o Node.js
   valida certificado por padrão e rejeitava o autoassinado) — corrigido com `NODE_EXTRA_CA_CERTS`
   apontando pro mesmo certificado, só para app/worker (`docker-compose.single-server.yml`, commit
   `e44d080`).

Usuário questionou por que não seguimos "o caminho comum" (DNS direto pro IP da VPS, sem Cloudflare
Tunnel) — resposta: essa escolha foi feita antes, por boas razões (não expor o IP real, não abrir portas,
proteção/HTTPS de graça — decisão que already existia desde antes do DeskcommCRM entrar em cena), e o
instalador do DeskcommCRM assume por padrão o caminho comum, daí o atrito. Optamos por manter o túnel
(usuário concordou depois de entender o motivo) em vez de simplificar pro caminho direto.

**Resultado:** https://app.layssafaria.com no ar, login funcionando, HTTPS válido (certificado de verdade
da Cloudflare na borda — o autoassinado é só um detalhe interno entre containers, invisível pro
visitante). Admin criado: `admin@app.layssafaria.com`, senha em
`deskcommcrm/.runtime/admin-credentials` (permissão 600) na VPS.

**Pendente para a próxima sessão:** logar no painel e trocar a senha do admin, conectar o WhatsApp
(+55 64 99626-2769) via QR Code, cadastrar uma chave de IA (OpenAI/Anthropic) em IA › Credenciais,
configurar SMTP.

**Arquivos alterados:** `hostgator-setup-kit/install.sh`, `Caddyfile.single-server`,
`docker-compose.single-server.yml`, `memoria/03-pendencias.md`, `memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 9) — Revisão de segurança, mitigação de RAM, código na VPS, backup agendado

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário decidiu seguir com a VPS atual (3.8GB RAM, abaixo dos 8GB recomendados pelo
instalador, mas acima do mínimo de 4GB), pedindo pra minimizar ao máximo risco de travamento/lentidão.
Definido o número de WhatsApp: **+55 64 99626-2769** (número já usado pela clínica, não um novo dedicado
— decisão consciente do usuário mesmo após aviso do risco). Usuário delegou explicitamente a revisão dos
scripts de instalação ("você tá no comando").

Revisão de segurança completa: lidos `ubuntu-production-installer.sh` e
`hostgator-setup-kit/install-single-server.sh` inteiros, mais varredura por padrões de risco (chamadas de
rede, comandos destrutivos, enfraquecimento de firewall, exfiltração de dados) em `_common.sh`,
`install.sh`, `agent.sh` e `backup.sh`. Nada suspeito — instalador oficial do Supabase é baixado e
conferido por SHA-256 antes de rodar, credenciais geradas aleatoriamente com arquivo protegido (chmod
600), backup com verificações de integridade, nenhuma chamada de rede não documentada (o único POST
externo visto em `agent.sh` é pro próprio app, fila interna de automações). **Aprovado.**

Mitigação de RAM: swap ampliado de 2GB pra **6GB** e `vm.swappiness` ajustado pra **10** (usa RAM real
primeiro, só recorre a swap sob pressão de verdade) — reduz risco de OOM em picos (ex. build) sem custo
adicional. Se ainda assim ficar lento/instável em uso real, o próximo passo seria upgrade de RAM na
Hostgator.

Código enviado pra VPS: `git clone` do nosso próprio repositório em `/home/deskcomm/layssafaria` (inclui
`deskcommcrm/`). Cron de backup diário configurado (`0 3 * * *`, via `hostgator-setup-kit/backup.sh`,
log em `~/backup.log`) — só vai gerar backup de verdade depois que a stack estiver instalada e rodando.

**Pendente para a próxima sessão:** rodar `bash ubuntu-production-installer.sh --domain
app.layssafaria.com` na VPS.

**Arquivos alterados:** `memoria/03-pendencias.md`, `memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 8) — Código-fonte do DeskcommCRM vendorizado no repositório

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário pediu pra trazer o código completo do DeskcommCRM
(https://github.com/melgarafael/DeskcommCRM) pra dentro da pasta do projeto. Clonado em
`deskcommcrm/` (152MB sem contar `.git`), removido o histórico Git upstream (não usamos submodule —
os arquivos entram direto no nosso próprio histórico, mais simples de gerenciar). Ajustado o
`.gitignore` da raiz: adicionadas exceções pra `.env.hostgator.example` e `.env.voip.example`
(templates que o DeskcommCRM versiona e que a regra genérica `.env.*` estava bloqueando por engano) e
mais padrões de build (`out/`, `dist/`, `build/`, `.turbo/`, `.vercel/`, `coverage/`) que o
`.gitignore` próprio do DeskcommCRM usa. Conferido que nenhum segredo real (`node_modules`, `.env`
reais, `asterisk/pjsip.conf`/`ari.conf`) veio no clone. Commitado e enviado ao GitHub.

**Observação:** perdemos o histórico de commits do projeto original ao remover o `.git` interno — se
precisarmos comparar com atualizações futuras do upstream, o caminho é re-clonar numa pasta separada e
diferenciar manualmente, ou reconsiderar submodule mais adiante.

**Pendente para a próxima sessão:** revisar `deskcommcrm/ubuntu-production-installer.sh` (o entrypoint
atual — chama `hostgator-setup-kit/install-single-server.sh` por baixo; os nomes antigos citados em
`docs/infraestrutura.md`, `install.sh`/`comecar.sh`, ficaram desatualizados) linha a linha antes do
primeiro deploy.

**Arquivos alterados:** `.gitignore`, `deskcommcrm/**` (novo), `memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 7) — Túnel Cloudflare instalado e funcionando

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário optou por criar o túnel pelo painel do **Zero Trust** (produto que exige
cadastro de cartão mesmo no plano free, medida antifraude da Cloudflare — sem custo, avisado sobre a
alternativa via CLI sem cartão, mas o usuário preferiu seguir pelo painel). Criado o túnel `clinica-layssa`
(conector Cloudflared), gerado o token de instalação, e o Claude Code instalou o `cloudflared` na VPS via
apt e ativou o serviço com o token — rodando via systemd, conectado ao edge de São Paulo (`gru13`).
Configurado o Public Hostname `app.layssafaria.com` → `localhost:3000` (porta é uma suposição baseada no
padrão do Next.js, DeskcommCRM ainda não foi instalado). Testado com `curl`: domínio responde **HTTP
502**, que é o resultado esperado agora (confirma DNS + SSL + túnel funcionando ponta a ponta; só falta a
aplicação real rodando na porta).

**Pendente para a próxima sessão:** rodar o instalador guiado do DeskcommCRM na VPS (revisar o script
antes, ver nota de segurança em `docs/infraestrutura.md`) e ajustar a porta do Public Hostname se
necessário.

**Arquivos alterados:** `memoria/03-pendencias.md`, `memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 6) — Acesso SSH à VPS estabelecido, sudo liberado pro Claude Code

**Participantes:** Samue + Claude Code

**O que aconteceu:** primeiro acesso à VPS via SSH. Achado no caminho: `ssh root@IP` na porta 22 dava
"connection timed out" — a Hostgator usa **porta 22022**, não a 22 padrão (só aparece no painel de
detalhes da VPS). Depois de conectar, usuário rodou o roteiro de bootstrap (senha nova de root, criação
do usuário `deskcomm` com sudo, chave pública do Claude Code copiada pra `authorized_keys`, firewall
`ufw` liberando 22022/80/443). Configurado o alias `layssafaria-vps` em `~/.ssh/config` no PC do Samue e
confirmado que o Claude Code já consegue rodar comandos direto na VPS via SSH (chave ed25519 sem
passphrase, decisão consciente do usuário — trade-off de segurança documentado).

Discutido e aprovado liberar **`sudo` sem senha** (`NOPASSWD:ALL`) pro `deskcomm`, pra evitar interromper
o usuário a cada comando administrativo — registrado como decisão **D008** em `01-decisoes.md`, com nota
explícita pra reavaliar/apertar antes de ter dado real de paciente na VPS.

**Pendente para a próxima sessão:** criar o túnel no painel do Cloudflare (Zero Trust → Networks →
Tunnels) — isso é ação de conta, só o usuário consegue fazer — e depois rodar o comando de instalação
do `cloudflared` que o painel gerar, agora direto por SSH.

**Arquivos alterados:** `memoria/01-decisoes.md` (D008), `memoria/03-pendencias.md`,
`memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 5) — Domínio ativo no Cloudflare, SSL/TLS configurado

**Participantes:** Samue + Claude Code

**O que aconteceu:** confirmação de ativação do domínio no Cloudflare chegou bem mais rápido que as 24h
previstas ("Seu domínio agora está protegido pela Cloudflare"). Configurado SSL/TLS: modo de criptografia
**"Completo"** (Cloudflare↔origem também criptografado), **"Sempre usar HTTPS"** ativado, e **versão
mínima de TLS elevada de 1.0 para 1.2** (protocolos antigos/inseguros desativados). Certificado Universal
já ativo cobrindo `layssafaria.com` e `*.layssafaria.com` (o curinga já cobre `app.layssafaria.com`
quando o túnel for criado, sem precisar emitir nada novo).

**Pendente para a próxima sessão:** criar o túnel nomeado no Cloudflare (Zero Trust → Networks → Tunnels)
e configurar o Public Hostname `app.layssafaria.com` — falta decidir como rodar o comando de instalação
do `cloudflared` na VPS, já que o acesso SSH ainda não foi estabelecido (perguntado ao usuário se prefere
rodar ele mesmo e colar a saída, ou configurar uma chave sem passphrase pra eu rodar direto).

**Arquivos alterados:** `memoria/03-pendencias.md`, `memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 4) — Domínio adicionado ao Cloudflare, nameservers trocados

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário criou conta free no Cloudflare, adicionou `layssafaria.com` e revisou os
registros DNS detectados automaticamente (1 A, 3 CNAME — `www`, `ftp`, `mail` —, 1 MX). Orientado a manter
o MX (e-mail já configurado no domínio) e a desligar o proxy (nuvem laranja) dos registros `mail` e `ftp`,
já que o proxy da Cloudflare só funciona para HTTP/HTTPS e quebraria e-mail/FTP se ficasse ligado;
`www`/`A` da raiz mantidos com proxy (tráfego web normal). Depois trocou os nameservers no painel da
Hostgator (usando a opção "Outra plataforma de hospedagem") de `dns3/dns4.hostgator.com.br` para
`jessica.ns.cloudflare.com`/`sid.ns.cloudflare.com` — confirmado com sucesso pela Hostgator, aguardando
propagação/ativação no Cloudflare (até 24h).

**Pendente para a próxima sessão:** confirmar ativação do domínio no Cloudflare (aviso por e-mail), depois
criar o túnel nomeado e o Public Hostname `app.layssafaria.com` — isso ainda depende do acesso SSH à VPS,
que segue pendente.

**Arquivos alterados:** `memoria/03-pendencias.md`, `memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 3) — Domínio oficial comprado: layssafaria.com

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário conversou com a Dra. Layssa, que comprou o domínio **`layssafaria.com`** na
Hostgator (1 ano). Era exatamente o gatilho previsto na decisão D005 para migrar do modo rápido do
Cloudflare Tunnel para um túnel nomeado com domínio fixo. Registrada a decisão **D007** em
`01-decisoes.md` com o passo a passo (Cloudflare, nameservers, túnel nomeado, CNAME).

**Pendente para a próxima sessão:** criar a conta Cloudflare, trocar os nameservers na Hostgator, criar o
túnel nomeado e o CNAME.

**Arquivos alterados:** `memoria/01-decisoes.md` (D007), `memoria/03-pendencias.md`,
`memoria/00-diario-do-projeto.md`.

---

## 2026-09-26 (cont. 2) — Repositório GitHub criado e projeto enviado

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário criou o repositório público `layssafaria-agentes/Deskcodee-Sistema-Layssa-Faria`
no GitHub e pediu para subir todos os arquivos. Gerado um token fine-grained (escopo só desse repo,
Contents: Read and write) guardado temporariamente em `.env` na raiz do projeto (arquivo já coberto pelo
`.gitignore`, separado do `infra/.env` que guarda as chaves da aplicação). Usado para autenticar o
`git push` e removido do `git remote` logo em seguida (URL do remoto ficou limpa, sem token). Branch
local renomeada de `master` para `main` para bater com o padrão do GitHub. Confirmado após o push que
apenas `infra/.env.example` (o template) está versionado — nenhum segredo real subiu.

**Nota:** o usuário pode apagar o `.env` da raiz (ou só o valor do `GITHUB_TOKEN`) e revogar o token no
GitHub, já que não é mais necessário depois do push inicial — só voltaria a ser preciso gerar outro para
um próximo push feito por mim.

**Arquivos alterados:** `memoria/00-diario-do-projeto.md`, `memoria/03-pendencias.md`.

---

## 2026-09-26 (cont.) — Projeto reaberto no disco D, pasta antiga apagada

**Participantes:** Samue + Claude Code

**O que aconteceu:** usuário reabriu o projeto a partir de `D:\Projetos\layssafaria` e apagou a pasta
antiga em `C:\Users\Samue\OneDrive\Área de Trabalho\layssafaria` (confirmado via `Test-Path`). Mudança de
pasta da decisão D006 concluída — não há mais segredos duplicados sincronizando com o OneDrive. Próximo
passo: gerar a chave SSH para a VPS.

**Pendente para a próxima sessão:** testar a conexão SSH com a VPS, configurar `~/.ssh/config`, instalar
`cloudflared`.

---

## 2026-09-26 — VPS confirmada, .env preenchido, projeto saiu do OneDrive

**Participantes:** Samue + Claude Code

**O que aconteceu:**
1. Usuário confirmou que a VPS é de verdade, **Ubuntu 22.04**, e preencheu `infra/.env` com dados reais
   do Supabase (URL, anon key, service role key) e a chave da OpenAI.
2. Usuário perguntou onde guardar com segurança IP/porta/usuário da VPS. Resposta: não no `.env` do
   projeto (categoria de segredo muito mais sensível que chaves de API) — guia criado em
   `docs/infraestrutura.md` usando chave SSH + arquivo `~/.ssh/config` (fora da pasta do projeto).
3. Ao revisar o `.env` preenchido, identificado que **a pasta do projeto sincroniza com o OneDrive**, o
   que expõe os segredos reais na nuvem da Microsoft independente do `.gitignore`. Perguntado ao usuário
   como proceder — optou por mover o projeto pra fora do OneDrive.
4. Usuário sugeriu mover para o disco D:. Confirmado que é uma boa solução (D: não é sincronizado pelo
   OneDrive por padrão) e que existe espaço (668GB livres). Projeto copiado (robocopy, preservando
   histórico do Git) para **`D:\Projetos\layssafaria`** — decisão **D006**.
5. Tentativa de apagar a pasta antiga no OneDrive falhou (estava em uso pela sessão atual do
   editor/terminal). **Ação pendente do usuário:** reabrir o projeto a partir de `D:\Projetos\layssafaria`
   no editor, depois apagar a pasta antiga (ver `03-pendencias.md`).
6. Respondida a pergunta "qualquer domínio serve?": sim, qualquer domínio de qualquer registrador
   funciona com Cloudflare Tunnel/DNS + Let's Encrypt (incluindo `.com.br`), desde que se controle o DNS.

**Arquivos alterados (já em `D:\Projetos\layssafaria`):** `docs/infraestrutura.md`,
`memoria/01-decisoes.md` (D006), `memoria/03-pendencias.md`.

**Pendente para a próxima sessão:** confirmar que o usuário reabriu o projeto no novo caminho, apagar a
pasta antiga do OneDrive, gerar a chave SSH e testar a conexão com a VPS.

---

## 2026-09-25 — Domínio, SSH, .env e segurança do WAHA

**Participantes:** Samue + Claude Code

**O que aconteceu:**
1. Usuário perguntou se dava pra usar o domínio grátis da Vercel. Resposta: não funciona (arquitetura
   roda em Docker/VPS com sessão persistente do WAHA, incompatível com hospedagem serverless). Depois de
   discutir alternativas (subdomínio da agência, DuckDNS, comprar domínio barato), o usuário perguntou
   sobre Cloudflare — confirmado que o **Cloudflare Tunnel** resolve o problema de graça, sem precisar
   de domínio nenhum agora. Decisão registrada em **D005** (`01-decisoes.md`): usar Cloudflare Tunnel,
   sem comprar domínio por enquanto.
2. Documentado passo a passo de como conseguir acesso SSH na VPS Hostgator (painel do cliente, checar
   se é VPS de verdade e não hospedagem compartilhada, IP + senha root, primeiros passos de segurança)
   em [`../docs/infraestrutura.md`](../docs/infraestrutura.md).
3. Criado `infra/.env` (real, protegido pelo `.gitignore`, confirmado via `git check-ignore`) e
   `infra/.env.example` (template seguro para versionar) — usuário vai preencher as chaves (OpenAI,
   Supabase) direto no arquivo, sem colar no chat.
4. Usuário confirmou que vai buscar as informações da clínica (serviços, preços, etc.) diretamente com
   a Dra. Layssa, sem previsão de data.
5. Pesquisado a segurança e o custo real do WAHA (pergunta direta do usuário): software confiável, risco
   é o uso não-oficial do protocolo do WhatsApp (mesmo trade-off da decisão D002), e o custo real é
   ~US$5/mês (tier "Community", não mais "Plus" a ~US$19/mês como a documentação antiga sugeria).
   Detalhes em `02-pesquisa-deskcommcrm.md`.

**Arquivos criados/alterados:** `infra/.env`, `infra/.env.example`, `docs/infraestrutura.md`,
`memoria/01-decisoes.md` (D005), `memoria/02-pesquisa-deskcommcrm.md`, `memoria/03-pendencias.md`.

**Pendente para a próxima sessão:** acesso SSH à VPS, chave OpenAI, conta Supabase, decidir número de
WhatsApp dedicado, instalar `cloudflared` assim que houver acesso à VPS.

---

## 2026-09-24 — Kickoff do projeto

**Participantes:** Samue (marketing/gestão) + Claude Code

**O que aconteceu:**
1. Definido o objetivo geral: criar um sistema de agentes de IA para a clínica da Dra. Layssa Faria
   (implantodontista) que funcione como recepcionista, vendedora e assistente de agendamento via WhatsApp.
2. Pesquisado o projeto **DeskcommCRM** (github.com/melgarafael/DeskcommCRM) para confirmar que é real e
   entender a arquitetura. Resultado: projeto legítimo e ativo (criado por Rafael Melgaço, conta desde
   2022, ~3.700 stars, MIT license, commits diários). Detalhes completos em
   [`02-pesquisa-deskcommcrm.md`](02-pesquisa-deskcommcrm.md).
3. Pesquisado o "Jev" mencionado pelo usuário como orquestrador em alta. Descoberta: não é um orquestrador
   de agentes — é um modelo de decisão tipada (System One) da TypeSafe AI, lançado em 15/09/2026,
   hospedado e cobrado por token, ainda em early access. Decisão: fora do escopo da v1.
4. Criada a estrutura de pastas do projeto (`memoria/`, `docs/`, `clinica/`, `infra/`) e este diário.
5. Perguntas de arquitetura respondidas pelo usuário:
   - Canal WhatsApp: começar com **WAHA** (QR Code), migrar para Meta Cloud API depois se o volume
     justificar o processo de verificação.
   - Infraestrutura: usuário **já tem parte** (VPS Hostgator contratada), mas não confirmou ainda
     domínio e chave da OpenAI — precisa detalhar o que falta.
   - Jev: confirmado que fica **fora da v1**.

**Pendente para a próxima sessão:**
- Confirmar com o usuário exatamente o que falta de infra (domínio? conta Supabase? chave OpenAI?).
- Coletar informações reais da clínica (serviços, preços, horários, endereço, convênios) para preencher
  [`clinica/base-conhecimento.md`](../clinica/base-conhecimento.md).
- Ver [`03-pendencias.md`](03-pendencias.md) para lista completa.

**Arquivos criados nesta sessão:**
- `README.md`
- `memoria/00-diario-do-projeto.md` (este arquivo)
- `memoria/01-decisoes.md`
- `memoria/02-pesquisa-deskcommcrm.md`
- `memoria/03-pendencias.md`
- `docs/arquitetura.md`
- `docs/agentes-ia.md`
- `docs/infraestrutura.md`
- `clinica/base-conhecimento.md`
