# Webhooks guide (EN)

🇧🇷 Português: [WEBHOOKS.pt-BR.md](./WEBHOOKS.pt-BR.md)

Webhooks push alerts out of Apify into Slack, Make, Zapier, n8n, or your own API.

Delivery is **included** in the `place-monitored` PPE event. Failures and resends are **never** billed separately.

---

## Quick setup

```json
{
  "places": ["ChIJxxxxxxxx"],
  "webhookUrls": [
    "https://hooks.slack.com/services/T000/B000/XXXXXXXX"
  ],
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "webhookSlackCompatible": true
}
```

### `notifyOn`
| Value | Sends |
| --- | --- |
| `none` | Nothing |
| `priority_actions` | Priority queue only (recommended) |
| `changes` | Change events only |
| `priority_and_changes` | Both |
| `all` | Always send a payload (even if empty lists) |

### `minPriorityToNotify`
`critical` · `high` · `medium` · `low`  
Only priority actions at this level or above are included.

---

## Payload example (Slack-compatible)

```json
{
  "text": "*Google Maps Review Change Feed* — Padaria Aurora — Paulista\nPriority actions: 1\nhttps://www.google.com/maps/place/...",
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Google Maps Review Change Feed* — Padaria Aurora — Paulista\nPriority actions: 1\nhttps://www.google.com/maps/place/..."
      }
    }
  ],
  "source": "google-maps-review-change-feed",
  "place": {
    "placeKey": "ChIJ...",
    "placeId": "ChIJ...",
    "placeName": "Padaria Aurora — Paulista",
    "placeUrl": "https://www.google.com/maps/place/...",
    "label": "Paulista",
    "clientName": "Padaria Aurora",
    "tags": ["sp", "capital"]
  },
  "priorityActions": [
    {
      "priority": "critical",
      "actionType": "new_negative",
      "reason": "New 1-star review without owner response",
      "rating": 1,
      "authorName": "Carla",
      "text": "Fila enorme e pão frio",
      "hasOwnerResponse": false,
      "reviewUrl": "https://..."
    }
  ],
  "changes": [],
  "detectedAt": "2026-11-11T12:00:00.000Z"
}
```

---

## Make / Zapier / n8n

1. Create a Catch Hook / Webhook module  
2. Paste the URL into `webhookUrls`  
3. Map fields:
   - `place.clientName`
   - `place.placeName`
   - `priorityActions[0].reason`
   - `priorityActions[0].reviewUrl`
4. Route to Slack, email, CRM task, WhatsApp Business API, etc.

---

## Reliability notes

- Failed webhooks are logged and counted — they **do not** fail the Actor run
- Multiple URLs are supported (fan-out)
- No webhook URL configured → silent Dataset-only mode
- Tested in CI with mocked HTTP success/failure paths

---

## Recommended production defaults

```json
{
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "webhookSlackCompatible": true
}
```

This keeps noise low and focuses the team on unanswered negatives and urgent rating issues.
