# PROOF_OF_CONCEPT_LEVEL.md

> **Status: DESIGN PHASE ONLY. This describes a level; it does NOT implement one.** No scenes, nodes, or assets are created.

## 0. Purpose

One complete example level — **"Voltline Rush"** — designed in full to demonstrate every core system working together, and to serve as the blueprint for the eventual proof-of-concept build. It demonstrates:

- Three difficulty paths (HIGH / MIDDLE / LOW)
- Falling between routes (drops) + at least one climb-back
- Momentum, precision platforming, movement puzzles
- Score collectibles + score-multiplier opportunities
- Shields + Speed Shoes
- Checkpoints (micro + junction banking)
- Secrets + at least one major shortcut
- Qualification meter in action
- Boss transition (handoff) into the fighting arena vs. **VOLT** (`BOSS_DESIGN.md`)

## 1. Theme & Boss Rhyme

- **Theme:** a neon "speedway" — rails, springs, boost pads, electric hazards. Visually fast.
- **Boss:** **VOLT, the Momentum Rival** (rushdown; weakness = whiff-punish).
- **Teaching contract (`BOSS_DESIGN.md`):** the level drills **momentum, dashing, rails, and reading over-committed speed** — exactly the skills needed to bait and punish Volt's dash-heavy offense.

## 2. Macro Layout

```
START ─[ SEGMENT 1: The On-Ramp ]─ JUNCTION A ─[ SEGMENT 2: The Rail Yard ]─ JUNCTION B ─[ SEGMENT 3: The Overpass ]─ FINISH → HANDOFF → VOLT
          (teach momentum)            (bank+choose)    (test momentum+rails)       (bank+choose)   (combine everything)
```

- **3 segments, 2 junctions**, per `LEVEL_DESIGN_GUIDE.md`.
- Targets: expert HIGH clear **< 90s**; first-time learning run **2–4 min**; full level incl. fight **well under 10 min**.
- Each segment weaves all three routes; junctions reconnect, bank score, offer a forced choice, then split again.

## 3. Legend

- **◆** premium collectible (high value) · **•** standard collectible
- **🛡** Shield · **⚡** Speed Shoes · **2×** Score Multiplier pedestal
- **▲** spring · **»** boost pad · **≈** grind rail · **⌁** electric hazard (SHIELD-BREAK)
- **☠** instant-death hazard · **▼** drop point (to route below) · **↑** climb-back
- **✦** secret · **⚑** checkpoint (micro) · **🏦** junction checkpoint (banks score)

---

## SEGMENT 1 — "The On-Ramp" (teach momentum)

**Intent:** safely introduce running, variable jump, dash, boost pads, and the first drop. Low lethality; this is where the player learns the verbs. Signature puzzle: **P10 Momentum Bank-and-Spend** (a run-up that spends speed on a long gap).

```
HIGH   •◆•   » ─── gap ─── ◆    ⌁    ▼(to MID)        ✦(hidden alcove, needs dash)
       ────────────────────────────────────────
MID    •   » ── ⚑ ── small gap ── •   ▼(to LOW)        2×/🛡 choice near end
       ────────────────────────────────────────
LOW    ⚑ ── wide floor ── ▲(↑ to MID) ── • ── ⚑        (recovery lane)
```

### HIGH PATH
- **Challenge:** build speed on the boost pad `»`, then a **momentum-spend long jump** over a wide gap landing on a small ledge. Dash mid-air to extend. Requires boosted speed (`MOVEMENT_SYSTEM.md`).
- **Hazards:** one electric field `⌁` (SHIELD-BREAK) just after the landing — punishes overshoot.
- **Reward:** two **◆** premium collectibles; a **✦ secret** alcove reachable only with a well-timed dash (teaches dash as a tool → `PUZZLE_DESIGN.md` P8).
- **Score opportunity:** premium collectibles + sustained speed feeding the multiplier.
- **Failure destination:** miss the long jump → **▼ drop to MIDDLE** (authored catch-surface, keeps momentum).

