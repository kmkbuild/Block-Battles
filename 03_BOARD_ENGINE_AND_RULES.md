# 03_BOARD_ENGINE_AND_RULES.md
## Block Battles — Board Engine and Rules

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`
**Document status:** Implementation-level system design

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23. It implements the Board Rules of `01_GAMEPLAY_SPECIFICATION.md` Section 5 and the placement geometry contract of `02_SHAPE_LIBRARY.md` Section 2, at the level of a deterministic, testable logical system. It does not redefine any rule those documents own; where this document adds detail, it narrows rather than reverses.

---

## 1. Board Model

The Board is the single logical grid the Gameplay Specification's `PLACEMENT_RESOLUTION` and `LINE_CLEAR_RESOLUTION` states operate on (`01_GAMEPLAY_SPECIFICATION.md` Section 4).

**DESIGN DECISION:** The Board is modeled as a fixed-size 2D array of Cells, `board[row][col]`, `8×8` (Gameplay Spec Section 23), never resized at runtime. Board size is a Constant owned by the Gameplay Spec; this document consumes it, it does not define it.

- The Board is the single source of truth for occupancy. No other system (Combat, Objectives, Relics) maintains a parallel occupancy record — they query the Board Engine (Section 11).
- The Board Engine has no knowledge of Enemies, Damage, Relics, or Objectives. It exposes state and events (Section 12); everything above the Board reacts to those events. This satisfies Master Vision Section 20 (Modular Systems).
- The Board owns exactly one turn's worth of mutation at a time: one Placement, its resulting Clears, in that fixed order. It does not queue or batch across Turns (Gameplay Spec Section 7).

---

## 2. Coordinate System

Inherited verbatim from `02_SHAPE_LIBRARY.md` Section 2, restated here for Board Engine authority:

- `row` increases downward, `col` increases rightward. `(row, col) ∈ [0,7] × [0,7]`.
- A Shape is placed by mapping its normalized `local_coordinates` (Shape Library Section 2–3) onto a target origin `(boardRow, boardCol)`: local cell `(r, c)` → Board cell `(boardRow + r, boardCol + c)`.
- **In-bounds check:** every mapped cell must satisfy `0 ≤ boardRow + r ≤ 7` and `0 ≤ boardCol + c ≤ 7`. This is evaluated for every local cell independently — a Shape's bounding box being in-bounds is necessary but not sufficient, since irregular Shapes (Shape Library Section 6, IRREGULAR) do not fill their bounding box.

---

## 3. Cell State Model

Each Board cell has a state that combines two distinct logical properties: **Occupancy** and **Cell Modifier**. The Board Engine is the single source of truth for both properties.

### 3.1 MVP States (Occupancy + Modifier)

**Occupancy:**
- `EMPTY`: No block occupies this cell.
- `FILLED`: A player block occupies this cell, tagged with the owning Placement's identity.

**Cell Modifier / State:**
- `NORMAL`: The cell behaves according to standard rules.
- `BLOCKED`: The cell is unavailable for placement, remains empty, and cannot receive a block. It is still part of the grid.
- `FROZEN`: The cell contains a block but is temporarily protected from normal clearing / line-clear destruction. It remains occupied, and placement into it is impossible.

**Valid MVP Combinations:**
- **EMPTY + NORMAL:** Available for normal placement.
- **FILLED + NORMAL:** Contains a placed block. Participates in line clearing.
- **EMPTY + BLOCKED:** Unavailable for placement.
- **FILLED + FROZEN:** Contains a placed block but is protected from line-clear destruction.

*Note: Enemy mechanics may request board-state changes (applying BLOCKED or FROZEN) through the Board Engine, but enemy logic must not directly mutate raw board storage.*

### 3.2 Extensible Deferred States / Mechanics

The following mechanics are explicitly **deferred** (post-MVP) and must not be treated as active MVP requirements:

- `HAZARDOUS`: Generalized hazard systems or hazard creation.
- `INDESTRUCTIBLE`: Occupied, never clearable, never overwritten by Placement.
- `SPECIAL`: Any generalized rule-altering cell system.
- General hazard/special-cell framework.

**MASTER DESIGN RULE (inherited from Gameplay Spec Section 11):** Do not implement a generalized hazard/special-cell framework during MVP. Deferred states must not become MVP requirements. No enemy may invent generalized rule-altering mechanics beyond applying `BLOCKED` or `FROZEN` to cells.

---

## 4. Placement Validation

**Inputs:** a Shape's `local_coordinates` (Shape Library Section 3), a target origin `(boardRow, boardCol)`, and the current Board state.

**Algorithm — `validatePlacement(shape, origin, board)`:**

1. For each `(r, c)` in `shape.local_coordinates`:
   a. Compute `target = (origin.row + r, origin.col + c)`.
   b. **Bounds check:** if `target.row ∉ [0,7]` or `target.col ∉ [0,7]`, return `INVALID: OUT_OF_BOUNDS` immediately — do not continue checking remaining cells.
   c. **Placement check:** if `!CanReceiveBlock(board[target.row][target.col])`, return `INVALID: CELL_NOT_PLACEABLE` immediately.
2. If every local cell passed both checks, return `VALID`.

*(Note: `CanReceiveBlock(cell)` evaluates to true if and only if `cell.occupancy == EMPTY` AND `cell.modifier == NORMAL`.)*

**DESIGN DECISION:** Validation short-circuits on the first failing cell (matches Shape Library Section 11, Test T6: "the validator must reject on the first occupied target cell found rather than only checking the origin cell"). Order of cell iteration does not affect the VALID/INVALID result, only which specific failure is reported first when multiple cells fail — this is intentionally not specified as deterministic across cells, since only the boolean outcome is contractually meaningful (Section 20 clarifies what must remain reproducible).

3. Validation is **read-only**: it must never mutate `board` under any code path, including early returns.

---

## 5. Atomic Placement

**DESIGN DECISION:** `commitPlacement(shape, origin, board)` is atomic — it either applies every one of a Shape's cells or none of them.

**Algorithm:**

1. Call `validatePlacement(shape, origin, board)`. If the result is not `VALID`, return the failure and perform zero mutations. The Gameplay Spec's `DRAGGING → PLACEMENT_RESOLUTION` transition (Section 4) only occurs after this call returns `VALID`; an `INVALID` result routes back to `PIECE_SELECTED`/`ACTIVE` and never reaches this function's mutation step.
2. Only after step 1 succeeds: for each `(r, c)` in `shape.local_coordinates`:
   - Set `board[origin.row + r][origin.col + c].occupancy = FILLED`
   - Set `board[origin.row + r][origin.col + c].modifier = NORMAL`
   - Tag it with the current Placement's identity (a monotonically increasing `placementId`, Section 13).
3. No intermediate state produced during step 2 is ever observed outside this function — the mutation loop is not interruptible by any other Board Engine call (Section 13, Determinism).

This guarantees the Gameplay Spec Section 24 acceptance criterion: "any other drop target is rejected" with the Board left byte-for-byte unchanged on rejection.

---

## 6. Line Detection

**Definition:** A row `r` is a **candidate cleared row** if `CountsTowardLineClear(board[r][c])` evaluates to true for all `c ∈ [0,7]`. A column `c` is a **candidate cleared column** if `CountsTowardLineClear(board[r][c])` evaluates to true for all `r ∈ [0,7]`.

*(Note: `CountsTowardLineClear(cell)` evaluates to true if and only if `cell.occupancy == FILLED` AND `cell.modifier == NORMAL`.)*

**DESIGN DECISION:** A cell being occupied does not automatically mean it contributes to a line clear. Only `FILLED + NORMAL` counts toward line completeness for MVP. A `FILLED + FROZEN` cell will read as "not counted" without touching this function's control flow, only its per-state predicate (Section 9).

**Algorithm — `detectLines(board)`:**

1. Compute `filledRows = { r | ∀c, CountsTowardLineClear(board[r][c]) }` by scanning all 8 rows.
2. Compute `filledCols = { c | ∀r, CountsTowardLineClear(board[r][c]) }` by scanning all 8 columns.
3. Both computations run against the **same, single resulting board state** produced by `commitPlacement` (Section 5) — never against an intermediate state, and never sequentially re-derived after a partial clear. This satisfies `01_GAMEPLAY_SPECIFICATION.md` Section 5's "identified simultaneously" rule.
4. Return `(filledRows, filledCols)` as the immutable clear-set for this Turn.

---

## 7. Simultaneous Clears

Given `(filledRows, filledCols)` from Section 6, every combination is covered by the single clear-set model — no special-casing is required per combination, but they are enumerated here for test-fixture completeness (Section 14):

| Combination | Example | Handling |
|---|---|---|
| Zero lines | No row/col fully filled | `filledRows = filledCols = ∅`; `LINE_CLEAR_RESOLUTION` produces no `LinesCleared` event (Section 12); Combat Resolution still runs but the Damage Pipeline receives 0 lines (Gameplay Spec Section 9 begins at "Detect Clear"). |
| Single row only | 1 row, 0 columns | Standard single-line clear. |
| Single column only | 0 rows, 1 column | Standard single-line clear. |
| Multiple rows, no columns | e.g. 2 rows | Both rows clear together in one mutation pass (Section 8). |
| Multiple columns, no rows | e.g. 2 columns | Symmetric to above. |
| Rows and columns together | 1 row + 1 column | Both clear together; their intersection cell is cleared once (Section 8). |
| Maximum case | Up to 8 rows and 8 columns (a Placement that completes the entire remaining Board) | Every row and column clears in the same pass; Board returns to fully `EMPTY` (matches Shape Library/Gameplay Spec Section 21 edge case). |

**DESIGN DECISION:** The Board Engine emits the full `filledRows`/`filledCols` sets as one event (Section 12); it is the Combat layer's responsibility to derive Combo count from `|filledRows| + |filledCols|` (Gameplay Spec Section 8) — the Board Engine does not itself compute or expose a "Combo" concept, staying within its owned domain (Master Vision Section 20).

---

## 8. Board Mutation

**DESIGN DECISION:** Mutation order for a single Turn is fixed and must not be reordered by any caller:

```text
1. commitPlacement   — apply the Shape's cells (Section 5), FILLED + NORMAL
2. detectLines        — compute filledRows, filledCols against the post-Placement board (Section 6)
3. clearLines         — for every cell in any filledRow or filledCol, set occupancy = EMPTY and modifier = NORMAL, exactly once per cell
4. emit BoardChanged, PlacementCommitted, and (if applicable) LinesCleared / CellsCleared (Section 12)
```

**`clearLines` algorithm:**

1. Build `cellsToClear = { (r,c) | r ∈ filledRows } ∪ { (r,c) | c ∈ filledCols, ∀r ∈ [0,7] }` — a set union, so a cell at the intersection of a clearing row and clearing column appears once.
2. For each `(r,c)` in `cellsToClear`:
   - Set `board[r][c].occupancy = EMPTY`
   - Set `board[r][c].modifier = NORMAL`
   Since this is a set, no cell is written twice (Gameplay Spec Section 5's "cleared once" rule is structurally guaranteed, not just asserted).
3. This is the only step in the Turn that transitions cells from `FILLED` back to `EMPTY`; `commitPlacement` (Section 5) only ever writes `EMPTY → FILLED`.

---

## 9. Extension Seams

The Board Engine implements `FROZEN` and `BLOCKED` for MVP (Gameplay Spec Section 11 assigns their triggering behaviors to Enemy Mechanics). It exposes exactly three extension points so deferred mechanics (like Hazards) can be added without touching Sections 4-8's core algorithms:

1. **Placement predicate (used by Section 4):** `CanReceiveBlock(cell)` — MVP implementation is `cell.occupancy == EMPTY && cell.modifier == NORMAL`.
2. **Line-completeness predicate (used by Section 6):** `CountsTowardLineClear(cell)` — MVP implementation is `cell.occupancy == FILLED && cell.modifier == NORMAL`. A `FROZEN` state overrides this predicate to return `false` while `cell.occupancy` still reports `FILLED` for rendering/query purposes (Section 11), stalling a clear without a second occupancy flag.

*Note: The Occupancy and Modifier attributes are stored separately for compactness and extensibility, but MVP gameplay only authorizes the documented combinations. `EMPTY + FROZEN` and `FILLED + BLOCKED`, for example, are not valid active MVP states.*
3. **Post-clear hook (used by Section 8, step 2):** an event fired per cleared cell (`CellsCleared`, Section 12) that a future Hazard system can subscribe to for side effects, without the Board Engine itself knowing what those side effects are.

**DESIGN DECISION — do not overbuild:** No duration counters, no unfreeze/unblock scheduling, and no hazard trigger table are implemented in this document. Only the three predicate/hook seams above exist at MVP; this satisfies the instruction "do not overbuild them" while keeping Master Vision Section 15's expansion path open at zero rework cost to Sections 4–8.

---

## 10. Game-Over Detection

**Definition (inherits Gameplay Spec Section 15, rule 1 — Board Lock):** the Board Engine is queried at the start of `ACTIVE` (Gameplay Spec Section 4) to determine whether any Tray Piece has any legal Placement anywhere on the Board.

**Algorithm — `hasAnyLegalPlacement(tray, board)`:**

```text
for each shape in tray (up to 3 pieces):
    for boardRow in 0..7:
        for boardCol in 0..7:
            if validatePlacement(shape, (boardRow, boardCol), board) == VALID:
                return true   // early exit — a single legal placement is sufficient
