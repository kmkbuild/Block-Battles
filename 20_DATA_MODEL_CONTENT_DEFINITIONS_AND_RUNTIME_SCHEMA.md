# 20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md
## Block Battles — Data Model and Content Schema

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`, all Layer A design specs.
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the exact Dart data structures that fulfill the architecture outlined in Doc 18. It enforces the separation between static design data (immutable Dart objects) and mutable runtime data.

---

## 1. Data Architecture Philosophy

**"Code is the Engine; Data is the Fuel."**
The core gameplay systems must be entirely agnostic to the content they are processing. The `CombatEngine` does not know what an "Iron Mallet" is; it only knows how to process a `RelicDefinition` object that provides a flat damage modifier on a specific trigger. This ensures AI coding agents can add thousands of new items purely by creating new definition constants, without ever rewriting core logic.

---

## 2. Runtime Data vs. Authoring Data

There is a strict boundary between what the designer authors and what the game manipulates:

- **Authoring Data (Dart Definitions):** Static, read-only at runtime. Authored as `const` or `final` classes. Example: `EnemyDefinition` (contains `maxHp = 100`). Replaces Unity ScriptableObjects.
- **Runtime Data (State Classes):** Mutable state instantiated at runtime. Example: `EnemyRuntimeState` (contains `currentHp = 75`, initialized from `EnemyDefinition.maxHp`).

---

## 3. Content Definition Rules

1. **Read-Only at Runtime:** All definition classes must have `final` fields and a `const` constructor. They cannot be mutated.
2. **Flat Serialization:** Avoid deeply nested polymorphic serialization for save data. Use Enums + Parameter classes for composition.
3. **Self-Validation:** Provide factory constructors or `assert` statements to catch designer errors immediately (e.g., ensuring a Shape matrix has exactly 25 elements).

---

## 4. Shape Schema (`shape_definition.dart`)

Defines the geometry of a draggable piece.

```dart
class ShapeDefinition {
  final String shapeId; // e.g., "shape_T_3x3"
  final String displayName;
  final String family;
  final String category;
  final String orientation;
  final List<String> tags;
  
  /// Flat array of 25 bools. True = block, False = empty.
  /// Indexing: row * 5 + col. Bottom-Left is 0,0.
  final List<bool> blockMatrix; 
  
  const ShapeDefinition({
    required this.shapeId,
    required this.displayName,
    required this.family,
    required this.category,
    required this.orientation,
    required this.tags,
    required this.blockMatrix,
  }) : assert(blockMatrix.length == 25, 'blockMatrix must be exactly 25 elements');

  // Computed properties
  int get blockCount => blockMatrix.where((b) => b).length;
  int get width { /* calculate from matrix */ }
  int get height { /* calculate from matrix */ }

  List<List<bool>> toShapeMatrix() {
    // Converts flat 25 array to a 2D matrix bounding box
  }
}
```

---

## 5. Enemy Schema (`enemy_definition.dart`)

Defines an enemy encounter and its behavior pattern.

```dart
class EnemyDefinition {
  final String enemyId;
  final String nameKey; // Localization key
  
  // Stats
  final int hp;
  
  // Behavior Sequence
  // Executes in order, looping when reaching the end.
  final List<EnemyAction> behaviorList; 
  final List<PhaseThreshold>? phaseThresholds;
  
  // Presentation
  final String portrait; // Asset path
  
  const EnemyDefinition({
    required this.enemyId,
    required this.nameKey,
    required this.hp,
    required this.behaviorList,
    required this.portrait,
    this.phaseThresholds,
  });
}

class EnemyAction {
  final EnemyActionType type; // e.g., Attack, Defend, FreezeBoard
  final int powerValue;       // e.g., Damage amount, or number of cells to freeze
  final int telegraphTurns;   // Number of turns before execution
  
  const EnemyAction({
    required this.type,
    required this.powerValue,
    required this.telegraphTurns,
  });
}
```

---

## 6. Relic Schema (`relic_definition.dart`)

Defines a passive modifier. Uses the Trigger/Effect architecture required by Doc 05.

```dart
class RelicDefinition {
  final String relicId;
  final String nameKey;
  final String descriptionKey;
  
  // Mechanics
  final RelicTrigger trigger; 
  final RelicEffectType effectType; 
  final double effectValue;
  
  // Conditions (Optional)
  final RelicCondition condition; 
  final double? conditionValue;
  
  // Metadata
  final RelicRarity rarity;
  
  const RelicDefinition({
    required this.relicId,
    required this.nameKey,
    required this.descriptionKey,
    required this.trigger,
    required this.effectType,
    required this.effectValue,
    required this.rarity,
    this.condition = RelicCondition.none,
    this.conditionValue,
  });
}

