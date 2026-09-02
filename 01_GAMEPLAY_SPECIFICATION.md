# 01_GAMEPLAY_SPECIFICATION.md
## Block Battles — Gameplay Specification

**Governing document:** `00_MASTER_GAME_VISION.md`
**Document status:** Implementation-level ruleset

---

## 1. Purpose and Ownership

This document defines the complete playable ruleset for Block Battles: board rules, tray rules, turn definition, combat resolution, enemy behavior, objectives, victory/failure conditions, and run structure. A programmer must be able to implement the base game loop from this document without inferring undocumented behavior.

**Ownership:** This file owns gameplay *rules and state flow*. It does not own exact shape geometry (Shapes doc), RNG weighting (RNG doc), final damage/scoring formulas (Scoring doc), input/gesture handling (Input doc), or individual objective/relic content (Objectives, Relics docs) — see Section 25.

**Precedence:** Per `00_MASTER_GAME_VISION.md` Section 1, this document may not contradict the Master Vision, including the Section 10 decision that **player HP is not part of the Absolute MVP** and the Section 13 forbidden-scope list. Where the Master Vision left an item as an Open Decision, this document resolves it for implementation purposes and marks the resolution as a **DESIGN DECISION**.

---

## 2. Canonical Terminology

This document uses the Master Terminology defined in `00_MASTER_GAME_VISION.md` Section 22 (Run, Battle, Board, Piece, Placement, Clear, Combo, Damage, Relic, Objective, Enemy, Boss, Persistent Reward, Block Battles, Block Quest) without redefinition. Additional implementation-level terms introduced here:

- **Turn:** One resolved player placement (Section 7).
- **Tray:** The set of up to 3 pieces currently available to the player.
- **Telegraph:** A visible preview of an enemy's next action, shown one turn in advance.
- **Board Lock:** A board state in which no piece in the current tray can be legally placed anywhere.
- **Encounter:** A single Battle instance, standard or Objective-modified.

---

## 3. Overall Game State

```text
MAIN MENU
   ↓
RUN SETUP
   ↓
BATTLE ←──────────────┐
   ↓                  │
BATTLE VICTORY         │
   ↓                  │
RELIC SELECTION         │
   ↓                  │
TRANSITION ────────────┘
   ↓ (periodically, per content cadence — see Master Vision Section 14)
BOSS (a Battle variant)
   ↓
RUN DEFEAT (from any Battle) ──→ REWARDS ──→ RESTART ──→ MAIN MENU / RUN SETUP
```

- **Main Menu:** Entry point; no run in progress.
- **Run Setup:** Initializes a new Run: resets relic set, resets encounter index, generates or loads the first Battle.
- **Battle:** Active gameplay per the Battle State Machine (Section 4).
- **Battle Victory:** Reached when a Battle's Primary Objective is satisfied (Section 14).
- **Relic Selection:** Player chooses one relic (Section 17).
- **Transition:** Deterministic setup of the next Battle (next enemy, or Boss if the cadence is reached, or Objective encounter).
- **Boss:** A Battle using the same state machine with an elevated enemy and, optionally, an Objective layer.
- **Run Defeat:** Reached when a Battle ends in failure (Section 15). Ends the Run.
- **Rewards:** Persistent progression is granted (Section 18) regardless of victory or defeat outcome at Run end.
- **Restart:** Returns to Run Setup or Main Menu per player input.

**DESIGN DECISION:** A Run has exactly one life. Any single Battle failure ends the Run (see Section 15). There is no mid-run continue at Absolute MVP, consistent with Master Vision Section 13.

---

## 4. Battle State Machine

