# 02_SHAPE_LIBRARY.md
## Block Battles — Shape Library

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`
**Document status:** Implementation-level content registry

---

## 1. Purpose

This document owns every logical playable shape (Piece) in Block Battles: its exact geometry, its metadata, and the rules for validating and referencing it. `01_GAMEPLAY_SPECIFICATION.md` Section 5 establishes that Pieces are placed **without rotation**; this document is therefore also the authority on how rotation is compensated for by registering each needed orientation as its own distinct Shape entry.

Shapes are treated as data objects, not pure geometry: their metadata (category, tags, strategic classification) is the interface that Relics, Objectives, and RNG systems use to reference groups of shapes without hardcoding individual geometry. This document owns that metadata contract.

**Does not own:** RNG draw weighting (RNG doc), Relic effect definitions (Relics doc), Objective predicate implementations (Objectives doc), or visual/animation treatment (Art/VFX docs) — see Section 14.

---

## 2. Coordinate System

- **Origin:** Each Shape's local coordinate origin is the top-left cell of its bounding box, `(row, col) = (0, 0)`.
- **Axes:** `row` increases **downward**; `col` increases **rightward**. This matches the Board coordinate convention defined in `01_GAMEPLAY_SPECIFICATION.md` Section 5.
- **Normalization:** Every Shape's local coordinate list is stored normalized such that `min(row) == 0` and `min(col) == 0` across all its cells. No shape may be stored with negative or non-minimal offsets.
- **Bounding box:** `width = max(col) + 1`, `height = max(row) + 1`, computed from the normalized coordinate list. The bounding box is the minimal rectangle containing all of a Shape's cells; it is not required to be fully filled (see Section 6, IRREGULAR).
- **Board placement:** At placement time, a Shape's origin is mapped to a target Board cell `(boardRow, boardCol)`; every local cell `(r, c)` occupies Board cell `(boardRow + r, boardCol + c)`, per the legality rule in `01_GAMEPLAY_SPECIFICATION.md` Section 5.

---

## 3. Canonical Shape Schema

Every registered Shape entry contains exactly these fields:

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique identifier, format `SHP_0##`. Immutable once shipped (Section 13). |
| `display_name` | string | Player-facing name (not currently shown in UI per Master Vision Section 5.1, but reserved for tooltips/accessibility text). |
| `internal_name` | string | Developer-facing snake_case name used in code/config. |
| `category` | enum | Gameplay/relic/objective/RNG grouping — see Section 5. |
| `family` | string | Geometric family this orientation belongs to (e.g., `TETROMINO_L`) — links sibling orientations together. |
| `local_coordinates` | list of (row, col) | Normalized cell list per Section 2. |
| `block_count` | integer | `len(local_coordinates)`. |
| `width` | integer | Bounding box width per Section 2. |
| `height` | integer | Bounding box height per Section 2. |
| `symmetry` | enum | `NONE`, `MIRROR`, `POINT`, `AXIAL`, `BIAXIAL`, `FULL` — see Section 4 notes. |
| `orientation` | string | Which rotation of the family's canonical base this entry represents (`Base`, `90°`, `180°`, `270°`, `Horizontal`, `Vertical`, or `N/A (symmetric)`). Never used for runtime rotation — purely a content/dedup label (Section 13). |
| `tags` | list of enum | Reusable metadata tags — see Section 6. |
| `strategic_classification` | enum | Gameplay-role label — see Section 7. |

---

## 4. Shape Registry

**DESIGN DECISION:** The collection below is **OUR DESIGN SET** — an original registry designed for Block Battles' needs. It is loosely informed by common conventions observed across block-placement puzzle games in general (REFERENCE OBSERVATION: small polyomino sets of this general shape are common in the genre), but no specific count, list, or proprietary detail from any existing title is asserted or reproduced here.

Because Pieces cannot be rotated at runtime (`01_GAMEPLAY_SPECIFICATION.md` Section 5), each geometric **family** below is registered as multiple **orientation entries** — one per rotation actually needed for solvable play. The registry therefore has two useful counts:

- **15 Shape Families** (distinct geometric identities) — sits within the "low double-digit" polyomino guidance of `00_MASTER_GAME_VISION.md` Section 14.
- **34 registered Shape entries** (families × required orientations) — the actual implementable pool, per Section 13's Change Management note reconciling this with the Master Vision.

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

