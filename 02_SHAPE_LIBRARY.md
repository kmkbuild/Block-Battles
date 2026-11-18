# 02_SHAPE_LIBRARY.md
## Block Battles — Shape Library

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`
**Document status:** Implementation-level content registry

---

## 1. Purpose

This document owns every logical playable shape (Piece) in Block Battles: its exact geometry, its metadata, and the rules for validating and referencing it. `01_GAMEPLAY_SPECIFICATION.md` Section 5 establishes that Pieces are placed **without runtime rotation**; this document is therefore the authority on how rotation is handled by registering each needed orientation as its own distinct Shape entry.

Shapes are treated as data objects, not pure geometry: their metadata (category, tags, strategic classification) is the interface that Scoring, Stage Objectives, and RNG systems use to reference groups of shapes without hardcoding individual geometry. This document owns that metadata contract.

**Does not own:** RNG draw weighting (RNG doc), Stage Objective predicate implementations (Objectives doc), or visual rendering/animations (Art/VFX docs) — see Section 14.

---

## 2. Coordinate System

- **Origin:** Each Shape's local coordinate origin is the top-left cell of its bounding box, `(row, col) = (0, 0)`.
- **Axes:** `row` increases **downward**; `col` increases **rightward**. This matches the Board coordinate convention defined in `01_GAMEPLAY_SPECIFICATION.md` Section 5.
- **Normalization:** Every Shape's local coordinate list is stored normalized such that `min(row) == 0` and `min(col) == 0` across all its cells. No shape may be stored with negative or non-minimal offsets.
- **Bounding box:** `width = max(col) + 1`, `height = max(row) + 1`, computed from the normalized coordinate list. The bounding box is the minimal rectangle containing all of a Shape's cells.
- **Board placement:** At placement time, a Shape's origin is mapped to a target Board cell `(boardRow, boardCol)`; every local cell `(r, c)` occupies Board cell `(boardRow + r, boardCol + c)`, per the legality rule in `01_GAMEPLAY_SPECIFICATION.md` Section 5.

---

## 3. Canonical Shape Schema

Every registered Shape entry contains exactly these fields:

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique identifier, format `SHP_0##`. Immutable once shipped (Section 13). |
| `display_name` | string | Player-facing name for accessibility and display. |
| `internal_name` | string | Developer-facing snake_case name used in code/config. |
| `category` | enum | Gameplay, objective, and RNG grouping — see Section 5. |
| `family` | string | Geometric family this orientation belongs to (e.g., `TETROMINO_L`). |
| `local_coordinates` | list of (row, col) | Normalized cell list per Section 2. |
| `block_count` | integer | `len(local_coordinates)`. |
| `width` | integer | Bounding box width per Section 2. |
| `height` | integer | Bounding box height per Section 2. |
| `symmetry` | enum | `NONE`, `MIRROR`, `POINT`, `AXIAL`, `BIAXIAL`, `FULL`. |
| `orientation` | string | Orientation label (`Base`, `90°`, `180°`, `270°`, `Horizontal`, `Vertical`, or `N/A (symmetric)`). |
| `tags` | list of enum | Reusable metadata tags — see Section 6. |
| `strategic_classification` | enum | Gameplay-role label — see Section 7. |

---

## 4. Shape Registry

Because Pieces cannot be rotated at runtime, each geometric **family** is registered as multiple **orientation entries** — one per rotation needed for solvable play:

- **15 Shape Families** (distinct geometric identities).
- **34 registered Shape entries** (families × required orientations) — the complete canonical catalog available in the engine.

### Family Overview

| # | Family | Block Count | Orientations Registered | Symmetry |
|---|---|---|---|---|
| 1 | Single | 1 | 1 (symmetric) | FULL |
| 2 | Domino | 2 | 2 (H, V) | AXIAL |
| 3 | Tromino Line | 3 | 2 (H, V) | AXIAL |
| 4 | Tromino Corner | 3 | 4 (all corner-missing variants) | NONE |
| 5 | Tetromino Line | 4 | 2 (H, V) | AXIAL |
| 6 | Tetromino Square (O) | 4 | 1 (symmetric) | FULL |
| 7 | Tetromino L | 4 | 4 (Base, 90°, 180°, 270°) | NONE |
| 8 | Tetromino J | 4 | 4 (Base, 90°, 180°, 270°) | NONE |
| 9 | Tetromino T | 4 | 4 (Base, 90°, 180°, 270°) | MIRROR |
| 10 | Tetromino S | 4 | 2 (H, V) | POINT |
| 11 | Tetromino Z | 4 | 2 (H, V) | POINT |
| 12 | Pentomino Line | 5 | 2 (H, V) | AXIAL |
| 13 | Pentomino Plus (Cross) | 5 | 1 (symmetric) | FULL |
| 14 | Square 3×3 | 9 | 1 (symmetric) | FULL |
| 15 | Rectangle 2×3 | 6 | 2 (2×3, 3×2) | BIAXIAL |

