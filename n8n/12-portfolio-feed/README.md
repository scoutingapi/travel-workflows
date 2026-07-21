<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/portfolio-feed.ts. Edit the config + re-run. -->

# Portfolio price & occupancy feed — n8n

A daily digest across a saved set of listings — price, rating and recent reviews per listing in one report.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `portfolio-feed.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer stay_live_YOUR_KEY` and assign it to the StayingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://stayingapi.com/signup>. Rendered detail page: [https://stayingapi.com/workflows/portfolio-feed](https://stayingapi.com/workflows/portfolio-feed).
