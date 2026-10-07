# MOVEMENT_SYSTEM.md

> **Status: DESIGN PHASE ONLY.** Numbers below are *design targets* for tuning, not final values.

## 1. Philosophy

Movement is the core verb (Pillar 2). The model must be:
- **Easy to understand** — a new player can run and jump in seconds.
- **Momentum-driven** — speed is a resource you build, keep, convert, and lose.
- **High skill ceiling** — experts chain mechanics to hold the high route and set records.
- **Honest** — the character goes where physics says; no hidden auto-assist that experts can't predict.

The feel target sits between **Super Meat Boy's precision** (tight, responsive, instant) and **Sonic's momentum** (speed accumulates and matters), leaning toward *grounded precision with a momentum layer* rather than slippery high-speed auto-run.

## 2. The Movement Model (two-layer)

### Layer A — Core control (always on, easy)
- **Run** with analog/directional input. Snappy acceleration to a comfortable *base top speed*.
- **Variable-height jump** — tap for short hop, hold for full height.
- **Air control** — strong but not total; you can adjust an arc, not fully reverse a committed leap.

This layer alone is a complete, playable precision platformer. A beginner never *needs* Layer B to clear the low route.

### Layer B — Momentum & expression (skill ceiling)
Mechanics that let a player exceed base top speed, redirect momentum, and reach higher routes:
- **Dash** (ground & air), limited charges, refunded on landing / on specific pickups.
- **Slide / crouch-slide** — convert height/speed into a low, fast burst; preserves momentum under low ceilings and down slopes.
- **Wall jump** and **wall slide** — vertical recovery and route-climbing.
- **Momentum jump** — jumping while at or above a speed threshold converts horizontal speed into extra jump distance/height ("you keep what you earned").
- **Slope & ramp acceleration** — downhill builds speed above base top speed; uphill bleeds it.
- **Bounce / spring / launcher** surfaces — world-driven momentum injection.
- **Grind rails** — ride to carry/convert momentum along a set path (great for high route).
- **Speed boosters / conveyors** — environmental speed modifiers.

## 3. Recommended Mechanic Set (what we actually build)

The brief lists many options and asks us to recommend. **Recommendation: ship a focused kit and resist adding the rest until the core proves out.**

### Core kit (POC + v1) — build these first
| Mechanic | Why it earns its slot |
| --- | --- |
| Run + variable jump + air control | Non-negotiable foundation |
| **Dash (charged)** | The expressive verb; gap-closer, route-climber, puzzle tool |
| **Wall jump / wall slide** | Primary vertical recovery and climb-back mechanic (serves Pillar 1 & 3) |
| **Slide** | Momentum preservation + low-route traversal + puzzle (slide under timed door) |
| **Momentum jump** (speed→distance) | Makes speed *mean something* for platforming, not just scoring |
| **Springs / launchers** | World-driven route connectors (serves interconnection) |
| **Grind rails** | Signature high-route fantasy; converts/sustains momentum |

### Deferred (add only if the core proves fun and we need more texture)
Wall-running, slow-mo as a movement verb, double-jump as a *base* ability (it exists as a temporary power-up instead — see `POWERUPS.md`), conveyor puzzles beyond simple cases.

### Rejected for the base kit (reasons)
- **Permanent double jump as a core ability:** flattens the precision of single-jump arcs and undercuts wall-jump identity. Kept as a power-up only.
- **Full air reversal:** removes commitment, which is where precision tension lives.

> **Rationale:** every mechanic is a thing the player must learn, the level designer must account for across three routes, and we must tune. A tight kit that interacts richly beats a long list that interacts shallowly.

## 4. Momentum as a Resource — the rules

The whole system hinges on these:

1. **Base top speed** is the comfortable running cap. Reaching it is trivial.
2. **Boosted speed** (above base) is only reached via momentum sources: downhill slopes, dashes, springs, rails, speed boosters, chained momentum jumps.
3. **Boosted speed decays** over flat ground back toward base, but **slowly enough to be spent** on a jump, a gap, or a puzzle window.
4. **Momentum converts between axes:** a momentum jump trades horizontal speed for arc; a steep landing + slide trades vertical for horizontal.
5. **You keep what you earn until you waste it.** Hitting a wall, over-braking, or a bad landing bleeds momentum. This is the skill expression: *routing to never lose speed.*
6. **High routes assume boosted speed.** Many high-route gaps are *only* clearable while boosted, which is why the high route is both faster and harder (Pillar 3).

## 5. Design Targets (for later tuning — not final)

These exist to give the eventual implementation a starting point and to let us reason about level geometry now.

| Parameter | Target intent |
| --- | --- |
| Ground accel to base | Fast (~0.15–0.25s) — feels responsive |
| Base top speed | The "comfortable" reference unit = `1.0 v` |
| Boosted ceiling | ~`1.6–2.0 v` from chained sources |
| Boost decay on flat | Gentle — losing boost should take a couple seconds, not instant |
| Jump: tap vs hold | Short hop ~40% of full height |
| Coyote time | ~5–7 frames (forgiveness after leaving a ledge) |
| Jump buffer | ~5–7 frames (press slightly before landing still jumps) |
| Dash | Fixed-distance burst; 1–2 charges; refund on land |
| Wall-jump kick | Fixed horizontal + vertical impulse; brief air-control lockout to make it readable |

**Forgiveness features (coyote time, jump buffer, generous hitbox-vs-visual) are mandatory** — they're what make precision feel fair instead of cheap, straight out of the Super Meat Boy playbook.

## 6. Movement State Machine (conceptual)

Implemented as the **Platforming State Machine** in `GODOT_ARCHITECTURE.md`. States, not code:

```
GROUNDED ──run/idle──┐
   │  jump            │ land
   ▼                  │
AIRBORNE ─────────────┘
   │  touch wall
   ▼
WALL_SLIDE ──wall-jump──> AIRBORNE
   │
DASH (ground or air, time-boxed) ──> back to GROUNDED/AIRBORNE
SLIDE (grounded, crouch+speed) ──> GROUNDED
RAIL (on grind rail) ──> AIRBORNE on exit
LAUNCHED (spring/launcher) ──> AIRBORNE
HITSTUN (took damage) ──> brief, then AIRBORNE/GROUNDED
DEAD ──> retry
```

Each state defines: allowed inputs, how momentum carries in/out, and which hazards/surfaces it interacts with. Transitions preserve momentum unless a rule in Section 4 bleeds it.

## 7. How Movement Serves the Three Routes

| Route | Movement demand |
| --- | --- |
| **High** | Sustain boosted speed; chain dash→momentum-jump→rail; tiny landing zones; wall-jump climbs; near-zero margin |
| **Middle** | Base speed sufficient; occasional dash or wall-jump; readable gaps |
| **Low** | Core control only; no boosted speed required; wide landings; springs do the hard work for you |

A missed high-route momentum jump doesn't kill — it drops you (Pillar 1). The movement system and the level geometry are co-designed so that *the distance you fall short* determines *which lower route you land on* (see `THREE_PATH_LEVEL_DESIGN.md`).

## 8. Open Movement Questions (for later)

- Should dash charges be shared with the fight, or is dash purely a platforming verb? (Leaning: platforming-only; fight has its own dodge.)
- Does boosted speed persist through a spring, or does the spring overwrite it with a fixed launch? (Leaning: spring sets a floor, doesn't cap — so arriving fast still helps.)
- Grind-rail control model: auto-speed vs. player-modulated? (Leaning: player can lean forward/back to modulate within a band.)