---

## 5. Shape Categories

The `category` field groups Shapes for gameplay balance, Stage Objective predicates, and RNG weighting:

| Category | Includes Families | Primary Use |
|---|---|---|
| `SINGLE` | Single | Minimal piece; precision gap-filling; RNG floor-filler weighting. |
| `LINE` | Domino, Tromino Line, Tetromino Line, Pentomino Line | Direct line-clear enablers; horizontal/vertical channel completion. |
| `CORNER` | Tromino Corner | Low-risk small filler with irregular footprint; early-game staple. |
| `SQUARE` | Tetromino Square (O), Square 3×3 | Dense, gap-free filler; compact spatial placement. |
| `RECTANGLE` | Rectangle 2×3 | Large, dense, non-square filler; distinct balance profile. |
| `L_FAMILY` | Tetromino L, Tetromino J | Irregular, high-tactical-tension pieces; corner/edge cleanup. |
| `T_FAMILY` | Tetromino T | Single-axis-symmetric pieces; center anchoring. |
| `S_Z_FAMILY` | Tetromino S, Tetromino Z | Irregular, point-symmetric pieces; high-skill placement pieces. |
| `CROSS` | Pentomino Plus | Unique high-footprint, fully-symmetric piece; high-risk multi-line setup. |

---

## 6. Shape Tags

Tags are derived, boolean, non-exclusive metadata computed from a Shape's own fields, ensuring they never drift out of sync with geometry:

| Tag | Rule | Purpose |
|---|---|---|
| `SINGLE` | `block_count == 1` | Targets minimal 1-cell pieces. |
| `SMALL` | `block_count <= 3` | Targets low-commitment pieces. |
| `LARGE` | `block_count >= 5` | Targets high-commitment, high-placement-score pieces. |
| `LINE` | `(width == 1 or height == 1) and block_count > 1` | Direct line-clear enablers. |
| `WIDE` | `width > height` | Horizontal clearing and board-width planning. |
| `TALL` | `height > width` | Vertical clearing and column-depth planning. |
| `COMPACT` | `width * height == block_count` and `width > 1` and `height > 1` | Fully gap-free solid blocks (2×2, 3×3, 2×3). |
| `IRREGULAR` | `width * height > block_count` | Shapes with internal concave pockets or missing corners. |
| `SYMMETRIC` | `symmetry != NONE` | Shapes exhibiting mirror, rotational, or full symmetry. |

---

## 7. Strategic Classifications

Strategic classifications describe how a Shape functions in puzzle flow:

| Family | Strengths | Weaknesses | Strategic Opportunities | Dangerous Board States |
|---|---|---|---|---|
| Single | Fits anywhere; zero placement risk | Contributes only 1 cell toward a line | Precision gap-filling to complete a stalled line | Rarely dangerous; risk is tray opportunity cost |
| Domino | Very flexible; slots into small gaps | Low placement points (+2) | Cheap, reliable partial-line progress | Low risk; small impact on crowded board |
| Tromino Line | Direct progress toward a 3-cell line segment | Requires 3 contiguous open cells in one axis | Reliable mid-size line contribution | Heavily fragmented boards with no 3-cell runs |
| Tromino Corner | Fills irregular gaps other pieces cannot | Leaves an asymmetric footprint | Cleaning up single-cell-offset gaps | Repeated placement can isolate single cells |
| Tetromino Line | Completes half of an 8-cell line at once | Requires 4 contiguous open cells | Efficient, high-value line clears | Boards with no open 4-run become unplaceable |
| Tetromino Square | Dense, symmetric, easy to nest | Commits a full 2×2 area | Reliable filler that never creates awkward gaps | Tight pockets smaller than 2×2 reject it |
| Tetromino L / J | Bridges perpendicular partial lines | Leaves an asymmetric notch | High tactical value for dual-line setups | Careless placement creates single-cell notches |
| Tetromino T | Flexible with one axis of symmetry | Central stem can divide open space | Ideal for centering within open plus-shaped gaps | Leaves two small side gaps without planning |
| Tetromino S / Z | Irregular; rewards forward planning | Hardest tetrominoes to place without stray gaps | Bridges staggered diagonal pockets | Flat boundaries create unreachable holes |
| Pentomino Line | Clears 5 cells of a line in one placement | Requires 5 contiguous open cells | Massive line clear tool when a long run is open | High risk of board lock on cluttered boards |
| Pentomino Plus | Simultaneously touches 5 rows and columns | Requires open space on all 4 sides of a center | Unique multi-line setup potential | Extremely difficult to fit on crowded boards |
| Square 3×3 | Highest single placement score (+9 points) | Requires a large 3×3 open area | Early game massive line clear setups | Severe board lock hazard on crowded boards |
| Rectangle 2×3 | High placement score (+6 points) | Requires 6 contiguous cells | Fills wide pockets cleanly | High lock risk if board space is fragmented |

