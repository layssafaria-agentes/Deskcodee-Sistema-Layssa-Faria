# Pendências e Próximos Passos

Lista viva do que falta para avançar. Marcar `[x]` quando resolvido e mover para o diário
([`00-diario-do-projeto.md`](00-diario-do-projeto.md)) com a data.

## Bloqueadores para o deploy (precisam de resposta/ação do usuário)

- [x] **Domínio:** resolvido em 2026-09-25 — não vamos comprar domínio agora, vamos usar Cloudflare
      Tunnel (ver decisão D005 em `01-decisoes.md`). Comprar domínio fica para quando a Dra. Layssa
      decidir.
- [x] **VPS confirmada em 2026-09-26:** é uma VPS de verdade, Ubuntu 22.04.
- [x] **Acesso SSH estabelecido em 2026-09-26.** Porta SSH da Hostgator é **22022** (não a 22 padrão —
      causou um "connection timed out" até descobrirmos isso no painel). Usuário `deskcomm` criado com
      sudo, chave ed25519 **sem passphrase** (`~/.ssh/layssafaria_vps` no PC do Samue) configurada em
      `authorized_keys`, alias `layssafaria-vps` em `~/.ssh/config`. Escolha explícita do usuário: chave
      sem passphrase pra permitir que o Claude Code rode comandos direto na VPS sem interação (opção B
      discutida — troca segurança por praticidade, é uma decisão consciente, ver diário). Firewall `ufw`
      ativo liberando 22022/80/443. **Sudo sem senha liberado pro `deskcomm` (decisão D008)** — reavaliar/
      apertar antes de produção com dado real de paciente.
- [x] **Chave da API OpenAI:** preenchida pelo usuário em `infra/.env` em 2026-09-26.
- [x] **Supabase:** URL, anon key e service role key preenchidas pelo usuário em `infra/.env` em
      2026-09-26. Falta só o `SUPABASE_DB_URL` (connection string do Postgres).
- [ ] **Número de WhatsApp dedicado:** a clínica vai usar um número novo só para a IA, ou o número que já
      usa hoje? (Recomendado: número novo, para não misturar histórico/contatos pessoais com o agente.)
      Ainda não perguntado/respondido.
- [ ] **Cloudflare Tunnel:** instalar o `cloudflared` na VPS assim que o SSH estiver disponível.
- [ ] **Domínio `layssafaria.com` comprado (2026-09-26, Hostgator, 1 ano) — ver decisão D007.** Progresso:
      1. [x] conta free criada no Cloudflare, domínio adicionado, registros DNS revisados (mantido MX;
         desativado proxy em `mail` e `ftp` por serem protocolos não-HTTP; `www`/`A` da raiz mantidos com
         proxy);
      2. [x] nameservers trocados no painel da Hostgator para os do Cloudflare, **domínio ativo/protegido
         pela Cloudflare confirmado em 2026-09-26** (propagou em poucas horas, não precisou das 24h);
         SSL/TLS configurado: modo "Completo", TLS mínima 1.2, "Sempre usar HTTPS" ativado;
      3. [x] **Túnel `clinica-layssa` criado via Zero Trust (cartão cadastrado pra ativar o produto, sem
         custo no plano free) e instalado na VPS em 2026-09-26** — serviço `cloudflared` rodando via
         systemd, conectado ao edge de São Paulo (`gru13`). Public Hostname configurado:
         `app.layssafaria.com` → `localhost:3000` (porta é um chute baseado em Next.js padrão — ajustar
         se o instalador do DeskcommCRM expuser outra). Confirmado com `curl`: domínio responde **HTTP
         502** (esperado — túnel/DNS/SSL funcionando de ponta a ponta, só falta a aplicação rodando);
      4. [x] `app.layssafaria.com` já é o `NEXT_PUBLIC_APP_URL` a usar quando instalarmos o DeskcommCRM
         (raiz do domínio reservada pro futuro site institucional).
- [x] **Domínio — "qualquer um serve?" respondido em 2026-09-26:** sim, qualquer domínio de qualquer
      registrador funciona (incluindo `.com.br`), desde que se controle o DNS dele. Não é bloqueador.

## Informações da clínica (para a base de conhecimento dos agentes)

- [ ] Preencher [`clinica/base-conhecimento.md`](../clinica/base-conhecimento.md) com dados reais:
      serviços/procedimentos oferecidos, faixa de preço ou política de "sob consulta", horário de
      funcionamento, endereço, convênios/planos aceitos, formas de pagamento, diferenciais da Dra.
      Layssa (formação, especializações, tempo de experiência), fotos/depoimentos que possam virar prova
      social no atendimento. **Status (2026-09-25): usuário vai buscar essas informações direto com a
      Dra. Layssa** — sem previsão de data ainda.

## Decisões técnicas ainda abertas

- [x] **WAHA — segurança e custo confirmados em 2026-09-25:** o software em si é confiável (projeto
      open source popular, `devlikeapro/waha`, sem malware/red flags). O risco real não é o software,
      é o **ToS do WhatsApp**: WAHA automatiza o WhatsApp Web de um jeito não-oficial, então existe
      risco (baixo se usado com bom senso) de bloqueio do número — mitigar não fazendo disparo em massa
      para quem nunca falou com a clínica, aquecendo o número aos poucos, respeitando opt-out. Sobre
      custo: desde a v2026.6.1 não existe mais separação Core/Plus — o que era "Plus" (mídia, múltiplas
      sessões) está incluído a partir do tier pago **"Community" (~US$5/mês)**. Atualizado em
      `docs/infraestrutura.md` (orçamento) e `memoria/02-pesquisa-deskcommcrm.md`.
- [ ] Definir se vamos integrar com o Ileva (sistema de gestão) nesta fase ou deixar para uma fase 2 —
      hoje não está claro se o Ileva tem API pública para esse tipo de integração.
- [ ] Ler `deskcommcrm/ubuntu-production-installer.sh` e
      `deskcommcrm/hostgator-setup-kit/install-single-server.sh` linha a linha antes do primeiro deploy
      (checklist de segurança — nomes atualizados em 2026-09-26 após vendorizar o código, ver
      `docs/infraestrutura.md`).

## Mudança de pasta (concluída)

- [x] **Projeto reaberto a partir de `D:\Projetos\layssafaria` e pasta antiga do OneDrive apagada** —
      confirmado em 2026-09-26. Não há mais `.env` duplicado sincronizando com a nuvem.

## Depois que o bloqueado acima estiver resolvido

- [ ] Rodar o instalador guiado do DeskcommCRM na VPS.
- [ ] Configurar o cron do `backup.sh` (Supabase free tier não tem backup automático).
- [ ] Escrever e revisar os prompts dos 3 agentes definidos em
      [`docs/agentes-ia.md`](../docs/agentes-ia.md) (Recepção, Vendas/Qualificação, Follow-up).
- [ ] Conectar WAHA com o número de WhatsApp escolhido (QR Code).
- [ ] Configurar Google Calendar para agendamento automático.
- [ ] Testar o fluxo ponta a ponta com números de teste antes de liberar para clientes reais.
- [x] **Repositório no GitHub criado e projeto enviado em 2026-09-26:**
      https://github.com/layssafaria-agentes/Deskcodee-Sistema-Layssa-Faria (branch `main`). Push feito
      via token fine-grained temporário (escopo Contents: read/write só desse repo), removido do
      `git remote` logo depois de usado. Confirmado que só `infra/.env.example` foi versionado — os
      `.env` reais (app e o token do GitHub) continuam fora do Git.
