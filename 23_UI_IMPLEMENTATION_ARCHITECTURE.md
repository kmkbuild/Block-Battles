# 23_UI_IMPLEMENTATION_ARCHITECTURE.md
## Block Battles — UI Implementation Architecture

**Governing documents:** `08_UI_UX_MASTER_SPECIFICATION.md`, `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md`, `18_TECHNICAL_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document dictates the Flutter Widget and Flame Component structures required. It enforces a strict boundary between Flutter (App UI) and Flame (Gameplay Rendering).

---

## 1. UI Architecture Philosophy

**"The UI is Dumb. The UI is Reactive."**

1.  **Passive View:** A Flutter widget or Flame component never calculates math or modifies state directly without sending an intent to a Riverpod notifier or Domain Engine.
2.  **Boundary Rule:** The Board, Pieces, and Drag/Drop interactions happen inside Flame. HUD, Menus, Meta-currency, and Overlays happen in Flutter.
3.  **Responsive Layout:** Flutter's `LayoutBuilder`, `SafeArea`, `Expanded`, and `MediaQuery` handle screen scaling.

---

## 2. Screen & Route Registry

Navigation is handled by `GoRouter`.

| Route | Widget | Description |
|---|---|---|
| `/` | `HomeScreen` | Main menu, meta-progression, Run Start |
| `/battle` | `BattleScreen` | Hosts `GameWidget` + `BattleHudOverlay` |
| `/settings` | `SettingsScreen` | Full screen or dialog for configuration |
| `/relics` | `RelicSelectionOverlay` | Flutter modal overlay over the game |
| `/victory`| `VictoryOverlay` | End of battle/run stats |

---

## 3. The BattleScreen Stack (Flutter + Flame)

The `BattleScreen` heavily uses Flutter's `Stack` to layer UI over the Flame game.

```dart
Widget build(BuildContext context) {
  return Scaffold(
    body: SafeArea(
      child: Stack(
        children: [
          // 1. Base Layer: Flame Engine
          GameWidget(game: blockBattlesGame),
          
          // 2. HUD Layer: Flutter Widgets ignoring Flame touches
          IgnorePointer(
            child: BattleHudOverlay(),
          ),
          
          // 3. Modal Layer: Pause, Relics, Settings
          if (isPaused) PauseOverlay(),
        ],
      ),
    ),
  );
}
```
*Note:* To prevent Flutter overlays from intercepting Flame touch events, use `IgnorePointer` on non-interactive HUD elements.

---

## 4. Flame Render & Input (The Game World)

- **BoardView/TrayView:** Implemented as Flame `PositionComponent`s inside `FlameGame`. 
- **Piece Dragging:** Flame's `DragCallbacks` mixin handles piece interactions.
- **VFX:** Flame `SpriteAnimationComponent` or Particle effects spawn on `LinesCleared` events.
- **Animations:** Use Flame effects (`MoveEffect`, `ScaleEffect`) for board elements.

---

## 5. Flutter UI Components (The HUD & Menus)

- **Enemy HUD (`EnemyHudWidget`):** Flutter widget placed via `Positioned` in the stack. Reads HP from Riverpod or event streams.
- **Objective UI (`ObjectiveCardWidget`):** Shows progression. Uses Flutter `AnimationController` for fill bars.
- **Relic Cards (`RelicCardWidget`):** Reusable Flutter UI for relic selection.

---

## 6. Reactive Data Binding

We use Riverpod `ConsumerWidget` for declarative UI updates.

```dart
class EnemyHudWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final enemyState = ref.watch(enemyStateProvider);
    return Text('${enemyState.currentHp} / ${enemyState.maxHp}');
  }
}
```

---

## 7. Animation Boundary

- **UI Animation:** Flutter's `AnimatedWidget`, `AnimationController`, or implicit animations (`AnimatedContainer`).
- **Game World Animation:** Flame's `Effect` classes.
- **Synchronization:** When the UI needs to tell the domain it finished animating (e.g., enemy takes damage visual), the widget or component dispatches a `UIAnimationCompleteEvent` to the `BattleService` state machine, moving it out of `waitingForAnim`.

---

## 8. Widget Naming Conventions

- Use PascalCase for classes.
- Suffix screens with `Screen` (e.g., `HomeScreen`).
- Suffix overlays with `Overlay` (e.g., `PauseOverlay`).
- Suffix reusable pieces with `Widget` (e.g., `GameButtonWidget`, `EnemyHudWidget`).

---

## 9. Safe Areas and Responsiveness

- Wrap the main stack in a `SafeArea` widget to avoid notches.
- Use `LayoutBuilder` inside the `GameWidget` wrapper if the Flame game needs explicit bounds, or let Flame's camera system scale the board dynamically while Flutter handles the edges.

---

## 10. Localization & Fonts

- Text uses standard Flutter `Text` widgets with custom font families declared in `pubspec.yaml`.
- Use an i18n package (like `slang` or `flutter_localizations`) to replace static strings.

---

**End of `23_UI_IMPLEMENTATION_ARCHITECTURE.md`.**
