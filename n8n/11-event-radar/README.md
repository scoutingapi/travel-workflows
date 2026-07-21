<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/event-radar.ts. Edit the config + re-run. -->

# Event / peak-date demand radar — n8n

Watch supply and prices around an event or peak date-window in a city and get alerted when demand tightens.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `event-radar.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer stay_live_YOUR_KEY` and assign it to the StayingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://stayingapi.com/signup>. Rendered detail page: [https://stayingapi.com/workflows/event-radar](https://stayingapi.com/workflows/event-radar).