| State | Description | Legal Next States |
|---|---|---|
| `PREPARING` | Battle initialized: board reset/pre-filled per Objective (if any), enemy loaded, first tray generated, first enemy telegraph shown. | `ACTIVE` |
| `ACTIVE` | Waiting for player input; no piece currently selected. | `PIECE_SELECTED`, `RUN_DEFEAT` (if Board Lock detected on entry, Section 21) |
| `PIECE_SELECTED` | Player has picked up a tray piece but not yet dragging. | `DRAGGING`, `ACTIVE` (deselect) |
| `DRAGGING` | Piece is following input pointer; legality of current hover position is evaluated continuously (Input doc owns exact gesture handling). | `PLACEMENT_RESOLUTION` (valid drop), `PIECE_SELECTED`/`ACTIVE` (invalid drop / cancel) |
| `PLACEMENT_RESOLUTION` | Piece committed to board cells; tray slot marked consumed; turn counter incremented. | `LINE_CLEAR_RESOLUTION` |
| `LINE_CLEAR_RESOLUTION` | All fully-filled rows/columns identified and cleared simultaneously (Section 5). | `COMBAT_RESOLUTION` |
| `COMBAT_RESOLUTION` | Damage Pipeline (Section 9) executed if at least one line cleared; Enemy HP updated. | `OBJECTIVE_EVALUATION` |
| `OBJECTIVE_EVALUATION` | Active Objective's Primary/Bonus conditions checked deterministically (Section 12). | `ENEMY_DEFEATED` (if Primary Objective met), `ENEMY_ACTION` (otherwise, if Enemy still alive), `RUN_DEFEAT` (if a failure condition is met) |
| `ENEMY_ACTION` | The previously telegraphed enemy action executes (Section 11); a new telegraph is generated for the following turn. | `ACTIVE` (loop) |
| `ENEMY_DEFEATED` | Enemy HP ≤ 0 or Primary Objective satisfied via non-HP condition. | `BATTLE_VICTORY` |
| `BATTLE_VICTORY` | Battle-level rewards computed (Bonus Objective check, Section 13). | (exits state machine to Relic Selection, Section 3) |
| `RUN_DEFEAT` | Board Lock or explicit failure condition reached (Section 15). | (exits state machine to Rewards, Section 3) |

**MASTER DESIGN RULE (inherited):** Tray replacement (Section 6) does not have its own state; it is evaluated as a side effect of `PLACEMENT_RESOLUTION` when the tray is fully consumed.

---

## 5. Board Rules

- The Board is an **8×8 grid** of cells, indexed `(row, col)` with `row, col ∈ [0, 7]`.
- **Cell State Model:** Each Board cell carries two logically distinct attributes: **Occupancy** and a **Cell Modifier**. These are stored compactly (see `03_BOARD_ENGINE_AND_RULES.md` Section 3) but are semantically independent.

  | Occupancy | Modifier | Meaning |
  |---|---|---|
  | Empty | Normal | **Normal Empty.** Default state. Can receive a Piece. |
  | Filled | Normal | **Normal Filled.** Contains a regular placed block. Counts toward Line Clear detection. |
  | Empty | Blocked | **Blocked.** Cannot receive a Piece; does not count toward Line Clear. Duration or condition managed by enemy system. (MVP — used by enemy Block mechanic, Section 11.) |
  | Filled | Frozen | **Frozen.** Occupied, but does not count toward Line Clear detection until unfrozen. (MVP — used by enemy Freeze mechanic, Section 11.) |
  | Filled | Indestructible | **Indestructible.** Occupied, never clearable, never overwritten. (Deferred — reserved for future Objective Board preset or enemy mechanic. Not required at Absolute MVP.) |

  **DESIGN DECISION:** `Blocked` and `Frozen` are confirmed MVP states, required by the documented enemy mechanic categories in Section 11. `Indestructible`, `Hazardous`, and `Special` are reserved as explicitly deferred (post-MVP) states — their data slots may exist in the Cell model, but no MVP logic activates them. The full technical predicate model is owned by `03_BOARD_ENGINE_AND_RULES.md` Section 9.

