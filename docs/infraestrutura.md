# Plano de Infraestrutura

## Checklist antes do primeiro deploy

- [ ] Domínio registrado e DNS acessível para criar registro A.
- [ ] VPS Hostgator com acesso SSH (root ou sudo), Docker instalável.
- [ ] Conta Supabase criada (tier free serve para começar).
- [ ] Chave de API da OpenAI, com billing ativo.
- [ ] Número de WhatsApp dedicado, com o app instalado em um celular para escanear o QR Code do WAHA.
- [ ] Ler o `hostgator-setup-kit/install.sh` do DeskcommCRM linha a linha antes de rodar em produção.

## Passo a passo (alto nível)

1. Apontar o domínio (registro A) para o IP da VPS Hostgator.
2. Acessar a VPS via SSH.
3. Rodar o instalador guiado:
   ```bash
   curl -fsSL https://raw.githubusercontent.com/melgarafael/DeskcommCRM/main/hostgator-setup-kit/comecar.sh | bash
   ```
   (revisar o script antes — ver nota de segurança abaixo)
4. Fornecer ao instalador: domínio, `OPENAI_API_KEY`, credenciais do Supabase, senha do admin.
5. Aguardar emissão do certificado HTTPS (~1 min) e acessar `https://<dominio>`.
6. Conectar o WhatsApp escaneando o QR Code gerado pelo WAHA dentro da interface do DeskcommCRM.
7. Configurar o cron do `backup.sh` (Supabase free tier não faz backup automático).
8. Configurar integração com Google Calendar (OAuth) para agendamento.

## Nota de segurança sobre "curl | bash"

Rodar `curl | bash` direto de um script de terceiros em uma VPS de produção é uma prática que merece
cautela, mesmo quando o projeto é confiável (e este parece ser, ver
[`../memoria/02-pesquisa-deskcommcrm.md`](../memoria/02-pesquisa-deskcommcrm.md)). Antes do deploy real:

1. Baixar o script localmente primeiro (`curl -o comecar.sh ...`), ler o conteúdo, só depois rodar.
2. O mesmo vale para `install.sh`, que é quem de fato mexe no sistema (Docker, Postgres, HTTPS, cron).
3. Preferir rodar como usuário com sudo, não como root direto, quando possível.
4. Fazer isso com um snapshot/backup da VPS tirado antes (se a Hostgator oferecer), para poder reverter.

## Variáveis de ambiente que vamos precisar preencher

Ver a lista completa pesquisada em
[`../memoria/02-pesquisa-deskcommcrm.md`](../memoria/02-pesquisa-deskcommcrm.md). As essenciais para a
v1 (WAHA + OpenAI + Supabase):

```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
SUPABASE_DB_URL=

OPENAI_API_KEY=

WAHA_API_BASE_URL=
WAHA_API_KEY=
WAHA_HMAC_SECRET=

GOOGLE_CALENDAR_CLIENT_ID=

APP_NAME=
APP_LOCALE=pt-BR

LGPD_DPO_EMAIL=
```

**Nunca commitar um `.env` preenchido com valores reais no Git.** O `.gitignore` da raiz já bloqueia
isso — ver arquivo `infra/.env.example` (a ser criado quando tivermos os valores reais para validar
contra o `.env.example` oficial do projeto).

## Orçamento recorrente estimado (a validar)

| Item | Custo estimado | Observação |
|---|---|---|
| VPS Hostgator | já contratada pelo usuário | confirmar plano/specs (RAM mínima recomendada: 4GB) |
| Domínio | ~R$40-60/ano | se ainda não tiver |
| Supabase | R$0 (tier free) para começar | pode precisar upgrade se crescer muito |
| OpenAI API | variável, por uso (tokens) | monitorar via `AI_BUDGET_ENFORCEMENT` |
| WAHA Plus | **a confirmar** — ver pendência em `memoria/03-pendencias.md` | pode ter custo de licença |
| Upstash Redis | R$0 (tier free) para começar | opcional |
