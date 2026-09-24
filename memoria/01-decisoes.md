# Decisões de Arquitetura (ADR leve)

Cada decisão importante entra aqui, com data, o que foi decidido, alternativas consideradas e o porquê.
Se uma decisão mudar depois, não apague a antiga — adicione uma nova entrada dizendo que substitui a anterior.

---

## D001 — Base do sistema: DeskcommCRM

**Data:** 2026-09-24 · **Status:** Aprovado

Usar o [DeskcommCRM](https://github.com/melgarafael/DeskcommCRM) (MIT) como base do CRM + motor de
agentes de IA + integração WhatsApp, em vez de construir do zero ou usar SaaS pago (Kommo, Octadesk,
Intercom).

**Por quê:** é open source, self-hosted (dados da clínica ficam sob controle dela — importante para
dados de saúde/LGPD), já tem parceria oficial com Hostgator (VPS que o usuário já possui), já integra
WhatsApp (WAHA + Meta Cloud API), Google Calendar, e tem um motor de automação (WHEN/IF/THEN) e um
sistema de follow-up ("Vivo") prontos — economiza meses de desenvolvimento.

**Alternativas consideradas:** construir agente do zero com Vercel AI SDK/LangGraph direto (mais
flexível, porém reconstruiria CRM, pipeline de leads, integração WhatsApp e compliance LGPD do zero).

---

## D002 — Canal de WhatsApp: WAHA na v1

**Data:** 2026-09-24 · **Status:** Aprovado

Começar com **WAHA** (conexão via QR Code, número de WhatsApp comum), e não com Meta Cloud API.

**Por quê:** é o caminho padrão e mais rápido do DeskcommCRM, não exige verificação de empresa na Meta
nem aprovação de templates. Permite validar o produto com a Dra. Layssa rapidamente.

**Trade-off aceito:** risco (baixo, mas existente) de bloqueio do número pelo WhatsApp se houver disparo
em massa ou uso fora das políticas. Mitigação: seguir boas práticas de uso (não fazer disparo em massa
para números que não iniciaram conversa, respeitar opt-out, etc.) — detalhar em
[`docs/infraestrutura.md`](../docs/infraestrutura.md).

**Gatilho para migrar para Meta Cloud API:** quando o volume de conversas ou a dependência do número
único justificar o esforço de verificação de negócio na Meta.

---

## D003 — Jev (TypeSafe AI) fica fora da v1

**Data:** 2026-09-24 · **Status:** Aprovado

Não usar o Jev como camada de orquestração/decisão no sistema inicial.

**Por quê:** é um produto lançado há poucos dias (15/09/2026), somente hospedado (não self-hosted, o que
destoa um pouco da filosofia de soberania de dados do projeto), cobrado por token, e ainda em
"early-access" — ou seja, sem histórico de estabilidade em produção. O próprio motor de agentes do
DeskcommCRM (Vercel AI SDK + OpenAI) já cobre roteamento e decisões do fluxo inicial.

**Reavaliar quando:** o Jev tiver mais maturidade em produção, ou quando surgir um caso de uso concreto
de roteamento/decisão que o motor atual do DeskcommCRM não resolva bem.

---

## D004 — LLM provider: OpenAI

**Data:** 2026-09-24 · **Status:** Aprovado

Usar a API da OpenAI como provedor de LLM (o DeskcommCRM suporta OpenAI, Anthropic e OpenRouter
intercambiáveis via Vercel AI SDK, então essa escolha pode ser revisitada facilmente trocando a variável
de ambiente, sem re-arquitetar nada).

**Por quê:** pedido explícito do usuário.
