# henneth-analytics-data

Durable data store for the henneth.app analytics dashboard. This repo holds **data only** — no app code, no dashboard HTML. It exists so the daily refresh task never has to re-pull 30 days of history from GA4/GSC/PostHog/Supabase; it only pulls yesterday, appends it here, and trims old rows.

## Layout

`store/` — one JSON file per day-level data series:

- `dailySeries.json` — GA4 totals per day (sessions, users, newUsers, pageviews, events, engagedSessions, engagementSec)
- `dailyPages.json` — GA4 pagePath breakdown per day
- `dailyChannels.json` — GA4 channel group per day
- `dailySrcMedium.json` — GA4 source/medium per day
- `dailyCountries.json` — GA4 country per day
- `dailyDevices.json` — GA4 device category per day
- `dailyBrowsers.json` — GA4 browser per day
- `dailyOS.json` — GA4 OS per day
- `dailyGSC.json` — Search Console clicks/impressions/ctr/position per day
- `dailyPHEvents.json` — PostHog event counts per day
- `dailyPHHost.json` — PostHog henneth.app vs desk.henneth.app split per day
- `_meta.json` — `{lastUpdated, dataMax, dataMin}` bookkeeping

`store/users.json` is intentionally **not stored here** — it contains real customer email addresses, and Wasay asked to keep PII out of any store beyond the dashboard file itself. The daily task re-pulls the Supabase user list fresh each run instead (cheap, single query).

## Retention

Each file keeps a rolling window (default 90 days). On every run: read the file, append yesterday's row(s), drop anything older than the window, dedupe by date (+label/path/event where applicable), write back.

## Who writes here

The "Henneth daily analytics report" scheduled task, via the connected GitHub account (Composio), once per day. Nothing else should write to this repo.
