# 04_SPAWN_RNG_AND_DIFFICULTY.md
## Block Battles — Spawn RNG and Difficulty

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`
**Document status:** Implementation-level system design

**Inheritance confirmation:** This document implements Tray Generation and the solvability safeguards referenced by Master Vision Sections 5.5 and 18, using `02_SHAPE_LIBRARY.md`'s registry and metadata contract and `03_BOARD_ENGINE_AND_RULES.md`'s query/event surface as its only inputs for both **Mode 1 (Classic Endless)** and **Mode 2 (Goal-Driven Stage Challenge)**. It does not redefine Board rules, Shape geometry, or Turn structure — it decides *which* Shapes enter the Tray and *when*.

---

## 1. RNG Philosophy

**DESIGN DECISION:** RNG in Block Battles exists to create *decisions*, never *helplessness* (Master Vision Section 5.5, Controlled Randomness; Section 5.6, Fair Difficulty).

Three non-negotiable philosophy rules:

1. **Solvability first.** A Tray that cannot be placed anywhere on the current Board (Board Lock, `03_BOARD_ENGINE_AND_RULES.md` Section 10) must never be generated when a solvable alternative exists within the same weighting pass (Section 5).
2. **No manufactured loss.** This document must not introduce a mechanic whose sole function is to end a session — every generation rule here either preserves player agency or makes spatial constraints legible in advance. Difficulty rises by *shrinking the margin for error*, never by *removing the possibility of correct play*.
3. **Randomness is diegetic, not adversarial.** The system does not "know" the player is doing well and punish them for it, nor reward them for doing poorly. Any board-awareness (Section 4) or difficulty scaling (Section 10) responds to *board state*, not to a hidden performance judgment of the player.

**MASTER DESIGN RULE (inherited):** Randomness must create decisions, not frustration.

---

## 2. Base Shape Selection

**Inputs:** The full 34-entry Shape Registry (`02_SHAPE_LIBRARY.md` Section 12), grouped by `family`, `category`, and `tags`.

**DESIGN DECISION:** Base selection is a two-stage draw, not a flat uniform pick across all 34 entries:

1. **Family draw:** One of the 15 Shape Families (`02_SHAPE_LIBRARY.md` Section 4) is selected first, using Family Weights (Section 15, Table 15.1).
2. **Orientation draw:** Within the selected Family, one registered orientation entry is selected uniformly at random (e.g., Tetromino L has 4 equally-likely orientations once the L Family itself is chosen).

**Rationale:** Weighting at the Family level keeps the tuning surface compact (15 tunable weights instead of 34) while guaranteeing that orientation is never itself a strategic signal.

---

## 3. Weighted Generation

Family Weights are the single tunable surface for baseline shape frequency. They are informed by `block_count` and `strategic_classification` (Shape Library Section 3, Section 7) as declarative data definitions:

- **Filler-classified** Shapes (`SINGLE`, `LINE` per Shape Library Section 7) are weighted higher — they are low-risk, board-friendly, and support instant clarity.
- **Finisher/High-Risk Finisher** Shapes (large, irregular, e.g. Pentomino Plus, Square 3×3) are weighted lower — they carry higher variance and are meant to be occasional high-reward/high-risk draws.
- **Precision Piece** Shapes (S/Z/T/L/J tetrominoes) sit at a moderate middle weight — these are the primary drivers of spatial tension.

---

## 4. Board-Aware Generation

Generation reads Board state via `03_BOARD_ENGINE_AND_RULES.md` Section 11 Board Queries — specifically `getFilledCellCount()`, `getRowFillCount(row)`, `getColFillCount(col)`, and `hasAnyLegalPlacement(candidateShape)`:

1. **Soft reweighting (always active):** If the Board's occupancy ratio is high (a "tight" Board, $>65\%$ filled), the weight of `SMALL`-tagged Shapes (`02_SHAPE_LIBRARY.md` Section 6) is boosted and `LARGE`-tagged Shapes is reduced, *before* the draw — increasing the odds of a board-friendly Shape without guaranteeing one.
2. **Hard filtering (Solvability Protection only, Section 5):** A candidate Shape is discarded and redrawn *only* if including it would violate the minimum fairness guarantee.

**MASTER DESIGN RULE:** Soft reweighting must never reduce a Shape's weight to zero — a tight Board makes large Shapes *less likely*, not *impossible*.

---

## 5. Solvability Protection

**Minimum fairness guarantee:** At the moment a full 3-Shape Tray batch is generated (`01_GAMEPLAY_SPECIFICATION.md` Section 6), **at least one of the 3 Shapes must have at least one legal Placement on the current Board**, verified via `03_BOARD_ENGINE_AND_RULES.md` Section 10's `hasAnyLegalPlacement` logic.

### Algorithm — `generateTray(board)`:

```text
1. Draw 3 candidate Shapes via Section 2–4 (family draw → orientation draw → board-aware reweight).
2. For each of the 3 candidates, check hasAnyLegalPlacement([candidate], board) individually.
3. If at least 1 of 3 candidates has a legal placement: accept the batch as-is. STOP.
4. If none of the 3 candidates has a legal placement:
   a. Discard the lowest-weighted candidate.
   b. Redraw a replacement candidate from the weighted pool.
   c. Re-check hasAnyLegalPlacement for the new candidate.
   d. Repeat up to 5 redraw attempts.
