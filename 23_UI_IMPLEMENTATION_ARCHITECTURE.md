# 23_UI_IMPLEMENTATION_ARCHITECTURE.md
## Block Battles — UI Implementation Architecture

**Governing documents:** `08_UI_UX_MASTER_SPECIFICATION.md`, `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md`, `18_TECHNICAL_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document dictates the Flutter Widget and Flame Component structures required for the game. It enforces a strict boundary between Flutter (App UI, HUD, Modals) and Flame (Board & Piece Rendering, VFX, Drag Input) for both **Mode 1 (Classic Endless)** and **Mode 2 (Goal-Driven Stage Challenge)**.

---

## 1. UI Architecture Philosophy

**"The UI is Dumb. The UI is Reactive."**

1. **Passive View:** A Flutter widget or Flame component never calculates scoring math or modifies game state directly; it sends user intents to Riverpod notifiers or domain engines.
2. **Boundary Rule:** The 8×8 Board, Tray Pieces, Drag/Drop feedback, and line clear particle effects live inside Flame. HUD, Menus, Goal Cards, and Modals live in Flutter.
3. **Responsive Layout:** Flutter's `LayoutBuilder`, `SafeArea`, `Expanded`, and `MediaQuery` handle phone and tablet screen scaling.

---

## 2. Screen & Route Registry

Navigation uses `GoRouter` for top-level application screens:

| Route | Widget | Description |
|---|---|---|
| `/` | `HomeScreen` | Mode selection (Classic Endless vs. Stage Challenge), High Scores, Settings |
| `/classic` | `ClassicGameScreen` | Hosts `GameWidget` + Classic Endless HUD & Overlays |
| `/stages` | `StageSelectScreen` | Mode 2 Stage grid with star progression and level select |
| `/stage_play` | `StageGameScreen` | Hosts `GameWidget` + Goal-Driven Stage HUD & Overlays |
| `/settings` | `SettingsScreen` | Audio volume, SFX/Music toggles, Haptics preferences |

### Game-Local Overlays
In-game overlays are shown via Flutter `Stack` within `ClassicGameScreen` or `StageGameScreen` — they are NOT separate GoRouter routes:

| Overlay | Widget | Mode | Trigger |
|---|---|---|---|
| Pause | `PauseOverlay` | Both | Player taps pause button |
| Game Over | `GameOverOverlay` | Classic | Board locked with no valid moves |
| Stage Clear | `StageClearOverlay` | Stage | All stage objectives completed (1–3 stars awarded) |
| Stage Failed | `StageFailedOverlay` | Stage | Out of moves before completing objectives |

Overlays are driven by `ClassicGameService` and `StageGameService` state observed via Riverpod. They are rendered conditionally in the screen's `Stack`, not pushed onto the navigation history.

---

## 3. The Gameplay Screen Stack (Flutter + Flame)

The gameplay screens use Flutter's `Stack` to layer responsive UI over the Flame canvas:

```dart
Widget build(BuildContext context) {
  final sessionState = ref.watch(classicSessionProvider);

  return Scaffold(
    body: SafeArea(
      child: Stack(
        children: [
          // 1. Base Layer: Flame Engine (Board, Tray, VFX)
          GameWidget(game: blockGameInstance),
          
          // 2. HUD Layer: Flutter Widgets (Score, Streak Multiplier)
          IgnorePointer(
            ignoring: false, // Buttons receive taps; backdrop ignores
            child: ClassicHudOverlay(),
          ),
          
          // 3. Modal Layer: Pause, Game Over
          if (sessionState.isGameOver) GameOverOverlay(),
          if (isPaused) PauseOverlay(),
        ],
      ),
    ),
  );
}
```
*Note:* To prevent Flutter overlays from intercepting Flame touch and drag events, use `IgnorePointer` on non-interactive HUD elements.

---

## 4. Flame Render & Input (The Game World)

- **BoardView & TrayView:** Implemented as Flame `PositionComponent`s inside `FlameGame`.
- **Piece Dragging:** Flame's `DragCallbacks` mixin handles piece pickup, pointer tracking, and snapping.
- **Ghost Preview:** Highlights valid target cells on the 8×8 board when a piece is dragged overhead.
- **VFX & Particles:** Flame `ParticleSystemComponent` or sprite burst components spawn on line clear events.
- **Animations:** Flame effects (`MoveEffect`, `ScaleEffect`, `OpacityEffect`) animate cell clears and board clears.

---

## 5. Flutter UI Components (The HUD & Menus)

- **Classic Score HUD (`ScoreHudWidget`):** Displays current score, high score record, and active streak multiplier badge (e.g., $1.5\times$).
- **Stage Goal HUD (`StageGoalHudWidget`):** Displays remaining moves counter, target score progress bar, and color block collection quotas.
- **Star Display Widget (`StarRatingWidget`):** Displays 1 to 3 animated stars based on stage milestone thresholds.
- **Streak Multiplier Badge (`StreakBadgeWidget`):** Pulsing badge highlighting consecutive line clears.

---

## 6. Reactive Data Binding

We use Riverpod `ConsumerWidget` for declarative UI updates:

```dart
class ScoreHudWidget extends ConsumerWidget {
  const ScoreHudWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(classicSessionProvider);
    return Row(
      mainAxisAlignment: MainAxisAlignment.spaceBetween,
      children: [
        Text('SCORE: ${state.currentScore}'),
        Text('BEST: ${state.highestScore}'),
        if (state.streakCount > 1)
          Text('STREAK x${state.currentMultiplier.toStringAsFixed(1)}'),
      ],
    );
  }
}
```

---

## 7. Animation Boundary

- **UI Animation:** Flutter's `AnimatedWidget`, `AnimationController`, or implicit animations (`AnimatedContainer`, `AnimatedScale`).
- **Game World Animation:** Flame's `Effect` classes and particle engines.
- **Synchronization:** Presentation animations run independently and do not block domain state. After domain state transitions (e.g., `LinesClearedEvent` fired), the Flame visual components play particle bursts and grid collapse animations concurrently. Gameplay input is restored immediately following clear resolution.

---

## 8. Widget Naming Conventions

- Use PascalCase for all classes.
- Suffix full screens with `Screen` (e.g., `HomeScreen`, `ClassicGameScreen`).
- Suffix in-game modals with `Overlay` (e.g., `PauseOverlay`, `GameOverOverlay`, `StageClearOverlay`).
- Suffix reusable components with `Widget` (e.g., `ScoreHudWidget`, `StageGoalHudWidget`).

---

## 9. Safe Areas and Responsiveness

- Always wrap the root gameplay stack in a `SafeArea` widget to avoid device notches, home indicators, and camera cutouts.
- Use `LayoutBuilder` inside the `GameWidget` wrapper to scale the Flame camera dynamically while Flutter HUD elements pin to screen edges.

---

## 10. Localization & Typography

- Text uses standard Flutter `Text` widgets with custom font families declared in `pubspec.yaml`.
- All player-facing copy references keys from `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md` to ensure zero hardcoded strings.

---

**End of `23_UI_IMPLEMENTATION_ARCHITECTURE.md`.**
