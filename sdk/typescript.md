# TypeScript — get started

Node.js 22.6 or newer. `fetch` is built in. No compiler and no package. On Node 22 pass `--experimental-strip-types`. Newer Node builds strip types without the flag.

Contract and field list: [api.md](api.md).

## Use cases

- Daily monitor: `runMode: "monitor"` and the same `historyStoreName`.
- Agency: `placeItems` with `label`, `clientName`, and `tags`.
- Your app reads `priority_action` rows after the run.
- Cap spend with `maxBudgetUsd` and `maxPlacesPerRun`. A successful place is USD 0.10.

## Run

Save as `monitor.ts`.

```bash
APIFY_TOKEN=apify_api_xxx node --experimental-strip-types monitor.ts
```

```ts
const token = process.env.APIFY_TOKEN;
if (!token) throw new Error('Set APIFY_TOKEN');

const actor = 'basta_compilar~google-maps-review-change-feed';

type RunMode = 'monitor' | 'import_history' | 'reset_baseline' | 'export_snapshot';
type NotifyOn = 'none' | 'priority_actions' | 'changes' | 'priority_and_changes' | 'all';

interface PlaceItem {
    place: string;
    label?: string;
    clientName?: string;
    tags?: string[];
}

interface ActorInput {
    placeItems: PlaceItem[];
    runMode: RunMode;
    locale: string;
    mapsHost: string;
    sortBy: 'newest' | 'relevant' | 'highest' | 'lowest';
    reviewSource: 'auto' | 'maps_panel';
    maxReviewsPerPlace: number;
    maxPlacesPerRun: number;
    maxBudgetUsd: number;
    historyStoreName: string;
    includeFlatExport: boolean;
    includeRunDigest: boolean;
    notifyOn: NotifyOn;
    minPriorityToNotify: 'critical' | 'high' | 'medium' | 'low';
    emailMode: 'digest' | 'per_place';
}

interface RunData {
    id: string;
    status: string;
    defaultDatasetId: string;
}

interface DatasetItem {
    recordType: string;
    placeName?: string;
    priority?: string;
    reason?: string;
    review?: { rating?: number; authorName?: string; text?: string };
}

const input: ActorInput = {
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

async function apify<T>(method: string, path: string, body?: unknown): Promise<T> {
    const response = await fetch('https://api.apify.com/v2' + path, {
        method,
        headers: {
            Authorization: `Bearer ${token}`,
            'Content-Type': 'application/json',
        },
        body: body === undefined ? undefined : JSON.stringify(body),
    });
    const text = await response.text();
    if (!response.ok) throw new Error(`${response.status} ${text}`);
    return JSON.parse(text) as T;
}

const started = await apify<{ data: RunData }>('POST', `/acts/${actor}/runs`, input);
const runId = started.data.id;
const datasetId = started.data.defaultDatasetId;
const terminal = new Set(['SUCCEEDED', 'FAILED', 'ABORTED', 'TIMED-OUT']);
let status = started.data.status;

for (let i = 0; i < 120 && !terminal.has(status); i += 1) {
    await new Promise((resolve) => setTimeout(resolve, 5000));
    status = (await apify<{ data: RunData }>('GET', `/actor-runs/${runId}`)).data.status;
}

const items = await apify<DatasetItem[]>('GET', `/datasets/${datasetId}/items?format=json&clean=true`);
const actions = items.filter((item) => item.recordType === 'priority_action');
console.log(JSON.stringify({ runId, status, actions }, null, 2));
if (status !== 'SUCCEEDED') process.exitCode = 1;
```

Keep `historyStoreName` stable for that portfolio. The first successful run is the baseline.

Full field list: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
