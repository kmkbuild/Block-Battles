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
│   │   └── enemies/
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
│   │   ├── combat/
│   │   ├── enemies/
│   │   ├── objectives/
│   │   ├── relics/
│   │   ├── run/
│   │   └── rng/
│   ├── data/
│   │   ├── definitions/
│   │   └── persistence/
│   ├── application/
│   ├── presentation/
│   │   ├── home/
│   │   ├── battle/
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

The app uses `GoRouter` for strictly defined, deep-linkable paths. The MVP requires these core routes:

1. `/` — Home Screen (Main Menu)
2. `/battle` — Core Gameplay Screen (Hosts `GameWidget`)
3. `/settings` — App Settings Menu

**Battle-local Overlays (NOT GoRouter routes):**
Battle specific UI such as Pause Menu, Relic Selection, Victory, and Defeat screens are **Flutter Overlays** stacked on top of the `GameWidget` within the `/battle` route, NOT separate GoRouter routes.

---

## 5. Home Screen Architecture (`/`)

- **Purpose:** Handles the Main Menu, run continuation, and meta-progression.
- **Widgets:** Standard Flutter UI (`Scaffold`, `Column`, `ElevatedButton`).
- **Flow:** When the player taps "Start Run", a Riverpod controller generates the Run Seed, instantiates the `RunModel`, saves to Hive, and calls `context.go('/battle')`.

---

## 6. Battle Screen Architecture (`/battle`)

- **Purpose:** The core game loop container. 
- **Widgets:** 
  - A `Scaffold` containing a `Stack`.
  - The bottom layer of the Stack is `GameWidget<BlockBattlesGame>`.
  - The top layers are reactive Flutter overlays (HUD, Score, Pause button) listening to Riverpod providers.
- **Flow:** `BlockBattlesGame` (the Flame game engine) is instantiated and mounted. It runs the battle logic. When the battle ends, it communicates back to Riverpod, which triggers the Flutter Relic Selection overlay.

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
        path: '/battle',
        builder: (context, state) => const BattleScreen(),
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

In `battle_screen.dart`, the overlay ensures UI is easy to build with standard Flutter widgets:

```text
Stack
├── GameWidget(game: BlockBattlesGame())
├── Positioned(top) -> HUDWidget (Score, HP, Enemy Telegraph)
├── Positioned(bottom) -> ActionButtons (Pause, Powerups)
└── RiverpodConsumer -> Conditional Overlays (PauseMenu, RelicSelection, GameOver)
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
> This project structure ensures that when a bug occurs in the relic system, the developer knows exactly which folder to open. When the UI needs a tweak, the core game loop is entirely insulated from accidental breakage. 
>
> By enforcing pure Dart domain boundaries and banning Flame physics for grid logic, we guarantee that AI assistants can write clean, testable logic without needing to manipulate a visual inspector.

---

**End of `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`.**
