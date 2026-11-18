# 20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md
## Block Battles — Data Model and Content Schema

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`, all Layer A design specs.
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the exact Dart data structures that fulfill the architecture outlined in Doc 18. It enforces the separation between static design data (immutable Dart objects) and mutable runtime data for the two core game modes: **Mode 1 (Classic Endless)** and **Mode 2 (Goal-Driven Stage Challenge)**.

---

## 1. Data Architecture Philosophy

**"Code is the Engine; Data is the Fuel."**
The core gameplay systems must be entirely agnostic to specific stages or shapes. The `BoardEngine` does not hardcode level layouts or special cases; it operates on pure matrix math and grid rules. The `ScoreService` evaluates scoring models deterministically from placement, line clears, and streak multipliers. This ensures AI coding agents and human designers can scale the game by adding hundreds of new stage definitions and shape configurations purely through declarative data definitions.

---

## 2. Runtime Data vs. Authoring Data

There is a strict boundary between what the designer authors and what the game engine manipulates:

- **Authoring Data (Dart Definitions):** Static, read-only at runtime. Authored as `const` or `final` Dart classes.
  - Examples: `ShapeDefinition` (geometry of 34 canonical shapes), `StageDefinition` (Mode 2 level objectives, move limits, obstacle layouts), `ScoringConfig` (base line clear scores, streak thresholds).
- **Runtime Data (State Classes):** Mutable state instantiated at runtime and persisted during sessions.
  - Examples: `BoardState` (64-element grid occupancy and tile status), `TrayState` (3 draggable piece slots), `ClassicSessionState` (current score, streak count, high score), `StageSessionState` (remaining moves, goal progress counters, star rating).

---

## 3. Content Definition Rules

1. **Read-Only at Runtime:** All definition classes must have `final` fields and a `const` constructor. They cannot be mutated during gameplay.
2. **Flat Serialization:** Avoid deeply nested polymorphic serialization for save data. Use simple data types, enums, and explicit DTOs for Hive CE storage.
3. **Self-Validation:** Provide factory constructors or `assert` statements to catch authoring errors immediately at startup or in test suites (e.g., verifying a Shape matrix has exactly 25 elements, or verifying a stage move limit is $\ge 1$).

---

## 4. Shape Schema (`shape_definition.dart`)

Defines the geometry, bounding box, and metadata of a draggable block piece (34 canonical shapes across 15 families and 9 functional categories).

```dart
enum ShapeCategory {
  single,
  line,
  corner,
  square,
  rectangle,
  lFamily,
  tFamily,
  sZFamily,
  cross,
}

enum ShapeFamily {
  single,
  domino,
  trominoLine,
  trominoCorner,
  tetrominoLine,
  tetrominoSquare,
  tetrominoL,
  tetrominoJ,
  tetrominoT,
  tetrominoS,
  tetrominoZ,
  pentominoLine,
  pentominoPlus,
  square3x3,
  rectangle2x3,
}

class ShapeDefinition {
  final String shapeId; // e.g., "shape_square_2x2", "shape_line_1x4"
  final String displayName;
  final ShapeFamily family;
  final ShapeCategory category;
  final int defaultColorIndex; // 0..7 matching Color Palette
  final int rarityWeight;      // RNG selection weight
  
  /// Flat array of 25 booleans representing 5x5 bounding grid.
  /// Indexing: row * 5 + col. True = filled block, False = empty cell.
  final List<bool> blockMatrix; 

  const ShapeDefinition({
    required this.shapeId,
    required this.displayName,
    required this.family,
    required this.category,
    required this.defaultColorIndex,
    required this.rarityWeight,
    required this.blockMatrix,
  }) : assert(blockMatrix.length == 25, 'blockMatrix must be exactly 25 elements (5x5)');

  // Computed properties
  int get blockCount => blockMatrix.where((b) => b).length;

  int get boundingWidth {
    int minCol = 5, maxCol = -1;
    for (int r = 0; r < 5; r++) {
      for (int c = 0; c < 5; c++) {
        if (blockMatrix[r * 5 + c]) {
          if (c < minCol) minCol = c;
          if (c > maxCol) maxCol = c;
        }
      }
    }
    return maxCol >= minCol ? (maxCol - minCol + 1) : 0;
  }