return false   // no shape, at no origin, has any legal placement
```

- This is an exhaustive brute-force search: for each of up to 3 Shapes, all 64 possible origins are checked (a Shape's own bounding box makes many origins trivially out-of-bounds via the Section 4 bounds check, but the search does not pre-filter — it relies on `validatePlacement`'s own short-circuit for efficiency, Section 16).
- **Board Lock** is declared if and only if this function returns `false` for the current Tray against the current Board. This is the sole trigger for the Board-Engine-owned side of Gameplay Spec Section 15, rule 1; Objective-specific failure conditions (Section 15, rule 2) are not evaluated by the Board Engine.
- Per Shape Library Section 11, Test T8, this check must consider all 3 Tray pieces, not just one — a single "dead" Piece with no legal placement is not itself a Board Lock while another Tray Piece still has one (Gameplay Spec Section 21 edge case).

---

## 11. Board Queries

Read-only functions exposed to Combat, Objectives, Relics, and rendering layers. None of these mutate Board state.

| Query | Signature (conceptual) | Returns |
|---|---|---|
| Cell contents | `getCell(row, col)` | A Cell object containing its Occupancy and Modifier. |
| Full board snapshot | `getBoardSnapshot()` | An immutable copy of the full 8×8 state, for autosave (Gameplay Spec Section 20) and rendering. |
| Placement legality | `validatePlacement(shape, origin)` | `VALID` / `INVALID` + reason (Section 4). |
| Any legal placement for a Tray | `hasAnyLegalPlacement(tray)` | Boolean (Section 10). |
| Current filled-cell count | `getFilledCellCount()` | Integer, for debugging/telemetry (Section 17). |
| Row/column fill state | `getRowFillCount(row)` / `getColFillCount(col)` | Integer 0–8, for UI "almost full" affordances (Master Vision Section 5.4). |
| Cumulative lines cleared this Battle | Not owned here — Objectives doc tracks cumulative counters (Gameplay Spec Section 12) by subscribing to `LinesCleared` events (Section 12 below); the Board Engine itself is stateless across Turns beyond the live grid. |

---

## 12. Event Outputs

The Board Engine is the sole emitter of these logical events, consumed by Combat Resolution, Objective Evaluation, and rendering (Gameplay Spec Section 4). Events are emitted in the fixed order below for a single Turn, matching Section 8's mutation order:

| Event | Emitted when | Payload (conceptual) |
|---|---|---|
| `PlacementCommitted` | Immediately after Section 5's mutation completes. | `placementId`, `shapeId`, `origin`, `cellsFilled: [(r,c)...]` |
| `LinesCleared` | If `filledRows ∪ filledCols ≠ ∅` after Section 6. | `rowsCleared: [r...]`, `colsCleared: [c...]`, `lineCount: |rows|+|cols|` |
| `CellsCleared` | Same trigger as `LinesCleared`, one event carrying the deduplicated cell set from Section 8. | `cells: [(r,c)...]` (the exact `cellsToClear` set) |
| `BoardChanged` | Once per Turn, after all mutation for that Turn is complete (fires regardless of whether any line cleared). | `snapshot` (post-Turn board state) |
| `BoardLockDetected` | When `hasAnyLegalPlacement` (Section 10) returns `false` at the Section 10 query point. | none (trigger only) |

**DESIGN DECISION:** `LinesCleared` and `CellsCleared` are only emitted when at least one line cleared — a zero-line Turn emits `PlacementCommitted` and `BoardChanged` only, matching Gameplay Spec Section 8 ("Damage Events... at most one... aggregating all lines cleared") which requires Combat to distinguish a real clear from a no-op cleanly.

---

## 13. Determinism

**DESIGN DECISION:** Given an identical starting Board state, an identical Shape, and an identical target origin, `commitPlacement` and `detectLines` produce byte-identical resulting Board state and identical event payloads, every time, with no hidden randomness or wall-clock dependency anywhere in Sections 4–9.

- The Board Engine consumes no RNG. All randomness (Tray generation, Relic offers) is owned upstream (Gameplay Spec Section 25) and enters the Board Engine only as a fully-resolved `shape` + `origin` pair.
- `placementId` (Section 5) is a monotonically increasing counter seeded from the Battle's turn counter (Gameplay Spec Section 7) — reproducible from the same Battle log, not from system time.
- A single Turn's Board Engine calls (`validatePlacement` → `commitPlacement` → `detectLines` → `clearLines`) execute as one uninterruptible logical sequence; no other Board Engine call may interleave mid-Turn. This is what makes the Gameplay Spec Section 20 autosave/resume guarantee ("no partially-resolved Turn ever exposed") possible at the Board layer.
- Determinism holds only for Sections 4–8 (the MVP core). The Extensible states in Section 3.2 must preserve this property when implemented — any future randomness (e.g., which cell a "Block 1 random Empty cell" Enemy action targets) is resolved by the caller *before* it reaches the Board Engine as a concrete cell coordinate, never inside it.

---

## 14. Test Fixtures

Fixed boards for deterministic unit testing. For visual simplicity, `.` denotes `EMPTY + NORMAL`, and `#` denotes `FILLED + NORMAL`.

