# 19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md
## Block Battles — Flutter/Flame Project Structure & App/Game Architecture

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md` (and all prior Layer A & B docs).
**Document status:** Layer C — Engineering Authority (Primary)

**Inheritance confirmation:** This document physically enforces the architecture defined in Doc 18. It defines the exact folder structure, app routing, and Flame component paradigms to ensure the solo-developer and AI-assisted workflows remain organized, scalable, and decoupled.

---

## 1. Flutter/Dart SDK Policy

**Project Standard:**
- **SDK:** Flutter channel: stable. Use the current stable Flutter SDK available at the time of project initialization.
- **Language:** Dart (with strict null safety)
- **Target Platform:** Android (primary), iOS (secondary)
- **Build Type:** Production Mobile Release (App Bundle)
- **Project Policy:** Once development begins, Flutter SDK version remains locked in `pubspec.yaml` via standard version constraints unless a critical production issue requires an upgrade.

---

## 2. Project Folder Tree

```text
block_battles/
├── android/
├── assets/
│   ├── images/
│   ├── sprites/
│   │   ├── board/
│   │   ├── ui/
│   │   └── goals/
│   ├── audio/
│   │   ├── music/
│   │   └── sfx/
│   ├── fonts/
│   ├── effects/
│   └── data/
├── lib/
│   ├── main.dart
│   ├── app/
│   │   ├── app.dart
│   │   └── app_router.dart
│   ├── core/
│   │   └── game_constants.dart
│   ├── domain/
│   │   ├── board/
│   │   ├── shapes/
│   │   ├── tray/
│   │   ├── scoring/
│   │   ├── objectives/
│   │   ├── game_loop/
│   │   └── rng/
│   ├── data/
│   │   ├── definitions/
│   │   └── persistence/
│   ├── application/
│   ├── presentation/
│   │   ├── home/
│   │   ├── game/
│   │   ├── settings/
│   │   └── widgets/
│   └── game/
│       ├── block_battles_game.dart
│       ├── components/
│       ├── systems/
│       ├── input/
│       └── effects/
├── test/
├── integration_test/
├── pubspec.yaml
└── analysis_options.yaml
```

**Rule:** Domain layers (`lib/domain/`, `lib/application/`, `lib/data/`) must never import `package:flutter/material.dart` or `package:flame/game.dart`.

---

## 3. App Initialization Flow

The application boot sequence ensures all persistent services are ready before a single pixel is drawn:

1. **`main.dart` execution:**
   - `WidgetsFlutterBinding.ensureInitialized()`
   - Initialize Hive CE (`Hive.initFlutter()`, register adapters).
   - Initialize SharedPreferences.
   - Load critical `FlameAudio` cached sounds.
2. **State Injection:**
   - Wrap the root app in Riverpod's `ProviderScope`, overriding any uninitialized sync providers with their loaded instances.
3. **App Mounting:**
   - Run `MaterialApp.router` (defined in `app.dart`), passing the `GoRouter` configuration.
4. **First Frame:**
   - `GoRouter` resolves the initial route `/` (Home Screen).

---

## 4. Screen/Route Inventory

The app uses `GoRouter` for strictly defined, deep-linkable paths corresponding to the two-mode architecture:

1. `/` — `HomeScreen` (Main Menu, Mode Selection: Classic & Stages, High Score Trophy, Settings)
2. `/classic` — `ClassicGameScreen` (Mode 1 Endless: hosts `GameWidget` + Classic HUD & Overlays)
3. `/stages` — `StageSelectScreen` (Mode 2 Stage Challenge: grid of stages, star progress $1\dots 3\star$, level select)
4. `/stage_play` — `StageGameScreen` (Mode 2 Gameplay: hosts `GameWidget` + Stage HUD with move counter and color objectives)
5. `/settings` — `SettingsScreen` (Audio volume, SFX/Music toggles, Haptics preferences)

**Game-local Overlays (NOT GoRouter routes):**
Gameplay UI such as Pause, Game Over, and Stage results are **Flutter Overlays** rendered conditionally in a `Stack` on top of the `GameWidget` within `ClassicGameScreen` or `StageGameScreen`, NOT separate GoRouter routes:
- `PauseOverlay` (Both modes: resume, restart, settings, quit to home)
- `GameOverOverlay` (Classic mode: board locked, final score tally, new best celebration)
- `StageClearOverlay` (Stage mode: objectives complete, 1–3 stars awarded, next level)
- `StageFailedOverlay` (Stage mode: out of moves before completing objectives)

---

## 5. Home Screen Architecture (`/`)

- **Purpose:** Handles the Main Menu, mode selection (Classic Endless or Stage Challenges), session continuation, and settings navigation.
- **Widgets:** Standard Flutter UI (`Scaffold`, `Column`, `ElevatedButton`).
- **Flow:** When the player taps "Play Classic", navigation calls `context.go('/classic')`. When the player taps "Stage Mode", navigation calls `context.go('/stages')`.

---

## 6. Gameplay Screen Architecture (`/classic` and `/stage_play`)

- **Purpose:** The core game loop container for both modes. 
- **Widgets:** 
  - A `Scaffold` containing a `Stack`.
  - The bottom layer of the Stack is `GameWidget<BlockBattlesGame>`.
  - The top layers are reactive Flutter HUD widgets (`ClassicHudOverlay` for Classic Mode; `StageHudOverlay` for Stage Mode) and conditional modal overlays (`PauseOverlay`, `GameOverOverlay`, `StageClearOverlay`, `StageFailedOverlay`) listening to Riverpod providers.
- **Flow:** `BlockBattlesGame` (the Flame game engine) is instantiated and mounted. It runs the pure puzzle board and drag logic. When a round ends (Board Lock or Stage Completion/Failure), it communicates via the Event Bus / Riverpod, triggering the corresponding overlay.

---

## 7. GoRouter Route Definitions

Routing must be centralized in `app_router.dart`:

```dart
final goRouterProvider = Provider<GoRouter>((ref) {
  return GoRouter(
    initialLocation: '/',
    routes: [
      GoRoute(
        path: '/',
        builder: (context, state) => const HomeScreen(),
      ),
      GoRoute(
        path: '/classic',
        builder: (context, state) => const ClassicGameScreen(),
      ),
      GoRoute(
        path: '/stages',
        builder: (context, state) => const StageSelectScreen(),
      ),
      GoRoute(
        path: '/stage_play',
        builder: (context, state) => const StageGameScreen(),
      ),
      GoRoute(
        path: '/settings',
        builder: (context, state) => const SettingsScreen(),
      ),
    ],
  );
});
```

---

## 8. Flame Component Hierarchy

Inside `BlockBattlesGame` (extending `FlameGame`), the component tree must remain shallow and semantic:

```text
BlockBattlesGame
├── BackgroundComponent
├── BoardComponent
│   └── CellComponent (x64)
├── TrayComponent
│   └── PieceComponent (x3)
└── VFXManagerComponent
    └── ParticleSystemComponent