5. Fallback: deterministically substitute SHP_001 (Single Block) as the final candidate, guaranteed placeable on any non-full board.
```

**DESIGN DECISION:** This guarantees the *Tray* is not dead on arrival. It does not guarantee the player can survive indefinitely; later Board Lock remains an earned outcome resulting from the player's own prior placements.

---

## 6. Pity / Anti-Frustration Mechanics

Block Battles uses **Consecutive Large-Shape Suppression**:

- **Trigger:** If the previous 2 consecutive Tray batches (6 Shapes total) each contained at least one `LARGE`-tagged Shape (`block_count >= 5`), the next batch suppresses `LARGE`-tagged weight by $50\%$ for that single batch.
- **Reset:** The suppression counter resets to zero the instant a batch is generated with zero `LARGE`-tagged Shapes.
- **Rationale:** Prevents back-to-back runs of unwieldy pieces from overwhelming the player through pure variance.

---

## 7. Diverse-Tray Bias

A single 3-Shape Tray batch should offer variety:
- The spawner applies a mild negative weight modifier ($\times 0.25$) to any Shape Family already selected for slot 0 or slot 1 when drawing slot 2.
- This prevents duplicate identical pieces from clogging the tray (e.g., drawing three identical 3×3 squares at once).

---

## 8. Stage-Objective-Aware Generation (Mode 2)

In Mode 2, generation may gently align with active stage goals:
- **Soft nudge:** If a Stage objective requires clearing specific color blocks or lines, generation provides a minor ($+15\%$) weight nudge toward relevant shapes (e.g. `LINE` shapes for line-clear stages).
- **Anti-tamper rule:** Generation NEVER guarantees specific shapes to auto-solve a level. Objectives must be earned by player placement skill.

---

## 9. Difficulty Modeling

Difficulty is measured through four objective runtime signals:

| Signal | Definition | Data Source |
|---|---|---|
| **Board Occupancy** | `getFilledCellCount() / 64` (fraction of board filled) | `03_BOARD_ENGINE_AND_RULES.md` |
| **Legal Move Count** | Count of valid coordinates across the 3 tray pieces | `BoardEngine.countValidMoves()` |
| **Piece Footprint Mix** | Average `block_count` of current tray pieces | `TrayState` |
| **Objective Pressure** | Remaining moves vs. required goal progress (Mode 2) | `StageProgressState` |

---

## 10. Difficulty Scaling by Mode

### Mode 1: Classic Endless Scaling
- As the player's score increases ($0 \to 1,000 \to 5,000 \to 10,000+$):
  - Base weights gradually shift from high `Filler` frequency toward balanced `Precision Piece` and `Finisher` frequency.
  - Board-aware soft reweighting thresholds tighten, requiring more precise board maintenance.

### Mode 2: Stage Progression Scaling
- Stage difficulty is declared authoritatively in `StageDefinition`:
  - Early stages (Stages 1–10): Generous move limits, clear objectives, minimal initial obstacles.
  - Mid stages (Stages 11–25): Tighter move limits, multi-objective combinations, single-layer stone obstacles.
  - Advanced stages (Stages 26+): Strict move budgets, multi-hit ice obstacles, pre-filled board layouts.

---

## 11. Deterministic Pseudo-Randomness (Mulberry32)

Every session utilizes a single deterministic seed:
- **Seed Initialization:** Generated at session start from system entropy or user-specified debug seed.
- **Deterministic Stream:** All draws (Family, Orientation, Solvability checks) consume from this single continuous stream.
- **Mid-Session Resumability:** Saving `rngCallsConsumed` allows exact state restoration upon resuming an active session.

---

## 12. Automated Simulation & Balancing

Balancing is validated via headless automated simulation:
1. **Random-Legal Agent:** Makes random legal placements to establish the baseline fairness floor.
2. **Greedy-Clear Agent:** Prioritizes moves that complete lines or maximize open cells.
3. **Lookahead Agent:** Evaluates 2-turn placement consequences, modeling a skilled human player.

A balanced design demonstrates:
- 0% unwinnable first-tray scenarios across 100,000 simulated seeds.
- Clear performance separation between random play and strategic play.

---

## 13. Telemetry and Analytics Metrics

Key metrics tracked during testing:
- **Average Session Length:** Placements and lines cleared per session.
- **Board Lock Distribution:** Average board occupancy at the moment of game-over.
- **Streak Frequency:** Percentage of turns maintaining active streak multipliers ($1.2\times \dots 2.5\times$).
- **Stage Completion Rate:** Percentage of attempts achieving 1, 2, or 3 stars per Stage.

---

## 14. Anti-Frustration Guarantees

1. **Zero Unplaceable Trays:** Solvability protection guarantees at least 1 placeable piece per fresh batch.
2. **No Triple Massive Piece Trays:** Diverse-tray bias prevents three high-footprint pieces simultaneously.
3. **Transparent Rules:** RNG never actively sabatoges the player; board tension is purely the result of prior placements.

---

## 15. Tuning Tables

### Table 15.1: Baseline Family Weights

| Family | Base Weight | Category |
|---|---|---|
| Single | 12 | SINGLE |
| Domino | 14 | LINE |
| Tromino Line | 12 | LINE |
| Tromino Corner | 12 | CORNER |
| Tetromino Line | 10 | LINE |
| Tetromino Square | 10 | SQUARE |
| Tetromino L | 9 | L_FAMILY |
| Tetromino J | 9 | L_FAMILY |
| Tetromino T | 8 | T_FAMILY |
| Tetromino S | 7 | S_Z_FAMILY |
| Tetromino Z | 7 | S_Z_FAMILY |
| Pentomino Line | 6 | LINE |
| Pentomino Plus | 4 | CROSS |
| Square 3×3 | 4 | SQUARE |
| Rectangle 2×3 | 5 | RECTANGLE |

---

## 16. Final RNG Contract

> **The dealer is fair; the puzzle is real.**
>
> Block Battles RNG creates captivating spatial puzzles with guaranteed solvability. By enforcing deterministic seeding, board-aware soft reweighting, and strict anti-frustration rules, the spawner ensures every high score and every stage clear is a genuine triumph of player skill.

---

**End of `04_SPAWN_RNG_AND_DIFFICULTY.md`.**