Each family's base geometry, illustrated at its first registered orientation:

```
Single           Domino H         Tromino Line H   Tromino Corner A
X                XX               XXX              X.
                                                    XX

Tetromino Line H Tetromino Sq O   Tetromino L (Base) Tetromino J (Base)
XXXX             XX               X.                 .X
                  XX               X.                 .X
                                    XX                 XX

Tetromino T (Base) Tetromino S H   Tetromino Z H    Pentomino Line H
XXX                 .XX             XX.              XXXXX
.X.                 XX.             .XX

Pentomino Plus   Square 3×3       Rectangle 2×3
.X.               XXX              XXX
XXX               XXX              XXX
.X.               XXX
```

Full per-orientation coordinates are given in the master table (Section 12).

---

## 5. Shape Categories

The `category` field groups Shapes for gameplay balance, Relic targeting, Objective predicates, and RNG weighting — a coarser grouping than `family` (which only distinguishes geometry), and orthogonal to `tags` (which describe properties, not identity).

| Category | Includes Families | Primary Use |
|---|---|---|
| `SINGLE` | Single | Relic/Objective targeting of the minimal piece; RNG floor-filler weighting. |
| `LINE` | Domino, Tromino Line, Tetromino Line, Pentomino Line | Direct line-clear enablers; Relic bonuses for "clear using a LINE piece." |
| `CORNER` | Tromino Corner | Low-risk small filler with irregular footprint; early-game RNG staple. |
| `SQUARE` | Tetromino Square (O), Square 3×3 | Dense, gap-free filler; Relic/Objective targeting of compact placement. |
| `RECTANGLE` | Rectangle 2×3 | Large, dense, non-square filler; distinct from `SQUARE` for balance/RNG purposes. |
| `L_FAMILY` | Tetromino L, Tetromino J | Irregular, high-tactical-tension pieces; Relic/Objective targeting of "L-shaped" pieces without caring about handedness. |
| `T_FAMILY` | Tetromino T | Irregular, single-axis-symmetric pieces; distinct balance identity from L/J and S/Z. |
| `S_Z_FAMILY` | Tetromino S, Tetromino Z | Irregular, point-symmetric pieces; the genre's classically hardest-to-place tetrominoes. |
| `CROSS` | Pentomino Plus | Unique high-footprint, fully-symmetric piece; Relic/Objective and RNG special-cased due to rarity and risk. |

---

## 6. Shape Tags

Tags are derived, boolean, non-exclusive metadata computed from a Shape's own fields (never authored independently), so they can never drift out of sync with the geometry. All thresholds below are fixed rules, applied identically to every current and future Shape.

| Tag | Rule | Purpose |
|---|---|---|
| `SINGLE` | `block_count == 1` | Relic targeting of the minimal piece. |
| `SMALL` | `block_count <= 3` | Relic/Objective targeting of low-commitment pieces. |
| `LARGE` | `block_count >= 5` | Relic/Objective targeting of high-risk, high-reward pieces (Master Vision Section 6, Combat Relevance example). |
| `LINE` | `(width == 1 or height == 1) and block_count > 1` | Direct line-clear-enabler pieces (excludes the 1×1 Single, which is trivially both dimensions 1). |
| `WIDE` | `width > height` | Board-shape planning; Relic bonuses for horizontal-clear-oriented builds. |
| `TALL` | `height > width` | Board-shape planning; Relic bonuses for vertical-clear-oriented builds. |
| `COMPACT` | `width * height == block_count` and `width > 1` and `height > 1` | Fully gap-free, non-line rectangular pieces — dense, flexible fillers. |
| `SQUARE` | `COMPACT` and `width == height` | Fully gap-free square pieces specifically. |
| `IRREGULAR` | `width * height != block_count` | Pieces that do not fill their own bounding box — the tactically hardest pieces to place cleanly. |
| `CROSS` | `family == "Pentomino Plus"` | Single-family tag reserved for the unique cross shape. |
| `SYMMETRIC` | `symmetry != NONE` | Relic/Objective targeting of pieces with any reflective/rotational symmetry. |

**MASTER DESIGN RULE (inherited from Master Vision Section 23):** No tag is added to this table unless a concrete Relic, Objective, or RNG use case (Sections 8–9) motivates it. A tag with no consumer is removed at the next Change Management pass (Section 13).

---