- **Placement:** A Piece may be placed at a target origin cell if and only if every cell the Piece's shape occupies (relative to that origin) exists on the Board AND has `CanReceiveBlock == true`. At MVP: `Normal Empty` cells satisfy this; `Blocked`, `Filled`, and all other states do not. An illegal placement is rejected and returns control to `PIECE_SELECTED`/`ACTIVE` (Section 4).
- **No rotation:** A Piece is placed exactly in the orientation it is presented in the tray. The Piece pool (owned by the Shapes doc, Section 25) is responsible for including any rotated variants needed for board solvability; this document does not permit runtime rotation as a player action.
- **Clearing:** After a Placement resolves, every row that satisfies `CountsTowardLineClear == true` for all 8 cells and every column that satisfies `CountsTowardLineClear == true` for all 8 cells is identified **simultaneously** (based on the single resulting board state, not sequentially), then all identified rows/columns are cleared (occupancy set to `Empty`, modifier reset to `Normal`) together in one resolution step. `Frozen` cells do NOT count toward Line Clear detection; `Blocked` cells are not Filled and therefore trivially do not contribute.
- **DESIGN DECISION:** A cell that is part of both a clearing row and a clearing column is cleared once; no double-counting of the cell itself (only of line-count for Combo purposes, Section 8).

---

## 6. Tray Rules

- The Tray holds up to **3 Pieces** at any time.
- **Generation:** At Battle start (`PREPARING`), the Tray is fully generated (3 Pieces) per rules owned by the RNG doc (Section 25).
- **Consumption:** When a Piece is placed, its Tray slot is marked consumed and removed from the visible Tray.
- **Replacement timing — DESIGN DECISION:** New Pieces are generated **only when all 3 Tray slots have been consumed**, and all 3 are generated together as a new batch. Pieces are not replaced one at a time as they are used. This keeps piece-availability planning legible (Master Vision Section 5.1, Instant Clarity) and gives the RNG/solvability system (Section 25) a single well-defined batch boundary to validate against.
- **Order independence:** The player may place the 3 Tray Pieces in any order they choose; no slot has positional priority.

---

## 7. Turn Definition

**DESIGN DECISION (resolves Master Vision Section 10 Open Decision):**

> **One Turn = one resolved Placement** (i.e., one full pass through `PLACEMENT_RESOLUTION` → `LINE_CLEAR_RESOLUTION` → `COMBAT_RESOLUTION` → `OBJECTIVE_EVALUATION`).

This is the atomic unit that:
- increments the Battle's turn counter,
- triggers exactly one `ENEMY_ACTION` (Section 10),
- is the unit referenced by any turn-based Objective (Section 12) and Constant (Section 23).

A Turn is **not** equal to a full Tray (3 placements); Turn granularity is per-Piece to keep enemy pacing and telegraphing fine-grained and predictable, satisfying Master Vision Section 5.6 (Fair Difficulty).

---

## 8. Combat Rules

- **Enemy HP:** A non-negative integer, set at Battle start per enemy definition (owned by a future Enemy Content doc).
- **Damage:** Generated only as a result of Line Clears (Section 5); no other player action generates Damage.
- **Damage Events:** Each Turn produces at most one Damage Event, aggregating all lines cleared by that single Placement.
- **Line-clear damage:** A base Damage value is assigned per line cleared (Constants, Section 23).
- **Combo damage:** When a single Placement clears more than one line simultaneously, a Combo multiplier/bonus is applied on top of the summed base Damage (exact formula owned by the Scoring doc; base structure defined in Section 9).
- **Relic-modified damage:** Active Relics may add flat Damage, multiply Damage, or apply conditional Damage bonuses; applied per the pipeline order in Section 9.
- **Enemy defeat:** Occurs the instant Enemy HP reaches 0 or below as a result of a Damage Event, or the instant a non-HP Primary Objective condition is satisfied (Section 12).

---

## 9. Damage Pipeline

```text
Placement
 → Detect Clear                (Section 5: identify all fully-filled rows/columns)
 → Base Damage                 (sum of per-line base damage, Section 23)
 → Relic Modifiers             (flat adds, then multipliers, in Relic-defined order — owned by Relics doc)
 → Combo Modifiers             (bonus applied when lines-cleared-this-turn > 1)
 → Objective Modifiers         (Objective-specific damage rule, if any — owned by Objectives doc)
 → Enemy Defense                (flat and/or percentage reduction, default 0)
 → Final Damage                (clamped to a minimum of 0)
 → Enemy HP Update              (Enemy HP -= Final Damage, floored at 0)
```

