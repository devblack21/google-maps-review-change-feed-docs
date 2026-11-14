# Python SDK

Use the official [`apify-client`](https://docs.apify.com/api/client/python/) with your Apify API token. Create the token in Apify Console → Settings → Integrations. Keep it in an environment variable.

```bash
pip install apify-client
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("basta_compilar/google-maps-review-change-feed").call(run_input={
    "placeItems": [
        {
            "place": "https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia",
            "label": "Hortolândia",
            "clientName": "Demo Brand",
            "tags": ["brasil"],
        }
    ],
    "runMode": "monitor",
    "locale": "pt-BR",
    "maxReviewsPerPlace": 20,
    "maxPlacesPerRun": 50,
    "maxBudgetUsd": 5,
    "historyStoreName": "google-maps-review-history",
    "includeFlatExport": True,
    "notifyOn": "priority_actions",
    "minPriorityToNotify": "high",
    "webhookSlackCompatible": True,
})

items = client.dataset(run["defaultDatasetId"]).list_items().items
actions = [item for item in items if item.get("recordType") == "priority_action"]
print(actions)
```

Use the **same** `historyStoreName` on every call for that portfolio. The first successful run is the baseline.

`call()` waits until the run finishes. For a daily check, save the same input as an Apify Task and attach a Schedule in the Console.

More fields: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
