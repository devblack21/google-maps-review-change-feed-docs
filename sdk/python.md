# Python — get started

Python 3.9 or newer. Uses `urllib` from the standard library. No pip install.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode: "monitor"` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your app reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `monitor.py`.

```bash
APIFY_TOKEN=apify_api_xxx python3 monitor.py
```

```python
import json
import os
import time
import urllib.request

token = os.environ["APIFY_TOKEN"]
actor = "basta_compilar~google-maps-review-change-feed"
input_body = {
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
    "mapsHost": "www.google.com.br",
    "sortBy": "newest",
    "reviewSource": "auto",
    "maxReviewsPerPlace": 20,
    "maxPlacesPerRun": 50,
    "maxBudgetUsd": 5,
    "historyStoreName": "google-maps-review-history",
    "includeFlatExport": True,
    "includeRunDigest": True,
    "notifyOn": "priority_actions",
    "minPriorityToNotify": "high",
    "emailMode": "digest",
}


def apify(method, path, body=None):
    data = None if body is None else json.dumps(body).encode()
    request = urllib.request.Request(
        "https://api.apify.com/v2" + path,
        data=data,
        method=method,
        headers={
            "Authorization": f"Bearer {token}",
            "Content-Type": "application/json",
        },
    )
    with urllib.request.urlopen(request, timeout=60) as response:
        return json.load(response)


started = apify("POST", f"/acts/{actor}/runs", input_body)
run_id = started["data"]["id"]
dataset_id = started["data"]["defaultDatasetId"]
status = started["data"]["status"]
terminal = {"SUCCEEDED", "FAILED", "ABORTED", "TIMED-OUT"}

while status not in terminal:
    time.sleep(5)
    status = apify("GET", f"/actor-runs/{run_id}")["data"]["status"]

items = apify("GET", f"/datasets/{dataset_id}/items?format=json&clean=true")
actions = [item for item in items if item.get("recordType") == "priority_action"]
print(json.dumps({"runId": run_id, "status": status, "actions": actions}, ensure_ascii=False, indent=2))
if status != "SUCCEEDED":
    raise SystemExit(1)
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
