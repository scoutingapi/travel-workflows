---
name: scoutingapi-price-drop-watcher
description: Watch a specific stay and get alerted the moment its total price falls to or below your threshold. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/price-drop-watcher
license: Proprietary — see https://scoutingapi.com/terms
tools: [price, price-compare]
auth: bearer-api-key | oauth2-pkce
---

# Price-drop watcher

Track a known listing for specific dates on a schedule and alert when its total price falls to or below a threshold you set. It calls GET /v1/price, compares the returned totalPrice to your priceBelow threshold, and — tracking the last-alerted price in static data — fires only on a NEW drop (first time below, or a further drop since the last alert) with the new price and a booking link. This is the automation twin of the App’s price-drop saved search.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com/mcp`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **Pick the stay and set a threshold** — Set the platform, listing id, check-in / check-out dates and occupancy, plus the total-price threshold you want to be alerted below.
2. **Re-price on a schedule** — Every few hours the workflow calls GET /v1/price for the listing and dates, returning a real numeric total for that stay.
3. **Compare to the threshold and de-dup** — A Code node checks the total against your threshold and against the last-alerted price (stored in n8n static data), so it fires only on a genuinely new drop — never twice for the same price.
4. **Send the price-drop alert** — When the price drops, a message with the new total, the threshold, the per-night rate and a booking link is posted to Slack — swap for email, Telegram or Discord.

## When to use this

The user wants to be told when a specific stay gets cheaper — "watch this listing and ping me if it
drops below X." Reach for `price` (one listing) or `price-compare` (a property’s cheapest cross-OTA
rate).

## How to do it

1. Take `platform` + `listingId` (or a `url`), the `checkIn`/`checkOut` dates and occupancy, plus a
   threshold total.
2. Call `GET /v1/price` (or the `@scoutingapi/sdk` `price()` method).
3. Compare `data.totalPrice` to the threshold. Track the last price you alerted on and fire only on a
   NEW drop (first time at/below, or a further drop) so you never notify twice for the same price.

## Cross-OTA variant

To watch a whole property’s cheapest rate, call `GET /v1/price-compare` and threshold on
`data.min` instead.

## Reliability

A no-price result throws a typed error and is free — treat it as "no reading this run", not a drop.
Honour retries with backoff on `upstream_unavailable`/`upstream_timeout`.

### Example — REST

```bash
curl -sS "https://api.scoutingapi.com/v1/price?platform=booking&listingId=abramovic2&checkIn=2026-07-13&checkOut=2026-07-20&adults=2&currency=EUR" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
```

### Example — @scoutingapi/sdk

```ts
import { ScoutingApiClient } from '@scoutingapi/sdk';

const scout = new ScoutingApiClient({ apiKey: process.env.SCOUTINGAPI_KEY });
const threshold = 2000; // alert at/below this total

const { data } = await scout.price({
  platform: 'booking',
  listingId: 'abramovic2',
  checkIn: '2026-07-13',
  checkOut: '2026-07-20',
  adults: 2,
  currency: 'EUR',
});

if (data.totalPrice <= threshold) {
  console.log('DROP:', data.totalPrice, data.currency, '→', data.url);
}
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

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/price-drop-watcher