---

## 8. Scoring and Tactical Relevance

The shape metadata (`category`, `tags`, `block_count`, `width`, `height`) governs scoring calculations and stage evaluation:

1. **Placement Points:** Every shape awards points equal to its `block_count` ($1\text{ pt/cell}$). A 3×3 square awards +9 points, while a Domino awards +2 points.
2. **Multi-Line Clear Potential:** Shapes with `WIDE` or `TALL` tags provide high probability of intersecting both horizontal and vertical lines, triggering simultaneous $L \ge 2$ clears.
3. **Streak Maintenance:** Small pieces (`SMALL` tag) provide high placement flexibility to sustain active streak multipliers ($1.2\times \to 2.5\times$) during tight board situations.

---

## 9. Stage Objective Relevance

Mode 2 Stage Objectives query shape metadata rather than hardcoded IDs to evaluate puzzle goals:
- *"Clear 3 lines using only `LINE`-category pieces."*
- *"Complete the stage without placing any `IRREGULAR`-tagged pieces."*
- *"Place 5 `COMPACT`-tagged pieces to unlock the star milestone."*

---

## 10. Shape Validation Rules

Every registered Shape entry must satisfy:
1. `local_coordinates` contains no duplicate `(row, col)` pairs.
2. `local_coordinates` is normalized (`min(row) == 0`, `min(col) == 0`).
3. The cell set is orthogonally connected (every cell reachable via adjacent edges).
4. `block_count == len(local_coordinates)`.
5. `width == max(col) + 1` and `height == max(row) + 1`.
6. Bounding box contains no fully empty border row or column.
7. `width <= 8` and `height <= 8`.
8. `id` is globally unique and immutable.
9. `tags` and `symmetry` are strictly derived from geometry.

---

## 11. Shape Test Cases

| Test | Shape | Board State | Target Origin | Expected Result |
|---|---|---|---|---|
| T1 | `SHP_001` (Single) | Empty | `(0,0)` | Legal — placed at `(0,0)`. |
| T2 | `SHP_010` (Tetromino Line H) | Empty | `(7,5)` | Illegal — cells `(7,5)-(7,8)` exceed Board bounds (`col` max index 7). |
| T3 | `SHP_012` (Tetromino Square O) | Empty | `(0,7)` | Illegal — requires columns 7 and 8; column 8 does not exist. |
| T4 | `SHP_013` (Tetromino L, Base) | Cells `(2,1)` already Filled | `(0,1)` | Illegal — local cell `(2,0)` maps to `(2,1)`, which is occupied. |
| T5 | `SHP_031` (Pentomino Plus) | Empty | `(3,3)` | Legal — occupies `(3,4),(4,3),(4,4),(4,5),(5,4)`. |
| T6 | `SHP_032` (Square 3×3) | Row 6 fully Filled | `(5,0)` | Illegal — target cells intersect filled row; rejected. |
| T7 | `SHP_025` (Tetromino S, Horizontal) | Empty | `(0,0)` | Legal — occupies `(0,1),(0,2),(1,0),(1,1)`. |
| T8 | Any Tray with 3 pieces, none placeable | Crowded | — | Board Lock condition met; triggers Game Over. |

---

## 12. Shape Registry Table (34 Canonical Shapes)

