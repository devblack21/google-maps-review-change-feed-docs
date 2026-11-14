# Support

Use a public channel. Do not send API tokens, SMTP passwords, or Google app passwords.

## Ask here

1. [Open an issue](https://github.com/devblack21/google-maps-review-change-feed-docs/issues/new) in this repository.
2. Or open an issue on the [Actor page](https://apify.com/basta_compilar/google-maps-review-change-feed).

## What to include

- What you expected and what happened
- The run URL from Apify Console (remove the token from the link)
- `runMode`, `historyStoreName`, and how many places were in the input
- Whether it was the first run (baseline) or a scheduled run
- A sample place URL, with personal data removed if needed

## Before you write

- The same `historyStoreName` must be used on every run of that portfolio. A new name starts a new baseline.
- The first successful run stores reviews. It does not flood you with “everything is new”.
- A place that fails does not, by itself, fail the whole run. Check `place_error` records and `RUN_SUMMARY`.
- You are charged **USD 0.10** only for a place that was successfully verified. See [pricing](docs/PRICING.en.md).

## What we can change

Input behavior, alerts, scheduling, and the published Actor. Feature requests belong in an issue with the use case (how many locations, how often, which channel).
