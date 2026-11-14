# Google Maps Review Change Feed — Product Guide (EN)

**Your Google Maps reputation radar.**  
Monitor many locations. See what changed. Know what to answer first.

🇧🇷 Versão em português: [PRODUCT.pt-BR.md](./PRODUCT.pt-BR.md)

---

## The problem (plain English)

If you manage **restaurants, clinics, stores, salons, hotels, franchises or local SEO clients**, Google Maps reviews never stop.

Opening Maps one by one every day does not scale.

What you actually need is:

> “What changed since yesterday — and what must I reply to right now?”

That is exactly what this Actor delivers.

---

## What the product does

1. Collects **public Google Maps reviews** for your places (any country)
2. Stores a durable **history** between scheduled runs
3. Detects **new reviews** and **changes** (rating, text, owner reply, photos)
4. Builds **priority actions** (especially unanswered negatives)
5. Writes structured results to the Apify **Dataset**
6. Optionally notifies **Slack / Make / Zapier / your API** via webhooks

It is **not** just a one-shot review scraper.  
It is a **change feed + action queue** for reputation teams.

---

## Who it is for

### International
- Multi-location brands (US, EU, LATAM, APAC)
- Franchise networks
- Reputation / local SEO agencies
- Customer experience teams

### Brazil
- Agências de marketing local / SEO local
- Redes de restaurantes, clínicas, academias, varejo
- Franquias com dezenas de unidades
- Operações que precisam responder rápido no Google

---

## Where you see the alerts

| Channel | What you get |
| --- | --- |
| **Apify Dataset** | Priority actions, changes, place summaries, CSV-ready rows |
| **RUN_SUMMARY** | Run totals and health |
| **Webhooks** | Push to Slack / automation tools when something needs attention |
| **Schedule** | Daily/weekly automatic monitoring |

---

## Real-world examples

### Example A — Brazilian bakery network (Campinas + São Paulo)

**Input**

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxxxxxxxxxxxxxxxx",
      "label": "Campinas Centro",
      "clientName": "Padaria Aurora",
      "tags": ["sp", "interior"]
    },
    {
      "place": "https://www.google.com.br/maps/place/?q=place_id:ChIJyyyyyyyyyyyyyyyyyyyy",
      "label": "Paulista",
      "clientName": "Padaria Aurora",
      "tags": ["sp", "capital"]
    }
  ],
  "runMode": "monitor",
  "maxReviewsPerPlace": 20,
  "maxPlacesPerRun": 50,
  "maxBudgetUsd": 5,
  "negativeRatingThreshold": 2,
  "historyStoreName": "aurora-maps-history",
  "webhookUrls": ["https://hooks.slack.com/services/XXX/YYY/ZZZ"],
  "alertEmails": ["ops@padariaaurora.com"],
  "emailMode": "digest",
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "webhookSlackCompatible": true
}
```

**Day 1 (baseline)**  
History is created. Unanswered negatives already become priority actions.

**Day 2 (monitor)**  
Dataset may show:

```json
{
  "recordType": "priority_action",
  "priority": "critical",
  "actionType": "new_negative",
  "reason": "New 1-star review without owner response",
  "clientName": "Padaria Aurora",
  "placeName": "Padaria Aurora — Paulista",
  "review": {
    "rating": 1,
    "authorName": "Carla",
    "text": "Fila enorme e pão frio",
    "hasOwnerResponse": false,
    "reviewUrl": "https://..."
  }
}
```

Slack receives a short alert with the place link and reason.

### Example B — US multi-location dental group

```json
{
  "placeItems": [
    {
      "place": "ChIJzzzzzzzzzzzzzzzzzzzz",
      "label": "Miami Lakes",
      "clientName": "BrightSmile Dental",
      "tags": ["usa", "fl"]
    }
  ],
  "locale": "en-US",
  "mapsHost": "www.google.com",
  "sortBy": "newest",
  "notifyOn": "priority_and_changes"
}
```

Useful alerts:
- new 1–2 star reviews
- rating drops
- owner reply appeared (case closed)
- possible removal after repeated misses (investigate, never assume hard delete)

### Example C — Agency managing 40 clients

Use `clientName` + `label` + `tags` so every Dataset row is CRM-ready:

- filter by client in Sheets
- route Slack alerts by workspace/channel
- export only `priority_action` for morning standups

---

## Why the value is high

| Without this Actor | With this Actor |
| --- | --- |
| Manual Maps checking | Automatic schedule |
| Missed bad reviews | Priority queue |
| No memory between days | Durable history |
| Raw review dumps | Actionable change feed |
| Guessing what matters | Evidence + links + reasons |

A single unanswered 1-star review can cost appointments.  
Catching it the same day is the product.

---

## Safety principles (trust)

- Missing from one scrape **≠ deleted**
- `possible_removal` only after repeated misses
- Baseline run does not flood “everything is new”
- One place failure does not kill the whole run

---

## Suggested operating rhythm

1. First run → baseline  
2. Schedule daily at morning local time  
3. Keep the same `historyStoreName`  
4. Send `priority_actions` to Slack  
5. Export `flat_export` weekly for reporting

---

## Pricing reality (Apify Store)

**Pay per event + usage billed to the user** (Console toggle required). Only paid custom event: `place-monitored` at **USD 0.10** per successful verification. Changes, actions, webhooks and email included. Failures/retries free. Safety cap 50 places/run if budget unset.

Frequency (30 places, Actor events only): **1×/day → $90 / 30 days**; **4×/day → $360**, plus Apify platform usage on your account. Details: [PRICING.en.md](./PRICING.en.md).

Details: [PRICING.en.md](./PRICING.en.md)

---

## Related docs

- [How to use + Schedule (EN)](./USAGE.en.md)
- [Input field guide (EN)](./INPUT.en.md)
- [PPE Pricing (EN)](./PRICING.en.md)
- [Webhooks guide (EN)](./WEBHOOKS.en.md)
- [Email alerts (EN)](./EMAILS.en.md)
- Portuguese: [PRODUCT.pt-BR.md](./PRODUCT.pt-BR.md) · [USAGE.pt-BR.md](./USAGE.pt-BR.md)
