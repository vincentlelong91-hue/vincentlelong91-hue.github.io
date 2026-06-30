# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

AGGÓ is a mobile sports app prototype — a single self-contained HTML file that simulates an iOS app experience in the browser. No build system, no dependencies to install. Open `index.html` directly in a browser to run it.

The file is named `index.html` so it can be served as a GitHub Pages site at `https://vincentlelong91-hue.github.io` (repo: `vincentlelong91-hue/vincentlelong91-hue.github.io`).

## Architecture

Everything lives in one file: `index.html`. It has three major sections:

1. **`<head>` / `<style>`** — All CSS. Fonts (DM Sans via Google Fonts, Groonex embedded as base64 TTF), CSS variables, and shared component classes.
2. **`<body>`** — A `.scene` div (393×852px iPhone frame) containing all screens as sibling `div.screen` elements, each with `id="s{N}"`. Only one screen is visible at a time.
3. **`<script>`** (inline, at bottom) — All JavaScript: navigation engine, data, map, search, filters, etc.

### Screens

| ID | Screen | Notes |
|----|--------|-------|
| s0 | Splash | Entry point; "Commencer" → s8, "Se connecter" → s13 |
| s1 | Carte | Leaflet map with draggable bottom sheet + spot list |
| s2 | Sessions list | Tab 2; session cards with multi-filter |
| s3 | Défis | Tab 3; challenges + rank banner |
| s4 | Profil | Tab 4; stats, badges, sports played |
| s5 | Challenge Entreprise | Corporate leaderboard detail |
| s6 | Session detail | Populated dynamically by `openSession(id)` |
| s7 | Défi detail | Challenge detail |
| s8–s12 | Onboarding 1–5 | Flow: s8 → s9 → s10 → s11 → s12 → s1 |
| s13 | Connexion | Login; `loginCheck()` validates non-empty then `go(1)` |
| s14 | Modifier profil | Editable form; `saveProfile()` shows toast then `back()` |
| s15 | Créer une session | Form; `createSession()` shows toast then `goTab(2)` |
| s16 | Fiche Spot | Spot detail; populated by `openSpot(i)` |
| s17 | Amis | Friends list + social leaderboard |
| s18 | Classement | Global leaderboard |
| s19 | Spots favoris | Saved spots from `FAVS` set |
| s20 | Proposer un spot | Crowd-sourced spot suggestion form |

### Navigation engine

Screens use three CSS classes: `on`, `off-right`, `off-left`. The JS maintains a history `stack[]` array.

- **`go(idx)`** — pushes `s{idx}`, animates in from right. Triggers lazy `initMap()` when entering s1.
- **`back()`** — pops stack, slides current screen back to the right.
- **`goTab(idx)`** — switches main tabs (Carte=1, Sessions=2, Défis=3, Profil=4). Slide direction is inferred by comparing numeric IDs. **Resets `stack` to `[newId]`**, clearing all forward history.
- `resetAll(exceptIds)` + `setClass(id, cls)` are internal helpers. `el.offsetHeight` is called before setting `on` to force a reflow and trigger the CSS transition.

The bottom nav bar (with active-state styling) is duplicated inside each tab screen (s1–s4). The FAB (`+`) always calls `go(15)`.

### Key data structures

- **`SESSIONS_DATA`** — object keyed by sport slug. Passed to `openSession(id)` to populate s6 fields via DOM writes.
- **`SESSIONS`** — array of `{lat, lng, type, name, lieu, meta, players}` objects, used only in early map iterations (currently the map shows `SPOTS` only).
- **`SPOTS`** — array of 11 outdoor sport locations around Bordeaux (lat/lng, sport, name, hours, accessibility, rating, state, reviews). Rendered in the map and s16. Indexed by position; `curSpot` global tracks the open spot.
- **`spotRT`** — runtime state array mirroring `SPOTS`, holds mutable `{state, when, confirms}` updated when users report spot conditions.
- **`STICKERS`** — object mapping sport keys (`football`, `basket`, `tennis`, `skate`, `badminton`, `muscu`) to base64-encoded PNG data URLs. Applied at init via `applyStickers()` to all `[data-stk-img]`, `[data-stk-bg]`, and `[data-stk-over]` elements.
- **`SFsel`** — sessions filter state: `{sport, date, dist, level, fav}`. Mutated by `cycleSF(key)` / `toggleSFFav()`, applied by `applySF()`.

### Map (s1)

- Leaflet.js via CDN. Stored in `mapInstance` (lazy init on first entry to s1).
- Tiles: CartoDB `light_all` — free, no API key required.
- Only `SPOTS` are plotted as markers (sessions were removed from the map). Clicking a marker calls `openSpot(i)`.
- Custom SVG pin icons per sport colour: `makeSpotIcon(s)` and `makeUserIcon()`.
- Search uses the public Nominatim geocoding API (`nominatim.openstreetmap.org`). `geocode(q, flyFirst)` fetches results; `flyToPlace(i)` pans the map.
- Bottom sheet is draggable with 3 snap heights managed by `sheetIdx` and `snapSheet(idx)`.

### State & persistence

- **`localStorage`**: only `aggoFavSpots` (JSON array of spot indices in the `SPOTS` array). Loaded into a `Set` called `FAVS` at startup.
- No other persistent state — all other UI state (filters, session fav, etc.) resets on page reload.
- Login (`loginCheck`) only checks that fields are non-empty; there is no real auth.

### Design system

CSS variables defined in `:root`:

```
--lime: #C0F333   --pink: #FF2CB5   --beige: #E5E5DC
--black: #000     --white: #fff     --green: #3DB950
--g1: #F7F7F7     --g2: #EBEBEB     --g3: #B0B0B0    --g4: #555
```

Brand font `Groonex` is embedded as a base64 TTF at the top of the `<style>` block. UI font is DM Sans (Google Fonts CDN).

`fitScene()` scales `.scene` proportionally so the phone frame fits any viewport size.
