# DAMAGE_AND_CHECKPOINTS.md

> **Status: DESIGN PHASE ONLY.**

## 1. Philosophy

Platforming must be **dangerous without constantly stopping play** (Pillar 4 + Pillar 1). The guiding principle:

> **Falling is usually a gameplay event (a drop to a lower route), not death. Death is reserved for clearly-telegraphed lethal hazards and is made cheap by instant retry.**

Two distinct "bad outcomes" exist and must never be confused by the player:
- **DROP** — you lose height, route, pace, and maybe a shield/multiplier, but the run continues.
- **DEATH** — instant retry at the last checkpoint; a clean reset, no slow punishment.

## 2. Player Condition Model

- **Health:** a small pool (target: 3 "pips"). Health is for *soft* hazards and the end-of-stage Health Bonus. Running out of health does **not** instantly end the run — it causes a knock-down/drop and multiplier reset (configurable), except where a hazard is explicitly lethal.

> **Two health models, one character (see I1 in `DESIGN_ANALYSIS.md`).** The game uses **two deliberately different** health representations because the two modes have different needs:
> - **Platforming:** a 3-pip pool (this doc). Pips absorb soft hazards; most real danger is lethal hazards (instant retry) or drops (route demotion), not pip attrition.
> - **Fighting:** a single continuous health bar (`FIGHTING_SYSTEM.md`), standard for a duel.
>
> They are **not** the same number and do not carry over directly. The bridge is the loadout: **remaining platforming health → the Health Bonus (score)** and **rank → the combat bar's starting fill** (`PROGRESSION.md` §4). Shields are the one resource that crosses modes directly (held shields → combat armor). The data split lives in `CharacterData` as separate `platforming_health` and `combat_health` fields (`DATA_MODEL.md`).
- **Shields:** stackable one-hit absorbers (`POWERUPS.md`). A shield intercepts the next damaging/knockdown event *before* health. Shields carry to the fight as armor.
- **Multiplier:** the score combo (`SCORING_SYSTEM.md`); many hazards' real cost is a multiplier hit, not an HP hit.

Order of absorption on a hit: **Invulnerability → Shield → Health → consequence.**

## 3. Hazard Taxonomy

Each hazard declares exactly one *primary consequence*. This is a hard design rule for readability.

| Consequence class | What happens | Example hazards |
| --- | --- | --- |
| **DAMAGE (soft)** | Lose 1 health pip; brief hitstun; multiplier partial drop | Light enemy touch, minor environmental zap |
| **SHIELD-BREAK** | If shielded, consume shield + knockback (no health lost); if unshielded, treat as DAMAGE | Spiky bumpers, charged enemies, electric fields |
| **KNOCK-TO-ROUTE** | Knockback/launch that *moves you down a route* (the designed drop) | Blast jets, heavy enemy slams, collapsing ledges |
| **CHECKPOINT-RESET** | Reset to last checkpoint (NOT a full death; keeps banked score) | Crushers you got pinned by, non-bottomless deadends |
| **INSTANT-DEATH** | Instant retry at last checkpoint | Spikes, lava, saw blades, bottomless pits (where authored), one-shot lasers |

### Design mandate: telegraphing
- INSTANT-DEATH hazards must be **visually unmistakable** (spikes read as spikes; lethal pits are clearly bottomless/dark; lasers hum and flash).
- KNOCK-TO-ROUTE and DROP zones must **not** look lethal — the player should trust that missing here means demotion, not death.
- The difference between "a pit that drops you to LOW" and "a pit that kills you" must be legible at a glance (e.g., you can see a catch-floor below the first, only darkness below the second).

## 4. Falling: the central event

Per `THREE_PATH_LEVEL_DESIGN.md`, most "falls" are **drops**:
- **HIGH miss → lands on MIDDLE.** Keeps most momentum; small multiplier dip; qualification meter visibly drops.
- **MIDDLE miss → lands on LOW.** Keeps some momentum; larger pace cost.
- **LOW miss →** either a **wide safe floor** (just a stumble) **or** a **clearly-marked bottomless pit** (INSTANT-DEATH → checkpoint).

