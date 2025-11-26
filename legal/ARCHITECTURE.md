# `legal/` – Policies shipped with MTGATracker

Legal copy lives alongside the source code so every distribution (GitHub releases, installer bundles, mtgatracker.com) can reference the same authoritative text. These files are linked throughout `README.md`, the installer UI, and the marketing site.

## Files
| File | Audience | Highlights |
|------|----------|------------|
| `privacy.md` | All end-users | Defines what data MTGATracker collects (game telemetry, anonymized hashes), how long it is retained (~3 months), what constitutes PII vs “linkable information,” and assurances around never selling user data. Includes sections on GDPR legal basis, data categories, and historical (deprecated) behaviors. |
| `tos.md` | All end-users, especially parents/guardians | Defines the binding contract for using MTGATracker/Inspector, outlines what MTGATracker provides, what users provide, restrictions/termination rules, minor consent requirements, contact info, and reiterates the MIT software license. |

## Maintenance checklist
1. **Keep links valid** – Both files reference GitHub commit history, parental consent forms, and email addresses; ensure those stay accurate whenever you reorganize.
2. **Version bumps** – Update the “Last updated with version X” line in `privacy.md` (currently `5.0.0`) when releasing major changes to data handling.
3. **Consistent terminology** – Terms introduced in `privacy.md` (PII, Linkable Information, Platform Information) should match the nouns used in UI strings and telemetry prompts.
4. **Cross-file dependencies** – `tos.md` explicitly points readers to `privacy.md`; if you add new rights/obligations in one file, make sure the other references the change if necessary.
5. **Distribution** – When packaging the app, bundle both files verbatim. Avoid copying text into installer dialogs; link or embed the markdown so you maintain a single source of truth.

## Adding new policies
If future features (e.g., cloud sync, telemetry dashboards) require new legal text, create a separate markdown file in this directory and link it from both existing documents and the UI. Keeping everything under `legal/` makes it easier for translators, lawyers, and auditors to diff changes over time.
