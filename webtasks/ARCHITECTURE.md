# `webtasks/` – Serverless API pointers

The actual Webtask.io source code lives in a private repository (per `webtasks/README.md`). This folder exists solely to remind contributors how the public repo consumes those functions.

## What the missing code does
| Endpoint (see `docs/js/main.js`) | Purpose | Notes |
|----------------------------------|---------|-------|
| `/mtga-tracker-game/games/count` | Returns `{game_count}` | Increments when the backend posts anonymized “game complete” pings. Powers the “Total games tracked” badge on the marketing site. |
| `/mtga-tracker-game/users/count` | Returns `{unique_user_count}` | Counts distinct tracker IDs seen by telemetry. Surfaces on the landing page hero stats. |
| `/mtga-tracker-game/gh-stat-cache` | Returns `{latestVersionString, totalDownloads, latestVersion}` | Cached GitHub Releases info so the landing page can display the latest version and download count without hitting GitHub rate limits. |

## Getting access
1. Join the MTGATracker Discord and request access in `#dev-corner` (see `webtasks/README.md`).
2. Once added to the private repo, clone it alongside this one so you can keep the API payloads in sync.
3. Deploy changes using the instructions in the private repo (usually `wt` CLI) and update environment secrets/keys as needed.

## Touchpoints in this repo
- `docs/js/main.js` parses the JSON responses listed above.
- Future telemetry (e.g., Inspector dashboards) should use the same naming conventions so the marketing site can switch endpoints without code churn.
- When payload shapes change, document the new schema here and update both the marketing site and any backend callers.

## If you cannot reach Webtask
- Temporary outages should degrade gracefully; the marketing site merely logs console warnings and leaves the counters blank.
- For long-term replacements (e.g., migrating to Netlify Functions), keep this file updated so contributors know where to look.
