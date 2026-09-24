# Pendências e Próximos Passos

Lista viva do que falta para avançar. Marcar `[x]` quando resolvido e mover para o diário
([`00-diario-do-projeto.md`](00-diario-do-projeto.md)) com a data.

## Bloqueadores para o deploy (precisam de resposta/ação do usuário)

- [ ] **Domínio:** já existe um domínio para a clínica (ex.: `clinicalayssafaria.com.br`)? Se sim, qual
      registrador? Se não, vamos precisar registrar um.
- [ ] **Acesso à VPS Hostgator:** IP, usuário SSH e senha/chave já em mãos? Painel é cPanel, VPS pura
      (root) ou algo tipo CyberPanel?
- [ ] **Chave da API OpenAI:** já existe uma conta/organização OpenAI com billing ativo? Se não, criar em
      https://platform.openai.com e gerar uma API key (colocar em `infra/.env` local, **nunca commitar**
      no Git — ver `.gitignore`).
- [ ] **Conta Supabase:** criar uma conta gratuita em https://supabase.com (usada para banco de dados,
      auth e storage — o DeskcommCRM automatiza a criação do projeto Supabase durante a instalação se
      dermos um `SUPABASE_ACCESS_TOKEN`, mas a conta em si precisa existir).
- [ ] **Número de WhatsApp dedicado:** a clínica vai usar um número novo só para a IA, ou o número que já
      usa hoje? (Recomendado: número novo, para não misturar histórico/contatos pessoais com o agente.)

## Informações da clínica (para a base de conhecimento dos agentes)

- [ ] Preencher [`clinica/base-conhecimento.md`](../clinica/base-conhecimento.md) com dados reais:
      serviços/procedimentos oferecidos, faixa de preço ou política de "sob consulta", horário de
      funcionamento, endereço, convênios/planos aceitos, formas de pagamento, diferenciais da Dra.
      Layssa (formação, especializações, tempo de experiência), fotos/depoimentos que possam virar prova
      social no atendimento.

## Decisões técnicas ainda abertas

- [ ] Confirmar se "WAHA Plus" (mencionado no README do DeskcommCRM) tem custo adicional além da VPS —
      impacta o orçamento mensal.
- [ ] Definir se vamos integrar com o Ileva (sistema de gestão) nesta fase ou deixar para uma fase 2 —
      hoje não está claro se o Ileva tem API pública para esse tipo de integração.
- [ ] Ler o script `hostgator-setup-kit/install.sh` linha a linha antes do primeiro deploy (checklist de
      segurança).

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
