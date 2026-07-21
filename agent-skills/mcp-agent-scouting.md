---
name: stayingapi-mcp-agent-scouting
description: Connect StayingAPI as a Claude/Cursor MCP connector and scout stays, compare prices and check availability conversationally — no keys pasted. Powered by the StayingAPI REST API / MCP server.
version: 1.0.0
homepage: https://stayingapi.com/workflows/mcp-agent-scouting
license: Proprietary — see https://stayingapi.com/terms
tools: [search, price-compare, availability, price, listing, reviews]
auth: bearer-api-key | oauth2-pkce
---

# Agent-native scouting via MCP

Give an AI agent first-class access to accommodation data without pasting an API key into the model. Connect StayingAPI’s native MCP server once (OAuth 2.1 + PKCE), and the agent gains seven read-only tools — search stays, check availability, get a listing, get a price, compare prices across OTAs, get reviews, and poll async jobs — that it can call in the flow of a conversation over one unified schema. It is the agent-native path the scraping-template catalogs cannot match.

- **Base URL:** `https://api.stayingapi.com/v1` (REST) · **MCP:** `https://mcp.stayingapi.com/mcp`
- **Auth:** `Authorization: Bearer stay_live_…` (free sandbox: `stay_test_…`).
- **Get a free key** (no card): https://stayingapi.com/signup · Full machine contract: https://api.stayingapi.com/openapi.json

## Steps

1. **Add StayingAPI as a custom MCP connector** — In Claude.ai (or Claude Desktop / Cursor) add a custom connector with the server URL https://mcp.stayingapi.com/mcp. No API key is pasted into the agent.
2. **Authorize once (OAuth 2.1 + PKCE)** — Complete the browser OAuth prompt — with dynamic client registration — to link the connector to your StayingAPI account and its credit balance. Usage is metered to that account.
3. **Scout in the conversation** — Ask the agent to find stays, compare a property across OTAs, or check availability. It calls search_stays, compare_prices, check_availability and the other read-only tools directly.
4. **Get answers in the flow of the chat** — The agent reads the unified-schema results and answers inline — the cheapest cross-OTA rate, bookable dates, or a ranked shortlist — no glue code, no keys in the prompt.

## When to use this

The runtime is an MCP-capable agent (Claude, Cursor, or anything speaking the Model Context
Protocol) and the user wants accommodation scouting inside the conversation — search, price
comparison, availability, reviews — without managing an API key.

## Connect

1. Add a custom connector with URL `https://mcp.stayingapi.com/mcp` (Claude.ai connector settings, or
   `claude_desktop_config.json` for Desktop/Cursor).
2. Complete the OAuth 2.1 + PKCE prompt in the browser (dynamic client registration — nothing to
   pre-register). The connector is linked to your account and metered there.

## The seven read-only tools

`search_stays`, `check_availability`, `get_listing`, `get_price`, `compare_prices`, `get_reviews`,
and `get_job`. They mirror the REST endpoints over one unified schema.

## Async contract

A live tool call that has to scrape returns a pending job; poll `get_job` with the returned
`jobId` until it completes, then read the payload from `data.result`. Sandbox and cache hits return
immediately.

## MCP vs REST

Prefer MCP for interactive agents (OAuth, tools surfaced to the model). Prefer the REST API +
`@stayingapi/sdk` for programmatic backends and scheduled jobs. Same data, same schema — see the
stayingapi-mcp and stayingapi-search skills.

## MCP (no key pasted into the agent)

On an MCP-capable runtime, connect the StayingAPI server at **https://mcp.stayingapi.com/mcp** (OAuth 2.1 + PKCE) and use:
- `search_stays` — search stays across platforms by location, dates, occupancy and filters.
- `check_availability` — day-by-day availability for a known listing over a date window.
- `get_listing` — full detail for one listing.
- `get_price` — a real price quote for one listing for specific dates.
- `compare_prices` — compare one property across OTAs with computed min + median.
- `get_reviews` — normalized, paginated reviews for one listing.
- `get_job` — poll an async scrape job to completion (free).

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

**Get your free key → https://stayingapi.com/signup** · Docs: https://stayingapi.com/docs · Workflow: https://stayingapi.com/workflows/mcp-agent-scouting
