# Pendências e Próximos Passos

Lista viva do que falta para avançar. Marcar `[x]` quando resolvido e mover para o diário
([`00-diario-do-projeto.md`](00-diario-do-projeto.md)) com a data.

## Bloqueadores para o deploy (precisam de resposta/ação do usuário)

- [x] **Domínio:** resolvido em 2026-09-25 — não vamos comprar domínio agora, vamos usar Cloudflare
      Tunnel (ver decisão D005 em `01-decisoes.md`). Comprar domínio fica para quando a Dra. Layssa
      decidir.
- [x] **VPS confirmada em 2026-09-26:** é uma VPS de verdade, Ubuntu 22.04.
- [ ] **Acesso SSH ainda não estabelecido pelo usuário.** Guia completo (incluindo como guardar
      IP/porta/usuário com segurança via chave SSH + `~/.ssh/config`, sem depender de arquivo nenhum
      dentro do projeto) em
      [`../docs/infraestrutura.md`](../docs/infraestrutura.md#onde-guardar-ip-porta-e-usuário-da-vps-com-segurança).
      **Próxima ação do usuário:** gerar a chave SSH e copiá-la pra VPS seguindo o guia.
- [x] **Chave da API OpenAI:** preenchida pelo usuário em `infra/.env` em 2026-09-26.
- [x] **Supabase:** URL, anon key e service role key preenchidas pelo usuário em `infra/.env` em
      2026-09-26. Falta só o `SUPABASE_DB_URL` (connection string do Postgres).
- [ ] **Número de WhatsApp dedicado:** a clínica vai usar um número novo só para a IA, ou o número que já
      usa hoje? (Recomendado: número novo, para não misturar histórico/contatos pessoais com o agente.)
      Ainda não perguntado/respondido.
- [ ] **Cloudflare Tunnel:** instalar o `cloudflared` na VPS assim que o SSH estiver disponível.
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
- [ ] Ler o script `hostgator-setup-kit/install.sh` linha a linha antes do primeiro deploy (checklist de
      segurança).

## Ação imediata pendente: finalizar a mudança de pasta

- [ ] **Reabrir o projeto no editor/VSCode a partir de `D:\Projetos\layssafaria`** (o projeto foi movido
      pra fora do OneDrive em 2026-09-26 — ver decisão D006 em `01-decisoes.md`). A pasta antiga em
      `C:\Users\Samue\OneDrive\Área de Trabalho\layssafaria` ainda existe fisicamente (não foi possível
      apagar porque estava em uso pela sessão atual) e **contém os mesmos segredos reais** — apagar assim
      que possível depois de reabrir no novo local, pra não ficar com o `.env` duplicado dentro do
      OneDrive.

## Depois que o bloqueado acima estiver resolvido

- [ ] Rodar o instalador guiado do DeskcommCRM na VPS.
- [ ] Configurar o cron do `backup.sh` (Supabase free tier não tem backup automático).
- [ ] Escrever e revisar os prompts dos 3 agentes definidos em
      [`docs/agentes-ia.md`](../docs/agentes-ia.md) (Recepção, Vendas/Qualificação, Follow-up).
- [ ] Conectar WAHA com o número de WhatsApp escolhido (QR Code).
- [ ] Configurar Google Calendar para agendamento automático.
- [ ] Testar o fluxo ponta a ponta com números de teste antes de liberar para clientes reais.
- [ ] Criar repositório no GitHub e subir o projeto (usuário pediu para isso ser feito "quando tiver
      algo" — ainda não fizemos `git init`/push, está tudo local por enquanto).
