# GAME_DESIGN.md

> **Status: DESIGN PHASE ONLY.** This document describes intent. No code, scenes, or assets exist yet.

## 1. One-Line Pitch

A high-speed 2D precision platformer where every level is a vertically-stacked web of three interlocking routes, and your performance through it is literally loaded into the magazine of a one-on-one fighting-game duel at the end.

## 2. The Hybrid

The game fuses four reference experiences into a single arc:

| Reference | What we take | What we leave |
| --- | --- | --- |
| **Super Meat Boy** | Instant retry, death-is-cheap tension, razor-tight controls, readable hazards, "one more try" loop | Pure linearity, death as the only failure state |
| **Sonic the Hedgehog** | Momentum, speed as a resource, tiered alternate routes stacked vertically, "see places you can't reach yet" | Momentum that feels floaty or auto-piloted; losing all rings on one hit |
| **Puzzle platformers (Celeste B-sides, Braid-adjacent)** | Understand-the-room-before-you-run, multi-solution traversal puzzles | Stopping the action to solve a static logic puzzle |
| **Street Fighter / Mortal Kombat** | A real character duel as the climax, movesets with identity, reads and spacing | Deep execution barriers (motion-input combos, frame-perfect links) |

The design north star: **the platforming half and the fighting half are two movements of one song, not two mini-games bolted together.** The bridge between them is the score system (Section 11 of the brief / `PROGRESSION.md`).

## 3. Design Pillars

These five pillars are the tie-breakers for every future decision. If a feature doesn't serve a pillar, it is scope.

### Pillar 1 — Failure changes the run; it rarely ends it
Missing a hard jump drops you to a lower, safer, lower-scoring route. The run continues at a disadvantage. Death (instant retry) is reserved for genuine hazards and bottomless pits, not for "you weren't good enough at that jump." This is the project's defining emotional beat: *"I screwed up, but I can still save this."*

### Pillar 2 — Momentum is the core verb
Speed is earned, stored, spent, and lost. The whole kit (jumps, dashes, slides, springs, rails) exists to convert momentum between horizontal and vertical, and to let skilled players sustain it. A player standing still is a player not playing.

### Pillar 3 — One level, every skill level
The three-route structure means a single authored level serves beginners (survive on the low road), intermediates (bounce between high and middle), and experts (hold the high road end-to-end). We do not ship difficulty variants of a level; the routes *are* the difficulty settings, chosen live and continuously.

### Pillar 4 — Readable, honest, and generous
Hazards look dangerous. Safe things look safe. The player always has a rough sense of whether they're on pace to fight the boss (the qualification meter). We never spring a 10-minute "you failed, start over" on the player — see `PROGRESSION.md` score-gate alternatives.

### Pillar 5 — The platforming earns the fight
The duel's starting conditions are the payoff for the run. High score, held shields, found secrets, speed, and route difficulty all convert into concrete combat advantages (bonus health, armor, extra super meter, unlocked moves). A great run should *feel* great the instant the fight starts, before a single button is pressed.

## 4. Player Fantasy

> "I am a momentum artist. I read a room, pick a line, and commit. When I nail the high road I feel untouchable — and then I walk into the arena already winning. When I blow it, I don't rage-quit; I scramble to claw back enough points on the low road to still earn my shot."

Two fantasies, one character:
- **The runner** — flow, speed, mastery of space.
- **The duelist** — reads, spacing, a decisive 1v1.

The transition between them is the emotional climax of each level.

### Character model (decision + future hook)
**Decision:** the demo ships a **single playable character, used across both halves** (same character runs the stage and fights the boss). This keeps the runner/duelist identity unified and the scope tight.

> **Future extension — multiple characters with distinct movesets.** The architecture must *not* assume there is only one character. A later roster is an explicit, planned extension point:
> - Each character is a `CharacterData` Resource (`DATA_MODEL.md`) carrying **both** its platforming **movement profile** (speed, jump, dash tuning — `MOVEMENT_SYSTEM.md`) **and** its combat **moveset** (`MoveData` lists — `FIGHTING_SYSTEM.md`). One Resource spans both halves, preserving "two fantasies, one character" per character.
> - Characters are already listed as an unlock type in `PROGRESSION.md`; a new character = new data + animations, **no system rewrite** (`DATA_MODEL.md` authoring flow).
> - Different characters may lean the feel slider differently (one flowier, one tighter) and favor different routes/bosses — a strong replay and expression driver (`REPLAYABILITY.md`).
> - **Out of demo scope**, but all of the above is why `CharacterData` exists now rather than hard-coding the one character.

