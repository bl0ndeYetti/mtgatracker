# `docs/` – Marketing site & data explorer

Everything under `docs/` is published via GitHub Pages (`mtgatracker.github.io/mtgatracker`). It covers both the main landing page and a lightweight “explore” microsite for visualizing anonymized match data.

## Repository layout
| Path | Role | Notes |
|------|------|-------|
| `index.html` | Main landing page | Pulls in `css/style.css`, `js/main.js`, and assets from `img/`. |
| `css/style.css` | Stylesheet bundle | Derived from Bootstrap-like template; includes animation classes used by WOW.js. |
| `js/main.js` | Front-end behavior | Handles sticky nav, smooth scrolling, mobile menu, and fetches live stats from Webtask (see below). |
| `img/`, `icon_large.png` | Assets | Hero images, favicons, CTA backgrounds referenced by `<img>` tags in `index.html`. |
| `explore/` | Standalone data explorer | Contains its own `index.html`, Plotly-powered JS (`js/graphs.js`), sample data (`data/anonymized_set_nodecks.json`), and helper scripts to anonymize raw match exports. |
| `explore/script/` | Data preprocessing utilities | `anonymizer.py` and `transformer.py` expect MTGATracker export files and emit the sanitized JSON consumed by the explorer. |
| `old/` | Archived versions of the marketing site | Retained for reference; not linked from the current landing page. |
| `sitemap` | Static sitemap used by search engines | Update this when you add/remove top-level pages. |

## Live metrics and external dependencies
- `js/main.js` performs three `$.get` calls against Auth0 Webtask endpoints to display:
  - Total tracked games (`.../games/count`)
  - Unique user count (`.../users/count`)
  - Latest release metadata plus download totals (`.../gh-stat-cache`)
- Those endpoints live in the private `webtasks` repository; if you change their URLs or payload shapes, update the selectors in `main.js` accordingly.
- `explore/js/graphs.js` uses Plotly (`explore/js/vendor/plotly-latest.min.js`). Keep vendor updates isolated in that folder so GitHub Pages doesn’t try to fingerprint third-party scripts.

## Local development
1. Serve `docs/` with any static server (e.g., `python3 -m http.server` from the repo root) and navigate to `/docs/index.html`.
2. The explorer can be opened directly via `docs/explore/index.html`. The default dataset is already bundled; to update it, run:
   ```bash
   cd docs/explore
   pip install -r requirements.txt
   python script/anonymizer.py path/to/raw.json
   python script/transformer.py anonymized.json data/anonymized_set_nodecks.json
   ```
3. Commit the resulting JSON so GitHub Pages serves the new charts.

## Styling & customization tips
- The hero section relies on CSS classes defined in `css/style.css` (`#intro`, `.back-to-top`, etc.). When adding sections, reuse these classes to stay consistent with the animation triggers in `main.js`.
- All imagery lives in `.readme_data/` or `docs/img/`. Prefer referencing `.readme_data` when you need the same image the GitHub README uses; otherwise drop site-specific assets into `docs/img/`.
- The `old/` directory is intentionally disconnected; feel free to remove it once you are confident the historical content is no longer needed.

## Testing checklist before publishing
- Verify the Webtask calls succeed (their responses should populate `#tracked-game-count`, `#unique-user-count`, and `#latest-release` without console errors).
- Load `docs/explore/index.html` and confirm Plotly renders graphs using the bundled anonymized dataset.
- Run `npx htmlhint docs/index.html` (or your HTML linter of choice) to catch stray markup before pushing to GitHub Pages.
