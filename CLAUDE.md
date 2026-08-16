# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not a software project** — there is no build, lint, or test tooling. It is a data/log repository
that stores periodic Taiwan stock market breadth reports. Every commit in the git history is an automated
"Add market breadth report for `<date>` `<time>`" commit that adds one new report file. Reports are produced
by querying the **Taiwan-Stock MCP server** (`mcp__claude_ai_Taiwan-Stock__*` tools, e.g.
`get_realtime_market_breadth`, `get_realtime_top_movers`, `get_realtime_volume_surge`, `get_market_overview`,
`get_volume_surge_stocks`) and writing the result as a Markdown briefing.

## Repository structure

- `half_hourly/<YYYY-MM-DD>/<HHMM>.md` — one market-breadth report per run, named by the run's Taipei-time
  timestamp (24h `HHMM`, no colon). Reports are generated roughly every half hour to hour during TWSE trading
  hours (~09:00–13:30 Asia/Taipei).
- `half_hourly/hourly_report_<YYYY-MM-DD>_<HHMM>.md` — legacy naming used for the very first reports before
  the `half_hourly/<date>/<time>.md` convention was adopted; do not reuse this pattern for new reports.

## Producing a new report

1. Determine the current Taipei-time date and time; the target file is `half_hourly/<YYYY-MM-DD>/<HHMM>.md`
   (create the date directory if it doesn't exist).
2. Call the Taiwan-Stock MCP tools to gather market breadth data (rising/falling/unchanged counts, up/down
   ratio, top gainers/losers, volume anomalies). Prefer realtime tools (`get_realtime_*`) during trading
   hours; fall back to `daily_price`-backed tools (`get_market_overview`, etc.) when realtime data or the
   current trading day isn't available yet.
3. Write the report as Markdown following the structure used in existing reports (see below), then commit it
   with a message in the established style: `Add market breadth report for <YYYY-MM-DD> <HHMM>` — append a
   short parenthetical note when something is degraded or unavailable (e.g. connector errors, stale data),
   mirroring the wording used in recent commits (`git log --oneline` shows the exact convention).

## Report format conventions

Current reports are written in **Traditional Chinese** (early reports in this repo were in English; Chinese
is now the established convention — follow it for new reports unless told otherwise). A typical report
contains, in order:

- Title (`# 台股市場廣度簡報`)
- **產出時間** (generation time, Asia/Taipei) and **資料交易日** (trading date covered, with open/closed and
  intraday/close-of-day status)
- A results section — either the actual data tables (rising/falling/unchanged counts, up/down ratio, average
  % change, top 5 gainers/losers, volume anomalies) or, if a tool/data source failed, a clearly marked
  failure section explaining what broke and why
- When data is unavailable: list exactly which items could not be produced, any fallback/alternate data used
  (clearly labeled as not representing the current run), and a **建議** (recommendations) section

**If the Taiwan-Stock MCP connector is broken or degraded when you go to produce a report, say so plainly in
the report** (as prior reports do) rather than fabricating figures or silently omitting the report — this
repository's history is itself a record of the connector's `realtime_price` reliability over time, so
accuracy about failures matters as much as accuracy about data.

## Working in this repo

- There is no code to build, lint, or test. Do not add package.json/requirements.txt/CI config unless
  explicitly asked — this repo is intentionally just Markdown reports plus git history.
- When asked to analyze trends across reports (e.g. "how long has the connector been broken", "summarize
  this week's market breadth"), read the relevant `half_hourly/<date>/*.md` files directly and/or scan
  `git log` — the commit messages themselves already encode a timeline of connector health.
