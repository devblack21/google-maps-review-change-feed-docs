# JavaScript — get started

Node.js 18 or newer. `fetch` is built in. No npm install.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode: "monitor"` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your app reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `monitor.mjs`. Set `APIFY_TOKEN`. Paste a Google Maps address-bar URL, or a Maps search URL, into `place`.

```bash
APIFY_TOKEN=apify_api_xxx node monitor.mjs
```

```js
const token = process.env.APIFY_TOKEN;
if (!token) throw new Error('Set APIFY_TOKEN');

const actor = 'basta_compilar~google-maps-review-change-feed';
const input = {
    placeItems: [
        {
            place: 'https://www.google.com/maps/search/Ifome+Burguer+Hortol%C3%A2ndia',
            label: 'Hortolândia',
            clientName: 'Demo Brand',
            tags: ['brasil'],
        },
    ],
    runMode: 'monitor',
    locale: 'pt-BR',
    mapsHost: 'www.google.com.br',
    sortBy: 'newest',
    reviewSource: 'auto',
    maxReviewsPerPlace: 20,
    maxPlacesPerRun: 50,
    maxBudgetUsd: 5,
    historyStoreName: 'google-maps-review-history',
    includeFlatExport: true,
    includeRunDigest: true,
    notifyOn: 'priority_actions',
    minPriorityToNotify: 'high',
    emailMode: 'digest',
};

async function apify(method, path, body) {
    const response = await fetch('https://api.apify.com/v2' + path, {
        method,
        headers: {
            Authorization: `Bearer ${token}`,
            'Content-Type': 'application/json',
        },
        body: body === undefined ? undefined : JSON.stringify(body),
    });
    const text = await response.text();
    const json = text ? JSON.parse(text) : null;
    if (!response.ok) throw new Error(`${response.status} ${text}`);
    return json;
}

const started = await apify('POST', `/acts/${actor}/runs`, input);
const runId = started.data.id;
const datasetId = started.data.defaultDatasetId;

const terminal = new Set(['SUCCEEDED', 'FAILED', 'ABORTED', 'TIMED-OUT']);
let status = started.data.status;
for (let i = 0; i < 120 && !terminal.has(status); i += 1) {
    await new Promise((resolve) => setTimeout(resolve, 5000));
    status = (await apify('GET', `/actor-runs/${runId}`)).data.status;
}

const items = await apify('GET', `/datasets/${datasetId}/items?format=json&clean=true`);
const actions = items.filter((item) => item.recordType === 'priority_action');
console.log(JSON.stringify({ runId, status, actions }, null, 2));
if (status !== 'SUCCEEDED') process.exitCode = 1;
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
