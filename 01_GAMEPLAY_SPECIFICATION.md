# 01_GAMEPLAY_SPECIFICATION.md
## Block Battles — Gameplay Specification

**Governing document:** `00_MASTER_GAME_VISION.md`
**Document status:** Implementation-level ruleset

---

## 1. Purpose and Ownership

This document defines the complete playable ruleset for Block Battles: board rules, tray rules, turn definition, scoring and combo resolution, board lock detection, Game Over flow, and the separation between Mode 1 (Classic Endless) and Mode 2 (Goal-Driven Hurdles). A programmer must be able to implement the complete gameplay loop from this document without inferring undocumented behavior.

**Ownership:** This file owns gameplay *rules and state flow*. It does not own exact shape geometry (`02_SHAPE_LIBRARY.md`), RNG weighting (`04_SPAWN_RNG_AND_DIFFICULTY.md`), exact scoring tables (`05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`), or gesture handling (`06_INPUT_DRAG_AND_DROP_UX.md`).

---

## 2. Canonical Terminology

- **Turn:** One resolved player piece placement.
- **Tray:** The set of up to 3 pieces currently available to the player.
- **Board Lock:** A mathematical board condition where none of the available tray pieces can be legally placed anywhere on the 8×8 board.
- **Game Over:** The player-facing failure presentation triggered by Board Lock.
- **High Score:** The persistent all-time highest score recorded on the local device.
- **L-Value:** The total number of lines (rows + columns) cleared by a single piece placement (1 to 4+).
- **Streak / Combo:** Consecutive placement turns that clear at least one line, applying an escalating score multiplier.

---

## 3. Overall App State Machine

```text
APP LAUNCH
    ↓
SPLASH SCREEN (Smooth branded intro)
    ↓
HOME SCREEN (Minimalist: Play Classic, High Score Trophy, Settings, Goal Mode Portal)
    │
    ├─────────────────────────────────────────┐
    ▼                                         ▼
[MODE 1: CLASSIC MODE]              [MODE 2: GOAL-DRIVEN MODE (Reserved)]
    ↓                                         ↓
GAMEPLAY CANVAS (8×8 Board + Tray)       LEVEL HURDLES (Target Score, Obstacles)
    ↓
BOARD LOCK (Zero valid placements)
    ↓
GAME OVER DIALOG (Final Score, New Best Celebration, Instant Replay / Home)
```

---

## 4. Classic In-Game Turn State Machine

| State | Description | Legal Next States |
|---|---|---|
| `ACTIVE` | Waiting for player input; no piece currently selected. | `DRAGGING`, `BOARD_LOCK_GAME_OVER` |
| `DRAGGING` | Piece follows pointer with 48dp finger-lift offset; live ghost preview snaps to grid with valid (green) or invalid (red) feedback. | `PLACEMENT_RESOLUTION` (valid drop), `ACTIVE` (invalid drop / cancel) |
| `PLACEMENT_RESOLUTION` | Piece committed to board cells atomically; tray slot marked consumed; turn placement points awarded. | `LINE_CLEAR_RESOLUTION` |
| `LINE_CLEAR_RESOLUTION` | Fully-filled rows and columns identified simultaneously; intersection cells cleared once; white-hot flash, particle bursts, and camera shake trigger. | `SCORING_AND_STREAK` |
| `SCORING_AND_STREAK` | Line clear points and multi-line bonuses awarded; combo streak incremented if lines cleared, or reset to 1.0x if zero lines cleared. Floating score popups spawn. | `TRAY_EVALUATION` |
| `TRAY_EVALUATION` | If all 3 tray pieces consumed, generate a new batch of 3 pieces via `TrayEngine` with solvability protection. | `BOARD_LOCK_EVALUATION` |
| `BOARD_LOCK_EVALUATION` | `BoardLockDetector` exhaustively checks all remaining tray pieces across all 64 board cells. | `ACTIVE` (moves exist), `BOARD_LOCK_GAME_OVER` (zero moves exist) |
| `BOARD_LOCK_GAME_OVER` | Final score evaluated against local High Score; celebratory Game Over modal displayed with "Play Again" and "Home". | `ACTIVE` (restart), `HOME` (exit) |

---

## 5. Board Rules

