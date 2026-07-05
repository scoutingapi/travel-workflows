---
name: scoutingapi-cross-ota-price-comparison
description: Compare one property across booking platforms and report the cheapest rate, with savings vs. the median. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/cross-ota-price-comparison
license: Proprietary — see https://scoutingapi.com/terms
tools: [price-compare]
auth: bearer-api-key | oauth2-pkce
---

# Multi-source price comparison → cheapest-rate report

Given a property (or a search) plus dates and occupancy, fetch prices across booking platforms and produce a cheapest-rate report — the trusted, ToS-clean replacement for the fragile "hotel price comparison via scraping" templates. It calls the flagship cross-OTA endpoint, sorts every offer, picks the cheapest, and computes the savings against the ScoutingAPI-computed median.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **Provide the property and dates** — Give the workflow a property name (plus an optional location to disambiguate), check-in / check-out dates, occupancy and a currency.
2. **Call the flagship price-compare endpoint** — The workflow calls GET /v1/price-compare, which resolves one property and returns every OTA offer for the dates — plus ScoutingAPI-computed min and median aggregates.
3. **Pick the cheapest and compute savings** — It sorts the offers by total price, selects the cheapest, and computes the savings versus the median so you see both the best deal and how good it is.
4. **Deliver the cheapest-rate report** — A formatted report (cheapest OTA, price, savings, the full offer list and the credits charged) is posted to Slack — swap the last node for email, Telegram, Discord or a Google Sheet.

## When to use this

The user wants the cheapest price for a known stay, or a comparison of OTA offers for one
property. Reach for the flagship `price-compare` endpoint — it resolves one property and returns
every OTA offer for the dates, plus ScoutingAPI-computed `min` and `median`.

## How to do it

1. Resolve the property with **one** of `name`, `googleHotelId`, or `location` (+ `checkIn`,
   `checkOut`, optional `adults`/`children`/`childAges`, `currency`).
2. Call `GET /v1/price-compare`.
3. Sort `data.offers[]` by `totalPrice` ascending; the first is the cheapest. Report it with its
   `url` and the savings vs. `data.median` (`data.median - cheapest.totalPrice`).

## Alternative: search fan-out

For "cheapest across a *set* of listings in an area," call `GET /v1/search` to discover
candidates, then `GET /v1/price` per top candidate, and compare the quotes.

## Partial failures

Check `meta.partial` and `meta.platformResults[]` — a platform may fail while others succeed;
report what you have and note any `meta.warnings[]`.

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

const cheapest = [...data.offers].sort((a, b) => a.totalPrice - b.totalPrice)[0];
console.log('cheapest', cheapest.ota, cheapest.totalPrice, '· saves', data.median - cheapest.totalPrice);
```

## MCP (no key pasted into the agent)

On an MCP-capable runtime, connect the ScoutingAPI server at **https://mcp.scoutingapi.com** (OAuth 2.1 + PKCE) and use:
- `compare_prices` — compare one property across OTAs with computed min + median.

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

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/cross-ota-price-comparison
