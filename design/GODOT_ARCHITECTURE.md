# GODOT_ARCHITECTURE.md

> **Status: DESIGN PHASE ONLY. No code exists.** This describes the *intended* Godot 4.x architecture. Any pseudocode is illustrative only.

## 1. Principles

1. **Modular, reusable systems — no god scripts.** Each manager owns one concern. No single script "runs the game."
2. **Composition over inheritance.** Behaviors are components (nodes) composed into scenes; data lives in Resources.
3. **Decoupled communication via a signal bus.** Systems emit/consume signals; they don't reach into each other's internals.
4. **Data-driven.** Levels, bosses, power-ups, moves, hazards, scoring rules are **Resources** (`DATA_MODEL.md`), not hard-coded.
5. **Two gameplay modes, shared shell.** Platforming and fighting are distinct scenes/state machines sharing the same managers (score, save, audio, input, game state).

## 2. High-Level System Map

```
                         ┌───────────────────────┐
                         │   Game State Manager   │  (autoload)
                         │  BOOT→MENU→…→VICTORY    │
                         └───────────┬───────────┘
                                     │ drives scene swaps
        ┌────────────────────────────┼────────────────────────────┐
        ▼                            ▼                            ▼
 ┌─────────────┐            ┌──────────────────┐          ┌──────────────┐
 │ MAIN MENU / │            │ PLATFORMING SCENE│          │ FIGHTING SCENE│
 │ LEVEL SELECT│            │  (a Level)       │          │  (an Arena)   │
 └─────────────┘            └────────┬─────────┘          └──────┬───────┘
                                     │                            │
        ┌───────── SIGNAL BUS (autoload EventBus) ──────────────────────┐
        │  score events, route events, pickup events, damage, state…    │
        └───────────────────────────────────────────────────────────────┘
   Shared autoload managers (always present):
   Score ▪ PowerUp ▪ Checkpoint ▪ Route ▪ LevelTransition ▪ Save ▪ Unlock
   ▪ Audio ▪ Input ▪ (HUD is a scene layer driven by signals)
```

## 3. Autoload Singletons (global managers)

Autoloads are the always-available services. Kept **thin** and **event-driven**.

| Autoload | Responsibility | Does NOT do |
| --- | --- | --- |
| **GameStateManager** | Owns the state machine (`CORE_GAMEPLAY_LOOP.md`); orchestrates scene swaps and the handoff | Gameplay logic |
| **EventBus** | Global signals; the decoupling layer | Hold state |
| **ScoreManager** | Tracks run score, live multiplier, banking, rank computation (reads `ScoreRuleData`) | Know about specific level geometry |
| **PowerUpManager** | Applies/times active power-ups; tracks carry-to-fight items (reads `PowerUpData`) | Spawn pickups (levels do that) |
| **CheckpointManager** | Registers checkpoints, resolves respawn point & banked state | Decide hazard lethality |
| **RouteManager** | Tracks which route tier the player is on (via route-tag volumes); feeds route-difficulty bonus | Move the player |
| **LevelTransitionManager** | Loads/unloads level & arena scenes; builds the handoff loadout payload | Fighting logic |
| **SaveManager** | Serialize/deserialize save data (records, unlocks, settings) | Gameplay |
| **UnlockManager** | Tracks and grants unlocks (levels, moves, characters) | UI |
| **AudioManager** | Music/SFX buses, ducking, state-driven music | Gameplay |
| **InputManager** | Maps raw input to context actions (platforming vs fighting), remapping, buffering | Interpret game rules |

> **Rule:** managers talk through **EventBus signals** and **shared Resources**, not direct references, wherever practical. This keeps them independently testable and swappable.

## 4. The Player

The player is a **scene** reused across both modes, with mode-specific state machines swapped in.

```
Player (CharacterBody2D)
├─ Sprite/AnimationTree
├─ CollisionShape(s)
├─ InputReader            (reads InputManager actions for current context)
├─ MovementController     (physics, momentum rules — MOVEMENT_SYSTEM.md)
├─ PlatformingStateMachine  (GROUNDED/AIRBORNE/WALL_SLIDE/DASH/SLIDE/RAIL/… )
├─ CombatStateMachine       (NEUTRAL/ATTACK/BLOCK/GRAB/DODGE/HITSTUN/SUPER/… )
├─ HealthComponent        (pips, shields, armor)
├─ Hurtbox / Hitbox       (combat; also hazard interaction in platforming)
└─ StatusComponent        (active power-up effects / loadout bonuses)
```

- **Only one state machine is active per mode.** In `PLATFORMING`, the PlatformingStateMachine drives; in `FIGHTING`, the CombatStateMachine drives. The `MovementController` is shared but configured differently.
- The player **does not own game rules** — it emits events ("picked up X", "took damage", "entered route Y") on the EventBus; managers react.

## 5. State Machines (two of them)

### Platforming State Machine
States from `MOVEMENT_SYSTEM.md` (GROUNDED, AIRBORNE, WALL_SLIDE, DASH, SLIDE, RAIL, LAUNCHED, HITSTUN, DEAD). Each state is a node/class with `enter/exit/physics_process/handle_input`. Transitions preserve momentum per the resource rules.

