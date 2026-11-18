# AI_START_HERE.md
## Block Battles — AI Orientation Document

**Read this file first. Always.**
This is the single entry point for any AI coding agent, language model, or automated tool interacting with this repository. It provides project orientation and authoritative document routing. Do not skip it.

---

## 1. Project Identity

**Name:** Block Battles (Pure Arcade Engine)

**One-sentence definition:** Block Battles is a tactile, premium mobile block-placement puzzle game featuring an endless Classic High-Score mode and an extensible Goal-Driven challenge architecture.

**One-paragraph definition:**
Block Battles fuses the deeply satisfying, tactile loop of an 8x8 block-placement puzzle (in the spirit of classic arcade block puzzles) with state-of-the-art audiovisual polish, physical game feel, and competitive local device scoring. The player places polyomino shapes from a 3-slot tray onto an 8x8 board. Completing full rows and columns clears lines, scores points, and triggers escalating combo streaks with rich particle VFX, screen shake, and melodic chimes. The app features an instant, clutter-free flow: App Icon → Animated Splash → Minimalist Home → Classic Mode (endless play competing against local device high scores until board lock) with architectural space reserved for Goal-Driven hurdle modes. Designed Android-first, offline, and solo-developer ready.

---

## 2. Core Game Loop (Visual)

```
[APP LAUNCH]
     │
     ▼
[ANIMATED SPLASH SCREEN] (Smooth branded animation)
     │
     ▼
[MINIMALIST HOME SCREEN] (Play Classic, High Score Trophy, Settings)
     │
     ├─────────────────────────────────────────┐
     ▼                                         ▼
[MODE 1: CLASSIC MODE (Active Focus)]     [MODE 2: GOAL-DRIVEN MODE (Reserved)]
     │                                         │
     ▼                                         ▼
┌─────────────────────────┐               ┌─────────────────────────┐
│ 8x8 PUZZLE BOARD        │               │ LEVEL OBJECTIVES        │
│ 3-Slot Shape Tray       │               │ Target Score, Hurdles   │
└────────────┬────────────┘               └─────────────────────────┘
             │
             ▼
   ┌───────────────────┐
   │ PLAYER TURN       │◄────────────────┐
   │ Pick piece (lift) │                 │
   │ Drag (ghost glow) │                 │
   │ Drop & Snap       │                 │
   └─────────┬─────────┘                 │
             │                           │
             ▼                           │
     Rows/Cols full?                     │
      YES │     NO │                     │
          │        └────────────────────►┤
          ▼                              │
  LINE CLEAR & JUICE                     │
  Simultaneous Row/Col Scan              │
  White-hot Flash + Particles            │
  Melodic Chime + Screen Shake           │
  Score + Combo Streak Multiplier        │
          │                              │
          ▼                              │
  Tray empty? ──YES──► Refill 3 Shapes   │
          │                              │
          ▼                              │
  Any legal placement for tray pieces?   │
     YES │      NO (Board Lock)          │
         └─────────────┬─────────────────┘
                       │
                       ▼
               [GAME OVER DIALOG]
          Final Score & High Score Check
          Celebratory New Best Fanfare
          One-Tap "Play Again" or "Home"
```

---

## 3. What Makes the Game Different

| Feature | What It Does |
|---|---|
| **Zero "AI Slop" Visuals** | Beveled, rich gradient block materials, frosted glass board frame, glowing ghost previews, and white-hot particle dissipation. |
| **Tactile Physical Game Feel** | Finger-occlusion lift, elastic drag curves, micro-snap haptics, and proportionate camera shake on multi-line clears. |
| **Pure Zen Endless Loop** | No timers, no stress, no energy gates. Pure spatial puzzle mastery and chasing your own device high score. |
| **Clutter-Free Minimalism** | Clean app launch: Splash screen straight into a sleek Home menu. No bloated menus, no enemy clutter, no confusing micro-transactions. |
| **Deterministic Fair RNG** | Board-aware shape generation ensures solvable, intelligent piece generation with duplicate prevention. |
| **Goal-Driven Architecture** | Cleanly decoupled underlying architecture allows hurdle-based level modes to be introduced without polluting the Classic mode. |

---

## 4. Core Player Experience

The game should feel: **zen, tactile, premium, satisfying — with moments of explosive arcade reward.**

The player should feel clever, not lucky. The board is a spatial playground. Placing a piece must feel physically *right* — an elastic pickup lift, a tactile snap, and a clean haptic pulse. A multi-line clear should feel like an explosive celebration. Losing should feel earned and immediately prompt a one-tap retry.

---

## 5. MVP Scope

