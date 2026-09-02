# 05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md
## Block Battles — Scoring, Combat, Combo, and Progression

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`
**Document status:** Implementation-level system design

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23. It is the **Scoring doc** and **Combat & Battle Systems doc** referenced by `01_GAMEPLAY_SPECIFICATION.md` Section 25 — it supersedes the placeholder Constants in Gameplay Spec Section 23 with exact formulas, without reordering the fixed Damage Pipeline stage sequence Gameplay Spec Section 9 already established. It consumes `03_BOARD_ENGINE_AND_RULES.md`'s `LinesCleared`/`CellsCleared` events as its sole "Detect Clear" input and `02_SHAPE_LIBRARY.md`'s Shape metadata for Relic condition matching, per those documents' own Section 20/14 cross-references.

---

## 1. Purpose

This document owns the complete causal chain from a resolved Placement to a Run-level reward: **puzzle action → score → damage → relic modification → combo → enemy HP → battle victory → reward → persistent progression.** It is the single authority for every numeric formula in that chain. No other document may define a damage, score, HP, or reward formula; they may only reference this document's outputs (Section 26).

**Ownership boundary:** this document does not own Board occupancy, Placement legality, or Tray generation (owned by `03_BOARD_ENGINE_AND_RULES.md` and `04_SPAWN_RNG_AND_DIFFICULTY.md` respectively) — it only consumes their event outputs as inputs to its own pipeline.

---

## 2. Scoring Philosophy

**DESIGN DECISION:** Score is not a separate currency from Damage — Score and Damage are two read-outs of the same underlying Clear Value (Section 4), diverging only at the final stage (Section 9) where Enemy Defense reduces Damage but never reduces Score. This is deliberate: Score exists to make puzzle-layer skill legible and comparable across Battles/Runs (Balance Metrics, `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 15) independent of which Enemy the player happened to face, while Damage exists to make that same skill *matter* to the specific fight in front of the player (Master Vision Section 3, "Escalation").

This single-source model is what prevents this document from becoming "a health bar above a Block Blast clone" (Primary Objective): the puzzle-layer number and the combat-layer number are structurally the same event, not two parallel systems that happen to both go up when you clear a line.

---

## 3. Combat Philosophy

**DESIGN DECISION:** Combat has exactly one player-controllable input: the quality of Placement decisions that feed the Damage Pipeline (Section 9). There is no secondary combat action, no targeting choice, no timing-based input — every combat outcome traces to a puzzle decision already made under `01_GAMEPLAY_SPECIFICATION.md` Section 5's placement rules. This directly implements the Master Prompt's Design Principle: "Combat should emerge from puzzle excellence," not run alongside it as a separate mini-game.

Three philosophy rules binding every mechanic below:

1. **No damage without a Clear.** Restated from Gameplay Spec Section 8 — Damage is generated only as a result of Line Clears; this document introduces no exception (e.g., no "placement itself deals chip damage" mechanic — see Section 5).
2. **Every modifier is legible.** A player must be able to look at a resulting Damage number and reconstruct which stages contributed (Section 9's fixed order exists precisely so this reconstruction is always possible), satisfying Master Vision Section 5.4 (Satisfying Feedback) and Section 5.6 (Fair Difficulty: "why am I losing" must always be answerable).
3. **Skill compounds, luck doesn't dominate.** Relic and Combo systems (Sections 7, 13–15) are designed so that their ceiling is reached through decision quality (which Relic to pick, how to sequence placements for Combos) rather than through RNG alone — consistent with `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 1's "no manufactured loss / decisions not helplessness" philosophy, extended here to the combat layer.

---

## 4. Core Mathematical Model

**DESIGN DECISION:** All formulas below use exact integer/percentage arithmetic; no formula in this document is expressed as a vague quantity. Every constant referenced is centralized in Section 23.

**Core variables:**

| Symbol | Meaning |
|---|---|
| `L` | Number of lines cleared this Turn (`|filledRows| + |filledCols|` from `03_BOARD_ENGINE_AND_RULES.md` Section 12's `LinesCleared` event). |
| `BD(n)` | Base Damage value for the n-th line cleared within this Turn's clear-set (Section 6). |
| `ClearValue` | `Σ BD(n)` for `n = 1..L` — the summed base value for this Turn, before any modifier stage. |
| `ComboMult` | The Combo Modifier multiplier for this Turn (Section 7). |
| `RelicMod` | The net effect of all active Relic Modifiers for this Turn (Section 9, Relic stage). |
| `ObjMod` | The net effect of the active Objective's Damage Modifier, if any (Section 9, Objective stage; `04_SPAWN_RNG_AND_DIFFICULTY.md` does not define this — it is this document's own Section 16). |
| `EnemyDef` | The active Enemy's Defense value (Section 11). |
| `FinalDamage` | The Damage Event's final value, applied to Enemy HP (Section 9's terminal stage). |
| `ScoreValue` | The Score Event's final value (Section 2) — identical derivation to `FinalDamage` up to and excluding the Enemy Modifier stage (Section 9). |

**Top-level formula (expanded in full in Section 9):**

```text
ClearValue   = Σ BD(n), n = 1..L
StageA       = ClearValue × ComboMult                         (Combo Modifier stage)
StageB       = (StageA + RelicFlat) × RelicMult                (Relic Modifier stage, Section 13)
StageC       = StageB × ObjMod                                  (Objective Modifier stage)
ScoreValue   = round(StageC)                                    (Score terminates here — no Enemy stage)
StageD       = StageC - (StageC × EnemyDef.percent) - EnemyDef.flat   (Enemy Modifier stage)
FinalDamage  = max(0, round(StageD))                             (clamped only here, per Gameplay Spec Section 9's inherited rule)
```

This expands Gameplay Spec Section 9's fixed stage order (`Placement → Detect Clear → Base Damage → Relic Modifiers → Combo Modifiers → Objective Modifiers → Enemy Defense → Final Damage`) into exact arithmetic, while additionally deriving `ScoreValue` from the same chain at the point immediately before the Enemy-specific stage — satisfying Section 2's single-source model. Note the Combo Modifier stage is applied before the Relic Modifier stage in this document's fully-specified pipeline (Section 9 explains why this refines, but does not contradict, the Gameplay Spec's stage-name ordering).

---

## 5. Base Block Placement Value

**DESIGN DECISION: Placement alone contributes zero Score and zero Damage.**

**Why:** Gameplay Spec Section 8 is explicit — "Damage is generated only as a result of Line Clears; no other player action generates Damage" — and this document does not introduce an exception. Awarding value for placement-without-clearing would:

- Reward "filling space" over "solving the board," directly undermining Master Vision Section 5.2 (Tactical Placement: "the core loop remains engaging on its own merits," not because every action independently pays out) and Section 5.3 (a placement-value mechanic would make "place anywhere, clears happen automatically" — the Section 5.2 failure condition — strictly *more* attractive, not less).
- Decouple the player-facing number from the actual skill signal (efficient line completion), which is precisely the "health bar above a Block Blast clone" failure mode the Primary Objective warns against — a placement-value tick is generic-puzzle-game plumbing, not a combat-strategy signal.
- Blur the "no damage without a Clear" philosophy rule (Section 3) established as non-negotiable.

