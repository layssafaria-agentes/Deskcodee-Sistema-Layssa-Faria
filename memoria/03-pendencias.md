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
- [x] **Número de WhatsApp definido em 2026-09-26: +55 64 99626-2769** (número que a clínica já usa,
      não um número novo dedicado — usuário decidiu conscientemente, mesmo depois de avisado do risco
      teórico de bloqueio por automação num número com histórico real; mitigar seguindo as boas práticas
      já documentadas: não fazer disparo em massa pra quem nunca falou com a clínica, aquecer aos poucos).
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
- [x] **Revisão de segurança concluída em 2026-09-26.** Lidos por completo
      `ubuntu-production-installer.sh` e `hostgator-setup-kit/install-single-server.sh`; varredura por
      padrão (rede externa, comandos destrutivos, enfraquecimento de firewall/permissões, exfiltração)
      em `_common.sh`, `install.sh`, `agent.sh`, `backup.sh`. Nada suspeito encontrado — instalador oficial
      do Supabase é baixado e conferido por **SHA-256** antes de rodar, senhas geradas aleatoriamente com
      arquivo `chmod 600`, backup com verificação de integridade (gzip -t, checa sessão não-vazia), o
      único endpoint que recebe POST é o próprio app (fila interna de automações), sem telemetria externa.
      Aprovado para rodar em produção.

## Mudança de pasta (concluída)

- [x] **Projeto reaberto a partir de `D:\Projetos\layssafaria` e pasta antiga do OneDrive apagada** —
      confirmado em 2026-09-26. Não há mais `.env` duplicado sincronizando com a nuvem.

## Depois que o bloqueado acima estiver resolvido

- [x] **DeskcommCRM instalado e no ar em 2026-09-26:** https://app.layssafaria.com respondendo,
      login funcionando (Deskcomm CRM). Admin: `admin@app.layssafaria.com` (senha no arquivo protegido
      `deskcommcrm/.runtime/admin-credentials` na VPS, chmod 600 — troque assim que logar). Ver
      `00-diario-do-projeto.md` para os 3 problemas resolvidos na instalação (validador de chave,
      certificado do Caddy, confiança de TLS do app/worker).
- [x] Cron do backup configurado (ver entrada de 2026-09-26 mais acima).
- [x] **WhatsApp conectado em 2026-09-27:** número `+55 62 98199-1595` (do Samue, **número de teste** —
      não é o `+55 64 99626-2769` da clínica ainda) com status Conectado (via QR Code). IA em modo de
      teste (nenhum número autorizado ainda — proposital, trava de segurança até terminarmos a
      configuração).
- [ ] **Trocar pelo número real da clínica quando estiver tudo pronto:** conectar
      `+55 64 99626-2769` em Conexões, e só então configurar "Aviso no WhatsApp" com o número do Samue
      como destinatário (o sistema não deixa usar o mesmo número nos dois papéis — por isso está
      bloqueado agora, testando com o próprio número do Samue).
- [x] **Credencial de IA já validada:** "Chave do onboarding" (OpenAI/GPT), cadastrada automaticamente
      durante o onboarding inicial — não precisou de ação extra.
- [x] **Funil "Agendamentos" configurado em 2026-09-27** seguindo o pacote oficial de clínica: 7 etapas
      (Novo contato → Já respondi → Entendendo o caso → Quer agendar → Escolhendo horário → Consulta
      marcada [ganho] → Não vai marcar [perdido]), vocabulário (paciente/consulta/marcada/não marcou).
- [x] **Follow-ups instalados (rascunho) em 2026-09-27:** "Falta" (remarcar quem não veio) e "Consulta"
      (retomar quem sumiu na marcação) — via galeria de modelos do produto. **Ainda não publicados**
      (falta revisar o texto e ligar no agente quando ele existir).
- [ ] **Follow-ups "Exame" e "Cirurgia" não instalados** — pedem uma etapa do funil como gatilho e nosso
      funil (só vai até a avaliação inicial) não tem uma etapa de "aguardando exame"/"decidindo cirurgia".
      Decidir com a Dra. Layssa se o processo dela precisa dessas etapas extras.
- [x] **Agente "Recepção" rascunhado em 2026-09-27:** número conectado, funil "Agendamentos" ligado,
      20 de 25 capacidades ativas (Atender e responder completo; Desmarcar um compromisso; Encerrar o
      negócio como ganho/perdido; Retomar o atendimento automático), follow-ups Falta+Consulta armados,
      palavras de handoff padrão. Prompt ainda com placeholder `[Dra.Layssa Faria]` — falta o texto final
      da doutora. **Testado em modo sandbox (dry-run) com 2 cenários do roteiro oficial: reconheceu dor
      forte/urgência (chamou humano + alerta de sinal de emergência por conta própria) e pediu remarcação
      corretamente (sem inventar dado); todos os portões de segurança passaram (`agenda_stall` incluso).**
      Ainda em rascunho v1, não publicado.
- [x] **Teto de gasto de IA configurado em 2026-09-27:** R$30/mês, `enforcement_mode: bloquear` (bloqueia
      de verdade ao estourar, não só avisa), alarme em 80%. Descoberta no caminho: existe um campo
      separado "o que fazer ao bater o teto" que precisa ser mudado de "Desligado" — só colocar o valor
      não ativa a trava sozinho.
- [ ] **Quem recebe o aviso de handoff (humano) ainda não decidido** — perguntado ao usuário, sem
      resposta ainda (provavelmente ele mesmo por enquanto, até a Dra. Layssa organizar a equipe dela).
- [ ] **Questionário enviado pra Dra. Layssa em 2026-09-27** (`clinica/questionario-dra-layssa.md` +
      PDF) — aguardando resposta. Bloqueia: base de conhecimento, o prompt final dos 3 agentes
      (`docs/agentes-ia.md`), memória da organização (regras da casa), e publicar qualquer coisa.
- [ ] Cadastrar uma chave de IA **da Anthropic** (opcional — hoje só tem OpenAI) se quisermos usar Claude
      em algum ponto de uso específico.
- [ ] Configurar SMTP (e-mail) — hoje "esqueci a senha"/confirmação de cadastro não enviam e-mail.
- [ ] Configurar Google Calendar para agendamento automático.
- [ ] Testar o fluxo ponta a ponta com números de teste antes de liberar para clientes reais (e antes
      disso, autorizar números de teste em Conexões → Configurar acesso da IA).
- [x] **Repositório no GitHub criado e projeto enviado em 2026-09-26:**
      https://github.com/layssafaria-agentes/Deskcodee-Sistema-Layssa-Faria (branch `main`). Push feito
      via token fine-grained temporário (escopo Contents: read/write só desse repo), removido do
      `git remote` logo depois de usado. Confirmado que só `infra/.env.example` foi versionado — os
      `.env` reais (app e o token do GitHub) continuam fora do Git.
