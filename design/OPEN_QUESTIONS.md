# OPEN_QUESTIONS.md

> **Status: DESIGN PHASE ONLY.** A single register of every unresolved design question across the `/design` docs. Each question has a stable ID so it can be referenced, tracked, and checked off. The source docs keep their own "Open Questions" sections; this file is the master index.

## How to read this

- **ID** — stable reference (e.g. `Q-MOVE-01`). Won't be renumbered as questions resolve.
- **Priority:**
  - 🔴 **P1 — Foundational.** Affects architecture, scope, or core feel. Should be decided before/early in implementation.
  - 🟡 **P2 — Shaping.** Affects a system's identity but not the whole build. Decide during that system's design/first pass.
  - 🟢 **P3 — Tuning / playtest.** Genuinely wants player data; fine to defer.
- **Status:** `OPEN` · `LEANING (x)` = a provisional default is recorded · `RESOLVED` (kept for history).
- Inline in the source docs, unresolved items are tagged **`❓ OPEN QUESTION [Q-ID]`** and resolved ones struck through with a **Resolved** note.

> Tag convention for grepping: search **`❓ OPEN QUESTION`** to jump to any open item in the docs, or search a specific ID (e.g. `Q-FEEL-01`).

---

## 🔴 P1 — Foundational (decide first)

| ID | Question | Source doc | Status |
| --- | --- | --- | --- |
| `Q-FEEL-01` | Where on the Sonic-fast ↔ Meat-Boy-precise slider does movement sit? Drives accel, air control, boost decay, high-route punishment. | `MOVEMENT_SYSTEM.md` | **OPEN** (awaiting user) |
| `Q-HEALTH-01` | Platforming health: keep the 3-pip pool, or go near-one-hit "Meat Boy" purity with shields as the only buffer? | `DAMAGE_AND_CHECKPOINTS.md` | LEANING (3 pips) |
| `Q-ART-01` | Art direction: pixel art or clean vector? Affects tooling, animation pipeline, and hazard readability (Pillar 4). | `GAME_DESIGN.md` §7 | **OPEN** (awaiting user) |
| `Q-SCOPE-01` | Demo scope confirmed = one level + VOLT, single character. Any smaller movement-only first slice desired? | `GAME_DESIGN.md` §6 | RESOLVED (demo scope accepted; design-only for now) |
| `Q-CHAR-01` | Same character across both halves vs. a combat persona? | `GAME_DESIGN.md` §4 | RESOLVED (same character; roster is a planned extension) |

## 🟡 P2 — Shaping (decide during the relevant system's pass)

