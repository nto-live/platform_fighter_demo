# Design Documentation

> **DESIGN PHASE ONLY.** This directory contains design and technical-design documentation for a 2D Godot game that fuses Super Meat Boy precision, Sonic momentum/alternate routes, puzzle-platforming, and a climactic 1v1 fighting-game duel. **No implementation exists yet** — no scenes, nodes, scripts, or assets.

## The Concept in One Line

A high-speed 2D precision platformer where every level is a vertically-stacked web of three interlocking routes, and how well you run it is loaded directly into a one-on-one fighting-game duel at the end.

## Core Philosophy

**Failure changes the run instead of always ending it.** Missing a hard jump drops you to a lower, safer, lower-scoring route — the run continues. Expert players hold the high route; intermediates bounce between high and middle; beginners survive on the low route. One authored level serves every skill level.

## Reading Order

Start at the top for vision, then drill into systems.

### Vision & Loop
1. [GAME_DESIGN.md](GAME_DESIGN.md) — vision, pillars, references, glossary
2. [CORE_GAMEPLAY_LOOP.md](CORE_GAMEPLAY_LOOP.md) — the loop, game states, the handoff

### Platforming Systems
3. [MOVEMENT_SYSTEM.md](MOVEMENT_SYSTEM.md) — momentum, the movement kit, skill ceiling
4. [THREE_PATH_LEVEL_DESIGN.md](THREE_PATH_LEVEL_DESIGN.md) — high/middle/low routes & interconnection
5. [SCORING_SYSTEM.md](SCORING_SYSTEM.md) — the scoring formula & readability
6. [POWERUPS.md](POWERUPS.md) — power-up catalog & risk/reward choices
7. [DAMAGE_AND_CHECKPOINTS.md](DAMAGE_AND_CHECKPOINTS.md) — hazards, falling-as-event, checkpoints
8. [PUZZLE_DESIGN.md](PUZZLE_DESIGN.md) — momentum-integrated puzzle patterns
9. [PLATFORMING_COMBAT.md](PLATFORMING_COMBAT.md) — enemies in the stage, defeated through movement

### Fighting Systems
10. [FIGHTING_SYSTEM.md](FIGHTING_SYSTEM.md) — the simplified duel
11. [BOSS_DESIGN.md](BOSS_DESIGN.md) — bosses as characters; the level-teaches-boss contract

### Building & Tech
12. [LEVEL_DESIGN_GUIDE.md](LEVEL_DESIGN_GUIDE.md) — authoring large interconnected levels
13. [GODOT_ARCHITECTURE.md](GODOT_ARCHITECTURE.md) — systems, scenes/nodes, communication
14. [DATA_MODEL.md](DATA_MODEL.md) — Godot Resource schemas (data-driven design)

### Progression, Replay, Feedback
15. [PROGRESSION.md](PROGRESSION.md) — ranks, the soft score gate, unlocks, loadout
16. [REPLAYABILITY.md](REPLAYABILITY.md) — exploration → mastery, ghosts, leaderboards
17. [UI_AND_PLAYER_FEEDBACK.md](UI_AND_PLAYER_FEEDBACK.md) — HUD, qualification meter, feedback

### The Example Level
18. [PROOF_OF_CONCEPT_LEVEL.md](PROOF_OF_CONCEPT_LEVEL.md) — "Voltline Rush," one full level section-by-section, ending in a duel vs. VOLT

### Review
- [DESIGN_ANALYSIS.md](DESIGN_ANALYSIS.md) — internal review: strengths, gaps, resolved inconsistencies, ranked risks

## Design Pillars (quick reference)

1. **Failure changes the run; it rarely ends it.**
2. **Momentum is the core verb.**
3. **One level, every skill level.**
4. **Readable, honest, and generous.**
5. **The platforming earns the fight.**

## Status

Design phase complete and awaiting review. **No code should be written until the design is approved.**