**Placement's actual role in the model:** Placement is not scored, but it is not valueless — it is the *decision substrate* that determines `L` (Section 4) on a future Turn. A placement that sets up a large simultaneous multi-line clear is exactly how Master Vision Section 3's "Escalation" fantasy is earned: the value is deferred to the Clear it enables, never paid out early.

---

## 6. Line-Clear Damage

**DESIGN DECISION:** `BD(n)` is a flat per-line value, identical for every line regardless of whether it is a row or a column (Board Engine doc Section 7 treats rows and columns symmetrically; this document inherits that symmetry) — but it is **not** flat across `n` within a multi-line Turn; each additional simultaneous line contributes an escalating increment, so that `ClearValue` for a multi-line Turn is meaningfully more than `L × BD(1)`.

**Base per-line values (Section 23, Table 23.1):**

| Lines in this Turn's clear-set (`L`) | `BD(n)` for each line, `n = 1..L` | `ClearValue = Σ BD(n)` |
|---|---|---|
| 1 | `BD(1) = 10` | 10 |
| 2 | `BD(1) = 10`, `BD(2) = 12` | 22 |
| 3 | `BD(1) = 10`, `BD(2) = 12`, `BD(3) = 15` | 37 |
| 4+ | `BD(1) = 10`, `BD(2) = 12`, `BD(3) = 15`, `BD(n≥4) = 18` each | `37 + 18×(L-3)` |

**Rationale for escalating (not flat, not purely multiplicative) per-line value:** a purely flat model (`L × 10`) makes a 4-line clear feel arithmetically identical in "value per line" to a 1-line clear, undercutting the Combo System's own job (Section 7) of rewarding simultaneity; a purely multiplicative model (applying Combo Modifier on top of an already-escalating `BD(n)`) would double-reward simultaneity and risk runaway values late-run. Keeping `BD(n)` escalation modest and letting the Combo Modifier (Section 7) carry the majority of the multi-line reward keeps the two systems' contributions distinguishable to the player (Master Vision Section 5.4, "players can distinguish clear magnitude by feedback alone").

**These are the exact numeric values that supersede the Gameplay Spec Section 23 placeholder ("Base Damage per line cleared: 10 (placeholder)") per Gameplay Spec Section 25's explicit delegation to the Scoring doc.**

---

## 7. Combo System

A Combo, per Gameplay Spec Section 2/8, is the bonus applied when a single Placement clears more than one line simultaneously — this document is the sole owner of its exact mechanics.