**DESIGN DECISION:** This order is fixed at the specification level; exact formulas within each step are owned by the Scoring doc (Section 25) but must not reorder these stages without a revision to this document.

**MASTER DESIGN RULE (inherited):** No stage may produce negative intermediate Damage; Damage is clamped to ≥ 0 only at the Final Damage stage, not at every stage, so that a highly negative Enemy Defense stage cannot be exploited to under-flow earlier stages (implementation must apply defense as a final-stage subtraction/percentage, not an earlier multiplier).

---

## 10. Enemy Turn Rules

**DESIGN DECISION (resolves Master Vision Section 10 Open Decision):**

> The Enemy acts exactly **once per player Turn**, always via a **Telegraph-then-Execute** pattern:

1. On `PREPARING`, the Enemy's first action is chosen and its Telegraph is shown to the player *before* any Placement is made.
2. After a Turn completes `OBJECTIVE_EVALUATION` (and the Enemy is still alive and the Battle has not ended), the state machine enters `ENEMY_ACTION`: the **previously telegraphed** action executes.
3. Immediately after execution, the Enemy's **next** action is chosen and its Telegraph is shown, before returning to `ACTIVE`.

This guarantees every Enemy action was visible to the player for at least one full Turn before it resolved, satisfying Master Vision Section 5.6 and Section 18 (Fairness Philosophy).

---

## 11. Enemy Mechanics

Enemy actions modify the **Board and rules**, not a Player HP resource (none exists at Absolute MVP per Master Vision Section 10). Supported mechanic categories at Absolute MVP:

**Active MVP Mechanic Categories:**
- **Block Cells:** Mark one or more currently-Empty cells as **Blocked** (cannot be targeted by any Placement) for a defined duration (turns) or until a defined condition clears them.
- **Freeze Cells:** Mark one or more currently-Filled cells as **Frozen**, meaning they do not count toward Line Clear detection (Section 5) until unfrozen, effectively stalling a would-be clear.

**Deferred (Post-MVP) Mechanics:**
- **Create Hazards:** Place a special cell type that carries a defined side effect on Placement or Clear (e.g., HAZARDOUS or SPECIAL cells).
- **Alter Rules:** Generalized temporary rule changes (e.g., temporarily disabling Combo bonuses).

**MASTER DESIGN RULE (inherited):** Every Enemy mechanic must be expressible as data. Deferred mechanics (Hazards, Rule Alteration) must remain explicitly deferred and must NOT become MVP requirements. No enemy may invent generalized rule-altering mechanics beyond applying `BLOCKED` or `FROZEN` to cells.

---

## 12. Battle Objectives

An Objective is a deterministic, evaluable condition checked at `OBJECTIVE_EVALUATION` (Section 4) after every Turn. Supported category types at Absolute MVP:

- **Defeat Enemy** — Enemy HP ≤ 0 (default Primary Objective for standard Battles).
- **Clear X Rows** — count of distinct rows cleared (cumulative across the Battle) ≥ X.
- **Clear X Columns** — count of distinct columns cleared (cumulative) ≥ X.
- **Clear X Total Lines** — rows + columns cleared (cumulative) ≥ X.
- **Achieve X Combo** — a single Turn clears ≥ X lines simultaneously.
- **Survive X Turns** — turn counter reaches X while the Enemy is still alive and no failure condition has triggered (used as a Primary win condition for specific Objective encounters, distinct from the default Board Lock failure condition).
- **Defeat Enemy Under Move Limit** — Defeat Enemy condition met while turn counter ≤ X (exceeding X turns without defeating the Enemy triggers Battle failure for that Objective — see Section 15).
- **Clear Lines Using a Particular Behavior** — a Clear event satisfies an additional structural predicate (e.g., "clear a row using a specific Piece type"); exact predicate library owned by the Objectives doc.
- **Complete Condition Before Enemy Reaches Phase** — a tracked condition must be satisfied before a defined Enemy-side counter (e.g., number of Enemy actions taken) reaches a threshold.