### MIDDLE PATH
- **Challenge:** moderate boost-and-jump over a *smaller* gap; readable, forgiving.
- **Hazards:** none lethal; a gentle paced jump. Micro-checkpoint `⚑` early.
- **Reward:** standard **•** collectibles; end-of-segment **forced choice pedestal — 🛡 Shield OR 2× Score** (`POWERUPS.md`). First real risk/reward decision.
- **Score opportunity:** steady collectibles; the 2× choice for greedy players.
- **Failure destination:** miss the gap → **▼ drop to LOW**.

### LOW PATH
- **Challenge:** flat, wide floor; trivial jumps. The teaching-wheels lane.
- **Hazards:** essentially none; dense checkpoints `⚑`.
- **Reward:** sparse **•** collectibles; a plentiful-safety vibe.
- **Score opportunity:** low — enough to keep pace *barely* if played cleanly.
- **Recovery opportunity:** a **spring ▲** mid-segment lets a dropped player **↑ climb back to MIDDLE** — the first taught climb-back.

**Reconnection:** all three routes funnel toward **JUNCTION A**.

---

## JUNCTION A 🏦

- **Reconnects** all three routes into a shared room.
- **Banks score** (`SCORING_SYSTEM.md`) and shows pace: *"BANKED — ON TRACK FOR B."*
- **Decision point:** three clearly-signposted entrances to Segment 2's routes (color-coded HIGH/MIDDLE/LOW).
- **Forced-choice power-up:** **⚡ Speed Shoes OR 🛡 Shield** — speed (greedy, helps HIGH) vs. safety.
- **Route-change puzzle (optional climb, `P7`):** a switch here opens the HIGH entrance for a few seconds — hit it, then sprint to make the HIGH entrance before it closes (`P5 Beat-the-Door` flavor). Miss it → you take MIDDLE.
- Re-levels momentum before the split.

---

## SEGMENT 2 — "The Rail Yard" (test momentum + rails)

**Intent:** the showcase segment. Grind rails `≈`, springs `▲`, a timed door, and the level's **major shortcut**. Signature puzzle: **P3 Platform Redirection + P5 Beat-the-Door** on HIGH; **P2 Bounce Sequence** as a climb-back on LOW.

```
HIGH   ≈≈≈≈ rail ─ ▼(to MID) ─ timed door ]═ ◆◆ ─ ⌁ ─ MAJOR SHORTCUT »»» (skips to Jct B)
       ──────────────────────────────────────────────
MID    •• ─ moving platform (redirect) ─ ⚑ ─ • ─ ▼(to LOW) ─ ✦(destructible wall)
       ──────────────────────────────────────────────
LOW    ⚑ ─ ▲▲ bounce sequence (↑ to MID) ─ wide floor ─ ☠ pit (telegraphed) ─ ⚑
```

### HIGH PATH
- **Challenge:** mount a **grind rail ≈** carrying boosted speed, dismount into a **timed door** you must dash through before it shuts (`P5`), then collect `◆◆` past an electric field `⌁`. Ends in the **MAJOR SHORTCUT** `»»»` — a boost-pad cannon that launches you straight to **Junction B**, skipping the rest of the segment.
- **Hazards:** `⌁` SHIELD-BREAK after the door; falling off the rail.
- **Reward:** two **◆** premiums; the major shortcut (huge time save → Time Bonus + style).
- **Score opportunity:** highest in the level — rail-grind style points, premiums, speed multiplier, time save.
- **Failure destination:** fall off rail or miss the door → **▼ drop to MIDDLE** (not death).

### MIDDLE PATH
- **Challenge:** **redirect a moving platform** (`P3`) by hitting a switch, then *beat it to the far ledge* using your own speed. Micro-checkpoint `⚑`.
- **Hazards:** gaps between platform positions; mistime the redirect and you drop.
- **Reward:** standard collectibles; a **✦ secret** behind a **destructible wall** (needs a dash or Phase Dash → cross-route tunnel that can pop you up toward HIGH).
- **Score opportunity:** puzzle-completion points + secret.
- **Failure destination:** mistimed platform → **▼ drop to LOW**.

