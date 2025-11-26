# `app/` – Python log-watching backend

The `app` package ingests MTG Arena logs, maintains live game/deck state, and streams structured updates to any websocket client (usually the Electron UI). This document explains how the pieces connect so you can extend parsers, add queues, or script the backend without touching the front end.

## Entrypoint: `mtgatracker_backend.py`
| Concern | Details |
|---------|---------|
| Invocation | `python -m app.mtgatracker_backend [--log_file path] [--no_follow] [--read_full_log] [--mouse_events] [--port 8089]` |
| Threads spun up | 1) `tasks.block_watch_task` (log-to-JSON pipeline) 2) `tasks.json_blob_reader_task` (blob dispatch/state updates) 3) optional mouse hook via `pynput` 4) asyncio event loop driving websocket server |
| Websocket contract | Each loop iteration `asyncio.wait`s on `stats()`, `decks()`, `output()`, and `consumer_handler()`. Messages are JSON dicts tagged with `data_type` so the UI can route them. |
| Shutdown | Any client sending `"die"` triggers `all_die_queue.put("DIE")`, which short-circuits the outer loop, cancels pending coroutines, drains queues, and joins worker threads. |
| Log discovery | Defaults to `%APPDATA%/../LocalLow/Wizards Of The Coast/MTGA/output_log.txt` on Windows if `--log_file` is omitted. |

## Queue topology (`queues.py`)
All inter-thread communication relies on `multiprocessing.Queue` to avoid GIL contention. The canonical queues are:

| Queue | Producers | Consumers | Payload |
|-------|-----------|-----------|---------|
| `block_read_queue` | `KillableTailer` in `mtgatracker_backend.py` | `tasks.block_watch_task` | Raw text blocks exactly as they appear in `output_log.txt`. |
| `json_blob_queue` | `tasks.block_watch_task` | `tasks.json_blob_reader_task`, `mtgatracker_backend.json_watch_process` | Dicts enriched with metadata (block title, timestamps, sequence numbers). |
| `game_state_change_queue` | `tasks.json_blob_reader_task` | Websocket `stats()` coroutine | Serializable snapshots returned by `Game.game_state()`. |
| `decklist_change_queue` | `tasks.json_blob_reader_task`, `mtga_app.mtga_watch_app` | Websocket `decks()` coroutine | `{deck_id: Deck.to_serializable(transform_to_counted=True)}` style payloads. |
| `general_output_queue` | Parsers, MTGA app helpers, mouse events | Websocket `output()` coroutine | Errors, info messages, draft pick reports, mouse clicks, etc. |
| `all_die_queue` | Websocket consumers | Everyone | Acts as a kill switch for long-running loops. |

Tip: When you add a new producer, prefer reusing these queues; adding a brand-new queue means updating both worker bootstrap and websocket fan-out.

## State orchestration (`mtga_app.py`)
`MTGAWatchApplication` is a singleton (`mtga_app.mtga_watch_app`) that:
- Holds shared locks (`game_lock`) protecting `Game`/`Match` transitions.
- Persists decks, player IDs, and collection info to `%APPDATA%/../LocalLow/MTGATracker/settings.json`. On startup, `load_settings()` seeds `decklist_change_queue` so the UI paints the last-known state immediately.
- Emits human-readable errors/messages via `general_output_queue`.
- Provides long-lived structures such as `player_decks`, `draft_history`, and `collection`, all of which parsers mutate under lock.

If you need additional persistent state, model it here so every parser can reference it.

## Parsing pipeline (`tasks.py`, `dispatchers.py`, `parsers.py`)
1. **`tasks.block_watch_task`** – Converts heterogeneous log formats into valid JSON. It accounts for historical quirks (`UnityCrossThreadLogger` headers, list blocks, deprecated brackets) and annotates each blob with `block_title`, `request_or_response`, `timestamp`, etc.
2. **`tasks.json_blob_reader_task`** – Dedupe repeated blobs, detect decklist/collection changes (triggering `decklist_change_queue` publishes), and call `dispatchers.dispatch_blob(blob)`. It also compares hashes of hero libraries/opponent hands before and after dispatch to decide whether to push a new `game_state_change_queue` update.
3. **`dispatchers.py`** – Lightweight router that sends blobs to the appropriate parser based on top-level keys:
   - `greToClientEvent` → `parsers.parse_game_state_message` and friends.
   - `block_title == "Deck.GetDeckListsV3"` → `parsers.parse_get_decklists`.
   - `block_title == "Draft.DraftStatus"` → `parsers.parse_draft_status`.
   - JSON-RPC `method` values (e.g., `DirectGame.Challenge`) → deck/queue parsing helpers.
4. **`parsers.py`** – Heavy-lifting functions that mutate `Game`, `Player`, `Zone`, and `Deck` objects, compute chess timers, annotate event logs, and emit insights (draft recommendations, mulligan summaries, target specs).

### Adding a new parser
1. Figure out how the log chunk is surfaced (new `block_title`, JSON-RPC method, or nested inside `greToClientEvent`).
2. Extend `dispatchers.dispatch_blob` (or one of its helper dispatchers) to branch on that signal.
3. Implement the parser in `parsers.py` (or a dedicated module) and ensure it operates under `mtga_watch_app.game_lock` if it touches mutable game state.
4. Decide whether downstream consumers need to know about it (e.g., publish to `general_output_queue` or create a new queue).

## Domain model (`models/`)
- `models/game.py` – Defines `Player`, `Game`, and `Match`, plus helpers for calculating draw odds, serializing outcomes, and tracking annotations (target specs, chess timers, mulligans). These classes are the canonical shape of data sent to the UI.
- `models/card.py` – Declares `Card`, `GameCard`, and `Ability`. Game cards track both MTGA `grpid` and in-match `iid` so parsers can follow object ID swaps mid-match.
- `models/set.py` – Provides `Deck`, `Zone`, and `Library` abstractions used by both constructed decks and in-flight game zones.

When processing decklists (`util.process_deck`), cards are materialized into `Deck` objects that embed `mtga.set_data` card definitions for quick lookups.

## Other notable modules
- `dispatchers.py` wraps parser invocations with `@util.debug_log_trace` to aid in debugging; use the decorator for new entry points to get consistent logging.
- `parsers.parse_draft_status` leans on helper ranking functions in `util.py` (`rank_rarity`, `rank_colors`, `rank_cost`). This is the place to tweak draft heuristics.
- `conftest.py` bootstraps pytest fixtures (currently minimal).
- `models/__init__.py` exposes convenience imports for external callers.

## Secrets and external calls
`mtga_app.py` and `models/game.py` attempt to import `app._secrets` for `API_URL` and `hash_json_object`; when missing, they fall back to `secrets_template.py`. Keep sensitive keys out of source by copying `secrets_template.py` to `_secrets.py` locally.

`mtga_watch_app` uses `requests` to talk to remote services only when sending telemetry or resolving hashed versions. If you are packaging the backend separately, ensure `requests` is bundled.

## Troubleshooting checklist
- Check `mtga_watch.log` (rotating handler configured in `mtga_app.py`) for stack traces tagged with `util.debug_log_trace`.
- If `general_output_queue` is silent, verify that `tasks.json_blob_reader_task` is still pulling from `json_blob_queue`; stale `all_die_queue` entries will short-circuit the loop.
- When the UI stops updating decks, confirm `Deck.to_serializable(transform_to_counted=True)` is returning non-empty data; this usually indicates `mtga_watch_app.player_decks` never got seeded because `Deck.GetDeckListsV3` did not fire.

Use this file alongside the root-level `ARCHITECTURE.md` whenever you need to reason about backend-only changes.
