---
name: scoutingapi-review-intelligence-digest
description: Pull normalized reviews for a property on a schedule and deliver a weekly digest — average rating, recurring themes, and the latest complaints to act on. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/review-intelligence-digest
license: Proprietary — see https://scoutingapi.com/terms
tools: [reviews, listing]
auth: bearer-api-key | oauth2-pkce
---

# Review-intelligence digest → weekly reputation report

Reputation is spread across platforms and scored on different scales, so problems surface late. This workflow pulls normalized reviews for a property (native rating scales preserved, never silently rescaled), summarizes the average, the recurring themes, and the most recent low ratings, and delivers a weekly reputation digest. On the AI-agent path the agent reads the reviews and writes the sentiment summary itself; on the n8n path the aggregation is plain JavaScript, so no LLM key is required — add an LLM node only if you want richer narrative.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **Choose the property to monitor** — Give the workflow a platform and the platform-native listing id (from a prior /v1/search), how many reviews to pull, and a sort order (recent is the default for a reputation watch).
2. **Pull the normalized reviews** — It calls GET /v1/reviews, which returns reviews in one schema with each native rating scale preserved and echoed (5 for Airbnb/Vrbo/TripAdvisor, 10 for Booking.com/Expedia/Hotels.com).
3. **Summarize sentiment and themes** — The agent reads the reviews and writes the summary; the n8n transform aggregates an average on the native scale, a theme-frequency table from the text, and the most recent low ratings — no LLM key needed.
4. **Deliver the weekly digest** — A reputation digest (average, top themes, recent complaints) is posted to Slack or email on your cadence — swap the last node or add an LLM node for a more narrative write-up.

## When to use this

The user is a host or property manager who wants to **monitor reputation** for a property (or a
comp-set) across platforms in one schema. Reach for `get_reviews` / `GET /v1/reviews`: normalized,
paginated reviews with each **native rating scale preserved and echoed**.

## How to do it

1. Call `GET /v1/reviews` with `platform` + `listingId` (or `url`), `limit`, `sort=recent`.
   Paginate with `meta.pagination.nextCursor` if you need more than one page.
2. Read `ratingScale` on each review and **normalize before averaging** (5 vs 10 scales differ).
3. Summarize: the average on the native scale, recurring themes/complaints in `text`, and the most
   recent low ratings (e.g. rating ≤ 60% of scale). YOU are the summarizer on the agent path.

## Delivering a weekly digest

Run on a weekly cadence and post the summary to Slack/email/Notion. Lead with the average and its
trend, then the top themes, then the specific recent complaints worth a reply.

## Comp-set version

To benchmark against competitors, repeat the reviews call for each listing id in the comp-set and
compare normalized averages; attach names via `GET /v1/listing/{platform}/{id}`.

### Example — REST

```bash
curl -sS "https://api.scoutingapi.com/v1/reviews?platform=booking&listingId=abramovic2&limit=40&sort=recent" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
```

### Example — @scoutingapi/sdk

```ts
import { ScoutingApiClient } from '@scoutingapi/sdk';

const scout = new ScoutingApiClient({ apiKey: process.env.SCOUTINGAPI_KEY });

const { data } = await scout.reviews({
  platform: 'booking',
  listingId: 'abramovic2',
  limit: 40,
  sort: 'recent',
});

const scale = data[0]?.ratingScale ?? 5;
const avg = data.reduce((s, r) => s + r.rating, 0) / data.length;
console.log(data.length, 'reviews · avg', avg.toFixed(2), '/', scale);
```

## MCP (no key pasted into the agent)

On an MCP-capable runtime, connect the ScoutingAPI server at **https://mcp.scoutingapi.com** (OAuth 2.1 + PKCE) and use:
- `get_reviews` — normalized, paginated reviews for one listing.

## Async & partial failures

A live call that has to scrape returns `202` + a `jobId`; poll `GET /v1/jobs/{jobId}` (free)
until `data.status` is `completed`, then read `data.result`. The `@scoutingapi/sdk` auto-polls.
On a fan-out, check `meta.partial` and `meta.platformResults[]` — report what succeeded and note
any `meta.warnings[]`.

## Credit awareness

Costs are per-endpoint and metered by result (v3). **Failed, empty and blocked calls are never
billed**, and sandbox (`scout_test_`) calls are always free. The exact, current costs live only in
https://scoutingapi.com/pricing and the machine-readable https://api.scoutingapi.com/openapi.json — read them there, don't assume.

---

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/review-intelligence-digest
