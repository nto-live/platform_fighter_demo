# PROGRESSION.md

> **Status: DESIGN PHASE ONLY.** Thresholds and numbers are tuning targets.

## 1. Two Kinds of Progression

1. **In-level progression** — the score gate, ranks, and the run→fight loadout. Happens every play.
2. **Meta progression** — unlocks that persist across sessions (levels, moves, characters, cosmetics).

Both exist to make *playing well* feel rewarded immediately (loadout) and *long-term* (unlocks + records).

## 2. Ranks

Five ranks evaluated at stage end (`SCORING_SYSTEM.md`):

| Rank | Meaning | Typical route behavior |
| --- | --- | --- |
| **S** | Mastery | Held HIGH route, near-flawless, premium collection, beat par |
| **A** | Excellent | Mostly HIGH/MIDDLE, strong multiplier, few mistakes |
| **B** | Good | Solid MIDDLE run, some HIGH, qualifies comfortably |
| **C** | Passing | Mixed MIDDLE/LOW, meets the gate |
| **D** | Minimum | Mostly LOW; reaches the boss but at a disadvantage |

Rank is derived from `RUN SCORE` against `rank_thresholds` (`DATA_MODEL.md`).

## 3. The Score Gate — a *soft* gate, not PASS/FAIL

The brief warns: do not let a player grind a 10-minute level only to be told they can't fight. Our gate is **layered and forgiving**, built from several of the brief's suggested mechanisms working together:

### Layer 1 — Visible qualification meter (never a surprise)
A persistent gauge (`UI_AND_PLAYER_FEEDBACK.md`) shows current score vs. the boss threshold with rank ticks. The player always knows their pace. There is no end-of-level ambush.

### Layer 2 — Mid-level banking + pace feedback
At each **junction** the score banks and the game tells the player **ahead / on-track / behind**. A struggling player learns at junction 1, not at the finish — and can adjust route choice.

### Layer 3 — The gate is low; rank is the real reward
Reaching the boss requires only the **minimum gate** (roughly a clean LOW/C-D run). Almost any player who finishes qualifies to *attempt* the boss. The *quality* of your run determines your **loadout advantage**, not whether you fight at all.

### Layer 4 — Last-Chance Challenge (if below the minimum)
If a player finishes *below even the minimum gate*, they enter an optional **Last-Chance Challenge**: a short, self-contained score micro-room (a bonus gauntlet) that lets them earn the shortfall. Pass → boss. It's quick, so failure costs seconds, not the whole level.

### Layer 5 — Disadvantaged boss attempt (opt-in)
Rather than a hard wall, a player can choose to **fight anyway at a disadvantage** (the boss gets `disadvantage_scaling` buffs — `DATA_MODEL.md`). This respects player time: you can always *try* the fight.

### Result
There is **no "you wasted 10 minutes, start over" moment.** The worst case is "fight at a disadvantage or do a 20-second last-chance room." This directly satisfies the brief's anti-frustration requirement.

## 4. Run → Fight Loadout (the payoff mapping)

The core of Pillar 5. Built into a `LoadoutPayload` at the handoff (`CORE_GAMEPLAY_LOOP.md`, `DATA_MODEL.md`) and applied to the player in the arena (`FIGHTING_SYSTEM.md`).

| Platforming performance | Combat effect |
| --- | --- |
| **Higher rank / score** | More starting health (scaled by rank: S = full+, D = reduced) |
| **Shields held at finish** | **Armor points** (each absorbs a hit's knockback/chip) |
| **Special collectibles / Overcharge carried** | **Super meter pre-fill** |
| **High-route clear** (held HIGH through a segment) | Unlocks a **bonus attack / EX option** for this fight |
| **Secrets found** | Unlock **alternate move(s)** for this fight |
| **Fast completion (beat par)** | **First-strike / approach-speed** edge at round start |
| **Poor score (D / below)** | **Boss advantage** (`disadvantage_scaling`: more HP / faster / extra armor) |

**Balance rule (restated):** the loadout tilts odds *meaningfully but not deterministically*. A great run makes the fight a victory lap you earned; a bad run makes it hard but winnable with good play.

## 5. Meta Progression (persistent unlocks)

Tracked in `SaveData` / `UnlockManager` (`GODOT_ARCHITECTURE.md`).

| Unlock type | Earned by |
| --- | --- |
| **Next level** | Beating the current boss |
| **Player moves** | Finding secrets, full-high-route clears, S-ranks |
| **Characters** | Milestone achievements (beat all bosses, cumulative S-ranks) |
| **Cosmetics / palettes** | Records, personal bests, bragging-rights feats |
| **Rival / hard boss variants** | Beating a boss at high-rank entry (`BOSS_DESIGN.md`) |

Unlocked moves/characters feed back into both halves — a new character has its own `CharacterData` moveset and movement profile, deepening replay (`REPLAYABILITY.md`).

## 6. Difficulty & Accessibility Progression

- **No forced difficulty select** — the three routes *are* the difficulty, chosen live (Pillar 3).
- **Optional assists** (`DifficultyData`): extra coyote time, infinite dash charges, slower boss, auto-block. Never required; may reduce leaderboard eligibility (`REPLAYABILITY.md`).
- Assists let the lowest skill tier *finish and enjoy* without changing the authored content.

## 7. Record Keeping (the long game)

Per level, `SaveData.records` tracks: best score, best rank, best time, secrets found, and whether a **full high-route clear** was achieved. These drive the "chase your ghost" replay loop and leaderboards (`REPLAYABILITY.md`).

## 8. Progression Pacing (shipping shape, beyond POC)

Rough intended arc (not POC scope):
- Early levels teach one mechanic each; gentle gates; generous loadouts.
- Mid levels combine mechanics; bosses demand the taught counters.
- Late levels assume mastery; S-rank requires near-perfect high-route runs.
- Unlocks (moves/characters) arrive steadily to keep the kit feeling fresh.

## 9. Open Questions

- Exact HP scaling curve by rank (how much does S vs D actually swing?). Needs playtest to stay "meaningful but not deterministic."
- Should the Last-Chance Challenge cost anything (e.g., caps your rank at C)? (Leaning: yes — it gets you *in*, but not a good loadout.)
- Do assists disable unlocks entirely or just leaderboard entries? (Leaning: unlocks still work; leaderboards flagged "assisted.")
- How many unlockable characters for v1 vs. post-launch? (Scope; POC = single character + Volt.)