**Fixture F1 — Empty Board:** all 64 cells `.`. Baseline for placement/legality tests (mirrors Shape Library Section 11, Tests T1–T3, T5, T7).

**Fixture F2 — Single Pre-filled Cell (for occupancy rejection):**
```
row 2: . . # . . . . .
```
All other cells `.`. Used for a test equivalent to Shape Library T4 (placement overlapping one occupied cell).

**Fixture F3 — One Full Row Pending (row 6 all `.` except needs 1 cell):**
```
row 6: # # # # # # # .
```
Placing a 1-cell Shape at `(6,7)` must trigger `filledRows = {6}`, `LinesCleared` with `lineCount = 1`.

**Fixture F4 — Simultaneous Row + Column:**
```
row 0: # # # # # # # .
row 1..7, col 7: # (all filled except row 0)
```
Placing a 1-cell Shape at `(0,7)` must trigger `filledRows = {0}`, `filledCols = {7}`, with cell `(0,7)` cleared exactly once (Section 8 test).

**Fixture F5 — Full-Board Clear:** every cell `#` except one `EMPTY` cell at `(4,4)`; placing a 1-cell Shape there must clear all 8 rows and all 8 columns, returning the Board to Fixture F1.

**Fixture F6 — Board Lock:** a Board pattern with scattered single-cell gaps too small/disconnected to fit any of a given 3-piece Tray (e.g., Shape Library `SHP_012` Tetromino Square, `SHP_010` Tetromino Line H, `SHP_032` Square 3×3, against a Board with only isolated single-cell and 1×2 gaps). `hasAnyLegalPlacement` must return `false`.

