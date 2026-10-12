# Compass Error · MT TITANIA

Offline-capable PWA for ship's officers to compute gyro and magnetic compass errors from a bearing of a charted object (Singapore Strait beacons/lighthouses) and fill in the compass error book.

## Layout

Static site, no build step, no dependencies, no package.json.

- `index.html` — the whole app: CSS, markup, and one inline `<script>` (data, math, rendering).
- `sw.js` — service worker. Network-first, cache fallback.
- `manifest.webmanifest` — PWA manifest (standalone, theme `#0b6e8a`).
- `*.png` — app icons.

## Running

Serve the folder over HTTP (service workers need http/https, not `file://`):

```bash
python3 -m http.server 8000
```

There are no tests or linters. Verify changes in the browser.

## Core logic (`index.html`, `calc()`)

- `inv()` — Vincenty inverse on WGS84; gives true bearing and distance from ship to object.
- Gyro error = True − Obs gyro bearing (+ = Low, − = High).
- Obs magnetic bearing = Obs gyro + (Mag heading − Gyro heading).
- Compass error = True − Obs magnetic (+ = E).
- Variation = base + years since reference year × annual change (minutes/60). Defaults: 0.2°E, ref 2020, −2'/yr.
- Deviation = Compass error − Variation. Compared against `cardDev()`, a linear interpolation of the 8-point deviation card (`CARD_HDG`/`CARD_DEV`, card dated 18-09-26).
- Time input is local (UTC+8, hardcoded) or UTC; the book shows UTC date and GMT.
- `rebuildList()` sorts `OBJECTS` by distance and groups them as Recommended (1–6 NM), Too close (<1 NM) and Farther (6–12 NM). The `g` field (A/B/C) is a preference grade, with A the best.
- Output is the Magnetic and Gyro book rows, KPIs, checks and a working log. "Copy row" copies a tab-separated row for pasting into a spreadsheet.

## Conventions and gotchas

- **Bump `VERSION` in `sw.js` whenever any file changes**, otherwise installed phones keep serving the old cache. Add any new file to `FILES` too.
- Sign conventions are navigational: E/W for errors, Low/High for gyro. Keep the `ew()` and `hl()` formatters.
- Thresholds are domain rules: gyro error >1° → report; deviation vs card >2° → check, 1–2° → monitor; object <1 NM → caution. Change them only on explicit request.
- Leave inputs blank rather than guessing: missing values render as `—`. The steering compass is optional and must never be estimated.
- The deviation card values and variation defaults are ship-specific data and go stale. Update them when a new card is issued.
- Theming uses CSS variables with `prefers-color-scheme` dark mode and a `data-theme` override. Reuse the tokens instead of hardcoding colours.
- Keep it a single dependency-free file; no frameworks or build tooling.
