# JavaScript SDK

Use the official [`apify-client`](https://docs.apify.com/api/client/js/) with your Apify API token. Create the token in Apify Console → Settings → Integrations. Keep it in an environment variable.

```bash
npm install apify-client
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('basta_compilar/google-maps-review-change-feed').call({
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
    maxReviewsPerPlace: 20,
    maxPlacesPerRun: 50,
    maxBudgetUsd: 5,
    historyStoreName: 'google-maps-review-history',
    includeFlatExport: true,
    notifyOn: 'priority_actions',
    minPriorityToNotify: 'high',
    webhookSlackCompatible: true,
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
const actions = items.filter((item) => item.recordType === 'priority_action');
console.log(actions);
```

Use the **same** `historyStoreName` on every call for that portfolio. The first successful run is the baseline.

`call()` waits until the run finishes. For a schedule, save the same input as an Apify Task and attach a Schedule in the Console. Code samples do not replace that schedule.

More fields: [INPUT.en.md](../docs/INPUT.en.md). Pricing: [PRICING.en.md](../docs/PRICING.en.md).
