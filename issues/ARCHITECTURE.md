# `issues/` – Contributor-facing reporting playbook

This directory supplements the root `CONTRIBUTING.md` by giving players and contributors a single place to learn how to report bugs. The canonical content lives in `issues/README.md`; this file explains how to keep it useful.

## What `issues/README.md` covers
- **Before filing anything** – Directs users to search existing GitHub issues and “+1” them instead of opening duplicates.
- **Lightweight support channels** – Points to Twitter and the `devs.mtgatracker@gmail.com` inbox for simple questions.
- **Bug report template** – Lists required attachments:
  - `output_log.txt` (from `%APPDATA%/../LocalLow/Wizards of the Coast/MTGA`)
  - `mtga_watch.log`
  - A zipped copy of the settings directory (`~/.mtga_tracker`)
  - Screen capture or GIF covering at least a couple of game-state changes before/after the issue
- **Special case: `Unknown mtga_id` errors** – Users can skip the template and instead attach the exported deck plus a screenshot labeled `missing-cards`.
- **Feature requests** – Encourages GitHub issues, Discord outreach, or Beerpay “wishes”.
- **Open feature stub** – Mentions the (not yet implemented) “report an issue” button that would auto-upload log files; leave the strikethrough text until the feature exists.

## When to update the guide
- Logging paths change (e.g., Wizards moves `output_log.txt`), or MTGATracker adds automatic log bundling.
- A new high-volume error class appears (similar to `Unknown mtga_id`) that deserves its own reporting instructions.
- Support channels change (e.g., Discord channel names move).
- Legal/Privacy requirements evolve (e.g., you need to inform users about how attachments are stored).

## Checklist for editing
1. Keep the tone welcoming; most readers are non-technical players.
2. Verify every link (Discord, GitHub issues, Beerpay) still works.
3. When adding required attachments, justify *why* they are needed so users know the request is reasonable.
4. If you add new sections, update any references in the root `README.md` that point into this document.

By following this guidance you keep the support funnel efficient for both users and maintainers.
