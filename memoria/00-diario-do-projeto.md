# Diário do Projeto

Registro cronológico de tudo que foi feito, decidido e descoberto. Toda sessão de trabalho deve
adicionar uma entrada nova no topo (mais recente primeiro).

> **Nota de localização (2026-09-26):** o projeto vive agora em `D:\Projetos\layssafaria` (antes estava
> em `OneDrive\Área de Trabalho\layssafaria`). Ver decisão D006.

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
