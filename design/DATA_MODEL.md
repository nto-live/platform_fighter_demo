# DATA_MODEL.md

> **Status: DESIGN PHASE ONLY.** Schemas describe intended Godot `Resource` types. Field types are indicative; no `.gd`/`.tres` files exist yet.

## 1. Why Data-Driven

Levels, bosses, power-ups, moves, hazards, and balance should be **authored as data (Godot Resources)**, not baked into code. Goals:
- Add a new level / boss / power-up / character **without rewriting systems**.
- Let designers tune balance by editing `.tres` files, not GDScript.
- Keep systems (`GODOT_ARCHITECTURE.md`) generic; behavior comes from the data they consume.

Resources are Godot's native serialized data objects (editable in the Inspector, saved as `.tres`), which makes them the right tool here.

## 2. Resource Catalog

| Resource | Consumed by | Purpose |
| --- | --- | --- |
| `CharacterData` | Player/Fighter | Playable character stats, moveset, visuals |
| `MoveData` | Combat State Machine | One attack's full definition |
| `BossData` | Fighting scene, Boss AI | A boss's identity, kit, arena, scaling |
| `LevelData` | Level scene, managers | Per-level config: par, thresholds, boss ref, theme |
| `PowerUpData` | PowerUpManager, pickups | One power-up's effect/visuals/rules |
| `HazardData` | Hazard scenes | One hazard's consequence/visuals |
| `EnemyData` | Enemy scenes | A platforming enemy's archetype, vulnerability, threat (`PLATFORMING_COMBAT.md`) |
| `ScoreRuleData` | ScoreManager | Scoring values, multiplier curves, rank thresholds |
| `DifficultyData` | multiple | Global difficulty profile / assist settings |
| `CheckpointData` | CheckpointManager | (lightweight) per-checkpoint flags |
| `LoadoutPayload` | LevelTransitionManager | Runtime (not authored) run→fight bundle |
| `SaveData` | SaveManager | Persistent player progress |

## 3. Schemas

Indicative fields; `[Res]` denotes a reference to another Resource; `[ ]` a typed array.

### CharacterData
> **This is the roster extension point** (`GAME_DESIGN.md` §4). One `CharacterData` fully defines a character across **both** halves: its platforming **movement profile** *and* its combat **moveset**. The demo authors exactly one; a future roster adds more as pure data (+ animations) with no system rewrite. Systems (MovementController, Combat State Machine) always read the *active* character's `CharacterData` — they never assume a single global character.

```
id : StringName
display_name : String
sprite_frames / anim_tree_ref
# --- platforming movement profile (MOVEMENT_SYSTEM.md §5) ---
base_move_speed : float          # the "1.0 v" reference
boosted_ceiling_mult : float
jump_profile : {tap_h, full_h, coyote, buffer}
dash : {charges, distance, refund_on_land}
platforming_health : int         # pip pool for the stage (target 3); see DAMAGE_AND_CHECKPOINTS.md
# --- combat moveset (FIGHTING_SYSTEM.md) ---
combat_health : int              # single-bar max for the duel; rank sets starting fill (PROGRESSION.md §4)
move_set : [Res MoveData]
special_moves : [Res MoveData]
super_move : Res MoveData
# --- meta ---
unlockable : bool                # false for the demo's starting character; true for future roster unlocks
```

### MoveData
```
id : StringName
display_name : String
input_trigger : enum {LIGHT, HEAVY, JUMP_ATK, GRAB, DODGE, SPECIAL+dir, SUPER}
damage : int
startup_frames / active_frames / recovery_frames : int
on_hit : {hitstun, knockback_vec, launch:bool, knockdown:bool}
on_block : {blockstun, pushback, chip}
meter_cost : int                 # 0 for basics
meter_gain : int
cancel_into : [StringName]       # combo/branch routing
properties : flags {ARMOR, PROJECTILE, ANTI_AIR, INVULN_WINDOW,...}
vfx_ref / sfx_ref
```

### BossData
```
id / display_name / personality_blurb
style_tag : enum {ZONER, RUSHDOWN, GRAPPLER, TRICKSTER, BRUISER}
arena_scene_ref : PackedScene
base_health : int
meter_rules : {gain_rate, super_cost}
move_set / special_moves / super_move : [Res MoveData]
weakness_tag : enum {SEQUENCE, WHIFF_PUNISH, ANTI_AIR, CORNER, REFLECT}
ai_profile_ref : Res AiProfileData       # aggression, telegraph len, punish prob
level_connection : {level_id, taught_mechanic}
disadvantage_scaling : {hp_mult, speed_mult, extra_armor}   # applied when player under-scored
visual_refs / audio_refs
```

### LevelData
```
id / display_name / theme_tag
segments : [Res SegmentData]       # optional structuring
par_time : float
score_rules_ref : Res ScoreRuleData    # per-level overrides
rank_thresholds : {D,C,B,A,S : int}    # or inherited from ScoreRuleData
boss_ref : Res BossData
music_refs
intro_tease_ref
collectible_total : int            # reachable this level (for % bonus)
secrets_total : int
```

### SegmentData (optional, aids tooling)
```
id / name / theme
has_junction : bool
banks_score : bool
forced_choice_powerups : [Res PowerUpData] (pair)
route_change_puzzle_ref : optional
```

