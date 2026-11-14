# Input field guide (EN)

🇧🇷 Português: [INPUT.pt-BR.md](./INPUT.pt-BR.md)

This document explains **every input field** and **every input group** for developers, agencies, and companies using the Actor.

**Practical rule:** provide at least one place in `places` and/or `placeItems`. Everything else has a safe default.

---

## Groups at a glance

| Group | Fields | Purpose |
| --- | --- | --- |
| Places | `places`, `placeItems` | Which establishments to monitor |
| Mode | `runMode` | Monitor, import history, or reset baseline |
| Budget | `maxBudgetUsd`, `maxPlacesPerRun` | Cap PPE charges this run |
| Collection | `maxReviewsPerPlace`, `sortBy`, `reviewSource`, `locale`, `mapsHost` | How / how much to collect from Maps |
| Filters | `minRating`, `maxRating`, `languages`, `onlyWithText`, `onlyWithoutOwnerResponse`, `publishedWithinDays` | Narrow what enters the diff |
| Priority & history | `negativeRatingThreshold`, `maxPriorityActionsPerPlace`, `emitBaselineAsNew`, `trackPossibleRemovals`, `consecutiveMissThreshold`, `dedupeEmittedChanges` | Prioritization and possible removals |
| Output | `includeUnchangedReviews`, `includeFlatExport`, `includeRunDigest` | What is written to the Dataset |
| Persistence | `historyStoreName` | Memory across schedules |
| Reliability | `maxConcurrency`, `delayBetweenPlacesMs`, `navigationTimeoutSecs`, `maxRetriesPerPlace`, `failRunOnAnyPlaceError` | Stability / platform cost |
| Customer keys | `customerGoogleApiKey` | Customer-owned Google key (never the Actor developer’s) |
| Notifications | `webhookUrls`, `alertEmails`, `emailMode`, `notifyOn`, `minPriorityToNotify`, `webhookSlackCompatible` | Slack/API and email |
| Proxy / debug | `proxyConfiguration`, `debug` | User Apify Proxy and verbose logs |

---

## 1. Places to monitor

### `places` (string list)

- **What:** Google Maps URLs or Place IDs (`ChIJ…`).
- **When:** simple lists without metadata.
- **Valid examples:**
  - `ChIJJ3SpfQsZTIYRIcqOUQVKF1M`
  - `https://www.google.com/maps/place/?q=place_id:ChIJ…`
  - Full Maps place URL
- **Required?** Not alone — but you need **at least one** entry in `places` **or** `placeItems`.

### `placeItems` (object list)

For agencies / multi-client setups. Each item:

| Subfield | Required | Role |
| --- | --- | --- |
| `place` | yes | URL or Place ID |
| `label` | no | Internal location label (e.g. “Downtown”) |
| `clientName` | no | Client/account name (Dataset, webhook, email) |
| `tags` | no | Reporting tags (e.g. `vip`, `flagship`) |

`places` and `placeItems` are **merged**. Duplicates resolve to the same internal place key.

---

## 2. Run mode (`runMode`)

| Value | Use when | Depth | High-frequency schedule? |
| --- | --- | --- | --- |
| `monitor` (**default**) | Daily/weekly change feed | limited (max 40 reviews) | **Yes** |
| `import_history` | One-shot deeper import | up to 100 reviews | **No** |
| `reset_baseline` | Wipe history for listed places and re-baseline | monitor caps | On demand only |
| `export_snapshot` | Alias of `import_history` | up to 100 | **No** |

**First `monitor` run:** establishes baseline (no flood of “new”). Unanswered negatives can still become priority actions.

---

## 3. Budget (PPE)

Only `place-monitored` is charged (each **successfully verified** place). Changes, actions, webhook and email are **included**.

### `maxBudgetUsd`

- Soft USD cap for this run (planned at **$0.10** per successful place).
- Example: `5` ≈ up to ~50 charged places.
- Also set Apify run **max total charge**.

### `maxPlacesPerRun`

- Hard cap of successful charges this run.
- Suggested prefill: `50`.
- If **both** budget fields are empty, a safety cap of **50** applies.

Collection failures are **not** PPE-billed.

---

## 4. Collection

### `maxReviewsPerPlace`

- How many most-recent public reviews to analyse per place.
- Default: **20**.
- Cap: **40** for `monitor` / `reset_baseline`; **100** for `import_history`.

### `sortBy`

Review order requested before collection:

| Value | Meaning |
| --- | --- |
| `newest` (default) | Newest — best for monitoring |
| `relevant` | Most relevant |
| `highest` | Highest rating |
| `lowest` | Lowest rating |

### `reviewSource`

| Value | Meaning |
| --- | --- |
| `auto` (default) | Read the public **Google Search reviews dialog** for the place (sortable, paginated, same review ids as Maps). Falls back to the Maps panel if Search does not open it. |
| `maps_panel` | Only the Google Maps place panel (legacy). |

**Why:** anonymous Google Maps sessions get a *limited view* — about 5 "most relevant" reviews, and both *Sort* and *See more reviews* open a sign-in wall. A `newest` monitor on the Maps panel alone therefore cannot see the latest reviews.

When a sample is incomplete (`limited_public_view` or `sort_not_applied`), the run still succeeds, but:

- possible-removal counters are **not** advanced (no false `possible_removal`);
- previously unseen reviews published before the baseline are stored silently instead of being emitted as `new`;
- `place_summary.summary.collection` reports `source`, `issues`, `sampleIncomplete` and `removalTracking`, and alert emails explain it.

### `locale`

Browser locale **and Google interface language** (sent as `hl=`; without it Google picks the language from the IP). Examples: `en-US`, `pt-BR`. Also selects the email template language (`pt*` → Portuguese).

### `mapsHost`