- The Board is an **8×8 grid** of cells, indexed `(row, col)` with `row, col ∈ [0, 7]`.
- **Cell State Model:** Each Board cell carries two logically distinct attributes: **Occupancy** and a **Cell Modifier**. These are stored compactly (see `03_BOARD_ENGINE_AND_RULES.md` Section 3) but are semantically independent.

  | Occupancy | Modifier | Meaning |
  |---|---|---|
  | Empty | Normal | **Normal Empty.** Default state. Can receive a Piece. |
  | Filled | Normal | **Normal Filled.** Contains a regular placed block. Counts toward Line Clear detection. |
  | Empty | Blocked | **Blocked.** Cannot receive a Piece; does not count toward Line Clear. (Reserved for Mode 2 Goal-Driven Obstacles, Section 12.) |
  | Filled | Frozen | **Frozen.** Occupied, but does not count toward Line Clear detection until unfrozen. (Reserved for Mode 2 Goal-Driven Obstacles, Section 12.) |
  | Filled | Indestructible | **Indestructible.** Occupied, never clearable, never overwritten. (Reserved for Mode 2 Goal-Driven Obstacles, Section 12.) |

  **DESIGN DECISION:** `Normal Empty` and `Normal Filled` are the core Classic Endless states. `Blocked` and `Frozen` are reserved as architectural states for Mode 2 (Goal-Driven Hurdles). The full technical predicate model is owned by `03_BOARD_ENGINE_AND_RULES.md` Section 9.

- **Placement:** A Piece may be placed at a target origin cell if and only if every cell the Piece's shape occupies (relative to that origin) exists on the Board AND has `CanReceiveBlock == true`. At MVP: `Normal Empty` cells satisfy this; `Blocked`, `Filled`, and all other states do not. An illegal placement is rejected and returns control to `PIECE_SELECTED`/`ACTIVE` (Section 4).
- **No rotation:** A Piece is placed exactly in the orientation it is presented in the tray. The Piece pool (owned by the Shapes doc, Section 25) is responsible for including any rotated variants needed for board solvability; this document does not permit runtime rotation as a player action.
- **Clearing:** After a Placement resolves, every row that satisfies `CountsTowardLineClear == true` for all 8 cells and every column that satisfies `CountsTowardLineClear == true` for all 8 cells is identified **simultaneously** (based on the single resulting board state, not sequentially), then all identified rows/columns are cleared (occupancy set to `Empty`, modifier reset to `Normal`) together in one resolution step. `Frozen` cells do NOT count toward Line Clear detection; `Blocked` cells are not Filled and therefore trivially do not contribute.
- **DESIGN DECISION:** A cell that is part of both a clearing row and a clearing column is cleared once; no double-counting of the cell itself (only of line-count for Combo purposes, Section 8).

---

## 6. Tray Rules

- The Tray holds up to **3 Pieces** at any time.
- **Generation:** At session start (`PREPARING`), the Tray is fully generated (3 Pieces) per rules owned by the RNG doc (Section 25).
- **Consumption:** When a Piece is placed, its Tray slot is marked consumed and removed from the visible Tray.
- **Replacement timing — DESIGN DECISION:** New Pieces are generated **only when all 3 Tray slots have been consumed**, and all 3 are generated together as a new batch. Pieces are not replaced one at a time as they are used. This keeps piece-availability planning legible (Master Vision Section 5.1, Instant Clarity) and gives the RNG/solvability system (Section 25) a single well-defined batch boundary to validate against.
- **Order independence:** The player may place the 3 Tray Pieces in any order they choose; no slot has positional priority.

---

## 7. Turn Definition

> **One Turn = one resolved Placement** (i.e., one full pass through `PLACEMENT_RESOLUTION` → `LINE_CLEAR_RESOLUTION` → `SCORING_AND_STREAK` → `TRAY_EVALUATION` → `BOARD_LOCK_EVALUATION`).

This is the atomic unit that:
- Commits block cells to the 8×8 grid.
- Consumes the placed piece from the tray.
- Awards placement points and line-clear points.
- Increments combo streak (if lines cleared) or resets streak to 1.0× (if 0 lines cleared).
- Spawns floating score text popups.
- Checks if all 3 tray pieces have been consumed (triggers new batch generation if empty).
- Evaluates Board Lock condition across all remaining tray pieces.

---

## 8. Pure Puzzle Scoring Rules

- **Placement Points:** Every placed block awards 1 point per cell occupied (e.g., a 3-cell piece awards 3 points).
- **Line Clear Points:** Completing 1 or more full lines (rows or columns) awards points based on the number of lines cleared simultaneously ($L$):
  - $L = 1$: 100 points
  - $L = 2$: 300 points ($1.5\times$ per line)
  - $L = 3$: 600 points ($2.0\times$ per line)
  - $L = 4$: 1,000 points ($2.5\times$ per line)
  - $L \ge 5$: $1,000 + (L - 4) \times 500$ points
- **Streak Multiplier:** Consecutive turns that clear lines build an escalating multiplier applied to line clear points:
  - Streak 0 (initial/reset): $1.0\times$
  - Streak 1 (2 consecutive clearing turns): $1.5\times$
  - Streak 2 (3 consecutive clearing turns): $2.0\times$
  - Streak 3+ (4+ consecutive clearing turns): $2.5\times$ (cap)
- **Turn Total Formula:**
  $$\text{TurnScore} = \text{PlacementPoints} + (\text{BaseLineClearPoints}(L) \times \text{StreakMultiplier})$$
- Detailed mathematical tables and audio/visual pitch escalation are governed by `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`.