### PowerUpData
```
id / display_name / icon
effect_type : enum {SHIELD, DOUBLE_SHIELD, INVULN, SPEED_SHOES, DOUBLE_JUMP,
                    DASH_CHARGE, SCORE_MULT, MAGNET, COMBO_KEEPER, SLOW_MO,
                    PHASE_DASH, MOMENTUM_LOCK, SPRING_BOOTS, GHOST_RAIL,
                    SCORE_SHIELD, OVERCHARGE, DECOY}
magnitude : float                  # e.g. 2.0 for 2× score, speed mult, etc.
duration : float                   # 0 = instant/stateful (shields)
stack_rule : enum {REFRESH, STACK, REPLACE}
carries_to_fight : bool
carry_effect : enum {NONE, ARMOR_POINT, METER_PREFILL, SPEED_START}
rarity : enum {COMMON, UNCOMMON, RARE}
allowed_routes : flags {HIGH, MIDDLE, LOW, SECRET}
pickup_vfx/sfx ▪ active_vfx/sfx
```

### HazardData
```
id / display_name
consequence : enum {DAMAGE, SHIELD_BREAK, KNOCK_TO_ROUTE, CHECKPOINT_RESET, INSTANT_DEATH}
damage_amount : int
knockback_vec / knockback_force     # for KNOCK_TO_ROUTE
multiplier_effect : enum {NONE, PARTIAL_DROP, RESET}
is_traversal_tool : bool            # can be used as a tool if approached right
reset_on_respawn : bool
telegraph_vfx/sfx
```

### ScoreRuleData
```
id
base_values : { collectible_common, collectible_premium, enemy, puzzle, secret, near_miss }
multiplier : { growth_per_event, decay_per_sec_idle, drop_on_route_fall, partial_drop_amount, cap }
            # NOTE: whether a hit dips vs. resets the multiplier is authored per-hazard via
            # HazardData.multiplier_effect, NOT a global boolean here (see I2 in DESIGN_ANALYSIS.md).
end_bonuses : { time_weight, health_weight, shield_weight, route_difficulty_weight,
                no_hit_bonus, no_death_bonus, collection_weight, secret_weight, style_weight }
rank_thresholds : {D,C,B,A,S : int}
boss_gate_min : int                 # the soft minimum to reach the boss
```

### DifficultyData
```
id : enum {STANDARD, ASSIST, ...}
assist : { auto_block, slow_boss, extra_coyote, infinite_dash_charges, ... }
global_mults : { player_hp, hazard_damage }
```

### LoadoutPayload (runtime-built, not authored)
```
# Produced by LevelTransitionManager at the handoff (CORE_GAMEPLAY_LOOP.md)
rank : enum {S,A,B,C,D}
final_score : int
bonus_start_health : int            # from score/rank
armor_points : int                  # from shields held
meter_prefill : float               # from Overcharge / special collectibles
unlocked_moves : [StringName]       # from secrets / high-route clear
first_strike : bool                 # from fast completion
boss_advantage : BossData.disadvantage_scaling or null   # from poor score
```

### SaveData
```
version : int
settings : { input_map, audio, assists, DifficultyData id }
unlocks : { levels:[], moves:[], characters:[] }
records : { per_level: { best_score, best_rank, best_time, secrets_found, full_high_clear:bool } }
leaderboard_cache : optional
```

## 4. Authoring Flows (what adding content looks like)

| To add… | Author… | Touch code? |
| --- | --- | --- |
| A **power-up** | A `PowerUpData` `.tres` + an icon/VFX; drop a pickup scene referencing it | No (if effect_type exists) |
| A **hazard** variant | A `HazardData` `.tres` on an existing hazard scene | No |
| A **move** | A `MoveData` `.tres` + animation | No (uses Combat SM) |
| A **boss** | A `BossData` `.tres` (moves, arena, ai_profile) + arena scene | No (uses Combat SM) |
| A **level** | A `LevelData` `.tres` + a level scene built from components | No (uses managers) |
| A new **effect_type** / state | — | **Yes** (extend the relevant system) |

The boundary is clear: *new content = data*; *new kinds of behavior = code*. Most content growth is pure data.

## 5. Relationships (how the data connects)

```
LevelData ──boss_ref──▶ BossData ──move_set/super──▶ MoveData
    │                        │
    │ score_rules_ref        └─ arena_scene_ref ──▶ (Arena scene)
    ▼
ScoreRuleData                 PowerUpData ◀── pickups in Level
    ▲                         HazardData  ◀── hazards in Level
    │
ScoreManager reads ScoreRuleData every run

Run produces ──▶ LoadoutPayload ──▶ applied to PlayerFighter in Arena
SaveData persists records/unlocks across runs
```

## 6. Versioning & Safety

- `SaveData.version` enables migration as schemas evolve.
- Resources validated on load (missing refs → clear editor error, never a silent gameplay bug).
- Default/fallback Resources exist for every type so a half-authored level still boots during development.

## 7. Open Questions

> See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) for the master register. Grep **`❓ OPEN QUESTION`**.

- ❓ OPEN QUESTION [Q-DATA-01] Should `rank_thresholds` live on `LevelData` or `ScoreRuleData`? (Leaning: default in ScoreRuleData, optional per-level override on LevelData.)
- ❓ OPEN QUESTION [Q-DATA-02] Represent `SegmentData` as real Resources, or keep segments purely as scene structure? (Leaning: optional lightweight Resource; not required for a level to function.)
- ❓ OPEN QUESTION [Q-DATA-03] `AiProfileData` as its own Resource vs. inline on `BossData`? (Leaning: separate, reusable across bosses and rematches.)