```

---

## 9. Flutter Widget Hierarchy (HUD)

In `classic_game_screen.dart` and `stage_game_screen.dart`, the overlay ensures UI is easy to build with standard Flutter widgets:

```text
Stack
├── GameWidget(game: BlockBattlesGame())
├── Positioned(top) -> Mode-Specific HUD (Classic: Score, High Score, Streak Badge; Stage: Goals, Moves)
├── Positioned(topRight) -> ActionButtons (Pause)
└── RiverpodConsumer -> Conditional Overlays (PauseOverlay, GameOverOverlay, StageClearOverlay, StageFailedOverlay)
```

---

## 10. Component Ownership Rules

- **Flutter UI** owns meta-game state displays, menus, text rendering, and screen routing.
- **FlameGame** owns the 60fps render loop, coordinate spaces, board rendering, and drag interactions.
- **Domain/Application** owns the mathematical board array and turn rules. Flame merely observes the Domain layer via Dart streams/callbacks.

---

## 11. Rendering Layers (Flame Priority)

Use Flame's `priority` property to control render order between components. Higher priority renders on top.

1. `Background` (priority: 0)
2. `Board` (priority: 10)
3. `PlacedPieces` (priority: 20)
4. `VFX_Behind` (priority: 25)
5. `DraggedPiece` (priority: 30) - Ensures dragged pieces float over everything.
6. `VFX_Front` (priority: 40)
7. `Flutter UI` (Handled by Flutter's `Stack` z-index above `GameWidget`).

---

## 12. Input Policy

- **Board Interaction:** `PieceComponent` must mix in `DragCallbacks` provided by Flame to handle moving blocks in the game world.
- **UI Interaction:** All buttons, pauses, and menus use standard Flutter `GestureDetector` or `ElevatedButton`. 
- **Rule:** Flame collision detection is NOT used to check if a block is placed on a valid grid spot. Drag drop coordinates are mathematically transformed to an 8x8 grid index, which is verified against the Domain layer's array.

---

## 13. Asset Loading

- Assets must be explicitly declared in `pubspec.yaml` under the `flutter: assets:` section.
- Flame caches images via `Flame.images.loadAll()` and audio via `FlameAudio.audioCache.loadAll()` during the `onLoad` phase of the game or the boot sequence.

---

## 14. Resource Lifetime in Flame

- **Initialization:** Use `BlockBattlesGame.onLoad()` to pre-load sprites and initialize the component tree.
- **Cleanup:** `GameWidget` handles lifecycle automatically. When `context.go('/')` removes the `GameWidget` from the tree, `onRemove()` is called on components, clearing memory.

---

## 15. Navigation Rules

- **Between Screens:** Use `GoRouter.push()` or `GoRouter.go()`.
- **In-Game Menus:** Toggle booleans in Riverpod to display Flutter widgets in the `Stack` above the `GameWidget`. Do NOT route to a new screen for a pause menu.

---

## 16. Analysis Options & Lints

The `analysis_options.yaml` must enforce strict typing and best practices:
- `include: package:flutter_lints/flutter.yaml`
- Enforce `always_declare_return_types`
- Enforce `prefer_const_constructors`
- Enforce `avoid_print` (use a structured logging package or custom logger).

---

## 17. Build Validation Checklist

Before a commit or build, these commands must pass:
- [ ] `flutter analyze` (Zero warnings/errors)
- [ ] `flutter test` (All domain unit tests and widget tests pass)
- [ ] `flutter build appbundle --release` (Successfully compiles)

---

## 18. Final Architecture Contract

> **Organized storage yields organized thought.**
>
> This project structure ensures that when a bug occurs in the scoring or tray system, the developer knows exactly which folder to open. When the UI needs a tweak, the core game loop is entirely insulated from accidental breakage. 
>
> By enforcing pure Dart domain boundaries and banning Flame physics for grid logic, we guarantee that AI assistants can write clean, testable logic without needing to manipulate a visual inspector.

---

**End of `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`.**