| ID | Display Name | Family | Category | Local Coordinates | Count | W | H | Symmetry | Orientation | Tags | Strategic Class |
|---|---|---|---|---|---|---|---|---|---|---|---|
| SHP_001 | Single Block | Single | SINGLE | (0,0) | 1 | 1 | 1 | FULL | N/A (symmetric) | SINGLE, SMALL, SYMMETRIC | Filler |
| SHP_002 | Domino (H) | Domino | LINE | (0,0)(0,1) | 2 | 2 | 1 | AXIAL | Horizontal | SMALL, LINE, WIDE, SYMMETRIC | Line Enabler |
| SHP_003 | Domino (V) | Domino | LINE | (0,0)(1,0) | 2 | 1 | 2 | AXIAL | Vertical | SMALL, LINE, TALL, SYMMETRIC | Line Enabler |
| SHP_004 | Tromino Line (H) | Tromino Line | LINE | (0,0)(0,1)(0,2) | 3 | 3 | 1 | AXIAL | Horizontal | SMALL, LINE, WIDE, SYMMETRIC | Line Enabler |
| SHP_005 | Tromino Line (V) | Tromino Line | LINE | (0,0)(1,0)(2,0) | 3 | 1 | 3 | AXIAL | Vertical | SMALL, LINE, TALL, SYMMETRIC | Line Enabler |
| SHP_006 | Tromino Corner (TL) | Tromino Corner | CORNER | (0,0)(0,1)(1,0) | 3 | 2 | 2 | NONE | Base | SMALL, IRREGULAR | Tactician |
| SHP_007 | Tromino Corner (TR) | Tromino Corner | CORNER | (0,0)(0,1)(1,1) | 3 | 2 | 2 | NONE | 90° | SMALL, IRREGULAR | Tactician |
| SHP_008 | Tromino Corner (BR) | Tromino Corner | CORNER | (0,1)(1,0)(1,1) | 3 | 2 | 2 | NONE | 180° | SMALL, IRREGULAR | Tactician |
| SHP_009 | Tromino Corner (BL) | Tromino Corner | CORNER | (0,0)(1,0)(1,1) | 3 | 2 | 2 | NONE | 270° | SMALL, IRREGULAR | Tactician |
| SHP_010 | Tetromino Line (H) | Tetromino Line | LINE | (0,0)(0,1)(0,2)(0,3) | 4 | 4 | 1 | AXIAL | Horizontal | LINE, WIDE, SYMMETRIC | Line Enabler |
| SHP_011 | Tetromino Line (V) | Tetromino Line | LINE | (0,0)(1,0)(2,0)(3,0) | 4 | 1 | 4 | AXIAL | Vertical | LINE, TALL, SYMMETRIC | Line Enabler |
| SHP_012 | Tetromino Square (O) | Tetromino Square | SQUARE | (0,0)(0,1)(1,0)(1,1) | 4 | 2 | 2 | FULL | N/A (symmetric) | SQUARE, COMPACT, SYMMETRIC | Filler |
| SHP_013 | Tetromino L (Base) | Tetromino L | L_FAMILY | (0,0)(1,0)(2,0)(2,1) | 4 | 2 | 3 | NONE | Base | TALL, IRREGULAR | Tactician |
| SHP_014 | Tetromino L (90°) | Tetromino L | L_FAMILY | (0,0)(0,1)(0,2)(1,0) | 4 | 3 | 2 | NONE | 90° | WIDE, IRREGULAR | Tactician |
| SHP_015 | Tetromino L (180°) | Tetromino L | L_FAMILY | (0,0)(0,1)(1,1)(2,1) | 4 | 2 | 3 | NONE | 180° | TALL, IRREGULAR | Tactician |
| SHP_016 | Tetromino L (270°) | Tetromino L | L_FAMILY | (0,2)(1,0)(1,1)(1,2) | 4 | 3 | 2 | NONE | 270° | WIDE, IRREGULAR | Tactician |
| SHP_017 | Tetromino J (Base) | Tetromino J | L_FAMILY | (0,1)(1,1)(2,0)(2,1) | 4 | 2 | 3 | NONE | Base | TALL, IRREGULAR | Tactician |
| SHP_018 | Tetromino J (90°) | Tetromino J | L_FAMILY | (0,0)(1,0)(1,1)(1,2) | 4 | 3 | 2 | NONE | 90° | WIDE, IRREGULAR | Tactician |
| SHP_019 | Tetromino J (180°) | Tetromino J | L_FAMILY | (0,0)(0,1)(1,0)(2,0) | 4 | 2 | 3 | NONE | 180° | TALL, IRREGULAR | Tactician |
| SHP_020 | Tetromino J (270°) | Tetromino J | L_FAMILY | (0,0)(0,1)(0,2)(1,2) | 4 | 3 | 2 | NONE | 270° | WIDE, IRREGULAR | Tactician |
| SHP_021 | Tetromino T (Base) | Tetromino T | T_FAMILY | (0,0)(0,1)(0,2)(1,1) | 4 | 3 | 2 | MIRROR | Base | WIDE, IRREGULAR, SYMMETRIC | Anchor |
| SHP_022 | Tetromino T (90°) | Tetromino T | T_FAMILY | (0,1)(1,0)(1,1)(2,1) | 4 | 2 | 3 | MIRROR | 90° | TALL, IRREGULAR, SYMMETRIC | Anchor |
| SHP_023 | Tetromino T (180°) | Tetromino T | T_FAMILY | (0,1)(1,0)(1,1)(1,2) | 4 | 3 | 2 | MIRROR | 180° | WIDE, IRREGULAR, SYMMETRIC | Anchor |
| SHP_024 | Tetromino T (270°) | Tetromino T | T_FAMILY | (0,0)(1,0)(1,1)(2,0) | 4 | 2 | 3 | MIRROR | 270° | TALL, IRREGULAR, SYMMETRIC | Anchor |
| SHP_025 | Tetromino S (H) | Tetromino S | S_Z_FAMILY | (0,1)(0,2)(1,0)(1,1) | 4 | 3 | 2 | POINT | Horizontal | WIDE, IRREGULAR, SYMMETRIC | Precision Piece |
| SHP_026 | Tetromino S (V) | Tetromino S | S_Z_FAMILY | (0,0)(1,0)(1,1)(2,1) | 4 | 2 | 3 | POINT | Vertical | TALL, IRREGULAR, SYMMETRIC | Precision Piece |
| SHP_027 | Tetromino Z (H) | Tetromino Z | S_Z_FAMILY | (0,0)(0,1)(1,1)(1,2) | 4 | 3 | 2 | POINT | Horizontal | WIDE, IRREGULAR, SYMMETRIC | Precision Piece |
| SHP_028 | Tetromino Z (V) | Tetromino Z | S_Z_FAMILY | (0,1)(1,0)(1,1)(2,0) | 4 | 2 | 3 | POINT | Vertical | TALL, IRREGULAR, SYMMETRIC | Precision Piece |
| SHP_029 | Pentomino Line (H) | Pentomino Line | LINE | (0,0)(0,1)(0,2)(0,3)(0,4) | 5 | 5 | 1 | AXIAL | Horizontal | LINE, WIDE, LARGE, SYMMETRIC | Finisher |
| SHP_030 | Pentomino Line (V) | Pentomino Line | LINE | (0,0)(1,0)(2,0)(3,0)(4,0) | 5 | 1 | 5 | AXIAL | Vertical | LINE, TALL, LARGE, SYMMETRIC | Finisher |
| SHP_031 | Pentomino Plus | Pentomino Plus | CROSS | (0,1)(1,0)(1,1)(1,2)(2,1) | 5 | 3 | 3 | FULL | N/A (symmetric) | LARGE, IRREGULAR, CROSS, SYMMETRIC | High-Risk Finisher |
| SHP_032 | Square 3×3 | Square 3×3 | SQUARE | (0,0)(0,1)(0,2)(1,0)(1,1)(1,2)(2,0)(2,1)(2,2) | 9 | 3 | 3 | FULL | N/A (symmetric) | LARGE, SQUARE, COMPACT, SYMMETRIC | High-Risk Finisher |
| SHP_033 | Rectangle 2×3 | Rectangle | RECTANGLE | (0,0)(0,1)(0,2)(1,0)(1,1)(1,2) | 6 | 3 | 2 | BIAXIAL | Base | LARGE, COMPACT, WIDE, SYMMETRIC | Finisher |
| SHP_034 | Rectangle 3×2 | Rectangle | RECTANGLE | (0,0)(0,1)(1,0)(1,1)(2,0)(2,1) | 6 | 2 | 3 | BIAXIAL | 90° | LARGE, COMPACT, TALL, SYMMETRIC | Finisher |

---

## 13. Change Management

Adding or modifying a Shape entry follows strict validation:
1. Assign the next unused `SHP_0##` ID; never reuse a retired ID.
2. Complete every Section 3 field; derive all Section 6 tags algorithmically.
3. Add RNG draw weight entries in the Spawner configuration.
4. Verify stage objectives that reference `category` or `tags` remain balanced.
5. Add unit test coverage verifying coordinates and bounding box dimensions.

---

## 14. Cross-References

- **Scoring Engine:** Consumes `block_count` for placement score points and line clear bonuses.
- **RNG Spawner:** Consumes the full shape registry for tray generation and solvability protection.
- **Stage Objective Engine:** Consumes shape categories and tags to evaluate objective milestones.
- **Board Engine:** Consumes `local_coordinates`, `width`, and `height` for placement validation and collision checks.

---

**End of `02_SHAPE_LIBRARY.md`.**
