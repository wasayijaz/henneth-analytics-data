# Daily append recipe

Run this every day instead of re-pulling history.

## 1. Pull ONLY yesterday (Asia/Karachi) from each source

- GA4 (`GOOGLE_ANALYTICS_RUN_REPORT`, property `properties/546231582`): dateRanges `[{startDate:'yesterday',endDate:'yesterday'}]`, run once per dimension group listed in README (date+pagePath, date+channel, date+sourceMedium, date+country, date+device, date+browser, date+OS), plus the plain `date` totals row.
- GSC (`GOOGLE_SEARCH_CONSOLE_SEARCH_ANALYTICS_QUERY`, site `sc-domain:henneth.app`): dimensions `['date']`, start=end=2 days ago (GSC lags 2–3 days — don't expect yesterday to have data yet).
- PostHog (`mcp__PostHog__exec`): same two queries as before but `WHERE toDate(timestamp) = yesterday()` instead of `INTERVAL 30 DAY`.
- Supabase: re-run the full snapshot query (cheap, no history needed) to get current totals + last 24h signups. Do NOT persist this to GitHub — email PII stays out of the repo.

## 2. Read current files from GitHub

`GITHUB_GET_REPOSITORY_CONTENT` for each `store/*.json` path in `wasayijaz/henneth-analytics-data`, ref `main`. Decode base64 to get the current array.

## 3. Merge

For each file: append yesterday's new rows, then dedupe by the row's natural key (date+path, date+label, date+event, etc — last value wins if a date reappears), then drop any row older than 90 days from today, then sort by date descending.

## 4. Write back

`GITHUB_CREATE_OR_UPDATE_FILE_CONTENTS` for each changed file, same path, with the `sha` from step 2 (the tool auto-fetches it if omitted, but passing it avoids a 409 on concurrent runs). One commit per file is fine — this runs once a day, no concurrency to worry about.

Update `store/_meta.json` with the new `dataMax` / `lastUpdated`.

## 5. Recompute rollups and splice into the dashboard

Read the 90-day (or however much exists) arrays back out, recompute `kpis`, `pagesRanked`, `countries`, etc. the same way the existing dashboard build does, splice into the HTML template kept in the live chat session, verify with Playwright (zero JS errors, all tabs), then deliver.

If any source fails: keep that file's previous content untouched and say so in the report. Never write partial/fabricated rows.
