<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from the workflow SSOT (content/workflows). Edit the config + re-run. -->

# StayingAPI Workflows

Ready-to-use accommodation & travel automations for your AI agent or favorite tool. Each turns availability, search & price data into something useful — alerts, reports, datasets. Use them as a **skill for your AI agent** ([`agent-skills/`](agent-skills/)) or import them into **n8n** ([`n8n/`](n8n/)).

**[Get a free key — no card →](https://stayingapi.com/signup)** · Rendered catalog: [https://stayingapi.com/workflows](https://stayingapi.com/workflows)

## The catalog

- **Multi-source price comparison → cheapest-rate report** — Compare one property across booking platforms and report the cheapest rate, with savings vs. the median.
- **Property availability watch → alert** — Watch a listing over a date window and get alerted the moment a wanted date becomes available.
- **Price-drop watcher** — Watch a specific stay and get alerted the moment its total price falls to or below your threshold.
- **Cross-OTA rate-parity monitor → parity-exception digest** — Watch how your own listing's rate shows across OTAs on a schedule and flag any platform underselling the market median.
- **STR market scan → Google Sheet dataset** — Scan a short-term-rental market — discover listings across platforms, price the top N for your dates, and append a normalized row-per-listing dataset to a Google Sheet.
- **New-inventory monitor → CRM lead feed** — Re-run a saved search on a schedule, diff it against the last run, and sync only the newly-listed properties into a CRM or Sheet as leads.
- **Review-intelligence digest → weekly reputation report** — Pull normalized reviews for a property on a schedule and deliver a weekly digest — average rating, recurring themes, and the latest complaints to act on.
- **Competitor rate & availability benchmark → report** — Benchmark your listing’s rate and availability against a defined comp-set and get a weekly position report.
- **Agent-native scouting via MCP** — Connect StayingAPI as a Claude/Cursor MCP connector and scout stays, compare prices and check availability conversationally — no keys pasted.
- **Booking-ready cheapest-link resolver** — Resolve a property + dates to the single cheapest bookable link (with price and OTA) as clean JSON — for a concierge bot, a "book now" button, or an internal tool.
- **Event / peak-date demand radar** — Watch supply and prices around an event or peak date-window in a city and get alerted when demand tightens.
- **Portfolio price & occupancy feed** — A daily digest across a saved set of listings — price, rating and recent reviews per listing in one report.

## Two ways to run each one

- **AI agent** — install the `SKILL.md` from [`agent-skills/`](agent-skills/); your agent is the brain, no separate AI key needed.
- **n8n** — import the JSON from [`n8n/`](n8n/); the transforms are plain JavaScript, so no LLM key is required.

_11 of the 12 workflows ship an n8n import; the MCP-native ones are agent/MCP-only._

## Credits

Number-free by design — costs live on the [pricing page](https://stayingapi.com/pricing). Failed/empty calls are free; sandbox calls cost 0.

---

## Related repositories

Part of the StayingAPI open resource set — one unified accommodation-data API across Airbnb, Booking.com, Vrbo and Google Hotels, with a REST API, a native MCP server, and importable workflows.

- [airbnb-api](https://github.com/stayingapi/airbnb-api)
- [booking-com-api](https://github.com/stayingapi/booking-com-api)
- [vrbo-api](https://github.com/stayingapi/vrbo-api)
- [google-hotels-api](https://github.com/stayingapi/google-hotels-api)
- [hotel-api](https://github.com/stayingapi/hotel-api)
- [travel-api](https://github.com/stayingapi/travel-api)
- [travel-skills](https://github.com/stayingapi/travel-skills)
- [hotel-mcp](https://github.com/stayingapi/hotel-mcp)
- [airbnb-skills](https://github.com/stayingapi/airbnb-skills)
- [booking-com-skills](https://github.com/stayingapi/booking-com-skills)
- [vrbo-skills](https://github.com/stayingapi/vrbo-skills)
- [google-hotels-skills](https://github.com/stayingapi/google-hotels-skills)

- Website: <https://stayingapi.com> · Docs: <https://stayingapi.com/docs> · Status: <https://status.stayingapi.com>

---

**Get your free key — no card → <https://stayingapi.com/signup>** · [Docs](https://stayingapi.com/docs) · [Pricing](https://stayingapi.com/pricing) · [Status](https://status.stayingapi.com)

Built with [StayingAPI](https://stayingapi.com) — trusted, ToS-clean, real-time accommodation data across Airbnb, Booking.com, Vrbo and Google Hotels, normalized to one schema. REST + a native MCP server.