### LOW PATH
- **Challenge:** a **bounce sequence** (`P2`) across springs `▲▲` — the main *climb-back* opportunity to reach MIDDLE. Otherwise a long, safe floor.
- **Hazards:** one **☠ telegraphed instant-death pit** near the end — clearly bottomless/dark, the honesty contract in action (`DAMAGE_AND_CHECKPOINTS.md`). Dense checkpoints `⚑` around it.
- **Reward:** low-value collectibles; safety.
- **Score opportunity:** minimal — a full-LOW run here falls behind pace (meter dips to "BEHIND").
- **Recovery opportunity:** the bounce sequence `▲▲ ↑` to MIDDLE; nailing it is the skill-gated climb back into contention.

**Reconnection:** HIGH (via shortcut) and MIDDLE/LOW (via normal routing) all arrive at **JUNCTION B**.

---

## JUNCTION B 🏦

- **Reconnects** routes; **banks score** again; pace readout updates (a HIGH-shortcut player sees *"AHEAD — ON TRACK FOR A/S"*).
- **Forced-choice power-up:** **2× Score OR 🛡 Double Shield** — pure greed-vs-safety before the final push (double shield = 2 armor in the Volt fight).
- **Decision point** into Segment 3's three entrances.
- Optional **✦ secret** here: a hidden **Overcharge** pickup (`POWERUPS.md`) tucked behind the junction — carry it to the finish without dying to pre-fill super meter vs. Volt. High risk/reward, rewards exploration.

---

## SEGMENT 3 — "The Overpass" (combine everything)

