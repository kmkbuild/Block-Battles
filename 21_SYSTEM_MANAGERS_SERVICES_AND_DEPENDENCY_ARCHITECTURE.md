# 21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md
## Block Battles — System Managers, Services, and Dependencies

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document dismantles the classic Unity "GameManager God Class" anti-pattern. It assigns strict boundaries and lifetimes to every functional component required by the game, enforcing the Clean Architecture rules established in Doc 18.

---

## 1. System Inventory

The architecture is divided into two distinct categories: **Persistent Services** (Global) and **Domain Engines** (Contextual).

### Persistent Services
- `BootstrapService`
- `SaveService`
- `SettingsService`
- `AudioService`
- `HapticService`
- `MonetizationService`

### Contextual Engines (Run & Battle)
- `RunManager`
- `BattleOrchestrator`
- `BoardEngine`
- `TrayEngine` (Handles Piece/Placement input state)
- `CombatEngine` (Handles Scoring/Combo/Damage)
- `EnemyEngine`
- `RelicEngine`
- `GameRNG`

*(Note: VFX, UI, and Input are mapped to the Presentation Layer, not core systems).*

---

## 2. Responsibility Matrix

| System | Owns | Does Not Own | Inputs | Outputs |
|---|---|---|---|---|
| **SaveService** | JSON disk I/O, serialization of runtime state. Does NOT own gameplay logic. Static content is reloaded from canonical assets. | Cloud syncing logic, gameplay state logic. | `RunSaveData` | `RunSaveData` (loaded) |
| **RunManager** | Owns run state (Stardust, active Relics list, current floor index). Initializes from loaded save data. | Battle specifics, Enemy stats. | Battle Win/Loss events | Next Floor data, Run Over event |
| **BattleOrchestrator**| Owns battle state and turn phases. Restores exact mid-battle state on load. | Board array, HP math. | `OnDrop`, `OnAnimFinish` | `TurnStateChange` |
| **BoardEngine** | 8x8 cell-state array (each cell encodes Occupancy + Modifier compactly per `03` Sec 3), collision math, line clearing math. Enemy systems request board mutations via `BoardEngine` API — they MUST NOT write directly to the internal array. | Piece drag graphics, scoring. | Piece coordinates, enemy-requested cell mutations | `LinesCleared`, `CellsCleared`, `BoardChanged` events |
| **TrayEngine** | The 3 available pieces, Ghost preview logic. | Input touch events, Board collision. | `ShapeData[]`, Board queries | `PieceCommitted` event |
| **CombatEngine** | Damage calculation, Combo (L-value) multipliers. | Relic definitions, Enemy HP. | `LinesCleared` event | `DamageDealt` event |
| **RelicEngine** | Passing modifier math to the CombatEngine based on triggers. | Combat resolution. | `LinesCleared`, `TurnStart` | `RelicProc` event |
| **EnemyEngine** | Enemy HP, telegraph intent generation, board-modification requests. Enemy resolves which cells to target, then calls `BoardEngine.SetCellModifier(cell, modifier, duration)` — it never writes raw cell values directly. | Damage dealing logic, Board math internals. | `DamageDealt` event | `EnemyDefeated`, `EnemyAction`, `BoardCellModified` |

---

## 3. Service Lifetime

- **Persistent (Global):** Created in `00_Boot` scene. Persists via `DontDestroyOnLoad` for the entire application session. (Save, Audio, Settings).
- **Run-Local:** Created when a player clicks "Start Run". Destroyed when the player Abandons or loses the Run. (RunManager).
- **Battle-Local:** Created at the start of `02_Battle` scene. Destroyed on victory/defeat. (BoardEngine, CombatEngine, EnemyEngine).
- **Transient:** Created and destroyed multiple times per frame or turn (e.g., `DamageEvent` structs).

---

## 4. Dependency Graph

*Dependencies point downwards. Higher levels orchestrate lower levels.*

```text
[Global Service Locator] (Provides generic access to IAudio, ISave)

[Application Orchestration]
RunManager ──────> BattleOrchestrator

[Battle Domain]
BattleOrchestrator
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

**Decision:** Lightweight Service Locator (for Globals) + Constructor Injection (for Domain).
**Forbidden:** Heavy Reflection-based DI frameworks (e.g., Zenject/Extenject).

### 5.1 Global Services (Locator)
To avoid passing `AudioService` through 15 layers of constructors, use a simple static locator for Global Services.
```csharp
// Used by Presentation layer
ServiceLocator.Get<IAudioService>().PlaySFX(SFX.PiecePlace);
```

### 5.2 Domain Engines (Constructor Injection)
Core gameplay systems NEVER use the Service Locator. They must explicitly declare dependencies in their constructor. This ensures perfect testability for AI and unit tests.
```csharp
// Pure C#, highly testable
public class CombatEngine {
    private readonly IRelicEngine _relicEngine;
    
