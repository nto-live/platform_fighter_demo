# THREE_PATH_LEVEL_DESIGN.md

> **Status: DESIGN PHASE ONLY.**

## 1. The Core Idea

Every major level is one continuous space divided into **three vertically-stacked routes** that constantly weave around each other. They are not three separate levels — they are three lines through one shared geometry, and the player moves between them continuously, mostly by falling (Pillar 1) and occasionally by climbing (skill reward).

```
HIGH  ====\        /====\            /========  (fast, deadly, rich)
           \  ↓   /      \    ↑drop  /
MIDDLE ======\===/========\=========/=========  (normal)
              \ ↓          \ ↑climb
LOW    =========\===========\================== (slow, safe, poor)
```

## 2. The Three Routes

### HIGH PATH — Expert
- Extreme precision; assumes **boosted speed** (see `MOVEMENT_SYSTEM.md`).
- Tiny landing zones, long momentum jumps, grind rails, dangerous shortcuts.
- Puzzles solved *while moving*.
- Fewest checkpoints, rarest power-ups, highest-value collectibles, biggest multipliers, fastest clear.
- **Greatest reward. A mistake usually drops you to MIDDLE, not death.**

### MIDDLE PATH — Intermediate (the "spine")
- Moderate difficulty; base speed sufficient.
- Forgiving jumps, moderate enemies, environmental puzzles.
- Standard collectibles/power-ups, more checkpoints.
- **A mistake may drop you to LOW. Skilled play can climb back to HIGH.**

### LOW PATH — Recovery / Easy
- Easiest jumps, slower, most checkpoints, fewest hazards.
- Lower-value collectibles, smaller multipliers, longer route, fewer shortcuts.
- **Not punishment — a recovery lane that keeps the run alive.** But a full-level low run makes the boss score-gate significantly harder to meet (by design — see `SCORING_SYSTEM.md` and `PROGRESSION.md`).

## 3. Vertical Relationship Rules

These rules make the routes feel like one space:

1. **Routes share a continuous X axis.** The level reads left-to-right START→END for all three; they differ in Y (height) and in the challenges at that height.
2. **Higher = faster + harder + richer.** Height correlates with speed demand, danger, and reward. This is the legibility contract with the player.
3. **Failure drops downward, not to death** (within reason — bottomless pits and lethal hazards still exist on specific beats; see `DAMAGE_AND_CHECKPOINTS.md`).
4. **Climbing is earned, not given.** Moving up a route requires a skill expression (wall-jump chain, a dash through a window, a nailed launcher) or solving a route-change puzzle.
5. **Routes periodically reconnect at "junction rooms,"** then split again. Junctions re-level the playing field and give the player a decision point.

## 4. Interconnection Toolkit

The connective tissue between routes. Level designers pick from these (catalogued fully in `LEVEL_DESIGN_GUIDE.md`):

| Element | Direction | Typical use |
| --- | --- | --- |
| **Drop point** | Down | Edge/gap where a miss sends you to the route below |
| **Recovery jump** | Up (small) | A reachable ledge to claw back one tier if you react fast |
| **Spring / bounce pad** | Up | World-driven climb; often the low→middle connector |
| **Launcher / cannon** | Up (big) | Dramatic multi-tier climb; often puzzle-gated |
| **Secret elevator** | Up | Hidden; rewards exploration with a route upgrade |
| **Wall-jump shaft** | Up | Skill-gated vertical climb between tiers |
| **Hidden tunnel** | Lateral/diagonal | Secret cross-route connection |
| **Moving platform** | Up/down/across | Timed connector; miss it and you wait or drop |
| **Timed door** | Lateral | Carry momentum through before it shuts (route-gating) |
| **Switch / mechanism** | Changes another route | Pull a lever on MIDDLE that opens a HIGH shortcut |
| **Destructible passage** | Any | Break through to reveal a connection |
| **Grind rail** | Up/across | Signature high-route connector |
| **Risk/reward shortcut** | Usually down-then-up | A gamble: nail it to skip ahead, miss it to drop |

**Design mandate:** a skilled player should, over several runs, be able to *mentally map* how all three routes connect. The connections are consistent and learnable, never random.

## 5. The "See-But-Can't-Reach" Principle

The player should **frequently see areas they can't currently reach** (a Sonic hallmark and a core replay driver — `REPLAYABILITY.md`). Implementation guidance:
- From the LOW route, the HIGH route's rails and collectibles are often *visible above* through gaps in the mid-ground.
- A locked shortcut seen on run 1 becomes a known target on run 5.
- Visibility must be honest: if you can see a collectible, there is a real (if hard) way to reach it.

## 6. Falling-As-Event: the drop model

The signature mechanic. When the player fails a jump:

```
HIGH miss ──> short fall ──> lands on MIDDLE (keeps most momentum)
MIDDLE miss ──> short fall ──> lands on LOW (keeps some momentum)
LOW miss ──> depends: wide safe floor OR a marked lethal pit (retry)
```

Rules for drops:
- **Drops preserve momentum** where possible — you land *running*, so the run keeps flowing (Pillar 2).
- **The landing is designed, not random.** Each high/middle drop zone has an authored catch-surface on the route below, placed so the fall feels like a demotion, not a disaster.
- **A drop costs score pace, not the run.** You lose the high route's multiplier/collectibles for that stretch; the qualification meter visibly dips, creating the "I can still save this" tension.
- **Only specific, clearly-telegraphed beats are lethal** (bottomless pits on the low route, spikes, instant-death hazards). Everywhere else, down ≠ dead.

See `DAMAGE_AND_CHECKPOINTS.md` for the full hazard taxonomy and which hazards drop vs. kill.

## 7. Climb-Back Opportunities

To keep hope alive and reward recovery:
- Every stretch where the player can drop should have, within a reasonable distance, **at least one climb-back opportunity** (spring, wall shaft, launcher, elevator).
- Climb-backs are **harder than staying up** would have been — recovery has a skill cost — but they exist.
- Some climb-backs are **secret** (hidden elevators/tunnels), rewarding exploration with a route upgrade mid-run.

## 8. Junction Rooms (reconnection beats)

Periodically all three routes funnel into a shared **junction room**:
- Acts as a **soft checkpoint and decision point**: "which route do I commit to next?"
- Often contains a **score checkpoint** (banking — `SCORING_SYSTEM.md`) and sometimes a **power-up choice** (`POWERUPS.md`).
- Re-levels momentum so each new segment can set its own speed demands.
- Good place for a **route-change puzzle** (hit the switch to open the high entrance before the timer runs out).

Rough cadence: a level has a few junctions dividing it into **segments**; within each segment the three routes split and weave independently.

## 9. Authoring a Segment (checklist for level designers)

For each segment, define per route:
- **Challenge** (the main traversal demand)
- **Hazards** (what threatens the player)
- **Reward** (collectibles / power-ups / multiplier)
- **Score opportunity** (how pace is gained here)
- **Failure destination** (which route below you drop to — or recovery, on LOW)
- **Climb opportunity** (how, if at all, you can move up from here)

This is exactly the template used in `PROOF_OF_CONCEPT_LEVEL.md`.

## 10. Why This Serves One-Level-Every-Skill (Pillar 3)

The three routes are not difficulty *modes* chosen in a menu — they're chosen *continuously, by play*. A single authored level is simultaneously:
- A gentle, survivable experience on the LOW route for beginners,
- A dynamic high↔middle dance for intermediates,
- A ruthless speed-precision gauntlet on the HIGH route for experts.

No separate "easy version" is ever authored. The geometry *is* the difficulty curve, laid out in space rather than selected in a menu.