## 5. Target Experience by Skill Tier

| Tier | Platforming behavior | Typical rank | Boss experience |
| --- | --- | --- | --- |
| Beginner | Mostly low route, uses recovery aids, finishes levels | C–D | Fights boss with a disadvantage or via last-chance challenge; can still win with good defense |
| Intermediate | Bounces high↔middle, recovers from mistakes, hunts shortcuts | B–A | Fair fight, maybe a small edge |
| Expert | Holds high route, chains momentum, collects everything | S | Walks in with armor, extra meter, bonus move, speed edge |

The game must be **winnable and fun at the bottom tier** and **deep and expressive at the top tier**, from the same content.

## 6. Scope Posture (Design Phase)

- This phase produces documentation only (see list in `README` of `/design`).
- The eventual **proof of concept** should prove the *risky, novel* claims — route interconnection, falling-as-event, and the platforming→fight handoff — not the full roster or meta-progression. See the final summary in chat and `PROOF_OF_CONCEPT_LEVEL.md`.
- Everything is designed to be **data-driven** (Godot Resources) so levels, bosses, power-ups, and characters can be authored without rewriting systems (`DATA_MODEL.md`).

## 7. Platform & Tech Assumptions

- **Engine:** Godot 4.x (2D). All system designs assume Godot's scene/node/Resource model and the signal bus pattern. See `GODOT_ARCHITECTURE.md`.
- **Perspective:** 2D side-on. ❓ OPEN QUESTION [Q-ART-01] 🔴 **(awaiting user)** pixel art vs. clean vector — undecided; affects tooling, animation pipeline, and hazard readability (Pillar 4). Not a design-phase blocker. See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md).
- **Input:** Gamepad-first, full keyboard support. Fighting controls are deliberately low-execution (no motion inputs required; see `FIGHTING_SYSTEM.md`).
- **Framerate target:** 60 FPS fixed-step for deterministic physics and fighting-game feel.

## 8. Document Map

| Document | Covers |
| --- | --- |
| `CORE_GAMEPLAY_LOOP.md` | The macro loop and all game states |
| `MOVEMENT_SYSTEM.md` | The core verb: momentum and the movement kit |
| `THREE_PATH_LEVEL_DESIGN.md` | High/middle/low routes and how they interconnect |
| `SCORING_SYSTEM.md` | The scoring formula and its readability |
| `POWERUPS.md` | Power-up catalog and risk/reward choices |
| `DAMAGE_AND_CHECKPOINTS.md` | Hazards, damage, falling-as-event, checkpoints |
| `PUZZLE_DESIGN.md` | Momentum-integrated puzzle patterns |
| `FIGHTING_SYSTEM.md` | The simplified duel |
| `BOSS_DESIGN.md` | Bosses as characters; level-teaches-boss |
| `LEVEL_DESIGN_GUIDE.md` | Authoring large interconnected levels |
| `GODOT_ARCHITECTURE.md` | Systems and how they communicate |
| `DATA_MODEL.md` | Godot Resource schemas |
| `PROGRESSION.md` | Ranks, score gates, unlocks, meta |
| `REPLAYABILITY.md` | Exploration → mastery |
| `UI_AND_PLAYER_FEEDBACK.md` | HUD and feedback |
| `PROOF_OF_CONCEPT_LEVEL.md` | One full example level |

## 9. Glossary

- **Route / Path:** One of the three vertically-stacked lanes (high/middle/low) through a level.
- **Drop:** A failure or deliberate descent that moves the player to a lower route without death.
- **Climb:** A skill- or puzzle-gated opportunity to move to a higher route.
- **Qualification meter:** The always-visible indicator of whether current score is on pace for the boss.
- **Banked score:** Score locked in at mid-level checkpoints so it can't be fully lost.
- **Handoff:** The transition from platforming to the fight, carrying performance data.
- **Loadout bonus:** A concrete combat advantage granted by platforming performance.