**Fixture F7 — Near-Lock, One Legal Cell:** identical to F6 but with one gap exactly matching one Tray Shape's footprint. `hasAnyLegalPlacement` must return `true`, and must identify that specific placement.

---

## 15. Edge Cases

| Edge Case | Resolution |
|---|---|
| Shape's bounding box is in-bounds but an individual local cell (irregular Shape) is out-of-bounds | Rejected — Section 2 requires per-local-cell bounds checking, not bounding-box-only (relevant to IRREGULAR-tagged Shapes, Shape Library Section 6). |
| Placement would complete a line, but validation is checked before commit | Irrelevant to validation — `validatePlacement` only checks current board state (via `CanReceiveBlock`), never look-ahead to post-Placement state; line completion is only ever evaluated after commit (Section 6). |
| A Shape has zero legal origins but the Board is not locked (other Tray Shapes have legal placements) | Not a Board Lock (Section 10; matches Gameplay Spec Section 21). |
| Two different rows/columns share a clearing cell | Cleared exactly once via set union (Section 8). |
| `detectLines` called on a Board with zero Placements yet made (Battle `PREPARING`) | Returns `(∅, ∅)` — an empty Board can never contain a filled line by construction. |
| A `FROZEN` cell exists mid-line at clear time | The `CountsTowardLineClear` predicate evaluates to false. The line does not clear. |
| `validatePlacement` called with an origin fully outside `[0,7]×[0,7]` even for the shape's single origin cell | First local cell fails the bounds check immediately; returns `INVALID: OUT_OF_BOUNDS` without evaluating remaining cells. |
| Rapid repeated `hasAnyLegalPlacement` queries mid-`ACTIVE` (e.g., re-triggered by UI) | Idempotent and side-effect-free — it is a pure read query (Section 11); may be called any number of times with identical results given an unchanged Board/Tray. |

