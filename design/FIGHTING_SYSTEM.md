# FIGHTING_SYSTEM.md

> **Status: DESIGN PHASE ONLY.**

## 1. Philosophy

The level's climax is a **1v1 duel** inspired by Street Fighter and Mortal Kombat — but **approachable, not competitive-execution**. The target player is a platformer fan, not a fighting-game specialist.

Design goals:
- **Low execution floor:** no motion inputs (no quarter-circles), no frame-perfect links. Specials are single-button + direction, gated by meter.
- **High read ceiling:** depth comes from spacing, timing, resource management, and reading the opponent — not from input difficulty.
- **Short and punchy:** fights last ~45–120 seconds. This is a climax, not a separate game.
- **The run is in the room:** platforming performance visibly shapes the fight's opening state (Pillar 5).

## 2. Control Scheme

A fighting-focused control set that reuses the platformer's muscle memory where sensible:

| Action | Input (gamepad) | Notes |
| --- | --- | --- |
| Move / approach | Left stick / D-pad | Walk, dash-step |
| Jump | A / South | Reused from platforming |
| **Light attack** | X / West | Fast, low damage, combo starter |
| **Heavy attack** | Y / North | Slow, high damage, armor-breaking |
| **Jump attack** | Jump + attack | Aerial approach / anti-ground |
| **Block** | Hold back toward opponent-facing, or dedicated LB/L1 | "Hold back to block" is classic & readable; a dedicated button is an accessibility option |
| **Grab / throw** | B / East (near opponent) | Beats block; the block-breaker |
| **Dodge / roll** | RB/R1 or double-tap direction | Short i-frames; the mobility/defense verb |
| **Special** | RT/R2 + direction | Costs meter; character-specific |
| **Super** | LT+RT / both triggers | Costs full super meter; big payoff |

> **No motion inputs.** Every special is "modifier + direction/button." This is the core accessibility decision.

## 3. The Combat Triangle (readable depth)

Depth without execution comes from a clean rock-paper-scissors core:

```
         ATTACK
        /      \
   beats        loses to
      /            \
  GRAB  ◄── beats ── BLOCK
```

- **Attack** beats **Grab** (hit them before they grab).
- **Grab** beats **Block** (throw a turtling opponent).
- **Block** beats **Attack** (absorb/reduce incoming).
- **Dodge** sidesteps the triangle for a cost (limited, i-frames, whiff-punishable).

Every exchange is a legible mind-game on top of spacing. A new player understands it in one sentence; an expert plays the yomi.

## 4. Combos (simple, satisfying)

- **Auto-link light chains:** mashing/rhythm-tapping Light produces a short, forgiving auto-combo (e.g., L-L-L → launcher). No precise timing required to get *something*.
- **Branch points:** skilled players branch (L-L → Heavy, or L-L → Special) for more damage/positioning.
- **No 20-hit touch-of-death.** Combos are short (2–5 hits), high-impact, and end in a knockdown or reset to neutral.
- **Combo scaling:** damage scales down within a combo so long strings don't trivialize fights.

The intent: *everyone* can do a cool-looking combo; experts optimize routing and resource use.

## 5. Resources

### Health
- A single health bar per fighter (classic FG). Platforming performance adjusts the **player's** starting health (`PROGRESSION.md`).

### Super Meter
- Builds by dealing/taking damage and by landing specials.
- Spends on **Super** (big damage/reversal) and possibly on an **EX** enhance of a special.
- **Platforming can pre-fill meter** (Overcharge power-up, special collectibles) — a direct run→fight payoff.

### Armor (from shields)
- Shields held at stage finish become **armor points**: each absorbs one hit's chip/knockback or lets you power through one attack during the fight (tune which). A visible pip UI.

### Dodge stamina (optional)
- To prevent dodge-spam: a small regenerating stamina gates dodges. Needs playtest; may be cut for simplicity.

## 6. Neutral, Pressure, and Reads

The fight plays out in recognizable FG phases, simplified:
- **Neutral:** spacing and footsies; whiff-punish with Heavy; poke with Light.
- **Pressure:** on knockdown, apply mix-ups (strike vs. grab) — the combat triangle.
- **Defense:** block, dodge, or reversal super.

Boss AI is tuned to be **readable**: it telegraphs specials (wind-up tells, matching the level's teaching) so the player can learn to punish. Difficulty comes from the boss's *kit identity*, not from inhuman reactions.

## 7. How Platforming Shapes the Fight (the payoff)

Summarized here; full mapping in `PROGRESSION.md`.

| Platforming performance | Combat effect |
| --- | --- |
| High score / rank | More starting health |
| Shields held | Armor points |
| Special collectibles | Pre-filled / larger super meter |
| High-route clear | Unlocks a **bonus attack** or EX option |
| Secrets found | Unlock **alternate moves** |
| Fast completion | **First-strike / approach-speed** edge at round start |
| Poor score | Boss gets an advantage (more health / faster / extra armor) |

The player should *feel* their run the instant the fight begins — the loadout summary (`CORE_GAMEPLAY_LOOP.md` handoff) tells them exactly what they earned.

## 8. Round Structure

- **Single round or best-of-1 with a comeback mechanic**, to keep fights short and climactic. (Leaning: single decisive round; the "comeback" is the Super meter.)
- A disadvantaged (low-score) player can still win with good defense and reads — the loadout tilts odds, it doesn't predetermine the result.
- **Fight retry** keeps the loadout (`CORE_GAMEPLAY_LOOP.md`), so losing the fight doesn't force a stage re-run.

## 9. Accessibility & Approachability

- **Hold-back-to-block + optional block button.**
- **Auto-combos** so everyone lands something cool.
- **No motion inputs.**
- **Readable telegraphs** on boss specials.
- **Input buffering** and generous cancel windows (the FG equivalent of coyote time).
- Optional **assist toggles** (auto-block scrub, slower boss) for the lowest skill tier — never required.

## 10. Data-Driven Hook

- **`CharacterData`** / **`BossData`** define movesets, health, meter rules, and which moves exist.
- **`MoveData`** defines each attack: damage, startup/active/recovery (frame data as tuning values), hit/block properties, meter cost, cancel options, VFX/SFX refs, and the input that triggers it.
- The **Combat State Machine** (`GODOT_ARCHITECTURE.md`) consumes these; adding a new fighter or move is authoring data, not rewriting the engine. See `DATA_MODEL.md`.

## 11. Open Questions

- Single health bar vs. 2-round match? (Leaning: single round for pace; revisit if fights feel too swingy.)
- Does the player use the *same* runner character in the fight, or a combat persona? (Leaning: same character, reinforcing the two-fantasies-one-character identity in `GAME_DESIGN.md`.)
- Is dodge stamina worth the added complexity? (Playtest.)
- How hard should the boss AI read/punish? Must stay "readable," never "oppressive."