    public CombatEngine(IRelicEngine relicEngine) {
        _relicEngine = relicEngine;
    }
}
```

---

## 6. Initialization Order

1. **Boot:** `BootstrapService` initializes Unity subsystems.
2. **Persistent Services:** `SaveService` -> `SettingsService` -> `AudioService` -> `MonetizationService`.
3. **Run:** `RunManager` initializes from Save Data, spins up `GameRNG` with the saved seed AND advances it by the saved `rngStateCalls` to resume exact deterministic sequence.
4. **Battle:** `BattleOrchestrator` -> `BoardEngine` -> `EnemyEngine` -> `RelicEngine` -> `CombatEngine` -> `TrayEngine`.

*Rule:* If System A depends on System B, System B must be completely initialized before System A's constructor is called.

---

## 7. Shutdown Order

Order of destruction prevents memory leaks and dangling event subscriptions.
1. **Battle End:** `BattleOrchestrator` cleans up event subscriptions between core Engines -> Destroys Battle Domain.
2. **Run End:** `RunManager` serializes state -> Sends to `SaveService` -> Destroys Run Domain.
3. **App Quit:** Disconnects `AudioService` and `MonetizationService`.

---

## 8. Communication Rules

- **Upward Communication (Domain to App/Presentation):** Always use C# `event` or `Action`. The BoardEngine does not call `AudioManager.Play()`. It fires `OnLineCleared`, and the Presentation layer listens and plays the audio.
- **Downward Communication (App to Domain):** Use direct method calls. `BattleOrchestrator` calls `BoardEngine.PlacePiece()`.
- **Lateral Communication (Domain to Domain):** Use direct interface calls if injected via constructor. 

---

## 9. Cross-System Contracts

Example sequence when a piece is dropped:
1. `InputController` (Presentation) calls `TrayEngine.TryCommitPlacement(x, y)`.
2. `TrayEngine` queries `BoardEngine.IsValid(piece, x, y)`.
3. If valid, `TrayEngine` consumes the piece, tells `BoardEngine` to write to the array.
4. `BoardEngine` detects lines, clears them, fires `OnLinesCleared(L_Value)`.
5. `BattleOrchestrator` hears the event, pauses input.
6. `BattleOrchestrator` tells `CombatEngine.CalculateDamage(L_Value)`.
7. `CombatEngine` queries `RelicEngine` for modifiers.
8. `CombatEngine` tells `EnemyEngine.TakeDamage(amount)`.
9. `EnemyEngine` updates HP, fires `OnHPChanged`.
10. `Presentation` layer reacts to all fired events with VFX/Animations.

---

## 10. Error Handling

- **Guard Clauses:** Every Domain method must immediately validate inputs and throw `ArgumentException` or `InvalidOperationException` if constraints are violated (e.g., placing a piece out of bounds).
- **Service Failure:** If `MonetizationService` fails to initialize (e.g., no internet), the game degrades gracefully (Revive button is hidden), it does NOT crash the puzzle loop.

---

## 11. Testability

Because we use Constructor Injection for the Battle Domain, AI can instantly generate NUnit tests:
```csharp
[Test]
public void CombatEngine_AppliesRelicMultiplier() {
    var mockRelics = new MockRelicEngine(); // Returns 2x multiplier
    var combat = new CombatEngine(mockRelics);
    int damage = combat.CalculateDamage(linesCleared: 1, baseScore: 10);
    Assert.AreEqual(20, damage);
}
```

---

## 12. Debugging

- **IDebugService:** Implemented only in Editor/Development builds. 
- Automatically logs all state transitions in `BattleOrchestrator` to the Unity Console.
- Exposes a Cheats UI to force-trigger `CombatEngine.CalculateDamage` or `EnemyEngine.TakeDamage` for fast QA testing.

---

## 13. Forbidden Dependencies (Anti-Patterns)

1. **Circular Dependencies:** System A requires System B, which requires System A. (Fix: Extract an interface or move logic to a shared third system).
2. **Domain depending on Presentation:** E.g., `CombatEngine` waiting for an `Animator` to finish. (Fix: Presentation must track the animator and notify the `BattleOrchestrator` when ready).
3. **Singleton Abuse:** E.g., `BoardEngine.Instance`. The BoardEngine is battle-local, not a singleton. If two battles were instantiated in memory, a singleton would crash. 

---

## 14. Example Battle Lifecycle (Turn State Machine)

The `BattleOrchestrator` operates this internal state machine:
- `STATE_INIT:` Spin up engines. Transition ->
- `STATE_PLAYER_TURN:` Unlock input. Wait for `TrayEngine.OnCommit`. Transition ->
- `STATE_COMBAT_EVAL:` Calculate damage, apply to enemy. Transition ->
- `STATE_ANIMATION_WAIT:` Input locked. Wait for Presentation layer to signal VFX complete. Transition ->
- `STATE_ENEMY_TURN:` `EnemyEngine` executes Board modifications. Transition ->
- `STATE_BOARD_EVAL:` Check for Move Limit or Board Lock. If game over -> `STATE_BATTLE_END`. If safe -> `STATE_PLAYER_TURN`.

---

## 15. Final Service Contract

> **Everything has an owner. Nothing owns everything.**
>
> By strictly defining these 14 core systems, we eliminate the spaghetti code that dooms most indie Unity projects. An AI coding agent will never need to guess where a line of code belongs. 
> 
> If it calculates damage, it goes in the Combat Engine. If it writes to disk, it goes in the Save Service. If it orchestrates turn order, it goes in the Battle Orchestrator. 

---

**End of `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md`.**
