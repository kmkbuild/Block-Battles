# 21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md
## Block Battles — System Managers, Services, and Dependencies

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document dismantles the classic "GameManager God Class" anti-pattern. It assigns strict boundaries and lifetimes to every functional component required by the game, enforcing the Clean Architecture rules established in Doc 18. The tech stack relies on Flutter, Flame, and Riverpod for two distinct gameplay experiences: **Mode 1 (Classic Endless)** and **Mode 2 (Goal-Driven Stage Challenge)**.

---

## 1. System Inventory

The architecture is divided into three distinct categories: **Infrastructure Services** (Global), **Application Services** (Session & Progression Orchestration), and **Domain Engines** (Contextual, Pure Dart).

### Infrastructure Services (Global Riverpod Providers)
- `BootstrapService` (App initialization, config loading)
- `SaveRepository` (Wraps Hive CE for persistent high scores and stage stars)
- `SettingsService` (Wraps SharedPreferences for audio toggles and haptic preferences)
- `AudioService` (Wraps Flame Audio for sound effects and music)
- `HapticService` (Wraps native haptic feedback)
- `MonetizationService` (Ad mob wrappers, rewarded revive state)

### Application Services (Riverpod State Notifiers)
- `ClassicGameService` & `ClassicStateNotifier` (Orchestrates Mode 1: score accumulation, streaks, high score updates, board lock game-over)
- `StageGameService` & `StageStateNotifier` (Orchestrates Mode 2: move limits, stage goal validation, star ratings, stage progression)

### Contextual Domain Engines (Pure Dart, No Riverpod/Flutter/Flame coupling)
- `BoardEngine` (8×8 grid state, shape placement validation, line detection, cell clearing)
- `TrayEngine` (Manages 3 piece slots, shape replenishment, ghost placement queries)
- `ScoreService` (Evaluates cell points, line clear bonuses, consecutive streak multipliers, board clear bonus)
- `StageObjectiveEngine` (Mode 2 progress tracker: target scores, color quotas, line milestones, obstacle removal)
- `GameRNG` (Deterministic seeded pseudorandom shape generation)

*(Note: Flame components and Flutter widgets belong strictly to the Presentation Layer, not core systems).*

---

## 2. Responsibility Matrix

| System | Owns | Does Not Own | Inputs | Outputs |
|---|---|---|---|---|
| **SaveRepository** | Hive CE box initialization, JSON/binary disk I/O, serialization. | Gameplay rules, scoring calculations. | `SaveData` DTOs | Persisted game state |
| **ClassicGameService** | Mode 1 session flow, score state, high score record comparison. | Board grid math, rendering. | Piece placements, Line clears | Game Over event, High Score record |
| **StageGameService** | Mode 2 stage progression, move counters, star ratings. | Board collision math, rendering. | Piece placements, Goal progress | Stage Clear, Stage Failed |
| **BoardEngine** | 8×8 cell-state array, collision checks, line detection, clears. | Piece drag graphics, audio, UI score. | Shape placement coordinates | `LinesClearedEvent`, `BoardClearedEvent` |
| **TrayEngine** | The 3 available draggable pieces, ghost preview legality. | Touch input events, board cell storage. | Shape pool, Placement queries | `SlotReplenishedEvent`, `TrayDepletedEvent` |
| **ScoreService** | Placement score (1 pt/cell), line clear math, streak multiplier (1.0×–2.5×). | Board state array, stage goals. | Cleared line counts, placement events | `ScoreUpdatedEvent`, Multiplier changes |
| **StageObjectiveEngine** | Objective tracking (target score, color block counts, lines cleared, obstacles). | Piece placement mechanics. | Lines cleared, cleared cell colors | `GoalCompletedEvent`, `AllGoalsMetEvent` |

---

## 3. Service Lifetime

- **App-Level (Global):** Provided via Riverpod globally in `ProviderScope` at `main.dart`. Persists for the entire application session (SaveRepository, AudioService, SettingsService).
- **Mode 1 Session-Local:** Instantiated when starting a Classic Endless game; destroyed upon returning to the Main Menu or restarting.
- **Mode 2 Stage-Local:** Instantiated upon launching a specific Stage; destroyed upon stage completion or stage selection exit.

---

## 4. Dependency Graph

*Dependencies point downwards. Higher levels orchestrate lower levels.*

