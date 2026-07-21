---
name: stayingapi-event-radar
description: Watch supply and prices around an event or peak date-window in a city and get alerted when demand tightens. Powered by the StayingAPI REST API / MCP server.
version: 1.0.0
homepage: https://stayingapi.com/workflows/event-radar
license: Proprietary — see https://stayingapi.com/terms
tools: [search, availability, price-compare]
auth: bearer-api-key | oauth2-pkce
---

# Event / peak-date demand radar

Around a concert, conference or holiday weekend, accommodation supply tightens and prices climb — and the window to react is short. This radar scans the market for a city and date-window on a schedule (GET /v1/search), tracks the available-listing count and median price against the previous run, deep-checks a bellwether listing’s availability and cross-OTA price, and alerts when supply drops or prices rise beyond your thresholds — a pricing-opportunity signal for hosts and an early demand read for investors.

- **Base URL:** `https://api.stayingapi.com/v1` (REST) · **MCP:** `https://mcp.stayingapi.com/mcp`
- **Auth:** `Authorization: Bearer stay_live_…` (free sandbox: `stay_test_…`).
- **Get a free key** (no card): https://stayingapi.com/signup · Full machine contract: https://api.stayingapi.com/openapi.json

## Steps

1. **Set the city, the event window and thresholds** — Give the radar the city, the event check-in / check-out, the platforms to scan, and the supply-drop / price-rise percentages that should trigger an alert.
2. **Scan the market on a schedule** — Each run calls GET /v1/search for the city and dates across platforms, returning the currently-available supply and each listing’s price.
3. **Deep-check a bellwether listing** — It picks a representative listing and calls GET /v1/availability (how many dates are still open) and GET /v1/price-compare (its cross-OTA min / median) for a precise read on the window.
4. **Alert on tightening supply or rising prices** — A Code node diffs this run’s supply and median price against the previous run (stored in n8n static data) and posts an alert to Slack only when either crosses your threshold — swap for email or Telegram.

## When to use this

A host or investor wants an early warning when demand tightens around an event or peak window in a
city — "tell me when supply drops or prices climb for these dates in Split."

## How to do it

1. On a schedule, call `GET /v1/search` for the city + date window across platforms. The result’s
   length is your supply; each Property’s `price.totalPrice` gives the price distribution.
2. Track the available count and the median total across runs. When the count falls or the median
   rises past your thresholds, that is tightening demand.
3. For a concrete read, deep-check a bellwether listing with `GET /v1/availability` (open dates) and
   `GET /v1/price-compare` (cross-OTA min/median). Use `@stayingapi/sdk` or the REST endpoints.

## Reliability

A live search runs asynchronously — the SDK auto-polls; raw REST callers poll `GET /v1/jobs/{id}`.
Check `meta.partial`/`meta.warnings`; a blocked platform is charged 0. Current per-call credit costs
are on the pricing page.

### Example — REST

```bash
# Scan the market for the event window…
curl -sS "https://api.stayingapi.com/v1/search?location=Split,HR&checkIn=2026-07-13&checkOut=2026-07-20&platforms=airbnb,booking&limit=40" \
  -H "Authorization: Bearer $STAYINGAPI_KEY"
# …then compare a bellwether property's cross-OTA price
curl -sS "https://api.stayingapi.com/v1/price-compare?name=Hotel%20X%20Split&location=Split,HR&checkIn=2026-07-13&checkOut=2026-07-20" \
  -H "Authorization: Bearer $STAYINGAPI_KEY"
```

### Example — @stayingapi/sdk

```ts
import { StayingApiClient } from '@stayingapi/sdk';

const scout = new StayingApiClient({ apiKey: process.env.STAYINGAPI_KEY });

const { data } = await scout.search({
  location: 'Split, HR',
  checkIn: '2026-07-13',
  checkOut: '2026-07-20',
  platforms: ['airbnb', 'booking'],
  limit: 40,
});

const totals = data.map((p) => p.price?.totalPrice).filter((n) => typeof n === 'number').sort((a, b) => a - b);
const supply = data.length;
const median = totals[Math.floor((totals.length - 1) / 2)] ?? null;
console.log('supply', supply, 'median', median); // compare to your last run to spot tightening
```

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

**Get your free key → https://stayingapi.com/signup** · Docs: https://stayingapi.com/docs · Workflow: https://stayingapi.com/workflows/event-radar