## 7. Shape Gameplay Role

Documented per **family** (orientation siblings share identical gameplay role; only their footprint direction differs).

| Family | Strengths | Weaknesses | Strategic Opportunities | Dangerous Board States |
|---|---|---|---|---|
| Single | Fits anywhere; zero placement risk | Contributes least toward any single line; inefficient use of a Tray slot if overused | Precision gap-filling to complete an otherwise-stalled line | Rarely dangerous on its own; risk is opportunity cost |
| Domino | Very flexible; easy to slot into small gaps | Minimal Damage contribution per placement | Cheap, reliable partial-line progress | Low risk; can be a "wasted" placement late in a Battle |
| Tromino Line | Direct progress toward a 3+ cell line segment | Requires 3 contiguous open cells in one direction | Reliable mid-size line contribution | Boards with heavy fragmentation (no 3-in-a-row open run) |
| Tromino Corner | Fills irregular gaps other pieces cannot | Leaves an asymmetric footprint that can fragment remaining space | Cleaning up single-cell-offset gaps left by earlier placements | Repeated corner placement in the same region can create isolated unreachable cells |
| Tetromino Line | Strong single-placement line-completion tool | Requires a full 4-cell contiguous open run | Efficient, high-value line clears | Boards with no open 4-run in either axis make this piece "dead" (Section 21 of Gameplay Spec) |
| Tetromino Square (O) | Dense, symmetric, easy to place against existing filled blocks | Commits a full 2×2 area at once | Reliable filler that never creates awkward internal gaps | Board regions smaller than 2×2 cannot accept it at all |
| Tetromino L / J | Enables completing two perpendicular partial lines with one placement | Leaves an asymmetric notch that can be hard to fill later | High tactical value for corner/edge cleanup and dual-line setups | Placed carelessly, creates a single-cell notch reachable only by Single/Corner pieces |
| Tetromino T | Reasonably flexible with one axis of symmetry | Central stem can block adjacent placements | Good for centering within an open plus-shaped gap | Leaves two small side gaps if not paired with a complementary piece |
| Tetromino S / Z | Genuinely irregular; forces active spatial planning | Hardest tetrominoes to place without creating stray gaps | Rewards players who plan two placements ahead (Master Vision Section 5.2) | Placed against a flat wall, commonly creates unreachable single-cell pockets |
| Pentomino Line | Very high single-placement Damage potential (Section 8) | Requires a full 5-cell contiguous open run — rare on a cluttered board | Late-Battle finishing tool when a long run is available | Frequently "dead" on a moderately filled board; a high-risk Tray draw |
| Pentomino Plus (Cross) | Simultaneously touches 5 different rows/columns in a compact footprint | Requires open space on all 4 sides of a center cell plus the center | Unique multi-line setup potential in open board regions | Extremely hard to place on a cluttered board; can force a near-Board-Lock if drawn late |
| Square 3×3 | Very high single-placement Damage potential; simple to reason about | Requires a full 3×3 open area | Strong early-Battle placement while the board is mostly empty | Late-Battle draw with no 3×3 open area is effectively a dead Tray slot |
| Rectangle 2×3 | High Damage potential with a less restrictive footprint than the 3×3 square | Still requires a sizable contiguous open area | Reliable large-clear tool when board space allows | Contributes to Board Lock risk if drawn when only fragmented space remains |

---

## 8. Combat Relevance

`01_GAMEPLAY_SPECIFICATION.md` Section 9 (Damage Pipeline) allows Relic Modifiers to be conditional. This document's `category` and `tags` fields are the only vocabulary a Relic definition should use to express such a condition — a Relic must never hardcode a specific Shape `id`.

**Examples of valid Relic-condition phrasing enabled by this schema:**
- *"Deal +30% Damage when the clearing Placement used a `SINGLE`-tagged piece."* (references Section 6 tag)
- *"Deal +1 flat Damage per block for any Placement using a `LARGE`-tagged piece (block_count ≥ 5)."* (references Section 6 tag)
- *"Gain a bonus effect when a Placement uses a piece from the `S_Z_FAMILY` category."* (references Section 5 category)
- *"Whenever a `SYMMETRIC`-tagged piece completes a Combo (`01_GAMEPLAY_SPECIFICATION.md` Section 8), apply an additional Combo Modifier."* (references Section 6 tag, chains into the existing pipeline stage)

