# Email alerts (EN)

🇧🇷 Português: [EMAILS.pt-BR.md](./EMAILS.pt-BR.md)

Send reputation alerts by email in addition to (or instead of) webhooks.

Delivery is **included** in the `place-monitored` PPE event. Failures and resends are **never** billed separately. Email provider costs and quotas apply separately from Apify usage.

---

## Quick setup

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxx",
      "label": "Paulista",
      "clientName": "Padaria Aurora"
    }
  ],
  "alertEmails": ["ops@brand.com", "owner@brand.com"],
  "emailMode": "digest",
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "locale": "en-US"
}
```

You can combine `alertEmails` and `webhookUrls` in the same run. Both use the same `notifyOn` / `minPriorityToNotify` filters.

---

## Modes

| `emailMode` | Behaviour |
| --- | --- |
| `digest` (default) | One email at the **end of the run**, with a section per establishment that had alerts |
| `per_place` | One email **per place** as soon as that place finishes |

Digest is recommended for multi-location schedules.

---

## Template content

Each establishment section includes:

- Place label / name, client, Google Maps link  
- Priority actions (priority, type, reason, rating, author, review text, review link)  
- Changes (when `notifyOn` includes changes)  

Language follows `locale` (`pt*` → Portuguese template, otherwise English).

---

## How sending works

Delivery is direct, without another Actor. Choose the transport using environment variables; never place credentials in input, code or logs.

### Free Gmail without a domain (default)

- `EMAIL_PROVIDER=gmail` (or omit).
- `GMAIL_USER`: sender Gmail address.
- `GMAIL_APP_PASSWORD`: Google app password, marked **secret**.

Enable 2-Step Verification and generate an app password for this Actor if your account supports it. Never use your normal Google password. Enter the app password directly in Apify secrets, not chat. Gmail uses `smtp.gmail.com:465` with TLS and the authenticated Gmail address as sender. Personal alerts can be sent to that same address.

Personal Gmail has no sending subscription fee, but Google account limits and anti-abuse controls apply. Use it for low-volume alerts, not bulk commercial email. Apify execution costs remain separate. If app passwords are unavailable, do not weaken account security: use an authorized SMTP provider or a separate OAuth integration.

### Custom-domain SMTP later

A registered domain alone does not provide SMTP. Configure a mail provider for the domain and authenticate it following that provider's SPF/DKIM/DMARC instructions.

| Variable | Setting |
| --- | --- |
| `EMAIL_PROVIDER` | `smtp` |
| `SMTP_HOST` | Provider server |
| `SMTP_PORT` | `465` (TLS) or `587` (STARTTLS) |
| `SMTP_SECURE` | `true` for implicit TLS; `false` for mandatory STARTTLS |
| `SMTP_USER` | Sending account |
| `SMTP_PASSWORD` | SMTP password/token, **secret** |
| `EMAIL_FROM` | Sender authorized by the provider |

Certificate validation stays enabled. STARTTLS is required when implicit TLS is disabled.

### Optional Resend

Use `EMAIL_PROVIDER=resend`, secret `RESEND_API_KEY` and authorized `EMAIL_FROM`. Requests go to `POST https://api.resend.com/emails`. Verify your domain for production; the `resend.dev` test sender only allows your authorized Resend account email. Provider quotas and costs apply.

### Verification

Save variables in Source → Environment variables, build the Actor, and run the task with `alertEmails` and `notifyOn: "all"` to test even without changes. Check email statistics and inbox/spam. Server acceptance does not guarantee inbox delivery.

Connections have 30-second timeouts. Failures count in email stats without interrupting collection. Partial recipient rejection counts as failure. There is no fallback to another Actor or automatic retry that could duplicate accepted messages. Digest/per_place modes, filters and schedules are unchanged.

---

## Example (Brazil schedule + Slack + email)

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

## Related

- [Webhooks](./WEBHOOKS.en.md)
- [Pricing](./PRICING.en.md)
- [Product guide](./PRODUCT.en.md)

## Summaries without changes

With `notifyOn: "all"`, emails include collection counts, UTC check time, overall rating, total Google reviews and up to 20 matching collected reviews with author, rating, text and owner response. This snapshot is not a list of new reviews or an exhaustive extraction of Google reviews. Baselines and unchanged checks have distinct messages. Set `locale: "pt-BR"` for Portuguese. Notification filters still control when to send.
