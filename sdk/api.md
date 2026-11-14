# Apify API — get started

Call the Actor over HTTPS. No SDK and no extra package. Every language guide in this folder is this same flow with the standard library of that language.

Actor: `basta_compilar/google-maps-review-change-feed`  
API id (use this in the path): `basta_compilar~google-maps-review-change-feed`  
Base URL: `https://api.apify.com/v2`

Create a token in Apify Console → Settings → Integrations. Keep it in `APIFY_TOKEN`. Do not put it in source code.

## Use cases

| Case | What you send |
| --- | --- |
| Daily check of many locations | `runMode: "monitor"` and the **same** `historyStoreName` every day |
| Agency portfolio | `placeItems` with `label`, `clientName` and `tags` |
| First memory of a portfolio | One successful `monitor` run. It stores the baseline and does not mark every old review as new |
| Deeper one-shot import | `runMode: "import_history"` once, then go back to `monitor` |
| Your own app, Slack, or inbox | Read `priority_action` rows from the Dataset, or set `webhookUrls` / `alertEmails` |
| Spend cap | `maxBudgetUsd` and `maxPlacesPerRun`. A successful place check is **USD 0.10** |

## Flow

1. `POST /acts/basta_compilar~google-maps-review-change-feed/runs` with the input JSON.
2. Read `data.id` and `data.defaultDatasetId`.
3. `GET /actor-runs/{id}` until `data.status` is terminal.
4. `GET /datasets/{defaultDatasetId}/items` and keep the rows you need.

Terminal statuses: `SUCCEEDED`, `FAILED`, `ABORTED`, `TIMED-OUT`.

`SUCCEEDED` means the run finished. A place can still have failed inside it. Read `place_error` and `run_digest`. You are charged only for a place that was verified.

`waitForFinish` on the start call waits at most **60 seconds**. A Maps check is often longer, so the samples poll.

## Start a run

```http
POST /v2/acts/basta_compilar~google-maps-review-change-feed/runs HTTP/1.1
Host: api.apify.com
Authorization: Bearer $APIFY_TOKEN
Content-Type: application/json
```

```bash
curl -sS -X POST \
  "https://api.apify.com/v2/acts/basta_compilar~google-maps-review-change-feed/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d @input.json
```

`input.json`:

```json
{
  "placeItems": [
    {
      "place": "https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia",
      "label": "Hortolândia",
      "clientName": "Demo Brand",
      "tags": ["brasil"]
    }
  ],
  "runMode": "monitor",
  "locale": "pt-BR",
  "mapsHost": "www.google.com.br",
  "sortBy": "newest",
  "reviewSource": "auto",
  "maxReviewsPerPlace": 20,
  "maxPlacesPerRun": 50,
  "maxBudgetUsd": 5,
  "historyStoreName": "google-maps-review-history",
  "includeFlatExport": true,
  "includeRunDigest": true,
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "emailMode": "digest",
  "webhookSlackCompatible": false
}
```

Open the place in Google Maps and paste that address-bar URL into `place`. A Maps search URL also works. Use the same `historyStoreName` for that portfolio on every run. A new name starts a new baseline.

Response (only the fields the samples read):

```json
{
  "data": {
    "id": "RUN_ID",
    "status": "READY",
    "defaultDatasetId": "DATASET_ID"
  }
}
```

## Poll

```bash
curl -sS \
  "https://api.apify.com/v2/actor-runs/RUN_ID" \
  -H "Authorization: Bearer $APIFY_TOKEN"
```

Wait a few seconds between calls. Stop when `data.status` is terminal.

## Read the Dataset

```bash
curl -sS \
  "https://api.apify.com/v2/datasets/DATASET_ID/items?format=json&clean=true" \
  -H "Authorization: Bearer $APIFY_TOKEN"
```

The body is a JSON array. Filter `recordType`.

| `recordType` | Meaning |
| --- | --- |
| `priority_action` | What to answer first. `priority` is `critical`, `high`, `medium`, or `low` |
| `change` | New, edited, owner reply, or possible removal |
| `place_summary` | Counts for one place |
| `place_error` | That place failed. The run can still be `SUCCEEDED` |
| `review_snapshot` | Unchanged review. Only when `includeUnchangedReviews` is true |
| `flat_export` | One flat row per review, for a sheet |
| `run_digest` | Totals for the whole run |

Useful fields on an action or a change: `placeName`, `placeUrl`, `clientName`, `label`, `priority`, `actionType`, `changeType`, `reason`, `review.rating`, `review.authorName`, `review.text`, `detectedAt`.

## Input fields used by integrations

| Field | Type | Role |
| --- | --- | --- |
| `places` | string[] | Maps URL or place id. Use this **or** `placeItems` |
| `placeItems` | object[] | Same places plus `label`, `clientName`, `tags`. Each item needs `place` |
| `runMode` | `monitor` \| `import_history` \| `reset_baseline` \| `export_snapshot` | `monitor` for schedules. `export_snapshot` is an alias of `import_history` |
| `historyStoreName` | string | Memory between runs. Default `google-maps-review-history` |
| `locale` | string | Browser language, e.g. `pt-BR`, `en-US` |
| `mapsHost` | string | `www.google.com` or `www.google.com.br` when a place id is turned into a URL |
| `sortBy` | `newest` \| `relevant` \| `highest` \| `lowest` | Default `newest` |
| `reviewSource` | `auto` \| `maps_panel` | `auto` uses the public reviews dialog, then the Maps panel |
| `maxReviewsPerPlace` | number | Monitor default 20, hard max 40. Import max 100 |
| `maxPlacesPerRun` | number | Cap of successful checks. Safety cap 50 if no budget is set |
| `maxBudgetUsd` | number | Soft cap at USD 0.10 per successful place |
| `negativeRatingThreshold` | number | Ratings at or below this are negative. Default 2 |
| `emitBaselineAsNew` | boolean | Default false. The first run stores history and still emits unanswered negatives |
| `includeFlatExport` | boolean | Default true |
| `includeRunDigest` | boolean | Default true |
| `webhookUrls` | string[] | JSON alerts. Included in the place price |
| `alertEmails` | string[] | Recipients. Sending is configured on the Actor, not in this payload |
| `emailMode` | `digest` \| `per_place` | `digest` is one email per run |
| `notifyOn` | `none` \| `priority_actions` \| `changes` \| `priority_and_changes` \| `all` | Default `priority_actions` |
| `minPriorityToNotify` | `critical` \| `high` \| `medium` \| `low` | Default `high` |
| `webhookSlackCompatible` | boolean | Add Slack `text` and `blocks` |
| `proxyConfiguration` | object | Apify proxy object for large or frequent lists |

Every field: [INPUT.en.md](../docs/INPUT.en.md) · [INPUT.pt-BR.md](../docs/INPUT.pt-BR.md).  
Pricing: [PRICING.en.md](../docs/PRICING.en.md). Webhook body: [WEBHOOKS.en.md](../docs/WEBHOOKS.en.md).

## Language guides

Same program, standard library only:

| Language | Guide |
| --- | --- |
| JavaScript | [javascript.md](javascript.md) |
| TypeScript | [typescript.md](typescript.md) |
| Python | [python.md](python.md) |
| Java | [java.md](java.md) |
| Kotlin | [kotlin.md](kotlin.md) |
| Go | [go.md](go.md) |
| C# | [csharp.md](csharp.md) |
| PHP | [php.md](php.md) |
| Ruby | [ruby.md](ruby.md) |

Rust is not listed: the language has no HTTP client in the standard library. The contract above is what a Rust program would call.
