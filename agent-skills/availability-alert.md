---
name: scoutingapi-availability-alert
description: Watch a listing over a date window and get alerted the moment a wanted date becomes available. Powered by the ScoutingAPI REST API / MCP server.
version: 1.0.0
homepage: https://scoutingapi.com/workflows/availability-alert
license: Proprietary — see https://scoutingapi.com/terms
tools: [availability, listing]
auth: bearer-api-key | oauth2-pkce
---

# Property availability watch → alert

Given a known listing (or a batch) and a date window, poll day-by-day availability on a schedule and alert the instant a wanted date becomes bookable. It calls GET /v1/availability with onlyAvailable=true, flattens the returned calendar, de-dups against the previous run so you are only pinged on a real change, and posts the newly-available dates. This is the automation twin of the App’s availability-alert saved search.

- **Base URL:** `https://api.scoutingapi.com/v1` (REST) · **MCP:** `https://mcp.scoutingapi.com`
- **Auth:** `Authorization: Bearer scout_live_…` (free sandbox: `scout_test_…`).
- **Get a free key** (no card): https://scoutingapi.com/signup · Full machine contract: https://api.scoutingapi.com/openapi.json

## Steps

1. **Pick the listing and the date window** — Set the platform, the listing id (or a full listing URL), and the start / end dates to watch (window ≤ 365 days, not in the past).
2. **Poll availability on a schedule** — Every few hours the workflow calls GET /v1/availability with onlyAvailable=true, returning the day-by-day calendar for the listing.
3. **Detect what newly became available** — A Code node flattens the calendar to the bookable dates and compares them to the previous run (stored in n8n static data), so you are alerted only when the available set changes — no duplicate pings.
4. **Send the availability alert** — When a wanted date opens up, a message listing the newly-available dates is posted to Slack — swap the last node for email, Telegram, a webhook or Discord.

## When to use this

The user wants to know when a specific stay (or one of a batch) becomes bookable over a date
window — e.g. "tell me when this Airbnb opens up for my week." Reach for `availability`.

## How to do it

1. Take `platform` + `listingId` (or a full listing `url`) and the `startDate`/`endDate` window
   (≤ 365 days, not in the past).
2. Call `GET /v1/availability` with `onlyAvailable=true` (or the `check_availability` MCP tool, or
   `@scoutingapi/sdk`).
3. Flatten `data[].dates[]` and keep the entries where `available && bookable`. Compare to what you
   saw last time and report only the NEWLY-available dates so you never notify twice.

## Scheduling

To watch continuously, re-run on a cadence (n8n Schedule Trigger, or your agent’s scheduler). Each
re-run is a real billed call — align the cadence to the 6-hour availability cache so you are not
paying for identical results.

## Partial failures

Check `meta.partial` and `meta.warnings[]`; a listing that returns no calendar is charged 0. Report
what you have.

### Example — REST

```bash
curl -sS "https://api.scoutingapi.com/v1/availability?platform=airbnb&listingId=42307961&startDate=2026-07-13&endDate=2026-07-20&onlyAvailable=true" \
  -H "Authorization: Bearer $SCOUTINGAPI_KEY"
```

### Example — @scoutingapi/sdk

```ts
import { ScoutingApiClient } from '@scoutingapi/sdk';

const scout = new ScoutingApiClient({ apiKey: process.env.SCOUTINGAPI_KEY });

const { data } = await scout.availability({
  platform: 'airbnb',
  listingId: '42307961',
  startDate: '2026-07-13',
  endDate: '2026-07-20',
  onlyAvailable: true,
});

const open = data.flatMap((l) => l.dates).filter((d) => d.bookable).map((d) => d.date);
console.log(open.length, 'bookable dates', open.join(', '));
```

## MCP (no key pasted into the agent)

On an MCP-capable runtime, connect the ScoutingAPI server at **https://mcp.scoutingapi.com** (OAuth 2.1 + PKCE) and use:
- `check_availability` — day-by-day availability for a known listing over a date window.

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

**Get your free key → https://scoutingapi.com/signup** · Docs: https://scoutingapi.com/docs · Workflow: https://scoutingapi.com/workflows/availability-alert
