# MTGATracker (Non-Electron) Architecture

This document describes how everything in this repository fits together **except the Electron front end**, per the user request. It is meant to be an orientation layer that points you toward the deeper, per-directory breakdowns added alongside it.

## Runtime data flow (TL;DR)
1. `app/mtgatracker_backend.py` is the CLI entry point that accepts log-watching flags, spins up an asyncio websocket server (default `127.0.0.1:8089`), and boots worker threads.
2. A `KillableTailer` defined in `util.py` follows `output_log.txt` (or a provided log) and pushes raw log “blocks” into `app/queues.block_read_queue`.
3. `app/tasks.block_watch_task` normalizes those blocks, extracts JSON payloads, and forwards structured blobs to `json_blob_queue`.
4. `app/tasks.json_blob_reader_task` deduplicates blobs, keeps `mtga_app.mtga_watch_app` in sync, and publishes game/deck/collection changes to dedicated multiprocessing queues (e.g., `game_state_change_queue`, `decklist_change_queue`, `general_output_queue`).
5. `app/dispatchers.py` inspects each blob to determine which parser routine in `app/parsers.py` should run (game state messages, deck updates, inventory events, direct challenges, etc.).
6. Parser routines manipulate the domain model under `app/models/`, update the running `Game`/`Match` objects, persist deck collections via `mtga_watch_app`, and emit derived insights (draft pick analysis, chess timers, draw odds).
7. The websocket handler in `mtgatracker_backend.py` multiplexes coroutine tasks that stream `game_state`, `decklist_change`, and `general_output` messages to any connected UI (typically the Electron client).
8. Ancillary tooling—root-level scripts, docs, legal copy, and static assets—support development, release pipeline, and community operations.

```
┌──────────┐   blocks   ┌────────┐   blobs   ┌────────────┐    events    ┌──────────┐
│ Tailer   │───────────►│ tasks  │──────────►│ dispatchers│─────────────►│ parsers  │
└──────────┘            │ block/ │           │ + queues   │              └────┬─────┘
                        │ json   │           └────────────┘                   │
                        └────────┘                                            ▼
                                                    ┌──────────────────────────────────┐
                                                    │ mtga_watch_app (state + models) │
                                                    └──────────────┬───────────────────┘
                                                                   │
                                                            ┌──────▼──────┐
                                                            │ websocket  │
                                                            │  server    │
                                                            └──────┬──────┘
                                                                   │
                                                          UI / automation clients
```

## Directory map
| Directory | Purpose | Key entry points | Deeper documentation |
|-----------|---------|------------------|----------------------|
| `app/` | Python log-watching backend, domain models, parsers, queues | `mtgatracker_backend.py`, `tasks.py`, `parsers.py`, `models/` | `app/ARCHITECTURE.md` |
| `docs/` | Static marketing/docs site served via GitHub Pages | `index.html`, `js/main.js`, `explore/` content | `docs/ARCHITECTURE.md` |
| `.readme_data/` | Image assets embedded throughout the main README and guides | Image files referenced via raw GitHub URLs | `.readme_data/ARCHITECTURE.md` |
| `issues/` | Contributor-facing instructions for reporting bugs and feature requests | `issues/README.md` | `issues/ARCHITECTURE.md` |
| `legal/` | Privacy policy & terms of service referenced from installers and docs | `privacy.md`, `tos.md` | `legal/ARCHITECTURE.md` |
| `scripts/` | One-off or historical maintenance scripts (card ID generation, log splitting) | `generate_base_mtga_id_lookup.py`, `log_splitter.py` | `scripts/ARCHITECTURE.md` |
| `webtasks/` | Placeholder for the serverless endpoints consumed by `docs/js/main.js` | Public README explaining move to private repo | `webtasks/ARCHITECTURE.md` |
| `electron/` | Desktop UI (explicitly out of scope for this documentation pass) | `electron/main.js`, renderer bundles | `electron/ARCHITECTURE.md` (scope caveat) |

## Root-level supporting files
- `requirements.txt` and `requirements-tests.txt` pin the Python dependencies used by `app/`. Install these into your virtual environment before running `mtgatracker_backend.py`.
- `build.sh`, `deploy.sh`, `appveyor.yml`, and `.travis.yml` capture historical CI/CD steps used to package the Electron app with the Python backend via PyInstaller.
- `util.py` is intentionally at the repo root because it is shared by both the backend and packaging scripts. Notable exports include `KillableTailer`, deck-processing helpers, rarity/color ranking utilities, and `resource_path()` for PyInstaller compatibility.
- `mtgatracker_backend.spec` defines the PyInstaller build manifest that glues the Python backend into the Electron bundle.
- Global documentation such as `README.md`, `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, and `stream_guide.md` remain the authoritative references for users and contributors; the per-directory docs you are reading layer on internal details needed for engineering work.

## Operational notes
- **Log storage expectations:** `mtgatracker_backend.py` assumes Windows-style `%APPDATA%/../LocalLow/Wizards Of The Coast/MTGA/output_log.txt` unless `--log_file` is provided. Running it on other platforms requires pointing to a captured log file.
- **Threading model:** The backend mixes `threading.Thread` workers (for log parsing and mouse hooks) with `asyncio` coroutines (for websocket IO) and `multiprocessing.Queue` instances (for inter-thread communication). Avoid blocking operations in parser functions to keep the websocket loop responsive.
- **Message contract:** Websocket messages are JSON dictionaries annotated with a `data_type` field (`game_state`, `decklist_change`, `message`, `error`). Downstream clients should be tolerant of missing keys because replay logs sometimes omit fields.
- **Persistence:** Deck lists and collection snapshots are serialized to `%APPDATA%/../LocalLow/MTGATracker/settings.json` via `mtga_watch_app.save_settings()`. Editing that file manually without recalculating hashes can desynchronize odds calculations.
- **External services:** The only active network calls outside the websocket listener are the optional HTTP requests in `mtga_app.mtga_watch_app` (for version hashing) and the marketing site’s calls to Auth0 Webtask endpoints (see `docs/js/main.js`).

## Scope exclusions
- Electron renderer/main-process behavior, build tooling (`npm`, webpack, Sass), and UI assets live under `electron/` and remain intentionally undocumented here other than a pointer.
- Commercial/legal artifacts kept in private repositories (notably the Auth0 webtasks referenced from the docs site) are described only at a high level because their implementation lives elsewhere.

Refer to the per-directory `ARCHITECTURE.md` files for drills-down into each area.