```text
[Global Riverpod Providers] (SaveRepository, AudioService, SettingsService)
              │
              ▼
[Application Orchestration]
ClassicGameService / StageGameService
       ├──> TrayEngine
       ├──> BoardEngine
       ├──> ScoreService
       └──> StageObjectiveEngine (Mode 2 only)
              │
              ▼
[Data Models & Pure Utilities]
(BoardState, ShapeDefinition, StageDefinition, GameRNG)
```

---

## 5. Dependency Injection Strategy

**Decision:** Riverpod for App/Infrastructure Services + Constructor Injection for Domain Engines.
**Forbidden:** Singleton instances (e.g., `BoardEngine.instance`).

### 5.1 App/Infrastructure Services (Riverpod)
To avoid passing `AudioService` through layers of widgets, we use Riverpod.
```dart
// Used by Presentation layer (Flutter Widget or Game Controller)
ref.read(audioServiceProvider).playSfx(Sfx.piecePlace);
```

### 5.2 Domain Engines (Constructor Injection)
Core gameplay systems NEVER use Riverpod or Flutter packages. They must explicitly declare dependencies in their constructors. This ensures unit testability and determinism.
```dart
// Pure Dart, highly testable
class ScoreService {
  final ScoringConfig config;
  int _currentStreak = 0;

  ScoreService({this.config = const ScoringConfig()});

  ScoreResult registerClear({required int linesCleared, required int placedCellCount}) {
    // Pure calculation: placement points + line clear bonus * streak multiplier
  }
}
```

---

## 6. Initialization Order

1. **Boot (`main.dart`):** `main()` awaits `Hive.initFlutter()`, `SharedPreferences.getInstance()`, and Flame Audio initialization before running `runApp(ProviderScope(...))`.
2. **Persistent Services:** Instantiated on first read or eagerly initialized by a bootstrap provider.
3. **Session Start:** `ClassicGameService` or `StageGameService` initializes from Save Data or Stage Definition, instantiating `BoardEngine`, `TrayEngine`, and `ScoreService`.
4. **Game Presentation (`FlameGame.onLoad`):** The `FlameGame` reads domain engines and wires visual components (`BoardComponent`, `TrayComponent`) to the pure Dart engines.

*Rule:* If System A depends on System B, System B must be completely initialized before System A's constructor is called.

---

## 7. Shutdown Order

1. **Session End:** Game service disposes domain engine stream controllers. `FlameGame` unloads visual components.
2. **Progress Save:** Game service serializes high scores and star ratings to `SaveRepository`.
3. **App Quit:** OS handles memory cleanup; Hive boxes close gracefully.

---

## 8. Communication Rules

- **Upward Communication (Domain to App/Presentation):** Always use Dart `Stream` or an Event Dispatcher. The `BoardEngine` does not call `audioService.play()`. It fires a `LinesClearedEvent`, and the Application or Presentation layer listens and plays the audio effect.
- **Downward Communication (App to Domain):** Use direct method calls. `StageGameService` calls `BoardEngine.placeShape()`.
- **Lateral Communication (Domain to Domain):** Use direct method calls if injected via constructor.

---

## 9. Error Handling

- **Guard Clauses:** Every Domain method must validate inputs and throw `StateError` or `ArgumentError` if constraints are violated (e.g., placing an out-of-bounds piece).
- **Service Failure:** If an external infrastructure service fails (such as ads or cloud services), it gracefully falls back to a disabled state without crashing gameplay.

---

## 10. Testability

Constructor Injection for domain engines makes unit testing trivial in pure Dart:
```dart
test('ScoreService applies streak multiplier accurately', () {
  final scoreService = ScoreService();
  
  // First clear (streak = 1, multiplier = 1.0x)
  final result1 = scoreService.registerClear(linesCleared: 1, placedCellCount: 4);
  expect(result1.pointsAdded, 104); // 4 placement + 100 line clear
  
  // Second consecutive clear (streak = 2, multiplier = 1.2x)
  final result2 = scoreService.registerClear(linesCleared: 1, placedCellCount: 4);
  expect(result2.pointsAdded, 124); // 4 + (100 * 1.2) = 124
});
```

---

## 11. FlameGame vs Services

`FlameGame` acts strictly as the presentation bridge. Inside `onLoad()`, the game resolves required engines from Riverpod or a passed-in game service and instantiates Flame visual components (`BoardComponent`, `BlockComponent`, `TrayComponent`). Flame components DO NOT contain game rules or scoring math.

---

**End of `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md`.**
