# 07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md
## Block Battles — Battle Objectives and Level Design

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `06_INPUT_DRAG_AND_DROP_UX.md`
**Document status:** Implementation-level content and encounter-design system

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23. It is the **Objectives / Block Quest doc** referenced by `01_GAMEPLAY_SPECIFICATION.md` Section 25, Section 12–13 (the Objective category framework and Primary/Bonus distinction), `02_SHAPE_LIBRARY.md` Section 9 (shape-predicate Objectives), `03_BOARD_ENGINE_AND_RULES.md` Section 20 (Board Queries/events as the only legal Objective data source, and the `INDESTRUCTIBLE` cell state explicitly assigned to this document's ownership), `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 8/11 (Objective-Aware Generation and Encounter Parameters), and `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 16 (the Objective Modifier stage contract). It introduces exactly one additive extension to the existing tracked-state model (Section 5, Damage Threshold) and reframes zero existing rules — every mechanic below is built from primitives those six documents already expose.

---

## 1. Purpose

This document owns the **handcrafted encounter and content layer**: the Objective framework, the reusable Objective vocabulary, Encounter Modifiers, Boss design rules, the Battle Authoring Template, and the MVP content plan. It does not own Board mechanics (`03`), Shape geometry (`02`), Tray generation (`04`), Damage/Score/Relic formulas (`05`), or input/gesture feel (`06`) — it consumes all five as fixed primitives and recombines them into authored content.

**Core mandate (Master Prompt, restated as a binding rule):** every Battle must be produced by **recombining a small, reusable set of primitives** — Objective categories (Section 5), Encounter Modifiers (Section 8), and Difficulty Tiers (Section 14) — never by inventing a new mechanic per encounter. This is the same discipline `01_GAMEPLAY_SPECIFICATION.md` Section 8 already applied to Block Quest in general; this document is where that discipline becomes concrete, authorable content.

---

## 2. Level Design Philosophy

**DESIGN DECISION:** A Battle creates **tension** through Board-state pressure and Objective proximity (never a Player HP countdown, per `00_MASTER_GAME_VISION.md` Section 10 / `05` Section 12's binding exclusion), **goals** through a single legible Primary Objective plus an optional Bonus Objective (Gameplay Spec Section 13), **decision-making** through the interaction of the current Tray, the Objective's constraint, and the player's Relic build (`05` Section 15), **mastery** through the same reusable Objective vocabulary recurring in new combinations (so pattern-recognition genuinely transfers across encounters), and **replayability** through Section 17's build/RNG variance rather than through hand-authored branching content.

**MASTER DESIGN RULE (inherited, Master Vision Section 23, rule 5):** An encounter that requires a bespoke, one-off mechanic to be fun is not shippable as designed — it must be decomposed into existing primitives or the primitive set itself must be revised here first (Section 4).

---

## 3. Encounter Anatomy

Every encounter (standard Battle, Objective-modified Battle, or Boss) is fully described by:

```text
Encounter Identity      — id, display name, environment/flavor text (no engine dependency)
+ Enemy                 — HP band + Defense (05 Sections 10-11, Table 23.3/23.6); identity/flavor owned by a future Enemy Content doc
+ Board Rules           — standard empty-start Board, or a Section 8 pre-fill Modifier
+ Primary Objective     — exactly one, from the Section 5 vocabulary (Gameplay Spec Section 13)
+ Optional Bonus Objective — zero or one, from the same vocabulary (Gameplay Spec Section 13)
+ Difficulty            — one Section 14 Tier, driving Encounter Parameters (04 Section 11)
+ Reward                — standard Relic Selection (05 Section 17) + any Bonus-triggered supplement
```

This is the schema every Section 18 Authoring Template instance and every Section 27 `BattleDefinition` populates. No encounter field exists outside this list at Absolute MVP.

---

## 4. Objective System

Every Objective instance — regardless of category (Section 5) — is defined by exactly this schema:

| Field | Description |
|---|---|
| `id` | Unique identifier, format `OBJ_0##`. |
| `name` | Player-facing short name (Section 11). |
| `objective_type` | One of the Section 5 category codes. |
| `trigger` | Which existing event stream (Board Engine Section 12, Scoring doc Section 9/27) this Objective's counter subscribes to — never a new event type (Section 12, Objective Timing). |
| `counter` | The specific tracked value this Objective accumulates or checks — always one of the values enumerated in Section 5's "Tracked State" column, never an invented one. |
| `success_condition` | A deterministic comparison against `counter` (e.g., `counter >= target`). |
| `failure_condition` | Optional; only present for categories that can independently end the Battle in failure (Section 6). |
| `progress_state` | The live value of `counter` relative to `target`, exposed for UI (Section 11). |
| `ui_representation` | A reference to Section 11's communication contract — this field never specifies pixel-level UI (per the Master Prompt's explicit instruction). |
| `reward_consequence` | Primary (unlocks Relic Selection, Section 3) or Bonus (supplementary reward only, Section 6). |

**MASTER DESIGN RULE (inherited, Gameplay Spec Section 12):** "Every Objective is evaluated purely from state already tracked by this specification" — Section 5 below is the complete, closed enumeration of what `counter` may reference. Adding a new `counter` source requires a Section 5 revision first, exactly as Gameplay Spec Section 12 requires of itself.

---

## 5. Objective Categories

**DESIGN DECISION:** Ten atomic categories — nine inherited verbatim from `01_GAMEPLAY_SPECIFICATION.md` Section 12, plus exactly one additive extension (Damage Threshold) justified below. This is the complete, closed vocabulary; it is intentionally not "dozens of objective types."

| Category | Gameplay Spec Section 12 origin | Tracked State (`counter`) | Notes |
|---|---|---|---|
| **Defeat** | "Defeat Enemy" (default Primary) | Enemy HP (`05` Section 10) | The default for standard Battles. |
| **Line Count** | "Clear X Total Lines" | Cumulative `lineCount` across Turns (sum of `LinesCleared.lineCount`, `03` Section 12) | Rows + columns combined. |
| **Row Count** | "Clear X Rows" | Cumulative distinct rows cleared (`03` Section 12's `rowsCleared`, deduplicated across Turns) | |
| **Column Count** | "Clear X Columns" | Cumulative distinct columns cleared (`03` Section 12's `colsCleared`, deduplicated) | |
| **Combo** | "Achieve X Combo" | Single-Turn `L` value (`05` Section 4/7) — never cumulative (Gameplay Spec Section 21 edge case) | Satisfied the instant any one Turn's `L ≥ target`. |
| **Move Limit** | "Defeat Enemy Under Move Limit" | Turn counter (Gameplay Spec Section 7) vs. a maximum | See Section 13 for exact counting rules. |
| **Survival** | "Survive X Turns" | Turn counter vs. a target, Enemy still alive | Primary win condition variant, not a failure clause by itself. |
| **Special Piece / Behavior** | "Clear Lines Using a Particular Behavior" | A structural predicate over the Shape that produced a given Clear, evaluated against `02_SHAPE_LIBRARY.md` `category`/`tags` (Shape Library Section 8–9) — e.g. "the clearing Placement's Shape was tagged `SINGLE`" | This is the family that covers "1×1 piece" and "≥5-block piece" style requirements the Master Prompt names; it is a sub-type of the existing category, not a new one. |
| **Conditional Phase** | "Complete Condition Before Enemy Reaches Phase" | A tracked condition vs. an Enemy-side action counter (incremented at `ENEMY_ACTION`, Gameplay Spec Section 10) | Owned mechanically by a future Enemy Content doc's phase definitions; this document only defines the comparison shape. |
| **Damage Threshold** *(extension)* | Not present in Gameplay Spec Section 12 as written | Cumulative `FinalDamage` across Turns this Battle | See justification below. |

**DESIGN DECISION — Damage Threshold justification:** this category requires no new emitted event and no new Board/Combat mechanism — it sums `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 9's already-emitted `DamageEvent.FinalDamage` across Turns, exactly the same way Line Count sums `LinesCleared.lineCount`. It is therefore additive to, not in tension with, Gameplay Spec Section 12's closed-tracked-state rule. **OPEN DECISION:** `04_SPAWN_RNG_AND_DIFFICULTY.md` Table 17.6 currently has no row for this category; per that document's own Section 8 rule ("An Objective category with no entry in Table 17.6 receives no reweighting at all"), Damage Threshold defaults to neutral generation weighting until a future revision of `04` adds an explicit entry — no `04` edit is required for this category to function correctly today.

**What was deliberately not added as an 11th category:** "Conditional Battle" (Master Prompt) is modeled as a **combination pattern** (Section 7), not an atomic category — see Section 7's worked example. This keeps the vocabulary count small per Section 1's mandate rather than growing it for a pattern that decomposes cleanly into existing primitives.

---

## 6. Primary vs Bonus Objectives

Restated with this document's specific content rules, inheriting `01_GAMEPLAY_SPECIFICATION.md` Section 13's structural definition without alteration:

- **Primary Objective:** exactly one per encounter (Section 3); its `success_condition` ends the Battle in victory; any category-specific `failure_condition` (Move Limit exceeded, Board Lock) ends the Run (Gameplay Spec Section 15).
- **Bonus Objective:** zero or one per encounter; evaluated once, at `BATTLE_VICTORY` (Gameplay Spec Section 4), against data accumulated during the Battle; **never** independently fails the Battle — a Bonus Objective's `failure_condition` field is always empty by construction (Section 4's schema still declares the field for uniformity, but Bonus instances leave it null).
- **Reward implications — DESIGN DECISION:** meeting Primary always grants standard Relic Selection (`05` Section 17, 3 offered / 1 chosen). Meeting Bonus additionally grants a supplementary reward. **This document fixes the supplementary reward type at Absolute MVP as a `PersistentCurrency` bonus** (`05` Section 18/Table 23.5) — a flat addition on top of the standard per-Battle-Won accrual, rather than a fourth Relic offer or any new reward primitive. This keeps the reward surface to the single currency Master Vision Section 12's ceiling already allows, and avoids inventing a Relic-count exception to `05` Section 17's fixed "3 offered / 1 chosen" rule. **ASSUMPTION (placeholder value, pending Section 24-style simulation):** Bonus completion grants **+3 `PersistentCurrency`**, flat, regardless of Tier (Section 14).
- **Progression influence:** Bonus completion does not currently unlock anything beyond the currency bonus above — consistent with `01_GAMEPLAY_SPECIFICATION.md` Section 18's explicit statement that unlock/spend systems are out of scope at Absolute MVP.

---

## 7. Objective Combinations

**This is the mechanism that produces roguelite-scale variety from ten primitives (Section 5).**

**DESIGN DECISION:** An encounter's Primary Objective may be **paired with** (a) a Bonus Objective from a *different* category, and/or (b) one or more Encounter Modifiers (Section 8). The Objective System itself (Section 4) never allows two simultaneous Primary Objectives — combination happens at the encounter-authoring level (Section 18), not inside the Objective schema.

**Worked examples (Master Prompt's named cases, mapped onto the Section 5 vocabulary):**

| Combination | Primary | Bonus / Modifier |
|---|---|---|
| "Defeat the Goblin within 12 placements." | **Defeat** | Encounter Modifier: **Move Limit** (Section 8) attached to the Defeat Primary — mechanically this *is* the "Defeat Enemy Under Move Limit" category (Section 5), authored as Defeat + a Move Limit Modifier rather than a separate category, per the Master Prompt's own framing of it as a combination. |
| "Defeat the Ice Mage after clearing 3 columns." | **Defeat** | Bonus: **Column Count** (target 3) — Bonus completion here is authored as *required-before-victory-is-meaningful* flavor, but mechanically remains a true Bonus (Section 6): the Battle can still be won by Defeat alone; the design intent (Section 18 field) documents that this pairing is the intended read. |
| "Survive 10 turns while maintaining a combo." | **Survival** (target 10) | Bonus: **Combo** (e.g., target `L≥2`, re-checked... — see note below) |
| "Defeat the Dragon after performing one 3-line clear." | **Defeat** | Bonus: **Combo** (target `L≥3`, matches `05` Section 8's `isFullCombo` flag directly — zero new tracking required) |

**Note on "maintaining" a Combo across Survival:** because Combo (Section 5) is strictly single-Turn (Gameplay Spec Section 21 edge case, inherited by `05` Section 7), "maintaining a combo" across a Survival window is authored as **"achieve `L≥2` at least once during the Survival window,"** not as a persistent streak requirement — this keeps the combination decomposable into existing, unmodified primitives rather than inventing a new streak-tracking mechanism.

**"Conditional Battle" resolved as a combination, not a category (Section 5):** *"Defeat the enemy while a Hazard remains active"* is authored as **Defeat** Primary + an Encounter Modifier (Section 8) that keeps a Hazard-category Board effect present for the Battle's duration — the "condition remaining active" is a Modifier-layer guarantee (an authoring/content constraint on what the Encounter spawns), not a new Objective-evaluation code path.

---

## 8. Encounter Modifiers

| Modifier | Purpose | Rules | Player Readability | Difficulty Impact | Implementation Complexity | MVP or Later |
|---|---|---|---|---|---|---|
| **Move Limit** | Add pressure independent of Board state. | Attaches a maximum Turn count to any Primary Objective (Section 13 defines exact counting). | Shown as a remaining-moves counter (Section 11). | Moderate–high, scales with limit tightness. | Low — reuses Gameplay Spec Section 7's existing Turn counter. | **MVP.** |
| **Enemy Armor** | Make an Enemy mitigate Damage. | Sets non-zero `EnemyDef.percent`/`.flat` (`05` Section 11) for this Encounter's Enemy. | Shown via visibly reduced Damage-per-clear relative to Enemy HP (no separate "armor" UI required at MVP). | Moderate, scales with value. | Low — `05` Section 11 already supports arbitrary values per Enemy. | **MVP.** |
| **Special Line Requirement** | Force spatial-planning variety. | A Special Piece / Behavior Objective (Section 5) referencing `02` `category`/`tags`. | Objective description states the required category/tag in plain language (Section 11). | Low–moderate, scales with how restrictive the tag is. | Low — pure predicate check against already-emitted Shape metadata. | **MVP.** |
| **Soft Pre-fill (ordinary `FILLED` cells)** | Start a Battle with a partially-occupied Board for immediate spatial tension. | Uses only the MVP `FILLED` state (`03` Section 3.1) — pre-filled cells are ordinary and clearable like any player-placed cell. | Visible on Board at `PREPARING` (Gameplay Spec Section 4). | Low–moderate. | Low — no new Board Engine state required. | **MVP.** |
| **Hard Pre-fill (`INDESTRUCTIBLE` obstacles)** | Permanent, never-clearable obstacle cells for a themed encounter. | Requires `03_BOARD_ENGINE_AND_RULES.md`'s reserved `INDESTRUCTIBLE` state (Section 3.2), explicitly assigned there to this document's ownership for *which cells and why* — but that state is Extensible-only until a Gameplay Spec revision activates it (`03` Section 3.2's own rule: "None may be added to MVP scope... without a revision to `01`"). | Would be visually distinct from ordinary `FILLED` (owned by a future Art doc). | Structural — permanently shrinks the playable Board. | Low once activated; presently blocked on a cross-document activation step this document cannot perform unilaterally. | **Later (MVP+)** — this document defines the *intent*; `01`/`03` must jointly activate the state first. |
| **Increased Enemy Pressure** | Raise threat via more frequent/severe Enemy Telegraphs. | References `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 10's "Enemy pressure" signal and Section 12's difficulty-band posture; concrete Telegraph behaviors are Gameplay Spec Section 11 mechanics. | Always Telegraphed one Turn ahead (Gameplay Spec Section 5.6, inherited, non-negotiable). | High, but only as steep as the concrete Enemy mechanics allow. | Depends entirely on the future Enemy Content doc's mechanic definitions. | **Later** — the dial exists (`04` Section 12), the mechanics it drives do not yet. |
| **Frozen Cells** | Stall a specific line from completing. | Uses `03` Section 3.2's `FROZEN` state. | Visually distinct from `FILLED` (future Art doc). | Moderate–high, forces rerouting. | Low — `FROZEN` logic is now MVP-active via `03` Section 9's predicates. | **MVP.** |
| **Blocked Cells** | Deny placement into specific cells temporarily. | Uses `03` Section 3.2's `BLOCKED` state. | Visually distinct, non-interactable cell marker. | Moderate. | Low — logic is MVP-active. | **MVP.** |

**MASTER DESIGN RULE (inherited, `03` Section 3.2):** No "Later" row above may be promoted to MVP by this document alone — each requires the named cross-document activation step first. This document is not overbuilding these; it is registering intent against seams other documents already reserved.

---

## 9. Enemy + Objective Synergy

**Flavor-naming caveat (per the Master Prompt's own Section 22 instruction, honored here in Section 9 as well):** "Ice Mage," "Stone Golem," "Goblin," and "Dragon" below are **illustrative archetype labels only**. No Enemy Content doc yet exists; this document's Objective/Modifier framework is fully agnostic to Enemy identity, and none of these names are hardcoded anywhere in Sections 4–8's mechanisms.

| Archetype (flavor only) | Illustrative Objective | Primitives used | MVP-ready today? |
|---|---|---|---|
| "Ice Mage" | *Clear 3 rows before the third freeze cycle.* | **Row Count** Primary + **Conditional Phase**-style Enemy-action counter, dependent on the `FROZEN` state / freeze-cycle mechanic. | **Yes** — `FROZEN` is now MVP-active. |
| "Stone Golem" | *Break all 3 armor stacks.* | **Defeat** Primary, authored as 3 sequential HP thresholds within one Battle's Enemy HP pool, each threshold representationally an "armor stack" — no new mechanic beyond Enemy HP (`05` Section 10). | **Yes**, if "3 stacks" is authored as 3 equal HP segments of one pool rather than a literal separate armor resource (which `05` Section 10 explicitly excludes — "Shields/Armor... not implemented... as a separate HP-like pool"). |
| "Goblin" | *Defeat without exceeding 15 placements.* | **Defeat** Primary + **Move Limit** Modifier (Section 8). | **Yes**, fully MVP-ready. |
| "Dragon" | *Achieve at least one 3-line clear.* | **Defeat** Primary + **Combo** Bonus (target `L≥3`, `05` Section 8's `isFullCombo`). | **Yes**, fully MVP-ready. |

---

## 10. Objective Fairness

Inherits `01_GAMEPLAY_SPECIFICATION.md` Section 18 and `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 1 without exception. Concrete rules specific to authoring:

- **Understandable before Battle:** every Objective's `name` and plain-language description (Section 11) must be shown at `PREPARING` (Gameplay Spec Section 4), before the first Turn — never revealed mid-Battle.
- **Achievable using known mechanics:** an Objective may only reference Section 5 categories and Section 8 MVP-tagged Modifiers; a "Later" Modifier (Section 8) may not appear in a shipped Absolute MVP encounter.
- **Never dependent on impossible RNG:** any Special Piece / Behavior Objective (Section 5) referencing a `02_SHAPE_LIBRARY.md` `category`/`tag` inherits `04` Section 5's Solvability Protection guarantee automatically — the Tray is never dead-on-arrival regardless of which tag an Objective names, and `04` Section 8 additionally forbids this document from *disfavoring* Shapes an active Objective needs (a strict floor of "neutral or better").
- **Sufficient room for strategy:** a Move Limit (Section 8) must never be set tighter than the minimum Turn count a Heuristic-lookahead-tier virtual player (`04` Section 14, Policy 3) requires in `04`/`05`'s joint simulation — see Section 25's playtesting gate.
- **No hidden requirements:** every field in Section 4's schema is player-visible in some form (Section 11) except `trigger`/`counter`, which are internal wiring with a directly corresponding visible `progress_state`.

---

## 11. Objective Communication

What the player must be able to see, without specifying final UI art (per the Master Prompt's explicit instruction — this section names the *information*, not the *pixels*):

| Element | Source field (Section 4) | When shown |
|---|---|---|
| Objective name | `name` | From `PREPARING` onward. |
| Description (plain language) | Derived from `objective_type` + `target` (e.g., "Clear 5 total lines") | From `PREPARING` onward. |
| Progress | `progress_state` | Updated at every Section 12 evaluation point. |
| Remaining amount | `target − progress_state` (or Turns remaining, for Move Limit/Survival) | Continuously visible during `ACTIVE` (Gameplay Spec Section 4). |
| Failure state | Only for categories with a non-null `failure_condition` (Move Limit, Board Lock) | Shown the instant the condition is approached/tripped — never silently. |
| Completion state | `success_condition` becoming true | Shown at `BATTLE_VICTORY` (Gameplay Spec Section 4), coincident with `06_INPUT_DRAG_AND_DROP_UX.md` Section 12's Objective Evaluation feedback stage. |

---

## 12. Objective Timing

**DESIGN DECISION:** Objective evaluation is not a new timing model — it nests inside two already-canonical sequences: `03_BOARD_ENGINE_AND_RULES.md` Section 21 (Final Turn Resolution) and `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 27 (Final Combat Resolution Contract). Every Section 5 category's `counter` updates at exactly one of these existing points:

```text
After Placement (03 Section 21, step 2 — commitPlacement)
   → no Objective counter updates here (Placement alone has zero Score/Damage, 05 Section 5)

After Clear Resolution (03 Section 21, step 4 — clearLines / LinesCleared / CellsCleared)
   → Line Count, Row Count, Column Count cumulative counters update
   → Combo counter is set fresh from this Turn's L (Section 5; never cumulative)

After Damage (05 Section 27, steps 9-14 — DamageEvent)
   → Damage Threshold cumulative counter updates (Section 5 extension)
   → Defeat's counter (Enemy HP) updates

After Enemy Action (Gameplay Spec Section 10, ENEMY_ACTION)
   → Conditional Phase's Enemy-side action counter updates (future Enemy Content doc mechanic)

At Battle End (Gameplay Spec Section 4, BATTLE_VICTORY)
   → Bonus Objective checked once (Section 6), against final accumulated counters
   → Move Limit / Survival's Turn-counter comparison is finalized
```

This is fully deterministic: every counter update is anchored to an existing, already-deterministic event (`03` Section 13, `05`'s inherited determinism) — Objective Evaluation introduces no new source of nondeterminism.

---

## 13. Move Limits

**DESIGN DECISION, resolved unambiguously per the Master Prompt's explicit questions:**

| Question | Answer |
|---|---|
| Do invalid placements count? | **No.** An invalid Placement never reaches `commitPlacement` (`03` Section 4–5); the Turn counter (Gameplay Spec Section 7) only increments on a *resolved* Placement, and `06_INPUT_DRAG_AND_DROP_UX.md` Section 11 confirms an invalid release triggers only a return animation, never a Board mutation or Turn increment. |
| Do valid placements count? | **Yes, always.** One resolved Placement = one Turn (Gameplay Spec Section 7) = one Move Limit consumption, with no exception. |
| Do automatic events count? | **No.** `ENEMY_ACTION` (Gameplay Spec Section 10) is a distinct state from a Turn's Placement-resolution chain; it never increments the Turn counter itself. |
| Do Enemy turns count? | **No**, for the same reason — Move Limit is defined purely in Turn-counter terms (player Placements), never in wall-clock or Enemy-action terms. |

This makes Move Limit trivially reproducible under `03` Section 13's and `05`'s determinism guarantees: given an identical sequence of valid Placements, the Move Limit's remaining value is always identical.

---

## 14. Difficulty Tiers

**DESIGN DECISION:** Five content-authoring Tiers, distinct in *purpose* from `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 12's three *generation-weighting* bands (Early/Mid/Late-run) — the two systems serve different layers (authored encounter design vs. procedural Tray weighting) and are cross-referenced, not merged, to avoid contradicting either document's ownership.

| Tier | Definition | Typical Objective shape | Typical `04` band alignment |
|---|---|---|---|
| **Tier 1 — Tutorial** | One obvious Objective, no Modifier. | Bare **Defeat**, generous Enemy HP floor (Table 23.3 low end). | Early-run. |
| **Tier 2 — Tactical** | Objective plus one complication. | **Defeat** + one Section 8 MVP Modifier, or a Bonus from a different category (Section 7). | Early-to-Mid-run. |
| **Tier 3 — Advanced** | Multiple interacting constraints. | Primary + Bonus + one Modifier, chosen so their constraints genuinely interact (e.g., Move Limit tight enough that Combo-seeking becomes the efficient path). | Mid-run. |
| **Tier 4 — Elite** | High-pressure combination. | Primary + Bonus + Modifier at tightened parameters (Section 17-style tuning, this document's own equivalent table lives in Section 19). | Mid-to-Late-run. |
| **Boss** | Distinct structure (Section 21) | Elevated Enemy HP/Defense (`05` Table 23.6) + exactly one major mechanic. | Any phase, per content cadence (`00` Section 14). |

**MASTER DESIGN RULE:** No sixth Tier is added without revising this document first — five is the deliberately small ceiling matching Master Vision Section 12's complexity-budget discipline.

---

## 15. Handcrafted Battle Structure

**DESIGN DECISION:** Every Battle in Block Battles is handcrafted **at the parameter level**, never procedurally generated at the structural level. There is no "procedural encounter generator" that invents new Objective/Modifier combinations at runtime — `04_SPAWN_RNG_AND_DIFFICULTY.md` owns only Tray *content* generation (Section 4–5 of that document), not Battle *structure* generation, and this document's Section 18 Template is the sole mechanism for defining new Battles.

**Coexistence with procedural systems:** a handcrafted `BattleDefinition` (Section 27) declares its Objective/Modifier/Enemy-band choices explicitly (authored), while the *Tray* the player actually draws within that Battle is procedurally generated per `04`'s existing pipeline, filtered/reweighted by whatever Encounter Parameters (`04` Section 11) the `BattleDefinition` specifies. This is precisely how "the strategic variety of a roguelite" (Section 1's mandate) coexists with "avoiding hundreds of unique gameplay systems" — the *structure* is authored and reused; the *moment-to-moment puzzle* is procedural within that structure.

---

## 16. Roguelite Run Integration

Restated with no new state (fully owned by `01` Section 16, `05` Section 19):

```text
Battle → Victory → Reward → Relic Choice → Next Battle
```

**Why the same handcrafted Objective still feels different on replay:** a fixed `BattleDefinition` (e.g., "Goblin — Defeat under 15 placements," Section 9) is deterministic in its *structure* but not in its *resolution* — the specific Trays drawn (`04`), the Relics the player has accumulated by that point (`05` Section 13–15), and the resulting optimal placement sequence all vary per Run. A player holding RLC_004 (Chain Reaction, `05` Table 23.4) approaches a Move-Limit encounter by banking Combos; a player holding RLC_001/002 (Single-Minded/Efficient Filler) approaches the same encounter via frequent small clears. The encounter's authored parameters (Section 18) are identical; the player's *route* through them is not.

---

## 17. Objective Replayability

Four independent sources of variance apply to every replay of the same `BattleDefinition`, none requiring new authored content:

1. **Relic build variance** (`05` Sections 13–15) — which of the 8 launch Relics the player holds by this Battle changes the efficient strategy (Section 16).
2. **Tray RNG variance** (`04` Sections 2–9) — the specific Shape sequence drawn, within the Encounter's Objective-Aware/Board-Aware reweight bounds (`04` Table 17.6/17.2), differs every Run.
3. **Board-state variance** — because Trays are consumed against whatever Board state the player's own prior Placements produced, no two Runs reach an identical Board at the same Turn even under identical Tray RNG.
4. **Strategic variance** — the same fixed Objective (e.g., Combo target `L≥3`) admits multiple valid solving routes (Master Vision Section 5.2's Tactical Placement pillar), so even a "solved" encounter remains a puzzle each time.

**DESIGN DECISION:** This document deliberately does not add a fifth variance source (e.g., randomized Objective parameters per replay) — the four above are already judged sufficient per Master Vision Section 5.8's solo-developer feasibility discipline, and randomizing Objective parameters would undermine Section 10's "understandable before Battle" fairness rule if not done carefully; it is left as a Master Vision Section 15 expansion decision, not a default.

---

## 18. Battle Authoring Template

Standardized template, expanded significantly per the Master Prompt's instruction, for every new `BattleDefinition` (Section 27):

```text
Battle ID:                 (unique, format BTL_0##)
Name:                       (player-facing flavor name)
Environment:                 (flavor/theme only — no engine dependency, owned by future Art doc)
Enemy Archetype (flavor):     (illustrative name only, per Section 9's caveat)
Enemy HP Band:                (one of 05 Table 23.3's Early/Mid/Late bands, or Table 23.6's Boss multiplier)
Enemy Defense:                 (EnemyDef.percent / .flat, 05 Section 11 — 0/0 unless this is an "Enemy Armor" Modifier Battle)
Special Rule / Modifier(s):     (zero or more Section 8 MVP-tagged Modifiers — never a "Later" row)
Primary Objective:              (one Section 5 category + numeric target)
Bonus Objective:                (optional; one Section 5 category, different from Primary, + numeric target)
Move Limit:                     (numeric, or "none")
Difficulty Tier:                (one Section 14 Tier)
Expected Duration:               (Turn-count estimate, informing 04 Section 15's "average battle duration" metric)
Rewards:                          (standard 3-offer/1-choice Relic Selection, 05 Section 17; + Section 6's Bonus currency if applicable)
Design Intent:                    (free text — what decision this Battle is meant to teach or test, tying back to Section 2's philosophy)
Failure Risks:                     (free text — what a designer should watch for in playtesting per Section 10's fairness rules)
Playtest Notes:                    (free text, populated post-Section 25 playtesting)
```

**MASTER DESIGN RULE:** every field above must resolve to a value already defined by Sections 5, 8, or 14 of this document, or by a numbered Table in `05`/`04` — a Template instance containing a value not traceable to an existing primitive is not a valid `BattleDefinition` (Section 27 enforces this structurally).

---

## 19. MVP Content Plan

**DESIGN DECISION, reconciled explicitly against prior documents:**

| Content | This document's target | Justification |
|---|---|---|
| Standard Enemy archetypes | **4** | Matches `00_MASTER_GAME_VISION.md` Section 14's "low single-digit archetype count" and the Master Prompt's suggested figure directly; sized so `05` Table 23.3's three HP bands each get meaningful reuse across Battles rather than one archetype per Battle. |
| Bosses | **2** | Within Master Vision Section 14's "1–3" range; supports the two-Boss-cadence-point progression in Section 20 below (one Mid-run, one Late-run), matching `04` Section 12's Mid-run "first Boss cadence point" note. |
| Relics | **8** (already fixed) | Not a new decision — `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 14 already fixes exactly 8 launch Relics; this document does not reopen that number. |
| Handcrafted encounters | **18** (placeholder, ASSUMPTION pending Section 25 playtesting) | Within the Master Prompt's suggested 15–25 range; large enough to populate Section 20's progression template twice over with variation, small enough for solo-developer authoring load (Master Vision Section 5.8) using the Section 18 Template. |
| Objective categories | **10** (Section 5, closed) | Already the complete, closed vocabulary — "a small number of reusable objective types" is satisfied by construction, not by further reduction. |

**Why these figures and not larger ones:** every figure above is sized to the same Master Vision Section 12 complexity-budget discipline already governing Relics (8), Shapes (34 entries/15 families), and RNG tables (`04` Section 17) — adding more Enemies/Bosses/encounters without a proven need would grow authoring load (Section 26) without a corresponding increase in *decisions* (Master Vision Section 11's Player Decision Hierarchy), which Section 1's mandate explicitly rules out.

---

## 20. Encounter Progression

**DESIGN DECISION:** A representative 12-Battle progression template, explicitly aligned to `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 12's existing band boundaries (Early 1–3, Mid 4–8 including "first Boss cadence point," Late 9+) rather than inventing new boundaries:

```text
Battle 1  — Tier 1 (Tutorial)
Battle 2  — Tier 2 (Tactical)
Battle 3  — Tier 2 (Tactical)
Battle 4  — Tier 3 (Advanced)
Battle 5  — Tier 3 (Advanced)
Battle 6  — Boss #1               (Mid-run cadence point, 04 Section 12)
Battle 7  — Tier 2 (Tactical)      — intentional breather after Boss #1
Battle 8  — Tier 3 (Advanced)
Battle 9  — Tier 4 (Elite)
Battle 10 — Tier 4 (Elite)
Battle 11 — Tier 3/4 (Advanced–Elite)
Battle 12 — Boss #2                (Late-run)
```

**DESIGN DECISION:** This is a *template*, not a fixed Run length — `01_GAMEPLAY_SPECIFICATION.md` Section 16 does not require a Run-completion condition at Absolute MVP (Open Decision carried forward), so this progression is one authored sequence among potentially several (Section 19's 18 encounters support more than one such sequence with variation), not the only possible Run shape.

---

## 21. Boss Design Rules

**DESIGN DECISION:** A Boss introduces exactly:

- **One primary identity** (a flavor archetype, Section 9's caveat still applies — no hardcoded engine dependency on the name).
- **One major mechanic**, expressed as exactly one Section 8 Modifier (MVP-tagged only) or one elevated-stat posture (`05` Table 23.6's HP ×2.0 / Defense +0.10/+5).
- **Possibly phases** — modeled, per Section 9's "Stone Golem" example, as sequential HP-threshold segments within one Enemy HP pool (`05` Section 10), never as a separate resource type.
- **Clear readability** — a Boss's Telegraph (Gameplay Spec Section 10) and Objective (Section 11) are held to the exact same fairness/communication standard as any standard Battle; nothing about "Boss" status relaxes Section 10's rules.

**MASTER DESIGN RULE (inherited, Master Prompt "avoid multi-system boss designs in MVP"):** a Boss may combine at most **one** Modifier with its elevated stat posture — never two or more simultaneous Modifiers — at Absolute MVP. This is this document's concrete enforcement of that instruction.

---

## 22. Boss Examples

Using the same Section 9 flavor-naming caveat, and the same MVP-readiness honesty:

| Boss (flavor) | One major mechanic | Phases? | MVP-ready? |
|---|---|---|---|
| **Ice Mage** | Freeze-cycle pressure (`FROZEN` state) | No (single mechanic per Section 21) | **Yes.** |
| **Stone Golem** | 3 sequential HP-threshold "armor stack" phases within one HP pool | Yes — 3 phases, still "one mechanic" (segmented Defeat) per Section 21's phase allowance | **Yes.** |
| **"Goblin King" (equivalent)** | Elevated Move Limit pressure (tighter than a standard Tier 2 Goblin's, Section 9) | No | **Yes.** |
| **Dragon** | Elevated Combo requirement (e.g., Bonus target raised to `L≥4`, `05` Table 23.2's capped tier) paired with elevated HP/Defense (Table 23.6) | No | **Yes.** |

**Architecture reusability confirmation:** none of the four rows above required a new Objective category (Section 5), a new Modifier type (Section 8), or a new Board state beyond what `03` Section 3.2 already reserves — exactly the Master Prompt's "reusable architecture, not hardcoded names" requirement.

---

## 23. Objective Anti-Frustration Rules

Directly enumerated, each traceable to an existing document's rule so none is asserted without grounding:

1. **No extremely rare Shapes required.** Every Special Piece / Behavior Objective (Section 5) must reference a `category`/`tags` group with multiple registry members (`02_SHAPE_LIBRARY.md` Section 5–6), never a single specific `id` — inherits `02` Section 8–9's "reference by metadata, never hardcode `id`" rule directly.
2. **No impossible RNG.** Inherits `04` Section 5's Solvability Protection guarantee unconditionally (Section 10 above).
3. **No exact obscure sequences.** An Objective's `success_condition` (Section 4) is always a simple threshold comparison (`counter >= target` or `counter == target` for phase-style conditions) — never an ordered sequence of specific actions, which `04`/`05`'s existing systems have no mechanism to track anyway.
4. **No hidden mechanics.** Section 11's communication contract is mandatory for every shipped Objective; an Objective whose `counter` cannot be expressed in plain language per Section 11 is not shippable.
5. **No excessive repetition.** Section 19's fixed content ceilings (10 categories, 18 encounters) combined with Section 7's combination mechanism are the structural prevention — variety comes from recombination, not from grinding one category's numeric target upward indefinitely.

---

## 24. Objective Balance Metrics

**DESIGN DECISION:** This document extends `04_SPAWN_RNG_AND_DIFFICULTY.md` Section 15's existing Balance Metrics table with Objective-specific rows, rather than defining a parallel metrics system:

| Metric | Definition | Used to validate |
|---|---|---|
| **Completion rate** | Fraction of simulated attempts (per `04`/`05`'s shared Simulation Framework, `04` Section 14) reaching `BATTLE_VICTORY` for a given `BattleDefinition`. | Whether a Tier's difficulty (Section 14) matches its intended band. |
| **Average moves** | Mean Turn count to `BATTLE_VICTORY` for a given `BattleDefinition`, across policy tiers. | Whether "Expected Duration" (Section 18 Template field) is accurate. |
| **Fail rate** | Fraction ending in `RUN_DEFEAT`, split by cause (Board Lock vs. Move-Limit-specific failure, per `04` Section 15's existing "Failure rate" row). | Anti-Frustration Rule 2/3 (Section 23) compliance. |
| **Average damage** | Mean cumulative `FinalDamage` per attempt, reused directly from `05` Section 24's simulation output — relevant specifically to Damage Threshold Objectives (Section 5). | Whether Damage Threshold targets are set within reach of the Greedy-clear policy tier (`04` Section 14, Policy 2), not only the Heuristic-lookahead tier. |
| **Bonus completion rate** | Fraction of victorious attempts that also satisfy the Bonus Objective (Section 6). | Whether Bonus targets are meaningfully harder than Primary without being nearly unreachable — target range TBD-post-simulation, matching the placeholder convention `04` Section 15 and `05` Section 23 already use. |
| **Perceived difficulty** | Qualitative playtester rating (Section 25) — not derivable from simulation alone. | Cross-checks quantitative metrics above against actual human experience, per Section 25's "is it fun" question. |

---

## 25. Playtesting Method

**DESIGN DECISION:** Three explicitly separated questions, tested by different means:

| Question | Method |
|---|---|
| **Can it be completed?** | Automated: run the `BattleDefinition` through `04`/`05`'s shared Simulation Framework across all three virtual-player policy tiers (`04` Section 14). A `BattleDefinition` with a 0% completion rate even at the Heuristic-lookahead tier fails this gate and is not shippable. |
| **Is it fair?** | Automated + rule-check: verify Section 10's and Section 23's rules hold (Solvability Protection inherited, no hidden mechanics, Move Limit not tighter than the Heuristic-lookahead policy's minimum Turn count per Section 10). |
| **Is it fun?** | Human-only: qualitative playtesting session, recorded in the Section 18 Template's "Playtest Notes" field — this question is explicitly never answered by simulation alone, matching `06_INPUT_DRAG_AND_DROP_UX.md` Section 20's own "does this actually feel good" / "functional correctness" split at the input layer, applied here at the encounter layer. |

**MASTER DESIGN RULE:** a `BattleDefinition` must pass "Can it be completed?" and "Is it fair?" (both automatable, both objective) before it is ever exposed to a human playtester for the "Is it fun?" pass — this ordering prevents wasted human playtesting time on a structurally broken encounter.

---

## 26. Content Production Pipeline

```text
Concept
 → Author Objective        (Section 4, choose from Section 5's 10 categories)
 → Assign Enemy              (flavor archetype, Section 9; HP/Defense band, 05 Table 23.3/23.6)
 → Assign Modifier            (zero or more Section 8 MVP-tagged Modifiers)
 → Simulate                    (Section 25's "Can it be completed?" gate)
 → Implement                    (populate a Section 27 BattleDefinition)
 → Playtest                      (Section 25's "Is it fun?" gate)
 → Balance                        (adjust Section 18 Template numeric fields per Section 24 metrics)
 → Polish                          (owned by future Art/VFX docs — no mechanic changes at this stage)
 → Approve                          (Battle enters the Section 20 progression pool)
```

**DESIGN DECISION — solo-developer emphasis:** every step above operates on the same closed primitive set (Sections 5, 8, 14) — authoring a new Battle is a *data-entry* task against the Section 18 Template, not a code-writing task, directly satisfying Master Vision Section 5.8 and Section 20's "data-driven content" philosophy.

---

## 27. Data-Driven Content Model

**DESIGN DECISION:** Every Battle is represented as a `BattleDefinition` data record — never hardcoded logic — conceptually:

```text
BattleDefinition
    id
    name
    environment_flavor
    enemy:
        archetype_flavor
        hp_band            (05 Table 23.3, or Table 23.6 if is_boss)
        defense             (EnemyDef.percent, EnemyDef.flat — 05 Section 11)
    is_boss                  (bool)
    board_start                (empty, or a Section 8 Soft Pre-fill cell set)
    primary_objective:
        objective_type            (one of Section 5's 10 categories)
        target                      (numeric)
    bonus_objective:                (optional; same shape as primary_objective, different category)
        objective_type
        target
    modifiers:                       (zero or more Section 8 MVP-tagged entries)
    move_limit                         (numeric, or null)
    difficulty_tier                     (one of Section 14's five Tiers)
    encounter_parameters                 (04 Section 11's family-weight override / objective-reweight override, if any)
    rewards:
        relic_offer_count = 3                (fixed, 05 Section 17 — never overridden per-Battle)
        bonus_currency = 3                     (Section 6's placeholder value, if bonus_objective is met)
```

This record is the literal input to Section 26's `Implement` step and the literal output of Section 18's Template — the two are the same data, one in author-facing prose form, one in engine-facing structured form.

---

## 28. Acceptance Criteria

A `BattleDefinition` cannot ship unless **all** of the following hold:

- Its `primary_objective.objective_type` is one of Section 5's 10 categories, and every `modifiers` entry is MVP-tagged in Section 8 (no "Later" row present).
- Its Objective(s) are understandable before the Battle begins, per Section 10/11 (name + plain-language description present).
- It passes Section 25's "Can it be completed?" simulation gate at the Heuristic-lookahead policy tier (`04` Section 14) with a non-zero completion rate, and ideally a non-trivial completion rate at the Greedy-clear tier.
- Given identical Board/Tray/Relic state, its outcome is deterministic (inherited from `03` Section 13, `04` Section 13, `05`'s own determinism) — no `BattleDefinition` field introduces hidden randomness beyond the already-deterministic `04` RNG stream.
- Its Reward (Section 6, Section 27) is exactly the standard 3-offer/1-choice Relic Selection, plus the fixed Bonus currency value if applicable — no ad hoc reward invented per-Battle.
- Its `difficulty_tier` (Section 14) is a deliberate authoring choice, cross-checked against Section 24's simulated Completion Rate for that Tier's expected band.
- No requirement in the `BattleDefinition` is hidden from the player (Section 23, rule 4).

---

## 29. Final Objective Design Contract

> **Create variety by recombining a small number of robust mechanics instead of creating a new mechanic for every level.**

Concretely, for Block Battles: **10 Objective categories** (Section 5) × **5 MVP-tagged Encounter Modifiers** (Section 8) × **5 Difficulty Tiers** (Section 14), authored through one closed **Battle Authoring Template** (Section 18) into one **`BattleDefinition` data schema** (Section 27), is the entire generative grammar for every current and future handcrafted encounter in the game. Any design idea that cannot be expressed as a combination within this grammar is not a new Battle — it is a proposal to revise this document's primitive set first (Section 2's Master Design Rule), exactly as every prior document in this series (`00`–`06`) has required of itself before allowing its own scope to grow.

---

**End of `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`.**
