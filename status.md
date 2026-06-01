# STATUS — Day-by-Day Review

> Live resume pointer. Context is cleared between phases. **To resume: read `CLAUDE.md`, then this file, then `implementation.md`.** Update this file at the end of every phase.

## Where we are

- **Last completed:** Phase 9 — Cross-cutting QA & wrap-up ✅. **ALL PHASES (0–9) COMPLETE.**
- **Next up:** **Owner review + final commit.** Awaiting owner sign-off to make a single clean commit on `main` (per CLAUDE.md, commit only when owner asks). No browser here — owner should eyeball `index.html` once (weather strips, Day-route buttons, tabs, task checkboxes persist).
- **Phase 9 outcome:** consistency sweep clean; Reference-tab fact audit fixed 5 items — **FX rate ~¥115→~¥125 = S$1** (whole table rebuilt), **AMDA number 050-3598-7574→03-6233-9266** (was wrong), **TRANSPORT Arima→Kyoto ¥1,850→¥2,000**, Haruka footnote ¥3,060→~¥2,900–3,100, and removed the false "no Osaka consulate" claim. Plus 37 bare ` & `→` &amp; `. Details + sources in `implementation.md` → Phase 9.
- **Mode:** Phases 1–5 done sequentially; 6–8 parallelized across sessions per owner. Owner wanted **each day expanded with a lot more detail** (done).

## Progress

| Phase | Day | Date | Status |
|---|---|---|---|
| 0 | Scaffolding (route map + weather hooks) | — | ✅ done |
| 1 | Arrival in Osaka | Tue Jun 2 | ✅ done |
| 2 | Osaka morning + USJ from 3pm | Wed Jun 3 | ✅ done |
| 3 | USJ full day | Thu Jun 4 | ✅ done |
| 4 | Kobe lunch → Arima Onsen | Fri Jun 5 | ✅ done |
| 5 | Arima → Kyoto | Sat Jun 6 | ✅ done |
| 6 | Arashiyama day | Sun Jun 7 | ✅ done |
| 7 | Kyoto book day or Nara | Mon Jun 8 | ✅ done |
| 8 | Departure · KIX 6pm | Tue Jun 9 | ✅ done |
| 9 | Cross-cutting QA & wrap-up | — | ✅ done |

## How to run the next phase (recipe)

1. Open `index.html`, find the **Day 5** object in `DAYS[]` (search `n:5,`). Read its current content.
2. Spawn a **research subagent** (`Agent`, `general-purpose`, WebSearch/WebFetch) with that day object + "Sat 6 June 2026" + traveller profile (family of 4 — husband, wife, 2 boys 11 & 13). Ask for: the **Arima → Kyoto highway bus** reality (the day card has a *booked e-ticket*: Kyoto–Arima Onsen Liner, **Keihan Bus**, dep Arima **11:30** → Kyoto Stn **12:45**, ~¥1,850 — verify operator/timetable/fare/boarding spot still valid for Sat Jun 6 2026), the **Saturday** behaviour of Fushimi Inari / Nishiki Market / any Kyoto first-afternoon stops (weekend crowds, hours, closures), Kyoto Airbnb check-in 3pm logistics from Kyoto Station (Bishamoncho, Shimogyo-ku), weather — **source URL per claim**.
3. Integrate corrections + heavy enrichment into the Day 5 object (refresh `weather`, `maps`, optional `route`; options carry a `sched`; eat venues carry map queries). See `CLAUDE.md` for block types & conventions.
4. **Validate JS** (command in `CLAUDE.md`).
5. Append a findings+sources entry under "Phase 5" in `implementation.md`; tick the Phase 5 box there.
6. Update this file (move Last completed → Phase 5, Next up → Phase 6).
7. Report to owner and pause.

## Day 5 — known focus areas to verify hardest