**DESIGN DECISION:** Every Objective is evaluated purely from state already tracked by this specification (turn counter, cumulative rows/columns/lines cleared, current Combo size, Enemy HP, Enemy action counter) — no Objective may require state outside this document's data model without an update here first.

---

## 13. Primary Objective vs Bonus Objective

- **Primary Objective:** The condition that, when satisfied, ends the Battle in victory (transition to `ENEMY_DEFEATED`/`BATTLE_VICTORY`, Section 4). Every Battle has exactly one Primary Objective. Default: Defeat Enemy.
- **Bonus Objective:** An optional, additional condition checked once, at `BATTLE_VICTORY`, against data accumulated during the Battle. It does not affect whether the Battle is won or lost.

**Example:**
- Primary: *Defeat Goblin.*
- Bonus: *Defeat Goblin using fewer than 12 placements.*

**Reward implications — DESIGN DECISION:** Meeting the Primary Objective always grants the standard victory reward (progression to Relic Selection, Section 3). Meeting the Bonus Objective additionally grants a supplementary reward (exact reward type — e.g., extra persistent currency, an additional relic offer slot — owned by the Progression/Relics docs). Failing a Bonus Objective never fails the Battle.

---

## 14. Battle Victory Rules

A Battle transitions to `BATTLE_VICTORY` when, during `OBJECTIVE_EVALUATION`, the Battle's Primary Objective (Section 12–13) evaluates true. For standard Battles this is Enemy HP ≤ 0. For Objective-modified encounters, this is whatever single condition was designated Primary for that encounter (Section 8, Block Quest).

---

## 15. Failure Rules (Terminology Hierarchy)

The project uses a strict failure terminology hierarchy:
1. **Board Lock:** A mathematical board condition. Defined as: *No remaining piece in the current tray has a legal placement anywhere on the board.* (Checked at the start of `ACTIVE`).
2. **Battle Failure:** The state a battle reaches when it becomes unwinnable. This occurs when either a Board Lock is detected, or an Objective-Specific Failure Condition is triggered (e.g., exceeding a turn cap for a "Move Limit" objective, checked at `OBJECTIVE_EVALUATION`).
3. **Run Defeat:** The run-level state resulting from a Battle Failure. Because Block Battles is a one-life roguelike, a Battle Failure instantly triggers a Run Defeat.
4. **Game Over:** This is a *player-facing* UI concept only. It should be used in UI copy or player-facing documentation, but internal technical states must use Board Lock, Battle Failure, or Run Defeat.

*Note on Victory:* The equivalent victory hierarchy is `Primary Objective Complete` → `Battle Victory`. (Do not merge these paths).

**Failure State Flow Diagram:**
```text
PLAYER TURN
    ↓
BOARD CHANGES
    ↓
HasAnyValidMove?
    │
    ├── YES → Continue Battle
    │
    └── NO
          ↓
     BOARD LOCK
          ↓
    BATTLE FAILURE
          ↓
      RUN DEFEAT
          ↓
    GAME OVER SCREEN
```

**Victory State Flow Diagram:**
```text
PLAYER ACTION
     ↓
OBJECTIVE EVALUATION
     ↓
PRIMARY OBJECTIVE COMPLETE
     ↓
BATTLE VICTORY
     ↓
RELIC REWARD
```

**MASTER DESIGN RULE (inherited):** Every failure condition must have been knowable to the player in advance (visible Board state, a stated Objective rule, or a Telegraph) — no failure condition may trigger from information the player could not have seen, per Master Vision Section 18.

---

## 16. Run Structure

```text
Battle
 → Reward (Bonus Objective check, Section 13)
 → Relic Choice (Section 17)
 → Next Battle (standard, Boss, or Objective, per content cadence — Master Vision Section 14)
```

