# Alertas por e-mail (PT-BR)

🇬🇧 English: [EMAILS.en.md](./EMAILS.en.md)

Envie alertas de reputação por e-mail além de (ou no lugar de) webhooks.

A entrega está **incluída** no evento PPE `place-monitored`. Falhas e reenvios **nunca** são cobrados à parte. Custos e limites do provedor de e-mail se aplicam separadamente do uso da Apify.

---

## Setup rápido

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxx",
      "label": "Paulista",
      "clientName": "Padaria Aurora"
    }
  ],
  "alertEmails": ["ops@marca.com", "dono@marca.com"],
  "emailMode": "digest",
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "locale": "pt-BR"
}
```

Você pode combinar `alertEmails` e `webhookUrls` na mesma run. Os dois usam os mesmos filtros `notifyOn` / `minPriorityToNotify`.

---

## Modos

| `emailMode` | Comportamento |
| --- | --- |
| `digest` (padrão) | Um e-mail no **fim da run**, com uma seção por estabelecimento com alertas |
| `per_place` | Um e-mail **por local** assim que aquele local termina |

Digest é o recomendado para agendas multi-local.

---

## Conteúdo do template

Cada seção de estabelecimento inclui:

- Label / nome do local, cliente, link do Google Maps  
- Ações prioritárias (prioridade, tipo, motivo, nota, autor, texto da review, link)  
- Mudanças (quando `notifyOn` inclui mudanças)  

O idioma segue `locale` (`pt*` → template em português; senão inglês).

---

## Como o envio funciona

O envio é direto, sem outro Actor. Escolha o transporte por variáveis do ambiente; não coloque senhas no input, código ou logs.

### Gmail gratuito, sem domínio (padrão)

- `EMAIL_PROVIDER=gmail` (ou omita).
- `GMAIL_USER`: endereço Gmail usado como remetente.
- `GMAIL_APP_PASSWORD`: senha de app Google, marcada como **secreta**.

Ative a verificação em duas etapas e gere uma senha de app para este Actor, se a conta permitir. Nunca use sua senha normal. Cadastre a senha diretamente nos segredos da Apify, sem compartilhá-la em chat. O Gmail usa `smtp.gmail.com:465` com TLS e o próprio usuário autenticado como remetente. Para alertas pessoais, pode usar o mesmo endereço como destinatário.

Não há mensalidade de envio do Gmail pessoal, mas limites e bloqueios antispam do Google se aplicam. Indicado para poucos alertas, não para envio comercial em massa. A execução da Apify continua com seus custos. Caso não haja opção de senha de app, não reduza a segurança da conta: use um provedor SMTP autorizado ou uma integração OAuth separada.

### Domínio próprio via SMTP no futuro

Registrar um domínio não fornece SMTP; contrate/configure um provedor de e-mail para ele e autentique o domínio conforme as instruções desse provedor (SPF/DKIM/DMARC).

| Variável | Configuração |
| --- | --- |
| `EMAIL_PROVIDER` | `smtp` |
| `SMTP_HOST` | Servidor do seu provedor |
| `SMTP_PORT` | `465` (TLS) ou `587` (STARTTLS) |
| `SMTP_SECURE` | `true` para TLS imediato; `false` para STARTTLS obrigatório |
| `SMTP_USER` | Usuário de envio |
| `SMTP_PASSWORD` | Senha/token SMTP, **secreto** |
| `EMAIL_FROM` | Remetente autorizado pelo provedor |

A validação de certificado permanece ativa. O código exige TLS também no modo STARTTLS.

### Resend opcional

Use `EMAIL_PROVIDER=resend`, `RESEND_API_KEY` como segredo e `EMAIL_FROM` autorizado. O envio usa `POST https://api.resend.com/emails`. Em produção, verifique seu domínio; o remetente de testes `resend.dev` permite somente o e-mail autorizado da conta Resend. Limites e custos do plano se aplicam.

### Verificação

Salve as variáveis em Source → Environment variables, faça o build e execute a tarefa com `alertEmails` e `notifyOn: "all"` para testar mesmo sem mudanças. Confira estatísticas e caixa de entrada/spam. O aceite do servidor não confirma entrega na caixa de entrada.

As conexões têm timeouts de 30 segundos. Falhas contam nas estatísticas, mas não interrompem a coleta. Rejeição parcial de destinatários conta como falha. Não há fallback para outro Actor nem retry automático, evitando duplicação de mensagens já aceitas. Filtros, resumo digest/per_place e agendamento continuam iguais.

---

## Exemplo (agenda BR + Slack + e-mail)

```json
{
  "runMode": "monitor",
  "maxReviewsPerPlace": 30,
  "locale": "pt-BR",
  "mapsHost": "www.google.com.br",
  "alertEmails": ["reputacao@padariaaurora.com.br"],
  "emailMode": "digest",
  "webhookUrls": ["https://hooks.slack.com/services/XXX/YYY/ZZZ"],
  "webhookSlackCompatible": true,
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high"
}
```

---

## Relacionados

- [Webhooks](./WEBHOOKS.pt-BR.md)
- [Precificação](./PRICING.pt-BR.md)
- [Guia do produto](./PRODUCT.pt-BR.md)

## Resumos sem novidades

Com `notifyOn: "all"`, o e-mail inclui contagem coletada, horário UTC, nota geral, total no Google e até 20 avaliações após os filtros, com autor, nota, texto e resposta do proprietário. Essa amostra não significa que as avaliações sejam novas nem que todas as avaliações do Google tenham sido extraídas. A primeira coleta é identificada como base inicial; as seguintes informam quando não há mudanças. `locale: "pt-BR"` seleciona o template em português. Filtros de alertas continuam controlando quando enviar.
