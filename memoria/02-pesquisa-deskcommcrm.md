# Pesquisa: DeskcommCRM

Notas de pesquisa sobre o projeto que vamos usar como base. Fonte: repositório público no GitHub
(`melgarafael/DeskcommCRM`), README oficial, árvore de arquivos e `.env.example` — consultados em
2026-09-24.

## O que é

CRM de vendas open source (licença **MIT**), self-hosted, com agentes de IA nativos que conversam por
WhatsApp, qualificam leads e os movem por um funil/pipeline. Alternativa aberta a Kommo, Octadesk e
Intercom. Site: https://deskcomm.com.br

## Legitimidade (checado porque o padrão de busca inicial parecia suspeito)

Ao buscar "DeskcommCRM" no Google, apareceram **dezenas de forks** (peubraw, rafaelcesardev,
lucasgiovannibr, victorrabyfs, jessefreitas, raphaelmartins, Vellyx-Crm, gideony...) todos com a
descrição idêntica — isso pareceu, à primeira vista, uma campanha de spam/SEO coordenada. Investigação:
- Repositório principal (`melgarafael/DeskcommCRM`): criado em 2026-04-28, **3.683 stars**, 919 forks,
  146 issues abertas, push mais recente no dia da pesquisa (24/09/2026), 203MB, linguagem principal
  TypeScript, license MIT.
- Dono (`melgarafael` = Rafael Melgaço): conta criada em **dezembro de 2022** (conta antiga e real, não
  descartável), 144 seguidores, 18 repositórios públicos.
- **Conclusão:** os "forks" são forks normais do GitHub (por isso a descrição é idêntica — é copiada
  automaticamente ao dar fork). Não há indício de campanha falsa. Projeto legítimo.

Vale manter esse hábito: sempre que formos instalar algo em produção (VPS real, dados de clientes),
checar idade da conta do mantenedor, histórico de commits e ler o script de instalação antes de rodar.

## Stack técnica

| Camada | Tecnologia |
|---|---|
| Frontend | Next.js 16 (App Router), React 19, Tailwind, shadcn/ui |
| Banco de dados | Postgres via Supabase (Row Level Security + pgvector para embeddings/RAG) |
| Auth | Supabase Auth (cookies HttpOnly, MFA opcional) |
| WhatsApp | WAHA Plus (QR Code, multi-número) **ou** Meta Cloud API (oficial) |
| IA | Vercel AI SDK v7 — troca de provedor (OpenAI/Anthropic/OpenRouter) pela tela, sem redeploy |
| Storage | Supabase Storage (buckets privados, URLs assinadas) — mídia do WhatsApp |
| Rate limiting | Upstash Redis (serverless) |
| Observabilidade | Sentry (dados sensíveis removidos, opt-in) |
| Voz | Há um "voice-agent" e integração SIP/Asterisk (ARI, AudioSocket) — telefonia/chamadas de voz com IA |
| E-commerce | Integração com Nuvemshop (não relevante para a clínica, mas existe) |
| Calendário | Google Calendar (relevante para agendamento de consultas) |

## Como se instala

Requisitos: VPS com Docker (mínimo 4GB RAM recomendado — parceria oficial com Hostgator), domínio com
registro A apontando pro IP da VPS, conta Supabase (tier free serve), chave de API de IA (OpenAI,
Anthropic ou OpenRouter), número de WhatsApp.

```bash
# instalador guiado (não é curl|bash direto para instalação — é um wizard que faz perguntas)
curl -fsSL https://raw.githubusercontent.com/melgarafael/DeskcommCRM/main/hostgator-setup-kit/comecar.sh | bash

# ou manual:
git clone https://github.com/melgarafael/DeskcommCRM.git
cd DeskcommCRM
bash hostgator-setup-kit/install.sh
```

**Revisão de segurança do `comecar.sh`:** é um wizard interativo — pergunta a situação atual do usuário
(sem servidor / servidor remoto / já está no servidor), não mexe em arquivos de sistema nem cria
usuários por conta própria, faz checagem de hostname antes de instalar, e só executa o `install.sh`
(esse sim faz a instalação de fato) mediante confirmação explícita. Antes de rodar o `install.sh` na VPS
real da clínica, vamos ler o conteúdo dele linha a linha (ainda não fizemos essa leitura completa —
fazer isso na sessão de deploy).

