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
- **R4 / R5** — star rating (0–5★)
- **Reaction time** — time from GO to click (in seconds)
- **Total score** — overall star rating (1–5★)

**Star calculation:**
- Breaking R1 or R2 → automatic 1★
- Otherwise: weighted sum of R3 (next spots), R4 (entrance zone), R5 (distance)
- ≥80% of max → 5★ / ≥60% → 4★ / ≥40% → 3★ / else → 2★
- Minimum passing score to advance: 2★

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

### Level 4 — R4: Stay inconspicuous
```
0 0 0 0 0 0 0 1
0 0 0 0 E 0 0 0
```
- ✅ Pass: positions 4, 5, or 6 (`00002001`, `00000201`, etc. — sweet zone)
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
| L11 | 7 urinals, #4 occupied, #2 and #6 recently used | Left |
| L12 | 7 urinals, #2 #4 #6 occupied | Right |
| L13 | 7 urinals, NPC last-minute arrival at stall 5 | Left |
| L14 | 9 urinals, #3 occupied + NPC takes best remaining | Left |
| L15 | 9 urinals, #3 and #7 occupied, #9 broken | Centre |

---

## NPC Last-Minute Arrival (L13–L14)

On NPC levels, a red dot bursts in during the countdown and claims a stall before the player can act.

- Countdown: `3` → blue dot appears at door and walks in
- On `1` → red NPC dot appears and races to its scripted stall
- On `GO` → NPC is settled, player can now click
- After click → blue dot walks to chosen stall, result reveals on arrival

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
    └── Debug → All levels accessible via nav bar
                Debug table + weight sliders + reaction time stats visible
```

---

## Countdown Animation (all levels)

All levels use the same countdown before the player can click:

- `3` → Blue dot (player) appears at door
- `200ms` → Blue dot walks into room, settles near door
- `2` → 
- `1` → (NPC levels only) Red dot bursts in and walks briskly to its stall
- `GO` → Timer starts, player can click
- After click → Blue dot walks leisurely to chosen stall (700ms)
- On arrival → Stalls turn green/red, score popup appears

---

## Stall Colour Feedback (after pick)

After the blue dot arrives at the chosen stall:
- 🟢 **Green** — stall would have been a passing pick (≥2★)
- 🔴 **Red** — stall would have failed
- Picked stall shows **✓** or **✗**
- Occupied/broken stalls remain unchanged

---

## Play vs Debug Mode

| Feature | Play | Debug |
|---------|------|-------|
| Nav bar (L1, L2…) | ✗ | ✓ |
| Level tag / titles | ✗ | ✗ |
| Progress dots | ✗ | ✗ |
| Debug score table | ✗ | ✓ |
| Weight sliders (R3/R4/R5) | ✗ | ✓ |
| Reaction time stats | ✗ | ✓ |
| Rules intro screens | ✓ | ✗ |
| Score popup | ✓ | ✓ |

---

## Backlog / Future Features

- [ ] **Global ranking system** — score per level feeds into a leaderboard. Game is virtually infinite (levels cycle/expand). No progress bar — ranking is the progression mechanic.
- [ ] **Kids stall** — special stall type with unique logic
- [ ] **New layouts** — U-shape and stadium configurations
- [ ] **Suspiciously far walk** — soft penalty for over-walking
- [ ] **Remove Debug mode** — before public share / production release
- [ ] **Android + Google Account login** — for ranking system

---

## Technical Notes

- Single file: `index.html` (~43KB)
- No dependencies, no build step
- Deployed via GitHub Pages from `main` branch
- Live URL: https://shaybogomoltz.github.io/Wheretopee/
- Repo: https://github.com/shaybogomoltz/Wheretopee