This loop repeats until `RUN_DEFEAT` (Section 15) or a future defined Run-completion condition (not specified at Absolute MVP; Master Vision does not require one — Open Decision carried forward).

---

## 17. Relic Choice

- **When:** Immediately after `BATTLE_VICTORY`, before Transition to the next Battle (Section 3).
- **Number of options:** 2–3 randomized Relics offered per choice (Master Vision Section 5.3, 5.7; exact pool/weighting owned by the Relics/RNG docs).
- **Choosing one:** The player selects exactly one offered Relic; it is added to the Run's active Relic set immediately and applies from the next Damage Pipeline evaluation onward (Section 9).
- **Rejecting / all-options behavior — DESIGN DECISION:** At Absolute MVP, Relic Choice is **mandatory**; there is no skip or reroll. This keeps the choice screen simple per Master Vision Section 5.8. (OPEN DECISION, carried from Master Vision: whether reroll/skip is added in MVP+.)
- **Persistence during the Run:** All chosen Relics remain active for the remainder of the Run and are cleared only at Run Setup (new Run). Relics do not persist across Runs (that is the role of Persistent Progression, Section 18).

---

## 18. Persistent Progression

**DESIGN DECISION:** At Absolute MVP, exactly one persistent mechanic exists, matching Master Vision Section 12's two-currency ceiling (one run-time implicit "currency" being Enemy Damage/Relics, which do not persist, plus one true persistent meta-currency):

- At Run end (`RUN_DEFEAT` or a future Run-completion), the player is granted an amount of a single **persistent meta-currency**, computed from Run performance (e.g., Battles won, Bosses defeated — exact formula owned by a future Progression doc).
- This currency is stored outside any Run and is not reset by Run Setup.
- No unlock/spend systems are specified in this document; they are explicitly out of scope here per Master Vision Section 13 (MVP+ / Post-launch).

---

## 19. Pause / Resume

- Pausing is permitted only while in the `ACTIVE` or `PIECE_SELECTED` state (Section 4); pausing is not permitted mid-resolution (`PLACEMENT_RESOLUTION` through `ENEMY_ACTION`) — the game completes the current Turn's resolution chain first, then honors a queued pause.
- While paused, no state changes: no Enemy Telegraph countdown, no Objective evaluation, no input is accepted on the Board.
- Resuming returns exactly to the state the game was paused in, with no time-based penalty (Block Battles has no real-time combat clock at Absolute MVP, per Master Vision Section 10).

---

## 20. Backgrounding / App Lifecycle

- On transition to background, the current Run state is **autosaved** in full: RNG continuation state (`rngStateCalls`), Board state (including Blocked/Frozen cells), Tray contents, Enemy HP and pending Telegraph, active Relic set, turn counter, cumulative Objective-tracking data, and persistent meta-currency balance.
- On return to foreground, state is restored exactly; if the app was backgrounded mid-resolution (between `PLACEMENT_RESOLUTION` and `ENEMY_ACTION`), the resolution chain completes automatically on resume before input is re-enabled, so the player never observes a partially-resolved Turn.
- If the process was terminated while backgrounded, the last autosave is loaded on next launch; if no autosave exists, the app opens to `MAIN MENU`.

---

## 21. Edge Cases

