# CLAUDE.md — Kansai Trip Companion App

Guidance for Claude Code when working in this repo. Read this first, then read `status.md` (live progress) and `implementation.md` (full plan + findings log).

## What this is

A single-file, offline-capable **family trip companion web app** for a Kansai trip (8 days, **Jun 2–9 2026**; a family of 4 — husband, wife, 2 boys aged **11 & 13**, travelling from Singapore). Everything lives in **`index.html`** — inline CSS + vanilla JS, no build step, no dependencies, no framework. It must stay openable as a plain file and work offline (only user-tapped external links — Google Maps, forecast, `tel:` — hit the network).

## Files

- **`index.html`** — the entire app (data + render + styles).
- **`CLAUDE.md`** — this guide.
- **`status.md`** — live progress / where to resume (update every phase).
- **`implementation.md`** — the approved plan, phase checklist, and per-day findings log with sources (update every phase).
- **`research.md`, `itinerary.md`** — background reference material written before the trip. Read for context, but **verify against the live web** before trusting — they may be stale.

## Architecture of `index.html`

Three tabs (Days / Reference / Food) rendered from JS data arrays into the DOM. Key sections (line numbers drift as content grows — search by symbol):

- **Data** (`const` arrays near the top of `<script>`):
  - `AREA` — region → color (`osaka`/`kobe`/`kyoto`/`nara`/`travel`).
  - `DAYS[]` — one object per day. **This is where almost all day-by-day editing happens.**
  - `STAYS[]`, `TRANSPORT[]`, `TASKS[]`, `PACKING[]`, `EMERGENCY[]`, `EHOSP[]`, `FX[]` — Reference tab.
  - `PLACES[]` — Food & Places tab (restaurants/cafes/bookshops).
- **Helpers:** `mapURL(q)`, `dayRouteURL(d)`, `weatherHTML(d)`, `placeQuery(p)`, icon constants `PIN` / `PHONE` / `ROUTE`.
- **Render:** `renderDays()` → per day calls `weatherHTML(d)` + `d.blocks.map(blockHTML)` + `navMapsHTML(d)`. Also `renderRef()`, `renderFood()`, `renderStatus()`.

### The `DAYS[]` day object

```js
{
  n:1, date:'2026-06-02', wd:'Tuesday', area:'osaka',
  title:'Arrival in Osaka',
  stay:'Airbnb · Fukushima-ku',
  weather:{city:'Osaka', hint:'...html...'},        // optional → renders weather strip
  maps:[ {label:'…', q:'google maps search query'} ],
  route:['query A','query B','query C'],             // optional → orders the "Day route" button; else uses maps[].q
  blocks:[ /* see block types */ ]
}
```

### `blocks[]` types (handled in `blockHTML()`)

- `logistics` / `eat` / `tip` / `see` — bullet lists. Each item is `['', '<html>']` **or** `['', '<html>', 'map query']` — a **3rd element adds an inline "📍 Map" link** (`mapLink()`).
- `sched` — time-stamped rows `['8:00','<html action>']`, with an **optional 3rd element = map query** (`['8:00','…','map query']`) → Map link on that row.
- `options` — `items:[ {tag:'Option A', name:'…', rec:true, points:[…], sched:[…]} ]`:
  - `rec:true` highlights the pick.
  - each `points[]` entry is a plain string **or** `['<html>','map query']` to add a Map link (`pointHTML()`).
  - **`sched:[ ['~13:30','<html>', 'map query?'] ]`** (optional) renders a **"Suggested timing"** mini-schedule inside the option.
- `callout` — `items:[ ['🍄','<html>'] ]` (emoji + emphasis box).
- `booking` — e-ticket card: `{title, bk, warn, cells:[{k,v,map,full}]}`.

Each block may have a `title`; otherwise a default label/icon from the `heads` map in `blockHTML()` is used.

**Standing requirements for every day (owner asks):**
1. **Afternoon/activity `options` must include a `sched`** (a suggested time schedule) for the proposed plan.
2. **Dinner/`eat` spots (and any specific venue in points/sched) must carry a map query** so they render a tappable Map link. Make the query unambiguous (venue + ward/city).

## Conventions (match the existing voice & format)

- **HTML inside JS strings:** strings are single-quoted. Use typographic apostrophes/quotes (`’ “ ”`) inside copy to avoid escaping. Use `&amp;` for `&`, `&lt;`/`&gt;` if needed. Bold key facts with `<b>…</b>`; secondary emphasis `<i>…</i>`.
- **Tone:** concise, phone-friendly, family-practical. It's a companion, not a guidebook — favor specific numbers (¥, minutes, hours, station names, closed-days) over prose.
- **Colors:** never hardcode; use the per-area `--c` / `--c-soft` CSS vars (set from `AREA`).
- **Maps:** `q` is a Google Maps search string — make it unambiguous (include city/ward). `route` should list a day's stops in travel order.
- Don't introduce dependencies, build steps, or split the file. Keep it offline-first.

## The day-by-day review workflow (current project)

Sequential, **one phase per day** (Phase 0 = scaffolding done; Phases 1–8 = days; Phase 9 = QA). For each day:

1. **Research subagent** (`Agent`, `subagent_type:general-purpose`, with WebSearch/WebFetch). Give it the day's current `DAYS[]` object + the exact 2026 weekday/date + traveller profile. Ask for a structured report: per-venue **2026 hours / prices / weekly closure day** (flag conflicts with that weekday — highest-value check), early-June-2026 events/closures, must-book-today items, transit time/fare accuracy, weather outlook, and a **source URL for every claim**.
2. **Adversarial check** for surprising/contradicting facts — only write confirmed facts as fact; hedge the rest as "verify on arrival."
3. **Integrate into `index.html`**: correct stale facts; enrich (walking/transfer minutes, a book-now/on-arrival callout, an indoor **rain backup**, **kid notes** for ages 11 & 13); ensure `maps[]` covers the real stops; set `d.weather`. Propagate corrections into `PLACES[]`/`TRANSPORT[]`/`TASKS[]` when relevant. Owner wants each day **expanded with a lot more detail.**
4. **Validate, log, pause.** Run the JS syntax check (below). Append a findings+sources entry to `implementation.md` and tick the phase box. Update `status.md`. Pause for owner review before the next day.

Guardrails: never replace a concrete confirmed detail with a vaguer one; preserve voice/format; every changed factual claim must trace to a source in `implementation.md`.

## Validate after every edit to `index.html`

The embedded script must stay syntactically valid:

```bash
node -e 'const fs=require("fs");const h=fs.readFileSync("index.html","utf8");const m=h.match(/<script>([\s\S]*)<\/script>/);fs.writeFileSync("/tmp/app.js",m[1]);' && node --check /tmp/app.js && echo "JS OK" && rm /tmp/app.js
```

No browser is available in this environment — ask the owner to open `index.html` to eyeball rendering (weather strip, "Day route" button, day cards expand, tabs, localStorage task checkboxes persist).

## Git

Branch `main`. Commit only when the owner asks. Repo is a personal/Singabyte account — see global git-account rules. Commit message co-author trailer per global instructions.
