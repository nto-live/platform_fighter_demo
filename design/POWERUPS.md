# POWERUPS.md

> **Status: DESIGN PHASE ONLY.** All durations/values are tuning targets.

## 1. Role of Power-Ups

Power-ups modify **traversal** and/or **scoring**, and some carry into the fight as **loadout bonuses** (`PROGRESSION.md`). They are a primary tool for the risk/reward tension that defines route choice: the best power-ups live on the HIGH route and in secrets.

Design rules:
- **Readable:** each power-up has a distinct silhouette, color, pickup sound, and active-state visual on the player.
- **Honest timers:** a visible countdown; a warning flash before expiry.
- **Non-stacking confusion:** clear rules for what happens when a second power-up is grabbed (Section 5).
- **Choice is the point:** the signature moments are *forced choices* between safety and score (Section 4).

## 2. Core Power-Up Catalog

### Defensive
| Power-up | Effect | Carries to fight? |
| --- | --- | --- |
| **Shield** | Absorbs one hit (then breaks instead of damaging you / knocking you down a route) | Yes → armor point |
| **Double Shield** | Absorbs two hits | Yes → 2 armor points |
| **Invulnerability** (rare) | Brief total immunity; blow through hazards | No (consumed in stage) |

### Offensive / Traversal
| Power-up | Effect | Carries to fight? |
| --- | --- | --- |
| **Speed Shoes** | Temporarily raises max speed (and boosted ceiling) | Partial → speed-start edge if active near finish |
| **Double Jump** | Temporary extra mid-air jump (not a base ability — see `MOVEMENT_SYSTEM.md`) | No |
| **Dash Charge** | Grants extra dash charges for a duration | No |

### Scoring
| Power-up | Effect | Carries to fight? |
| --- | --- | --- |
| **Score Multiplier (2×/3×)** | Multiplies **base points only** for a duration, *before* the live multiplier applies (see §9) | Indirectly (higher score → better loadout) |
| **Magnet** | Pulls nearby collectibles toward the player | No |
| **Combo Keeper** | Prevents the next multiplier reset (one-shot buffer) | No |

### Utility
| Power-up | Effect | Carries to fight? |
| --- | --- | --- |
| **Slow Motion** | Temporarily slows hazards/environment (player moves at normal perceived speed) | No |

## 3. Additional Power-Ups (new designs)

Designed to deepen the risk/reward and route identity:

| Power-up | Effect | Design intent |
| --- | --- | --- |
| **Phase Dash** | Next dash passes through a thin destructible wall / one hazard layer | Unlocks hidden cross-route connections; puzzle tool |
| **Momentum Lock** | Freezes boosted speed from decaying for a duration | Rewards building speed then spending it on a long HIGH stretch |
| **Spring Boots** | Landing from a fall auto-bounces you upward once | Turns a drop into a climb — a *recovery* power-up that supports Pillar 1 |
| **Ghost Rail** | Spawns a temporary grind rail along a set path | Creates a climb-back / shortcut on demand |
| **Score Shield** | Protects *banked multiplier* instead of health on next hit | Pure score-vs-safety wrinkle |
| **Overcharge** (rare) | Fills a chunk of the fight's super meter *now*, but you must carry it to the finish without dying | High-risk payoff that explicitly links stage play to the fight |
| **Decoy** | Drops a decoy that distracts/absorbs one enemy or hazard trigger | Traversal problem-solver, especially on MIDDLE |

## 4. The Forced Choice (signature mechanic)

At key points — especially **junction rooms** and HIGH-route gambles — the player meets a **choice pedestal / fork** offering two power-ups where **you can take only one**:

```
        ┌─────────────┐        ┌─────────────┐
        │   SHIELD    │   OR   │  2× SCORE   │
        │ (safety)    │        │ (greed)     │
        └─────────────┘        └─────────────┘
```

Grabbing one consumes/locks the other. Example tensions:
- **Shield vs 2× Score** — survive the next hazard, or gamble for pace.
- **Double Jump vs Score Multiplier** — easier traversal, or more points.
- **Invulnerability vs Overcharge** — safety now, or a combat edge later.
- **Speed Shoes vs Magnet** — go faster (risky), or vacuum points (safe).

This is where route philosophy and scoring collide: the *greedy* pick usually assumes you're skilled enough to not need the *safe* pick.

## 5. Stacking & Replacement Rules

- **Same type:** refreshes/extends duration (e.g., grabbing Speed Shoes again resets its timer).
- **Different types, non-conflicting:** stack (Magnet + Score Multiplier is fine).
- **Different types, conflicting (rare):** the newer replaces the older with a clear UI swap cue (e.g., Slow Motion while Speed Shoes active — define per pair during tuning).
- **Shields are state, not timed:** they persist until consumed; multiple shields stack as armor count.
- **Invulnerability overrides** damage from all non-instant-death sources for its duration.

## 6. Spawn & Placement Philosophy

| Route | Power-up profile |
| --- | --- |
| **HIGH** | Rare, powerful, score-focused (3× Score, Overcharge, Momentum Lock); few safety nets |
| **MIDDLE** | Balanced mix; the forced-choice pedestals mostly live here and at junctions |
| **LOW** | Safety-focused and plentiful (Shields, Spring Boots, Slow Motion); keeps beginners alive |
| **Secrets** | The rarest items (Invulnerability, Ghost Rail, Overcharge) — exploration payoff |

This placement reinforces the whole structure: safety trends downward, greed/power trends upward.

## 7. Carry-to-Fight Summary

Only a subset converts into combat loadout (full mapping in `PROGRESSION.md`):
- **Shields held at finish → armor points** in the fight.
- **Overcharge carried to finish → pre-filled super meter.**
- **Speed Shoes active near finish → first-strike/approach-speed edge.**
- Score-based power-ups contribute **indirectly** by raising final score → better rank → better loadout.

## 8. Data-Driven Hook

Every power-up is a **`PowerUpData`** Resource: id, display name, icon, pickup/active VFX+SFX refs, effect type enum, magnitude, duration, stack rule, carries-to-fight flag, rarity, and allowed-route tags. New power-ups are authored as data, not code (`DATA_MODEL.md`). The **Power-Up Manager** (`GODOT_ARCHITECTURE.md`) reads these and applies timed modifiers to the player state.

## 9. Open Questions

> See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) for the master register. Grep **`❓ OPEN QUESTION`**.

- ~~Should Score Multiplier power-ups multiply base points or the whole product?~~ **Resolved [Q-PUP-01]:** base points only, applied before the live multiplier (`SCORING_SYSTEM.md` §2). Exact magnitudes still to tune [Q-PUP-04].
- ❓ OPEN QUESTION [Q-PUP-02] How many forced-choice pedestals per level before it feels gimmicky? (Leaning: 2–3, usually at junctions.)
- ❓ OPEN QUESTION [Q-PUP-03] Does Overcharge risk (lose it on death) feel exciting or punishing? Needs playtest.
