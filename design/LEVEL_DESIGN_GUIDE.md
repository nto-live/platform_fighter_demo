# LEVEL_DESIGN_GUIDE.md

> **Status: DESIGN PHASE ONLY.** A practical handbook for authoring large, interconnected, three-route levels.

## 1. What a Level Is

A level = one **large, continuous platforming stage** divided into **segments** by **junction rooms**, followed by a **boss fight**. Conceptually:

```
START ─[ SEGMENT 1 ]─ JUNCTION A ─[ SEGMENT 2 ]─ JUNCTION B ─[ SEGMENT 3 ]─ FINISH → HANDOFF → BOSS
         (3 routes            (bank,              (3 routes             (bank,          (score
          weave)              choose,              weave)               choose)          eval)
```

Each segment contains all three routes (HIGH/MIDDLE/LOW) weaving around each other. Junctions reconnect them, bank score, offer choices, then split again.

## 2. Size & Length Targets

| Metric | Target |
| --- | --- |
| Segments per level | 3 (POC) → 3–5 (shipping) |
| Expert HIGH-route clear | < 90 seconds |
| First-time learning run | 2–4 minutes |
| Full level incl. fight (learning run) | well under 10 minutes |
| Screens/areas spanned | multiple; camera travels continuously |

Keeping the *baseline* well under the 10-minute ceiling is the primary structural defense against the brief's "wasted 10 minutes" frustration (reinforced by banking + last-chance, `PROGRESSION.md`).

## 3. The Three-Route Band

Author levels on a **vertical band** with three horizontal lanes. Rules of thumb:
- **HIGH** sits at the top: narrow, fast, lethal, rich.
- **MIDDLE** is the spine: the default readable path.
- **LOW** is the floor: wide, safe, slow, the recovery lane and the only place bottomless pits live.
- Lanes are **not parallel lines** — they rise, dip, cross, and briefly merge. The *tier identity* (danger/speed/reward) is what stays consistent, not a fixed Y coordinate.

## 4. Interconnection Toolkit (reference)

From `THREE_PATH_LEVEL_DESIGN.md`, the connectors you place:

| Direction | Elements |
| --- | --- |
| **Down (drop)** | Drop points, collapsing ledges, blast-jet knockdowns, missed gaps with authored catch-surfaces below |
| **Up (climb)** | Springs, launchers, wall-jump shafts, secret elevators, grind rails, Spring Boots pickups |
| **Lateral/secret** | Hidden tunnels, destructible walls, timed doors, moving platforms |
| **Logic** | Switches/mechanisms that open a connection on another route |

**Placement discipline:**
- Every stretch where the player can **drop** has a climb-back within reasonable distance.
- Every **climb** is harder than simply having stayed up — recovery costs skill.
- Every visible-but-unreachable reward (`see-but-can't-reach`) has a *real* access route, however hard.

## 5. The Segment Authoring Template

For **each segment**, fill this out **per route**. (This is the exact template used in `PROOF_OF_CONCEPT_LEVEL.md`.)

```
SEGMENT N — [name/theme]

HIGH PATH:
  Challenge:            (the main traversal demand)
  Hazards:              (what threatens the player; consequence class)
  Reward:               (collectibles / power-ups / multiplier fuel)
  Score opportunity:    (how pace is gained)
  Failure destination:  (which route you drop to)

MIDDLE PATH:
  Challenge / Hazards / Reward / Score opportunity / Failure destination

LOW PATH:
  Challenge / Hazards / Reward / Score opportunity / Recovery opportunity
```

Plus, per segment:
- **Signature puzzle** (one memorable set-piece; pick a pattern from `PUZZLE_DESIGN.md`).
- **At least one climb-back** opportunity.
- **Reconnection** into the next junction.

## 6. Pacing a Segment (micro-rhythm)

A good segment has an internal rhythm:
```
[ setup / breather ] → [ build / rising demand ] → [ signature challenge ] → [ release into junction ]
```
- Open each segment with a readable "here's the theme" beat.
- Escalate demand toward the signature puzzle/jump.
- Release into the junction so the player exhales and chooses.

Avoid flat difficulty; avoid unbroken max-intensity (it exhausts and muddies readability).

