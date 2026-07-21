<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/cross-ota-price-comparison.ts. Edit the config + re-run. -->

# Multi-source price comparison → cheapest-rate report — n8n

Compare one property across booking platforms and report the cheapest rate, with savings vs. the median.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `cross-ota-price-comparison.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer stay_live_YOUR_KEY` and assign it to the StayingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://stayingapi.com/signup>. Rendered detail page: [https://stayingapi.com/workflows/cross-ota-price-comparison](https://stayingapi.com/workflows/cross-ota-price-comparison).
