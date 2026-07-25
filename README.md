# henneth-analytics-data

Durable data store for the henneth.app analytics dashboard. This repo holds **data only** — no app code, and deliberately no dashboard HTML shell either (see "Where the dashboard HTML lives" below). It exists so the daily refresh task never has to re-pull 30 days of history from GA4/GSC/PostHog/Supabase; it only pulls yesterday, merges it in here, and trims old rows.

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
- `users.json` — current Supabase user snapshot (see PII note below)
- `_meta.json` — `{lastUpdated, dataMax, dataMin}` bookkeeping

## users.json contains real customer emails — read before touching

Stored here intentionally (Wasay's explicit call, 2026-07-25). It is a **full overwrite every run**, not an append — Supabase itself is the history source of truth; this file is just a current-state cache for the dashboard splice step, and git commit history gives a free daily point-in-time backup of it. Never paste its contents anywhere except this file and a file delivered directly to Wasay (no Slack, no chat text, no issues).

## Where the dashboard HTML lives (it's NOT here)

The 90KB+ shell (CSS/markup/render code) is deliberately **not** duplicated into this repo. Its durable copy is the Cowork artifact `henneth-analytics-desk` — fetch it in a live session via `device_stage_files(artifact_ids:["henneth-analytics-desk"])`. This repo stays data-only so there is exactly one canonical copy of the shell (the artifact) and exactly one canonical copy of the data (here) — never two copies that can drift out of sync.

## Retention

Day-level files keep a rolling 90-day window. On every run: read the file, merge in yesterday's row(s) deduped by natural key (date+path, date+label, date+event, etc — replace, never duplicate), drop anything older than the window, write back.

## Who writes here

The "Henneth Daily Analytics Report" scheduled task, via the connected GitHub account (Composio), once per day — data only, never the dashboard HTML or the Cowork artifact (that requires a live, desktop-bridged session and cannot run headless). Nothing else should write to this repo.
