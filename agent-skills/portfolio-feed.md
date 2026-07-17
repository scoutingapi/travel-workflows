---
name: scoutingapi-portfolio-feed
description: A daily digest across a saved set of listings — price, rating and recent reviews per listing in one report. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/portfolio-feed
license: Proprietary — see https://scoutingapi.com/terms
tools: [listing, price, reviews]
auth: bearer-api-key | oauth2-pkce
---

# Portfolio price & occupancy feed

Owners and property managers watch a fixed set of listings — their own units, or the ones a client cares about — and want one daily readout instead of opening each app. This feed takes your saved "platform:listingId" set, and for each listing pulls full detail (GET /v1/listing/{platform}/{id}), a real nightly price for your dates (GET /v1/price) and the latest reviews (GET /v1/reviews), then consolidates everything into a single report you can drop into Slack, email, a Google Sheet or a BI tool — the same shape across Airbnb, Booking.com and Vrbo.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com/mcp`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **List the portfolio and the dates** — Set your saved listings as "platform:listingId" pairs plus the check-in / check-out to price against. A Code node fans the set out into one item per listing.
2. **Pull detail, price and reviews per listing** — For each listing the pipeline calls GET /v1/listing/{platform}/{id} (name, rating, amenities), GET /v1/price (a real nightly + total for your dates) and GET /v1/reviews (the latest few).
3. **Consolidate into one digest** — A Code node correlates the three responses per listing, computes the portfolio median nightly, and formats one report — price, rating and recent-review count per listing.
4. **Deliver daily** — Post the digest to Slack or email, or append rows to a Google Sheet / BI feed for tracking over time. Runs on a daily schedule.

## When to use this

A host, owner or property manager wants one daily readout across a fixed set of listings — their
own units or a client's — instead of opening each platform's app.

## How to do it

For each saved `platform:listingId` in the set, make three calls and join them:

1. `GET /v1/listing/{platform}/{id}` (pass `checkIn`/`checkOut` to embed a best-effort price) →
   name, `guestRating`, `ratingScale`, amenities.
2. `GET /v1/price?platform=&listingId=&checkIn=&checkOut=` → a real `nightlyPrice` + `totalPrice`
   for the exact dates.
3. `GET /v1/reviews?platform=&listingId=&limit=5&sort=recent` → the latest guest reviews.

Roll one row per listing (price, rating, recent-review count) plus a portfolio median into a single
digest. Use `@scoutingapi/sdk` (`scout.listing`, `scout.price`, `scout.reviews`) or the REST
endpoints directly.

## Reliability

Live calls run asynchronously — the SDK auto-polls; raw REST callers poll `GET /v1/jobs/{id}`.
A blocked or empty call is charged 0 and simply shows as "price n/a" for that listing, so one bad
platform never sinks the whole digest. Current per-call credit costs are on the pricing page.

### Example — REST

```bash
# For each saved listing: detail, a real price for your dates, and recent reviews.
curl -sS "https://api.scoutingapi.com/v1/listing/airbnb/42307961?checkIn=2026-07-13&checkOut=2026-07-20" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
curl -sS "https://api.scoutingapi.com/v1/price?platform=airbnb&listingId=42307961&checkIn=2026-07-13&checkOut=2026-07-20" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
curl -sS "https://api.scoutingapi.com/v1/reviews?platform=airbnb&listingId=42307961&limit=5&sort=recent" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
```

### Example — @scoutingapi/sdk

```ts
import { ScoutingApiClient } from '@scoutingapi/sdk';

const scout = new ScoutingApiClient({ apiKey: process.env.SCOUTINGAPI_KEY });

const portfolio = [
  { platform: 'airbnb', listingId: '42307961' },
  { platform: 'booking', listingId: 'abramovic2' },
];
const dates = { checkIn: '2026-07-13', checkOut: '2026-07-20' };

const digest = [];
for (const { platform, listingId } of portfolio) {
  const [{ data: listing }, { data: price }, { data: reviews }] = await Promise.all([
    scout.listing(platform, listingId, dates),
    scout.price({ platform, listingId, ...dates }),
    scout.reviews({ platform, listingId, limit: 5, sort: 'recent' }),
  ]);
  digest.push({
    name: listing.name,
    nightly: price?.nightlyPrice ?? null,
    rating: listing.guestRating,
    recentReviews: reviews.length,
  });
}
console.table(digest);
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

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/portfolio-feed
