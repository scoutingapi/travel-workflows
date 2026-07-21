<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/price-drop-watcher.ts. Edit the config + re-run. -->

# Price-drop watcher — n8n

Watch a specific stay and get alerted the moment its total price falls to or below your threshold.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `price-drop-watcher.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer stay_live_YOUR_KEY` and assign it to the StayingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://stayingapi.com/signup>. Rendered detail page: [https://stayingapi.com/workflows/price-drop-watcher](https://stayingapi.com/workflows/price-drop-watcher).
