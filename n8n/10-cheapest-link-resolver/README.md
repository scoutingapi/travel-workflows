<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/cheapest-link-resolver.ts. Edit the config + re-run. -->

# Booking-ready cheapest-link resolver — n8n

Resolve a property + dates to the single cheapest bookable link (with price and OTA) as clean JSON — for a concierge bot, a "book now" button, or an internal tool.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `cheapest-link-resolver.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer scout_live_YOUR_KEY` and assign it to the ScoutingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://scoutingapi.com/signup>. Rendered detail page: [https://scoutingapi.com/workflows/cheapest-link-resolver](https://scoutingapi.com/workflows/cheapest-link-resolver).
