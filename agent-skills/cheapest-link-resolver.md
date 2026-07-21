---
name: stayingapi-cheapest-link-resolver
description: Resolve a property + dates to the single cheapest bookable link (with price and OTA) as clean JSON — for a concierge bot, a "book now" button, or an internal tool. Powered by the StayingAPI REST API / MCP server.
version: 1.0.0
homepage: https://stayingapi.com/workflows/cheapest-link-resolver
license: Proprietary — see https://stayingapi.com/terms
tools: [price-compare]
auth: bearer-api-key | oauth2-pkce
---

# Booking-ready cheapest-link resolver

Sometimes you do not want a report — you want the answer: the one cheapest bookable link for this property on these dates, as a clean object a bot or a button can act on. This workflow calls the flagship price-compare endpoint, keeps only offers that carry a bookable URL, sorts by total price, and returns the single cheapest — OTA, price, currency, deep link, and the savings versus the median. It is the "resolve to a deal link" primitive behind a concierge assistant, a checkout CTA, or an internal booking tool.

- **Base URL:** `https://api.stayingapi.com/v1` (REST) · **MCP:** `https://mcp.stayingapi.com/mcp`
- **Auth:** `Authorization: Bearer stay_live_…` (free sandbox: `stay_test_…`).
- **Get a free key** (no card): https://stayingapi.com/signup · Full machine contract: https://api.stayingapi.com/openapi.json

## Steps

1. **Provide the property and dates** — Give the workflow a property name (plus an optional location to disambiguate), check-in / check-out, occupancy and a currency.
2. **Compare every OTA offer** — It calls GET /v1/price-compare, which resolves the property and returns every OTA offer for the dates, each with a bookable url, plus computed min and median.
3. **Resolve to the single cheapest link** — The transform keeps only offers that carry a bookable URL, sorts by total price, and returns the single cheapest as a clean object — OTA, price, deep link, and savings vs. median.
4. **Return / deliver the deal link** — The compact JSON is returned to a webhook (for a bot or a "book now" button) or posted to Slack/Telegram — a one-call primitive you can drop into any surface.

## When to use this

The user wants the **single cheapest bookable link** for a known stay — not a report — to hand to a
bot, a "book now" button, or an internal tool. Use the flagship `compare_prices` /
`GET /v1/price-compare` endpoint and resolve to one deal link.

## How to do it

1. Resolve the property with **one** of `name`, `googleHotelId`, or `location` (+ `checkIn`,
   `checkOut`, optional `adults`, `currency`).
2. Call `GET /v1/price-compare`.
3. Keep only `data.offers[]` that have a `url` **and** a numeric `totalPrice`; sort ascending; the
   first is your answer. Return `{ ota, totalPrice, currency, bookUrl, savingsVsMedian }`.
4. If no offer has a bookable url, return an explicit "no bookable offer" result — never a guess.

## Keep it a primitive

Return the compact object (or just the `bookUrl`) — no prose — so a caller can render a button or
follow the link programmatically. Surface `data.median` only as an optional "you save X" badge.

### Example — REST

```bash
curl -sS "https://api.stayingapi.com/v1/price-compare?name=Hotel%20X%20Sibenik&location=Sibenik,HR&checkIn=2026-07-13&checkOut=2026-07-20&currency=EUR" \
  -H "Authorization: Bearer $STAYINGAPI_KEY"
```

### Example — @stayingapi/sdk

```ts
import { StayingApiClient } from '@stayingapi/sdk';

const scout = new StayingApiClient({ apiKey: process.env.STAYINGAPI_KEY });

const { data } = await scout.priceCompare({
  name: 'Hotel X Sibenik',
  location: 'Sibenik, HR',
  checkIn: '2026-07-13',
  checkOut: '2026-07-20',
  currency: 'EUR',
});

const cheapest = data.offers
  .filter((o) => o.url)
  .sort((a, b) => a.totalPrice - b.totalPrice)[0];
console.log(cheapest.ota, cheapest.totalPrice, cheapest.url);
```

## MCP (no key pasted into the agent)

On an MCP-capable runtime, connect the StayingAPI server at **https://mcp.stayingapi.com/mcp** (OAuth 2.1 + PKCE) and use:
- `compare_prices` — compare one property across OTAs with computed min + median.

## Async & partial failures

A live call that has to scrape returns `202` + a `jobId`; poll `GET /v1/jobs/{jobId}` (free)
until `data.status` is `completed`, then read `data.result`. The `@stayingapi/sdk` auto-polls.
On a fan-out, check `meta.partial` and `meta.platformResults[]` — report what succeeded and note
any `meta.warnings[]`.

## Credit awareness

Costs are per-endpoint and metered by result (v3). **Failed, empty and blocked calls are never
billed**, and sandbox (`stay_test_`) calls are always free. The exact, current costs live only in
https://stayingapi.com/pricing and the machine-readable https://api.stayingapi.com/openapi.json — read them there, don't assume.

---

**Get your free key → https://stayingapi.com/signup** · Docs: https://stayingapi.com/docs · Workflow: https://stayingapi.com/workflows/cheapest-link-resolver