## 7. Teaching & Testing (and feeding the boss)

- **Introduce each mechanic safely** the first time it appears (no lethal cost to learn it).
- **Combine mechanics later** under time/precision pressure.
- **Rhyme the level's vocabulary with its boss** (`BOSS_DESIGN.md`): a momentum/dash/rail level → a rushdown boss (Volt). The stage is covert practice for the fight.

## 8. Junction Room Checklist

Each junction should:
- [ ] Reconnect all three routes into a shared space.
- [ ] **Bank score** (score checkpoint) and show **pace feedback** (ahead/on-track/behind).
- [ ] Act as the **segment restart floor** for death-retry.
- [ ] Offer a **decision** (which route to commit to next) and often a **forced-choice power-up** (`POWERUPS.md`).
- [ ] Re-level momentum so the next segment sets its own speed demands.
- [ ] Optionally host a **route-change puzzle** (open the HIGH entrance before a timer).

## 9. Readability & Honesty Checklist (per the pillars)

- [ ] Lethal hazards look lethal; drop zones look survivable (`DAMAGE_AND_CHECKPOINTS.md`).
- [ ] Route tier is visually legible (color/altitude/decoration language).
- [ ] Every visible reward is actually reachable by *some* route.
- [ ] The qualification meter can be satisfied by a clean run on at least MIDDLE; LOW alone reaches only the minimum gate (`SCORING_SYSTEM.md` / `PROGRESSION.md`).
- [ ] No single mandatory puzzle hard-blocks all three routes.

## 10. Collectible & Power-Up Distribution

| Route | Collectibles | Power-ups |
| --- | --- | --- |
| HIGH | Premium, dense, high-multiplier fuel | Rare/greedy (3× Score, Overcharge, Momentum Lock) |
| MIDDLE | Standard, steady | Balanced; forced-choice pedestals |
| LOW | Sparse, low-value | Safety-focused, plentiful (Shields, Spring Boots) |
| Secrets | The best single items | The rarest items (Invulnerability, Ghost Rail) |

Distribution should make the *route choice* a scoring decision, not just a difficulty decision.

## 11. Level Construction in Godot (design-time guidance)

Detailed in `GODOT_ARCHITECTURE.md`; summary for level authors:
- A level is a **scene** composed of reusable component scenes: platforms, hazards, interactables (switches/doors/platforms), collectibles, power-up pickups, checkpoints, route-tag volumes, and the finish trigger.
- **Route-tag volumes** mark which tier a region belongs to (feeds the Route Manager and the route-difficulty score bonus).
- **TileMaps** for static geometry; **instanced component scenes** for anything interactive or scorable.
- Level-wide tuning (par time, score thresholds, boss ref, theme) lives in a **`LevelData`** Resource attached to the level (`DATA_MODEL.md`), so no level logic is hard-coded.
- Prefer **composition over inheritance**: a timed door is a door scene + a timer component + a signal, not a bespoke script per level.

## 12. Authoring Workflow (recommended)

1. **Block out the band**: rough the three lanes and junctions as grey boxes; validate the macro flow and length targets first.
2. **Thread connectors**: place drops, climbs, and secrets; verify every drop has a climb-back and every reward is reachable.
3. **Fill the segment template** per route; place the signature puzzle.
4. **Distribute** collectibles/power-ups per Section 10.
5. **Set `LevelData`**: par time, thresholds, boss ref.
6. **Playtest the three personas** (beginner-LOW, intermediate-mix, expert-HIGH) against the pillars.
7. **Tune** via data (`ScoreRuleData`, `LevelData`), not code.

## 13. Common Pitfalls

- ❌ Three parallel corridors that never interact (defeats the whole concept).
- ❌ Drops that feel like deaths (unauthored or lethal-looking catch zones).
- ❌ A HIGH route that's just MIDDLE-with-spikes (it must be *faster* and *richer*, not merely meaner).
- ❌ A LOW route that feels like a punishment corridor (it must be a genuine, pleasant recovery lane).
- ❌ Mandatory comprehension gates that stop all forward progress.
- ❌ Difficulty spikes with no breather; unreadable hazard soup.
