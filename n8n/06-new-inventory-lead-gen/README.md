<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/new-inventory-lead-gen.ts. Edit the config + re-run. -->

# New-inventory monitor → CRM lead feed — n8n

Re-run a saved search on a schedule, diff it against the last run, and sync only the newly-listed properties into a CRM or Sheet as leads.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `new-inventory-lead-gen.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer stay_live_YOUR_KEY` and assign it to the StayingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://stayingapi.com/signup>. Rendered detail page: [https://stayingapi.com/workflows/new-inventory-lead-gen](https://stayingapi.com/workflows/new-inventory-lead-gen).
