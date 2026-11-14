# Guia de Webhooks (PT-BR)

🇬🇧 English: [WEBHOOKS.en.md](./WEBHOOKS.en.md)

Webhooks enviam alertas do Apify para Slack, Make, Zapier, n8n ou sua própria API.

A entrega está **incluída** no evento PPE `place-monitored`. Falhas e reenvios **nunca** são cobrados à parte.

---

## Setup rápido

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
| Valor | Envia |
| --- | --- |
| `none` | Nada |
| `priority_actions` | Só a fila de prioridade (recomendado) |
| `changes` | Só eventos de mudança |
| `priority_and_changes` | Os dois |
| `all` | Sempre envia payload (mesmo com listas vazias) |

### `minPriorityToNotify`
`critical` · `high` · `medium` · `low`  
Só entram ações com essa prioridade ou mais alta.

---

## Exemplo de payload (compatível com Slack)

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

1. Crie um Catch Hook / módulo Webhook  
2. Cole a URL em `webhookUrls`  
3. Mapeie os campos:
   - `place.clientName`
   - `place.placeName`
   - `priorityActions[0].reason`
   - `priorityActions[0].reviewUrl`
4. Encaminhe para Slack, e-mail, tarefa no CRM, WhatsApp Business API, etc.

---

## Confiabilidade

- Webhook com falha é logado e contado — **não** derruba a run do Actor
- Várias URLs são suportadas (fan-out)
- Sem URL configurada → só Dataset
- Coberto por testes de CI (sucesso/falha HTTP mockados)

---

## Defaults recomendados em produção

```json
{
  "notifyOn": "priority_actions",
  "minPriorityToNotify": "high",
  "webhookSlackCompatible": true
}
```

Isso reduz ruído e foca o time em negativas sem resposta e urgências de nota.