- **Combo start:** A Combo is evaluated fresh every Turn from `L` (this Turn's `LinesCleared` count) — there is no persistent "Combo meter" that survives from one Turn to the next at Absolute MVP. A Combo "starts" the instant `L ≥ 2` in a single Turn's clear-set.
- **Combo increase:** Within a single Turn, `ComboMult` (Section 4) increases with `L` per Table 23.2 (Section 23) — it is a lookup on `L`, not an incrementing counter across Turns.
- **Combo continuation across Turns — DESIGN DECISION: not implemented at Absolute MVP.** A "Combo streak" spanning multiple consecutive Turns (e.g., "clear a line 3 Turns in a row") is explicitly out of scope here, matching Gameplay Spec Section 21's edge case ruling: "Combo achieved across two separate Turns... does not count toward 'Achieve X Combo,' which is strictly single-Placement, simultaneous-line-count based." This document's Combo system inherits that same strict single-Placement scope for Damage purposes, not just for the Objective predicate.
- **Combo break:** Since a Combo is re-evaluated fresh every Turn (not a persistent meter), there is no explicit "break" event to define — a Turn with `L = 1` or `L = 0` simply applies `ComboMult = 1.0×` (no bonus, no penalty) for that Turn only.
- **Multiplier:** Table 23.2 (Section 23) — `ComboMult` is `1.0×` at `L=1`, rising through `L=2,3` and capping at `L≥4` (see Maximum, below).
- **Maximum:** `ComboMult` caps at `L=4` and does not continue escalating for a theoretical 5+ line simultaneous clear — the Board is 8×8 (`03_BOARD_ENGINE_AND_RULES.md` Section 1), so `L` can reach at most 16 (8 rows + 8 columns) in the Fixture F5 full-board-clear edge case (Board Engine doc Section 14); capping the multiplier at `L≥4` prevents that rare maximal case from producing a runaway, un-tunable Damage spike while still rewarding it heavily via the escalating `BD(n)` tail (Section 6).
- **Relationship with Turns:** Combo is **strictly a single-Turn, single-Placement concept** — it is computed once, inside `COMBAT_RESOLUTION` (Gameplay Spec Section 4), from that Turn's `L` alone, and has no memory of prior or future Turns.

---

## 8. Critical / Exceptional Clears

**DESIGN DECISION:** No independent "critical hit" RNG roll exists at Absolute MVP. Instead, "exceptional" clears are entirely deterministic outcomes of the Section 6–7 formulas — a 4-line simultaneous clear is already mechanically the game's top-tier Damage event (highest `BD(n)` tail, highest `ComboMult`) without needing a separate random critical-chance layer stacked on top.

**Rationale:** an independent crit-roll would reintroduce exactly the kind of "unavoidable/uncontrollable" RNG-on-top-of-skill that `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 1 rules out for generation, and the same philosophy (Section 3, rule 3 of this document) applies to combat outcomes — a player who executes a 4-line clear should get the 4-line-clear reward *because they earned it*, not "usually, unless the crit roll fails."

**What does exist as an "exceptional clear" mechanic:**

- **Secondary effects, not bonus damage — DESIGN DECISION:** an `L≥3` clear (a "Full Combo" in player-facing terms) triggers a defined *non-numeric* secondary effect: an audiovisual escalation tier (owned by the future Juice/VFX doc, Master Vision Section 21) and a flag exposed on the `DamageEvent` (Section 9's event payload) marking it eligible for Bonus Objective tracking (`01_GAMEPLAY_SPECIFICATION.md` Section 13, e.g. "Achieve a 3-line Combo at least once"). No additional numeric Damage is layered on beyond what Sections 6–7 already computed.
- **Full-board clear (Board Engine Fixture F5) is not a distinct formula case** — it is simply the `L≥4` row of Table 23.1/23.2 applied at whatever `L` value results (up to 16); this document does not special-case it further, keeping the formula surface small (Master Vision Section 5.8, solo-developer feasibility).

---

## 9. Damage Pipeline

**Fully-specified stage order**, refining `01_GAMEPLAY_SPECIFICATION.md` Section 9's named stages into this document's exact sub-steps without reordering or removing any named stage:

```text
1. Base Event         — commitPlacement resolves (03_BOARD_ENGINE_AND_RULES.md Section 5)
2. Line Count          — L = |filledRows| + |filledCols| from the Board Engine's LinesCleared event
                          (03_BOARD_ENGINE_AND_RULES.md Section 12); if L = 0, pipeline terminates here,
                          no DamageEvent or ScoreEvent is emitted (Section 4 of this document)
3. Base Damage          — ClearValue = Σ BD(n), n = 1..L (Section 6, Table 23.1)
4. Combo Modifier       — StageA = ClearValue × ComboMult (Section 7, Table 23.2)
5. Relic Modifier       — StageB = (StageA + RelicFlat) × RelicMult
                          (Section 13; flat adds applied before multipliers, per Relic-defined order,
                          matching Gameplay Spec Section 9's "flat adds, then multipliers")
6. Objective Modifier   — StageC = StageB × ObjMod (Section 16; 1.0× if no active Objective modifier)
   [ScoreValue = round(StageC) is captured here — Score's terminal point, Section 4]
7. Enemy Modifier       — StageD = StageC − (StageC × EnemyDef.percent) − EnemyDef.flat (Section 11)
8. Final Damage          — FinalDamage = max(0, round(StageD)) — the ONLY clamp-to-zero point
                            in the entire pipeline (inherited MASTER DESIGN RULE, Gameplay Spec Section 9)
9. Enemy HP Update       — EnemyHP -= FinalDamage, floored at 0 (Section 10)
```

**DESIGN DECISION — exact ordering justification:** Combo Modifier is applied before Relic Modifier (matching the sequence implied by Gameplay Spec Section 9's stage list order — Relic Modifiers, then Combo Modifiers — this document interprets that as the *conceptual* grouping, and fixes the arithmetic order as Combo-then-Relic so that a flat Relic bonus, e.g. "+5 Damage," is added to an already-Combo-scaled value rather than being scaled *by* the Combo multiplier itself; this keeps flat Relic bonuses predictable in absolute terms regardless of how many lines were cleared that Turn). This refinement is additive detail, not a contradiction of Gameplay Spec Section 9's named order — the Master Vision/Gameplay Spec's named-stage sequence (Relic Modifiers, Combo Modifiers, Objective Modifiers, Enemy Defense) is preserved as the section-boundary order; this document's Section 4/9 numbering exists only to make the arithmetic within and across those boundaries exact and reproducible, and does not move any named stage across another named stage's boundary.

**Emitted event — `DamageEvent`:** `{ turnId, L, ClearValue, ComboMult, RelicMod: {flat, mult}, ObjMod, EnemyDef, FinalDamage, ScoreValue, isFullCombo: (L≥3) }` — every intermediate value is included, satisfying Section 3 rule 2 (legibility).

---

## 10. Enemy HP

- **Initial HP:** set per-Enemy at Battle `PREPARING` (Gameplay Spec Section 4), owned by a future Enemy Content doc as a data value; this document only defines the *update rule*, not per-Enemy numbers. **DESIGN DECISION (placeholder banding, Section 23, Table 23.3):** standard-Battle Enemy HP bands scale by run position (Early/Mid/Late, matching `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 12's difficulty bands) so this document's Section 20 Progression Balance has a concrete anchor even before the Enemy Content doc exists.
- **Scaling:** Enemy HP does not scale mid-Battle — it is fixed at `PREPARING` for that Battle's duration (consistent with Gameplay Spec Section 8, "a non-negative integer, set at Battle start"). Cross-Battle scaling (harder Enemies later in a Run) is a data-authoring concern for the Enemy Content doc, banded per Table 23.3.
- **Healing:** **not implemented at Absolute MVP.** No Enemy mechanic category in Gameplay Spec Section 11 (Block, Freeze, Hazard, Alter Rules) includes HP restoration, and this document introduces none — an Enemy that heals would work against Master Vision Section 5.6 (Fair Difficulty: losses traceable to a *player* decision) since it would make prior player progress reversible by a source outside player control. Reserved as an explicit non-goal, not a silent omission.
- **Shields/Armor:** distinct from Defense (Section 11) — **not implemented at Absolute MVP** as a separate HP-like pool. Defense (percentage/flat Damage reduction) is the only Enemy mitigation mechanic; a separate "shield HP bar" would duplicate Defense's function while adding a second UI/legibility surface, contradicting Master Vision Section 5.8's complexity budget.
- **Death threshold:** `EnemyHP ≤ 0` (Gameplay Spec Section 8) — checked immediately after Section 9's step 9, at `OBJECTIVE_EVALUATION` (Gameplay Spec Section 4); this is the default Primary Objective condition ("Defeat Enemy," Gameplay Spec Section 12) and triggers `ENEMY_DEFEATED` per Gameplay Spec Section 4's state table.

---

## 11. Enemy Defense

**DESIGN DECISION: kept deliberately minimal at Absolute MVP** — a single Enemy Defense value per Enemy, composed of at most two sub-parts, both optional and independently zero-able:

- **`EnemyDef.percent`** — a percentage reduction (0–100%, expressed as a decimal 0.0–1.0) applied to `StageC` (Section 9, step 7).
- **`EnemyDef.flat`** — a flat Damage subtraction applied after the percentage reduction, same stage.

**Default:** both `EnemyDef.percent = 0` and `EnemyDef.flat = 0` for all standard Enemies (Gameplay Spec Section 23, "Default Enemy Defense: 0, Overridable per enemy"), matching this document's inheritance of that Constant.

**Special rules:**

- Defense is applied **only once, at the single fixed pipeline stage** (Section 9, step 7) — it is never applied per-line, per-Combo-tier, or at any earlier stage, per the inherited MASTER DESIGN RULE (Gameplay Spec Section 9: "Damage is clamped to ≥0 only at the Final Damage stage... implementation must apply defense as a final-stage subtraction/percentage, not an earlier multiplier"). This document's exact arithmetic (percent-then-flat, both at step 7) is the concrete instantiation of that inherited rule.
- Defense values are never negative (a "negative defense" / vulnerability effect, if ever added, is modeled as an Objective or Relic Modifier at its own pipeline stage — Sections 5/6 of the pipeline — never as a negative `EnemyDef`, keeping the Enemy Modifier stage's semantics strictly "mitigation only").
- No Enemy Defense value may reduce `FinalDamage` below 0 pre-clamp in a way that "banks" negative value forward — Section 9 step 8's clamp is the sole and final word; this document introduces no carry-over mechanic.

---

## 12. Player Health

**DESIGN DECISION: Player HP is explicitly and permanently out of scope for Absolute MVP.**

This is not an oversight — it is a direct implementation of `00_MASTER_GAME_VISION.md` Section 10's binding decision (referenced by Gameplay Spec Section 1 as non-contradictable: "the Section 10 decision that player HP is not part of the Absolute MVP") and Gameplay Spec Section 11's explicit statement that "Enemy actions modify the Board and rules, not a Player HP resource (none exists at Absolute MVP)."

**Why this document does not reintroduce it despite combat-game convention:**

1. **The failure condition is already fully defined without it.** Board Lock (`03_BOARD_ENGINE_AND_RULES.md` Section 10, Gameplay Spec Section 15) is the sole Run-ending failure mechanism; a Player HP pool would create a *second*, parallel failure path, doubling the surfaces a player must track to answer "why did I lose" — directly working against Master Vision Section 5.6's Fair Difficulty pillar rather than supporting it.
2. **It would fight the Core Fantasy.** Master Vision Section 3 defines the fantasy as "solving a simple block puzzle while building an increasingly broken combat build," explicitly *not* "I'm a powerful hero" — a Player HP bar is the single most common signifier of the hero-survival fantasy Master Vision Section 3 deliberately excludes.
3. **Threat is already legible through the Board.** Enemy Mechanics (Gameplay Spec Section 11: Block Cells, Freeze Cells, Hazards) already express "danger" entirely through Board state the player can see and route around — a Player HP number would be a redundant, less spatially-grounded way to express the same pressure Master Vision Section 5.6's "tension under control" pillar wants expressed *through the puzzle itself*.

**If a future document reconsiders this:** per Gameplay Spec Section 1, doing so requires an explicit revision to `00_MASTER_GAME_VISION.md` Section 10 first — this document, `01_GAMEPLAY_SPECIFICATION.md`, and `03_BOARD_ENGINE_AND_RULES.md` all currently assume its absence structurally (e.g., Gameplay Spec Section 15's failure rules explicitly forbid "Enemy-Caused Failure... through a Player HP resource"), so reintroducing it is a cross-document change, not a local one.

---

## 13. Relic System

**DESIGN DECISION — architecture:** every Relic is defined as a **Modifier declaration** that plugs into one or more of the named Damage Pipeline stages (Section 9) or the Tray Generation reweight seam (`04_SPAWN_RNG_AND_DIFFICULTY.md` Section 9) — a Relic is data, never new code. This is the direct implementation of the Master Prompt's requirement: "A relic should modify an existing puzzle/combat rule instead of introducing an entirely independent mini-game," and matches Master Vision Section 20's "reusable effects... shared, extensible effect framework."

**Relic Modifier schema (conceptual):**

| Field | Description |
|---|---|
| `id` | Unique identifier, format `RLC_0##`. |
| `display_name` | Player-facing name and plain-language effect text (Master Vision Section 5.1/5.3 — "plain-language effects," "picks based on immediate understanding"). |
| `trigger` | The pipeline stage or event this Relic hooks (Section 9 stage name, or `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 9's generation seam). |
| `condition` | An optional predicate over `DamageEvent`/`BoardChanged`/Shape metadata (e.g., "if `L ≥ 3`," "if the placed Shape has tag `SMALL`") that must be true for the effect to apply this Turn. |
| `effect` | The exact modification: a flat add, a multiplier, or (for the generation seam only) a weight multiplier — never a new mechanic type outside these three primitives at Absolute MVP. |
| `affected_event` | Which of `DamageEvent`, `ScoreValue`, or `TrayGeneration` this Relic modifies. |
| `stacking_behavior` | Whether multiple copies/instances of effects in the same category sum, multiply, or are mutually exclusive (Section 13.1). |
| `limits` | Any cap on the effect (e.g., "max +50% total from this Relic category") — every Relic must declare one, even if the declared limit is "none," to satisfy `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 16 rule 6's "individually capped" pattern extended to combat. |
| `balance_category` | One of the Synergy Philosophy archetypes (Section 15) this Relic primarily supports. |

### 13.1 Stacking Behavior — general rules

- **Flat-add Relics** of the same `affected_event` **sum** (e.g., two "+3 flat Damage" Relics active together contribute +6 total at the Relic Modifier stage).
- **Multiplier Relics** of the same `affected_event` **multiply together**, not add (e.g., a +20% and a +15% Relic together yield ×1.20 × ×1.15 = ×1.38, not ×1.35) — this keeps the math simple and matches how `RelicMult` is defined as a single scalar in Section 4's formula (the scalar is the product of all active multiplier Relics).
- **Condition-gated Relics** whose conditions are mutually exclusive in a single Turn (e.g., one Relic triggers on `L=1`, another on `L≥3`) simply never both apply in the same Turn — no special resolution needed, since at most one condition is ever true.
- **Generation-seam Relics** (Section 15's archetypes referencing shape frequency) stack additively into `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 9's weight-multiplier seam, subject to that document's Section 5 Solvability Protection regardless of stacked Relic weight — Relic stacking can never bypass the Solvability guarantee.

---

## 14. Launch Relic Framework

**DESIGN DECISION: exactly 8 launch Relics**, sized to the Master Prompt's "approximately 6–10" guidance and Master Vision Section 12's complexity budget. Each is defined conceptually per Section 13's schema; exact numeric tuning lives in Section 23, Table 23.4.

| ID | Name (concept) | Trigger | Condition | Effect | Affected Event | Stacking | Limits | Balance Category |
|---|---|---|---|---|---|---|---|---|
| RLC_001 | Single-Minded | Combo Modifier stage | Placed Shape this Turn was `SHP_001` (Single, tag `SINGLE`) | Flat Damage add | DamageEvent | Sums with other flat-add Relics | Max +1 trigger per Turn (one placement) | 1×1 build |
| RLC_002 | Efficient Filler | Combo Modifier stage | Placed Shape tagged `SMALL` | Small flat Damage add | DamageEvent | Sums | Max +1 trigger per Turn | 1×1 build |
| RLC_003 | Line Momentum | Relic Modifier stage | `L ≥ 1` (any clear) | Percentage Damage multiplier | DamageEvent | Multiplies with other mult Relics | Capped at Table 23.4's stated ceiling | Line-clear build |
| RLC_004 | Chain Reaction | Objective Modifier stage (Damage side only, Section 16) | `L ≥ 2` (Combo occurred) | Percentage Damage multiplier, scaling with `L` | DamageEvent | Multiplies | Capped; does not apply when `L=1` | Combo build |
| RLC_005 | Bulk Breaker | Relic Modifier stage | Placed Shape tagged `LARGE` | Flat Damage add, larger than RLC_002's | DamageEvent | Sums | Max +1 trigger per Turn | Large-piece build |
| RLC_006 | Steady Defense | Enemy Modifier stage — **inverted seam**: reduces effective incoming Enemy Hazard severity rather than boosting outgoing Damage (see Section 13's `affected_event` allows `BoardChanged`-adjacent Hazard interactions per `03_BOARD_ENGINE_AND_RULES.md` Section 9's post-clear hook) | An Enemy Hazard mechanic is active this Battle | Reduces a Hazard's Board footprint (e.g., 1 fewer cell Blocked) | Board/Hazard event, not DamageEvent | N/A (single passive) | One Hazard-reduction per Enemy action | Healing/survival build (Board-state proxy, since no Player HP exists per Section 12) |
| RLC_007 | Glass Cannon | Combo Modifier + Relic Modifier stages | Always active | Large percentage Damage multiplier, paired with a Score-only penalty (never a loss-condition penalty, to respect Section 12) | DamageEvent, ScoreValue | Does not stack with itself; unique | Fixed magnitude, non-stacking | High-risk/high-reward build |
| RLC_008 | Objective Focus | Objective Modifier stage | Active Battle has a non-default Primary Objective (Gameplay Spec Section 12) | Percentage Damage multiplier | DamageEvent | Multiplies | Capped; inert on standard "Defeat Enemy" Battles | Objective-synergy build |

**DESIGN DECISION — explicitly not 50 Relics:** this table is the complete Absolute MVP Relic pool. Expanding it is a Master Vision Section 15 expansion decision (same pattern as Enemy Mechanics and Pity mechanics in prior documents), gated on this framework proving out in simulation (Section 24) first.

---

## 15. Synergy Philosophy

**DESIGN DECISION:** Builds emerge from Relic *category* overlap, not from explicit "combo pairs" hardcoded between specific Relic IDs — this keeps the combinatorial design space open (Master Vision Section 5.3, "multiple viable build archetypes") without this document needing to enumerate every pairing by hand (infeasible at solo-developer scale, Master Vision Section 5.8).

Archetypes supported by the Section 14 pool, each requiring only that its member Relics' `balance_category` and `condition` fields align — no bespoke interaction code:

- **1×1 build:** RLC_001 + RLC_002, further supported by `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 9's generation-seam category (a future generation-affecting Relic favoring `SINGLE`/`SMALL` tags would slot in here) — rewards frequent small, precise placements over big simultaneous clears.
- **Line-clear build:** RLC_003, stacking with Table 23.4's baseline — rewards simply clearing often, a low-complexity on-ramp for new players (Master Vision Section 5.1).
- **Combo build:** RLC_004, rewarding deliberately banking multiple lines for one large simultaneous clear (Section 7) — the direct mechanical expression of "planning better placements" (Primary Objective).
- **Large-piece build:** RLC_005, synergizing with `04_SPAWN_RNG_AND_DIFFICULTY.md` Table 17.1's `Finisher`/`High-Risk Finisher` weighted Shapes — rewards embracing the higher-variance Shape pool rather than avoiding it.
- **Healing/survival build:** RLC_006 — since Section 12 rules out Player HP, "survival" is reframed as *Board-state resilience* (reducing Hazard footprint) rather than health regeneration; this is the build archetype's Block-Battles-specific reinterpretation, not a literal healing mechanic.
- **High-risk/high-reward build:** RLC_007, alone or stacked with RLC_004 (Combo) or RLC_005 (Large-piece) for compounding variance — the build explicitly trading Score for Damage ceiling.
- **Objective-synergy build:** RLC_008, primarily relevant in Block Quest Objective encounters (Gameplay Spec Section 12) rather than standard Battles — demonstrates a build archetype that only "activates" in certain content, which is itself a form of build diversity across a Run's mixed Battle types.

**MASTER DESIGN RULE:** No two Relics in the Section 14 pool are mutually exclusive by design (i.e., every pair is legally co-selectable across a Run) except where Section 13.1 already defines non-stacking (RLC_007 with itself, which cannot occur since Relic Choice never re-offers an already-owned Relic — Section 17) — build identity comes from *which subset* a player accumulates, never from the game blocking a combination outright, preserving Master Vision Section 5.3's "meaningful trade-off," not "meaningless lockout."

---

## 16. Objective Interaction

**DESIGN DECISION:** the Objective Modifier stage (Section 9, step 6) is this document's own addition — Gameplay Spec Section 9 names the stage but assigns its exact formula to this document ("Section 9: Objective Modifiers — Objective-specific damage rule, if any — owned by the Objectives doc" cross-referenced against Gameplay Spec Section 25, which in turn defers numeric formulas generally to the Scoring doc; this document is authoritative for the *arithmetic*, a future Objectives doc is authoritative for *which* Objective types carry a modifier at all).

- **Default:** `ObjMod = 1.0×` (no change) for any Objective without an explicitly declared modifier — same "neutral default" pattern as `04_SPAWN_RNG_AND_DIFFICULTY.md` Table 17.6.
- **Damage interaction:** an Objective *may* declare a Damage modifier (e.g., a Block Quest Objective themed around "overwhelming force" could declare `ObjMod = 1.1×`) — this is authored per-Objective, not systemic, and is subject to the same "capped, documented, no hidden guarantee" principle `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 8 established for generation (this document's Section 16 is the combat-layer sibling of that generation-layer rule).
- **Score interaction:** Score (Section 2) is derived at the same pipeline point (Section 9, before the Enemy stage) and therefore is affected by `ObjMod` identically to Damage — an Objective modifier is never Damage-only or Score-only, preserving the single-source model (Section 2).
- **Relic interaction:** per Section 9's fixed stage order, Relic Modifiers resolve **before** Objective Modifiers (matching Gameplay Spec Section 21's edge case ruling: "Relic effect and Objective rule both modify the same Damage Pipeline stage... within a stage, Relic Modifiers resolve before Objective Modifiers" — here extended to the cross-stage case, since this document's exact pipeline places them at genuinely distinct steps 5 and 6).
- **Victory interaction:** Objective satisfaction (Gameplay Spec Section 12–14) is evaluated independently of Damage/Score values — a non-HP Primary Objective (e.g., "Clear 5 Total Lines") ends the Battle in victory regardless of `FinalDamage` accumulated, per Gameplay Spec Section 14; this document's pipeline still runs in full even on such Battles (so Score/RelicMod continue to have meaning for Bonus Objective tracking, Section 8's `isFullCombo` flag, and Balance Metrics) but its `FinalDamage` output may simply never reduce a tracked Enemy HP to the victory threshold in that Battle type.

---

## 17. Battle Rewards

**DESIGN DECISION: 3 relic choices → choose 1, as the default and only Absolute MVP path**, per the Master Prompt's stated preference and matching `01_GAMEPLAY_SPECIFICATION.md` Section 17's existing "2–3 randomized Relics offered" rule — this document narrows that range to exactly 3 at Absolute MVP for consistency and simplicity (a fixed count is simpler to balance and test, Master Vision Section 5.8), while remaining within the Gameplay Spec's stated 2–3 bound rather than contradicting it.

**Sequence, per Gameplay Spec Section 4/16/17 (this document adds no new states, only fills in the reward content):**

```text
ENEMY_DEFEATED
 → BATTLE_VICTORY  — Bonus Objective checked (Gameplay Spec Section 13); if satisfied, supplementary
                       reward granted (exact reward type TBD by a future Progression doc, Section 18)
 → Relic Selection  — 3 relics offered, drawn from the Section 14 pool excluding any already-owned
                       by the player this Run (no duplicate Relic ownership at Absolute MVP);
                       exactly 1 must be selected (Gameplay Spec Section 17, mandatory, no skip/reroll)
 → Transition        — next Battle prepared (04_SPAWN_RNG_AND_DIFFICULTY.md Section 11/12 curve applies)
```

**Relic offer weighting — DESIGN DECISION:** the 3 offered Relics are drawn uniformly at random from the not-yet-owned subset of the Section 14 pool at Absolute MVP (no weighting toward "synergizes with your current build") — a deliberate simplicity choice consistent with Master Vision Section 5.8; build-aware relic weighting is a named `04_SPAWN_RNG_AND_DIFFICULTY.md`-style future enhancement, not implemented here to avoid duplicating that document's reweighting machinery for a system this small (8 total Relics).

---

## 18. Persistent Progression

**DESIGN DECISION: exactly one persistent mechanic**, directly implementing `01_GAMEPLAY_SPECIFICATION.md` Section 18's existing decision (this document supplies the missing formula, that document supplied the structural decision) and Master Vision Section 12's two-currency ceiling.

- **Run Currency:** at Run end (`RUN_DEFEAT` or a future Run-completion, Gameplay Spec Section 3), the player is granted `PersistentCurrency = (BattlesWon × 5) + (BossesDefeated × 15)` (Section 23, Table 23.5) — a simple, auditable formula directly proportional to combat performance this document already tracks (Battle Victory events, Section 17), requiring no new tracked state.
- **No unlock/spend system defined here** — per Gameplay Spec Section 18, "No unlock/spend systems are specified in this document; they are explicitly out of scope... (MVP+ / Post-launch)" — this document inherits that scope boundary and does not expand it. The currency accrues and is stored (Gameplay Spec Section 18) even though its spend-side is undefined; this is intentional, not an oversight, matching Master Vision Section 12's explicit small-ceiling design.
- **DESIGN DECISION — no skill tree:** consistent with the Master Prompt's explicit instruction, no skill tree, talent web, or multi-currency meta-progression system is introduced. This satisfies both Gameplay Spec Section 18's scope boundary and Master Vision Section 13's forbidden-scope pattern (features not justified by a strong, stated reason are cut).

---

## 19. Run Structure

Restated with this document's specific reward content filled in (no contradiction of `01_GAMEPLAY_SPECIFICATION.md` Section 16's owning structure):

```text
Battle
 → Battle Reward:   Bonus Objective check (Section 16) → supplementary reward if met
 → Relic Selection:  3 choices → 1 chosen (Section 17)
 → Next Battle:      standard, Boss, or Objective (04_SPAWN_RNG_AND_DIFFICULTY.md Section 11/12 curve)
 ... loop ...
 → Boss Battle:      same Damage Pipeline (Section 9), elevated Enemy HP/Defense (Table 23.3, 23.6)
 ... loop ...
 → Run Defeat:        Board Lock or Objective-specific failure (Gameplay Spec Section 15)
 → Persistent Reward: PersistentCurrency granted (Section 18)
```

This document owns every reward *value* in this loop; `01_GAMEPLAY_SPECIFICATION.md` Section 3/16 owns the loop's *state structure*, which this document does not alter.

---

## 20. Progression Balance

**DESIGN DECISION — all values below are Absolute MVP placeholders (Section 23), pending simulation (Section 24):**

| Run phase | Player power posture | Enemy HP/Defense posture (Table 23.3/23.6) | Relic count typically owned |
|---|---|---|---|
| **Early-run** (Battles 1–3) | 0–2 Relics; `ClearValue`/`ComboMult` alone (Sections 6–7) carry most Damage. | Low HP band, `EnemyDef = 0` for all standard Early Enemies. | 0–2 |
| **Mid-run** (Battles 4–8, incl. first Boss) | 2–5 Relics; `RelicMod` becomes a comparable contributor to `ClearValue` for the first time. | Medium HP band; first non-zero `EnemyDef.flat` values introduced (small). | 2–5 |
| **Late-run** (Battles 9+) | 5+ Relics; stacked multipliers (Section 13.1) can meaningfully exceed base `ClearValue`'s contribution — this is the intended "increasingly broken combat build" escalation (Master Vision Section 3). | High HP band; both `EnemyDef.percent` and `.flat` may be non-zero, requiring the player's stacked Damage to overcome real mitigation for the first time. | 5–8 (approaching full Section 14 pool ownership) |
| **Boss** (any phase, per content cadence) | Same Relic count as the surrounding phase — Bosses are a Battle variant (Gameplay Spec Section 3), not a separate power tier for the player. | Elevated HP/Defense relative to that phase's standard Enemy band (Table 23.6 multiplier over Table 23.3). | N/A |

**Relic impact target (qualitative, pending Section 24 quantitative confirmation):** by late-run, a player's stacked Relic multipliers should be capable of roughly doubling their early-run `ClearValue`-only Damage output for an equivalent clear — enough to feel like "an increasingly broken combat build" (Master Vision Section 3) without any single Relic alone providing that entire doubling (Master Vision Section 5.3, no dominant option).

---

## 21. Mathematical Examples

All examples use Table 23.1/23.2 base values; `RelicMod`/`ObjMod`/`EnemyDef` shown explicitly where non-default.

**Example 1 — One-line clear, no modifiers:**
`L=1`. `ClearValue = BD(1) = 10`. `ComboMult = 1.0×` (Table 23.2). `StageA = 10`. No Relics: `RelicFlat=0, RelicMult=1.0` → `StageB = 10`. No Objective modifier: `ObjMod=1.0` → `StageC = 10`. `ScoreValue = 10`. `EnemyDef = 0` → `StageD = 10`. `FinalDamage = 10`.

**Example 2 — Two-line clear, no modifiers:**
`L=2`. `ClearValue = BD(1)+BD(2) = 10+12 = 22`. `ComboMult = 1.3×` (Table 23.2). `StageA = 28.6`. No Relics/Objective. `StageC = 28.6`. `ScoreValue = round(28.6) = 29`. `EnemyDef=0` → `FinalDamage = round(28.6) = 29`.

**Example 3 — Three-line clear, no modifiers:**
`L=3`. `ClearValue = 10+12+15 = 37`. `ComboMult = 1.6×`. `StageA = 59.2`. `StageC = 59.2`. `FinalDamage = ScoreValue = 59`.

**Example 4 — Four-line combo clear, no modifiers:**
`L=4`. `ClearValue = 10+12+15+18 = 55`. `ComboMult = 2.0×` (capped, Section 7). `StageA = 110`. `FinalDamage = ScoreValue = 110`.

**Example 5 — Relic-enhanced clear (RLC_003 Line Momentum, +15% mult, active):**
`L=1`. `ClearValue = 10`. `ComboMult = 1.0×` → `StageA = 10`. `RelicFlat=0`, `RelicMult = 1.15` (RLC_003 alone) → `StageB = 11.5`. `ObjMod=1.0` → `StageC = 11.5`. `ScoreValue = 12`. `EnemyDef=0` → `FinalDamage = 12`. (Compare Example 1's 10 — a modest, legible +2 from a single passive Relic on a single-line clear.)

**Example 6 — Stacked Relic clear (RLC_003 ×1.15 + RLC_004 Chain Reaction ×1.25 at L=2, plus RLC_001 flat +2 since Shape was Single... — illustrative combination):**
`L=2`, placed Shape tagged `SINGLE`. `ClearValue = 22`. `ComboMult = 1.3×` → `StageA = 28.6`. `RelicFlat = 2` (RLC_001) . `RelicMult = 1.15 × 1.25 = 1.4375` (RLC_003 × RLC_004, multiplied per Section 13.1) → `StageB = (28.6 + 2) × 1.4375 = 30.6 × 1.4375 ≈ 44.0`. `ObjMod=1.0` → `StageC ≈ 44.0`. `ScoreValue = 44`. `EnemyDef=0` → `FinalDamage = 44`. (Demonstrates flat-then-multiply ordering within the Relic Modifier stage, Section 13.1.)

**Example 7 — Objective-enhanced clear (RLC_008 Objective Focus, ×1.1, active on a non-default-Objective Battle):**
`L=1`. `ClearValue=10`. `ComboMult=1.0` → `StageA=10`. No other Relics → `StageB=10`. `ObjMod` from RLC_008's Objective-stage placement: `1.1×` → `StageC = 11.0`. `ScoreValue=11`. `FinalDamage=11`.

**Example 8 — Boss interaction with Enemy Defense:**
`L=3`, Boss has `EnemyDef.percent = 0.15`, `EnemyDef.flat = 5` (Table 23.6 example). `ClearValue=37`. `ComboMult=1.6×` → `StageA=59.2`. No Relics/Objective → `StageC=59.2`. `ScoreValue=59` (unaffected by Enemy Defense, Section 2/9). `StageD = 59.2 − (59.2×0.15) − 5 = 59.2 − 8.88 − 5 = 45.32`. `FinalDamage = round(45.32) = 45`. (Score of 59 vs. Damage of 45 — the divergence point where the single-source model (Section 2) branches, visible to the player as "you played it well, the Boss just mitigated some of it," satisfying Section 3's legibility rule.)

---

## 22. Exploit Prevention

| Exploit category | Risk | Prevention |
|---|---|---|
| **Double scoring** | A single Placement's clear-set being counted twice (e.g., once per row, again per column at an intersection cell). | Structurally prevented at the source — `03_BOARD_ENGINE_AND_RULES.md` Section 8's `cellsToClear` is a deduplicated set union; `L` (this document's Section 4) is derived from `|filledRows| + |filledCols|` (line-count, not cell-count), so an intersection cell contributes to `L` correctly (once per line it completes) without being double-*cleared* — these are different, both-correct countings the Board Engine and this document already agree on (Board Engine Section 7's own DESIGN DECISION). |
| **Duplicate triggers** | A single Relic firing more than once for one Turn's single `DamageEvent`. | Every Relic's `trigger` (Section 13) is scoped to a single named pipeline stage per Turn; a Turn produces at most one `DamageEvent` (Gameplay Spec Section 8, inherited), so a stage-scoped Relic mathematically cannot fire twice within it. Relics with explicit "max +1 trigger per Turn" limits (Table, Section 14) make this constraint doubly explicit for Relics whose condition could otherwise be misread as per-cell rather than per-Turn. |
| **Infinite loops** | A Relic or Objective modifier chain that re-triggers itself (e.g., a Damage-based effect that generates a new Clear, which re-triggers the same Damage effect). | Prevented by construction: no Relic in the Section 14 pool, and no Objective Modifier (Section 16), is defined to mutate the Board or generate a new Clear — every Relic/Objective effect in this document modifies a Damage Pipeline *value*, never Board state or `LinesCleared` count itself. Any future Relic proposal that would cause a Clear as a side effect of Damage must route through a full cross-document revision (Board Engine + this doc), not be added here unilaterally. |
| **Stacking exploits** | Unbounded multiplier stacking producing runaway Damage (e.g., many multiplier Relics compounding to absurd values). | Section 13.1's multiply-together rule combined with Section 14's per-Relic `limits` field (mandatory on every Relic) bounds each contributor; Section 20's Progression Balance qualitative target (roughly 2× by late-run) is the design-time check simulation (Section 24) validates against; `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 16 rule 6's "every reweight source is capped" pattern is mirrored here for Damage-side multipliers. |
| **Revive exploitation** | N/A directly (no Player HP, Section 12; no "revive" mechanic exists anywhere in this document or its dependencies) — included for completeness per the Master Prompt's checklist. | Structurally impossible at Absolute MVP: Gameplay Spec Section 3 states a Run has "exactly one life... no mid-run continue," and this document introduces no currency-purchased continue, checkpoint-reload, or Enemy-HP-restoration mechanic (Section 10's "no healing" rule closes the adjacent surface). If a future MVP+ revive mechanic is added, it must define its own exploit-prevention rules at that time — none are pre-emptively speculated here. |
| **Score/Damage divergence misuse** | A player or content author exploiting the Section 2 single-source-but-diverging model to inflate Score without proportional Damage (or vice versa) via a targeted Enemy-Defense-only stacking. | Not exploitable in the intended direction — `EnemyDef` (Section 11) only ever *reduces* `FinalDamage` relative to `ScoreValue`, never inflates it; there is no mechanic in this document that could make `FinalDamage > ScoreValue`, so this divergence has exactly one safe direction by construction. |

---

## 23. Tuning Tables

All values below are Absolute MVP placeholder **DESIGN DECISIONs**, matching the ASSUMPTION convention of `01_GAMEPLAY_SPECIFICATION.md` Section 23, pending Section 24 simulation-driven tuning.

**Table 23.1 — Base Damage per Line (`BD(n)`), Section 6**

| `n` (position within Turn's clear-set) | `BD(n)` |
|---|---|
| 1 | 10 |
| 2 | 12 |
| 3 | 15 |
| 4+ | 18 (each) |

**Table 23.2 — Combo Multiplier (`ComboMult`), Section 7**

| `L` (lines this Turn) | `ComboMult` |
|---|---|
| 1 | 1.0× |
| 2 | 1.3× |
| 3 | 1.6× |
| 4+ (capped) | 2.0× |

**Table 23.3 — Standard Enemy HP Bands, Section 10**

| Run phase | HP band |
|---|---|
| Early-run | 40–70 |
| Mid-run | 90–150 |
| Late-run | 180–300 |

**Table 23.4 — Launch Relic Magnitudes, Section 14**

| ID | Magnitude |
|---|---|
| RLC_001 | +2 flat Damage |
| RLC_002 | +1 flat Damage |
| RLC_003 | +15% Damage mult |
| RLC_004 | +10% per line beyond the first (`L≥2`), i.e. +10%×(L−1), capped at +40% |
| RLC_005 | +4 flat Damage |
| RLC_006 | −1 cell to next Hazard's footprint |
| RLC_007 | +35% Damage mult, −20% Score |
| RLC_008 | +10% Damage mult (Objective-stage only) |

**Table 23.5 — Persistent Currency Formula, Section 18**

| Term | Value |
|---|---|
| Per Battle Won | 5 |
| Per Boss Defeated | 15 (in addition to its Battle-Won value) |

**Table 23.6 — Boss Defense/HP Multiplier over standard band, Section 20**

| Parameter | Multiplier over same-phase standard Enemy |
|---|---|
| HP | ×2.0 |
| `EnemyDef.percent` | +0.10 absolute (e.g., 0.05 standard → 0.15 Boss) |
| `EnemyDef.flat` | +5 absolute |

---

## 24. Simulation and Balance Testing

**DESIGN DECISION:** this document's formulas are validated by extending `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 14's existing Simulation Framework, not by building a parallel one — the same virtual-player policy tiers (Random-legal, Greedy-clear, Heuristic-lookahead) and the same Run Seed determinism (`04_SPAWN_RNG_AND_DIFFICULTY.md` Section 13) apply here, with this document's Damage Pipeline (Section 9) plugged in as the consumer of each simulated Turn's `LinesCleared` event.

**Methodology additions specific to this document:**

1. **Relic-equipped simulation passes:** run the same 3-policy-tier simulation (`04_SPAWN_RNG_AND_DIFFICULTY.md` Table 17.8 scale, 5,000 Runs/tier) once with Relic Selection disabled (pure `BD`/`ComboMult` baseline) and once with a scripted Relic-picking heuristic (e.g., "always pick the offered Relic matching the virtual player's dominant build archetype, Section 15") to isolate Relic contribution from base-formula contribution.
2. **Enemy HP band validation:** for each Table 23.3 band, confirm the Greedy-clear policy tier defeats a same-band standard Enemy within a target Turn-count range (TBD-post-simulation, matching `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 15's placeholder-target convention) — too few Turns suggests HP is too low relative to Damage output; too many suggests the reverse.
3. **Exploit-prevention regression suite:** the Section 22 table is converted into automated assertions run against every simulated Turn (e.g., "assert `FinalDamage ≤ ScoreValue` for all Turns," directly encoding Section 22's "Score/Damage divergence" row) — any simulation run that violates one of these assertions is a hard failure, not a balance concern.
4. **Boss Battle isolation:** Boss encounters are simulated as a distinct scenario type (elevated Table 23.6 stats against a Relic-loadout snapshot representative of that Run-phase's Table in Section 20) rather than only inline within full-Run simulation, so Boss-specific tuning can be iterated without re-running full Runs each time.

---

## 25. Acceptance Criteria

- Every `DamageEvent` is fully reconstructable from its logged intermediate values (Section 9's event payload) — no Damage number is ever produced without an auditable stage-by-stage derivation (Section 3, rule 2).
- `FinalDamage` is never negative and is clamped to zero at exactly one pipeline stage (Section 9, step 8) — verified by the Section 24.3 automated assertion suite.
- Placement alone (zero lines cleared) never produces a non-zero `DamageEvent` or `ScoreValue` (Section 5, Section 9 step 2's early termination).
- `ScoreValue` is never less than `FinalDamage` for the same Turn (Section 22, Score/Damage divergence row) — the one-directional divergence is enforced for every simulated Turn.
- No two Relics in the Section 14 launch pool require bespoke interaction code beyond the general Section 13.1 stacking rules — verified by confirming every Relic's `effect` field is one of exactly three primitives (flat add, multiplier, generation-weight multiplier).
- Player HP is referenced nowhere in this document's mechanics beyond Section 12's explicit scope-exclusion rationale (a grep-level check: no formula in Sections 4–11 or 21 depends on a Player HP variable).
- Relic Selection always offers exactly 3 options with no duplicates against the player's currently-owned set, and always requires exactly 1 selection (Section 17).
- `PersistentCurrency` is computed by the Section 18/Table 23.5 formula alone, with no additional undocumented bonus source.
- All Section 21 worked examples are exactly reproducible by an implementation following Sections 4–11 and Table 23.1–23.2/23.6 with no rounding discrepancy beyond standard round-half-up at the two documented rounding points (`ScoreValue`, `FinalDamage`).

---

## 26. Cross-Document Contracts

- **Consumes from `01_GAMEPLAY_SPECIFICATION.md`:** the fixed named Damage Pipeline stage order (Section 9), Turn/Battle/Objective/Relic-timing state structure (Sections 4, 7–8, 12–13, 16–18), the Section 10 decision excluding Player HP (Section 12 of this document implements it), and Gameplay Spec Section 25's explicit delegation of exact numeric formulas to this document (superseding Gameplay Spec Section 23's placeholder Constants per that section's own note).
- **Consumes from `02_SHAPE_LIBRARY.md`:** `category`/`tags` fields as Relic `condition` predicates (Section 13, Section 14's RLC_001/002/005), per Shape Library Section 8/14's rule that consuming documents reference Shapes by metadata, never hardcoded `id` (this document's Relic conditions follow that rule with no exception).
- **Consumes from `03_BOARD_ENGINE_AND_RULES.md`:** `LinesCleared`/`CellsCleared` events (Section 12) as the sole "Detect Clear"/Line Count input (Section 9, steps 1–2); RLC_006's Hazard-footprint interaction consumes the Section 9 post-clear hook extension seam defined there, without this document implementing Hazard mechanics itself (still owned by the future Enemy Content doc per Board Engine Section 3.2/9).
- **Consumes from `04_SPAWN_RNG_AND_DIFFICULTY.md`:** the Simulation Framework and Run Seed determinism (Sections 13–14, extended by this document's Section 24) and the Section 9 generation weight-multiplier seam that Section 15's build archetypes reference for future generation-affecting Relics; this document does not alter Tray generation logic itself, only plugs into its existing seam per that document's own Section 9 ownership boundary.
- **Exposes to Objectives/Block Quest doc (future):** the Objective Modifier stage contract (Section 16) as the sole legal integration point for an Objective's Damage-side effect; any future Objective type wanting to affect Damage must declare an `ObjMod` value through this contract, never invent a new pipeline stage.
- **Exposes to Enemy Content doc (future):** the Enemy HP/Defense data contract (Sections 10–11, Tables 23.3/23.6) as the schema that document's per-Enemy authored values must conform to; this document owns the *update rule* and *banding*, that document owns the *specific per-Enemy numbers* within the band.
- **Exposes to Relics doc (future, if this document's Section 14 pool is later split out for authoring convenience):** the full Section 13 Modifier schema and Section 13.1 stacking rules as the binding architecture any additional Relic beyond the launch 8 must conform to.
- **Exposes to Progression/UI docs (future):** `PersistentCurrency` (Section 18) as the value those documents may spend or display, with no further mechanics implied by this document beyond the accrual formula.
- This document must not be edited to embed content owned by the above; it references them structurally only, per the pattern established across `01`–`04`.

---

## 27. Final Combat Resolution Contract

Canonical mathematical/event sequence for one Turn, from Board Engine hand-off through Battle-level consequence — the authoritative sequence this entire document exists to define:

```text
Input: LinesCleared event { filledRows, filledCols } from 03_BOARD_ENGINE_AND_RULES.md Section 12
       (only fires if L ≥ 1; if the Board Engine emits no LinesCleared event, this entire
       contract is skipped for the Turn — PlacementCommitted/BoardChanged alone are consumed
       with no DamageEvent/ScoreEvent produced, per Section 5/9 step 2)

1.  L = |filledRows| + |filledCols|
2.  ClearValue = Σ BD(n), n = 1..L                            [Section 6, Table 23.1]
3.  ComboMult = lookup(L)                                       [Section 7, Table 23.2]
4.  StageA = ClearValue × ComboMult
5.  RelicFlat, RelicMult = aggregate(activeRelics, Section 13.1) [Section 13]
6.  StageB = (StageA + RelicFlat) × RelicMult
7.  ObjMod = activeObjective.damageModifier ?? 1.0               [Section 16]
8.  StageC = StageB × ObjMod
9.  ScoreValue = round(StageC)                                    → emit ScoreEvent { ScoreValue }
10. EnemyDef = currentEnemy.defense                                [Section 11]
11. StageD = StageC − (StageC × EnemyDef.percent) − EnemyDef.flat
12. FinalDamage = max(0, round(StageD))                            [clamp — ONLY here]
13. EnemyHP -= FinalDamage; EnemyHP = max(0, EnemyHP)               [Section 10]
14. emit DamageEvent { L, ClearValue, ComboMult, RelicFlat, RelicMult,
                        ObjMod, EnemyDef, FinalDamage, ScoreValue,
                        isFullCombo: (L≥3) }                        [Section 8, Section 9]

15. return control to 01_GAMEPLAY_SPECIFICATION.md OBJECTIVE_EVALUATION
    → if EnemyHP ≤ 0 OR non-HP Primary Objective satisfied: ENEMY_DEFEATED → BATTLE_VICTORY
        → Bonus Objective check (Section 16) → supplementary reward if met
        → Relic Selection: 3 offered, 1 chosen (Section 17)
        → Transition → next Battle
    → else: proceed to ENEMY_ACTION (Gameplay Spec Section 10), loop to ACTIVE

16. (at eventual RUN_DEFEAT, any phase)
    PersistentCurrency = (BattlesWon × 5) + (BossesDefeated × 15)   [Section 18, Table 23.5]
    → granted at Rewards (Gameplay Spec Section 3)
```

Every step above is deterministic given the Board Engine's input event, the active Relic set, and the active Objective's declared modifier (Section 16) — no hidden or undocumented value may influence any step, matching the determinism standard already established by `03_BOARD_ENGINE_AND_RULES.md` Section 13 and `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 13.

---

**End of `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`.**
