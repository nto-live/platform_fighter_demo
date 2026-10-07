# PLATFORMING_COMBAT.md

> **Status: DESIGN PHASE ONLY.** Fills gap G1 from `DESIGN_ANALYSIS.md`: how the player interacts with enemies *during* the platforming stage (distinct from the duel in `FIGHTING_SYSTEM.md`).

## 1. Philosophy

Platforming enemies are **traversal elements, not a combat minigame.** They exist to shape routes, feed the multiplier, and act as hazards-or-tools (`DAMAGE_AND_CHECKPOINTS.md` §9). There is **no dedicated attack button in platforming mode** — the player defeats enemies *through movement* (Pillar 2). This keeps the platforming kit clean and preserves the hard mode-switch into the duel.

> **Rule:** if an interaction with an enemy can't be expressed with the existing movement kit (jump, dash, slide, momentum, bounce, rail), it doesn't belong in platforming.

## 2. How Enemies Are Defeated (movement-only)

| Method | Verb used | Result |
| --- | --- | --- |
| **Stomp / bounce** | Jump onto a vulnerable top | Enemy defeated; player bounces (preserves/redirects momentum → `P2 Bounce Sequence`) |
| **Dash-through** | Dash into a dash-vulnerable enemy | Enemy defeated; dash continues (gap-closer + kill) |
| **Slide-through** | Slide into a low enemy | Enemy knocked out; momentum preserved |
| **Rail / momentum impact** | Hit an enemy while boosted on a rail or above a speed threshold | Enemy defeated; style points |
| **Environmental** | Lure into a hazard, crusher, or pit | Enemy removed via the world (puzzle flavor) |

Defeating an enemy is a **scoring event** (`SCORING_SYSTEM.md` base points + multiplier feed) and often a **momentum event** (a bounce is both a kill and a jump).

## 3. Enemy Archetypes (data-driven)

Enemies are **traversal fixtures with a vulnerability profile**, not full combatants. Each is defined by *how* it can be defeated and *what* it threatens.

| Archetype | Threat (consequence class) | Vulnerable to | Role |
| --- | --- | --- | --- |
| **Walker** | DAMAGE on contact | Stomp, dash, slide | Basic timing obstacle |
| **Spiker** | SHIELD-BREAK (spiked top) | Dash/slide only (never stomp) | Teaches "read the vulnerability" |
| **Charger** | KNOCK-TO-ROUTE (rams you) | Stomp or dash when winding up | Can bump you down a route |
| **Floater / flyer** | DAMAGE | Jump-stomp mid-air, dash | Air-route obstacle; bounce chain fodder |
| **Turret / lobber** | DAMAGE (projectile) | Can't be killed easily — avoid/time it | Zoner flavor; feeds timing puzzles (`P5`) |
| **Bumper-bot** | SHIELD-BREAK or launch | Not killed — *used* as a bounce/launch tool | Pure hazard-as-tool (`P9`) |
| **Armored** | DAMAGE; shrugs off stomps | Only momentum/dash at boosted speed | Rewards building speed (`P10`) |

Vulnerability profiles are the key design knob: a Spiker punishes a lazy stomp; an Armored enemy *requires* boosted speed, so it gates casual kills behind momentum.

## 4. Enemies and the Three Routes

| Route | Enemy use |
| --- | --- |
| **HIGH** | Sparse but high-value; often moving fast or used as bounce-chain stepping stones; defeating them feeds a big multiplier |
| **MIDDLE** | Moderate density; standard archetypes; readable timing |
| **LOW** | Few, low-value, slow; rarely a real threat (recovery lane) |

Enemies never *block* a route outright — consistent with "no mandatory gate stops progress" (`PUZZLE_DESIGN.md` §5). They raise the *cost* or *reward* of a line, not the possibility of continuing.

## 5. Enemies as Puzzle & Momentum Elements

- **Bounce chains** (`P2`): a line of stompable enemies becomes a climb or a long aerial traversal.
- **Hazard-as-tool** (`P9`): bumper-bots launch you up a route; chargers can be baited into breaking a destructible wall.
- **Timing** (`P5`): turrets/lobbers create the projectile-timing windows that teach the counter-skill for a zoner boss (`BOSS_DESIGN.md` teaching contract).

## 6. Damage & Interaction Rules

Enemies obey the same consequence model as hazards (`DAMAGE_AND_CHECKPOINTS.md`):
- Contact resolves through **Invulnerability → Shield → Health → consequence**.
- An enemy's `consequence` class (DAMAGE / SHIELD-BREAK / KNOCK-TO-ROUTE) is authored per archetype.
- Defeating an enemy via the correct movement verb is **always safe** (the stomp that kills a Walker doesn't also damage you); mis-reading (stomping a Spiker) triggers the enemy's consequence instead.

## 7. Scoring Interaction

- Each defeat grants base points scaled by archetype and route (`SCORING_SYSTEM.md` §3).
- Defeats are **multiplier-feeding events** (chaining kills without slowing keeps the flow meter climbing).
- A **bounce chain** (multiple enemies without touching ground) is a dedicated style source feeding the Style Bonus.

## 8. Data-Driven Hook

Enemies are a reusable component scene parameterized by an **`EnemyData`** Resource (new; add to `DATA_MODEL.md`):
```
EnemyData
├─ id / display_name
├─ archetype_tag       (WALKER | SPIKER | CHARGER | FLOATER | TURRET | BUMPER | ARMORED)
├─ consequence         (reuses HazardData consequence enum: DAMAGE | SHIELD_BREAK | KNOCK_TO_ROUTE)
├─ vulnerable_to       flags {STOMP, DASH, SLIDE, MOMENTUM_IMPACT, ENVIRONMENT}
├─ requires_boosted    : bool        # e.g. ARMORED true
├─ base_score          : int
├─ route_tags          flags {HIGH, MIDDLE, LOW}
├─ movement_pattern_ref (patrol / charge / fly / stationary)
├─ is_traversal_tool    : bool       # bumper-bots etc.
└─ vfx/sfx refs
```
Behavior comes from the archetype + pattern, consumed by a generic enemy scene — no per-enemy bespoke scripts (consistent with `GODOT_ARCHITECTURE.md`).

## 9. Explicitly Out of Scope (platforming)

- No health bars on enemies, no combo meter against enemies, no lock-on, no ranged player attack. Those belong to the duel (`FIGHTING_SYSTEM.md`), which is deliberately a *separate mode*.
- Bosses are **never** platforming enemies — they are the duel (`BOSS_DESIGN.md`).

## 10. Open Questions

> See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) for the master register. Grep **`❓ OPEN QUESTION`**.

- ❓ OPEN QUESTION [Q-PCMB-01] Do any enemies drop collectibles/power-ups on defeat, or is loot purely placed? (Leaning: a few archetypes drop; most placed.)
- ❓ OPEN QUESTION Should a bounce chain have a visible on-screen counter (like the multiplier) to encourage it? (Leaning: yes, subtle.)
- ❓ OPEN QUESTION [Q-PCMB-02] Are turrets/lobbers ever defeatable (reflect), or strictly avoid-only? (Leaning: avoid-only in demo; revisit for a reflect-themed boss.)
