# 21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md
## Block Battles — System Managers, Services, and Dependencies

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_FLUTTER_PROJECT_STRUCTURE_AND_ROUTING.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document dismantles the classic "GameManager God Class" anti-pattern. It assigns strict boundaries and lifetimes to every functional component required by the game, enforcing the Clean Architecture rules established in Doc 18. The tech stack relies on Flutter, Flame, and Riverpod.

---

## 1. System Inventory

The architecture is divided into three distinct categories: **Infrastructure Services** (Global), **Application Services** (Run & Orchestration), and **Domain Engines** (Contextual, Pure Dart).

### Infrastructure Services (Global Riverpod Providers)
- `BootstrapService`
- `SaveRepository` (Wraps Hive CE)
- `SettingsService` (Wraps SharedPreferences)
- `AudioService` (Wraps Flame Audio)
- `HapticService`
- `MonetizationService` (Abstract, mock during dev)

### Application Services (Riverpod State Notifiers)
- `RunService` & `RunStateNotifier` (Handles overall Run progression)
- `BattleService` (Pure Dart orchestration of the battle state machine)

### Contextual Domain Engines (Pure Dart, No Riverpod/Flutter/Flame coupling)
- `BoardEngine`
- `TrayEngine` (Handles Piece/Placement input state)
- `CombatEngine` (Handles Scoring/Combo/Damage)
- `EnemyEngine`
- `RelicEngine`
- `GameRNG`

*(Note: Flame components and Flutter widgets belong to the Presentation Layer, not core systems).*

---

## 2. Responsibility Matrix

| System | Owns | Does Not Own | Inputs | Outputs |
|---|---|---|---|---|
| **SaveRepository** | Hive CE box initialization, JSON/binary disk I/O, serialization. | Cloud syncing logic, gameplay state logic. | `RunSaveData` | `RunSaveData` (loaded) |
| **RunService** | Owns run state (Stardust, active Relics list, current floor index) via `RunStateNotifier`. | Battle specifics, Enemy stats. | Battle Win/Loss events | Next Floor data, Run Over state |
| **BattleService**| Owns battle state and turn phases. Restores exact mid-battle state on load. | Board array, HP math. | Piece placements, Anim completions | Battle state changes |
| **BoardEngine** | 8x8 cell-state array (each cell encodes Occupancy + Modifier), collision math, line clearing math. | Piece drag graphics, scoring. | Piece coordinates, enemy mutations | `LinesCleared`, `CellsCleared` |
| **TrayEngine** | The 3 available pieces, Ghost preview logic. | Input touch events, Board collision. | `ShapeData[]`, Board queries | `PlacementCommitted` |
| **CombatEngine** | Damage calculation, Combo (L-value) multipliers. | Relic definitions, Enemy HP. | `LinesCleared` | `DamageDealt` |
| **RelicEngine** | Passing modifier math to the CombatEngine based on triggers. | Combat resolution. | `LinesCleared`, `TurnStart` | `RelicProc` |
| **EnemyEngine** | Enemy HP, telegraph intent generation, board-modification requests. | Damage dealing logic, Board math. | `DamageDealt` | `EnemyDefeated`, `EnemyAction` |

---

## 3. Service Lifetime

- **App-Level (Global):** Provided via Riverpod globally in `ProviderScope` at `main.dart`. Persists for the entire application session (Save, Audio, Settings).
- **Run-Local:** Lives inside the `RunStateNotifier` lifecycle. Reset when a run starts or ends.
- **Battle-Local:** Created at the start of a battle (via `BattleService` or inside `FlameGame`). Destroyed on victory/defeat. (BoardEngine, CombatEngine, EnemyEngine).

---

## 4. Dependency Graph

*Dependencies point downwards. Higher levels orchestrate lower levels.*

```text
[Global Riverpod Providers] (SaveRepository, AudioService, SettingsService)

[Application Orchestration]
RunService ──────> BattleService

[Battle Domain]
BattleService
 ├──> TrayEngine
 ├──> BoardEngine
 ├──> RelicEngine
 ├──> CombatEngine
 └──> EnemyEngine

[Data]
(All systems above query Data Models and GameRNG)
```

---

## 5. Dependency Injection Strategy

**Decision:** Riverpod for App/Infrastructure Services + Constructor Injection for Domain Engines.
**Forbidden:** Singleton instances (e.g., `BoardEngine.instance`). 

### 5.1 App/Infrastructure Services (Riverpod)
To avoid passing `AudioService` through layers of widgets, we use Riverpod.
```dart
// Used by Presentation layer (Flutter Widget)
ref.read(audioServiceProvider).playSfx(Sfx.piecePlace);
```

### 5.2 Domain Engines (Constructor Injection)
Core gameplay systems NEVER use Riverpod or Flutter packages. They must explicitly declare dependencies in their constructors. This ensures perfect testability for AI and unit tests.
```dart
// Pure Dart, highly testable
class CombatEngine {
  final RelicEngine _relicEngine;
  
  CombatEngine(this._relicEngine);
}
```

---

## 6. Initialization Order

1. **Boot (`main.dart`):** `main()` awaits `Hive.initFlutter()`, `SharedPreferences.getInstance()`, and Flame Audio initialization before running `runApp(ProviderScope(...))`.
2. **Persistent Services:** Instantiated on first read or eagerly initialized by a `bootstrapProvider`.
3. **Run:** `RunService` initializes from Save Data, spins up `GameRNG` with the saved seed.
4. **Battle (`FlameGame.onLoad`):** The `FlameGame` reads the domain engines from `BattleService` and wires up the visual components to the pure Dart engines.

*Rule:* If System A depends on System B, System B must be completely initialized before System A's constructor is called.

---

## 7. Shutdown Order

1. **Battle End:** `BattleService` disposes domain engines and stream controllers. `FlameGame` unloads components.
2. **Run End:** `RunService` serializes state -> Sends to `SaveRepository`.
3. **App Quit:** OS handles memory cleanup; Hive boxes close gracefully.

---

## 8. Communication Rules

- **Upward Communication (Domain to App/Presentation):** Always use Dart `Stream` or an Event Dispatcher. The BoardEngine does not call `audioService.play()`. It fires a `LinesCleared` event, and the Application or Presentation layer listens and plays the audio.
- **Downward Communication (App to Domain):** Use direct method calls. `BattleService` calls `BoardEngine.placePiece()`.
- **Lateral Communication (Domain to Domain):** Use direct interface/class calls if injected via constructor. 

---

## 9. Error Handling

- **Guard Clauses:** Every Domain method must validate inputs and throw `StateError` or `ArgumentError` if constraints are violated.
- **Service Failure:** If `MonetizationService` fails to initialize, it gracefully falls back to a mock/disabled state.

---

## 10. Testability

Constructor Injection for the Battle Domain makes unit testing trivial in pure Dart:
```dart
test('CombatEngine applies relic multiplier', () {
  final mockRelics = MockRelicEngine(); // Returns 2x multiplier
  final combat = CombatEngine(mockRelics);
  final damage = combat.calculateDamage(linesCleared: 1, baseScore: 10);
  expect(damage, 20);
});
```

---

## 11. FlameGame vs Services

`FlameGame` acts as the presentation bridge. Inside `onLoad()`, the game resolves required engines from Riverpod or a passed-in `BattleService` and instantiates Flame components (`PositionComponent`, etc.) passing the engines to them. Flame components DO NOT contain game rules.

---

**End of `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md`.**
