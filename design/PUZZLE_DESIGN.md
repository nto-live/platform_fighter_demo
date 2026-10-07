# PUZZLE_DESIGN.md

> **Status: DESIGN PHASE ONLY.**

## 1. Philosophy

Puzzles in this game are **traversal puzzles solved in motion**, not static logic puzzles that halt the platformer. The brief is explicit: *avoid puzzles where the player stops playing the platformer to solve an unrelated puzzle.*

Our definition:

> **A good puzzle here is a question about the room that you answer with movement.** The "aha" is a route or a sequence; the "execution" is platforming. Understanding the environment, not reflexes alone, unlocks the best lines (this is the Celeste/puzzle-platformer DNA layered onto Sonic momentum).

Two difficulty axes, always separable:
- **Comprehension** — do you understand what the room wants?
- **Execution** — can your hands do it?

Great puzzles let a player *see* the solution before they can *perform* it (the "see-but-can't-reach" principle), which drives replay.

## 2. Puzzle Design Rules

1. **Momentum-integrated:** the solution involves carrying, converting, or spending speed — never fully stopping.
2. **Readable premise:** the player can perceive the puzzle's elements (switches, doors, platforms) and infer cause→effect.
3. **Multi-solution where possible:** at least a hard/fast line and an easy/slow line, mapping to route tiers.
4. **Fail = drop, not stop:** flubbing a puzzle usually drops you to a lower route or a safe retry, not a hard dead-end.
5. **No pixel-hunting, no obscure logic:** comprehension should come from *watching the room*, not from trial-and-error on hidden rules.
6. **Teach then test:** introduce a mechanic safely, then combine it under pressure (feeds `BOSS_DESIGN.md`'s "level teaches the boss").

## 3. Core Puzzle Patterns (the toolkit)

Each pattern below includes intent, the momentum hook, and how multiple solutions map to routes.

### P1 — Sequenced Switches Under Momentum
Hit N switches in order (or within a time window) **while maintaining speed**.
- *Momentum hook:* switches are spaced so you must keep moving to reach the next before a timer/door resets.
- *Multi-solution:* HIGH line hits all switches in one fast pass; LOW line uses a slower platform cycle to hit them across several passes.

### P2 — Bounce Sequence
Bounce off objects (bumpers, enemies, springs) in a correct order to climb or cross.
- *Momentum hook:* each bounce preserves/redirects momentum; wrong order drops you.
- *Multi-solution:* precise 3-bounce climb (HIGH) vs. a safer 2-bounce + platform (MIDDLE).

### P3 — Platform Redirection
Redirect moving platforms (via switch, weight, or impact) to form a path.
- *Momentum hook:* you often must set the redirect, then *beat the platform to the spot* using your own speed.
- *Multi-solution:* redirect for a direct HIGH shortcut vs. ride the default slow loop (LOW).

### P4 — Quick Route Fork
A split where you must *choose and commit* in a split second based on a readable cue (color, light, hazard pattern).
- *Momentum hook:* no time to stop; the choice is made at speed.
- *Multi-solution:* the fork itself *is* the route selector (HIGH/MIDDLE/LOW branches).

### P5 — Beat-the-Door
Carry momentum through a closing door / timed gate before it shuts.
- *Momentum hook:* you must *arrive boosted*; walking won't make it.
- *Multi-solution:* dash+slide through the fast door (HIGH) vs. a lower door with a longer timer (LOW).

### P6 — Environmental Manipulation
Move/trigger objects (fans, water level, magnets, gravity pads) that reshape traversal.
- *Momentum hook:* the manipulated element changes *how* momentum flows (a fan that extends a jump, a magnet that redirects a dash).
- *Multi-solution:* use the manipulation to open a HIGH climb vs. ignore it and take the default MIDDLE/LOW path.

### P7 — Route-Change Mechanism
A switch on one route that opens a connection to another (the interconnection driver from `THREE_PATH_LEVEL_DESIGN.md`).
- *Momentum hook:* often a "pull the lever on MIDDLE, then sprint to the HIGH entrance it opened before it closes."
- *Multi-solution:* skip it (stay on current route) vs. use it to climb.

### P8 — Hidden Passage Discovery
Find and open a secret (destructible wall, false floor, obscured tunnel).
- *Momentum hook:* often requires a Phase Dash or a specific-speed hit to break through.
- *Multi-solution:* known shortcut for veterans; invisible to first-timers (replay driver).

### P9 — Hazard-as-Tool
Use an enemy or hazard as traversal (bounce off a saw's mount, ride a crusher, launch off a blast jet).
- *Momentum hook:* the hazard injects/redirects momentum if used correctly; damages you if not.
- *Multi-solution:* the risky tool-use (HIGH) vs. avoiding it entirely (LOW).

### P10 — Momentum Bank-and-Spend
Build boosted speed in a run-up chamber, then spend it on a single long traversal (big gap, uphill, long rail).
- *Momentum hook:* literally the momentum resource rules (`MOVEMENT_SYSTEM.md`) as a puzzle.
- *Multi-solution:* Momentum Lock power-up makes it trivial; raw skill makes it tight; LOW offers a bypass.

## 4. Multi-Solution & Skill Scaling

Every puzzle should ideally answer: *what does a beginner do, and what does an expert do?*

| Skill | Typical puzzle behavior |
| --- | --- |
| Beginner | Takes the slow, safe, telegraphed solution (usually drops them toward LOW); still progresses |
| Intermediate | Solves the MIDDLE version; occasionally spots a climb |
| Expert | Executes the fast, tight, HIGH solution in one fluid motion; often chains puzzles together |

This directly serves Pillar 3 (one level, every skill level): the puzzle's *comprehension* is shared, but its *execution ceiling* spans the whole skill range.

## 5. Pacing Puzzles Within a Level

- **Don't gate forward progress on comprehension.** A player who doesn't "get" a puzzle should still be able to fall/take the slow route onward. The puzzle gates *the better outcome* (HIGH route, more score), not *continuing to play*.
- **Cluster teaching early, testing late.** Early segments introduce a mechanic in isolation; later segments and the boss combine them.
- **One "signature puzzle" per segment** — a memorable set-piece — surrounded by lighter traversal.

## 6. Anti-Patterns (do not do)

- ❌ Full-stop logic puzzles (sliding-block, color-code memorization) with no movement.
- ❌ Puzzles whose solution is invisible until you've already failed lethally (unfair comprehension).
- ❌ Single-solution puzzles that hard-block all three routes (kills the one-level-every-skill promise).
- ❌ Trial-and-error with lethal cost per attempt (violates "fail = drop, not death").
- ❌ Puzzles that require releasing all momentum to solve (violates Pillar 2).

## 7. How Puzzles Feed the Boss

Per the brief, the platforming subtly teaches boss mechanics (`BOSS_DESIGN.md`):
- A level heavy on **P5 Beat-the-Door / timing** → a boss with punishing timing windows.
- A level heavy on **P9 Hazard-as-Tool / P2 Bounce** → a mobile, reflective, or counter-based fighter.
- A level heavy on **P1 Sequenced Switches** → a boss weak to a specific *sequence* of hits.

The puzzle vocabulary of a level should rhyme with its boss's demands, so the stage is covert practice for the fight.

## 8. Data-Driven Hook

Puzzle elements (switches, timed doors, redirectable platforms, destructibles) are reusable scene components parameterized by Resources where it helps (timer durations, required sequence, which route they affect). The puzzle *logic* lives in small, composable interactable nodes wired via signals (`GODOT_ARCHITECTURE.md`), not in a monolithic puzzle script. There is no single "PuzzleManager" god-object; puzzles emerge from composed interactables.

## 9. Open Questions

> See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) for the master register. Grep **`❓ OPEN QUESTION`**.

- ❓ OPEN QUESTION [Q-PUZ-02] How much *explicit* signposting (arrows) vs. pure environmental read? (Leaning: strong consistent color language, minimal arrows.)
- ❓ OPEN QUESTION [Q-PUZ-01] Should solved route-change mechanisms persist across death-retry within a segment? (Leaning: yes; reset only on segment restart.)
- ❓ OPEN QUESTION [Q-PUZ-03] Do we need a "puzzle solved" scoring popup, or does the route/score speak for itself? (Leaning: subtle popup for secrets only.)