| Edge Case | Resolution |
|---|---|
| No legal Placement exists for any of the 3 current Tray pieces | Board Lock → Battle Failure → `RUN_DEFEAT` (Section 15) |
| A single Placement completes multiple rows and columns simultaneously | All identified lines clear together in one `LINE_CLEAR_RESOLUTION` step; all count toward Combo (Section 8) and cumulative line-based Objectives (Section 12) |
| A Piece cannot fit anywhere on the current Board ("dead" Piece) but other Tray Pieces still have legal placements | Not a Board Lock (Section 15 requires *no* Tray piece to have a legal placement); play continues normally |
| A Placement exactly fills the entire remaining Board | All resulting full rows/columns clear per Section 5; Board returns to fully Empty if all rows/columns were completed |
| Enemy Telegraph targets a cell that is cleared before the Telegraph executes | The Telegraphed action re-validates its target set immediately before executing in `ENEMY_ACTION`; if the original target is no longer valid (e.g., now cleared/Empty), the action is redirected per a defined fallback rule (owned by the Enemy Content doc) rather than silently failing |
| Relic effect and Objective rule both modify the same Damage Pipeline stage | Both apply, in the fixed stage order of Section 9; within a stage, Relic Modifiers resolve before Objective Modifiers |
| Enemy is defeated (HP ≤ 0) in the same Turn a non-HP Primary Objective is also satisfied | Battle proceeds to `ENEMY_DEFEATED`/`BATTLE_VICTORY`; only one victory transition occurs regardless of how many conditions were simultaneously true |
| Turn-limit Objective ("Under Move Limit") reaches its cap on the same Turn the Enemy is defeated | Victory takes precedence if Enemy HP ≤ 0 is reached at or before the limit is exceeded within the same Turn's resolution; the failure check for that Objective only triggers if the limit is exceeded *without* victory in that same Turn |
| App is backgrounded/terminated mid-`ENEMY_ACTION` | On resume, the in-progress Enemy action re-executes from its start (idempotent by design — Enemy actions must be safe to re-run against the autosaved pre-action state) |
| Combo achieved across two separate Turns rather than one Placement | Does not count toward "Achieve X Combo" (Section 12), which is strictly single-Placement, simultaneous-line-count based |

---

## 22. Battle Examples

### Example A — Standard Battle (Normal)
1. `PREPARING`: Enemy "Slime" loaded with 10 HP. Tray generated: [Piece I (4 cells, vertical line), Piece L, Piece O (2×2 square)]. First Telegraph shown: "Slime will Block 1 random Empty cell next turn."
2. Turn 1: Player places Piece O in the bottom-left 2×2 area. No line completed. `ENEMY_ACTION`: Slime blocks one cell. New Telegraph: "Slime will Block 1 random Empty cell next turn."
3. Turn 2: Player places Piece I to complete one full column. Damage Pipeline: Base Damage (1 line) → no Relics yet → no Combo (1 line) → no Objective modifier → Enemy Defense 0 → Final Damage applied. Slime HP reduces accordingly (Constants, Section 23).
4. Turn 3: Player places Piece L, completing one row. Similar Damage Pipeline resolution. Slime HP reaches 0 → `ENEMY_DEFEATED` → `BATTLE_VICTORY`. No Bonus Objective was defined for this Battle, so only the standard reward is granted. Proceeds to Relic Selection.

