# Implementation Tracker — Day-by-Day Review & Enrichment

> Living document. Updated at the end of every phase. Every changed factual claim must trace to a source logged below.

## Context

`index.html` is a single-file, offline-capable Kansai family-trip companion (8 days, **Jun 2–9 2026**), built as data-driven vanilla JS: `DAYS[]`, `STAYS[]`, `TRANSPORT[]`, `TASKS[]`, `PLACES[]`, `FX[]`, `EMERGENCY[]` rendered into three tabs (Days / Reference / Food). It is already rich, but the content was written ahead of the trip and **the trip starts tomorrow (today is 2026-06-01)**. This effort is a disciplined day-by-day pass that (1) **fact-checks each day against the exact 2026 weekday/date** (hours, prices, weekly closures, events, must-book-now items), (2) **enriches** each day with on-the-ground usefulness, and (3) adds one new feature set: a **per-day route map + weather strip**. Accuracy and "what do I do/book TODAY" matter more than big new features.

Runs as **sequential per-day phases** — one day reviewed, verified, integrated, and shown for sign-off before the next — using research subagents with live web access plus an adversarial verification pass for anything uncertain.

## Scope (confirmed)

- **Priority:** Accuracy + usefulness. Each day fact-checked via live web research for its specific 2026 date.
- **New feature:** Per-day **route map** (one tap → Google Maps directions chaining that day's stops) + per-day **weather strip** (seasonal tsuyu hint + live forecast link for that city). No PWA, no notes/photos — out of scope.
- **Orchestration:** Sequential, one phase per day; subagents do research + drafting, integration by main agent, owner reviews between days.
- **Architecture:** Keep single-file `index.html`. No build step, no dependencies, stays offline-capable (only user-tapped external links hit the network).

## Critical files

- `index.html` — all changes land here.
  - Data: `DAYS[]` (~331–713), `STAYS[]` (~715), `TRANSPORT[]` (~721), `TASKS[]` (~734), `PLACES[]` (~764).
  - Render: `renderDays()` (~891), `navMapsHTML()` (~886), `blockHTML()` (~859), helpers `mapURL()` (~829), `placeQuery()` (~830).
- `implementation.md` — this tracker.
- `research.md` / `itinerary.md` — existing reference material; subagents read these first, then verify/extend with the web.

---

## Phase checklist

- [x] **Phase 0 — Foundation & feature scaffolding** (route map + weather render hooks, tracker, smoke test)
- [x] **Phase 1 — Day 1** · Tue Jun 2 · Arrival in Osaka
- [x] **Phase 2 — Day 2** · Wed Jun 3 · Osaka morning + USJ from 3pm
- [x] **Phase 3 — Day 3** · Thu Jun 4 · USJ full day
- [x] **Phase 4 — Day 4** · Fri Jun 5 · Kobe lunch → Arima Onsen
- [x] **Phase 5 — Day 5** · Sat Jun 6 · Arima → Kyoto
- [x] **Phase 6 — Day 6** · Sun Jun 7 · Arashiyama day
- [x] **Phase 7 — Day 7** · Mon Jun 8 · Kyoto book day or Nara
- [x] **Phase 8 — Day 8** · Tue Jun 9 · Departure · KIX 6pm
- [ ] **Phase 9 — Cross-cutting QA & wrap-up**

---

## Phase 0 — Foundation & feature scaffolding

Build the two reusable render hooks once, so each day phase only edits data.

1. **Create `implementation.md`** (this file).
2. **Per-day route map.** Add `dayRouteURL(d)` near `mapURL()`: builds a Google Maps **directions** URL from `d.maps[]` queries — `origin` = first stop (usually the stay), `destination` = last, middle stops as `waypoints=a|b|c`, `travelmode=transit`. Optional `d.route` array overrides stop ordering when chip order ≠ travel order. Render a **"Day route ↗"** button inside `navMapsHTML()`. Pure-derived → no data migration; degrades gracefully if a day has <2 maps.
3. **Per-day weather strip.** Optional `d.weather = {city, hint}`. Add `weatherHTML(d)` rendering a compact strip at the top of `.day-body`: the `hint` + a live **"Check forecast ↗"** link (`https://www.google.com/search?q=weather+<city>+<date>`). Renders only if `d.weather` exists, so days fill in progressively.
4. Minimal CSS for `.day-route` button and `.wx` strip, matching existing chip/callout language (reuse `--c`/`--c-soft`).
5. **Smoke test:** existing days still render; new hooks no-op cleanly before any data exists.

Subagent use: none (mechanical scaffolding).

---

## Phases 1–8 — One per day

Each day phase follows the same four steps:

1. **Research subagent** (`general-purpose`, `WebSearch` + `WebFetch`). Prompt includes the day's current `DAYS[]` object, relevant `research.md` notes, and the exact weekday/date. Returns a **structured findings report**:
   - Each named venue: confirmed 2026 opening hours, price (adult/child), **weekly closure day** (flag conflicts with the day's weekday — highest-value check).
   - Events/closures specific to early June 2026 (USJ anniversary, festivals, construction).
   - Anything that must be **booked/reserved today** to work on the trip date.
   - Transit legs: time/fare/transfer accuracy vs `TRANSPORT[]`.
   - Weather outlook for that city/date (tsuyu stage).
   - Inline source URLs for every claim.
2. **Adversarial verification** subagent for uncertain/surprising/contradicting findings — prompted to **refute**, default "unverified" if it can't confirm. Confirmed → written as fact; unconfirmed → hedged "verify on arrival" note.
3. **Integrate into `index.html`**: correct stale facts; add enrichment where thin — realistic **walking/transfer minutes**, a **"book now / on arrival" callout**, a **rain backup** (indoor option), **kid-specific notes** (ages 11 & 13); ensure `maps[]` covers real stops so the route button is useful; set `d.weather`. Propagate corrections into `PLACES[]`/`TRANSPORT[]`/`TASKS[]`.
4. **Render check + sign-off.** Confirm day card, route link, weather strip. Log findings + sources below. **Pause for owner review before next day.**

Guardrails: never overwrite a concrete confirmed detail with a vaguer one; preserve existing voice/format; keep additions concise (phone companion, not a guidebook); every changed factual claim traces to a source.

| Phase | Day | Date (2026) | Focus to verify hardest |
|---|---|---|---|
| 1 | Day 1 | Tue Jun 2 | KIX→Fukushima rapid + ICOCA, check-in/luggage, eSIM/ATM, **what to book tonight** |
| 2 | Day 2 | Wed Jun 3 | USJ 25th-anniv timed-entry reality, Osaka Castle/Kuromon hours |
| 3 | Day 3 | Thu Jun 4 | USJ rope-drop plan, single-rider availability, Studio Pass prices |
| 4 | Day 4 | Fri Jun 5 | Kobe beef lunch (reservations/closures), Arima onsen etiquette, ryokan logistics |
| 5 | Day 5 | Sat Jun 6 | Arima→Kyoto bus (already e-ticketed — verify), Fushimi Inari/Nishiki |
| 6 | Day 6 | Sun Jun 7 | Arashiyama hours, Monkey Park (cash-only, closures), weather swap plan |
| 7 | Day 7 | Mon Jun 8 | **Monday closures** — Manga Museum, bookshops, Nara/ninja options |
| 8 | Day 8 | Tue Jun 9 | Haruka Kyoto→KIX timing, last-day closures, ICOCA refund |

---

## Phase 9 — Cross-cutting QA & wrap-up

1. **Reference tab audit** (subagent + review): re-verify `TRANSPORT[]` fares/times, `TASKS[]`, `FX[]` rate note, `EMERGENCY[]`/`EHOSP[]` numbers, `PACKING[]`. Refine the rainy-season note if any day's research refined it.
2. **Consistency pass:** stays/dates align with day cards; every `maps[]`/route link well-formed; no day missing a weather strip; no broken HTML entities.
3. **Full app test** (see Verification). Then a single clean commit on `main` (only when owner asks).

---

## Verification

- **Render & interaction:** open `index.html` in a browser (and mobile viewport, 390px). All 8 day cards expand; Reference and Food tabs render; task checkboxes persist across reload (localStorage).
- **New feature:** each day's **"Day route"** → Google Maps transit directions through that day's stops in sensible order; **"Check forecast"** → forecast for the right city; weather strip shows on every day.
- **Offline:** load once, disable network — app still fully renders (only external map/forecast/tel links require network).
- **Facts:** spot-check high-risk items against sources below — especially **Monday (Jun 8) closures**, **USJ timed-entry**, **Arima→Kyoto bus**.
- **Regression:** diff `index.html`; only intended additions, existing days otherwise intact.

---

## Findings & changes log

> One entry per phase. Record: what changed, why, and the source URL for each verified fact.

### Phase 0 — ✅ complete
**Changes to `index.html`:**
- Added `.navchip.route` (filled, area-colored) + `.wx` weather-strip CSS after the `.navchip` rules (~line 114).
- Added `ROUTE` SVG icon constant beside `PIN`/`PHONE`.
- Added `dayRouteURL(d)` — builds a Google Maps **transit directions** URL chaining a day's stops (`origin` → `waypoints` → `destination`); uses optional `d.route[]` for explicit travel order, else `d.maps[].q`; returns `null` if <2 stops.
- Added `weatherHTML(d)` — renders a compact strip (seasonal hint + live "Forecast ↗" Google search link for `d.weather.city` + `d.date`); no-op unless `d.weather` is set.
- Wired **"Day route"** button into `navMapsHTML()`; prepended `weatherHTML(d)` inside `.day-body` in `renderDays()`.

**Verification:** route URL builds correctly (validated origin/destination/waypoints + transit mode), degrades to `null` for single-stop days; embedded `<script>` passes `node --check` (no syntax errors). Weather strip stays hidden until per-day `weather` data is added in Phases 1–8.

**No data changes** — route button auto-derives from existing `maps[]`, so all 8 days already show it. No sources (mechanical scaffolding).

### Phase 1 — Day 1 (Tue Jun 2) — ✅ complete
**Corrections (fact-checked vs 2026):**
- **Child ICOCA** is sold **only at the JR Ticket Office (Midori-no-madoguchi) with the child's passport** — not at vending machines. The **11yo qualifies** (child IC = ages 6 to under 12, half fares auto-applied); the **14yo pays adult fare**. Old copy implied all 4 cards bought at machines. — JR West FAQ (faq-support.westjr.co.jp), tamagodaruma.com
- **ICOCA is available again in 2026** (post the 2023–24 anonymous-card suspension); ¥2,000 (¥1,500 credit + ¥500 deposit). Buy at KIX **Terminal 1** (intl arrivals; T1 station 2F via Aeroplaza). — kansai-airport.or.jp FAQ
- **Haruka now stops at Osaka Station** (Umekita underground platforms, since Mar 2023) — the "Haruka + 1 loop stop" plan is valid; discount one-way **¥1,800 adult / ¥900 child**. — westjr.co.jp, en.wikipedia Haruka
- **Tennoji Zoo open Tue** (closed **Mondays**), 09:30–17:00 (last entry 16:00), adult ¥500, kids free/¥200. — the-kansai-guide.com
- **Abeno Harukas 300**: 09:00–22:00 (last entry 21:30); adult ¥2,000 / 14yo ¥1,200 / 11yo ¥700. — abenoharukas-300.jp
- **Don Quijote Dotonbori** confirmed 24h, Soemoncho 7-13. — gltjp.com
- Restaurants confirmed operating: **Hanakujira** oden daily 16:00–23:00 (Tuesday-safe), **POGO** (2-9-10 Fukushima), **Sobakiri Karani** 11–14/17–20:30 (flagged possible weekday rest), **551 Horai**, **MARUZEN & Junkudo** 7 Chayamachi open to 22:00. — tabelog, jpn-wabisabi.com

**Enrichments added:**
- New **"Arrival — order of operations"** schedule block (land → ATM/SIM → IC cards → Rapid → Fukushima walk), with the KIX-T1 new-commercial-area crowd warning for Jun 2.
- New **"Rapid vs Haruka"** logistics block; realistic walk times (JR Fukushima→Airbnb ~5–8 min; Fukushima→Umeda 1 stop/12–15 min walk); KIX coin-locker prices (L¥800/M¥600/S¥400).
- New **indoor rain/tired backup** block (depachika, MARUZEN, Pokémon Center Daimaru 13F, arcades, Kaiyukan 10:00–20:00).
- Expanded options with verified hours/prices; expanded dinner with a depachika/konbini fallback; first-night etiquette tips.
- Added **weather strip** (pre-tsuyu, ~20–27°C, shower possible) and extra map pins (Pokémon Center, Abeno Harukas); set explicit **`route`** = KIX → JR Fukushima → Dotonbori.

**Verify-on-arrival (left hedged):** Sobakiri Karani weekday rest day; exact Ichiran/551 branch hours for a Tuesday; child-ICOCA office queue on the busy reopening day; rainy-season exact onset (forecast ~Jun 6–7).

**Verification:** embedded JS passes `node --check`; route auto-builds from explicit `route[]`.

**Render-layer additions (post Phase 1, now standing requirements):**
- `options` blocks support an optional **`sched`** ("Suggested timing" mini-schedule) per option; option `points` may be `['<html>','map query']` for a Map link.
- List items (`eat`/`logistics`/`tip`/`see`) and `sched` rows support an optional **3rd tuple element = map query** → inline "📍 Map" link via new `mapLink()`/`pointHTML()`/`schedHTML()` helpers + `.li-map`/`.opt-sched` CSS.
- **Day 1 retrofitted:** both afternoon options now have suggested-timing schedules; all dinner spots link to Google Maps. These two rules now apply to every future day (documented in `CLAUDE.md` + `status.md`).

### Phase 2 — Day 2 (Wed Jun 3) — ✅ complete
**Closure check (Wed):** No blocking conflicts. Osaka Castle keep closed **Mondays** (open Wed). Kuromon has no market-wide Wed closure. Of the Dotonbori spots, **Mizuno is closed Thursdays** (not Wed) — flagged inline. — osakacastle.org, japan-guide.com, gltjp.com

**Corrections (fact-checked vs 2026):**
- **Osaka Castle keep admission raised to ¥1,200 adult** (was ¥600; raised **Apr 2025** with the new stone-wall museum). HS/college ¥600; **junior-high age &amp; younger free → both kids (11 &amp; 13) free**. Hours 9:00–17:00 / last entry 16:30 confirmed. *Adversarially confirmed.* — osakacastle.org/entrance-fee/
- **USJ scrapped all gate ticket booths on May 6 2025** — admission is online-only (same-day possible via the WEB Ticket Store on your phone, but can sell out). Old copy ("buy online to skip the gate queue") implied a gate queue still exists. *Adversarially confirmed via official press release.* — usj.co.jp/company/company_e/news/2025/0425/
- **Jan 5 2026: the official USJ app is the ONLY way to get Super Nintendo World timed-entry / standby (lottery) tickets** — in-park distribution machines no longer issue them; all passes in a group must be registered under one phone. (First researcher flagged this "unverified"; **adversarial pass CONFIRMED** it via the official announcement + 2026 JP guides. Official page is JS-rendered so rests on strong secondary corroboration — worth a 30-sec eyeball before the trip.) — usj.co.jp numbered-ticket app page; announcement screenshot.
- **25th anniversary "Discover U!!!"** confirmed: runs **Mar 4 2026 → Mar 30 2027** ("through Mar 2027" ✅). — nbcuniversal.com, tdrexplorer.com
- **1-Day Studio Pass** date-priced: adult (12+) ~¥8,600–10,900, child (4–11) ~¥5,400–6,200 → **13yo pays adult, 11yo is child**. — usj.co.jp/web/en/us/tickets

**Reframes / enrichments:**
- **SNW tonight callout flipped more optimistic:** on a quiet weekday USJ often **drops SNW area-entry restriction after ~2–3pm**, so a ~3pm arrival may walk straight in (~30–60 min). Kept the "do the full SNW deep-dive Jun 4" framing. — travelclassroom.net, kkday.com
- **Height limits softened:** tallest requirement is **132 cm** (Hollywood Dream, The Flying Dinosaur); both should clear unless the 11yo is petite; adult-companion exception doesn't apply to those two or Harry Potter. (Old copy: "both clear every height limit.") — japlanease.com, ourdepartureboard.com
- Added new **"Tickets &amp; height limits"** tip block (pass prices + child/adult split + Express Pass note).
- **Option A (Osaka Castle, recommended) now has a `sched`** (8:30 depart → 9:00 keep → lunch → ~14:00 to USJ) per standing requirement; added map links on points.
- **Dotonbori eat venues now carry map queries** (Wanaka, Mizuno, Daruma) per standing requirement; Kuromon points got map links + stall hours (~9:00–18:00, go before 11am).
- Added **weather strip** (tsuyu onset forecast ~Jun 6 → today likely dry, ~24–28°C, pocket umbrella) and explicit **`route`** = JR Fukushima → Osaka Castle → USJ.
- **Transit confirmed accurate:** Osaka Stn → Universal City via Nishikujo (or direct through-train ~12 min), ¥180; kept the conservative "~15–23 min, ~¥180–200." — osakastation.com
- **Propagated to TASKS[]:** `pre1` rewritten — no gate booths, buy online now, register all passes under one phone.

**Carry-forward for Phase 3 (Day 3, Thu Jun 4):**
- Day 3 callout (line ~493) still says "buy online ahead to skip the gate queue" — apply the **no-gate-booths-since-May-2025** correction there too.

**Headcount resolved (owner, start of Phase 3):** family of **4** — husband, wife, 2 boys aged **11 & 13** (the older boy was wrongly "14" everywhere). Fixed all "14yr/14yo" → "13" in `index.html`, and the profile in `CLAUDE.md`. App's "4 ICOCA / 4 tickets" was correct. (13 = junior-high age → still free at Osaka Castle; pays adult USJ Studio Pass & adult ICOCA fare — Phase 1/2 conclusions unchanged.)

**Verification:** embedded JS passes `node --check`; route auto-builds from explicit `route[]`.

### Phase 3 — Day 3 (Thu Jun 4) — ✅ complete
The existing rope-drop plan held up well against research; changes were corrections + targeted enrichment.

**Corrections (stale/wrong → fixed):**
- **Strategy callout:** removed "buy online ahead to skip the gate queue" → "buy online before you go — USJ scrapped its gate ticket booths May 2025." (Same fix as Phase 2, carried forward.) — usj.co.jp news 2025/0425
- **Kinopio's (Toad's) Cafe was wrong:** old copy said "mobile-order in app." Reality: **no online booking / no app pre-order — you scan a QR at the cafe door in person (GPS-gated, within ~80 m), slots release on the hour**, so queue ~1 hr ahead. Rewrote the lunch row and pointed families to counter-service (Mel's Drive-In / Studio Stars / Finnegan's, ~¥1,500–1,800 mains) as the easier bet. — nomadinjapan.com, usj.co.jp restaurants
- **Parade name:** confirmed it's the **"NO LIMIT! Parade ~ Discover U!!!"** 25th-anniversary parade (Mar 4 2026–Jan 11 2027, once daily). Time varies by date → kept "check the app." — usj.co.jp/.../parade, wdwnt.com

**Verified-as-accurate (kept, no change):**
- **Early opening is real** — USJ routinely opens before posted time, reports up to ~75 min early; "arrive ~60–75 min before posted open" stands. — travelcaffeine.com
- **SNW rope-drop direct entry** before timed-entry enforcement is real on quieter days (not guaranteed on peak days — already hedged in copy). — travelcaffeine.com, usj.co.jp timed-entry
- Single-rider lines confirmed on Mario Kart, Forbidden Journey, Flying Dinosaur, Hollywood Dream, JAWS, Minion. (Mine Cart Madness single-rider UNVERIFIED — copy doesn't claim it.) Added a **realism note**: savings vary (JAWS standout / coasters save less), lines aren't always open. — travelcaffeine.com, usj.co.jp single-rider
- Studio Pass prices + **13yo adult / 11yo child** split (fixed from "14yo" this phase).

**Enrichments added:**
- **Weather strip** (tsuyu ~Jun 6, ~26–28°C, ~75% humidity; ponchos + spare socks for Jurassic Park) + explicit **`route`** = Fukushima → USJ → Namba.
- **Dinner eat venues now carry map queries** (Ichiran Dotonbori main 24h; 551 Horai Namba main, closed 1st &amp; 3rd Tue but open Thu; Hozenji Yokocho sushi; Rikuro Ojisan Namba Honten) per standing requirement. — gltjp.com, ichiran.com, tabelog, rikuro.co.jp

**Verify-in-app-on-the-day (left hedged, not stated as fact):** exact Jun 4 open/close, parade start time, any nighttime SNW/lagoon show, Mine Cart Madness single-rider availability, whether SNW direct-entry holds on the day.

**Express Pass verdict (kept "no Express" plan):** a weekday rope-drop family using free app timed-entry + single-rider does not need it (~¥27,000–80,000+ for four). — kkday.com, usj.co.jp express

**Headcount fix applied this phase:** see the "Headcount resolved" note under Phase 2 — all "14"→"13", profile in `CLAUDE.md` updated, app's "4" confirmed correct.

**Verification:** embedded JS passes `node --check`.

### Phase 4 — Day 4 (Fri Jun 5) — ✅ complete
**Closure check (Fri):** No blocking conflicts. All Kobe lunch picks open Friday (Hiroshige closed Wed, Nagata Tank Suji closed Tue, Freundlieb closed Wed). Both Arima public baths open (Kin no Yu closed 2nd/4th Tue, Gin no Yu closed 1st/3rd Tue) — **Fri Jun 5 = weekday, both open & at weekday rate.**

**Corrections (fact-checked vs 2026):**
- **Sannomiya → Arima is TWO direct buses, both badged "Arima Express"** — the old copy presented only the JR "Arima Express" (¥780) as the simple no-transfer default, which is misleading. *Adversarially confirmed:*
  - **Shinki Bus ¥600, ~25 min, NO reservation** — board first / pay on exit, **ICOCA accepted**, ~hourly (the Kobe-Sanda Premium Outlets bus that stops at Arima). **Best for a family with luggage.** — shinkibus.co.jp, japan-guide.com
  - **JR/Honshi "Arima Express" ¥780 (¥700 if booked 7 days ahead), ~30 min — ALL reserved-seat** (Honshi weekdays, West JR Bus weekends), book via Kosoku Bus Net; you may still board on the day if seats are free. — nishinihonjrbus.co.jp, honshi-bus.co.jp, ja.wikipedia 有馬エクスプレス号
- **Train alt fare is ~¥720, not ~¥680** (Subway → Tanigami → Kobe Electric Rwy → Arimaguchi → Arima, 2 transfers). — japan-guide.com
- **Steakland lunch ~¥3,500** (150g Kobe-beef set), not ~¥3,000; 10:30–14:00/17:00–22:00, takes bookings. — steakland-kobe.jp
- **Hiroshige** gyudon ¥1,300 (large ¥1,500); 11:00–15:00/18:00–22:00, closed Wed; ~10 seats, walk-in only, sells out. — matcha-jp.com, tabelog
- **Nagata Tank Suji** 11:00–22:00, closed Tue; soba-meshi ¥800–950. — tabelog, gltjp.com
- **Freundlieb** 10:00–19:00, closed Wed (open Fri). — insidekyoto.com, feel-kobe.jp
- **Kin no Yu** ¥650 weekday/¥800 wknd, 8:00–22:00 (last 21:30), closed 2nd & 4th Tue; **Gin no Yu** ¥550/¥700, 9:00–21:00 (last 20:30), closed 1st & 3rd Tue; combo ¥1,200. Free footbath outside Kin no Yu (widely documented). — arimaspa-kingin.jp, visit.arima-onsen.com
- **Arima Gyoen** check-in 15:00 / out 10:00, on-site kinsen & ginsen baths; dinner+breakfast kaiseki plans. — en.arima-gyoen.co.jp

**Enrichments added:**
- Logistics rewritten to present both buses accurately (Shinki walk-on default vs reserved JR Express fallback) + the **"Arima Onsen Greedy Ticket" ¥2,400** (round-trip bus + both baths) money-saver; corrected ¥720 train alt.
- **Option A now has a `sched`** (11:30 Sannomiya/lockers → Kitano → 12:45 Kobe-beef lunch → 14:00 bus → 14:30 Arima) per standing requirement, with map links on points.
- **All Kobe-beef eat venues now carry map queries** per standing requirement; added 2026 hours/closed-days/reservation policy/price to each.
- Added **Taiko no Yu** day-spa (¥2,750 weekday; 13yo likely adult rate) and **Toys & Automata Museum** (10:00–17:30, ¥800/¥500, irregular closures — verify) with map queries; clarified weekday bath rates.
- Added a **"Sort today"** booking callout (reconfirm ryokan dinner, reserve JR bus seats or walk on Shinki, reserve Kobe-beef lunch).
- Refined onsen callout: **gender-separated baths → both boys (11 & 13) go to the men's side with Dad, Mum to women's** (both past mixed-bath age).
- Added **weather strip** (tsuyu cusp ~Jun 6 onset, Kobe ~26°C showers possible, Arima cooler/mistier in the hills) and explicit **`route`** (Sannomiya → Kitano → Nankinmachi → Bus Terminal → Arima Gyoen).
- **Propagated:** TRANSPORT[] Sannomiya→Arima row now shows both buses (¥600/¥780); TASKS[] `pre2` expanded (reconfirm dinner, reserve bus/lunch); PLACES[] Steakland price → ~¥3,500.

**Verify-on-arrival (left hedged):** Toys & Automata Museum open Fri (irregular closures); Kobe Steak Propeller weekly closed day (none found); Roushouki main-store reopening (was operating from sister shop "Souka Baozi Hall"); exact Osaka→Sannomiya fares to the yen; coin-locker availability/sizes; day-specific Jun 5 weather; whether any June bath maintenance at Kin no Yu.

**Verification:** embedded JS passes `node --check`; route auto-builds from explicit `route[]`.

### Phase 5 — Day 5 (Sat Jun 6) — ✅ complete
Transition day (Arima → Kyoto by highway bus, then a light Kyoto afternoon/evening). The e-ticketed bus was the highest-value check and held up; main work was fallback-fare accuracy, a dinner relabel, and standing-requirement enrichment.

**Bus (e-ticketed — verified, kept):**
- **Kyoto–Arima Onsen Liner (Keihan Bus)**, Arima Onsen **11:30 → Kyoto Sta Hachijo-guchi 12:45** (~75 min) — NAVITIME timetable shows exactly two Arima→Kyoto departures (11:30 & 17:20) and lists the **11:30 as running Saturday 6/6**. Boarding = **Arima Onsen Bus Info Center (有馬案内所)**. Fare **¥2,000 one-way** (Keihan official; older ¥1,850 figures stale) — added as reference on the card. **Reservation-priority** route; already e-ticketed. — keihanbus.jp, navitime.co.jp
- **Caveat flagged in-app (not as alarm):** the Keihan route page carries a *partial-suspension* banner while the live timetable still shows the 11:30 Sat departure → added a "reconfirm the week before" line + reservation phone **075-661-8200**.

**Miss-the-bus fallback (corrected):**
- **JR route:** Arima→Sannomiya via Shintetsu ~¥680 + Sannomiya→Kyoto JR Special Rapid **¥1,080** → ~90 min, **~¥1,760 total** (old copy said "~1 hr, ~¥1,110" — understated). — kyotostation.com
- **Shinkansen route:** Shin-Kobe→Kyoto **~¥2,870 unreserved, ~30 min — KEPT** (research subagent claimed this was wrong / "¥5,830 Nozomi-only"; **adversarial verification refuted that** — base ¥1,110 + 自由席特急 ¥1,760 = **¥2,870**; all Hikari stop at both, and 自由席 fare is identical across Nozomi/Hikari/Kodama). ¥5,830 was a booking-site markup. — ekitan.com, raillab.jp, tabiris.com

**Corrections / relabels:**
- **Kyou-no-Jidoriya Aki** was mislabelled "local-chicken, lunch ramen ~¥850." Reality: a **charcoal Kyoto-jidori (local chicken) izakaya near Nishiki Market (Nakagyo), dinner course from ~¥6,000pp** — no ramen. Relabelled + flagged as the priciest pick (reserve ahead). — kyonojidoriyaaki.gorp.jp
- **Fushimi Inari access kid-note:** take the JR Nara Line **Local (普通)** to JR Inari (~5 min ¥150) — **Rapid trains don’t stop at Inari** (common trap). Free, open 24h. — fushimi-inari.com, kyotostation.com

**Enrichments added:**
- **Weather strip** (tsuyu onset ~now; ~27°C/19°C, humid, passing showers; umbrella/poncho + quick-dry; torii + covered Nishiki arcade OK in light rain). — selfguidejapan.com
- **Explicit `route`** = Kyoto Sta Hachijo → Airbnb (Bishamonchō) → Nishiki → Fushimi Inari → Gion.
- **Both options now carry a `sched`** (standing requirement): Option A (Nishiki ~13:45 → Fushimi ~16:00 to dodge the Sat 11–3 crowd peak → Gion dinner); Option B (Gion → Nanzenji → Philosopher’s Path).
- **All dinner/eat venues now carry map queries** (standing requirement): Gion Tanto, Mimikou, Aki, Gion Tsujiri, % Arabica Higashiyama — with Sat hours, no-reservation/closed-day notes (Tanto closed Thu, Mimikou closed Sun). — tabelog, mimikou.jp, giontsujiri.co.jp, arabica.com
- **Kyoto Station lockers** ¥300/500/700/1,000 + **Crosta Kyoto** staffed-storage/hotel-delivery backup; Airbnb ~10 min NW walk confirmed (near Nishi Honganji). — kyotostation.com
- **Option B** enriched: Nanzenji paid halls 08:40–17:00 ¥600/¥400 (grounds + Suirokaku aqueduct free), Philosopher’s Path free/flat, **Gion side-alley photo ban** warning.
- **Rain backup** added (Fushimi torii / Nishiki arcade in light rain; Kyoto Station indoor complex — Ramen Koji, Isetan depachika, Porta, Yodobashi — for heavy rain).

**Verify-on-arrival (left hedged):** exact "H2" berth letter at Kyoto Sta Hachijo-guchi (confirm on the printed e-ticket/on-site signage); Keihan 11:30 still operating (reconfirm week before); individual Nishiki stall Saturday hours vary; % Arabica open 8 vs 9am.

**Verification:** embedded JS passes `node --check`; route auto-builds from explicit `route[]`; both options render a "Suggested timing" sched; all eat venues render Map links.

### Phase 6 — Day 6 (Sun Jun 7) — ✅ complete
Arashiyama + Kyoto temples. **No Sunday weekly closure on any venue** — the highest-value check came back clean. Work was a couple of fact corrections, structural conversion to `options`+`sched`, and heavy standing-requirement enrichment.

**Closure check (Sun):** all clear. Tenryu-ji, Kinkakuji, Ryoanji, Kiyomizudera, Nijo Castle, Okochi Sanso all open daily. Iwatayama Monkey Park open daily (closed Jan 1 + heavy rain only). Nishiki Market has no market-wide closure (but **individual shops often shut Sun/Wed**). — official sites below.

**Corrections (fact-checked vs 2026):**
- **Nijo Castle is OPEN Sun Jun 7.** Old copy's "closed some Tue" was misleading: Ninomaru Palace closes only **Tuesdays in Jan/Jul/Aug/Dec** (Jun fine); castle grounds close only Dec 26–31. Admission **¥800 grounds + ¥500 Ninomaru Palace = ¥1,300** (old "~¥1,300" was right but unexplained); 8:45–17:00, last palace entry ~16:10. — nijo-jocastle.city.kyoto.lg.jp/guide/annai
- **Tenryu-ji "¥600 combo with Gioji" removed — that combo is Gioji + Daikakuji, not Tenryu-ji.** Tenryu-ji = garden ¥500 (student ¥300) + ¥300 for buildings, opens 8:30. — tenryuji.com/en/visit, arashiyamabambooforest.com/gioji-temple
- **Kiyomizudera opens 6:00** (not 9:00) — Route B now starts at 7:30 to beat the Sunday crowds. ¥500 / **child ¥200** (elem/JHS). — onedayawaytravel.com/kiyomizudera-tickets
- **Kinkakuji ¥500 / child ¥300, 9:00–17:00; Ryoanji ¥600 / child ¥300, 8:00–17:00** (Jun). — kinkakujitemple.com/entrance-fee, ryoanji.jp/smph/eng
- **Monkey Park confirmed ¥800 / child ¥400, 9:00–16:30, cash only** (kept). — gltjp.com/en/directory/item/12282

**Structural change:** Routes A & B were two flat `sched` blocks → converted to a single **`options`** block (Route A = `rec`, both carry `points` + a "Suggested timing" `sched`) per the standing requirement. Added a `route[]` (Route A order) so the Day-route button chains the real stops.

**Enrichments added:**
- **Transit detail:** Arashiyama→Kinkakuji = JR to Enmachi (7 min ¥190) + bus #204/205, or City Bus #11/#93 (~30–40 min); Kinkakuji→Ryoanji bus #59 ~5 min; flat ¥230 city bus. — guidetokyoto.com, japan-guide.com/e/e3909
- **Named lunch venues with map queries:** Shoraian (riverside tofu kaiseki, ¥3,800–5,800, closed Wed, **phone-reservation only 075-861-0123**), Yudofu Sagano (¥3,000–5,000, 11:00–15:00), Arashiyama Yoshimura (soba ¥1,000–3,000, best-value kid pick). — shoraian.com, gltjp.com/en/directory/item/14747, /15341
- **Named dinner venues with map queries:** Gion Tanto (okonomiyaki, kid-friendly, ~¥2,000pp), Mouriya Gion (Kobe-beef teppanyaki, children 10+ → both boys OK), Pontocho alley stroll. — thetokyochapter.com/kyoto-restaurants-with-kids
- **`see` extras:** Okochi Sanso (¥1,000 / child ¥500 **incl. matcha + sweet**, 9:00–17:00), % Arabica (~9:00–18:00), **Sagano Romantic Train** (runs Sun, closed Wed, ¥880/¥440, reserve in peak), Togetsukyo riverbank. — insidekyoto.com/okochi-sanso-villa-arashiyama, arabica.com, sagano-kanko.co.jp/en/train-info
- **🐒 Monkey Park callout:** cash only, feed only from inside the caged hut, no eye contact/touching. **Weather strip** (tsuyu onset ~Jun 6–7, ~25–28°C, on-off showers) + **rainy-day backup** (Kyoto Railway Museum + Aquarium at Umekoji; Nishiki + Nijo indoor palace; swap Day 7). — japanhighlights.com/japan/rainy-season

**Verify-on-arrival (left hedged):** no Jun 2026 festival/construction surfaced for these temples (none found, not authoritatively confirmed); Gion Tanto weekly closure (likely Thu, not Sun); Pontocho Kyoshikian 2026 hours; % Arabica open time (9:00 vs 8:00 — one source each).

**Verification:** embedded JS passes `node --check`; route auto-builds from explicit `route[]`. (File was concurrently edited by the Phase 5 session — re-read Day 6 before editing; content was intact, only line-shifted.)

### Phase 7 — Day 7 (Mon Jun 8) — ✅ complete
The flexible "book day / Nara / ninja" day. **Highest-value check = Monday weekly closures — and the good news is none of the planned venues close Mondays.** Work was corrections (stale hours, a wrong location) + standing-requirement enrichment (a `sched` on every option, map links on every eat venue).

**Monday-closure check (all CONFIRMED open Mon Jun 8):**
- **Kyoto Intl Manga Museum** — weekly closed day is **Wednesday** (confirmed official; the "Tue/varies" rumor is wrong), so **open this Monday**. 10:00–17:00 (last adm 16:30). Prices ¥1,200 / ¥400 (JHS 13–18) / ¥200 (elementary 6–12) → the boys pay ¥400 (13) + ¥200 (11). **Added a defensive note: closed Jun 15–19 2026 for maintenance** (irrelevant to Jun 8, matters only if the day shifts). — kyotomm.jp
- **Todai-ji** daily, June 7:30–17:30, **¥800 adult (13+) / ¥400 child (6–12)**. Nara Park free/24h, shika senbei ~¥200. Kasuga Taisha 6:30–17:00, Kōfukuji ¥700 — all open Mon. — todaiji.or.jp
- **Ninja venues open Mon:** Kyoto Ninja Dojo (no fixed weekly closure), Samurai & Ninja Museum (daily 9:00–18:30), Iga-ryū Ninja Museum (open Mon, ¥800/¥500). Nijō Jin'ya still operating (Japanese-only tours). — ninjadojo.net, mansion-ninja.com, igaueno.net
- **Bookshops:** Seikōsha confirmed daily 10:00–20:00; **Hohohoza Monday hours UNVERIFIED** (only genuine gap — flagged "check before going" in copy).

**Corrections (stale/wrong → fixed):**
- **Keibunsha Ichijoji hours 10:00–21:00 → 11:00–19:00** (official/JP guides; English aggregators carry the stale 10–21). Propagated to **PLACES[]** too.
- **Kyoto Ninja Dojo location: "near Nijo" → near Shijō-Karasuma (Shimogyo-ku)** (was wrong); added price **¥8,000pp, must book ahead**.
- **Samurai & Ninja Museum:** price ~¥4,400 adult / ¥4,100 child, English tours every ~15–20 min (was "~every 30 min").
- **Nara transit clarified:** JR Miyakoji Rapid ~45 min ¥720 (JR Nara) vs **Kintetsu ¥760 ordinary** (Kintetsu-Nara closer to the park; reserved LE adds surcharge → ~¥1,490). Old copy implied Kintetsu Express was ¥760 — that's the ordinary fare; LE is pricier.

**Enrichments added:**
- **Weather strip** (tsuyu underway, ~28/19°C, humid, on/off showers) with an explicit indoor-fallback list — this day is strongly rain-proof (museums, bookshops, ninja museum, station dining); deer saved for a drier day. — selfguidejapan.com
- **`sched` added to ALL FOUR options** (standing requirement): A (Manga→Nishiki→Maruzen→Keibunsha), B (Nara deer→Todai-ji→Naramachi→Kōfukuji), C (arcades→Kyoto Station building), D (Ninja Dojo→Samurai museum).
- **Eat block restructured** so each venue carries a map query (standing requirement): Gyoza Chao Chao (Sanjo-Kiyamachi), Gyukatsu Katsugyu (Pontocho), Smart Coffee/Inoda, Ramen Kōji (10F) + Katsukura (The Cube 11F), Nakatanidou (Nara). — tabelog, kyoto-restaurant sources
- **Re-confirmed station venues:** Ramen Kōji = **10F**, Katsukura = **The Cube 11F** with free refills.
- **Pack-tonight callout** expanded: book the Ninja Dojo today; consolidate book/souvenir buys vs the SQ baggage limit (prep for Tue Jun 9 departure).
- Added **Seikōsha** map pin + a sensible Option-A book-day **`route`** (Kyoto Sta → Manga Museum → Nishiki → Keibunsha).

**Verify-on-arrival (left hedged):** Hohohoza Monday hours (unverified); Maruzen 11–20 vs 11–21; Inoda closing 18:00 vs 20:00; % the Manga Museum holiday-Wed shift rule (n/a Jun 8). Iga-Ueno is a full-day commitment — only if ninja is the whole theme.

**Verification:** embedded JS passes `node --check`; route auto-builds from explicit `route[]`; all four options render "Suggested timing"; all eat venues render Map links. Keibunsha hours fixed in both `DAYS[]` and `PLACES[]`.

### Phase 8 — Day 8 (Tue Jun 9) — ✅ complete
**Closure check (Tue):** No blocking conflicts for the departure plan. Fushimi Inari is open **24h, free, no weekly closure** (Tuesday fine). Kyoto Station omiyage/ekiben shops operate daily (Isetan dept store has occasional irregular days — non-blocking). — fushimi-inari.com/hours

**Corrections (fact-checked vs 2026):**
- **ICOCA refund wording was wrong** — old copy "¥500 back minus ¥220 handling" implied the fee comes off the deposit. *Adversarially confirmed via JR West FAQ:* the **¥500 deposit is always refunded in full**; the **¥220 handling fee is deducted from the remaining stored-value balance**, capped at that balance (balance ≤¥220 → fee = balance; balance ¥0 → no fee). Tip added: spend balance down near ¥0 first. — faq-support.westjr.co.jp, japan-guide.com/e/e2359_003.html
- **Haruka discount ticket:** the **¥2,200 one-way is foreigner-only** (Temporary Visitor passport), covers a **non-reserved** seat, and is now best **booked online ahead** (JR-WEST Online / HARUKA QR e-ticket) rather than as a same-day counter buy. **Child (6–11) = ¥1,100 → the 11yo pays ¥1,100, the 13yo pays adult ¥2,200.** Walk-up regular fare ~¥2,900–3,100 (old "~¥3,060" is in range, softened). ~75 min, every ~30 min. — westjr.co.jp/travel-information/en/tickets-passes/oneway/haruka
- **KIX terminal note added:** Haruka arrives at **Kansai Airport Station = Terminal 1** (full-service intl flights → walk straight to departures). **Terminal 2 is LCC-only with no train** (free shuttle from T1). Family must confirm their airline's terminal. — kansai-airport.or.jp/en/access/train
- **Fushimi Inari logistics corrected:** it's **2 stops south on the JR Nara Line** (~5 min, ¥150), **not on the Haruka line** — so old "on the way south" was misleading. Reworked to a **bag-free** visit using Kyoto Station coin lockers, returning to board the Haruka.
- **Nankai (Option B):** Rapi:t ltd express ~35 min ¥1,490 (¥300 off online) / Airport Express ~45 min ~¥930. — osakastation.com, nankai.co.jp

**Enrichments added:**
- **Option A now has a `sched`** (7:00 locker→Inari → 7:15 Senbon Torii loop → 8:15 back to Kyoto Stn → 12:45 Haruka) per standing requirement, with map links; kid note that the lower Senbon Torii loop (~30–45 min) beats the 2–3 hr summit hike on a departure morning.
- Departure timing detail (check-in closes 2 hrs before, self-kiosks open 3 hrs before; ~45 min security).
- **Map queries** on Fushimi Inari / Senbon Torii / Kyoto Station / KIX T1 / Dotonbori across options.
- Added **weather strip** (now in tsuyu, ~25–28°C, showers likely → favours the indoor Haruka route) and explicit **`route`** (Kyoto Station → Fushimi Inari → Kyoto Station → KIX T1).

**Verify-on-arrival (left hedged):** exact walk-up Haruka fare; whether 4 non-reserved Haruka seats are together on the chosen train; KIX security wait on the day; specific Kyoto Station/KIX shop hours + any one-off Tuesday dept-store closure; coin-locker availability at Kyoto Station/Namba; **the family's airline → which KIX terminal (T1 vs T2)**.

**Verification:** embedded JS passes `node --check`; route auto-builds from explicit `route[]`.

### Phase 9 — QA
_pending_

---

## Sources

> Master list of verified source URLs, grouped by phase. Populated as research completes.

**Phase 2 (Day 2 · Wed Jun 3):**
- Osaka Castle hours/price — https://osakacastle.org/entrance-fee/
- USJ gate-booth discontinuation (official, May 6 2025) — https://www.usj.co.jp/company/company_e/news/2025/0425/
- USJ SNW timed-entry / app — https://www.usj.co.jp/web/ja/jp/enjoy/numbered-ticket/app/timed-entry-ticket
- USJ Studio Pass tickets/prices — https://www.usj.co.jp/web/en/us/tickets
- USJ 25th anniversary "Discover U!!!" — https://www.nbcuniversal.com/article/complete-guide-universal-studios-japans-discover-u-25th-anniversary-celebration ; https://tdrexplorer.com/universal-studios-japan-announces-25th-anniversary-discover-u-event/
- SNW weekday walk-in / timed-entry behaviour — https://www.travelclassroom.net/eng/usj-app.html ; https://www.kkday.com/en/blog/54551/asia-japan-universal-studios-osaka-super-nintendo-world-timed-entry-tickets
- Ride height limits — https://japlanease.com/universal-studios-japan-rides-by-height/ ; https://www.ourdepartureboard.com/blog/universal-japan-ride-height-requirements
- Osaka Stn → Universal City transit — https://www.osakastation.com/the-jr-sakurajima-line-yumesaki-line-for-universal-studios-japan/
- Kuromon Ichiba hours — https://www.japan-guide.com/e/e4031.html
- Tsuyu 2026 onset forecast (~Jun 6) — https://selfguidejapan.com/blog/japan-rainy-season-2026
- Dotonbori venues (Wanaka, Ootako, Kukuru, Mizuno, Daruma) — gltjp.com ; fun-japan.jp ; dotonbori.or.jp

**Phase 3 (Day 3 · Thu Jun 4):**
- USJ park hours / early opening / rope-drop tips — https://www.usj.co.jp/web/en/us/park-guide/schedule/park-hour ; https://www.travelcaffeine.com/universal-studios-japan-tips/
- USJ SNW timed-entry — https://www.usj.co.jp/web/en/us/enjoy/numbered-ticket/timed-entry-ticket ; https://www.kkday.com/en/blog/54551/asia-japan-universal-studios-osaka-super-nintendo-world-timed-entry-tickets
- USJ single-rider — https://www.usj.co.jp/web/en/us/attractions/how-to-fun/single-rider
- Crowd calendar / live waits — https://www.thrill-data.com/trip-planning/crowd-calendar/usj ; https://queue-times.com/parks/284/queue_times
- 25th-anniversary "NO LIMIT! Parade ~ Discover U!!!" — https://www.usj.co.jp/web/en/us/25th-anniversary-discover-u/parade/ ; https://wdwnt.com/2026/03/no-limit-parade-discover-u-version-for-universal-studios-japans-25th-anniversary/
- Express Pass pricing/benefit — https://www.kkday.com/en/blog/57997/asia-japan-universal-studios-express-pass-guide ; https://www.usj.co.jp/web/en/us/tickets/express-pass
- Kinopio's Cafe QR/GPS entry — https://nomadinjapan.com/kinopio-toadstool-cafe/ ; counter dining: https://www.usj.co.jp/web/en/us/restaurants/mels-drive-in
- Osaka June weather / tsuyu — https://www.japanhighlights.com/japan/osaka/weather-june ; https://selfguidejapan.com/blog/japan-rainy-season-2026
- Namba dinner (Ichiran 24h, 551 Horai main, Rikuro Ojisan) — https://en.ichiran.com/shop/hourstable.html ; https://www.gltjp.com/en/directory/item/11882/ ; https://tabelog.com/en/osaka/A2701/A270202/27002336/

**Phase 4 (Day 4 · Fri Jun 5):**
- Sannomiya↔Arima buses (both options, fares, walk-on vs reserved) — https://www.japan-guide.com/e/e3558.html ; https://enjoy-osaka-kyoto-kobe.com/article/a/arima-onsen-access/
- Shinki Bus (¥600 walk-on, ICOCA, pay-on-board) — https://www.shinkibus.co.jp/highway/category/route_guidance/shinkobe_arima_express.html
- JR/Honshi "Arima Express" reserved-seat (¥780 / ¥700 早特7) — https://www.nishinihonjrbus.co.jp/trans/lp/arima/ ; https://www.honshi-bus.co.jp/express/view/8 ; https://ja.wikipedia.org/wiki/有馬エクスプレス号
- Steakland Kobe hours/price — https://steakland-kobe.jp/en/ ; https://japanuncharted.com/hyogo/wagyu/steakland-kobe-review
- Kobe Steak Propeller — https://www.the-kansai-guide.com/en/directory/item/22142/
- Hiroshige gyudon — https://matcha-jp.com/en/6372 ; https://tabelog.com/en/hyogo/A2801/A280101/28033392/
- Nagata Tank Suji — https://tabelog.com/en/hyogo/A2801/A280101/28007876/ ; https://www.gltjp.com/en/directory/item/16837/
- Nankinmachi street food (Roushouki) — https://www.byfood.com/blog/kobe/kobe-chinatown-street-food-guide ; https://www.gltjp.com/en/directory/item/11364/
- Freundlieb (Kitano) — https://www.insidekyoto.com/freundlieb ; https://www.feel-kobe.jp/en/facilities/detail/?code=0000000358
- Kitano Ijinkan — https://www.japan-guide.com/e/e3550.html ; https://www.gltjp.com/en/directory/item/11377/
- Kin no Yu / Gin no Yu hours/price/closures — https://arimaspa-kingin.jp/en-01.htm ; https://visit.arima-onsen.com/directory/do/gin-no-yu/
- Taiko no Yu day spa — https://www.gltjp.com/en/directory/item/14913/
- Arima Toys & Automata Museum — https://arima-toys.jp/en/facility/arima-toy-museum/
- Arima Gyoen ryokan (check-in/out, baths) — https://en.arima-gyoen.co.jp/
- Kobe June weather / tsuyu 2026 onset — https://www.weather2travel.com/japan/kobe/june/ ; https://hiddenjapan-gems.com/japan-tsuyu-2026/

**Phase 5 (Day 5 · Sat Jun 6):**
- Keihan Bus Kyoto–Arima route (¥2,000, reservation-priority, suspension banner, 075-661-8200) — https://www.keihanbus.jp/highway/line/kyoto-arima_onsen/
- NAVITIME Arima→Kyoto timetable (11:30 & 17:20; runs Sat 6/6) — https://www.navitime.co.jp/diagram/bus/00083827/00020407/0/
- Shin-Kobe→Kyoto Shinkansen fare (¥2,870 unreserved, ~30 min; refutes ¥5,830) — https://ekitan.com/transit/shinkansen/section/sf-5435/st-5198 ; https://raillab.jp/trip/shinkansen-kyoto-kobe ; https://shinkansen.tabiris.com/fare26.html
- Sannomiya↔Kyoto JR Special Rapid (¥1,080, ~52 min) — https://www.kyotostation.com/jr-kyoto-line-for-osaka-sannomiya-himeji/
- Fushimi Inari (free, 24h) + JR Nara Line Local to Inari (¥150) — https://fushimi-inari.com/hours/ ; https://www.kyotostation.com/jr-nara-line-for-tofukuji-inari-uji-nara/
- Nishiki Market hours / Sat fully open — https://www.insidekyoto.com/nishiki-market-downtown-kyoto ; https://www.japan-guide.com/e/e3931.html
- Kyoto Station lockers / Crosta luggage storage — https://www.kyotostation.com/kyoto-station-lockers-luggage-storage/
- Dinner venues — Gion Tanto https://tabelog.com/en/kyoto/A2601/A260301/26007590/ ; Mimikou https://mimikou.jp/honten/ ; Aki https://kyonojidoriyaaki.gorp.jp/ ; Gion Tsujiri https://www.giontsujiri.co.jp/store/giontsujiri-honten/ ; % Arabica https://arabica.com/en/location/arabica-kyoto-higashiyama/
- Nanzenji & Philosopher’s Path — https://www.japan-guide.com/e/e3905.html ; https://www.japan-guide.com/e/e3906.html
- Kyoto June weather / tsuyu 2026 onset — https://selfguidejapan.com/blog/japan-rainy-season-2026 ; https://www.umetravel.com/japan-weather/weather-in-june.html

**Phase 6 (Day 6 · Sun Jun 7):**
- Nijo Castle hours/closure (open Jun 7; Tue closures Jan/Jul/Aug/Dec) / ¥800 + ¥500 — https://nijo-jocastle.city.kyoto.lg.jp/guide/annai/?lang=en
- Tenryu-ji hours/price + Gioji combo correction — https://www.tenryuji.com/en/visit/ ; https://arashiyamabambooforest.com/gioji-temple/
- Iwatayama Monkey Park (¥800/¥400, 9:00–16:30, cash, daily) — https://www.gltjp.com/en/directory/item/12282/
- Kinkakuji (¥500/¥300, 9:00–17:00) — https://www.kinkakujitemple.com/entrance-fee/
- Ryoanji (¥600/¥300, 8:00–17:00 Jun) — http://www.ryoanji.jp/smph/eng/
- Kiyomizudera (6:00–18:00, ¥500/¥200) — https://www.onedayawaytravel.com/kiyomizudera-tickets/
- Okochi Sanso (¥1,000/¥500 incl. matcha, 9:00–17:00) — https://www.insidekyoto.com/okochi-sanso-villa-arashiyama
- % Arabica Arashiyama — https://arabica.com/en/location/arabica-kyoto-arashiyama/
- Kyoto Stn → Saga-Arashiyama (¥240, 15 min) — https://www.kyotostation.com/jr-sagano-line/
- Arashiyama → Kinkakuji / Kinkakuji → Ryoanji transit — https://guidetokyoto.com/how-to-get/kinkakuji/from-arashiyama-to-kinkakuji ; https://www.japan-guide.com/e/e3909.html
- Lunch venues — Shoraian https://www.shoraian.com/reservation-contact ; Yudofu Sagano https://www.gltjp.com/en/directory/item/14747/ ; Arashiyama Yoshimura https://www.gltjp.com/en/directory/item/15341/
- Dinner venues (Gion Tanto, Mouriya Gion) — https://thetokyochapter.com/kyoto-restaurants-with-kids/
- Sagano Romantic Train (runs Sun, closed Wed, ¥880/¥440) — https://www.sagano-kanko.co.jp/en/train-info/
- Nishiki Market hours / individual shop Sun-Wed closures — https://www.japan-guide.com/e/e3931.html
- Kyoto June weather / tsuyu 2026 onset — https://www.japanhighlights.com/japan/rainy-season/ ; https://www.umetravel.com/japan-weather/weather-in-june.html

**Phase 7 (Day 7 · Mon Jun 8):**
- Kyoto Intl Manga Museum (closed Wed, open Mon, hours, ¥1,200/¥400/¥200, Jun 15–19 maintenance) — https://kyotomm.jp/en/ ; https://kyotomm.jp/en/visit/
- Keibunsha Ichijoji hours 11:00–19:00 — https://www.keibunsha-store.com/ ; https://www.kyoto.travel/en/shopping/keibunsha.html
- Maruzen Kyoto BAL — https://honto.jp/store/detail_1572013_14HB320.html
- Seikōsha (誠光社) open daily 10:00–20:00 — https://www.seikosha-books.com/
- Todai-ji hours/admission (¥800/¥400) — https://www.todaiji.or.jp/en/information/
- Nara Park / deer / shika senbei — https://www.japan-guide.com/e/e4101.html
- Kintetsu vs JR Kyoto→Nara fares/times — https://www.kintetsu.co.jp/foreign/english/ ; https://www.japan-guide.com/e/e4115.html
- Kyoto Ninja Dojo & Store (Shijo-Karasuma, ¥8,000pp, book ahead) — https://ninjadojo.net/
- Samurai & Ninja Museum Kyoto (daily 9:00–18:30, English tours, prices) — https://mai-ko.com/samurai/
- Iga-ryū Ninja Museum (open Mon, ¥800/¥500) — https://www.iganinja.jp/en/
- Eat (Gyoza Chao Chao, Gyukatsu Katsugyu, Smart Coffee, Inoda, Katsukura/Ramen Koji floors, Nakatanidou) — tabelog.com ; https://www.kyoto-ramen-koji.com/ ; https://katsukura.jp/
- Kyoto June weather / tsuyu 2026 — https://selfguidejapan.com/blog/japan-rainy-season-2026 ; https://www.umetravel.com/japan-weather/weather-in-june.html

**Phase 8 (Day 8 · Tue Jun 9):**
- Haruka Kyoto→KIX time/frequency/fare + ¥2,200/¥1,100 foreigner one-way ticket — https://www.westjr.co.jp/travel-information/en/tickets-passes/oneway/haruka/ ; https://www.jrailpass.com/blog/haruka-express-kansai-airport
- HARUKA online/QR booking — https://www.westjr.co.jp/global/en/ticket/haruka-oneway/reservation/ ; https://www.kkday.com/en/blog/95359/jr-haruka-express-japan
- KIX terminals / train serves Terminal 1 / T2 shuttle — https://www.kansai-airport.or.jp/en/access/train ; https://www.kansai-airport.or.jp/en/faq/4831
- KIX check-in timing (2 hr close / 3 hr kiosks) — https://www.jal.co.jp/jp/en/inter/airport/kix/info/
- Nankai Namba→KIX (Rapi:t / Airport Express, fares) — https://www.osakastation.com/the-nankai-airport-line-the-airport-express-rapit-services-for-kansai-airport-namba/ ; https://www.nankai.co.jp/en/community/natts/odekake/rapit-express-airport-discount/
- ICOCA refund mechanics (¥500 deposit full, ¥220 fee off balance, capped) — https://faq-support.westjr.co.jp/hc/en-us/articles/8880706840847 ; https://www.japan-guide.com/e/e2359_003.html
- Fushimi Inari 24h / free / Tuesday open — https://fushimi-inari.com/hours/ ; https://fushimi-inari.com/entrance-fee/
- Kansai tsuyu 2026 onset (~Jun 6) / June conditions — https://selfguidejapan.com/blog/japan-rainy-season-2026 ; https://www.jrailpass.com/blog/rainy-season