---

## 9. Line Clearing and Simultaneous Combos

- Fully filled rows and columns are evaluated **simultaneously** from the post-placement board state.
- If a cell is part of both a filled row and a filled column (intersection cell), it is cleared once and produces a cross-clear particle burst.
- Simultaneous clears ($L \ge 2$) trigger elevated visual juice (white-hot flash, heavy camera shake, particle ring, dynamic text popups).

---

## 10. Local High Score Competition

- The player's running score is continuously compared against `classic_all_time_high_score` stored locally on the device.
- When current score surpasses the all-time best, the HUD's BEST trophy widget pulses with a golden glow.
- On Board Lock (Game Over), the game presents a celebration screen highlighting whether a new record was established, displaying total score, lines cleared, and max streak.

---

## 11. Mode 1: Classic Endless Mode (MVP Core Focus)

- **Objective:** Survive as long as possible and maximize high score.
- **Board:** Empty 8×8 grid at start.
- **Duration:** Endless until Board Lock occurs.
- **Failure Condition:** Board Lock is the only game-ending condition.
- **Offline:** 100% playable offline with zero network latency, accounts, or ads.

---

## 12. Mode 2: Goal-Driven Mode (Reserved Architecture)

- **Structure:** Level-based stages with discrete completion criteria.
- **Hurdles & Objectives:**
  - *Target Score:* Reach $X$ points.
  - *Obstacle Clears:* Clear pre-placed frozen or blocked obstacle cells within the grid.
  - *Move Limit:* Complete objectives within $M$ total piece placements.
- **Architecture Reservation:** Handled via `LevelDefinition` contracts in `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md` without polluting Classic Mode logic.

---

## 13. Failure Rules (Board Lock)

1. **Board Lock Definition:** A mathematical state where none of the remaining pieces currently in the tray can legally fit into any valid location on the 8×8 board.
2. **Evaluation Timing:** Evaluated at `BOARD_LOCK_EVALUATION` at the end of every placement turn, and immediately after a fresh tray batch is generated.
3. **Exhaustive Check:** The algorithm checks every cell $(r, c) \in [0..7] \times [0..7]$ for every remaining tray piece. If at least one valid placement exists, the game remains `ACTIVE`. If zero valid placements exist, the game transitions to `BOARD_LOCK_GAME_OVER`.

---

## 14. Game Over & Instant Replay Loop

```text
BOARD LOCK DETECTED
    ↓
BOARD GREYS OUT (Tactile failure audio & visual lock indicator)
    ↓
GAME OVER DIALOG
    ├── Final Score & Best Score Comparison
    ├── "NEW BEST!" Fanfare (if applicable)
    ├── [ PLAY AGAIN ] → Resets 8×8 board, seeds fresh Tray, immediate start
    └── [ HOME ] → Returns to Minimalist Home Screen
```

---

## 15. Backgrounding / App Lifecycle

- When the application is paused or backgrounded, current game state (board occupancy, score, streak, tray pieces, and RNG seed state) is autosaved atomically to local storage via Hive CE (`SaveRepository`).
- Resuming the app restores the exact state without penalty or lost progress.

---

## 16. Edge Cases

| Edge Case | Resolution |
|---|---|
| Single piece cannot fit, but other tray pieces can | Game remains `ACTIVE`. Player must use surviving pieces to create space. |
| Board completely cleared (Board Wipe) | Bonus 500 points awarded + "BOARD CLEAR!" celebration fanfare. |
| Multiple rows and columns cleared at once | Cleared simultaneously; intersection cells cleared once; $L = \text{rows} + \text{cols}$. |
| App closed mid-drag | On relaunch, piece returns to tray untouched; board state unchanged. |

---

## 17. Acceptance Criteria

- A piece can only be placed on empty, valid in-bounds cells.
- Fully-filled rows and columns clear simultaneously in a single turn resolution.
- Tray generates batches of exactly 3 pieces when all slots are empty.
- Board Lock is detected accurately with zero false positives or false negatives.
- Scores calculate deterministically per the formula in Section 8.
- High score persists reliably across app restarts.

---

## 18. Final Gameplay Contract

Given a game session in state `ACTIVE`, the deterministic turn sequence is:

```text
ACTIVE (Waiting for player input)
 → DRAGGING (Piece follows pointer with lift offset + snapping ghost preview)
 → PLACEMENT_RESOLUTION (Piece committed atomically; tray slot cleared)
 → LINE_CLEAR_RESOLUTION (Filled rows/cols cleared simultaneously)
 → SCORING_AND_STREAK (Placement + line clear points + streak multiplier applied)
 → TRAY_EVALUATION (Refill batch of 3 if all slots consumed)
 → BOARD_LOCK_EVALUATION (Exhaustive move check)
   ├── Moves exist → return to ACTIVE
   └── Zero moves exist → BOARD_LOCK_GAME_OVER
```