### Included (Classic Mode Engine)
- 8x8 grid board with responsive mobile auto-scaling.
- 3-slot tray with deterministic `GameRNG` and solvability protection.
- 24 canonical playable shape definitions (SHP_001–SHP_024).
- Simultaneous line clearing (rows + columns) with single-clear intersection union.
- Pure puzzle scoring system (placement points + line clear points + multi-line bonuses + combo streak multiplier).
- Board Lock detection: exhaustively validates all 3 tray pieces across the board; triggers Game Over when zero moves remain.
- Local High Score persistence (`shared_preferences` / `hive_ce`).
- Animated Splash Screen & Minimalist Home Screen.
- Polished Game Over dialog with "New Best!" celebration and instant replay.
- Rich audiovisual juice: beveled block shaders, particle bursts, screen shake, audio chimes.
- Offline-first: 100% playable without an internet connection.

### Excluded / Retired from Scope
- **RETIRED:** Enemy HP, combat attacks, damage formulas, and boss fights.
- **RETIRED:** Relic collection and draft systems.
- **RETIRED:** Dungeon floor progression runs.
- **EXCLUDED:** Cloud Save or external server backend.
- **EXCLUDED:** Energy / stamina timers.
- **EXCLUDED:** Pay-to-win mechanics.

### Reserved (Future Goal-Driven Mode)
- Level-based objective hurdles (reach target score, clear frozen obstacles, move limits) built on top of the same board engine.

---

## 6. Core Rules (Non-Negotiable)

| Rule | Value |
|---|---|
| Board Size | 8×8 grid |
| Tray Slots | 3 (refills when all 3 placed) |
| Piece Rotation | NONE. Pieces have a fixed orientation. |
| Line Clear | Full row OR full column clears. Both clear simultaneously. |
| Intersection Rule | Intersection cells between cleared rows and cols are cleared exactly once. |
| Scoring System | Block Placement Points + Line Clear Points + Multi-Clear Bonus + Streak Multiplier. |
| Game Over | Triggered when none of the available tray pieces has any legal placement. |
| High Score | Persisted locally on device; evaluated on game over. |
| Input | Touch drag with finger-lift offset and live validity ghost preview. |

All numeric values (BaseDamage, multiplier tables, Stardust rates) are authored exclusively in design documents. **Never hardcode them into logic.**

---

## 7. Architecture Summary

### Data-Driven Design
Enemies, Relics, Shapes, and Objectives are Dart const data classes. Adding content requires authoring data assets, not modifying pure logic. See `20`.

### RNG & Determinism
A seeded `GameRNG` class controls all piece spawns, enemy selections, and relic drops.
- **New Runs:** Deterministic from seed + inputs (the same seed + input sequence always produces the same game state).
- **Resumable Active Runs:** Restore exact serialized runtime state including deterministic RNG continuation (the exact number of RNG calls consumed), not just the seed. See `04`, `21`, and `24`.

### Event-Driven Communication
Domain systems (Board, Combat, Enemy) communicate via a type-safe `EventBus<T>`. Presentation systems (UI, Audio, VFX) listen to events and react. The Domain NEVER calls into Presentation. See `22`.

### Presentation Separation (Clean Architecture)
- **Domain Layer** (Pure Dart): Board math, damage formulas, RNG.
- **Application Layer** (Orchestration): Turn state machine, Run management.
- **Presentation Layer** (Flutter/Flame): Flame components, Flutter widgets, GoRouter routes.
- Dependencies point **inward only**. Domain knows nothing about Flutter or Flame. See `18`.

### Persistence
Local save data managed via Hive CE with atomic writes. Player data never corrupts on crash. See `24`.

---

## TECHNOLOGY STACK
| Concern | Technology |
|---------|------------|
| Language | Dart |
| App Framework | Flutter |
| Game Engine | Flame |
| State Management | Riverpod |
| Navigation | GoRouter |
| Local Save Data | Hive CE |
| Lightweight Preferences | SharedPreferences |
| Game Audio | Flame Audio |
| IDE | Android Studio |
| Platform | Android |
| Testing | dart test / flutter test / integration_test |
| Release | flutter build appbundle |

---

## ARCHITECTURE LAYERS (ONE-SENTENCE RULE)
| Layer | Rule |
|-------|------|
| lib/domain/ | Pure Dart only. ZERO imports from flutter, flame, riverpod, or hive packages. |
| lib/data/ | Data persistence and content definitions. May use Hive CE. NO Flame or Flutter UI imports. |
| lib/application/ | Orchestration and use cases. May use Riverpod. Pure Dart logic. |
| lib/game/ | Flame components and game world. Imports Flame. NO direct domain mutation. |
| lib/presentation/ | Flutter widgets, screens, overlays. Imports Flutter + Riverpod. |

---

## 8. Major Runtime Systems

