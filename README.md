# Projeto Layssa Faria — Clínica de Implantodontia com Agentes de IA

Sistema de agentes de IA para a clínica da **Dra. Layssa Faria** (implantodontista), construído sobre o
[DeskcommCRM](https://github.com/melgarafael/DeskcommCRM) (open source, MIT), com objetivo de:

- Atender e vender via WhatsApp com IA (recepção, tira-dúvidas, qualificação de leads);
- Agendar e confirmar consultas automaticamente (secretária virtual);
- Fazer follow-up de leads frios e pós-venda;
- Rodar 100% self-hosted em VPS Hostgator, com dados sob controle da clínica (LGPD).

## Status atual

🟡 **Fase 1 — Planejamento e preparação.** Ainda não há deploy em produção.
Veja o estado detalhado em [`memoria/03-pendencias.md`](memoria/03-pendencias.md).

## Como navegar neste repositório

| Pasta | Conteúdo |
|---|---|
| [`memoria/`](memoria/) | Diário do projeto, decisões tomadas e o porquê, pesquisa sobre o DeskcommCRM, pendências em aberto. **Leia isso primeiro para retomar o contexto.** |
| [`docs/`](docs/) | Arquitetura do sistema, especificação dos agentes de IA, plano de infraestrutura. |
| [`clinica/`](clinica/) | Base de conhecimento da clínica (serviços, preços, horários, FAQ) — usada para treinar os agentes. |
| [`infra/`](infra/) | Scripts, `.env` de referência e notas de deploy da VPS. |

## Stack escolhida

- **CRM + agentes de IA + WhatsApp:** [DeskcommCRM](https://github.com/melgarafael/DeskcommCRM) (Next.js + Supabase + WAHA + Vercel AI SDK)
- **LLM:** OpenAI API
- **Canal WhatsApp:** WAHA (QR Code) na v1, com migração futura para Meta Cloud API se o volume justificar
- **Hospedagem:** VPS Hostgator (parceria oficial do DeskcommCRM)
- **Orquestração entre sistemas** (ex.: futura integração com Ileva/Bitrix24): n8n, se necessário — não decidido ainda
- **Jev (TypeSafe AI):** fora do escopo da v1 (produto muito recente, hospedado, early-access — ver [`memoria/01-decisoes.md`](memoria/01-decisoes.md))

## Próximos passos

Ver [`memoria/03-pendencias.md`](memoria/03-pendencias.md).
