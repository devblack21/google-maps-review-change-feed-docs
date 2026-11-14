# How to use this Actor (EN)

🇧🇷 Português: [USAGE.pt-BR.md](./USAGE.pt-BR.md)

This guide is for **users** of Google Maps Review Change Feed on Apify.

---

## What the Actor does (simple)

1. You give it Google Maps places (URLs or Place IDs).  
2. You click **Run** (or let a **Schedule** run it).  
3. It reads public reviews, compares them with the last run, and saves history.  
4. If something important changed, it can alert you by **webhook** (Slack etc.) and/or **email**.

It does **not** watch Google 24/7 by itself. It only checks when a run happens.

---

## Step 1 — First run (baseline)

1. Open the Actor in Apify Console.  
2. Fill input (at least one place). Recommended start:

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxxxxxxxxxxxxxxxx",
      "label": "Main store",
      "clientName": "My Brand"
    }
  ],
  "runMode": "monitor",
  "maxReviewsPerPlace": 20,
  "maxPlacesPerRun": 10,
  "maxBudgetUsd": 1,
  "historyStoreName": "my-maps-review-history",
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "includeFlatExport": true
}
```

3. Click **Start**.  
4. Wait for success. Check the **Dataset** and **Key-value store** `RUN_SUMMARY`.

**First run = memory.** It stores the current reviews. It usually does **not** flood you with “everything is new”. Unanswered negative reviews can still become priority actions.

---

## Step 2 — Schedule automatic checks (Apify)

So it keeps monitoring without you clicking every day:

1. In Apify Console, open this Actor (or save a **Task** with your input).  
2. Go to **Schedules** (or create a Schedule that runs the Task/Actor).  
3. Choose frequency, for example:
   - **Daily** at 08:00 (local morning) — recommended for most brands  
   - Every 6 hours — only if you need faster alerts (costs more)  
4. Save the schedule.

**Critical:** use the **same** `historyStoreName` on every scheduled run.  
If you change that name, the Actor “forgets” history and starts a new baseline.

---

## Step 3 — Alerts

### Dataset (always)
Every run writes structured records you can export to CSV, Sheets, or CRM.

### Webhook (optional)
Set `webhookUrls` (Slack Incoming Webhook, Make, Zapier, n8n, your API).  
Guide: [WEBHOOKS.en.md](./WEBHOOKS.en.md)

### Email (optional)
Set `alertEmails` and `emailMode` (`digest` recommended).  
Guide: [EMAILS.en.md](./EMAILS.en.md)

### What triggers a notification?
Controlled by:

| Field | Meaning |
| --- | --- |
| `notifyOn` | `none` · `priority_actions` (default) · `changes` · `priority_and_changes` · `all` |
| `minPriorityToNotify` | Ignore actions below this level (`high` is a good default) |

No matching event → run still succeeds, **no alert spam**.

---

## Step 4 — How billing works (user view)

| What | Cost |
| --- | --- |
| Each place **successfully checked** in a run | **USD 0.10** (`place-monitored`) |
| Detected changes, priority actions, webhook, email | **Included** (no extra event) |
| Failed place / retry | **No** custom event charge |
| Apify platform usage (compute, proxy, storage) | Billed by Apify to **your** account |

Example: 10 places, once per day → about **USD 1.00/day** in Actor events (+ platform usage).

Details: [PRICING.en.md](./PRICING.en.md)

---

## Recommended production setup

1. First manual run → baseline  
2. Schedule **daily** morning  
3. Same `historyStoreName` forever for that portfolio  
4. `notifyOn`: `priority_actions`  
5. Slack webhook + email digest  
6. Set `maxBudgetUsd` / `maxPlacesPerRun` so a bad config cannot overspend  
7. Use your own Apify Proxy for large lists  

More product context: [PRODUCT.en.md](./PRODUCT.en.md)  
Every input field: [INPUT.en.md](./INPUT.en.md)
