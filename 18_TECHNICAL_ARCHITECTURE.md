# 18_TECHNICAL_ARCHITECTURE.md
## Block Battles — Technical Architecture (Pure Arcade Engine)

**Governing documents:** All Layer A (`01`-`07`) and Layer B (`08`-`17`) documents.  
**Document status:** Layer C — Engineering Authority (Primary)  
**Core Directive:** Decoupled, deterministic, pure-Dart puzzle domain; tactile, high-performance Flame game loop; clean Flutter app shell. Pure spatial puzzle architecture.

---

## 1. Technical Purpose

The purpose of this architecture is to build a robust, deterministic, tactile, and easily testable foundation for Block Battles. It acts as the engineering contract for all code generation, ensuring that logic remains decoupled from presentation, bugs are easily isolated, and performance remains locked at 60fps on mobile devices.

---

## 2. Architectural Goals

- **Strict Separation of Concerns:** Gameplay logic is 100% pure Dart, compiling and running without any Flutter or Flame dependencies.
- **Determinism:** The same player input and the same random seed ALWAYS produce the exact same board state, tray pieces, and score outcome.
- **AI-Readability:** Classes are small, single-responsibility, and explicitly bounded (under 300 lines) so AI coding agents can generate, refactor, and review them reliably.
- **Tactile Juice Priority:** Presentation components are structured to deliver immediate visual and acoustic feedback (ghost preview snapping, 48dp finger-lift offset, line-clear particle bursts, pitch escalation).
- **Smooth 60fps Performance:** Zero garbage-collection stutter in the core drag and clear loops.

---

## 3. High-Level System Map

```text
[ PLATFORM LAYER ]
   └── Save/Load IO (Hive CE for Game Progress & Sessions; SharedPreferences for Settings)

[ FLUTTER UI LAYER (App Shell & Overlays) ]
   ├── Screens: SplashScreen, HomeScreen, ClassicGameScreen, StageSelectScreen, StageGameScreen, SettingsScreen
   ├── Overlays: PauseOverlay, GameOverOverlay, StageClearOverlay, StageFailedOverlay
   └── State Wiring: Riverpod Providers

[ FLAME PRESENTATION LAYER (Game View & Feel) ]
   ├── BlockBattlesFlameGame (Main engine bridge)
   ├── BoardComponent & CellComponent (8x8 visual grid, bevel highlights)
   ├── TrayComponent & PieceComponent (Draggable shapes, 48dp lift offset)
   ├── GhostPreviewComponent (Grid snapping, valid green / invalid red)
   ├── VFX & Animation: LineClearBurst, WhiteFlashEffect, CameraShake
   └── Audio & Haptics: AudioManager (Pitch escalation chimes, haptic pulses)

[ APPLICATION LAYER (Orchestration & Services) ]
   ├── ClassicGameService (Turn loop orchestrator)
   ├── GoalDrivenGameService (Reserved Mode 2 stage orchestrator)
   ├── HighScoreService (Local device record persistence)
   └── SettingsService (Audio & haptic preferences)

[ DOMAIN LAYER (Pure Dart Rules & Math) ]
   ├── BoardModel (8x8 grid matrix, bounds checks, placement commits)
   ├── LineClearDetector (Simultaneous row/column full checks)
   ├── ShapeCatalog & ShapeDefinition (Immutable piece geometries)
   ├── TrayEngine (3-piece batch replenishment)
   ├── ScoreEngine (Placement points, line clear bonuses, streak multipliers)
   ├── BoardLockDetector (Exhaustive valid-move evaluation)
   ├── GameRng (Seeded deterministic random number generator)
   └── LevelDefinition (Reserved Mode 2 hurdle schemas)
```

---

## 4. Layering Model (Clean Architecture)

### 4.1 Domain Layer (The Core)
- **What it is:** Pure Dart files (`.dart`). Absolutely zero references to `flutter` or `flame` packages.
- **Responsibility:** Contains the 8×8 grid matrix, piece placement validation, simultaneous row/column clear detection, scoring formulas, streak logic, and Board Lock detection.
- **Rule:** Static piece orientations only. Runtime rotation is strictly prohibited.
- **Testing:** 100% testable via lightning-fast `dart test`.