- **Arima → Kyoto bus is already e-ticketed** in the day card (`booking` block): Kyoto–Arima Onsen Liner, **Keihan Bus**, dep **11:30** Arima → **12:45** Kyoto Station, ~¥1,850. Verify the 2026 operator/timetable/fare and the **boarding spot** at Arima are still correct; confirm the "if you miss it" fallback (Shintetsu/bus → Sannomiya → JR Special Rapid to Kyoto, or Shinkansen from Shin-Kobe).
- Arima check-out **10am**; ~1.5 hr to fill before the 11:30 bus → final stroll/soak + tansan-senbei souvenirs.
- **Kyoto first afternoon (Saturday):** Kyoto Airbnb (Bishamoncho, Shimogyo-ku) check-in 3pm; bus lands 12:45 → ~2 hr gap. Light afternoon, then check in. Verify any planned stops for **Saturday weekend crowds/hours** (Fushimi Inari is 24h but very busy Sat; Nishiki Market shops ~10:00–18:00).

## Notes / open verify-on-arrival items (carried forward)

- Day 8: exact walk-up Haruka fare; 4 non-reserved Haruka seats together; KIX security wait; coin-locker availability at Kyoto Stn/Namba; one-off Tue dept-store closures. **Load-bearing open question: the family's airline → which KIX terminal (T1 = train/walk-off; T2 = LCC, free shuttle).** Confirmed: ICOCA refund = ¥500 deposit back in full, ¥220 fee off balance (capped); Haruka ¥2,200 adult / ¥1,100 child (6–11) is foreigner-only, non-reserved, book online ahead; Fushimi Inari open 24h Tue.
- Day 6: no Jun-2026 festival/construction surfaced for the Arashiyama/Higashiyama temples (none found, not authoritatively confirmed); Gion Tanto weekly closure (likely Thu, not Sun); Pontocho Kyoshikian 2026 hours unverified; % Arabica open time (8:00 vs 9:00). **All Day-6 venues confirmed open Sun Jun 7; Nijo Castle Tue-closures are Jan/Jul/Aug/Dec only; Tenryu-ji has no Gioji combo (that's Gioji+Daikakuji); Kiyomizudera opens 6:00.**
- Day 4: Arima Toys & Automata Museum open Fri (irregular closures); Kobe Steak Propeller weekly closed day (none found); Roushouki Nankinmachi main-store reopening; exact Osaka→Sannomiya fares; coin-locker sizes; possible June bath maintenance at Kin no Yu. **Shinki bus ¥600 walk-on is the recommended Sannomiya→Arima option; JR/Honshi "Arima Express" ¥780 is reserved-seat (book on Kosoku Bus Net).**
- Day 3: exact Jun 4 USJ open/close + parade time + any nighttime show (left "check the app"); Mine Cart Madness single-rider availability; whether SNW rope-drop direct-entry holds on the day.
- Day 2: USJ Jun 3 park close time (left "check the app"); SNW weekday after-3pm walk-in is likely but not guaranteed; the Jan-5-2026 app-only rule rests on strong secondary sources (official page is JS-rendered) — eyeball the USJ app page before the trip.
- Day 1: Sobakiri Karani possible weekday rest day; exact Ichiran/551 branch hours for a Tuesday; child-ICOCA office queue on KIX T1 reopening day (Jun 2).
- Convention reminder: typographic apostrophes in JS strings; `&amp;` for `&`; bold key facts; keep it offline-first; no dependencies.

## Standing owner requirements (apply to EVERY day)

- **Options must include a `sched`** — a suggested time schedule for the proposed plan (renders as "Suggested timing" inside the option).
- **Dinner / eat venues must link to Google Maps** — add a map query as the 3rd tuple element on `eat` items (and on any specific venue in option `points`/`sched`). See `CLAUDE.md` → blocks for the exact data shapes (`mapLink`/`pointHTML`/`schedHTML`).
- Render layer already supports both (added end of Phase 1). Day 1 retrofitted.
