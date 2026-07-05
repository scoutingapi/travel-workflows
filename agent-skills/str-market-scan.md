---
name: scoutingapi-str-market-scan
description: Scan a short-term-rental market — discover listings across platforms, price the top N for your dates, and append a normalized row-per-listing dataset to a Google Sheet. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/str-market-scan
license: Proprietary — see https://scoutingapi.com/terms
tools: [search, price, listing]
auth: bearer-api-key | oauth2-pkce
---

# STR market scan → Google Sheet dataset

Underwriting a short-term-rental market means one repeatable dataset: what is listed, on which platform, at what price, with how many beds and what rating. This workflow fans a search across platforms to discover listings in your area and dates, keeps the top N, re-prices each precisely with the dedicated price endpoint, and appends a normalized row per listing — name, platform, beds, occupancy, rating (on its native scale), nightly and total price, and URL — to a Google Sheet or CSV. Re-run it on a cadence to build a time series for comps and occupancy signals.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **Define the market + dates** — Give the workflow a location, the check-in / check-out dates, occupancy, which platforms to scan, how many results to discover per platform, and how many of them to price precisely.
2. **Discover listings across platforms** — It calls GET /v1/search across the requested platforms and merges every match into one normalized Property list — the discovery pass.
3. **Price the top N precisely** — It ranks the discovered listings, keeps the top N, and calls GET /v1/price for each with the exact dates and occupancy — a precise per-date quote per listing.
4. **Append the market dataset** — Each listing becomes one normalized row (name, platform, beds, occupancy, rating + scale, nightly + total price, URL) appended to a Google Sheet or CSV — re-run on a cadence for a time series.

## When to use this

The user (an STR investor/analyst or a data team) wants a **repeatable market dataset**: every
listing in an area + dates, with price, beds, rating and URL, as rows to underwrite or trend.

## How to do it (two passes)

1. **Discover** — `GET /v1/search` with `location` (+ `checkIn`/`checkOut`, `platforms`, `limit`,
   `sort`, `currency`). Merge `data[]` across platforms; each Property already carries a
   `price`, `bedrooms`, `guestRating`/`ratingScale`, `url`.
2. **Price precisely** — for the top N listings, call `GET /v1/price` with `platform` +
   `listingId` (the `platformListingId`) + the exact dates. Use `price.totalPrice`; fall back to the
   search price if a quote is empty.
3. **Emit one normalized row per listing**: name, platform, listingId, city, bedrooms,
   maxOccupancy, guestRating+ratingScale, nightlyPrice, totalPrice, currency, url.

## Cost & scope control

Cost scales with the search breadth (`limit` × platforms) and the number of listings you price
(`topN`). Start small; failed/empty legs are never billed and sandbox is free. Optionally enrich a
row with `GET /v1/listing/{platform}/{id}` for amenities/photos.

## Time series

Re-run on a schedule and append a run-date column to build a comps-and-occupancy time series.

### Example — REST

```bash
# 1) Discover listings in the market
curl -sS "https://api.scoutingapi.com/v1/search?location=Split,HR&checkIn=2026-07-13&checkOut=2026-07-20&platforms=airbnb,vrbo,booking&sort=price_asc&limit=20" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"

# 2) Price one discovered listing precisely (repeat for your top N)
curl -sS "https://api.scoutingapi.com/v1/price?platform=airbnb&listingId=42307961&checkIn=2026-07-13&checkOut=2026-07-20&adults=2&currency=EUR" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
```

### Example — @scoutingapi/sdk

```ts
import { ScoutingApiClient } from '@scoutingapi/sdk';

const scout = new ScoutingApiClient({ apiKey: process.env.SCOUTINGAPI_KEY });
const dates = { checkIn: '2026-07-13', checkOut: '2026-07-20', adults: 2, currency: 'EUR' };

// 1) discover
const { data: listings } = await scout.search({
  location: 'Split, HR',
  platforms: ['airbnb', 'vrbo', 'booking'],
  sort: 'price_asc',
  limit: 20,
  ...dates,
});

// 2) price the top N and build a dataset row each
const topN = listings.slice(0, 10);
const rows = [];
for (const p of topN) {
  const { data: price } = await scout.price({
    platform: p.platform,
    listingId: p.platformListingId,
    ...dates,
  });
  rows.push({
    name: p.name,
    platform: p.platform,
    beds: p.bedrooms,
    rating: p.guestRating,
    total: price.totalPrice ?? p.price?.totalPrice,
    url: p.url,
  });
}
console.table(rows);
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

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/str-market-scan
