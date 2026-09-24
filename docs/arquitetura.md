# Arquitetura do Sistema

## Visão geral

```
Paciente/Lead (WhatsApp)
        │
        ▼
   WAHA (conexão QR Code)  ──── migração futura ───▶  Meta Cloud API (oficial)
        │
        ▼
   DeskcommCRM (Next.js, na VPS Hostgator)
        │
        ├── Motor de Agentes de IA (Vercel AI SDK + OpenAI)
        │      ├── Agente Recepção/Secretária ─── ver docs/agentes-ia.md
        │      ├── Agente Vendas/Qualificação
        │      └── Agente Follow-up (sistema "Vivo")
        │
        ├── Pipeline de Leads / CRM (kanban de funil de vendas)
        │
        ├── Automações WHEN/IF/THEN (tags, troca de etapa, notificações)
        │
        ├── Integração Google Calendar (agendamento de consultas)
        │
        └── Banco de dados (Postgres via Supabase, com pgvector p/ RAG)
                  │
                  ▼
        Base de Conhecimento da Clínica (clinica/base-conhecimento.md → importada como RAG)
```

## Componentes e onde rodam

| Componente | Onde roda | Observação |
|---|---|---|
| DeskcommCRM (app principal) | VPS Hostgator (Docker) | Next.js, front + back |
| WAHA | VPS Hostgator (Docker, container próprio) | Conexão WhatsApp via QR Code |
| Worker/Scheduler | VPS Hostgator (Docker) | Processa fila de automações e follow-ups |
| Postgres + Auth + Storage | Supabase (cloud, tier free) | Banco de dados gerenciado |
| Redis (rate limiting) | Upstash (cloud, serverless) | Opcional, mas recomendado |
| LLM | OpenAI (API) | Chamado pelo Vercel AI SDK a partir da VPS |
| Google Calendar | Google Cloud (OAuth) | Agendamento de consultas |
| Domínio + HTTPS | Registrador do domínio + Let's Encrypt (via instalador) | — |

**Nota sobre soberania de dados:** mesmo sendo "self-hosted", o banco de dados por padrão fica no
Supabase (nuvem), não na própria VPS. Isso é aceitável para a maioria dos casos (Supabase tem
certificações de segurança), mas vale deixar registrado: os dados dos pacientes não ficam fisicamente
só na VPS da clínica, ficam também no Supabase. Se isso for um problema para a Dra. Layssa (por
exemplo, por exigência de algum convênio ou política interna), existe a opção de rodar Postgres
self-hosted na própria VPS em vez de usar Supabase — decisão a discutir se necessário.

## Papéis dos "agentes de IA" (funcionários virtuais)

Ver detalhamento completo em [`agentes-ia.md`](agentes-ia.md). Resumo:

1. **Recepção/Secretária** — primeiro contato, tira dúvidas gerais, agenda e confirma consultas.
2. **Vendas/Qualificação** — conversa sobre procedimentos (ex.: implante), valores, condições, qualifica
   o lead e passa para avaliação presencial.
3. **Follow-up (Vivo)** — reengaja leads que pararam de responder, lembra de consultas, pede avaliação
   pós-atendimento.

Na prática, isso pode ser **um único agente com contexto e instruções diferentes por etapa do funil**
(mais simples de manter) ou **três agentes especializados com handoff entre si** (mais controle, mais
complexo). Recomendação inicial: começar com um agente único bem instruído, dividir em agentes separados
só se a complexidade justificar — evita over-engineering antes de validar o produto com uso real.

## Automação entre sistemas (Ileva, Bitrix24, etc.)

Ainda não decidido se é necessário nesta fase (ver pendência em
[`memoria/03-pendencias.md`](../memoria/03-pendencias.md)). Se for preciso sincronizar agendamentos ou
dados de pacientes com o Ileva, o caminho natural é usar **n8n** (já usado internamente pela empresa mãe
para o Auto América) como ponte, escutando webhooks do DeskcommCRM e chamando a API do Ileva (se
existir). Isso fica para uma fase 2 — não é bloqueador para o lançamento inicial do agente de WhatsApp.

## Segurança e LGPD

- Dados de saúde/odontológicos são **dado sensível** pela LGPD — atenção redobrada.
- Definir quem é o encarregado de dados (DPO) da clínica → variável `LGPD_DPO_EMAIL`.
- Ativar e revisar a política de retenção de dados (`LEAD_CAPTURE_RETENTION_DAYS`,
  `AUDIT_LOG_RETENTION_DAYS`) de acordo com o que fizer sentido para uma clínica.
- Nunca commitar `.env` com chaves reais no Git (ver `.gitignore` na raiz do projeto).
- Configurar backup periódico do banco (`backup.sh` via cron) — Supabase free tier não faz isso sozinho.
