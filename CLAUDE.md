# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Wigo4it Tech Radar is a fork of the [Zalando Tech Radar](https://github.com/zalando/tech-radar). It is a **pure static site** — no build system, no package manager, no framework. Deployment is serving static files. The live site is at [techradar.wigo4it.nl](https://techradar.wigo4it.nl).

## Running locally

Because `index.html` uses `fetch("/versions/...")` with absolute paths, opening it directly in a browser won't work. Serve with any static HTTP server, for example:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

## Architecture

### Data flow

1. `index.html` fetches `versions/versions.json` to get the list of radar versions.
2. It resolves the active version from the `?version=` query param (or defaults to the newest).
3. It fetches `versions/<version>.json` for the entries of that version.
4. It calls `radar_visualization(config)` from `radar.js`, passing the entries and title.

### Entry schema (`versions/YYYY-MM-DD.json`)

Each entry in a version file has:

| field | values | meaning |
|---|---|---|
| `quadrant` | 0–3 | counter-clockwise from bottom-right |
| `ring` | 0–3 | 0 = innermost |
| `label` | string | display name |
| `link` | URL | documentation link |
| `active` | bool | visible on radar |
| `status` | 0/1/2 | 0 = unchanged (circle), 1 = new (square), 2 = moved (triangle) |

### Quadrants (fixed order in `radar.js`)

| index | name | position |
|---|---|---|
| 0 | Platform & Cloud Services | bottom-right |
| 1 | Talen, Frameworks & Tools | bottom-left |
| 2 | Fundamentele Ontwerpkeuzes | top-left |
| 3 | Engineering Practices | top-right |

### Rings (fixed order)

| index | name |
|---|---|
| 0 | Gebruik |
| 1 | Probeer |
| 2 | Onderzoek |
| 3 | Verminder |

### Publishing a new radar version

1. Create `versions/YYYY-MM-DD.json` with the entries.
2. Add an entry to `versions/versions.json` with `version`, `title` (e.g. `"2026.06"`), and `description`.
3. The changelog and version picker update automatically.

### Other pages

- **`changelog.html`** — dynamically renders all versions from `versions.json`.
- **`wiki.html`** — serves markdown files from `wiki/` using `marked.js`. URL: `wiki.html?page=<filename-without-extension>`. Use `wiki/!-template.md` as the template for new pages.

### Styling

All brand colors are CSS custom properties in `radar.css` under `:root`, using Dutch names (`--kleur-gebruik`, `--kleur-probeer`, etc.). `radar.js` reads these at runtime via `getComputedStyle`, so color changes only need to happen in `radar.css`.

### Blip positioning

`radar.js` places blips using a D3 v4 force simulation with a seeded RNG (`seed = 42`) for reproducible layout. Changing entry order or count will shift all blip positions.

### Analytics

`index.html` includes Azure Application Insights with Click Analytics. The instrumentation key is in the script block — this is intentional and not a secret (it's a client-side telemetry key).
