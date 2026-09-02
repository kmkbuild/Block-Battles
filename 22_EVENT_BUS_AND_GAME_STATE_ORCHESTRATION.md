# 22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md
## Block Battles — Event Bus and Game State Orchestration

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the exact communication protocol between the decoupled systems outlined in Doc 21. It prevents tight coupling while explicitly avoiding enterprise-level message broker over-engineering.

---

## 1. Event Philosophy

**"Explicit, Typed, and Traceable."**

For a solo developer or AI coding agent, debugging string-based event buses (e.g., `EventManager.Fire("OnDamage", 50)`) is a nightmare because references are invisible to IDEs. 

We use a **Strongly-Typed Event Bus** or **Direct C# Events**. An event is a declaration of fact: *"This happened in the past."* It does not care who is listening. The Presentation layer (UI, Audio, VFX) listens to the Domain layer; the Domain layer never listens to the Presentation layer.

---

## 2. Commands vs Events

To maintain strict control flow, we separate Commands (Intent) from Events (Result).

- **Command:** `BoardEngine.PlacePiece(x, y)`
  - Moves downward in the architecture.
  - Can be rejected (e.g., invalid placement).
  - Returns a boolean or result struct.
- **Event:** `OnPiecePlaced(x, y)`
  - Moves upward/outward in the architecture.
  - Cannot be rejected (it already happened).
  - Triggers side effects (VFX, Audio, next State).

---

## 3. Game State Machine

The `BattleOrchestrator` governs the flow of a single run/battle. It ensures input is blocked when animations or logic are resolving.

| State | Description | Transitions To |
|---|---|---|
| `BOOT` | System initialization. | `MENU` |
| `MENU` | Main menu, Compendium. | `RUN_SETUP` |
| `RUN_SETUP` | Seed generation, floor init. | `BATTLE_IDLE` |
| `BATTLE_IDLE` | Awaiting player drag/drop input. | `RESOLVING_BOARD` |
| `RESOLVING_BOARD`| Line clearing, math calculation. | `RESOLVING_COMBAT` |
| `RESOLVING_COMBAT`| Applying Relics, Enemy damage. | `WAITING_FOR_ANIM` |
| `WAITING_FOR_ANIM`| Input locked until VFX/Audio signal complete. | `ENEMY_TURN`, `VICTORY`, `DEFEAT` |
| `ENEMY_TURN` | Enemy board modification. | `BATTLE_IDLE` |
| `VICTORY` | Encounter won. | `REWARD` |
| `REWARD` | Selecting a Relic. | `BATTLE_IDLE` (Next floor) |
| `DEFEAT` | Board Lock / Move Limit hit. | `MENU` |

---

## 4. Canonical Event Registry

This is the exhaustive list of Domain events required for the MVP. Do not create events unless a specific system needs to listen to them.

### Input & Board Events
- `PiecePickedUp` (Triggers scale up, audio)
- `PieceDroppedInvalid` (Triggers rejection shake, audio)
- `PiecePlaced` (Triggers board update, thud audio)
- `LinesCleared` (Triggers flash, fragment VFX)
- `BoardLocked` (Triggers game over sequence)
- `BoardCellModified` (MVP — fires when an enemy action applies or removes a cell modifier: `Blocked`, `Frozen`, etc. Triggers cell-state visual update on `BoardView`. Payload: `cell: (row, col)`, `modifier: CellModifier`, `duration: int turns`. Emitted by `BoardEngine` after `SetCellModifier` call from `EnemyEngine`.)
- `BoardCellModifierExpired` (MVP — fires when a timed cell modifier (e.g., Block duration elapses) is automatically cleared. Triggers cell restore visual. Payload: `cell: (row, col)`, `previousModifier: CellModifier`.)

### Combat & Progression Events
- `DamageDealt` (Triggers UI numbers, enemy hit animation)
- `EnemyDefeated` (Triggers shatter VFX)
- `RelicTriggered` (Triggers UI chip flash, causality trail)
- `ObjectiveProgressed` (Triggers HUD counter update)
- `ObjectiveCompleted` (Triggers HUD confetti burst)
- `MoveLimitChanged` (Triggers UI update, warning pulse if low)

### Run Events
- `BattleStarted` (Triggers BGM change, board slide-in)
- `BattleCompleted` (Triggers Victory sting)
- `RelicSelected` (Triggers reward acquisition)

---

## 5. Event Payloads

Events must carry all context necessary for the listeners so listeners don't have to query the Domain layer retroactively. 

We use lightweight `structs` for payloads to prevent Garbage Collection allocation.

```csharp
public struct LinesClearedEvent {
    public int lineCount;       // Number of simultaneous lines (L-Value)
    public int[] clearedCells;  // Array of 1D indices for VFX spawning
}

public struct DamageDealtEvent {
    public int amount;
    public bool isCritical;
    public string sourceRelicID; // Null if base damage
}

public struct RelicTriggeredEvent {
    public string relicID;
    public Vector2Int targetCell; // Used to draw the causality trail
}
```

---

## 6. Event Ordering (Critical Section)

When multiple events fire, they must fire in a predictable, synchronous sequence. 
*Example: A line clear that kills an enemy.*

1. Command: `PlacePiece`
2. Math resolves internally.
3. Event: `PiecePlaced`
4. Math detects 3 lines.
5. Event: `LinesCleared` (L=3)
6. Combat math resolves (Base + L3 multiplier).
7. Relic math resolves (Executioner triggers).
8. Event: `RelicTriggered` (Executioner)
9. Event: `DamageDealt` (150)
10. Enemy HP math resolves -> hits 0.
11. Event: `EnemyDefeated`
12. Event: `ObjectiveCompleted`

