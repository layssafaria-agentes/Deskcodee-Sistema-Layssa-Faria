# Diário do Projeto

Registro cronológico de tudo que foi feito, decidido e descoberto. Toda sessão de trabalho deve
adicionar uma entrada nova no topo (mais recente primeiro).

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