Passos que o instalador cobre: Docker, extensões do Postgres, criação do usuário admin, HTTPS
(certificado em ~1 min), instalação do cron de automações.

## Variáveis de ambiente relevantes para o nosso caso (`.env.example`)

- **Supabase:** `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`,
  `SUPABASE_DB_URL`
- **IA:** `OPENAI_API_KEY` (a que vamos usar), também suporta `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`;
  `AI_BUDGET_ENFORCEMENT=on` por padrão (proteção contra gasto descontrolado com IA — manter ligado)
- **WhatsApp WAHA:** `WAHA_API_BASE_URL`, `WAHA_API_KEY`, `WAHA_HMAC_SECRET`,
  `WAHA_WEBHOOK_REQUIRE_SIGNATURE`
- **WhatsApp Meta Cloud API** (para migração futura): `META_APP_ID`, `META_APP_SECRET`, `META_WABA_ID`,
  `META_GRAPH_VERSION`
- **Google Calendar:** `GOOGLE_CALENDAR_CLIENT_ID` (relevante para agendamento de consultas)
- **LGPD:** `LGPD_SIGNING_KEY`, `LGPD_DPO_EMAIL`, `LGPD_EXPORT_EXPIRES_HOURS` (72h) — relevante porque
  dados de saúde/odontológicos são dado sensível pela LGPD, então vale configurar isso com atenção real
  (definir quem é o DPO/encarregado de dados da clínica)
- **Retenção de dados:** `LEAD_CAPTURE_RETENTION_DAYS` (365), `AUDIT_LOG_RETENTION_DAYS` (1825)
- **Branding:** `APP_NAME`, `APP_LOGO_URL`, `APP_ACCENT_HEX`, `APP_LOCALE=pt-BR` (dá pra deixar com a
  identidade visual da clínica)
- **Backups:** Supabase free tier **não tem backup automático** — o próprio projeto fornece um
  `backup.sh` que precisa ser agendado via cron manualmente. **Ação necessária no deploy: não esquecer
  de configurar esse cron.**

## Estrutura de "agentes" no próprio repositório (skills)

O repo tem pastas `.agents/skills/` e `.claude/skills/` com skills prontas para quem for *desenvolver
em cima* do DeskcommCRM (não é a IA que atende o cliente da clínica, é tooling para nós mexermos no
código/configuração):

- `deskcomm-cliente-novo` — onboarding de cliente novo, workflows por nicho, importação de arquivos,
  prompts de agente, lógica de triagem. **Provavelmente o ponto de partida certo para configurar a
  clínica da Dra. Layssa.**
- `deskcomm-instalar` — domínios, DNS, setup do Supabase, troubleshooting de instalação
- `deskcomm-prompt` — engenharia de prompt (anatomia de prompt, ferramentas de diagnóstico) — útil na
  hora de escrever os prompts dos agentes de venda/secretária
- `deskcomm-metricas` — métricas e controles de acesso LGPD
- `deskcomm-extensao` — gerenciamento de pacotes de extensão
- `sistema-vivo` — sistema de follow-up automático (reengajamento de leads frios) — **muito relevante**
  para reativar pacientes que sumiram no meio do orçamento
- `deskcomm-doutrina` — princípios/filosofia do projeto
- `deskcomm-contribuir` — fluxo de contribuição para quem quiser mandar PR pro projeto

Também existem specs de handoff na raiz do repo sobre: conversa virando lead, automação de follow-up
(sistema Vivo), fila de atendimento ("W1 filing"), protocolo de notificação de leads.

## Perguntas em aberto sobre o DeskcommCRM (investigar na próxima sessão técnica)

- Ler o `install.sh` linha a linha antes do primeiro deploy real.
- Entender o modelo de "multi-tenant": para uma clínica só (não vamos revender o CRM), provavelmente
  usamos um único tenant — confirmar se isso simplifica algo na instalação.
- Entender exatamente como a skill `deskcomm-cliente-novo` estrutura o onboarding, para seguirmos o
  caminho oficial em vez de reinventar.
- Ver se existe algum limite/custo extra do WAHA Plus (o README menciona "WAHA Plus" — confirmar se é
  a versão paga do WAHA ou se está incluído).
