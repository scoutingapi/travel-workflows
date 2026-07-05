<!-- generated: DO NOT EDIT BY HAND — produced by apps/web/scripts/export-github-repos.ts from the workflow SSOT. Edit the config + re-run. -->

# Agent skills

One portable `SKILL.md` per workflow — install into any agent runtime that reads skill files.

- [Multi-source price comparison → cheapest-rate report](cross-ota-price-comparison.md) — Compare one property across booking platforms and report the cheapest rate, with savings vs. the median.
- [Property availability watch → alert](availability-alert.md) — Watch a listing over a date window and get alerted the moment a wanted date becomes available.
- [Price-drop watcher](price-drop-watcher.md) — Watch a specific stay and get alerted the moment its total price falls to or below your threshold.
- [Cross-OTA rate-parity monitor → parity-exception digest](rate-parity-monitor.md) — Watch how your own listing's rate shows across OTAs on a schedule and flag any platform underselling the market median.
- [STR market scan → Google Sheet dataset](str-market-scan.md) — Scan a short-term-rental market — discover listings across platforms, price the top N for your dates, and append a normalized row-per-listing dataset to a Google Sheet.
- [New-inventory monitor → CRM lead feed](new-inventory-lead-gen.md) — Re-run a saved search on a schedule, diff it against the last run, and sync only the newly-listed properties into a CRM or Sheet as leads.
- [Review-intelligence digest → weekly reputation report](review-intelligence-digest.md) — Pull normalized reviews for a property on a schedule and deliver a weekly digest — average rating, recurring themes, and the latest complaints to act on.
- [Competitor rate & availability benchmark → report](comp-set-benchmark.md) — Benchmark your listing’s rate and availability against a defined comp-set and get a weekly position report.
- [Agent-native scouting via MCP](mcp-agent-scouting.md) — Connect ScoutingAPI as a Claude/Cursor MCP connector and scout stays, compare prices and check availability conversationally — no keys pasted.
- [Booking-ready cheapest-link resolver](cheapest-link-resolver.md) — Resolve a property + dates to the single cheapest bookable link (with price and OTA) as clean JSON — for a concierge bot, a "book now" button, or an internal tool.
- [Event / peak-date demand radar](event-radar.md) — Watch supply and prices around an event or peak date-window in a city and get alerted when demand tightens.
- [Portfolio price & occupancy feed](portfolio-feed.md) — A daily digest across a saved set of listings — price, rating and recent reviews per listing in one report.
