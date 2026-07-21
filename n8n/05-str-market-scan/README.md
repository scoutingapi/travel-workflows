<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/str-market-scan.ts. Edit the config + re-run. -->

# STR market scan → Google Sheet dataset — n8n

Scan a short-term-rental market — discover listings across platforms, price the top N for your dates, and append a normalized row-per-listing dataset to a Google Sheet.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `str-market-scan.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer stay_live_YOUR_KEY` and assign it to the StayingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://stayingapi.com/signup>. Rendered detail page: [https://stayingapi.com/workflows/str-market-scan](https://stayingapi.com/workflows/str-market-scan).