| System | Responsibility |
|---|---|
| `BoardModel` | Owns the 8x8 cell-state array (compact occupancy + modifier). Validates and commits placements. |
| `TrayEngine` | Manages the 3 active pieces. Requests new pieces from `GameRNG`. |
| `LineScanner` | Detects and clears full rows/columns simultaneously after each placement. |
| `ScoreEngine` | Calculates placement points, line clears, multi-line bonuses, and combo streak multipliers. |
| `BoardLockDetector` | Exhaustively checks remaining tray pieces against board; triggers Game Over when zero moves remain. |
| `SaveService` | Reads/writes local High Score and preferences (Hive CE / SharedPreferences). |
| `AudioService` | Flame Audio wrapper. Manages placement pops, line clear chimes, combo risers. |
| `VFXManager` | Flame particle/VFX spawners. Fires white-hot line flash, particle bursts, and camera shake. |
| `ClassicGameService` | Pure puzzle game lifecycle orchestrator (New Game, Active Turn, Board Lock, Replay). |

---

## 9. Canonical Terminology

| Term | Definition |
|---|---|
| **L-Value** | The number of lines cleared by a single piece placement (1 to 4+). |
| **Tray** | The 3-slot panel holding the current available pieces. |
| **Board Lock** | Mathematical condition where none of the remaining tray pieces can legally be placed anywhere on the board. |
| **Game Over** | Player-facing failure presentation triggered by Board Lock. |
| **High Score** | The all-time highest score achieved on this local device. |
| **Ghost Preview** | The transparent indicator showing where a dragged piece will land (valid green, invalid red). |
| **Combo / Streak** | Consecutive line clears across turns triggering escalating score multipliers. |
| **ShapeData** | A Dart const defining a piece's normalized coordinates and bounding box. |

---

## 10. Source-of-Truth Rules

| Decision | Authority |
|---|---|
| Any gameplay rule | `01_GAMEPLAY_SPECIFICATION.md` |
| Any shape geometry | `02_SHAPE_LIBRARY.md` |
| Any combat formula | `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` |
| Any color value | `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` |
| Any animation timing | `12_ANIMATION_MOTION_AND_GAME_FEEL.md` |
| Any copy/text string | `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md` |
| Any architecture pattern | `18_TECHNICAL_ARCHITECTURE.md` |
| Any folder/file location | `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md` |
| Any data schema | `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md` |
| Any event definition | `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md` |
| Any save field | `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md` |
| Any naming convention | `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` |
| Any AI behavior rule | `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md` |

**Conflict Resolution:** If existing code disagrees with a document, the document is correct. Fix the code.

---

## COMMON AI MISTAKES TO AVOID
1. Creating Flame components that contain game rules
2. Importing flutter or flame packages into lib/domain/
3. Using setState() or StatefulWidget for game state (use Riverpod)
4. Using Flutter GestureDetector for board piece dragging (use Flame DragCallbacks)
5. Implementing runtime piece rotation (no rotation exists - orientation is static)
6. Calling runApp or WidgetsBinding methods outside main.dart
7. Adding packages to pubspec.yaml without documenting in Phase notes
8. Forgetting to run flutter analyze before declaring work complete
9. Ignoring the document hierarchy (game design docs are authoritative)
10. Inventing mechanics not in Layer A/B docs

---

## 11. Forbidden Behaviors

An AI working on this project MUST NEVER:
1. Write `if (relicName == "IronMallet")` in core logic. Use enum + data.
2. Put game logic in a UI script/widget (`Widget`, `StatelessWidget`, etc.).
3. Use Flutter physics for board placement.
4. Call `add()` or `removeFromParent()` arbitrarily during the combat turn loop outside Flame's lifecycle (`onLoad`, `update`, `render`).
5. Use `dart:io` in domain logic during gameplay.
6. Create a `GameManager.dart` that owns more than one responsibility.
7. Change Dart package import rules / ignore architecture layers.
8. Invent a mechanic, balance value, or system not specified in the docs.
9. Wrap a bug in `try { } catch { }` without logging and fixing the root cause.
10. Push to `main` without passing the `dart test` suite.
11. Implement a generalized hazard/special-cell framework during MVP unless a later approved document explicitly activates it. MVP board states are strictly limited to `EMPTY`, `FILLED`, `BLOCKED`, and `FROZEN`. `HAZARDOUS`, `INDESTRUCTIBLE`, and `SPECIAL` are explicitly deferred.
12. Implement board logic using raw single-state checks like `if (cell.state == EMPTY)`. The board uses a two-axis model (Occupancy + Modifier). All placement and clearing logic must use semantic predicates like `canReceiveBlock(cell)` and `countsTowardLineClear(cell)`.

---

## 12. Development Philosophy