### Example B — Difficult Battle
1. `PREPARING`: Enemy "Warden" loaded with high HP. Telegraph: "Warden will Freeze 2 Filled cells next turn." (Requires the player to already have Filled cells; if none exist, the Telegraphed action's fallback rule applies per Section 21.)
2. Several Turns pass where the player must balance clearing lines for Damage against the Warden repeatedly Freezing cells the player is relying on for an imminent clear, forcing the player to route around Frozen cells (Master Vision Section 5.2, Tactical Placement).
3. The player achieves a 2-line simultaneous Combo on a late Turn (Section 8, Section 21), applying the Combo Modifier stage in the Damage Pipeline, finally reducing Warden HP to 0.

### Example C — Objective-Specific Encounter
1. `PREPARING`: A Block Quest encounter with Primary Objective **"Clear 5 Total Lines"** and Bonus Objective **"Achieve a 3-line Combo at least once."** Enemy HP is present but not the win condition for this encounter.
2. Player accumulates cleared lines across multiple Turns; `OBJECTIVE_EVALUATION` checks the cumulative-lines counter after every Turn.
3. On Turn 6, cumulative lines reach 5 → Primary Objective satisfied → `ENEMY_DEFEATED`/`BATTLE_VICTORY` regardless of remaining Enemy HP (Section 14).
4. At `BATTLE_VICTORY`, the Bonus Objective is checked against tracked Combo history: if a 3-line Combo occurred on any single Turn during the encounter, the supplementary reward is also granted (Section 13).

---

## 23. Constants

**ASSUMPTION:** All numeric values below are Absolute MVP placeholders pending balance tuning in the Scoring doc; they exist so the base loop is implementable and testable, not as final tuned values.

| Constant | Value | Notes |
|---|---|---|
| Board size | 8 × 8 | Fixed, Section 5 |
| Tray size | 3 | Fixed, Section 6 |
| Base Damage per line cleared | 10 (placeholder) | Section 9 |
| Combo bonus (2 lines, same Turn) | +50% of summed base Damage (placeholder) | Section 8–9 |
| Combo bonus (3+ lines, same Turn) | +100% of summed base Damage (placeholder) | Section 8–9 |
| Default Enemy Defense | 0 | Overridable per enemy |
| Turns before first Enemy action | 0 (Enemy's first Telegraph is visible from Battle start; first action executes after Turn 1) | Section 10 |

---

## 24. Acceptance Criteria

- A Piece can be placed only onto a set of in-bounds cells where every target cell satisfies `CanReceiveBlock == true` (i.e., `Normal Empty`); any `Blocked`, `Filled`, or otherwise non-placeable cell in the target area causes the drop to be rejected.
- Every row and every column where all 8 cells satisfy `CountsTowardLineClear == true` is cleared in the same resolution step that produced them, never partially or across multiple frames of game logic. `Frozen` cells break line-clear eligibility for their row/column for the duration they are frozen.
- Exactly one Damage Event is produced per Turn, following the fixed Section 9 pipeline order, never skipping or reordering stages.
- The Enemy never executes an action that was not shown as a Telegraph at least one full Turn earlier.
- A Battle transitions to `RUN_DEFEAT` if and only if a documented Section 15 failure condition is met; it never ends in failure for an undocumented reason.
- A Battle transitions to `BATTLE_VICTORY` if and only if its designated Primary Objective evaluates true.
- Relic Choice always presents 2–3 options and always requires exactly one selection before Transition proceeds.
- An autosave restored after backgrounding reproduces an identical playable state to the moment of backgrounding, with no partially-resolved Turn ever exposed to the player.

---

## 25. Cross-Document Dependency Rules

- **Shapes doc:** Owns the full Piece pool, including any pre-rotated variants required for solvability given the no-rotation rule (Section 5).
- **Board Engine doc:** Owns low-level grid data structure and rendering concerns beneath the rules defined in Section 5.
- **RNG doc:** Owns Tray generation weighting (Section 6), Relic offer weighting (Section 17), and solvability safeguards referenced by Master Vision Section 5.5/18.
- **Scoring doc:** Owns exact numeric formulas for every stage of the Damage Pipeline (Section 9), superseding the placeholder Constants in Section 23.
- **Input doc:** Owns gesture/drag handling detail beneath the `PIECE_SELECTED`/`DRAGGING` states (Section 4).
- **Objective System doc (Block Quest):** Owns the full library of Objective predicates and Enemy hazard/rule-alteration behaviors referenced generically in Sections 11–13.

This document must not be edited to embed content owned by the above; it references them structurally only.

---

## 26. Final Gameplay Contract

Given a Battle in state `PREPARING`, the deterministic end-to-end pipeline is:

```text
PREPARING
 → ACTIVE (Tray ready, first Enemy Telegraph visible)
 → [ PIECE_SELECTED → DRAGGING → PLACEMENT_RESOLUTION
     → LINE_CLEAR_RESOLUTION → COMBAT_RESOLUTION
     → OBJECTIVE_EVALUATION ] = one Turn (Section 7)
 → ENEMY_ACTION (previously telegraphed action executes; next Telegraph shown)
 → ACTIVE (loop)
 ... repeats until either:
 → ENEMY_DEFEATED → BATTLE_VICTORY → Relic Selection → Transition → next Battle (Section 16)
   or
 → RUN_DEFEAT → Rewards (Section 18) → Restart (Section 3)
```

Every transition in this pipeline is deterministic given the current Board state, Tray, active Relics, Enemy state, and Objective definition — no hidden or undocumented state may influence any transition.