| ID | Question | Source doc | Status |
| --- | --- | --- | --- |
| `Q-MOVE-01` | Dash charges shared with the fight, or platforming-only? | `MOVEMENT_SYSTEM.md` | LEANING (platforming-only) |
| `Q-MOVE-02` | Does boosted speed persist through a spring, or does the spring set a fixed launch? | `MOVEMENT_SYSTEM.md` | LEANING (spring sets a floor, doesn't cap) |
| `Q-MOVE-03` | Grind-rail control: auto-speed vs. player-modulated? | `MOVEMENT_SYSTEM.md` | LEANING (player-modulated within a band) |
| `Q-DMG-01` | Multiplier reset on every hit, or only shield-break/knockdown? | `DAMAGE_AND_CHECKPOINTS.md` | RESOLVED (per-hazard `multiplier_effect`; soft=dip, knockdown/death=reset) |
| `Q-PUP-01` | Score Multiplier: base-only vs. whole product? | `POWERUPS.md` | RESOLVED (base points only, before live multiplier) |
| `Q-PUP-02` | How many forced-choice pedestals per level before it feels gimmicky? | `POWERUPS.md` | LEANING (2–3, at junctions) |
| `Q-FIGHT-01` | Same character in the fight vs. combat persona? | `FIGHTING_SYSTEM.md` | RESOLVED (same character — see `Q-CHAR-01`) |
| `Q-FIGHT-02` | Single decisive round vs. 2-round match? | `FIGHTING_SYSTEM.md` | LEANING (single round) |
| `Q-FIGHT-03` | Is dodge stamina worth the added complexity? | `FIGHTING_SYSTEM.md` | LEANING (optional; may cut) |
| `Q-FIGHT-04` | How hard should boss AI read/punish while staying "readable"? | `FIGHTING_SYSTEM.md` | OPEN |
| `Q-BOSS-01` | Do bosses ever have a short platforming-flavored arena phase, or stay a pure duel? | `BOSS_DESIGN.md` | LEANING (pure duel) |
| `Q-PROG-01` | Should the Last-Chance Challenge cap rank (e.g. at C)? | `PROGRESSION.md` | LEANING (yes — gets you in, not a good loadout) |
| `Q-PROG-02` | HP scaling curve by rank — how much does S vs D swing? | `PROGRESSION.md` | OPEN (needs playtest to stay non-deterministic) |
| `Q-PCMB-01` | Do enemies drop loot on defeat, or is loot purely placed? | `PLATFORMING_COMBAT.md` | LEANING (a few drop; most placed) |
| `Q-PCMB-02` | Are turrets/lobbers ever defeatable (reflect), or avoid-only? | `PLATFORMING_COMBAT.md` | LEANING (avoid-only in demo) |
| `Q-ARCH-01` | EventBus signals vs. a message queue for same-frame ordering? | `GODOT_ARCHITECTURE.md` | LEANING (signals + careful ordering) |
| `Q-ARCH-02` | RouteManager tier from volumes only, or altitude fallback too? | `GODOT_ARCHITECTURE.md` | LEANING (volumes authoritative, altitude sanity check) |
| `Q-ARCH-03` | Shared MovementController across modes, or separate combat mover? | `GODOT_ARCHITECTURE.md` | LEANING (shared body, different profiles) |
| `Q-DATA-01` | `rank_thresholds` on `LevelData` or `ScoreRuleData`? | `DATA_MODEL.md` | LEANING (ScoreRuleData default, LevelData override) |
| `Q-DATA-02` | `SegmentData` as real Resources or pure scene structure? | `DATA_MODEL.md` | LEANING (optional lightweight Resource) |
| `Q-DATA-03` | `AiProfileData` as its own Resource vs. inline on `BossData`? | `DATA_MODEL.md` | LEANING (separate, reusable) |
| `Q-PUZ-01` | Do solved route-change mechanisms persist across death-retry within a segment? | `PUZZLE_DESIGN.md` | LEANING (yes; reset only on segment restart) |

## 🟢 P3 — Tuning / playtest (fine to defer)

| ID | Question | Source doc | Status |
| --- | --- | --- | --- |
| `Q-DMG-02` | Respawn resource-restoration curve (sting vs. stakes). | `DAMAGE_AND_CHECKPOINTS.md` | OPEN (playtest) |
| `Q-PUP-03` | Does Overcharge risk (lose on death) feel exciting or punishing? | `POWERUPS.md` | OPEN (playtest) |
| `Q-PUP-04` | Score Multiplier exact magnitudes. | `POWERUPS.md` | OPEN (tune) |
| `Q-PUZ-02` | Signposting: explicit arrows vs. pure environmental read? | `PUZZLE_DESIGN.md` | LEANING (color language, minimal arrows) |
| `Q-PUZ-03` | "Puzzle solved" popup, or let route/score speak for itself? | `PUZZLE_DESIGN.md` | LEANING (subtle popup for secrets only) |
| `Q-REPLAY-01` | Ghosts: per-route or overall-best only? | `REPLAYABILITY.md` | LEANING (overall best + friend ghosts) |
| `Q-REPLAY-02` | Surface a per-level completion %? | `REPLAYABILITY.md` | LEANING (yes) |
| `Q-REPLAY-03` | Weekly challenges in v1 or post-launch? | `REPLAYABILITY.md` | LEANING (post-launch) |
| `Q-UI-01` | Pace-projection marker: helpful or anxiety-inducing? | `UI_AND_PLAYER_FEEDBACK.md` | LEANING (optional) |
| `Q-UI-02` | How minimal can the platforming HUD get before readability suffers? | `UI_AND_PLAYER_FEEDBACK.md` | LEANING (clean-HUD toggle) |
| `Q-UI-03` | Togglable floating popups for speedrunners? | `UI_AND_PLAYER_FEEDBACK.md` | LEANING (yes) |
| `Q-POC-01` | One major shortcut enough, or does HIGH need a second? | `PROOF_OF_CONCEPT_LEVEL.md` | OPEN (playtest) |
| `Q-POC-02` | Overcharge-carry risk read in this level. | `PROOF_OF_CONCEPT_LEVEL.md` | OPEN (playtest) |
| `Q-POC-03` | Are 2 junctions enough, or do we need 3? | `PROOF_OF_CONCEPT_LEVEL.md` | LEANING (2 for demo length) |
| `Q-POC-04` | Switch-sequence rhythm mirroring Volt — too subtle? | `PROOF_OF_CONCEPT_LEVEL.md` | OPEN (playtest covert-practice hypothesis) |
| `Q-BOSS-02` | Should the boss comment on the player's route taken? | `BOSS_DESIGN.md` | LEANING (nice-to-have voice lines) |
| `Q-BAL-01` | Prove the score spread with worked LOW-run and S-run examples. | `SCORING_SYSTEM.md` / `DESIGN_ANALYSIS.md` G5 | OPEN (draft when numbers exist) |

## Deferred gaps (not questions, but tracked)

| ID | Item | Source | Status |
| --- | --- | --- | --- |
| `G3-AUDIO` | No dedicated audio/music design doc. | `DESIGN_ANALYSIS.md` | DEFERRED |
| `G4-STORY` | No narrative/world framing. | `DESIGN_ANALYSIS.md` | DEFERRED |
| `G5-BALANCE` | Balance spread only shown with one worked example. | `DESIGN_ANALYSIS.md` | DEFERRED (see `Q-BAL-01`) |

---

## Awaiting-user shortlist

The questions that most need a human decision (not playtest) are:
- `Q-FEEL-01` — the feel slider.
- `Q-HEALTH-01` — 3 pips vs. near-one-hit.
- `Q-ART-01` — pixel vs. vector.

Everything else has a provisional lean recorded above and can proceed on that default, flagged "revisit at playtest."
