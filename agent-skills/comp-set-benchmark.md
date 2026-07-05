---
name: scoutingapi-comp-set-benchmark
description: Benchmark your listing’s rate and availability against a defined comp-set and get a weekly position report. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/comp-set-benchmark
license: Proprietary — see https://scoutingapi.com/terms
tools: [price, availability, listing]
auth: bearer-api-key | oauth2-pkce
---

# Competitor rate & availability benchmark → report

For a defined comp-set of listings, pull each one’s price and day-by-day availability, then benchmark your own listing against the set — where your rate ranks, the cheapest and median rates, and an occupancy signal from how many dates each comp has open. It fans the comp-set out, calls GET /v1/price and GET /v1/availability per listing, and correlates both into one report. A revenue-management staple, on a schedule.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **Define the comp-set and your listing** — List the platform, the comma-separated listing ids for the whole comp-set (including your own), which id is yours, and the stay dates.
2. **Price every listing in the set** — The workflow fans the comp-set out and calls GET /v1/price for each listing and the dates, returning a real total per competitor.
3. **Check each listing’s availability** — It also calls GET /v1/availability per listing over the window, so occupancy density (how many dates each comp has open) feeds the benchmark.
4. **Build and deliver the benchmark** — A Code node correlates price and availability by listing, ranks the rates, flags where you sit versus the cheapest and median, and posts the report to Slack — swap for email or a Google Sheet for a weekly digest.

## When to use this

A host, PM or revenue manager wants to benchmark a listing against a defined comp-set — "how does
my rate rank against these five competitors for this week, and how full are they?"

## How to do it

1. Take the `platform`, the comp-set `listingIds` (including your own), `yourListingId`, and the
   `checkIn`/`checkOut` dates.
2. For each listing in the set: call `GET /v1/price` (rate) and `GET /v1/availability` over the
   window (occupancy). Use `@scoutingapi/sdk` or the REST endpoints.
3. Correlate by `listingId`: sort the totals to rank your listing, compute the cheapest and median,
   and count each comp's bookable days as a demand signal.

## Cost & scheduling

Cost scales with the comp-set size (one price + one availability call per listing). Run it weekly on
a schedule for a recurring position report. Current per-call credit costs are on the pricing page.

## Partial failures

A listing with no price (a typed error) is free — treat it as "no reading" and rank the rest.
Check `meta.warnings[]` and skip any comp that could not be resolved this run.

### Example — REST

```bash
# Price one comp (repeat per listing in the set)…
curl -sS "https://api.scoutingapi.com/v1/price?platform=booking&listingId=abramovic2&checkIn=2026-07-13&checkOut=2026-07-20&currency=EUR" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
# …and its availability over the same window
curl -sS "https://api.scoutingapi.com/v1/availability?platform=booking&listingId=abramovic2&startDate=2026-07-13&endDate=2026-07-20&onlyAvailable=true" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
```

### Example — @scoutingapi/sdk

```ts
import { ScoutingApiClient } from '@scoutingapi/sdk';

const scout = new ScoutingApiClient({ apiKey: process.env.SCOUTINGAPI_KEY });
const platform = 'booking';
const compSet = ['abramovic2', 'amadria-park', 'hotel-x'];
const yourId = 'abramovic2';
const dates = { checkIn: '2026-07-13', checkOut: '2026-07-20', currency: 'EUR' };

const rows = [];
for (const listingId of compSet) {
  const { data } = await scout.price({ platform, listingId, ...dates });
  rows.push({ listingId, total: data.totalPrice, currency: data.currency });
}
rows.sort((a, b) => a.total - b.total);
const rank = rows.findIndex((r) => r.listingId === yourId) + 1;
console.log('your rank', rank, 'of', rows.length, '· cheapest', rows[0].total);
```

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

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/comp-set-benchmark