Rules:
1. Every drop zone has an **authored catch-surface** below (no accidental deaths from drops).
2. Drops **preserve momentum** so you land running (Pillar 2).
3. The **only** bottomless/lethal falls are on the LOW route (the floor of the level) or in explicitly telegraphed death gaps — never as the hidden consequence of a normal HIGH/MIDDLE miss.

## 5. Knockback & Route Movement

KNOCK-TO-ROUTE is the active (hazard-caused) sibling of a passive miss:
- A blast jet or heavy enemy can *launch* the player off the HIGH route onto MIDDLE.
- Direction and force are authored so the landing is a real route below, not a death.
- This lets designers create **"stay sharp or get bumped down"** pressure without lethality.

## 6. Checkpoint System

### Two tiers of checkpoint
1. **Micro-checkpoints (death respawn):** frequent, lightweight points the player instant-respawns at after an INSTANT-DEATH or CHECKPOINT-RESET. Density varies by route — **sparse on HIGH, dense on LOW** (part of what makes HIGH harder).
2. **Junction checkpoints (score banking):** at junction rooms; bank score (`SCORING_SYSTEM.md`), re-level momentum, and act as the restart floor for the segment.

### Checkpoint rules
- Instant retry: respawn is immediate (Super Meat Boy feel), with a very short readable reset.
- Respawn restores the segment to a sane state (resets consumed one-shot hazards nearby, keeps banked score, keeps unlocked shortcuts).
- **Which route you respawn on:** you respawn on the route your last checkpoint was on. A HIGH-route death puts you back on HIGH (sparse checkpoints make this costly). A player who dropped to LOW and died respawns on LOW.
- Checkpoints do **not** refill all resources generously — some health/shield restoration is deliberate and tuned so death still has a mild sting without being punishing.

## 7. Recovery

The run is designed to be *saveable* after mistakes:
- **Climb-back opportunities** (springs, wall shafts, launchers, Spring Boots power-up) let a dropped player regain a higher route — at a skill cost.
- **Health/shield pickups** are more common on LOW (recovery lane) so a struggling player can stabilize.
- **Score banking + last-chance challenges** mean even a rough run can still reach the boss (`PROGRESSION.md`).

## 8. Hazard–Route Matrix (authoring guidance)

| Route | Typical consequence mix |
| --- | --- |
| **HIGH** | Lots of INSTANT-DEATH and KNOCK-TO-ROUTE; sparse checkpoints; precision-or-drop |
| **MIDDLE** | Mix of DAMAGE, SHIELD-BREAK, some KNOCK-TO-ROUTE; moderate checkpoints |
| **LOW** | Mostly DAMAGE and the occasional bottomless pit; dense checkpoints; forgiving |

## 9. Hazards as Traversal Tools

Per the puzzle brief, some hazards double as tools (`PUZZLE_DESIGN.md`):
- A bumper that *shield-breaks* you if hit wrong can also *launch* you up a route if approached right.
- A moving saw can be ridden on its mounting arm.
- An enemy can be bounced off (Mario-style) to reach a ledge, feeding the multiplier.

The *same object* reading as both threat and tool, depending on skill, is a core source of depth.

## 10. Data-Driven Hook

Hazards are **`HazardData`** Resources: id, consequence class enum, damage amount, knockback vector/force, telegraph VFX/SFX refs, whether it's a usable traversal tool, and reset behavior. Checkpoints are nodes placed in the level that register with the **Checkpoint Manager** (`GODOT_ARCHITECTURE.md`), carrying a route tag and a "banks score?" flag. See `DATA_MODEL.md`.

## 11. Open Questions

> See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) for the master register. Grep **`❓ OPEN QUESTION`** to find open items.

- ❓ OPEN QUESTION [Q-HEALTH-01] 🔴 **(awaiting user)** Exact health pool size: 3 pips vs. 1-hit-with-shields "Meat Boy" purity. Leaning 3 to support the Health Bonus and soft-hazard texture, but the HIGH route may *feel* 1-hit due to lethal hazard density.
- ~~Should multiplier reset on *every* hit, or only on shield-break/knockdown?~~ **Resolved [Q-DMG-01]:** per-hazard `multiplier_effect` is authoritative — soft DAMAGE = partial dip, knockdown/death = reset; Combo Keeper covers one reset.
- ❓ OPEN QUESTION [Q-DMG-02] Respawn resource restoration curve — needs playtest to avoid both "too punishing" and "no stakes."
