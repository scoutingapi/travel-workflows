---
name: scoutingapi-rate-parity-monitor
description: Watch how your own listing's rate shows across OTAs on a schedule and flag any platform underselling the market median. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/rate-parity-monitor
license: Proprietary — see https://scoutingapi.com/terms
tools: [price-compare, listing]
auth: bearer-api-key | oauth2-pkce
---

# Cross-OTA rate-parity monitor → parity-exception digest

Rate parity erodes quietly: one OTA quietly undersells the rate you want your property to hold, and you only find out when direct bookings dry up. This monitor calls the flagship price-compare endpoint for your property on a schedule, ranks every OTA offer, measures each against the ScoutingAPI-computed median, and posts a parity-exception digest — which platforms are below the median, by how much, and the total spread — so a revenue manager can act on a paper trail instead of a hunch.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com/mcp`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **Point it at your property + a target stay** — Give the workflow your property name (plus an optional location to disambiguate), the check-in / check-out dates and occupancy to monitor, and a currency.
2. **Pull every OTA rate for the property** — On each scheduled run it calls GET /v1/price-compare, which resolves the one property and returns every OTA offer for those dates plus ScoutingAPI-computed min and median aggregates.
3. **Flag the parity exceptions** — The transform ranks the offers, measures each against the median, and flags any OTA sitting below it — the platforms undercutting the rate you want to hold — and computes the overall spread.
4. **Deliver the parity digest** — A digest (median, spread, each OTA with its delta vs. median, and the flagged exceptions) is posted to Slack — swap the last node for email or a Google Sheet to keep a running parity log.

## When to use this

The user is a host, property manager or revenue manager who wants to know whether their own
property is being **undersold on any OTA** — a rate-parity problem — rather than shopping for the
cheapest deal. Reach for the flagship `price-compare` endpoint: it resolves one property and
returns every OTA offer for the dates, plus ScoutingAPI-computed `min` and `median`.

## How to do it

1. Resolve the property with **one** of `name`, `googleHotelId`, or `location` (+ `checkIn`,
   `checkOut`, optional `adults`, `currency`).
2. Call `GET /v1/price-compare`.
3. Rank `data.offers[]` by `totalPrice`. Compare each against `data.median` (or the host's stated
   target/direct rate if they gave you one). **Flag every OTA below that reference** — those are the
   parity exceptions — and report the spread (`max - min`).

## Reading the result

Report, per exception: the OTA, its `totalPrice`, the delta vs. the reference, and the `url`.
Lead with the count of exceptions and the largest gap. If `meta.partial` is true, note which
platforms are missing from `meta.platformResults[]`.

## Scheduling

This is a monitor: run it on a cadence (daily is typical) and deliver a recurring digest, so parity
drift is caught the day it appears rather than at month-end.

### Example — REST

```bash
curl -sS "https://api.scoutingapi.com/v1/price-compare?name=Hotel%20X%20Sibenik&location=Sibenik,HR&checkIn=2026-07-13&checkOut=2026-07-20&currency=EUR" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
```

### Example — @scoutingapi/sdk

```ts
import { ScoutingApiClient } from '@scoutingapi/sdk';

const scout = new ScoutingApiClient({ apiKey: process.env.SCOUTINGAPI_KEY });

const { data } = await scout.priceCompare({
  name: 'Hotel X Sibenik',
  location: 'Sibenik, HR',
  checkIn: '2026-07-13',
  checkOut: '2026-07-20',
  currency: 'EUR',
});

const below = data.offers.filter((o) => o.totalPrice < data.median);
console.log(below.length, 'OTA(s) below median', data.median);
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

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/rate-parity-monitor