The UI/VFX layer receives these events instantly in Frame 1, and queues the corresponding animations to play sequentially (`12` Sec 20.1).

---

## 7. Event Re-entrancy

**Rule:** A listener responding to an event must NEVER trigger a Command that results in the same event firing again while the original is still resolving.

*Example problem:* A Relic clears a line when triggered. The line clear triggers the Relic. Infinite loop.
*Solution:* The Domain layer implements a strict Resolution Queue. Relic effects are evaluated against the initial state, aggregated, and applied. They do not trigger cascading mechanics recursively.

---

## 8. Save Restoration Event Policy

**Rule:** Restoring a saved mid-run state MUST be silent on the Event Bus. 

When the `BootstrapService` / `RunManager` initializes the `BoardEngine`, `CombatEngine`, and `EnemyEngine` with saved state data (`int[64]`, HP, etc.), no events from Section 4 are permitted to fire. 
- Restoring the board must NOT fire `PiecePlaced` or `LinesCleared`.
- Restoring the enemy must NOT fire `DamageDealt`.
- Restoring the tray must NOT trigger new piece generation routines or "Piece Spawned" equivalents.

*Why:* The Presentation layer listens to events to play VFX, subtract HP visually, and grant rewards. If loading fires events, the UI will play a chaotic sequence of explosions and damage numbers on boot, and objective trackers might double-count progress. The UI layer must poll the Domain once on scene load (e.g., `UpdateAllVisualsToMatchState()`), after which standard event-driven updates resume.

---

## 9. Event Bus Implementation (Solo-Dev Scope)

We use a static, type-safe event bus. It requires zero setup and is instantly understood by AI agents and IDEs.

```csharp
public static class EventBus<T> where T : struct {
    public static event Action<T> OnEvent;

    public static void Raise(T eventData) {
        OnEvent?.Invoke(eventData);
    }
}
```

**Usage (Publisher):**
`EventBus<DamageDealtEvent>.Raise(new DamageDealtEvent { amount = 50 });`

**Usage (Subscriber):**
`EventBus<DamageDealtEvent>.OnEvent += HandleDamage;`

---

## 9. Subscription Rules

Memory leaks happen when Unity objects are destroyed but remain subscribed to static events. 

- **Rule 1:** A `MonoBehaviour` MUST subscribe in `OnEnable()` and unsubscribe in `OnDisable()`.
- **Rule 2:** `Start()` and `OnDestroy()` are forbidden for static event subscriptions because script reloads or inactive game objects will bypass them.

```csharp
private void OnEnable() {
    EventBus<LinesClearedEvent>.OnEvent += OnLinesCleared;
}
private void OnDisable() {
    EventBus<LinesClearedEvent>.OnEvent -= OnLinesCleared;
}
```

---

## 10. Debugging Event Flow

For Editor builds only, wrap the `EventBus.Raise()` in a pre-processor directive to log all traffic. This allows instantaneous tracing of the game state.

```csharp
#if UNITY_EDITOR
Debug.Log($"[EVENT] {typeof(T).Name} | {JsonUtility.ToJson(eventData)}");
#endif
```

---

## 11. Testing Event Sequences

Because events are isolated structs, unit tests can easily verify complex combat logic without Unity.

```csharp
[Test]
public void L3Clear_Triggers_DamageDealt() {
    bool damageDealt = false;
    EventBus<DamageDealtEvent>.OnEvent += (e) => damageDealt = true;

    var combat = new CombatEngine();
    combat.CalculateDamage(linesCleared: 3);

    Assert.IsTrue(damageDealt);
}
```

---

## 12. Example Complete Turn Trace

**Scenario:** Player drops a 3x3 piece, clears 2 lines, triggers a combo relic, deals 100 damage, leaves enemy at 50 HP.

1. `TrayEngine` -> Checks `BoardEngine.IsValid()`. Returns True.
2. `TrayEngine` -> Calls `BoardEngine.CommitPlacement()`.
3. `BoardEngine` -> Writes to matrix. Raises `PiecePlacedEvent`.
4. `BoardEngine` -> Scans matrix. Finds 2 full rows.
5. `BoardEngine` -> Clears matrix rows. Raises `LinesClearedEvent { lineCount = 2 }`.
6. `BattleOrchestrator` -> Hears `LinesCleared`. Transitions to `RESOLVING_COMBAT`.
7. `BattleOrchestrator` -> Calls `CombatEngine.Execute(2)`.
8. `CombatEngine` -> Checks `RelicEngine`. Relic "Combo Capacitor" condition met.
9. `RelicEngine` -> Raises `RelicTriggeredEvent`. Returns x1.5 multiplier.
10. `CombatEngine` -> Base 20 * L2 (2.0) * Relic (1.5) = 60 Damage.
11. `CombatEngine` -> Calls `EnemyEngine.TakeDamage(60)`.
12. `EnemyEngine` -> Subtracts HP. Raises `DamageDealtEvent { amount = 60 }`.
13. `BattleOrchestrator` -> Transitions to `WAITING_FOR_ANIM`.
14. `Presentation Layer` -> Plays Flash VFX, triggers Relic Trail VFX, pops "60" damage text.
15. `Presentation Layer` -> Finishes playing. Calls `BattleOrchestrator.OnAnimationsComplete()`.
16. `BattleOrchestrator` -> Transitions to `BATTLE_IDLE`. Player input unlocks.

---

## 13. Final Event Contract

> **Information flows up; Orders flow down.**
>
> By utilizing a type-safe generic Event Bus and strictly typed structs, we ensure that adding a new visual effect or audio cue requires zero modifications to the core logic. 
> 
> The core systems scream into the void: *"A line was cleared!"* It is up to the presentation layer to listen and make it look beautiful.

---

**End of `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md`.**