### Combat State Machine
States from `FIGHTING_SYSTEM.md` (NEUTRAL, WALK, JUMP, ATTACK_LIGHT/HEAVY, BLOCK, GRAB, DODGE, SPECIAL, SUPER, HITSTUN, KO). Driven by `MoveData` (frame data, cancels, properties).

Both follow the same **pluggable state-machine pattern** so the implementation is shared and each state stays small and testable.

## 6. Level Scene Composition

```
Level (Node2D)  [attaches a LevelData resource]
├─ TileMaps                 (static geometry per route band)
├─ RouteVolumes             (Area2D tagged HIGH/MIDDLE/LOW → RouteManager)
├─ Interactables            (switches, timed doors, moving/redirectable platforms)
├─ Hazards                  (instanced from HazardData-configured scenes)
├─ Collectibles             (score pickups)
├─ PowerUpPickups           (PowerUpData-configured; incl. forced-choice pedestals)
├─ Checkpoints              (micro + junction; register with CheckpointManager)
├─ Springs/Launchers/Rails  (momentum connectors)
├─ CameraController         (follows player, respects band)
└─ FinishTrigger            (ends stage → SCORE_EVAL)
```

Everything interactive is an **instanced component scene**, parameterized by a Resource where useful. No per-level bespoke mega-scripts.

## 7. Fighting Scene Composition

```
Arena (Node2D)  [attaches BossData → arena_scene_ref]
├─ Background / stage art
├─ PlayerFighter      (Player scene in combat mode + loadout applied)
├─ BossFighter        (driven by BossData + ai_profile)
├─ CombatDirector     (round flow, win/lose, meter rules)
├─ HUD layer          (health bars, meters, armor pips)
└─ CameraController    (fighting-game framing: zoom to action)
```

The **loadout payload** built by LevelTransitionManager is applied to `PlayerFighter` on entry (health, armor, meter prefill, unlocked moves).

## 8. Communication Patterns (who talks to whom)

Illustrative signal flow (names are indicative, not final):

```
Collectible picked up
  → EventBus.collectible_collected(value)
      → ScoreManager (adds base × multiplier, updates meter)
          → EventBus.score_changed(total, multiplier)
              → HUD updates qualification meter + multiplier readout

Player enters HIGH route volume
  → RouteManager.set_route(HIGH)
      → EventBus.route_changed(HIGH)
          → ScoreManager (route-difficulty accrual)
          → HUD (route indicator)

Player takes a knock-to-route hazard
  → HealthComponent resolves (Invuln→Shield→Health→consequence)
      → EventBus.player_knocked(route_below)
          → ScoreManager (multiplier partial drop)
          → (player physically launched to lower route)

Finish trigger
  → EventBus.stage_finished()
      → ScoreManager.finalize() → rank
      → GameStateManager → SCORE_EVAL → LOADOUT → HANDOFF
          → LevelTransitionManager.build_loadout() → load Arena
```

**Key decoupling win:** the HUD never reads gameplay objects directly; it only listens to EventBus signals. The ScoreManager never knows what a "spring" is; it only hears events.

## 9. Save / Unlock / Progression

- **SaveManager** persists: per-level best score/rank/time, unlocks, settings, and leaderboard-eligible records (`REPLAYABILITY.md`).
- **UnlockManager** holds unlock state and grants on milestones (beat boss, S-rank, secrets) — emits `unlock_granted` for UI celebration.
- Both are pure data services; gameplay emits events, they record.

## 10. HUD & UI

- HUD is a **CanvasLayer scene** composed of independent widgets (qualification meter, multiplier, health/shield pips, power-up timers, route indicator) — each a small scene bound to EventBus signals (`UI_AND_PLAYER_FEEDBACK.md`).
- Menus and the Score-Eval/Loadout screens are their own scenes driven by GameStateManager.

## 11. Why This Avoids God Scripts

| Temptation | Our structure instead |
| --- | --- |
| One "GameManager" doing everything | Split into GameState + specialized autoloads, each one concern |
| Player script holding score/health/rules | Player emits events; managers own rules |
| Per-level scripts with copy-pasted logic | Reusable component scenes + LevelData |
| Hard-coded balance | ScoreRuleData / LevelData / DifficultyData Resources |
| Direct cross-system references | EventBus signals + shared Resources |

## 12. Testing & Determinism

- **Fixed-step physics (60 FPS)** for deterministic momentum and fighting feel.
- State machines and managers are small enough to unit-test in isolation (feed events, assert state).
- Data-driven design means balance tests can run against Resource values without touching logic.

## 13. Open Architecture Questions

> See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) for the master register. Grep **`❓ OPEN QUESTION`**.

- ❓ OPEN QUESTION [Q-ARCH-01] EventBus signals vs. a lightweight message/queue for ordering guarantees on the same frame? (Leaning: signals + careful emit ordering.)
- ❓ OPEN QUESTION [Q-ARCH-02] Should RouteManager infer tier purely from volumes, or also from player altitude as a fallback? (Leaning: volumes authoritative, altitude as a sanity check.)
- ❓ OPEN QUESTION [Q-ARCH-03] Shared MovementController across modes, or a separate combat mover? (Leaning: shared body, different tuning profiles.)
- Scene streaming for very large levels vs. single-scene load? (Defer; POC levels fit in one scene.)