### 4.2 Application Layer (The Orchestrator)
- **What it is:** Pure Dart controllers and services.
- **Responsibility:** Coordinates turn transitions: receives placement input $\to$ calls `BoardModel` $\to$ resolves line clears $\to$ calculates score $\to$ replenishes tray $\to$ evaluates Board Lock $\to$ updates High Score.

### 4.3 Flame Presentation Layer (The Game View)
- **What it is:** Flame `Component` classes, Effects, and audio controllers.
- **Responsibility:** Renders the board, animates piece dragging with a 48dp upward offset, draws snapping ghost previews, spawns line-clear explosion particles, and plays sound effects.
- **Rule:** The Presentation layer observes Domain state and emits input intents; it NEVER mutates Domain state directly.

### 4.4 Flutter UI Layer (The App View)
- **What it is:** Flutter Widgets, Riverpod providers, and Material dialogs.
- **Responsibility:** Splash screen animations, Minimalist Home menu, top HUD bar, Game Over modal, and Settings toggles.

---

## 5. Dependency Direction

**Dependencies always point INWARD:**
- `Flutter UI` $\to$ `Riverpod`, `Application`, `Domain`
- `Flame Presentation` $\to$ `Application`, `Domain`
- `Application` $\to$ `Domain`
- `Domain` $\to$ **NOTHING** (Pure Dart standard library only)

---

## 6. Runtime Lifecycle

1. **Boot:** Initialize local storage (Hive CE for game progression and active sessions; SharedPreferences for settings). Load all-time High Score from Hive CE.
2. **Splash Screen:** Display 1.2s animated logo $\to$ Transition to Home.
3. **Home Screen:** Display High Score trophy card $\to$ Player taps [ PLAY CLASSIC ].
4. **Classic Game Session:**
   - Mount `BlockBattlesFlameGame` via Flutter's `GameWidget`.
   - Seed `GameRng` and populate initial 3-piece Tray via `TrayEngine`.
   - Clear 8×8 `BoardModel` to all-empty.
   - Enter `ACTIVE` state.
5. **Turn Execution Loop:**
   - Player drags piece from tray (48dp offset, live snapping ghost preview).
   - Valid drop committed $\to$ `PlacementCommitted`.
   - Simultaneous row/col clears evaluated $\to$ `LinesCleared`.
   - `ScoreEngine` calculates placement points, clear points, and streak multiplier $\to$ `ScoreUpdated`.
   - If tray is empty $\to$ `TrayEngine` generates 3 new pieces.
   - `BoardLockDetector` performs exhaustive check across remaining tray pieces.
     - Legal move exists $\to$ Return to `ACTIVE`.
     - Zero legal moves exist $\to$ Trigger `BoardLockDetected`.
6. **Game Over Resolution:**
   - Board visually greys out.
   - If `Score > HighScore`, persist new High Score and flag celebratory state.
   - Show `GameOverOverlay` with "Play Again" and "Home" actions.

---

## 7. Determinism Strategy

1. **Seeded RNG:** All piece generation is controlled by a single seeded `GameRng` class.
2. **Reproducibility:** Given the same initial seed and the same sequence of piece selections and coordinates, the game state and score are 100% reproducible.
3. **No Unseeded Randoms:** `dart:math.Random()` is forbidden in domain logic without a seed.

---

## 8. Anti-Pattern Catalog

1. **No God Classes:** `BoardModel` does not play audio or draw widgets.
2. **No Business Logic in UI/Flame:** Flame touch callbacks do not compute score; they forward coordinates to `ClassicGameService`.
3. **No Hardcoded Magic Numbers:** Grid dimensions, combo multipliers, and point tables belong in `game_constants.dart`.
4. **No Visual Blocking of Logic:** Gameplay math resolves instantaneously and synchronously; animations play concurrently.

---

## 9. Technology Stack

| Concern | Technology |
|---|---|
| Language | Dart |
| App Framework | Flutter |
| Game Engine | Flame |
| State Management | Riverpod |
| Local Persistence | Hive CE (Game Progress & Sessions) + SharedPreferences (User Settings) |
| Audio | Flame Audio |
| Target Platform | Android |
| Testing | dart test & flutter test |

---

**End of `18_TECHNICAL_ARCHITECTURE.md`.**
