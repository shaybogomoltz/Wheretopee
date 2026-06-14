### v0.3.18
- **R#3 completely rewritten** — now simulates future arrivals following R1 optimally
  - `r3FutureCapacity(s, depth)` — recursive simulation: how many people can arrive before R1 breaks, capped at 2
  - Broken stalls transparent (reuses R1's `r1Neighbor`)
  - N/A when all valid picks leave the same future capacity
  - `00000`: ends (1,3,5) pass with capacity 2, middle (2,4) fail with capacity 1
  - Large empty rooms and tight rooms → naturally N/A
- Debug tables show capacity/max with N/A pill when rule doesn't differentiate
- Score popup shows N/A for R3 when applicable
- Bumped to v0.3.18

### v0.3.15
- **R#2 completely rewritten** using 4-tier cascading logic
  - SR1 — does your pick cause any NPC to become fully sandwiched? (avoidable = fail)
  - SR2 — do you end up sandwiched by NPCs? evaluated among SR1 survivors only
  - SR3 — does your pick increase partially man-sandwiched NPCs? SR1+SR2 survivors only
  - SR4 — does your pick increase partially wall-sandwiched NPCs? SR1+SR2+SR3 survivors only
  - Each tier only compares against picks that survived all previous tiers
- Old `maxSurround` / `anyAvoidSurround` functions removed
- Debug table (both modes): 5 new R#2 rows — NPC full, you sandwiched?, NPC partial man, NPC partial wall, decision
- Candidate filter in `bd()` updated to use new `r2Eval()` per stall
- Bumped to v0.3.15

# Where To Pee — Design Document

> A browser game about the unwritten rules of urinal etiquette.

---

## Concept

The player enters a bathroom and must choose a urinal. The goal is to pick the "correct" one based on a set of social rules that every man knows but never talks about. Each level introduces or tests one or more of these rules.

---

## The 5 Golden Rules

Rules 1–3 are **hard stops** — breaking them results in an automatic fail (1★).  
Rules 4–5 are **scored** — they contribute to the star rating (1–5★).

| # | Rule | Type | Scoring |
|---|------|------|---------|
| R1 | Whenever possible, leave an empty "buffer" between you and others. | Hard | ✓ / ✗ |
| R2 | Whenever possible, don't "trap" anyone between two occupied stalls. | Hard | ✓ / ✗ |
| R3 | Don't put an upcoming person into an unavoidable break of Rule #1. | Hard | ✓ / ✗ |
| R4 | Don't be the creepy "hiding something shameful" guy — avoid lurking at extremes. | Scored | ★★★★★ |
| R5 | Minimise walking distance while staying aware of the bathroom's entrance. | Scored | ★★★★★ |

---

## Scoring System

After the player picks a stall and the blue dot walks to it, a score popup appears showing:

- **R1 / R2 / R3** — ✓ Pass or ✗ Fail
- **R4 / R5** — star rating (1–5★)
- **Reaction time** — time from GO to click (in seconds)
- **Total score** — points out of 100

**Points breakdown (per level, max 100):**
- R4: up to 30 pts (`r4s / 5 × 30`)
- R5: up to 30 pts (`r5s / 5 × 30`)
- Speed: up to 40 pts (based on reaction time; 0 pts at ≥5s, 40 pts at ≤2s)
- Breaking R1, R2, or R3 → 0 pts total, game over

**R4 scoring detail:**
- R4.1 — proximity to others: scored relative to other valid candidates; being the "least conspicuously far" is best
- R4.2 — bell curve from door: very close to door (1★) → sweet zone mid-room (5★) → very far from door (1★)
- R4 combined = average of R4.1 and R4.2

**R5 scoring detail:**
- No occupied stalls: scored by walk distance from door (closer = better)
- With occupied stalls: sweet zone peaks at ~60% of max possible distance from nearest occupant

---

## Stall Status Legend

| Symbol | Meaning |
|--------|---------|
| 0 | Empty |
| 1 | Occupied |
| 2 | Player's chosen position |
| 3 | Broken |
| E | Entrance/door position (aligned to stall above) |

---

## Tutorial Levels (L1–L5)

Each tutorial level introduces one new rule. Before each tutorial level, a **rules screen** appears showing only the rules introduced so far (no spoilers for future rules).

### Level 1 — R1: Leave a buffer
```
1 0 0
0 E 0
```
- ✅ Pass: position 3 (`1 0 2`)
- ❌ Fail: position 2 (`1 2 0`)

*Reasoning: Stall 1 is occupied. The middle would put you right next to them. Pick the far end.*

---

### Level 2 — R2: Don't trap anyone
```
0 1 0 0 1 1
0 0 E 0 0 0
```
- ✅ Pass: position 1 or 3 (`2 1 0 0 1 1` or `0 1 2 0 1 1`)
- ❌ Fail: position 4 (`0 1 0 2 1 1`)

*Reasoning: R1 can't be respected — you'll always be adjacent to someone. So R2 kicks in: don't squeeze someone between two occupied stalls. Position 4 traps the person in stall 2.*

---

### Level 3 — R3: Think ahead
```
0 0 0 0 1
0 0 E 0 0
```
- ✅ Pass: position 1 or 3 (`2 0 0 0 1` or `0 0 2 0 1`)
- ❌ Fail: position 2 or 4 (`0 2 0 0 1` or `0 0 0 2 1`)

*Reasoning: Positions 1, 2, and 3 all respect R1 and R2. But position 2 leaves the next person only spots 1, 3, and 4 — all of which force an R1 violation. Position 3 adjacency (stall 4) also traps the next person. Only positions 1 and 3 leave a valid buffer option for the next arrival.*

*Note: 5 stalls chosen over 3 to make R3 meaningfully distinct from R1.*

---

### Level 4 — R4: Blend in
```
0 0 0 0 0 0 0 1
0 0 0 0 E 0 0 0
```
- ✅ Pass: positions 4, 5, or 6 (sweet zone)
- ❌ Fail: positions 1, 2, 3, 7, or 8

*Reasoning: Stall 8 is occupied so stall 7 breaks R1. R2 and R3 don't apply with so many stalls free. R4 kicks in: positions 1–3 are suspiciously far from the group; position 7 is adjacent. The sweet zone is 4–6.*

---

### Level 5 — R5: Minimise the walk
```
0 0 0 0 0 0 0 0
0 0 0 0 0 0 0 E
```
- ✅ Pass: positions 6, 7, or 8 (stalls near door, not immediately adjacent)
- ❌ Fail: positions 1–5 (over-walking) or position 8 (too close to entrance)

*Reasoning: Empty bathroom with door on the right. Don't trek to the far end unnecessarily. But don't hover right at the entrance either. The sweet zone is a few stalls in from the door.*

---

## Standard Levels (L6–L15)

All rules apply. No rules screen between levels — goes straight to the next level.

| Level | Setup | Door |
|-------|-------|------|
| L6  | 9 urinals, empty | Left |
| L7  | 7 urinals, #3 occupied | Left |
| L8  | 9 urinals, #5 occupied | Left |
| L9  | 7 urinals, #2 occupied + #4 broken | Left |
| L10 | 7 urinals, empty | Centre |
| L11 | 7 urinals, #4 occupied | Left |
| L12 | 7 urinals, #2 #4 #6 occupied | Right |
| L13 | 9 urinals, #3 and #7 occupied | Centre |
| L14 | 9 urinals, #3 and #7 occupied + #9 broken | Centre |
| L15 | 7 urinals, #1 and #7 occupied | Centre |

---

## NPC Last-Minute Arrival (L10+)

From L10 onward, an NPC may optionally appear during the countdown. Events are randomised with spacing (no two consecutive NPC levels):

- **Arrival only** (~50% chance): a red dot bursts in during the countdown and claims a stall before GO
- **Departure only** (~25% chance): an existing NPC walks out during the countdown, freeing a warm stall
- **Both** (L20+, ~15% chance): one NPC departs and a new one arrives

**Countdown timeline with NPC:**

| Time | Event |
|------|-------|
| 0ms | "3" — door opens, blue dot appears |
| 200–900ms | Blue dot walks in, settles near door |
| 1000ms | "2" — departing NPC (if any) starts walking out |
| 1100–1800ms | Departing NPC walks to door and disappears; stall becomes warm |
| 2000ms | "1" — arriving NPC (if any) bursts in |
| 2100–2700ms | Arriving NPC walks quickly to its stall |
| 3000ms | "GO" — NPC is settled, player can click |
| +700ms | Blue dot walks to chosen stall |
| On arrival | Stalls reveal green/red, score popup appears |

---

## Game Flow

```
Landing screen
    ├── Play →  Rules screen (R1 only)
    │               → L1 → Score popup → Rules screen (R1+R2)
    │               → L2 → Score popup → Rules screen (R1+R2+R3)
    │               → L3 → Score popup → Rules screen (R1+R2+R3+R4)
    │               → L4 → Score popup → Rules screen (all 5)
    │               → L5 → Score popup → L6 → L7 → ... (no rules screen)
    │
    └── Debug mode → Debug Chooser screen
                        ├── Debug Levels → all levels via nav bar
                        │                  debug table + weight sliders + stats
                        │
                        └── Debug Rules  → custom scenario builder
                                           inputs → apply → game visual → debug table
```

---

## Countdown Animation (all levels)

All levels use the same countdown before the player can click:

- `3` → Blue dot (player) appears at door
- `200ms` → Blue dot walks into room, settles near door
- `2` → (NPC departure, if scripted)
- `1` → (NPC arrival, if scripted) Red dot bursts in and walks briskly to its stall
- `GO` → Timer starts, player can click
- After click → Blue dot walks leisurely to chosen stall (700ms)
- On arrival → Stalls turn green/red, score popup appears

---

## Stall Colour Feedback (after pick)

After the blue dot arrives at the chosen stall:
- 🟢 **Green** — stall would have been a passing pick
- 🔴 **Red** — stall would have failed
- Picked stall shows **✓** or **✗**
- Occupied/broken stalls remain unchanged

---

## Play vs Debug Modes

| Feature | Play | Debug Levels | Debug Rules |
|---------|------|--------------|-------------|
| Nav bar (L1, L2…) | ✗ | ✓ | ✗ |
| Rules intro screens | ✓ | ✗ | ✗ |
| Score popup | ✓ | ✓ | ✗ |
| Timeout / fail state | ✓ | ✓ | ✗ (never fails) |
| Debug score table | ✗ | ✓ | ✓ (extended) |
| Weight sliders (R3/R4/R5) | ✗ | ✓ | ✗ |
| Reaction time stats | ✗ | ✓ | ✗ |
| Custom stall configuration | ✗ | ✗ | ✓ |
| Timing mode (3-2-1-Go) | always | always | optional |
| Result banner after pick | ✗ | ✗ | ✓ |

---

## Debug Levels Mode

Accessed via **Debug mode → Debug Levels**.

- Jump freely between all 15 base levels via the nav bar
- Full debug table visible at all times (updates after each pick and after countdown)
- Weight sliders let you adjust R3/R4.1/R5 scoring in real time
- Reaction time and best-time stats shown in the topbar
- No fail states — rule breaks are shown in the score popup but don't end the session

---

## Debug Rules Mode

Accessed via **Debug mode → Debug Rules**.

A custom scenario builder for understanding and validating rule logic. No fail states, no session score. Designed for iterating on edge cases.

### Inputs panel

| Input | Description |
|-------|-------------|
| Stall count | Slider from 2 to 12 stalls |
| Stall states | Per-stall dropdown: empty / occupied / broken (colour coded) |
| Entrance (door) | Button row selecting which stall the door aligns to |
| Timing checkbox | If checked, runs the 3-2-1-Go countdown animation before enabling clicks; never times out |

### Apply button

Clicking **Apply & render** locks in the current inputs and renders the game visual and debug table. Safe to re-apply at any time.

### Game visual

Normal SVG room rendered from the custom configuration. If timing is enabled, the player dot walks in via the countdown animation. After GO (or immediately if untimed), clicking any empty stall picks it and reveals the result.

### Result banner

After picking, a banner shows:
- ✓ Valid pick — R4 stars, R5 stars, estimated score (excl. speed)
- ✗ Rule broken — which rule (R1 / R2 / R3) and why

### Extended debug table

The Debug Rules table shows more variables than the standard debug table:

| Row | What it shows |
|-----|---------------|
| status | empty / occ / broken |
| r1 — buffer | n/a / ✓ / adj ✗ |
| ↳ adj to occ? | whether this stall is adjacent to an occupied one |
| r2 — surround | n/a / ✓ / surr ✗ |
| ↳ surround sides | how many sides of an occupied stall this pick would fill (0–2) |
| r3 — next spots | next-person valid spots after this pick / max across all candidates |
| r4 combined ★ | average of R4.1 and R4.2 |
| ↳ r4.1 not too far | score for distance to nearest occupied stall relative to other candidates |
| ↳ r4.2 bell curve | score for position relative to door (bell curve) |
| ↳ dist nearest occ | steps to nearest occupied stall |
| ↳ dist farthest occ | steps to farthest occupied stall |
| r5 — walk ★ | overall R5 score |
| ↳ walk from door | steps from the door to this stall |
| pts (excl speed) | estimated points from R4 + R5 (speed component excluded) |

Column highlight colours match the standard debug table: green = best candidate, grey = disqualified or ineligible.

---

## `bd()` — Per-Stall Breakdown Object

The core scoring function `bd(i, stalls, door)` returns a breakdown object for stall `i`. Key fields:

| Field | Type | Description |
|-------|------|-------------|
| `eligible` | bool | false if stall is occupied or broken |
| `disqualified` | bool | true if R1 or R2 is broken |
| `disqR` | string | `'r1'` or `'r2'` — which rule caused disqualification |
| `r1app` | bool | whether R1 is applicable (occupied stalls exist AND a non-adjacent option exists) |
| `r1pass` | bool | whether this stall passes R1 |
| `surr` | int | how many sides of an adjacent occupied stall this pick fills (0–2) |
| `spots` | int | valid next-person spots after this pick |
| `mxSpots` | int | max spots across all valid candidates |
| `r3pass` | bool | whether spots ≥ mxSpots − 1 |
| `r4s` | int | combined R4 score (1–5) |
| `r41s` | int | R4.1 sub-score (proximity) |
| `r42s` | int | R4.2 sub-score (bell curve from door) |
| `r5s` | int | R5 score (1–5) |
| `distToNearestOcc` | int | steps to nearest occupied stall (999 if none) |
| `distToFarthestOcc` | int | steps to farthest occupied stall |
| `walkD` | int | walk distance from door to this stall |
| `finalStars` | int | overall star rating (1–5) based on R4+R5 average |
| `cands` | int[] | indices of all valid candidates (pass R1+R2) |

---

## Backlog / Future Features

- [ ] **Global ranking system** — score per level feeds into a leaderboard. Game is virtually infinite (levels cycle/expand). No progress bar — ranking is the progression mechanic.
- [ ] **Kids stall** — special stall type with unique logic
- [ ] **New layouts** — U-shape and stadium configurations
- [ ] **Suspiciously far walk** — soft penalty for over-walking toward someone
- [ ] **Remove Debug mode** — before public share / production release
- [ ] **Android + Google Account login** — for ranking system

---

## Versioning

Version follows **MAJOR.MINOR.PATCH** (semver-inspired):

| Digit | When to bump |
|-------|-------------|
| MAJOR | Public launch, complete redesign, or breaking game mechanic change |
| MINOR | New feature, new mode, new rule, or new screen |
| PATCH | Bug fix, copy tweak, visual adjustment, or minor logic correction |

Version is displayed in two places on the landing screen:
- Under the tagline ("The unwritten rules of the urinal.")
- Fixed bottom-right corner (visible on all screens)

Controlled by the `GAME_VERSION` constant at the top of the script block in `index.html`.

---

## Changelog

### v0.3.8
- **R#1 completely rewritten** using tier-based logic from design spec (Rules.xlsx)
  - `isAlone()` — true when no NPCs present, all stalls pass automatically
  - `r1Classify()` — P (Perfect), Ac (Acceptable), Sd (Sandwich) per stall
  - `r1Neighbor()` — looks through broken stalls to find real neighbors; walls treated as empty
  - Priority order: must pick P if any exists; else must pick Ac; else Sd is forced, all pass
- **Debug table** (both Debug Levels and Debug Rules) now shows 5 R#1 rows matching the spreadsheet: alone?, perfect?, acceptable?, sandwich?, decision
- **`bd()` candidate filter** updated — candidates now determined by `r1Eval()` per stall, not old adjacency check
- **Level validator** added — warns in console if any base level has zero selectable stalls
- Bumped to v0.3.8

### v0.3.7
- **Debug table highlight fix** — `bests()` and `cc()` now require R3 to pass before a stall can be highlighted green. Previously stall #4 in `[X _ _ _ _ _ X]` was shown as best in the table even though clicking it correctly failed R3
- `finalStars` set to 0 whenever R3 fails — prevents R3-failing stalls from ever appearing as top candidates
- Both debug tables (Debug Levels and Debug Rules) affected

### v0.3.6
- **Tooltip repositioned** — purple pill now sits inline to the left of the badge in the same row, auto-sized, single line, never clips off-screen
- **Tooltips on all 5 rules** — hard rules (R1-R3) all show "Hard rule — get this wrong and your run is over"; scored rules (R4-R5) show "Scored rule — not just pass/fail, stars affect your score"
- **Rule names cleaned** — "R1 — Leave a buffer" → "Leave a buffer" everywhere on the rules screen (circle badge already shows the number)
- **Circle badges** now show "#1" not "R1" 
- **R1 → R #1** in score popup and debug table labels throughout
- **Stars now reflect total points** (including speed) — 90+ pts = 5★, 70+ = 4★, 50+ = 3★, 30+ = 2★. Previously stars only reflected R4+R5 average, ignoring speed entirely
- Bumped to v0.3.6

### v0.3.5
- Rules screen tooltips redesigned: no ⓘ icon — purple pill appears automatically on load, fades after 3s, reappears on hover
- R1 tooltip ("Get this wrong and your run is over") shown only on L1 intro screen
- R4 tooltip ("This isn't just pass/fail — stars go toward your score") shown only when R4 is first introduced
- Score popup ⓘ tips removed — too cluttered; debug table keeps its tooltips
- Bumped to v0.3.5

### v0.3.4
- **R3 logic fix** — `nextSpots()` now counts truly non-adjacent empty stalls after a pick, not all stalls when R1 becomes inapplicable. This fixes the bug where picking the centre stall in `[X _ _ _ _ _ X]` was incorrectly scoring higher than the off-centre picks
- **R3 pass threshold** tightened to exact match (`>= mxSpots`, not `>= mxSpots-1`) — a pick that leaves zero clean spots always fails R3 now
- **Tooltip** no longer requires focus state to dismiss — hover-only, vanishes immediately on mouse leave
- **New debug mode: Play from Level** — jump into play mode at any level with score pre-loaded as if you'd played perfectly (100 pts/level) up to that point
- Bumped to v0.3.4

### v0.3.3
- Renamed R4 "Stay inconspicuous" → **"Blend in"** everywhere (rules screen, score popup, debug table, tooltips)
- R4 description rewritten to plain language
- Rules screen: R1 tag now shows a hover tooltip "Get this wrong and your run is over" — only on the first screen (L1 introduction), not repeated after
- Rules screen: R4 tag shows a hover tooltip "This isn't just pass/fail — it's how well you pass. Stars go toward your score" — only when R4 is first introduced (L4)
- Score popup: R1, R4, R5 labels now have inline ⓘ tooltips explaining their type (hard rule vs scored rule)

### v0.3.2
- **Timer bar** now shows full (green) during the 3-2-1 countdown and only starts depleting on GO
- **Debug Levels rule break** — no longer sends you to game over; shows "Retry level" and "Next level" buttons instead
- **Play mode rule break** — delay before game-over screen increased from 2.5s to 4s so you can read the result
- **Game over screen** now auto-returns to landing after 3.5s (or immediately on "Back to start")
- **NPC events** now fire from L6 onwards (was L10); arrival probability raised to ~75%, departure to ~40%
- **Level designs overhauled** — all standard levels (L6–L15) now have at least one occupant; even stall counts (6, 8, 10) added throughout
- **No empty-room levels** after tutorial — every standard level starts with at least one NPC in a stall

### v0.3.1
- Fixed crash on page load caused by version script running before `corner-version` element existed in the DOM
- Wrapped version DOM writes in `DOMContentLoaded` listener

### v0.3.0
- Added **Debug Chooser** screen — "Debug mode" now branches into two sub-modes
- Added **Debug Rules** mode: custom scenario builder with stall count slider, per-stall state selectors, door picker, and optional timing mode
- Debug Rules includes an extended debug table with 14 rows and ⓘ hover tooltips explaining each metric and how it's calculated
- Tooltips use fixed positioning and follow the cursor to avoid clipping at screen edges
- Added version number display (landing tagline + fixed corner badge)
- `bd()` function extended with `r41s`, `r42s`, `distToNearestOcc`, `distToFarthestOcc`, `walkD` fields for richer debug output

### v0.2.0
- Added NPC departure events (NPC walks out during countdown, leaving a warm stall)
- Added combined NPC arrival + departure events (L20+)
- Sliding door animation on countdown open
- Hat system for NPCs — each NPC gets a unique character hat (Mario, Harry Potter, Link, etc.)
- Checkerboard floor tiles
- Improved urinal rendering (porcelain bowl, water reflection, flush button)

### v0.1.0
- Initial game: 5 tutorial levels + 10 standard levels
- Core rule engine (R1–R5) with hard stops and scored rules
- Countdown animation with player dot walking in
- NPC last-minute arrival (L10+)
- Score popup with reaction time and star rating
- Debug Levels mode with weight sliders and debug table
- Landing screen with leaderboard placeholder

---

## Technical Notes

- Single file: `index.html` (~91KB)
- No dependencies, no build step
- Deployed via GitHub Pages from `main` branch
- Live URL: https://shaybogomoltz.github.io/Wheretopee/
- Repo: https://github.com/shaybogomoltz/Wheretopee
