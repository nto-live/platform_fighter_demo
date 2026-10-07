# DESIGN_ANALYSIS.md

> **Status: DESIGN PHASE ONLY.** An internal review of the design doc set — structure, strengths, gaps, inconsistencies, and risks. This is a critique document, not a spec; the systems it reviews live in the other `/design` files.

## 1. Purpose

A pass over all system docs to assess coherence before implementation. Findings are grouped into: documentation structure, strengths, gaps, inconsistencies (with resolutions), and ranked risks. Items marked **[RESOLVED]** have been addressed by edits to the relevant doc; items marked **[OPEN]** remain for later decision.

## 2. Documentation Structure (as reviewed)

The set flows vision → systems → build/tech → meta → synthesis:

- **Vision:** `GAME_DESIGN`, `CORE_GAMEPLAY_LOOP`
- **Platforming systems:** `MOVEMENT_SYSTEM`, `THREE_PATH_LEVEL_DESIGN`, `SCORING_SYSTEM`, `POWERUPS`, `DAMAGE_AND_CHECKPOINTS`, `PUZZLE_DESIGN`, `PLATFORMING_COMBAT`
- **Fighting systems:** `FIGHTING_SYSTEM`, `BOSS_DESIGN`
- **Build & tech:** `LEVEL_DESIGN_GUIDE`, `GODOT_ARCHITECTURE`, `DATA_MODEL`
- **Meta:** `PROGRESSION`, `REPLAYABILITY`, `UI_AND_PLAYER_FEEDBACK`
- **Synthesis:** `PROOF_OF_CONCEPT_LEVEL`

Cross-referencing is consistent, terminology is shared, and every doc carries an honest "Open Questions" section. The five pillars are used as live decision tie-breakers, not decoration.

## 3. Strengths

- **Tight integration via score.** Route choice + speed + survival → score → rank → fight loadout. The three parts reinforce each other instead of sitting side by side.
- **Layered anti-frustration gate.** Visible meter → mid-level banking → low minimum gate → last-chance challenge → opt-in disadvantaged fight. Answers the brief's "wasted 10 minutes" warning from several directions.
- **Readability as a hard rule.** Each hazard declares exactly one consequence; "drops look survivable / deaths look lethal" is a mandate.
- **God-script-resistant architecture.** Thin autoload managers + EventBus + data-driven Resources, with a clean "content = data, behavior = code" boundary.
- **Scope discipline.** Movement kit explicitly defers/rejects mechanics with reasons; the POC proves only the risky claims.

## 4. Gaps Found

| # | Gap | Status |
| --- | --- | --- |
| G1 | No design for how the player fights/defeats enemies *during platforming* (scoring and movement both assume it) | **[RESOLVED]** — added `PLATFORMING_COMBAT.md` |
| G2 | "High-route clear" referenced by `PROGRESSION`/`FIGHTING` but never defined | **[RESOLVED]** — defined in `THREE_PATH_LEVEL_DESIGN.md` §11 |
| G3 | No dedicated audio/music design doc | **[OPEN]** — acceptable for design phase; `UI_AND_PLAYER_FEEDBACK` has a stub |
| G4 | No narrative/world framing (who is the runner, why these bosses) | **[OPEN]** — intentional mechanics-first posture; revisit for shipping |
| G5 | Balance spread asserted, not demonstrated (only one worked score example) | **[OPEN]** — needs LOW-run and S-run worked examples once numbers exist |

## 5. Inconsistencies Found (and resolutions)

| # | Inconsistency | Resolution |
| --- | --- | --- |
| I1 | **Health model**: platforming uses 3 pips (`DAMAGE_AND_CHECKPOINTS`), combat uses a single bar (`FIGHTING_SYSTEM`), and `CharacterData` only had `combat_health` | **[RESOLVED]** — stated the two-model relationship explicitly in all three docs; added `platforming_health` to `CharacterData` and documented how rank scales the combat bar |
| I2 | **Multiplier-reset rule**: `SCORING` wanted nuanced per-consequence behavior, but `ScoreRuleData` exposed a single `reset_on_damage` boolean (redundant with `HazardData.multiplier_effect`) | **[RESOLVED]** — made per-hazard `multiplier_effect` authoritative; removed the contradictory boolean from `ScoreRuleData`; `SCORING` now points at the hazard taxonomy |
| I3 | **Score Multiplier stacking**: `POWERUPS` catalog said "stacks on top of the live multiplier" while its own open question still debated base-only vs. whole-product | **[RESOLVED]** — committed to multiplying **base points only**; catalog and open question reconciled |
| I4 | **"best-of-1" wording** in `FIGHTING_SYSTEM` §8 (a best-of-1 is just one round) | **[RESOLVED]** — reworded to "single decisive round" |
| I5 | **Dodge vs. combat triangle** diagram could imply dodge is part of the RPS | **[RESOLVED]** — clarified dodge sits outside/around the triangle |

## 6. Ranked Risks (unchanged by edits — these are design-inherent)

1. **The handoff carries the whole concept.** If the fight doesn't *feel* caused by the run, the game splits into two mini-games. Hardest thing to validate on paper; the POC must prove it.
2. **Authoring cost of three interwoven routes.** One level ≈ three co-dependent levels that must weave, each drop needing an authored catch-surface + climb-back. Biggest *production* risk.
3. **Two full control schemes / two state machines.** Doubles feel-tuning and test surface.
4. **"Meaningful but not deterministic" loadout balance.** A narrow band; needs heavy playtest. Asserted, not proven.
5. **Drop-vs-death legibility at speed.** The core tension collapses into frustration if players can't instantly distinguish a survivable drop from a lethal pit while moving fast.

## 7. Recommendation

The design is internally consistent and unusually complete for a first pass. The substantive inconsistencies (I1–I3) and the two real gaps (G1–G2) have been addressed by targeted doc edits (see each doc's changelog note). The remaining open items (G3–G5, and all "Open Questions") are appropriate to defer to tuning/playtest.

**Implementation-first order remains as recommended in `GAME_DESIGN` / chat summary:** movement feel → EventBus+GameState shell → RouteManager + drop/catch model → ScoreManager + qualification meter → handoff/loadout → combat + VOLT. The POC (`PROOF_OF_CONCEPT_LEVEL.md`) should focus on validating Risk #1 and Risk #5 above all.

## 8. Change Log (edits made as a result of this review)

- Added `PLATFORMING_COMBAT.md` (G1).
- `THREE_PATH_LEVEL_DESIGN.md`: added §11 defining "high-route clear" (G2).
- `DAMAGE_AND_CHECKPOINTS.md`, `FIGHTING_SYSTEM.md`, `DATA_MODEL.md`: reconciled the health model (I1).
- `SCORING_SYSTEM.md`, `DATA_MODEL.md`: made per-hazard `multiplier_effect` authoritative; removed `reset_on_damage` (I2).
- `POWERUPS.md`: committed Score Multiplier to base-only (I3).
- `FIGHTING_SYSTEM.md`: "best-of-1" → "single decisive round" (I4); dodge/triangle clarification (I5).
- `README.md`: index updated to include new docs.
