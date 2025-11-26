# `scripts/` – Maintenance utilities

This folder holds one-off utilities that were useful during earlier MTGATracker releases. They are not part of the runtime product but remain handy when regenerating data or triaging logs.

## Inventory
| File | Purpose | Notes |
|------|---------|-------|
| `generate_base_mtga_id_lookup.py` | Bootstraps card metadata for new sets | Consumes `all_cards.json` exports, walks each set, and writes Python modules (e.g., `xln.py`) that define `Card` objects with MTGA IDs. Handles special layouts (aftermath, split cards) via `special_layouts`. Output lives under `app/set_data/` during generation runs. |
| `log_splitter.py` | Extracts individual games from a monolithic log | Reads a large MTGA log (`in_file = "5cs.txt"` by default), detects game boundaries by looking for `Event.JoinQueue` and `MatchCompleted` markers, and writes each game to `game_<n>.txt`. Useful for sharing minimal repro logs. |
| `readme.md` | Historical context | Explains that these scripts are legacy helpers and not required to run the app. |

## Running the scripts
1. Create/activate a Python 3 environment that has access to the same dependencies as the main backend (the scripts rely only on the standard library).
2. Adjust constants at the top of each file before running:
   - `generate_base_mtga_id_lookup.py` expects `all_cards.json`, plus optional per-set JSON dumps in `app/set_data/`. Update the `write_set()` calls at the bottom when targeting different sets.
   - `log_splitter.py` uses `in_file` and `out_fstr`; point those at your log source/destination.
3. Run the script directly with `python scripts/<name>.py`.

## When to use them
- **Onboarding a new MTGA set** – Export the set’s card JSON, tweak `generate_base_mtga_id_lookup.py`’s `write_set()` arguments, and check the generated module into version control.
- **Investigating a parsing bug** – Use `log_splitter.py` to isolate the relevant match and share `game_<n>.txt` with other contributors or tests.

Because these are ad-hoc tools, keep them documented here rather than in the main README to avoid confusing end-users.
