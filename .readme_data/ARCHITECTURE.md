# `.readme_data/` – README & guide imagery

This directory centralizes every binary asset referenced from Markdown (the root `README.md`, `stream_guide.md`, marketing docs, etc.). Hosting them inside the repo allows GitHub to serve them via `https://raw.githubusercontent.com/...`, which keeps the README portable across forks.

## Key assets
| File | Used in | Notes |
|------|---------|-------|
| `new_tracker_guide_graphic.png` | Hero image near the top of `README.md` | Explains tracker panels at a glance. |
| `readme_1.png`, `readme_2.png` | Installation walkthrough in `README.md` | Screenshots that walk users through Windows Defender prompts. |
| `example_debug_screenshot.png` | README FAQ → troubleshooting template | Shows the tracker in debug mode for bug reports. |
| `stream_guide_1.png` – `stream_guide_5.png` | `stream_guide.md` | Step-by-step streaming guide with annotated screenshots. |
| `backend_error.png`, `debug.png`, `missing.png`, `incognito.png`, `present.png`, `themes.png` | Various README sections (error handling, FAQ, theming) | Keep filenames stable because they are hard-coded in Markdown links. |
| `aciton_log.jpg` (typo intentional) | Legacy README excerpt | Retained for history; rename in Markdown only if you also fix the filename. |
| `discord.JPG` | README badge linking to Discord | JPG is capitalized; GitHub URLs are case sensitive. |

## Adding or updating images
1. Export assets at a reasonable resolution (target ≤ 1600px on the longest edge) to keep the repo size manageable.
2. Drop the file into `.readme_data/` and reference it from Markdown using raw GitHub URLs, e.g.\
   `![Tracker Features](https://raw.githubusercontent.com/mtgatracker/mtgatracker/master/.readme_data/new_tracker_guide_graphic.png)`
3. Prefer descriptive snake_case filenames. If you must rename an existing asset, search for every Markdown reference (`rg '.readme_data/old_name' -g '*.md'`) to avoid broken images.
4. Commit binaries with Git LFS if they exceed project policies; otherwise Git handles them normally.

## Housekeeping tips
- Group related assets with consistent prefixes (`stream_guide_*.png`, `readme_*.png`) so contributors can understand intent at a glance.
- When removing documentation sections, also delete their unused images to keep the repository smaller.
- For assets shared between the README and the GitHub Pages site, keep them here and reference the same URL from both places; that ensures screenshots stay in sync across surfaces.