Host when building URLs from Place IDs. Example: `www.google.com.br` for Brazil, `www.google.com` globally.

---

## 5. Filters (after collection)

Applied **after** scrape. They do not change what Maps shows; only what enters diff / actions.

| Field | Role | Default |
| --- | --- | --- |
| `minRating` | Keep rating ≥ value | off |
| `maxRating` | Keep rating ≤ value | off |
| `languages` | BCP-47 prefixes (`en`, `pt`…). Unknown language is **kept** | `[]` |
| `onlyWithText` | Drop star-only reviews | `false` |
| `onlyWithoutOwnerResponse` | Keep unanswered only | `false` |
| `publishedWithinDays` | Approximate publish window; unknown dates are **kept** | off |

---

## 6. Priority actions & history

### `negativeRatingThreshold` (default `2`)

Ratings ≤ this value count as “negative” for the priority queue.

### `maxPriorityActionsPerPlace` (default `50`)

Cap of `priority_action` records per place this run.

### `emitBaselineAsNew` (default `false`)

- `false`: first snapshot only builds history (recommended).
- `true`: treat baseline as “new” (can flood).

### `trackPossibleRemovals` (default `true`)

If a known review disappears across several **successful** scrapes, emit `possible_removal` (history is never hard-deleted).

### `consecutiveMissThreshold` (default `3`)

Successful scrapes in a row without seeing the review before `possible_removal`.

### `dedupeEmittedChanges` (default `true`)

Skip re-emitting the same change fingerprint to the Dataset.

---

## 7. Dataset output

| Field | Default | Effect |
| --- | --- | --- |
| `includeUnchangedReviews` | `false` | Also push unchanged `review_snapshot` rows |
| `includeFlatExport` | `true` | `flat_export` rows for CSV/Sheets |
| `includeRunDigest` | `true` | One `run_digest` with run totals |

Common record types: `priority_action`, `change`, `place_summary`, `flat_export`, `run_digest`, `place_error`.

---

## 8. Persistence & scheduling

### `historyStoreName` (default `google-maps-review-history`)

Named Apify Key-Value Store for history between runs.

**Important:** keep the **same name** on every scheduled run for the same place set. Changing it resets memory (new baseline).

---

## 9. Reliability (cost / stability)

| Field | Default | Notes |
| --- | --- | --- |
| `maxConcurrency` | `1` | Parallel places (max 3). Prefer 1 without residential proxy |
| `delayBetweenPlacesMs` | `1500` | Pause between places when concurrency is 1 |
| `navigationTimeoutSecs` | `35` | Page load timeout (max 60) |
| `maxRetriesPerPlace` | `0` | Extra attempts after failure (max 1). No PPE, but uses infra |
| `failRunOnAnyPlaceError` | `false` | Isolate per-place failures vs fail the whole run |

---

## 10. Customer API keys

### `customerGoogleApiKey` (optional, secret)

**Customer** Google Places API key. If scrape metadata is missing name/address, the Actor may enrich via Places Details using **this** key only. Never uses the Actor developer’s keys. Proxy is also customer-configured (`proxyConfiguration`).

---

## 11. Notifications

Webhooks and email share `notifyOn` / `minPriorityToNotify`. Included in `place-monitored` pricing.

### `webhookUrls`

HTTP endpoints for JSON payloads (Slack Incoming Webhook, Make, Zapier, n8n, your API).

### `alertEmails`

HTML/text recipients. Direct delivery through Gmail (default, no domain), custom SMTP, or Resend. Configure credentials as Actor environment secrets. See [email setup](./EMAILS.en.md).

### `emailMode`

| Value | Behaviour |
| --- | --- |
| `digest` (default) | One email at end of run, section per place |
| `per_place` | One email per place with alerts |

### `notifyOn`

| Value | Sends |
| --- | --- |
| `none` | Nothing |
| `priority_actions` (default) | Priority actions only |
| `changes` | Changes only |
| `priority_and_changes` | Both |
| `all` | Always (even if empty) |

### `minPriorityToNotify` (default `high`)

Ignore actions below: `critical` → `high` → `medium` → `low`.

### `webhookSlackCompatible` (default `false`)

When `true`, include Slack-friendly `text` / `blocks`.

Details: [WEBHOOKS.en.md](./WEBHOOKS.en.md) · [EMAILS.en.md](./EMAILS.en.md)

---

## 12. Proxy & debug

### `proxyConfiguration`

Apify Proxy editor (groups, country, etc.). Recommended for large lists or frequent schedules. Proxy cost is billed to the **user**.

### `debug` (default `false`)

Verbose scrape/diff logs. Use only for troubleshooting.

---

## Full example (daily monitor)

```json
{
  "placeItems": [
    {
      "place": "ChIJxxxxxxxxxxxxxxxxxxxxxx",
      "label": "Downtown",
      "clientName": "Demo Brand",
      "tags": ["flagship"]
    }
  ],
  "runMode": "monitor",
  "maxReviewsPerPlace": 20,
  "maxPlacesPerRun": 50,
  "maxBudgetUsd": 5,
  "sortBy": "newest",
  "locale": "en-US",
  "mapsHost": "www.google.com",
  "negativeRatingThreshold": 2,
  "historyStoreName": "demo-maps-history",
  "includeFlatExport": true,
  "alertEmails": ["ops@brand.com"],
  "emailMode": "digest",
  "webhookUrls": ["https://hooks.slack.com/services/XXX/YYY/ZZZ"],
  "webhookSlackCompatible": true,
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "maxConcurrency": 1,
  "maxRetriesPerPlace": 0
}
```

---

## Related

- [Product guide](./PRODUCT.en.md)
- [Pricing](./PRICING.en.md)
- [Webhooks](./WEBHOOKS.en.md)
- [Email](./EMAILS.en.md)
