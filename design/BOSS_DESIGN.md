# BOSS_DESIGN.md

> **Status: DESIGN PHASE ONLY.**

## 1. Philosophy

Bosses are **characters, not giant platforming obstacles.** Each is a 1v1 opponent in the fighting arena (`FIGHTING_SYSTEM.md`) with personality, identity, and a kit that *rhymes with the level that preceded it.*

Two mandates:
1. **The level teaches the boss.** Mechanics stressed during platforming are the same mechanics the boss weaponizes. The stage is covert practice.
2. **Each boss is a reason to replay the level a specific way.** Beating a boss cleanly rewards having learned the stage's lesson.

## 2. What Every Boss Has

| Attribute | Purpose |
| --- | --- |
| **Personality** | A clear attitude/voice; makes the duel feel personal |
| **Fighting style** | A one-line identity ("zoner," "rushdown," "grappler," "trickster") |
| **Unique moves** | 3–5 signature attacks beyond the shared basic kit |
| **Special attack(s)** | Meter or pattern-gated showpiece moves |
| **Super** | A climactic high-damage move with a clear tell |
| **Weakness** | A readable, exploitable pattern — the "solution" to the fight |
| **Arena** | A themed stage tied to the level's biome/motif |
| **Visual identity** | Silhouette, palette, animation signature |
| **Level connection** | The platforming mechanic they embody |

## 3. The Level→Boss Teaching Contract

This is the design glue. For each boss, the preceding level must **teach the counter-skill** the player needs.

| If the level emphasizes… | The boss is… | And the player learned to… |
| --- | --- | --- |
| Projectile/hazard **timing** (`P5 Beat-the-Door`, dodging patterns) | A **zoner** who throws projectiles | Time approaches and dodges through gaps |
| **Wall movement / high mobility** | A hyper-**mobile** fighter who teleports/wall-dashes | Track and intercept fast movement |
| **Sequenced switches** (`P1`) | A boss weak to a **specific hit sequence** | Execute a sequence under pressure |
| **Bounce/reflect** (`P2`, `P9`) | A **counter/reflect** fighter | Bait and punish, not mindlessly attack |
| **Momentum bank-and-spend** (`P10`) | A **charge/armor** bruiser | Commit big hits in the right window |

The player shouldn't be *told* this mapping — they should *feel* prepared because the stage drilled the skill.

## 4. How the Loadout Changes the Fight

Per `PROGRESSION.md` and `FIGHTING_SYSTEM.md`, the player enters with advantages earned in the stage. Boss design must remain **winnable at a disadvantage** (low score) and **satisfying at an advantage** (S rank):
- At a disadvantage: the boss has more health / faster / extra armor; the player can still win via reads and defense.
- At an advantage: the player has armor, pre-filled meter, a bonus move; the fight is a victory lap *they earned* — not a free win (the boss still fights).

**Balance rule:** loadout tilts the odds by a *meaningful but not deterministic* margin. A skilled player on a bad run can still win; a weak player on a great run still has to execute.

## 5. Boss Roster Template (data-authored)

Each boss is a **`BossData`** Resource (`DATA_MODEL.md`). Template fields:

```
BossData
├─ id / display_name
├─ personality_blurb
├─ style_tag            (zoner | rushdown | grappler | trickster | bruiser | ...)
├─ arena_scene_ref
├─ base_health
├─ meter_rules
├─ move_set            [ MoveData... ]  (basics + 3–5 signatures)
├─ special_moves       [ MoveData... ]
├─ super_move           MoveData
├─ weakness_tag         (sequence | whiff-punish | anti-air | corner | ...)
├─ ai_profile_ref       (aggression, telegraph length, punish-readiness)
├─ level_connection     (which level + which taught mechanic)
├─ disadvantage_scaling (how boss buffs when player under-scored)
└─ visual/audio refs
```

New bosses are authored as data + a moveset + an arena scene, reusing the shared Combat State Machine.

## 6. Example Boss (illustrative, for the POC)

Tied to the proof-of-concept level (`PROOF_OF_CONCEPT_LEVEL.md`).

### "VOLT" — the Momentum Rival
- **Personality:** cocky speed-freak; mirrors the player's own momentum fantasy. Taunts you for playing slow.
- **Style:** rushdown with dash-cancels; fast, aggressive, mobile.
- **Level connection:** the POC level emphasizes **momentum, dashing, and rails** — Volt weaponizes dashes and quick approaches.
- **Signature moves:**
  - *Dash Jab* — fast dashing light that closes space.
  - *Rail Slide* — Volt rides a brief ground streak across the arena (punishable on whiff — this is the taught counter).
  - *Spring Kick* — a bouncing anti-air mirroring the level's springs.
- **Special:** *Overdrive* — a short speed-up where his attacks chain faster (telegraphed by a charge flash).
- **Super:** *Light Speed Rush* — a cinematic multi-dash flurry; big tell (he crouches and flashes) so a prepared player can block/dodge.
- **Weakness:** **whiff-punish.** Volt over-commits to dashes; the level taught the player to recognize and punish over-committed momentum. Baiting his Rail Slide and punishing the recovery is the intended "solution."
- **Loadout interplay:** a player who held shields enters with armor to survive Volt's rushdown; an Overcharge carry lets them open with a Super of their own.

## 7. Boss Difficulty & Readability

- **Telegraphs scale with role, not fairness:** even the hardest boss telegraphs its super. Difficulty is in *frequency, mix-ups, and punish-tightness*, never in unreactable attacks.
- **The weakness is discoverable in one fight** and masterable over several — mirroring the stage's exploration→mastery curve (`REPLAYABILITY.md`).
- **AI profiles are data** (`ai_profile_ref`): aggression, telegraph length, how reliably it punishes player mistakes, and how it scales with the player's loadout (disadvantage scaling).

## 8. Progression & Unlocks via Bosses

- Beating a boss unlocks the next level and may unlock **player moves/characters** (`PROGRESSION.md`).
- Beating a boss at **S-rank entry** (great run) can unlock cosmetic/bragging rewards or a harder "rival" variant for replay value.
- Bosses may return as **tougher rematches** in later content, reusing the `BossData` with a buffed `ai_profile`.

## 9. Open Questions

- Do bosses ever have a short **platforming-flavored phase** inside the arena (e.g., Volt forces you to wall-jump off arena walls), or is the fight purely a fighter? (Leaning: keep the fight a pure duel for clarity; express the level's theme through *moves*, not by re-introducing platforming.)
- How many bosses for v1? (Scope question — the POC needs exactly one: Volt.)
- Should the boss comment on *how* the player beat the stage (route taken), for personality? (Nice-to-have; cheap via a few voice lines keyed to route-tier.)
