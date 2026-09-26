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

---

## D005 — Domínio: usar Cloudflare Tunnel, sem comprar domínio agora

**Data:** 2026-09-25 · **Status:** Aprovado

Não registrar/comprar domínio nesta fase. Em vez disso, expor a VPS via **Cloudflare Tunnel**
(`cloudflared`), que fornece uma URL pública com HTTPS válido de graça (modo rápido:
`*.trycloudflare.com`; modo estável: túnel nomeado, sem precisar de domínio próprio).

**Por quê:** o usuário perguntou se dava pra usar o domínio grátis da Vercel — não funciona, porque a
arquitetura roda em Docker numa VPS (sessão persistente do WAHA), incompatível com hospedagem
serverless (Vercel ou Cloudflare Pages/Workers têm a mesma limitação). O Cloudflare Tunnel resolve o
problema real por trás da pergunta (ter uma URL pública com HTTPS sem custo) sem essa incompatibilidade.

**Trade-off aceito:** a URL do modo rápido muda a cada reinício do túnel — ok para testes internos, mas
antes de divulgar para pacientes de verdade, migrar para um túnel nomeado (ainda sem custo). Integrações
que exigem callback estável (ex.: OAuth do Google Calendar) só devem ser configuradas depois dessa
migração.

**Quando revisitar:** quando a Dra. Layssa comprar o domínio oficial da clínica — nesse momento só se
adiciona o domínio no Cloudflare e aponta pro túnel nomeado, sem precisar reinstalar o DeskcommCRM.

---

## D006 — Projeto movido para fora do OneDrive (D:\Projetos\layssafaria)

**Data:** 2026-09-26 · **Status:** Aprovado

O projeto estava em `C:\Users\Samue\OneDrive\Área de Trabalho\layssafaria`, uma pasta sincronizada com a
nuvem da Microsoft. Depois que o `infra/.env` passou a ter segredos reais (chave da OpenAI, chaves do
Supabase incluindo a `service_role`, que dá acesso total ao banco ignorando RLS), isso significava que
esses segredos estavam sendo enviados para o OneDrive automaticamente — o `.gitignore` só impede o Git
de versionar, não impede a sincronização do OneDrive.

Movido o projeto inteiro (histórico do Git incluído) para **`D:\Projetos\layssafaria`**, uma unidade que
não é sincronizada pelo OneDrive por padrão.

**Por quê essa opção e não excluir só a pasta `infra/` do sync do OneDrive:** mais simples e definitivo —
não depende de configurar exclusões seletivas no cliente do OneDrive (que é fácil de esquecer/desfazer
sem perceber), e mantém documentação + segredos juntos num único lugar coerente.

**Pendência gerada:** a cópia antiga em `C:\Users\Samue\OneDrive\Área de Trabalho\layssafaria` não pôde
ser apagada automaticamente (estava em uso pela sessão do editor) — ver
[`03-pendencias.md`](03-pendencias.md#ação-imediata-pendente-finalizar-a-mudança-de-pasta) para o passo
que falta.

**A partir de agora, todo o trabalho neste projeto acontece em `D:\Projetos\layssafaria`.**

---

## D007 — Domínio oficial confirmado: layssafaria.com

**Data:** 2026-09-26 · **Status:** Aprovado · **Substitui o gatilho da D005**

A Dra. Layssa comprou o domínio **`layssafaria.com`** na Hostgator, por 1 ano.

**Por quê:** era exatamente o gatilho previsto na [D005](#d005--domínio-usar-cloudflare-tunnel-sem-comprar-domínio-agora)
("quando a Dra. Layssa comprar o domínio oficial da clínica"). O plano continua o mesmo: não precisa
reinstalar nada, só apontar o domínio para o Cloudflare Tunnel.

**Próximos passos (ver checklist em [`03-pendencias.md`](03-pendencias.md)):**
1. Criar conta gratuita no Cloudflare e adicionar `layssafaria.com`.
2. Trocar os nameservers do domínio no painel da Hostgator para os que o Cloudflare indicar.
3. Depois que propagar, criar um túnel **nomeado** (`cloudflared tunnel create clinica-layssa`) — já
   vale a pena fazer nomeado direto (não o modo rápido `trycloudflare.com`), já que agora existe domínio
   fixo para apontar.
4. Criar registro CNAME em `layssafaria.com` apontando para o túnel.
5. Usar `https://layssafaria.com` (ou um subdomínio, ex. `app.layssafaria.com`) como `NEXT_PUBLIC_APP_URL`
   no `.env` e como URL de callback (ex.: OAuth do Google Calendar, que exige URL estável — isso só fazia
   sentido depois de ter domínio fixo, ver D005).

---

## D008 — Sudo sem senha (`NOPASSWD:ALL`) para `deskcomm` na VPS

**Data:** 2026-09-26 · **Status:** Aprovado (temporário)

Liberado `sudo` sem senha pra `deskcomm` (`/etc/sudoers.d/deskcomm-nopasswd`), pra permitir que o Claude
Code rode comandos administrativos na VPS (instalar Docker, `cloudflared`, etc.) sem interromper o usuário
pedindo senha a cada comando.

**Por quê é aceitável agora:** o `deskcomm` já estava no grupo `sudo` (só faltava digitar senha — ou seja,
já tinha o poder, só com uma trava a mais), a chave SSH que dá acesso como `deskcomm` já é sem passphrase
(D007/decisão anterior — essa já era a trava real), e a VPS ainda está em fase de montagem, sem dado real
de paciente.

**Trade-off aceito:** remove a segunda trava entre "ter o arquivo da chave" e "ter root completo" — quem
tiver o arquivo `layssafaria_vps` no PC do Samue já vira root direto, sem senha nenhuma.

**Reavaliar antes de:** colocar dados reais de pacientes em produção — nesse momento, apertar de novo
(remover o `NOPASSWD` ou restringir a comandos específicos).