  int get boundingHeight {
    int minRow = 5, maxRow = -1;
    for (int r = 0; r < 5; r++) {
      for (int c = 0; c < 5; c++) {
        if (blockMatrix[r * 5 + c]) {
          if (r < minRow) minRow = r;
          if (r > maxRow) maxRow = r;
        }
      }
    }
    return maxRow >= minRow ? (maxRow - minRow + 1) : 0;
  }
}
```

---

## 5. Tile and Obstacle Schema (`tile_definition.dart`)

Defines the types and states of cells on the 8×8 board.

```dart
enum TileType {
  empty,
  filledBlock,
  stoneObstacle,  // Immovable; cleared by clearing adjacent or intersecting lines
  iceObstacle,    // Requires 2 clears to completely remove
}

class BoardCell {
  final TileType type;
  final int colorIndex; // 0 for empty/neutral, 1..7 for block themes
  final int health;     // Used for multi-hit obstacles (e.g., Ice = 2, 1, 0)

  const BoardCell({
    this.type = TileType.empty,
    this.colorIndex = 0,
    this.health = 1,
  });

  bool get isOccupied => type != TileType.empty;
  bool get canPlaceOver => type == TileType.empty;

  BoardCell copyWith({
    TileType? type,
    int? colorIndex,
    int? health,
  }) {
    return BoardCell(
      type: type ?? this.type,
      colorIndex: colorIndex ?? this.colorIndex,
      health: health ?? this.health,
    );
  }
}
```

---

## 6. Goal & Objective Schema (`stage_goal_definition.dart`)

Defines win conditions for Mode 2 (Goal-Driven Stage Challenges).

```dart
enum GoalType {
  reachTargetScore,
  clearTotalLines,
  clearColorBlocks,
  clearAllObstacles,
}

class StageGoal {
  final GoalType type;
  final int targetValue;    // e.g., 5000 points, 12 lines, 20 Amber blocks
  final int? targetColorIndex; // Required when type == clearColorBlocks

  const StageGoal({
    required this.type,
    required this.targetValue,
    this.targetColorIndex,
  });
}
```

---

## 7. Stage Schema (`stage_definition.dart`)

Defines pre-configured puzzles in Mode 2.

```dart
class StageDefinition {
  final int stageNumber;
  final String stageId;
  final String displayName;
  final int moveLimit; // Maximum placement turns allowed (e.g., 25 moves)
  final List<StageGoal> goals;
  
  // Star scoring thresholds based on remaining moves or bonus score
  final int star1Threshold;
  final int star2Threshold;
  final int star3Threshold;
  
  // Optional pre-filled cells on the 8x8 grid (index: row * 8 + col)
  final Map<int, BoardCell> initialObstacles;

  const StageDefinition({
    required this.stageNumber,
    required this.stageId,
    required this.displayName,
    required this.moveLimit,
    required this.goals,
    required this.star1Threshold,
    required this.star2Threshold,
    required this.star3Threshold,
    this.initialObstacles = const {},
  }) : assert(moveLimit > 0, 'moveLimit must be positive');
}
```

---

## 8. Mode Enumeration (`game_mode.dart`)

Enforces the strict two-mode taxonomy across the entire codebase.

```dart
enum GameMode {
  classicEndless,
  goalStage,
}
```

---

## 9. Persistent Runtime State Classes (`runtime_state.dart`)

Pure Dart state classes manipulated by engines during active gameplay and serialized via Hive CE.

```dart
class BoardState {
  /// 64 board cells (8x8 grid, row-major: row * 8 + col).
  List<BoardCell> cells;

  BoardState({required this.cells}) : assert(cells.length == 64);

  factory BoardState.empty() {
    return BoardState(cells: List.generate(64, (_) => const BoardCell()));
  }
}

class TraySlotState {
  final int slotIndex; // 0, 1, 2
  final ShapeDefinition? shape;
  final bool isPlaced;

  const TraySlotState({
    required this.slotIndex,
    this.shape,
    this.isPlaced = false,
  });
}

class TrayState {
  List<TraySlotState> slots;

  TrayState({required this.slots}) : assert(slots.length == 3);

  bool get isAllPlaced => slots.every((s) => s.isPlaced || s.shape == null);
}

