# 22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md
## Block Battles — Event Bus and Game State Orchestration (Pure Arcade Engine)

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `01_GAMEPLAY_SPECIFICATION.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`  
**Document status:** Layer C — Engineering Authority  
**Core Authority:** Strongly-typed domain events, synchronous state evaluation, pure stream decoupling between Domain and Presentation.

---

## 1. Event Philosophy

**"Explicit, Strongly-Typed, and Traceable."**

All game events represent facts that have already occurred in the Domain. 
- Domain logic runs synchronously and deterministically.
- Presentation layers (Flame Components, Flutter Widgets) subscribe to Domain events to trigger visual animations, particles, haptics, and audio.
- Visual animations NEVER block or delay domain state advancement.

---

## 2. Classic Game State Machine

Managed by `ClassicGameService`:

| State | Description | Legal Next States |
|---|---|---|
| `idle` | Waiting for player input. Board and tray are active. | `dragging`, `boardLocked` |
| `dragging` | Player is actively dragging a tray piece. Ghost preview snaps to grid. | `resolvingTurn` (valid drop), `idle` (cancelled/invalid drop) |
| `resolvingTurn` | Synchronous execution: commits placement, clears lines, updates score, replenishes tray if needed. | `evaluatingLock` |
| `evaluatingLock` | Exhaustive check for any legal moves among remaining tray pieces. | `idle` (moves exist), `boardLocked` (zero moves exist) |
| `boardLocked` | Game Over condition. Compares final score to High Score and triggers modal. | `idle` (on replay restart) |

---

## 3. Canonical Event Registry

All events extend `GameDomainEvent`:

### 3.1 Board & Input Events
- `PlacementCommitted`: Fired when a piece is legally committed to the 8×8 grid.
  - Payload: `{ String shapeId, Point origin, List<Point> placedCells }`
- `LinesCleared`: Fired when 1 or more rows/columns are completed simultaneously.
  - Payload: `{ int lineCount, List<int> rows, List<int> cols, List<Point> clearedCells }`
- `BoardCleared`: Fired when the entire 8×8 board is left completely empty ($0$ filled cells).
  - Payload: `{ int bonusPoints }`

### 3.2 Scoring & Streak Events
- `ScoreUpdated`: Fired after every placement turn, reporting points awarded.
  - Payload: `{ int turnPoints, int totalScore, int streak }`
- `StreakUpdated`: Fired when the combo streak increments or resets.
  - Payload: `{ int streakTier, double multiplier }`
- `NewHighScoreAchieved`: Fired the moment current score exceeds the all-time local record.
  - Payload: `{ int previousBest, int newBest }`

### 3.3 Tray & Game Lifecycle Events
- `TrayReplenished`: Fired when all 3 tray slots are consumed and a fresh batch is generated.
  - Payload: `{ List<ShapeDefinition> newShapes }`
- `BoardLockDetected`: Fired when zero legal moves exist for any remaining tray piece.
  - Payload: `{ int finalScore, int linesCleared, int maxStreak, bool isNewBest }`
- `GameRestarted`: Fired when the player taps "Play Again" or restarts from pause.
  - Payload: `{ int newSeed }`

---

## 4. Event Bus Architecture

```dart
abstract class GameDomainEvent {
  final DateTime timestamp;
  GameDomainEvent() : timestamp = DateTime.now();
}

class EventDispatcher {
  final StreamController<GameDomainEvent> _controller = StreamController<GameDomainEvent>.broadcast();

  Stream<T> on<T extends GameDomainEvent>() {
    return _controller.stream.where((event) => event is T).cast<T>();
  }

  void dispatch(GameDomainEvent event) {
    _controller.add(event);
  }

  void dispose() {
    _controller.close();
  }
}
```

---

## 5. Acceptance Criteria

- All events are immutable with strongly-typed properties.
- Flame presentation reacts to streams without holding references to internal application controllers.
- No event is emitted during silent state restoration from save files.
- Strict scope discipline: events represent pure puzzle grid actions, scoring, and stage progression.

---

**End of `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md`.**
