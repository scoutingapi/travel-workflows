<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/rate-parity-monitor.ts. Edit the config + re-run. -->

# Cross-OTA rate-parity monitor → parity-exception digest — n8n

Watch how your own listing's rate shows across OTAs on a schedule and flag any platform underselling the market median.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `rate-parity-monitor.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer stay_live_YOUR_KEY` and assign it to the StayingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://stayingapi.com/signup>. Rendered detail page: [https://stayingapi.com/workflows/rate-parity-monitor](https://stayingapi.com/workflows/rate-parity-monitor).