**MASTER DESIGN RULE (inherited):** A Relic effect that cannot be expressed purely in terms of `category`, `tags`, `block_count`, `width`, or `height` is not implementable against the current schema and requires either a schema revision here or a redesign of the Relic.

---

## 9. Objective Relevance

`01_GAMEPLAY_SPECIFICATION.md` Section 12 defines the "Clear Lines Using a Particular Behavior" Objective category as referencing "an additional structural predicate" owned by this document's metadata. Objectives must express that predicate against `category`/`tags`, never against a specific Shape `id`, so that new Shapes added later (Section 13) automatically qualify without an Objective rewrite.

**Examples:**
- *"Clear a line using a `LINE`-category piece."*
- *"Win the encounter without placing any `IRREGULAR`-tagged piece."*
- *"Achieve a Combo (`01_GAMEPLAY_SPECIFICATION.md` Section 8) where every piece placed that Turn is `COMPACT`."*

---

## 10. Shape Validation

Every registered Shape entry must satisfy all of the following before it is considered valid content:

1. `local_coordinates` contains no duplicate `(row, col)` pairs.
2. `local_coordinates` is normalized per Section 2 (`min(row) == 0`, `min(col) == 0`).
3. The cell set is **orthogonally connected** — every cell must be reachable from every other cell via a sequence of edge-adjacent (not diagonal-only) cells within the set.
4. `block_count == len(local_coordinates)` exactly.
5. `width` and `height` are exactly `max(col) + 1` and `max(row) + 1` respectively — no declared bounding box may be larger or smaller than the coordinates imply.
6. At least one cell exists in `row == 0`, one in `col == 0`, one in `row == height - 1`, and one in `col == width - 1` (the bounding box has no fully-empty border row/column, which normalization should already guarantee if computed correctly — this is a redundant integrity check).
7. `width <= 8` and `height <= 8` (a Shape must be able to fit somewhere on an empty 8×8 Board per `01_GAMEPLAY_SPECIFICATION.md` Section 5).
8. `id` is globally unique and, once shipped, is never reassigned to different geometry (Section 13).
9. `symmetry`, `tags`, and derived-tag values (Section 6) are recomputed from geometry, never hand-authored inconsistently with the rules in Sections 3–6.
10. Every `family` has at least one registered orientation whose `orientation` label is `Base` (or, for symmetric single-entry families, `N/A (symmetric)`).

---

## 11. Shape Test Cases

Representative expected-behavior scenarios for implementation testing, referencing an empty 8×8 Board unless stated otherwise.

| Test | Shape | Board State | Target Origin | Expected Result |
|---|---|---|---|---|
| T1 | `SHP_001` (Single) | Empty | `(0,0)` | Legal — placed at `(0,0)` only. |
| T2 | `SHP_010` (Tetromino Line H) | Empty | `(7,5)` | Illegal — cells `(7,5)-(7,8)` exceed Board bounds (`col` max index 7). |
| T3 | `SHP_012` (Tetromino Square O) | Empty | `(0,7)` | Illegal — requires columns 7 and 8; column 8 does not exist. |
| T4 | `SHP_013` (Tetromino L, Base) | Cells `(2,1)` already Filled | `(0,1)` | Illegal — local cell `(2,0)` maps to `(2,1)`, which is occupied. |
| T5 | `SHP_031` (Pentomino Plus) | Empty | `(3,3)` | Legal — occupies `(3,4),(4,3),(4,4),(4,5),(5,4)`. |
| T6 | `SHP_032` (Square 3×3) | Row 6 fully Filled | `(5,0)` | Illegal — local cells `(1,0)-(1,2)` map to `(6,0)-(6,2)`, which are occupied; the validator must reject on the first occupied target cell found rather than only checking the origin cell. |
| T7 | `SHP_025` (Tetromino S, Horizontal) | Empty | `(0,0)` | Legal — occupies `(0,1),(0,2),(1,0),(1,1)`; note local coordinates are not normalized to touch `(0,0)` itself, confirming placement uses the full local cell set, not just the origin cell. |
| T8 | Any Tray with 3 pieces, none of which has any legal placement anywhere on the current Board | — | — | Board Lock condition met per `01_GAMEPLAY_SPECIFICATION.md` Section 15 — must trigger `RUN_DEFEAT`. |

---

## 12. Shape Registry Table

