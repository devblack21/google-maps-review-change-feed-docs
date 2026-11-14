# Pricing (EN)

🇧🇷 Português: [PRICING.pt-BR.md](./PRICING.pt-BR.md)

You pay **per successful place check**, plus normal Apify platform usage on your account.

## What you pay the Actor

| Event | When it charges | Price |
| --- | --- | --- |
| `place-monitored` | Each place **successfully verified** in a run | **USD 0.10** |

Included in that price: detected changes, priority actions, webhook delivery, and email alerts.

Not charged as an Actor event:

- a place that failed to collect
- retries
- webhook or email resends
- places skipped because `maxBudgetUsd` or `maxPlacesPerRun` was reached

A quiet day with no new reviews still charges **USD 0.10** for each place that was checked successfully. The check is the product.

## What Apify bills besides the event

Compute, proxy, and storage are Apify platform usage on **your** account. They are separate from the USD 0.10 event.

## Examples (Actor events only, 30 days)

| Setup | Approx. event total |
| --- | --- |
| 10 places × once a day | ~USD 30 |
| 30 places × once a day | ~USD 90 |
| 30 places × 4 times a day | ~USD 360 |

Cap a single run with `maxBudgetUsd` and `maxPlacesPerRun`.

## Related

- [How to use](./USAGE.en.md)
- [Input fields](./INPUT.en.md)
