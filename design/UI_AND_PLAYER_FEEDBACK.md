# UI_AND_PLAYER_FEEDBACK.md

> **Status: DESIGN PHASE ONLY.** Layouts are described, not pixel-specified.

## 1. Feedback Philosophy

The game lives or dies on **readability and honesty** (Pillar 4). The player must *always* understand:
- **Am I on pace for the boss?** (qualification meter)
- **How well am I doing right now?** (live multiplier, score popups)
- **What's threatening me, and will it drop me or kill me?** (hazard telegraphs — `DAMAGE_AND_CHECKPOINTS.md`)
- **What did my run earn me for the fight?** (loadout summary)

Every number has a visible cause. Nothing about scoring or qualification is hidden or arbitrary.

## 2. Platforming HUD

A clean CanvasLayer of independent widgets, each driven by EventBus signals (`GODOT_ARCHITECTURE.md`). No widget reads gameplay objects directly.

```
┌──────────────────────────────────────────────────────────────┐
│ [SCORE 12,340]              ROUTE: ◭ MIDDLE        TIME 1:04   │
│ ×3.2 ▮▮▮▮▮▯  (multiplier, decaying)                            │
│                                                                │
│ QUALIFY ▮▮▮▮▮▮▮▮▯▯▯▯  D | C | B | A | S   ← ticks + "BEHIND"   │
│                                                                │
│ ♥♥♡  🛡🛡    [⚡ Speed 4s] [2× 7s]   (health, shields, powerups)│
└──────────────────────────────────────────────────────────────┘
```

### Widgets
| Widget | Shows | Behavior |
| --- | --- | --- |
| **Score** | Running total | Counts up smoothly; floating popups at the source |
| **Live multiplier** | Current ×, with a decay bar | Big, animated; pulses on growth, flashes red on decay/reset |
| **Qualification meter** | Score vs boss gate; rank ticks (D/C/B/A/S) | Fills with score; **visibly dips** on route-drop/multiplier loss; label ahead/on-track/behind |
| **Route indicator** | Current tier (HIGH/MIDDLE/LOW) | Changes instantly on route change; subtle color language |
| **Timer** | Elapsed vs par | Turns gold when beating par |
| **Health** | HP pips | Pip breaks with a hit flash |
| **Shields** | Shield icons | Shows stack; shatter animation on break |
| **Power-up tray** | Active power-ups + countdowns | Each with icon + timer; warning flash before expiry |

## 3. The Qualification Meter (the signature UI)

The most important widget — it's the anti-frustration guarantee (`PROGRESSION.md`).
- A horizontal bar from 0 to "max expected," with **tick marks at each rank boundary** and a bold mark at the **boss minimum gate**.
- Fills as score accrues; **drains visibly** when the player drops a route or loses multiplier — this is the "I screwed up but can save it" feedback made literal.
- A short text/icon state: **BEHIND** (red), **ON TRACK** (white), **AHEAD** (gold).
- Optional **pace projection** marker: a faint ghost tick estimating final rank at current pace.
- At **junctions**, a brief emphasized readout: "BANKED 8,200 — ON TRACK FOR B."

## 4. Floating Score Popups

Every point source spawns a small popup at its location: `+50`, `+200 SECRET`, `×COMBO!`, `NEAR MISS +25`. This makes cause→effect for scoring **impossible to miss** and teaches the scoring model implicitly.

## 5. Route & Drop Feedback

- **Route change:** a quick, non-intrusive banner/flash + the route indicator updates. HIGH entry feels rewarding (gold shimmer); a drop feels like a demotion (a brief downward whoosh + meter dip) — *but never like a death*.
- **Drop vs death clarity:** drops have a distinct "demotion" sound/VFX; deaths have a distinct "retry" cut. The player must never confuse the two.

## 6. Hazard Telegraphs

From `DAMAGE_AND_CHECKPOINTS.md` — the honesty contract:
- INSTANT-DEATH hazards: aggressive, unmistakable visuals (spikes glint, lasers hum+flash, lethal pits read as bottomless/dark).
- DROP / KNOCK-TO-ROUTE zones: look survivable; a visible catch-surface below.
- Consistent **color/shape language** across the game so players learn to read hazards at a glance.

## 7. Score Evaluation Screen

After the stage (`CORE_GAMEPLAY_LOOP.md` → SCORE_EVAL):
```
          ─ STAGE CLEAR ─
  Base × Multiplier ............ 9,800
  Time Bonus ................... 1,500
  Shield Bonus ................. 600
  Route Difficulty ............. 1,200
  No-Hit ....................... —
  Secrets (2/4) ................ 700
  Style (peak ×6.1) ............ 400
  ─────────────────────────────
  RUN SCORE .................... 14,200
             ┌─────────┐
             │  RANK B │   (stamp animation)
             └─────────┘
```
- Lines tally in sequence (skippable after first view).
- Rank stamp lands with a satisfying beat.

## 8. Loadout Summary Screen

The emotional bridge to the fight (Pillar 5). Each line references **a thing the player did**:
```
          ─ ENTERING THE ARENA ─
  Rank B .................. +Health
  2 Shields held ......... Armor ▮▮
  High route (Seg 2) ..... Bonus Attack unlocked
  Secret found ........... Alt move: Rising Dash
  (Par missed) ........... No speed edge
```
- Loadout icons visually attach to the player in the handoff transition (armor shimmer, meter pre-fill animates).
- Short, skippable after first view per level.

## 9. Fighting HUD

```
┌──────────────────────────────────────────────────────────────┐
│ PLAYER ▮▮▮▮▮▮▮▮▯▯  ARMOR ▮▮        VOLT ▮▮▮▮▮▮▮▮▮▮▮▮          │
│ SUPER ▮▮▮▮▮▯ (prefilled)                        SUPER ▮▮▯▯▯    │
└──────────────────────────────────────────────────────────────┘
```
- Classic FG framing: health bars top corners, super meters below.
- **Armor pips** (from shields) clearly shown and visibly consumed.
- Pre-filled meter is visually distinct so the player sees their run's payoff.
- Boss special wind-ups get a readable on-screen tell (`FIGHTING_SYSTEM.md`) — difficulty is in the mix-up, not in hidden information.

## 10. Accessibility

- **Colorblind-safe** route/hazard language (shape + color, never color alone).
- **Scalable HUD** and optional reduced-motion mode (fewer flashes).
- **Input remapping** and the assist toggles from `DifficultyData` surfaced clearly in options.
- Readable fonts; no critical info conveyed by audio alone (visual + audio redundancy for key cues).
- **Note:** full accessibility validation requires manual testing with assistive technologies and expert review — these are design intents, not a compliance claim.

## 11. Audio Feedback (brief)

Driven by AudioManager (`GODOT_ARCHITECTURE.md`):
- Distinct stings for: collectible, premium collectible, secret, multiplier-up, multiplier-reset, shield-break, drop, death-retry, checkpoint/bank, rank stamp.
- Music shifts subtly by route tier (higher = more intense) and ducks for the handoff/fight.

## 12. Open Questions

> See [`OPEN_QUESTIONS.md`](OPEN_QUESTIONS.md) for the master register. Grep **`❓ OPEN QUESTION`**.

- ❓ OPEN QUESTION [Q-UI-01] Is a pace-projection marker helpful or anxiety-inducing? (Playtest; make it optional.)
- ❓ OPEN QUESTION [Q-UI-02] How minimal can the platforming HUD get before readability suffers? (Leaning: a "clean HUD" toggle for veterans.)
- ❓ OPEN QUESTION [Q-UI-03] Should floating popups be togglable for speedrunners who find them noisy? (Leaning: yes.)