Canonical master table. `local_coordinates` are listed as `(row,col)` pairs in the order shown; implementations may store them in any internal order as long as the set is identical.

| ID | Display Name | Family | Category | Local Coordinates | Count | W | H | Symmetry | Orientation | Tags | Strategic Class |
|---|---|---|---|---|---|---|---|---|---|---|---|
| SHP_001 | Single Block | Single | SINGLE | (0,0) | 1 | 1 | 1 | FULL | N/A (symmetric) | SINGLE, SMALL, SYMMETRIC | Filler |
| SHP_002 | Domino (H) | Domino | LINE | (0,0)(0,1) | 2 | 2 | 1 | AXIAL | Horizontal | SMALL, LINE, WIDE, SYMMETRIC | Line Enabler |
| SHP_003 | Domino (V) | Domino | LINE | (0,0)(1,0) | 2 | 1 | 2 | AXIAL | Vertical | SMALL, LINE, TALL, SYMMETRIC | Line Enabler |
| SHP_004 | Tromino Line (H) | Tromino Line | LINE | (0,0)(0,1)(0,2) | 3 | 3 | 1 | AXIAL | Horizontal | SMALL, LINE, WIDE, SYMMETRIC | Line Enabler |
| SHP_005 | Tromino Line (V) | Tromino Line | LINE | (0,0)(1,0)(2,0) | 3 | 1 | 3 | AXIAL | Vertical | SMALL, LINE, TALL, SYMMETRIC | Line Enabler |
| SHP_006 | Tromino Corner A | Tromino Corner | CORNER | (0,0)(1,0)(1,1) | 3 | 2 | 2 | NONE | Base | SMALL, IRREGULAR | Setup Piece |
| SHP_007 | Tromino Corner B | Tromino Corner | CORNER | (0,1)(1,0)(1,1) | 3 | 2 | 2 | NONE | 90° | SMALL, IRREGULAR | Setup Piece |
| SHP_008 | Tromino Corner C | Tromino Corner | CORNER | (0,0)(0,1)(1,0) | 3 | 2 | 2 | NONE | 180° | SMALL, IRREGULAR | Setup Piece |
| SHP_009 | Tromino Corner D | Tromino Corner | CORNER | (0,0)(0,1)(1,1) | 3 | 2 | 2 | NONE | 270° | SMALL, IRREGULAR | Setup Piece |
| SHP_010 | Tetromino Line (H) | Tetromino Line | LINE | (0,0)(0,1)(0,2)(0,3) | 4 | 4 | 1 | AXIAL | Horizontal | LINE, WIDE, SYMMETRIC | Line Enabler |
| SHP_011 | Tetromino Line (V) | Tetromino Line | LINE | (0,0)(1,0)(2,0)(3,0) | 4 | 1 | 4 | AXIAL | Vertical | LINE, TALL, SYMMETRIC | Line Enabler |
| SHP_012 | Tetromino Square (O) | Tetromino Square | SQUARE | (0,0)(0,1)(1,0)(1,1) | 4 | 2 | 2 | FULL | N/A (symmetric) | SQUARE, COMPACT, SYMMETRIC | Flexible Filler |
| SHP_013 | Tetromino L — A | Tetromino L | L_FAMILY | (0,0)(1,0)(2,0)(2,1) | 4 | 2 | 3 | NONE | Base | TALL, IRREGULAR | Precision Piece |
| SHP_014 | Tetromino L — B | Tetromino L | L_FAMILY | (0,0)(0,1)(0,2)(1,0) | 4 | 3 | 2 | NONE | 90° | WIDE, IRREGULAR | Precision Piece |
| SHP_015 | Tetromino L — C | Tetromino L | L_FAMILY | (0,0)(0,1)(1,1)(2,1) | 4 | 2 | 3 | NONE | 180° | TALL, IRREGULAR | Precision Piece |
| SHP_016 | Tetromino L — D | Tetromino L | L_FAMILY | (0,2)(1,0)(1,1)(1,2) | 4 | 3 | 2 | NONE | 270° | WIDE, IRREGULAR | Precision Piece |
| SHP_017 | Tetromino J — A | Tetromino J | L_FAMILY | (0,1)(1,1)(2,0)(2,1) | 4 | 2 | 3 | NONE | Base | TALL, IRREGULAR | Precision Piece |
| SHP_018 | Tetromino J — B | Tetromino J | L_FAMILY | (0,0)(1,0)(1,1)(1,2) | 4 | 3 | 2 | NONE | 90° | WIDE, IRREGULAR | Precision Piece |
| SHP_019 | Tetromino J — C | Tetromino J | L_FAMILY | (0,0)(0,1)(1,0)(2,0) | 4 | 2 | 3 | NONE | 180° | TALL, IRREGULAR | Precision Piece |
| SHP_020 | Tetromino J — D | Tetromino J | L_FAMILY | (0,0)(0,1)(0,2)(1,2) | 4 | 3 | 2 | NONE | 270° | WIDE, IRREGULAR | Precision Piece |
| SHP_021 | Tetromino T — A | Tetromino T | T_FAMILY | (0,0)(0,1)(0,2)(1,1) | 4 | 3 | 2 | MIRROR | Base | WIDE, IRREGULAR, SYMMETRIC | Precision Piece |
| SHP_022 | Tetromino T — B | Tetromino T | T_FAMILY | (0,1)(1,0)(1,1)(2,1) | 4 | 2 | 3 | MIRROR | 90° | TALL, IRREGULAR, SYMMETRIC | Precision Piece |
| SHP_023 | Tetromino T — C | Tetromino T | T_FAMILY | (0,1)(1,0)(1,1)(1,2) | 4 | 3 | 2 | MIRROR | 180° | WIDE, IRREGULAR, SYMMETRIC | Precision Piece |
| SHP_024 | Tetromino T — D | Tetromino T | T_FAMILY | (0,0)(1,0)(1,1)(2,0) | 4 | 2 | 3 | MIRROR | 270° | TALL, IRREGULAR, SYMMETRIC | Precision Piece |
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