- **Read the spec first.** The Markdown documentation is the source of truth.
- **Make surgical changes.** Fix only what was asked. No unrelated refactors.
- **Data over code.** New enemies and relics are data. New mechanics are not added casually.
- **Test after every Domain change.** If you touched the Board, Combat, or Relics — run the `dart test` suite.
- **Ask, don't guess.** If a value is missing from a document, ask the developer. Do not invent defaults.
- **No speculative complexity.** Do not build `AbstractManagerFactoryBase<T>` for a solo mobile game.

---

## PROJECT WORKSPACE
- Documentation: E:\Block Buster (this folder)
- Game project: E:\block battles (when it exists)
- GitHub: https://github.com/kmkbuild/Block-Battles.git

---

## 13. Current MVP Content Target

| Category | MVP Target |
|---|---|
| Shapes | 24 canonical playable shapes (SHP_001–SHP_024) |
| Game Modes | Mode 1: Classic Endless (Active Focus) + Mode 2: Goal-Driven (Reserved Architecture) |
| Screens | 3 core flows (Animated Splash Screen, Minimalist Home, Classic Game with Game Over dialog) |
| Audio SFX | ~12 tactile clips (Pickup pop, grid snap, line clear chime chord, combo riser, game over fanfare) |
| Visual Polish | Beveled block textures, glowing glassmorphic board, particle bursts, screen shake |

---

## 14. Build Order Summary: 5 Master Phases

The project executes in **5 high-impact Master Phases** designed for immediate visual excellence:

1. **Phase 1: Pure Engine Foundations** — 8x8 BoardModel, 24 Canonical Shapes, Deterministic GameRNG, 3-slot TrayEngine with solvability protection.
2. **Phase 2: Tactile Visual Presentation & Drag-and-Drop UX** — Beveled block materials, frosted glass board frame, touch pickup lift, live ghost preview, snap haptics.
3. **Phase 3: Explosive Line Clearing & Game Feel Juice** — Simultaneous row/col scanning, white-hot flash, particle bursts, screen shake, audio chimes.
4. **Phase 4: Classic Endless Mode, Score Engine & Local High Score** — Pure arcade scoring, combo streak multiplier, Board Lock detection, local high-score persistence, Game Over celebratory dialog.
5. **Phase 5: App Shell, Splash Screen, Minimalist Home & Goal-Driven Hooks** — Animated Splash screen, clutter-free Home Screen, audio/haptic toggles, Goal-Driven mode data contracts.

---

## 15. Where to Look

| Question | Read |
|---|---|
| What is the overall game vision? | `00_MASTER_GAME_VISION.md` |
| Gameplay rules, piece behavior? | `01_GAMEPLAY_SPECIFICATION.md` |
| Shape geometry and matrices? | `02_SHAPE_LIBRARY.md` |
| Board mechanics and edge cases? | `03_BOARD_ENGINE_AND_RULES.md` |
| RNG, spawning, difficulty? | `04_SPAWN_RNG_AND_DIFFICULTY.md` |
| Damage formulas, combos, scoring? | `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` |
| Drag-and-drop input behavior? | `06_INPUT_DRAG_AND_DROP_UX.md` |
| Objectives and level design? | `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md` |
| UI screens and flows? | `08_UI_UX_MASTER_SPECIFICATION.md` |
| Visual design system? | `09_VISUAL_DESIGN_SYSTEM.md` |
| Color palette and theming? | `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` |
| Block art and material spec? | `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md` |
| Animation timings and feel? | `12_ANIMATION_MOTION_AND_GAME_FEEL.md` |
| Particle and VFX definitions? | `13_VFX_PARTICLE_AND_IMPACT_SPEC.md` |
| Sound effects and music? | `14_AUDIO_SFX_MUSIC_AND_MIXING.md` |
| Player-facing copy and text? | `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md` |
| Settings and accessibility? | `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` |
| Monetization and ads? | `17_MONETIZATION_REWARD_AND_AD_UX.md` |
| Overall engineering architecture? | `18_TECHNICAL_ARCHITECTURE.md` |
| Folder structure, screens, Flame components? | `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md` |
| Content schemas and definitions? | `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md` |
| System managers and ownership? | `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md` |
| EventBus, state machine, events? | `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md` |
| Flutter UI implementation? | `23_UI_IMPLEMENTATION_ARCHITECTURE.md` |
| Save data and persistence? | `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md` |
| Asset naming and pipeline? | `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` |
| Performance and optimization? | `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md` |
| Testing and QA strategy? | `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md` |
| Build, signing, and release? | `28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md` |
| AI coding rules and behavior? | `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md` |
| Implementation phase order? | `30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md` |

---

## 16. Final AI Instruction

> **Read this file first.**
> Then read ONLY the documents relevant to your current task.
> Never invent a rule, value, or system that already has a documented owner.
> If a specification is missing, ask the developer.
> When in doubt: be surgical, be explicit, be testable.

---

**End of `AI_START_HERE.md`.**