enum RelicTrigger { alwaysActive, onLineClear, onBattleStart, onDamageDealt }
enum RelicEffectType { addFlatDamage, addMultiplier, restoreMoves }
```

---

## 7. Objective Schema (`objective_definition.dart`)

Defines win conditions for a Battle.

```dart
class ObjectiveDefinition {
  final String objectiveId;
  final ObjectiveType type; // Enum: defeatEnemy, clearLines, surviveMoves
  final int targetCount;    // e.g., 5 (lines), 0 (enemy HP)
  final int? timeLimit;     // Optional time constraint
  
  const ObjectiveDefinition({
    required this.objectiveId,
    required this.type,
    required this.targetCount,
    this.timeLimit,
  });
}
```

---

## 8. Battle Schema (`encounter_definition.dart`)

Defines a specific floor or node in a Run.

```dart
class EncounterDefinition {
  final String encounterId;
  final EnemyDefinition enemy;
  final ObjectiveDefinition primaryObjective;
  final ObjectiveDefinition? bonusObjective;
  
  // Constraints
  final int startingMoveLimit; // 0 if infinite
  
  const EncounterDefinition({
    required this.encounterId,
    required this.enemy,
    required this.primaryObjective,
    this.bonusObjective,
    this.startingMoveLimit = 0,
  });
}

class RunDefinition {
  final List<EncounterDefinition> floors;
  final String seed;
  
  const RunDefinition({
    required this.floors,
    required this.seed,
  });
}
```

---

## 9. Persistent Runtime State Classes (`run_state.dart`)

POCO (Plain Old C# Object) equivalents in Dart. Mutated during play, serialized to JSON via Hive CE.

```dart
class BoardState {
  // 64 integers (Occupancy + Modifier). Auth: `03` Sec 3.
  List<int> grid;
}

class EnemyRuntimeState {
  int currentHp;
  int actionCounter;
  String telegraphedActionId;
}

class RelicRuntimeState {
  String relicId;
  int activationCount;
}

class ObjectiveRuntimeState {
  int currentProgress;
  bool isCompleted;
}

class RunState {
  String runSeed;
  int rngStateCalls;
  int currentFloor;
  int currentStardust;
  List<String> activeRelicIds;
  
  BoardState boardState;
  EnemyRuntimeState enemyState;
  List<ObjectiveRuntimeState> objectiveStates;
  List<RelicRuntimeState> relicStates;
}
```

---

## 10. Content Registry

Definitions are loaded via Dart constants grouped in catalog files.
- `shape_catalog.dart`
- `enemy_catalog.dart`
- `relic_catalog.dart`

These catalogs provide static lookup maps:
```dart
class RelicCatalog {
  static const Map<String, RelicDefinition> allRelics = {
    'relic_iron_mallet': RelicDefinition(...),
  };
  
  static RelicDefinition get(String id) => allRelics[id]!;
}
```

---

## 11. Data Loading

For extremely large catalogs that would bloat the Dart binary, JSON files in `assets/data/` can be used and deserialized at boot time. However, for the MVP, pure Dart constants are strongly preferred as they are instantly available, strongly typed, and enable immediate AI code generation.

---

## 12. Validation Rules

Use `assert` in constructors and unit tests to validate data.

```dart
// Inside shape_catalog_test.dart
test('All shape matrices have exactly 25 elements', () {
  for (var shape in ShapeCatalog.allShapes.values) {
    expect(shape.blockMatrix.length, 25);
  }
});
```

---

## 13. IDs and Versioning

- **String IDs:** All content relies on unique string IDs (`shape_square_2x2`), NOT memory references, when interacting with the Save System (Hive CE).
- **Immutability:** Once a Relic ID is shipped, it cannot be renamed. Renaming an ID will corrupt player save files referencing that ID.

---

## 14. Null/Missing Data Behavior

- **Graceful Fallback:** If `RunState` references a `relicId` that no longer exists (e.g., removed in a patch), the loading system logs a warning, skips the missing relic, and continues loading. The save file must NOT corrupt.

---

## 15. Designer Authoring Workflow

To empower solo-dev and AI content creation:
1. **AI Output:** An AI agent can generate a Dart file of new relics following the `RelicDefinition` constructor syntax.
2. **No Code Modification:** The AI does not need to edit `combat_engine.dart` to add the "Executioner" relic, because the Trigger/Effect enums already support its logic.

---

## 16. Anti-Pattern Rules

1. **The "Live State" Definition:** Never put `int currentHp = 100;` in `EnemyDefinition`. It must only contain the static `maxHp`.
2. **Hardcoded IDs in Core:** Never write `if (relic.id == 'relic_iron_mallet') { damage += 10; }` inside the `CombatEngine`. The engine must resolve the math generically based on the `effectType` enum.

---

## 17. Final Data Contract

> **Data is the absolute truth; Systems are just interpreters.**
>
> In Block Battles, we lock down our core Dart systems early and scale the game purely by injecting wider varieties of data classes. By maintaining strict discipline over our Definitions, we ensure that adding content is safe, deterministic, and highly compatible with AI-assisted mass generation.

---

**End of `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md`.**