**Intent:** the climax of the stage — combine momentum, dash, rails, timing, and route reads under pressure. Short and intense; releases into the finish/handoff. Signature puzzle: **P1 Sequenced Switches under momentum** on HIGH (which also foreshadows Volt's rhythm).

```
HIGH   switch-switch-switch (hit 3 at speed) ─ ≈ rail ─ ◆ ─ ▼(to MID) ─ FINISH (fast)
       ──────────────────────────────────────────────
MID    • ─ ⚑ ─ dash-gap ─ ▲(↑ to HIGH, hard) ─ • ─ FINISH
       ──────────────────────────────────────────────
LOW    ⚑ ─ wide safe descent ─ • ─ ⚑ ─ FINISH (slow)   [+ Last-Chance room entrance if BELOW gate]
```

### HIGH PATH
- **Challenge:** **hit 3 switches in sequence while maintaining speed** (`P1`) to keep a gate open, then a final **rail ≈** into the finish. The rhythm of the three switches deliberately mirrors Volt's dash cadence (covert fight practice).
- **Hazards:** the gate closes if you slow down (fail → drop); no cheap deaths.
- **Reward:** a final **◆**; biggest Route-Difficulty + Style bonus; fastest finish.
- **Score opportunity:** seals an S-rank run if clean.
- **Failure destination:** lose the switch rhythm → **▼ drop to MIDDLE**.

### MIDDLE PATH
- **Challenge:** a **dash-gap** (dash required) then an optional **hard spring climb ↑ to HIGH** for brave players (`climb-back`).
- **Hazards:** the gap (miss → drop to LOW); otherwise moderate.
- **Reward:** standard collectibles; the climb opportunity.
- **Score opportunity:** solid B/A pace.
- **Failure destination:** miss the dash-gap → **▼ drop to LOW**.

### LOW PATH
- **Challenge:** a wide, gentle descent to the finish; trivial.
- **Hazards:** none; dense checkpoints.
- **Reward:** low-value collectibles.
- **Score opportunity:** minimal — a full-LOW run arrives around the **minimum gate (C/D)**.
- **Recovery opportunity:** if the player is **below the gate** at the finish, the LOW route hosts the entrance to the **Last-Chance Challenge** room (`PROGRESSION.md`) — a ~20s score gauntlet to still qualify, or opt into a disadvantaged Volt fight.

**Reconnection → FINISH TRIGGER → SCORE_EVAL.**

---

## 4. Route Interconnection Summary (this level)

Demonstrates the brief's requirement that routes weave and reconnect, not run parallel:

| Beat | Interconnection shown |
| --- | --- |
| Seg 1 HIGH miss → MID; MID miss → LOW | Falling-as-event, two tiers of drop |
| Seg 1 LOW spring ▲ → MID | First climb-back |
| Junction A switch → opens HIGH entrance (timed) | Route-change puzzle (climb via skill) |
| Seg 2 rail → shortcut cannon → Junction B | **Major shortcut** skipping a whole segment |
| Seg 2 destructible wall ✦ → cross-route tunnel toward HIGH | Secret climb |
| Seg 2 LOW bounce sequence ▲▲ → MID | Skill-gated climb-back |
| Seg 3 MID hard spring ↑ → HIGH | Late climb for brave players |
| Seg 3 LOW → Last-Chance room (if below gate) | Anti-frustration recovery |

Every drop has a climb-back within reach; every visible ◆/✦ has a real (hard) access route (see-but-can't-reach).

## 5. Qualification Meter Through the Run (worked walkthrough)

- **Start:** meter at 0, gate marked, rank ticks visible.
- **Seg 1 clean MID + 2× pick:** meter rises to ~"ON TRACK FOR B."
- **Drop to LOW in Seg 2:** meter visibly **dips**, label flips to **BEHIND** → the "I screwed up but can save it" tension (Pillar 1).
- **Nail the LOW bounce climb + MID secret:** meter recovers to **ON TRACK**.
- **Junction B bank:** locks the recovery in; a death now can't drop below this.
- **Seg 3 HIGH switch sequence clean:** meter surges to **AHEAD — A/S**.
- **Finish:** Score Eval tallies bonuses → rank stamp.

This single walkthrough demonstrates the meter never ambushes the player and that recovery is always visibly possible.

## 6. Handoff → VOLT (the payoff)

At the finish (`CORE_GAMEPLAY_LOOP.md` handoff):

**Example A — strong run (A rank, 2 shields held, high-route Seg 2 clear, Overcharge carried, beat par):**
```
Rank A ................. +Health
2 Shields .............. Armor ▮▮
High route (Seg 2) ..... Bonus Attack: Rail Punish unlocked
Overcharge carried ..... Super meter ▮▮▮▮▮▯ pre-filled
Beat par ............... First-strike dash
```
→ The player walks into the arena armored, metered, and fast — a victory lap they *earned*, but Volt still fights.

**Example B — rough run (D rank, no shields, full-LOW, Last-Chance passed):**
```
Rank D ................. reduced Health
No shields ............. no armor
(Last-Chance only) ..... rank capped at C
Poor score ............. VOLT advantage: +HP, +speed
```
→ Winnable with good defense and reads — the fight is hard, not predetermined (`BOSS_DESIGN.md` balance rule).

## 7. The Fight: VOLT

Full design in `BOSS_DESIGN.md`. In arena:
- **Volt** rushes with Dash Jab, Rail Slide (whiff-punishable — the taught counter), Spring Kick (anti-air mirroring the springs).
- The player uses the combat triangle (`FIGHTING_SYSTEM.md`): bait Volt's over-committed Rail Slide, block/dodge, then punish the recovery — exactly the "read over-committed momentum" skill the level drilled.
- Loadout from the run tilts the opening; execution decides the win.

## 8. What This Level Proves (for the POC build)

If built, this level validates the **risky, novel claims** of the whole design:
1. **Three interconnected routes in one space** (not parallel corridors).
2. **Falling-as-event** with authored catch-surfaces + visible meter dips.
3. **At least one climb-back and one major shortcut** that feel earned.
4. **Momentum as the core verb** (rails, boost pads, momentum-spend jumps).
5. **The handoff**: platforming performance visibly becomes combat advantage vs. Volt.

These five are the make-or-break of the concept and should be the POC's focus (see the chat summary's recommended scope).

## 9. Open Questions (this level)

- Is one major shortcut enough, or does the HIGH route need a second to feel rewarding? (Playtest.)
- Does the Overcharge-carry risk (lose on death) read as exciting or punishing here? (Playtest.)
- Are 2 junctions enough to make banking/pace-feedback land, or do we need 3? (Leaning: 2 for POC length targets.)
- Switch-sequence rhythm in Seg 3 mirroring Volt — too subtle, or just right? (Playtest the "covert practice" hypothesis.)