---

## 16. Performance

- **Placement validation:** O(k) per call, where k = Shape cell count (max 9, Shape Library Section 4, Square 3×3). Bounded, negligible cost.
- **Line detection:** O(64) per call — fixed 8×8 scan, independent of how many cells are filled.
- **Line clearing:** O(≤64) — bounded by total Board size regardless of how many lines cleared simultaneously.
- **Game-over search (Section 10):** worst case O(3 shapes × 64 origins × k cells) ≈ O(3 × 64 × 9) = O(1,728) primitive cell checks — a small, fixed upper bound suitable for evaluation on every `ACTIVE` entry without caching, on mobile hardware (Master Vision Section 19).
- **DESIGN DECISION:** No spatial-indexing, bitboard, or incremental-recompute optimization is required at MVP scale given the fixed 8×8 grid and ≤1,728-check worst case; such optimization is an explicit future item only if profiling later shows otherwise (Master Vision Section 5.8, feasibility for a solo developer).

---

## 17. Debugging Tools

Conceptual, engine-agnostic debug affordances the Board Engine should expose (implementation left to the technical/engine-specific doc, per Master Vision Section 20's implementation-neutral stance):

- **ASCII/text Board dump:** render `getBoardSnapshot()` (Section 11) as the `.`/`#` grid format used in Section 14, for log output and automated test diffs.
- **Placement trace log:** a per-Turn record of `(placementId, shapeId, origin, filledRows, filledCols, cellsCleared)` — directly reconstructable from the Section 12 event stream, requiring no separate logging system.
- **Deterministic replay:** because Section 13 guarantees determinism, a sequence of `(shapeId, origin)` pairs replayed from Turn 1 against an empty Board must reproduce an identical Board state and event log every time — the primary debugging tool for reported bugs.
- **Forced-state injection (test-only):** a test-only setter to load an arbitrary Fixture (Section 14) directly into `board`, bypassing normal Placement, for isolated Line Detection / Game-Over testing.

---

## 18. Property-Based Testing

Properties that must hold for **any** randomly generated valid Shape/origin/Board combination, not just the fixed fixtures in Section 14:

1. **Atomicity:** for any Shape/origin pair, if `validatePlacement` returns `INVALID`, then `board` after calling `commitPlacement` is byte-identical to `board` before the call.
2. **Conservation:** for any successful `commitPlacement` not followed by a clear, `filledCellCountAfter == filledCellCountBefore + shape.block_count`.
3. **Clear conservation:** for any Turn that clears lines, `filledCellCountAfter == filledCellCountBefore + shape.block_count - |cellsToClear|`, where `cellsToClear` is the deduplicated set from Section 8.
4. **No double-clear:** for any `(filledRows, filledCols)` pair, every cell in `cellsToClear` appears in the set exactly once (a structural guarantee of set union, verifiable by asserting `len(cellsToClear as list with duplicates removed) == len(cellsToClear)`).
5. **Idempotent queries:** calling any Section 11 query twice in a row with no intervening mutation returns identical results both times.
6. **Bounds soundness:** for any Shape and any origin such that every local cell maps in-bounds, `validatePlacement` never returns `OUT_OF_BOUNDS`; for any mapping with at least one out-of-bounds local cell, it always does.
7. **Lock soundness:** `hasAnyLegalPlacement` returns `true` if and only if there exists at least one `(shape, origin)` pair in the Tray × 64-origin space for which `validatePlacement` returns `VALID` — verified by exhaustive cross-check against Section 10's own brute-force definition (a tautological but valuable regression guard if the implementation is later optimized per Section 16).

---

## 19. Acceptance Criteria

- A Shape commits to the Board if and only if every one of its local cells maps to an in-bounds, `EMPTY` target cell (Section 4); no partial commit is ever observable (Section 5).
- Every fully-filled row and column present in the Board state immediately after a Placement is cleared in the same resolution step, with no cell cleared more than once (Sections 6–8).
- `hasAnyLegalPlacement` exhaustively considers every Tray Shape at every Board origin and returns `false` only when none has any legal placement (Section 10).
- The Board Engine emits `PlacementCommitted` and `BoardChanged` on every Turn, and additionally `LinesCleared`/`CellsCleared` if and only if at least one line cleared (Section 12).
- Given identical inputs, all Board Engine outputs (resulting state and emitted events) are byte-identical across repeated runs (Section 13).
- No Extensible state (Section 3.2) affects MVP behavior unless and until a governing document activates it (Section 9).
- All Section 14 fixtures and Section 18 properties pass under automated test.

---

## 20. Cross-Document Contracts

- **Consumes from `01_GAMEPLAY_SPECIFICATION.md`:** Board size (Section 23), the two-state MVP model and simultaneous-clear rule (Section 5), Turn definition (Section 7), Board Lock as the default failure trigger (Section 15), and the Enemy Mechanic categories this document reserves extension points for (Section 11).
- **Consumes from `02_SHAPE_LIBRARY.md`:** the coordinate/normalization contract (Section 2), the Canonical Shape Schema fields this engine reads — `local_coordinates`, `block_count`, `width`, `height` (Section 3, also named explicitly in Shape Library Section 14 as consumed by "Board Engine doc") — and the Section 11 test-case behaviors this document's Section 14/18 fixtures are built to match.
- **Exposes to Combat/Scoring doc:** `LinesCleared`/`CellsCleared` events (Section 12) as the sole input to Damage Pipeline stage "Detect Clear" (Gameplay Spec Section 9); the Board Engine does not compute damage or Combo values itself.
- **Exposes to Objectives doc:** Board Queries (Section 11) and the full event stream (Section 12) as the only legal data sources for line/cumulative-count Objective predicates (Gameplay Spec Section 12); Objectives may not read Board internals outside these seams.
- **Exposes to Enemy Content doc (future):** the three Section 9 extension seams (`CanReceiveBlock`, `CountsTowardLineClear`, post-clear hook) as the only sanctioned integration points for Block/Freeze/Hazard mechanics (Gameplay Spec Section 11).
- **Exposes to autosave/backgrounding (Gameplay Spec Section 20):** `getBoardSnapshot()` (Section 11) as the serializable Board state; Section 13's determinism guarantee is what makes resuming a Turn mid-resolution safe to reconstruct.
- This document does not embed content owned by the above; it references them structurally only, per the pattern established in `01_GAMEPLAY_SPECIFICATION.md` Section 25 and `02_SHAPE_LIBRARY.md` Section 14.

---

## 21. Final Turn Resolution

Canonical logical sequence, from a validated piece placement through full Board resolution, expressed strictly in terms of this document's own functions and events (nests inside Gameplay Spec Section 4's `PLACEMENT_RESOLUTION → LINE_CLEAR_RESOLUTION` states):

