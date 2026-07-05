<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/availability-alert.ts. Edit the config + re-run. -->

# Property availability watch → alert — n8n

Watch a listing over a date window and get alerted the moment a wanted date becomes available.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `availability-alert.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer scout_live_YOUR_KEY` and assign it to the ScoutingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://scoutingapi.com/signup>. Rendered detail page: [https://scoutingapi.com/workflows/availability-alert](https://scoutingapi.com/workflows/availability-alert).
