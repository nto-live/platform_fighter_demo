# CORE_GAMEPLAY_LOOP.md

> **Status: DESIGN PHASE ONLY.**

## 1. The Macro Loop

```
                 ┌─────────────────────────────────────────────┐
                 │                                             │
   LEVEL SELECT ─┤                                             │
                 ▼                                             │
         [ LEVEL INTRO ]  (boss tease, route map glimpse)      │
                 ▼                                             │
     ┌──> [ PLATFORMING STAGE ] <── instant retry on death ──┐ │
     │           │                                           │ │
     │   navigate 3 interconnected routes                    │ │
     │   solve momentum puzzles                              │ │
     │   collect points / power-ups                          │ │
     │   drop to lower routes on mistakes                    │ │
     │   climb back up where skill allows                    │ │
     │           ▼                                           │ │
     │   mid-level SCORE CHECKPOINTS (bank score) ───────────┘ │
     │           ▼                                             │
     │   [ END-OF-STAGE SCORE EVALUATION ]                     │
     │           ▼                                             │
     │   ┌───────────────┬──────────────────────────────┐     │
     │   │ Met threshold │ Below threshold              │     │
     │   ▼               ▼                              │     │
     │ [ LOADOUT        [ LAST-CHANCE CHALLENGE ] ──fail─┼─────┘ (retry stage)
     │   SUMMARY ]        │ pass                         │
     │   ▼                ▼                              │
     │ [ HANDOFF: platforming → fighting ]               │
     │   ▼                                               │
     │ [ BOSS / FIGHTING STAGE ]                          │
     │   ▼                                               │
     │   ┌──────────┬───────────┐                        │
     │   │   WIN    │   LOSE    │                        │
     │   ▼          ▼           │                        │
     │ [ VICTORY /  [ RETRY FIGHT ] (keep loadout)       │
     │   UNLOCKS ]      or [ REPLAY STAGE for better run ]┘
     │   ▼
     └── NEXT LEVEL / LEVEL SELECT
```

The loop has **two nested cycles**:
- **Micro (platforming):** die → instant retry → learn → improve. Seconds long.
- **Macro (level):** run stage → evaluate → fight → win → next. Minutes long.

The design keeps the micro cycle friction-free (Super Meat Boy instant retry) while making the macro cycle *forgiving at the gate* (no "you wasted 10 minutes" moments — see `PROGRESSION.md`).

## 2. Game States (authoritative list)

These map directly to the **Game State Manager** in `GODOT_ARCHITECTURE.md`.

| State | Purpose | Entered from | Exits to |
| --- | --- | --- | --- |
| `BOOT` | Load config, save data | — | `MAIN_MENU` |
| `MAIN_MENU` | Title, options | `BOOT` | `LEVEL_SELECT` |
| `LEVEL_SELECT` | Choose level, see records | `MAIN_MENU`, post-level | `LEVEL_INTRO` |
| `LEVEL_INTRO` | Short tease: boss silhouette, route hint | `LEVEL_SELECT` | `PLATFORMING` |
| `PLATFORMING` | The main stage; movement controls active | `LEVEL_INTRO`, retry | `SCORE_EVAL`, (self on death-retry) |
| `SCORE_EVAL` | Compute rank, show breakdown | `PLATFORMING` | `LOADOUT`, `LAST_CHANCE` |
| `LAST_CHANCE` | Optional micro-challenge to reach threshold | `SCORE_EVAL` | `LOADOUT` (pass) / `PLATFORMING` (fail-retry) / `LEVEL_SELECT` (bail) |
| `LOADOUT` | Show combat advantages earned; confirm | `SCORE_EVAL`, `LAST_CHANCE` | `HANDOFF` |
| `HANDOFF` | Cinematic/transition; load combat scene | `LOADOUT` | `FIGHTING` |
| `FIGHTING` | The duel; fighting controls active | `HANDOFF`, retry | `VICTORY`, `DEFEAT` |
| `VICTORY` | Win screen, unlocks, record update | `FIGHTING` | `LEVEL_SELECT` |
| `DEFEAT` | Lose screen; retry fight or replay stage | `FIGHTING` | `FIGHTING` (retry) / `PLATFORMING` (replay) / `LEVEL_SELECT` |
| `PAUSE` | Overlay; can be entered from most states | any gameplay | returns to prior state |

> **Design note:** `PLATFORMING` and `FIGHTING` are the only two states with their own full control scheme. The transition between them is deliberately staged through `SCORE_EVAL → LOADOUT → HANDOFF` so the player (a) understands *why* they earned what they earned, and (b) gets a beat to mentally switch from "runner" to "duelist."

## 3. The Handoff (the glue)

The single most important seam in the game. It must make the fight feel *caused* by the run.

Sequence:
1. **Score Eval** — animated score breakdown, rank stamp (S/A/B/C/D). ~4–6s, skippable.
2. **Loadout Summary** — a tight list: "High-route clear → +1 armor. 3 shields banked → Guardian start. S-rank speed → first-strike dash." Each line references a *thing the player did*. ~3–5s.
3. **Handoff transition** — the runner character physically enters the arena; the boss is revealed (if not already). Loadout bonuses visually attach to the player (armor shimmer, meter pre-fill). ~3s.
4. **Fight start.**

The whole handoff is **under ~15 seconds and skippable after first view per level**, so repeat runs don't drag.

See `PROGRESSION.md` for the exact mapping of performance → loadout, and `FIGHTING_SYSTEM.md` for how bonuses express in combat.

## 4. Failure Handling in the Loop

| Failure | Loop response | Rationale (Pillar) |
| --- | --- | --- |
| Missed jump (non-lethal) | Drop to lower route, continue | Pillar 1 — failure changes the run |
| Hit a lethal hazard / bottomless pit | Instant retry at last checkpoint | Pillar 4 — death is cheap, not run-ending at macro scale |
| Below score threshold at stage end | Offer Last-Chance challenge or boss-with-disadvantage; never a hard dead-end | Avoid 10-min frustration |
| Lose the fight | Retry fight with same loadout, or replay stage to improve loadout | Separate "run quality" from "fight execution" |

**Key rule:** the player should never be forced to re-run a long stage *solely* because of a score shortfall discovered at the very end. Mid-level banking + last-chance + disadvantaged boss attempts all exist to honor this.

## 5. Session Shape

A single "sitting" might look like:
- 2–4 minute platforming stage (first-time players slower while exploring).
- ~10–15s handoff.
- 1–2 minute fight.
- Then: replay the stage for a better rank, or advance.

Target: a full level (stage + fight) completes in **well under 10 minutes even on a learning run**, and an expert speedrun of the stage is **under 90 seconds**. This directly addresses the brief's warning about 10-minute levels becoming frustrating — our *baseline* level length is tuned below that ceiling.

## 6. Pacing Intent

- **Platforming:** accelerate — the player should feel faster the more they understand the room.
- **Score eval / loadout:** a deliberate exhale and reveal.
- **Fight:** a focused, tense spike.
- **Victory/unlock:** a dopamine beat that points at the next thing to chase (`REPLAYABILITY.md`).