```text
Input: shape, origin, board (current state)

1. validatePlacement(shape, origin, board)
   → if INVALID: return failure; board unchanged; caller routes to PIECE_SELECTED/ACTIVE (Gameplay Spec Section 4)

2. commitPlacement(shape, origin, board)          [Section 5]
   → board cells for shape.local_coordinates set occupancy = FILLED, modifier = NORMAL, tagged with new placementId
   → emit PlacementCommitted

3. detectLines(board)                              [Section 6]
   → compute filledRows, filledCols against the single post-commit board state

4. clearLines(board, filledRows, filledCols)        [Section 8]
   → build cellsToClear (deduplicated set union)
   → set every cell in cellsToClear to occupancy = EMPTY, modifier = NORMAL
   → if cellsToClear ≠ ∅: emit LinesCleared, emit CellsCleared

5. emit BoardChanged                                [Section 12]
   → carries final post-Turn board snapshot

6. return control to Gameplay Spec COMBAT_RESOLUTION
   → Combat layer consumes LinesCleared (if any) as Damage Pipeline "Detect Clear" input (Gameplay Spec Section 9)
   → Objectives layer consumes the same events for cumulative-count Objectives (Gameplay Spec Section 12)

7. (later, at next ACTIVE entry, Gameplay Spec Section 4)
   hasAnyLegalPlacement(tray, board)                [Section 10]
     if false: emit BoardLockDetected; caller converts to Battle Failure -> Run Defeat
   → if true: caller proceeds normally
```

Every step above is deterministic (Section 13) and atomic within its own boundary (Section 5); no step is ever partially applied or reordered.

---

**End of `03_BOARD_ENGINE_AND_RULES.md`.**
