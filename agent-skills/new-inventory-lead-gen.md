---
name: stayingapi-new-inventory-lead-gen
description: Re-run a saved search on a schedule, diff it against the last run, and sync only the newly-listed properties into a CRM or Sheet as leads. Powered by the StayingAPI REST API / MCP server.
version: 1.0.0
homepage: https://stayingapi.com/workflows/new-inventory-lead-gen
license: Proprietary — see https://stayingapi.com/terms
tools: [search, availability]
auth: bearer-api-key | oauth2-pkce
---

# New-inventory monitor → CRM lead feed

Fresh inventory is a lead: a new short-term rental appearing in your target market is a management pitch, an acquisition target, or a comp to underwrite. This workflow re-runs your saved search on a schedule, keeps a memory of everything it has already seen, and emits only the listings that are new since the last run — one row per lead, with platform, name, URL, beds, rating and price — straight into your CRM or a Google Sheet. The first run quietly seeds the baseline so you are never flooded with the whole market.

- **Base URL:** `https://api.stayingapi.com/v1` (REST) · **MCP:** `https://mcp.stayingapi.com/mcp`
- **Auth:** `Authorization: Bearer stay_live_…` (free sandbox: `stay_test_…`).
- **Get a free key** (no card): https://stayingapi.com/signup · Full machine contract: https://api.stayingapi.com/openapi.json

## Steps

1. **Define the market to watch** — Give the workflow a location (and optional dates, platforms and result limit) — the same inputs you would use for a one-off search, saved as a recurring watch.
2. **Re-run the search on a schedule** — On each scheduled run it calls GET /v1/search across the requested platforms and merges every match into one normalized Property list.
3. **Diff against the last run** — The transform compares the result ids against the ids it saw last time (kept in n8n workflow static data) and keeps only the listings that are new — the first run seeds the baseline silently.
4. **Sync new leads to your CRM or Sheet** — Each new listing is emitted as its own row (platform, name, URL, beds, rating, price) and appended to a Google Sheet or pushed to your CRM — swap the delivery node for Slack to get a new-inventory ping instead.

## When to use this

The user wants to be told when **new inventory appears** in a market — a new short-term rental to
pitch, acquire, or benchmark — rather than running a one-off search. This is a scheduled,
stateful watch: search, then diff against what you saw last time.

## How to do it

1. Call `GET /v1/search` with the saved `location` (+ optional `checkIn`/`checkOut`,
   `platforms`, `limit`, `currency`).
2. Compare the returned `data[].id` values against the set of ids from the previous run (persist
   it — n8n static data, a Sheet column, or your CRM). The **new ids** are the leads.
3. Emit one record per new listing: `platform`, `name`, `url`, `location.city`, `bedrooms`,
   `maxOccupancy`, `guestRating`/`ratingScale`, `price.nightlyPrice`/`totalPrice`.
4. On the **first** run, seed the baseline and emit nothing — otherwise you flag the whole market.

## Newly-available variant

For "known listings that just opened up," keep a watchlist of listing ids and poll
`GET /v1/availability` over your window; a date flipping to `bookable: true` is also a lead.

## Partial results

Check `meta.partial` / `meta.platformResults[]` — attribute leads per platform and note any
platform that failed so a gap is not mistaken for "no new inventory."

### Example — REST

```bash
curl -sS "https://api.stayingapi.com/v1/search?location=Split,HR&platforms=airbnb,vrbo,booking&limit=30&currency=EUR" \
  -H "Authorization: Bearer $STAYINGAPI_KEY"
```

### Example — @stayingapi/sdk

```ts
import { StayingApiClient } from '@stayingapi/sdk';

const scout = new StayingApiClient({ apiKey: process.env.STAYINGAPI_KEY });

const { data, meta } = await scout.search({
  location: 'Split, HR',
  platforms: ['airbnb', 'vrbo', 'booking'],
  limit: 30,
  currency: 'EUR',
});

// Diff against your own store of previously-seen ids to get just the new leads.
const ids = data.map((p) => p.id);
console.log(data.length, 'listings', meta.creditsCharged, 'credits');
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

**Get your free key → https://stayingapi.com/signup** · Docs: https://stayingapi.com/docs · Workflow: https://stayingapi.com/workflows/new-inventory-lead-gen
