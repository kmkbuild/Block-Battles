# 22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md
## Block Battles — Event Bus and Game State Orchestration

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the explicit communication protocol between decoupled systems in Dart. It prevents tight coupling while avoiding over-engineered message brokers, leveraging Riverpod and Dart Streams.

---

## 1. Event Philosophy

**"Explicit, Typed, and Traceable."**

We use a **Strongly-Typed Event Dispatcher** backing Dart Streams. An event is a declaration of fact: *"This happened in the past."* The Presentation layer (Flutter Widgets, Flame Components) listens to the Domain layer; the Domain layer never listens to the Presentation layer.

---

## 2. Commands vs Events

- **Command:** `BoardEngine.placePiece(x, y)`
  - Moves downward in the architecture.
  - Can be rejected (throws or returns false).
- **Event:** `PiecePlacedEvent(x, y)`
  - Moves upward/outward via Streams.
  - Cannot be rejected.
  - Triggers side effects (Flame VFX, Flame Audio, Riverpod state updates).

---

## 3. Game State Machine

The `BattleService` governs the flow of a single battle using a pure Dart enum-based state machine. Flutter/Flame react to state changes to lock input or show animations.

| State | Description | Transitions To |
|---|---|---|
| `idle` | Awaiting player drag/drop input. | `resolvingBoard` |
| `resolvingBoard`| Line clearing, math calculation. | `resolvingCombat` |
| `resolvingCombat`| Applying Relics, Enemy damage. | `waitingForAnim` |
| `waitingForAnim`| Input locked until Flame signals VFX complete. | `enemyTurn`, `victory`, `defeat` |
| `enemyTurn` | Enemy board modification. | `idle`, `defeat` |
| `victory` | Encounter won. | `awaitingRelicSelection` |
| `awaitingRelicSelection`| Selecting a Relic via Flutter UI. | `idle` (Next floor) |
| `defeat` | Board Lock / Objective Failure hit. | (Ends Run) |

---

## 4. Canonical Event Registry

Events are sealed classes or final classes extending a base `DomainEvent`. A Riverpod provider `eventDispatcherProvider` or a dedicated `StreamController<DomainEvent>` broadcasts them.

### Input & Board Events
- `PlacementCommitted`
- `LinesCleared`
- `CellsCleared`
- `BoardLockDetected` (Triggers `defeat` state)
- `BoardCellModified`
- `BoardCellModifierExpired`

### Combat & Progression Events
- `DamageDealt`
- `EnemyDefeated`
- `RelicTriggered`
- `ObjectiveProgressed`
- `ObjectiveCompleted`
- `MoveLimitChanged`

### Run Events
- `BattleStarted`
- `BattleVictory`
- `BattleFailure`
- `RunCompleted`
- `RunDefeated`
- `RelicSelected`
- `FloorChanged`

---

## 5. Event Payloads

Events must carry all context necessary for listeners. We use final properties.

```dart
final class LinesClearedEvent extends DomainEvent {
  final int lineCount;
  final List<int> clearedCells;
  
  const LinesClearedEvent(this.lineCount, this.clearedCells);
}

final class DamageDealtEvent extends DomainEvent {
  final int amount;
  final bool isCritical;
  final String? sourceRelicId;
  
  const DamageDealtEvent(this.amount, this.isCritical, this.sourceRelicId);
}
```

---

## 6. Event Ordering & Re-entrancy

When multiple events fire, they resolve synchronously in the domain, emitting sequentially. The `BattleService` acts as the orchestrator to prevent infinite loops (e.g., Relic triggers line clear which triggers Relic). The UI/Flame layer queues animations based on the emitted stream of events.

---

## 7. Save Restoration Event Policy

**Rule:** Restoring a saved mid-run state MUST be silent on the Event Bus. 
When `RunService` restores state, no visual events (`LinesCleared`, `DamageDealt`) should emit, preventing chaotic VFX on app boot.

---

## 8. Event Bus Implementation

A central event dispatcher manages a stream.

```dart
class EventDispatcher {
  final _controller = StreamController<DomainEvent>.broadcast();

  Stream<T> on<T extends DomainEvent>() {
    return _controller.stream.where((event) => event is T).cast<T>();
  }

  void dispatch(DomainEvent event) {
    _controller.add(event);
  }

  void dispose() {
    _controller.close();
  }
}
```

---

## 9. Subscription Rules

To prevent memory leaks:
- **Riverpod:** Use `ref.listen` for global effects or Flutter UI.
- **Flame Components:** Store `StreamSubscription` and cancel it in the component's `onRemove()` method.
- **StatefulWidgets:** Store `StreamSubscription` and cancel in `dispose()`.

```dart
// In a Flame Component
late StreamSubscription _sub;

@override
void onLoad() {
  _sub = eventDispatcher.on<LinesClearedEvent>().listen(_onLinesCleared);
}

@override
void onRemove() {
  _sub.cancel();
  super.onRemove();
}
```

---

## 10. Final Event Contract

> **Information flows up; Orders flow down.**
>
> The core systems scream into the void: *"A line was cleared!"* It is up to the Flutter/Flame presentation layer to listen and make it look beautiful. No domain logic depends on the UI.

---

**End of `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md`.**
