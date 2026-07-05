<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from content/workflows/review-intelligence-digest.ts. Edit the config + re-run. -->

# Review-intelligence digest → weekly reputation report — n8n

Pull normalized reviews for a property on a schedule and deliver a weekly digest — average rating, recurring themes, and the latest complaints to act on.

## Import

1. In n8n: **Workflows → ⋯ → Import from File** and select `review-intelligence-digest.json`.
2. Create a **Header Auth** credential named `Authorization` with value `Bearer scout_live_YOUR_KEY` and assign it to the ScoutingAPI HTTP node(s).
3. Set your inputs, add your delivery credential (e.g. Slack), and run.

Get a free key (no card) at <https://scoutingapi.com/signup>. Rendered detail page: [https://scoutingapi.com/workflows/review-intelligence-digest](https://scoutingapi.com/workflows/review-intelligence-digest).
