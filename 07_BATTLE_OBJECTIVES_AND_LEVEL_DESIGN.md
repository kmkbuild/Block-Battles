# 07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md
## Block Battles — Mode 2: Goal-Driven Mode & Level Hurdles (Reserved Architecture)

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `03_BOARD_ENGINE_AND_RULES.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`  
**Document status:** Architecture Contract for Mode 2 (Goal-Driven Play)  
**Core Directive:** Clean isolation between Mode 1 (Classic Endless) and Mode 2 (Goal-Driven Hurdles). Pure spatial puzzle challenges.

---

## 1. Purpose and Ownership

This document defines the architecture, data schemas, and evaluation contracts for **Mode 2: Goal-Driven Mode**.
- Mode 1 (Classic Endless) focuses purely on endless survival, combos, and local high score.
- Mode 2 (Goal-Driven Mode) presents discrete, bite-sized puzzle stages with explicit completion hurdles (e.g., reaching a target score, clearing pre-placed obstacles, or surviving within move limits).

**Ownership Boundary:** This document owns level definitions, hurdle criteria, and level evaluation logic. It consumes the core Board Engine (`03`), Shape Catalog (`02`), and Scoring Engine (`05`).

---

## 2. Core Hurdle Types

Mode 2 levels are built by composing one or more of three deterministic hurdle types:

### Hurdle A: Target Score
- The player must accumulate `targetScore` points (e.g., 2,500 pts) before running out of moves or locking the board.
- Evaluated against `currentScore` from `ScoreEngine`.

### Hurdle B: Obstacle Clears (Pre-set Board Cells)
- The level starts with specific cells pre-filled on the 8x8 board:
  - **Frozen Cells:** Occupied blocks that do not clear until adjacent or containing lines are completed.
  - **Blocked Cells:** Empty cells that cannot accept placements until specific conditions are met.
- The goal is to clear all or a required count of obstacle cells.

### Hurdle C: Move Limit
- The player is allocated a fixed number of placements (e.g., 25 moves).
- Every placed piece decrements the remaining move counter by 1.
- If moves reach 0 before all primary hurdles are satisfied, the level ends in failure.

---

## 3. Level Definition Schema

Every level is represented as a pure Dart data model:

```dart
class LevelDefinition {
  final String id;              // e.g. "LEVEL_001"
  final String title;           // e.g. "Frostbite Awakening"
  final int moveLimit;          // Max placements permitted
  final int targetScore;        // Target score required (0 if purely obstacle-based)
  final List<ObstaclePlacement> initialObstacles; // Pre-filled board cells
  final StarCriteria starCriteria; // 1-star, 2-star, 3-star thresholds

  const LevelDefinition({
    required this.id,
    required this.title,
    required this.moveLimit,
    required this.targetScore,
    this.initialObstacles = const [],
    required this.starCriteria,
  });
}
```

---

## 4. Star Rating System

Each level awards up to 3 stars upon completion:
- **1 Star (Clear):** Meet the primary objective (e.g., reach target score or clear target obstacles).
- **2 Stars (Efficiency):** Complete the primary objective with at least 20% of moves remaining.
- **3 Stars (Mastery):** Complete all objectives with at least 40% of moves remaining or achieving a 3+ streak combo during the stage.

---

## 5. Architectural Isolation from Classic Mode

1. **Shared Domain Primitives:**
   - Both modes share `BoardModel`, `ShapeCatalog`, `GameRng`, and `TrayEngine`.
2. **Dedicated Game Services:**
   - Mode 1 runs on `ClassicGameService` (unbounded turns, high-score focus).
   - Mode 2 runs on `GoalDrivenGameService` (tracks moves remaining, hurdles, and star conditions).
3. **Zero Contamination:**
   - Classic Mode has no knowledge of move limits or star ratings.
   - Goal-Driven Mode logic is completely decoupled and can be expanded or activated without modifying Classic Mode.

---

## 6. Acceptance Criteria

- Level definitions are purely declarative data classes.
- Board Engine accepts pre-populated obstacle boards for Mode 2 setup.
- Move counter decrements strictly on valid piece placement.
- Level victory triggers immediately when all hurdle conditions evaluate true.
- Level failure triggers if moves reach 0 without fulfilling hurdles, or if Board Lock occurs.

---

**End of `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`.**