class ClassicSessionState {
  int currentScore;
  int highestScore;
  int streakCount;
  double currentMultiplier; // 1.0x .. 2.5x
  int totalLinesCleared;
  int placementsCount;
  bool isGameOver;

  ClassicSessionState({
    this.currentScore = 0,
    this.highestScore = 0,
    this.streakCount = 0,
    this.currentMultiplier = 1.0,
    this.totalLinesCleared = 0,
    this.placementsCount = 0,
    this.isGameOver = false,
  });
}

class StageProgressState {
  int remainingMoves;
  Map<int, int> goalCurrentValues; // Goal index -> current achieved count
  bool isCompleted;
  bool isFailed;
  int starsEarned; // 0..3

  StageProgressState({
    required this.remainingMoves,
    required this.goalCurrentValues,
    this.isCompleted = false,
    this.isFailed = false,
    this.starsEarned = 0,
  });
}
```

---

## 10. Content Registry (`catalogs`)

Definitions are authored as immutable Dart constants grouped in catalog registries:
- `shape_catalog.dart` (Contains all 34 canonical shape definitions)
- `stage_catalog.dart` (Contains all Mode 2 stage definitions)

```dart
class ShapeCatalog {
  static const Map<String, ShapeDefinition> allShapes = {
    // 34 canonical shapes...
  };

  static ShapeDefinition get(String id) => allShapes[id]!;
}

class StageCatalog {
  static const List<StageDefinition> allStages = [
    // Pre-authored progression stages...
  ];

  static StageDefinition getByNumber(int number) =>
      allStages.firstWhere((s) => s.stageNumber == number);
}
```

---

## 11. Data Loading

Pure Dart constants are the authoritative source of truth:
1. **Instant Availability:** Zero file I/O latency at app startup.
2. **Type-Safe Validation:** Dart compiler guarantees shape matrices and stage parameters are statically sound.
3. **Determinism:** Seeded RNG selections pick directly from statically indexed catalogs.

---

## 12. Validation Rules

Constructors and test suites validate data integrity before runtime execution:
- Shape matrices must have exactly 25 elements ($5 \times 5$).
- Board grid states must have exactly 64 cells ($8 \times 8$).
- Tray states must contain exactly 3 slots.
- Stage move limits must be $\ge 1$.
- Star thresholds must strictly increase: $\text{star1} \le \text{star2} \le \text{star3}$.

---

## 13. IDs and Versioning

- **Unique String IDs:** Content uses snake_case string IDs (e.g., `shape_l_3x2`, `stage_001_intro`).
- **Save State Migration:** Hive boxes store high scores keyed by mode and stage number. Adding new stages or shapes does not invalidate or corrupt existing player progress.

---

## 14. Null/Missing Data Behavior

- **Graceful Fallbacks:** If saved stage progress references an archived stage ID, the state manager logs a warning, skips the obsolete stage record, and loads the current active stage catalog cleanly without crashing.

---

## 15. Designer Authoring Workflow

To empower solo development and automated test generation:
1. **Authoring Stages:** Designers add stages by instantiating `StageDefinition` in `stage_catalog.dart` specifying target scores, line goals, or color block quotas.
2. **Zero Core Logic Edits:** Adding new puzzle levels never requires altering `BoardEngine` or `ScoreService`.

---

## 16. Anti-Pattern Rules

1. **No Live State in Definitions:** Never declare mutable counters in `StageDefinition` or `ShapeDefinition`. Definitions are compile-time constants.
2. **No Hardcoded Stage Branching in Engine:** Never write `if (stageNumber == 5) { ... }` inside `BoardEngine`. The engine only evaluates generic board rules, cell matches, and line clears.
3. **Strict Scope Discipline:** Never declare non-puzzle subsystems, obsolete legacy schemas, or unapproved mechanics in any data schema. The game is purely a block puzzle.

---

## 17. Final Data Contract

> **Data is the absolute truth; Systems are just interpreters.**
>
> In Block Battles, we lock down our core Dart puzzle systems early and scale the game purely by injecting clean, declarative data classes. By maintaining strict discipline over our Definitions, we ensure that adding content is safe, deterministic, and highly maintainable.

---

**End of `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md`.**