Adding, removing, or modifying a Shape entry has effects across every dependent system. The following checklist is mandatory for any registry change:

**Adding a Shape:**
1. Assign the next unused `SHP_0##` ID; never reuse a retired ID (Section 10, rule 8).
2. Complete every Section 3 field; recompute all Section 6 tags algorithmically, never by hand.
3. Add RNG draw weight entries (owned by the RNG doc) — an unweighted Shape must not silently enter the draw pool with an undefined weight.
4. Audit existing Relic definitions (Section 8) for `category`/`tags` conditions the new Shape now satisfies — confirm this is balance-intended, not an accidental side effect.
5. Audit existing Objective predicates (Section 9) for the same reason.
6. Add at least one Section 11-style test case for the new Shape.
7. Re-run/extend solvability validation (owned by the RNG doc) to confirm the new Shape does not break guaranteed-solvable Tray generation.

**Removing a Shape:**
1. Confirm no Relic or Objective references the specific `id` directly (Sections 8–9 mandate category/tag references specifically to make this safe, but hardcoded exceptions must still be searched for).
2. Redistribute its RNG weight among remaining pool members (owned by the RNG doc).
3. Remove its dedicated test case(s) or repurpose them.
4. Retire its `id` permanently — do not reassign (Section 10, rule 8).

**Modifying a Shape's geometry:**
- **Not permitted** on a shipped `id`. A geometry change is always modeled as removing the old entry and adding a new one with a new `id`, so that historical references (analytics, saved Run state per `01_GAMEPLAY_SPECIFICATION.md` Section 20) remain valid.

---

## 14. Cross-References

- **Combat / Scoring doc:** Consumes `category`, `tags`, `block_count`, `width`, `height` for Relic Modifier conditions in the Damage Pipeline (`01_GAMEPLAY_SPECIFICATION.md` Section 9); must not reference a Shape `id` directly (Section 8 of this document).
- **RNG doc:** Owns per-Shape draw weighting for Tray generation (`01_GAMEPLAY_SPECIFICATION.md` Section 6) and solvability safeguards; consumes this document's full registry as its input domain.
- **Objective System doc:** Owns the predicate library referenced generically in `01_GAMEPLAY_SPECIFICATION.md` Section 12; predicates referencing shape properties must use this document's `category`/`tags` vocabulary exclusively (Section 9 of this document).
- **Board Engine doc:** Consumes `local_coordinates`, `width`, `height` for placement legality checks per Section 2 of this document and `01_GAMEPLAY_SPECIFICATION.md` Section 5.

This document must not be edited to embed content owned by the above; it exposes the shape data and metadata contract they consume.
