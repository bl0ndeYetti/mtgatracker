# `electron/` – Explicitly out of scope

Per the original request these documentation updates cover *everything except the Electron front end*. The Electron sources remain untouched in this pass to avoid duplicating work or drifting from whatever internal docs you already maintain.

## What you need to know from outside Electron
- The desktop UI connects to the backend websocket exposed by `app/mtgatracker_backend.py` (default `127.0.0.1:8089`) and listens for the `game_state`, `decklist_change`, and `message/error` payloads described in `ARCHITECTURE.md`.
- When packaging the app you still bundle this directory with the Python backend via PyInstaller (see `mtgatracker_backend.spec` and `build.sh`).
- All UI-specific documentation, build scripts (`npm run start`, webpack configs, etc.), and assets should remain in whichever guide you already use for `electron/`. This file simply records that the omission is intentional.

If a future task requires Electron documentation, feel free to replace this placeholder with the appropriate content.
