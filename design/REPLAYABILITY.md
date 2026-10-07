# REPLAYABILITY.md

> **Status: DESIGN PHASE ONLY.**

## 1. The Replay Thesis

> **First run = exploration. Later runs = mastery.** A single level should transform from "where do I even go?" into "can I hold the high route end-to-end and S-rank it?"

This arc is the whole point of the three-route structure. Because the routes are chosen *live* (Pillar 3), a player naturally graduates from survival (LOW) to expression (HIGH) across repeated plays of the *same content* — no new levels required to keep improving.

## 2. The Mastery Curve of One Level

```
Run 1   : Explore. Mostly MIDDLE/LOW. Discover the shape. See things you can't reach.
Run 2-3 : Learn connectors. Find a secret. Climb once. Rank C→B.
Run 4-6 : Chain momentum. Hold HIGH through a segment. Beat par in places. Rank B→A.
Run 7+  : Full high-route clear. Max multiplier. S-rank. Chase the ghost.
```

Each run should reveal *one more thing*: a connector, a secret, a faster line, a cleaner puzzle solution.

## 3. Reasons to Replay (and the systems behind them)

| Reason | Backed by |
| --- | --- |
| Higher score / better rank | Scoring + ranks (`SCORING_SYSTEM.md`, `PROGRESSION.md`) |
| Faster times | Par times, Time Bonus, speedrun timer |
| S-rank hunting | Rank thresholds; S demands near-perfect HIGH runs |
| Discover shortcuts & secret paths | Interconnection toolkit + hidden connectors (`THREE_PATH_LEVEL_DESIGN.md`) |
| Collect hidden items | Premium collectibles, secret-only power-ups (`POWERUPS.md`) |
| Unlock power-ups / moves / characters | Meta progression (`PROGRESSION.md`) |
| Beat personal records | Record keeping + ghosts (Section 5) |
| Complete the **entire high path** | A tracked feat (`SaveData.full_high_clear`) |
| Find alternate routes | See-but-can't-reach design |
| Leaderboards | Online records (Section 6) |
| Better boss loadout | Run quality → combat advantage (Pillar 5) |

## 4. The "See-But-Can't-Reach" Engine

The single biggest replay driver (a Sonic hallmark). Implementation guidance from `THREE_PATH_LEVEL_DESIGN.md`:
- Players **constantly see** collectibles, rails, and shortcuts above them that they can't yet reach.
- Every visible reward has a *real* (hard) access route — the game never teases something unreachable.
- A locked shortcut on run 1 becomes a known target by run 5. The level "opens up" in the player's mind over time.

## 5. Ghosts & Personal Bests

- **Record a ghost** of the player's best run (position/time), replayable as a translucent racer.
- "Chase your ghost" turns solo play into a self-competition — a cheap, powerful mastery loop.
- Per-level bests (score, time, rank, secrets, full-high-clear) are shown on `LEVEL_SELECT`.

## 6. Leaderboards (shipping feature, not POC)

- Per-level score and time leaderboards.
- Separate boards for **assisted** vs. **unassisted** runs (`PROGRESSION.md`) to keep competition fair.
- Optional **weekly challenge** levels/modifiers for a recurring reason to return.
- Ghost download of top runs (watch how the best players route the high path).

## 7. Discovery Rewards (first-time dopamine)

Replay also means *first* play must plant hooks:
- Finding a secret plays a distinct sting + popup; it's logged as "X/Y secrets."
- Seeing the HIGH route zoom past above you on a LOW run plants aspiration.
- The boss tease at `LEVEL_INTRO` and Volt's taunts ("too slow?") bait a better run.

## 8. Challenge Layers (optional, late)

For players who exhaust the base mastery curve:
- **No-hit / no-death** medals per level.
- **High-route-only** challenge (fall = fail) for experts.
- **Rival boss variants** unlocked by high-rank entries (`BOSS_DESIGN.md`).
- **Modifier mutators** (low gravity, speed-only, mirrored level) for variety.

## 9. How Replay Interacts With Fairness

- Replays are *always* for a better outcome, never forced by frustration (`PROGRESSION.md` soft gate).
- Losing a fight → retry the fight (keep loadout) *or* replay the stage for a better loadout — the player's choice (`CORE_GAMEPLAY_LOOP.md`).
- Nothing is permanently missable in a run — collectibles reset each run, so replay is pure upside.

## 10. Metrics to Watch (for later tuning)

- Average runs-to-first-S per level (is the mastery curve too steep/shallow?).
- % of players who ever hold a full high-route clear.
- Secret discovery rate on runs 1 / 3 / 5.
- Replay rate after a win (are players chasing better ranks, or moving on?).

## 11. Open Questions

- Ghosts: per-route ghosts, or just overall-best? (Leaning: overall best + optional friend ghosts.)
- Do we surface a per-level "completion %" (routes seen, secrets, feats) to drive 100%-ing? (Leaning: yes, strong completionist hook.)
- Weekly challenges: in-scope for v1 or post-launch? (Post-launch.)
