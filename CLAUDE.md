# Compass Error · MT TITANIA

Offline-capable PWA for ship's officers to compute gyro and magnetic compass errors from a bearing of a charted object (Singapore Strait beacons/lighthouses) and fill in the compass error book.

Live: https://sailor-90.github.io/compass-error/ (GitHub Pages, public repo).

## Sebelum mengubah kode

- Baca graphify-out/GRAPH_REPORT.md dulu untuk memahami struktur proyek. Folder itu hanya ada di komputer ini dan tidak ikut git; kalau tidak ada atau usang, jalankan /graphify dulu.
- Setelah perubahan yang menambah atau mengubah fungsi, fitur, atau file, dan hasilnya sudah dicek serta di-commit, jalankan /graphify . --update atau ingatkan user.

## Layout

Static site, no build step, no dependencies, no package.json.

- `index.html` — the whole app: CSS, markup, and one inline `<script>` (data, math, rendering, tab navigation, intro).
- `sw.js` — service worker. Network-first, cache fallback. Current cache name `compass-err-v4`.
- `manifest.webmanifest` — PWA manifest (standalone, theme `#0b6e8a`).
- `*.png` — app icons.

## Running

Serve the folder over HTTP (service workers need http/https, not `file://`):

```bash
python3 -m http.server 8000
```

Then open http://127.0.0.1:8000/index.html (checked: serves 200). There are no automated tests or linters. Verify changes in the browser.

Manual regression check with the default inputs: set "Obs. gyro bearing" to `305.0` → True bearing `305.4°` (Bedok Lighthouse, 1.31 NM), Gyro err `0.4° Low`, Comp. err `3.5°W`, Deviation `3.4°W`, Card dev `0.8°W`, and a CHECK pill for deviation vs card. Anything else means a calculation changed.

## UI structure

- **Intro splash** (`#intro`): short animation on open, tap to skip, auto-closes after 2.5 s. Hidden when the OS has "reduce motion" on.
- **Dashboard** (`<header class="hero">`): sticky at the top. Shows `#kpis` — True bearing (main tile, with distance and object name), Gyro err, Comp. err, Deviation, Card dev. Visible only on the Input tab on phones (<900px); always visible on wide screens (≥900px) where Input is a permanent left column.
- **Bottom tab bar**: Input, Checks, Book, Working. Sections carry `data-view="input|summary|book|working"` (the Checks tab is the `summary` view). `setView()` sets `body[data-v]`; the last tab is remembered in `localStorage` key `ce-view` (wrapped in try/catch).
- The green/amber/red dot on the Checks tab is a MutationObserver on `#checks`.
- The `[hidden]{display:none!important}` rule is required: `.fields` sets `display:grid`, which otherwise overrides the `hidden` attribute on `#customBox`.

## Core logic (`index.html`, `calc()`)

- `inv()` — Vincenty inverse on WGS84; gives true bearing and distance from ship to object.
- Gyro error = True − Obs gyro bearing (+ = Low, − = High).
- Obs magnetic bearing = Obs gyro + (Mag heading − Gyro heading).
- Compass error = True − Obs magnetic (+ = E).
- Variation = base + years since reference year × annual change (minutes/60). Defaults: 0.2°E, ref 2020, −2'/yr.
- Deviation = Compass error − Variation. Compared against `cardDev()`, a linear interpolation of the 8-point deviation card (`CARD_HDG`/`CARD_DEV`, card dated 18-09-26).
- Time input is local (UTC+8, hardcoded) or UTC; the book shows UTC date and GMT.
- `rebuildList()` sorts `OBJECTS` by distance and groups them as Recommended (1–6 NM), Too close (<1 NM) and Farther (6–12 NM). The `g` field (A/B/C) is a preference grade, with A the best.
- Output: dashboard KPIs, the Magnetic and Gyro book rows, checks and a working log. "Copy row" copies a tab-separated row for pasting into a spreadsheet.

## Conventions and gotchas

- **Bump `VERSION` in `sw.js` whenever any file changes**, otherwise installed phones keep serving the old cache. Add any new file to `FILES` too. After a push, phones need two opens to pick up the new version.
- Sign conventions are navigational: E/W for errors, Low/High for gyro. Keep the `ew()` and `hl()` formatters.
- Thresholds are domain rules: gyro error >1° → report; deviation vs card >2° → check, 1–2° → monitor; object <1 NM → caution. Change them only on explicit request.
- Leave inputs blank rather than guessing: missing values render as `—`. The steering compass is optional and must never be estimated.
- The deviation card values and variation defaults are ship-specific data and go stale. Update them when a new card is issued.
- Theming uses CSS variables with `prefers-color-scheme` dark mode and a `data-theme` override. Reuse the tokens instead of hardcoding colours.
- Keep it a single dependency-free file; no frameworks or build tooling.

## Git workflow

- Public repo `sailor-90/compass-error`. Claude commits locally with the neutral project identity (`compass-error` / `noreply@example.com`); the user pushes with GitHub Desktop. Never push, create a repo or add a remote without permission in that session.
- Never commit credentials, tokens, personal data or the user's real name/email.
- `graphify-out/` and `.DS_Store` are in `.gitignore`.
