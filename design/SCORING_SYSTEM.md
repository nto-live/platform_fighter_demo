# SCORING_SYSTEM.md

> **Status: DESIGN PHASE ONLY.** Values are tuning targets, not final.

## 1. Why Score Matters

Score is the **bridge between the two halves of the game** (Pillar 5). It:
- Qualifies the player for the boss (score gate — but a *soft* one; see `PROGRESSION.md`).
- Determines **rank** (S/A/B/C/D).
- Converts into **combat loadout bonuses** at the handoff.

Therefore score must be **readable, honest, and never arbitrary** (Pillar 4). At any moment the player should have a rough sense: *am I on pace?*

## 2. The Scoring Formula (player-facing mental model)

We keep the *mental model* simple even though the backend sums many sources:

```
RUN SCORE  =  BASE POINTS  ×  LIVE MULTIPLIER   (+ end-of-stage bonuses)
```

- **BASE POINTS** — the raw value of things you touch: collectibles, enemies, puzzle completions, secrets.
- **LIVE MULTIPLIER** — a visible multiplier that *grows as you play well and resets/decays when you play poorly*. This is the combo system (Section 4).
- **END-OF-STAGE BONUSES** — flat awards computed once at the finish: time, remaining health, remaining shields, route-difficulty, no-death, completion.

> The player learns one sentence: **"Grab stuff, keep your multiplier high, finish fast and clean."** Everything else is detail they can discover.

## 3. Base Point Sources

| Source | Base value intent | Route scaling |
| --- | --- | --- |
| Standard collectible (common) | Small | Same everywhere |
| Premium collectible (rare) | Large | Mostly on HIGH, some hidden on MIDDLE/secrets |
| Defeat enemy | Small–medium | Higher-value enemies on higher routes |
| Puzzle completion | Medium | Harder puzzles (HIGH) worth more |
| Secret found | Medium–large | Anywhere, often cross-route |
| Near-miss / perfect-dodge | Small, combo-feeding | Any |

**Route scaling** is applied via the collectible/enemy placement, not a hidden tax: HIGH route simply *contains* more and richer things. This keeps it honest.

## 4. The Live Multiplier (combo system)

The multiplier is the moment-to-moment skill expression and the main source of score readability.

### How it grows
The multiplier climbs as the player chains **positive momentum events** without a reset:
- Collecting items in quick succession.
- Defeating enemies.
- Maintaining speed above base (momentum bonus ticks).
- Clean platforming (no hits, no unintended drops).
- Near-misses and stylish traversal (rail grinds, long momentum jumps).

Think of it as a **flow meter**: staying fast, clean, and high keeps it rising.

### How it decays / resets
- **Decays** gradually when the player is slow or idle (standing still bleeds multiplier — reinforces Pillar 2).
- **Partial drop** on a non-lethal mistake (a drop to a lower route): you keep *some* multiplier, so recovery runs still score.
- **Hard reset** on taking damage that breaks the flow (configurable per hazard) or on death-retry at a checkpoint.
- **Combo Keeper** power-up prevents one reset (see `POWERUPS.md`).

### Why this design
It ties the multiplier to the two things we most want to reward — **speed and the high route** — without a hidden formula. The player *sees* the number climb when they do well and dip when they don't. The route choice feeds it naturally: HIGH route has more multiplier fuel and demands the speed that keeps it high.

## 5. Route Difficulty Bonus

Staying high is rewarded structurally, not just via richer pickups:
- A **route-tier modifier** is tracked over the run (how much time/distance spent on HIGH vs MIDDLE vs LOW).
- At stage end this becomes a **Route Difficulty Bonus** and feeds the rank.
- A full-LOW run therefore caps out at a lower rank even if the player collected everything available on LOW — satisfying the brief's requirement that staying low makes the gate "significantly harder."

**Important fairness note:** LOW is still *viable*. A thorough, clean LOW run should reliably clear the *minimum* gate (reach the boss, perhaps at a disadvantage). It just can't reach S/A. See `PROGRESSION.md` thresholds.

## 6. End-of-Stage Bonuses

Computed once at the finish line and shown in the Score Eval breakdown:

| Bonus | Rewards |
| --- | --- |
| **Time Bonus** | Fast completion (scaled against a per-level par time) |
| **Health Bonus** | Remaining health at the finish |
| **Shield Bonus** | Shields still held (also convert to combat armor — `PROGRESSION.md`) |
| **Route Difficulty Bonus** | Time weighted toward higher routes |
| **No-Death / No-Hit Bonus** | Flawless-run awards |
| **Collection Bonus** | % of this run's reachable collectibles grabbed |
| **Secret Bonus** | Secrets discovered |
| **Style Bonus** | Peak multiplier reached, longest momentum chain |

## 7. Readability: how the player always knows their pace

This is the anti-arbitrariness guarantee (Pillar 4). See `UI_AND_PLAYER_FEEDBACK.md` for the HUD spec.

- **Qualification meter** — a persistent on-screen gauge showing current score vs. the boss threshold, with tick marks for rank boundaries (D/C/B/A/S). It fills as you score and visibly *dips* when you drop a route or lose multiplier.
- **Live multiplier readout** — big, animated; the single most-watched number.
- **Score checkpoints** — at junction rooms, the meter shows "pace" feedback: on-track / behind / ahead.
- **Floating score popups** — every point source shows a popup so cause→effect is never mysterious.
- **Pace projection** (optional, late addition): a faint marker on the qualification meter estimating final rank if the player maintains current pace.

## 8. Score Banking (anti-frustration)

To honor the "don't waste 10 minutes" rule:
- At each **junction room**, the current run score is **banked** — locked in as a floor.
- A subsequent death/retry restarts from the junction checkpoint with banked score intact; you can't lose below your last bank.
- Banking also drives **mid-level pace feedback**, so a player who's behind learns it *early* (at junction 1), not at the finish.

See `PROGRESSION.md` for how banking interacts with last-chance challenges and disadvantaged boss attempts.

## 9. Worked Example (illustrative, not final numbers)

A mid-skill run:
```
Base points collected ............ 4,200
Peak live multiplier ............. ×3.1  (dipped twice after drops)
Effective multiplied base ........ ~9,800
+ Time bonus ..................... 1,500   (slightly over par)
+ Health bonus ................... 800     (2 of 3 health)
+ Shield bonus ................... 600     (1 shield held)
+ Route difficulty bonus ......... 1,200   (mostly MIDDLE, some HIGH)
+ No-hit bonus ................... 0       (took one hit)
+ Collection/secret/style ........ 1,100
----------------------------------------
RUN SCORE ........................ ~15,000  → B RANK, qualifies for boss
```
An expert HIGH-route run on the same level might hit ~30,000+ (S rank) via a sustained ×6–8 multiplier, par-beating time, no-hit, and premium collectibles.

## 10. Anti-Exploit Notes (for later tuning)

- **Multiplier farming:** cap how long the multiplier can be sustained while idle-farming a respawning enemy (decay-on-slow handles most of this).
- **Backtracking:** collectibles are one-time per run; no re-grabbing.
- **Degenerate safe loops:** route-difficulty bonus and time par discourage camping a safe spot to pad score.

## 11. Data-Driven Hook

All of the above is tuned via **`ScoreRuleData`** Resources (per difficulty / per level overrides) — see `DATA_MODEL.md`. Point values, multiplier growth/decay curves, bonus weights, and rank thresholds are data, not code, so balance can iterate without engineering.
