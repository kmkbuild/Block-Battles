# 18_TECHNICAL_ARCHITECTURE.md
## Block Battles — Technical Architecture

**Governing documents:** All Layer A (`01`-`07`) and Layer B (`08`-`17`) documents.
**Document status:** Layer C — Engineering Authority (Primary)

**Inheritance confirmation:** This document translates the mechanical, visual, and UX requirements into a concrete Flutter/Flame/Dart software architecture. It prioritizes the solo-developer constraint (`00`), AI-assisted development paradigms, and long-term maintainability over over-engineered enterprise patterns.

---

## 1. Technical Purpose

The purpose of this architecture is to build a robust, deterministic, and testable foundation for Block Battles. It acts as the "constitution" for all subsequent code generation, ensuring that logic remains decoupled from presentation, bugs are easily isolated, and new content (Relics, Enemies, Shapes) can be added purely via data.

## 2. Architectural Goals

- **Strict Separation of Concerns:** Gameplay logic must compile and run even if the Flame rendering engine and Flutter UI are disabled.
- **Determinism:** The same player input and the same random seed must ALWAYS produce the exact same board state and combat outcome.
- **AI-Readability:** Classes must be small, single-responsibility, and explicitly bounded so AI coding agents can generate, refactor, and review them without hallucinating dependencies.
- **Data-Driven Scalability:** Adding a new Enemy or Relic should require zero changes to core systems, only the creation of a new Dart data class configuration.
- **High Performance:** Maintain a rigid 60fps on low-end Android devices using Flutter and Flame optimizations.

## 3. Architectural Non-Goals

- **Enterprise Over-engineering:** We will not use bloated frameworks that add massive learning curves and obscure control flow from AI agents unless strictly necessary.
- **Multiplayer/Network Sync:** The architecture does not need to support rollback netcode or server authority.
- **Physics-Driven Gameplay:** The board relies on discrete mathematical arrays, not Flame physics or Forge2D.

---

## 4. High-Level System Map

```text
[ PLATFORM LAYER ]
   ├── AnalyticsService
   ├── MonetizationService
   └── Save/Load IO (Hive CE)

[ FLUTTER UI LAYER (App & Menus) ]
   ├── Screens (Home, Settings, Overlays)
   ├── Riverpod Providers (State wiring)
   └── Navigation (GoRouter)

[ FLAME PRESENTATION LAYER (Game Engine) ]
   ├── Components (BoardComponent, PieceComponent)
   ├── VFX & Audio (Flame Audio, ParticleSystemComponent)
   ├── Input Handlers (DragCallbacks)
   └── Animation Sequencers (Flame Effects)

[ APPLICATION LAYER (Orchestration - Dart Services) ]
   ├── RunManager (State Machine)
   ├── BattleController (Turn Pipeline)
   ├── CombatEngine (Damage Calculation)
   └── ProgressionSystem

[ DOMAIN LAYER (Pure Dart Logic) ]
   ├── BoardModel (8x8 int array math)
   ├── PieceModel (Shape matrix, L-value logic)
   ├── GameRng (Seeded generation)
   └── RelicRules (Modifier mathematics)
```

---

## 5. Layering Model (Clean Architecture Variant)

The codebase is strictly divided into layers:

### 5.1 Domain Layer (The Core)
- **What it is:** Pure Dart files (`.dart`). Absolutely zero references to `flutter` or `flame` packages. 
- **Responsibility:** Contains the 8x8 grid matrix, piece placement math, line-clear detection, scoring formulas, and HP tracking. **Note: There is no runtime piece rotation. Piece orientation is static and authored per `ShapeDefinition` data class.** The board cell model carries two logically distinct attributes — Occupancy and Cell Modifier — stored compactly but semantically independent (e.g., a `Frozen` cell is Occupied but does not count toward Line Clear). Full cell-state authority is owned by `03_BOARD_ENGINE_AND_RULES.md` Section 3.
- **Why:** Allows lightning-fast unit testing via `dart test` without launching the Flutter engine.

### 5.2 Application Layer (The Orchestrator)
- **What it is:** Pure Dart controllers and services.
- **Responsibility:** Manages the State Machine (Menu -> Battle -> Post-Match). Evaluates combat sequences (e.g., "Line cleared -> Trigger Relics -> Apply Damage -> Return Result").

### 5.3 Flame Presentation Layer (The Game View)
- **What it is:** Flame `Component` classes, Effects, Flame Audio.
- **Responsibility:** Reads the data from the Domain layer and makes it visible/audible in the game loop. Captures user touch input via `DragCallbacks` and forwards it to the Application layer.
- **Rule:** The Presentation layer NEVER modifies the Domain layer directly. 

### 5.4 Flutter UI Layer (The App View)
- **What it is:** Flutter Widgets, GoRouter navigation, Riverpod providers.
- **Responsibility:** Manages app-level routing, HUD overlays, menus, and meta-game screens. Coordinates with Flame via `GameWidget`.

---

## 6. Dependency Direction

**THE GOLDEN RULE: Dependencies always point INWARD.**

- `Flutter UI` knows about `Riverpod`, `Application`, and `Domain`.
- `Flame` knows about `Application` and `Domain`.
- `Application` knows about `Domain`.
- `Domain` knows about **NOTHING** but itself.

*Example:* `BoardModel` does not know `BoardComponent` exists. When a piece is placed, `BoardModel` fires a pure Dart stream/event (`onPiecePlaced`). `BoardComponent` listens to this event and triggers the Flame animations.

---

## 7. Runtime Lifecycle

The strict state machine flow:

1. **Boot (Flutter):** Initialize Services (Hive CE, SharedPreferences, Audio). Load Player Data. Initialize ProviderScope.
2. **Menu:** Enter `HomeRoute`. Display Flutter UI.
3. **Run Start:** Generate Seed via `GameRng`. Initialize `RunModel` (HP, current Relics, Floor number).
4. **Battle Start:** Initialize `BattleController`. Load Enemy data. Clear `BoardModel`. Mount `FlameGame` via `GameWidget`.
5. **Turn Loop (Player):** Input (`DragCallbacks`) -> Ghost Preview -> Drop -> Board Update -> Combat Calc -> Resolve.
6. **Turn Loop (Enemy):** Enemy Telegraph -> Enemy Action -> Board Modification.
7. **Battle End:** Detect Win/Loss -> Grant Stardust -> Choose Relic -> Next Battle.
8. **Run End:** Save currency -> Display results (Flutter overlay) -> Return to Menu.

---

## 8. Core Gameplay Ownership

- **BoardEngine:** Owns the 2D integer array. Handles collision detection, valid placement checks, line-clear mathematics, and detects **Board Lock** (when no legal moves remain).
- **TurnOrchestrator (Battle Orchestrator):** A state machine managing whose turn it is and blocking input while animations resolve. It converts a Board Lock or Objective Failure into a **Battle Failure**.
- **RunManager:** Tracks floor progression and meta-currency. It converts a Battle Failure into a **Run Defeat** (since the game uses a one-life roguelike structure).
- **Presentation Layer (UI):** Presents the Battle Failure / Run Defeat to the player as a **Game Over** or **Run Defeat** screen. (This separation of responsibility is mandatory).
- **TrayEngine:** Owns the 3-piece Tray state. Serves pieces in their fixed, authored orientation. Runtime rotation is prohibited; orientation diversity is managed via the shape catalog, not runtime transforms.
- **CombatEngine:** Pure math calculator. Takes `BaseDamage` + `L-Value` + `RelicModifiers` -> Outputs `FinalDamage` and `DamageEvents` for the UI.

---

## 9. Presentation Ownership

- **BoardComponent:** Manages the visual rendering of the 8x8 grid in Flame. Spawns and colors `CellComponent` children.
- **PieceComponent:** Manages the drag-and-drop visuals, scale bounces, and ghost previews via Flame components and inputs.
- **VFXManager:** Owns the Object Pool for all particle systems defined in `13`, managed within Flame.
- **AudioManager:** Wraps `FlameAudio` to manage sound channels and mix buses defined in `14`.

---

## 10. Data-Driven Architecture

Content must be authored without touching complex imperative code.

- **Dart Const Classes:** Used for all static game data.
  - `ShapeDefinition` (Matrix arrays for pieces)
  - `EnemyDefinition` (HP, Portrait, Behavior list)
  - `RelicDefinition` (Name, ID, Base Multipliers, visual icon)
  - `EncounterDefinition` (What enemies spawn on Floor N)
- **Hive CE:** Used for persistent local Player Save Data (Stardust, Unlocked Relics, Active Run state).
- **SharedPreferences:** Lightweight mutable player settings.

---

## 11. Determinism Strategy

1. **RNG Ownership:** All gameplay randomness is controlled by a single pure Dart `GameRng` class (e.g., using a seeded PRNG).
2. **Seeded Runs:** At the start of a Run, a master seed is generated and saved. All piece spawns, enemy selections, and relic drops draw from this seed.
3. **Exact Save Restoration:** Mid-run saves restore the exact logical state immediately, relying on serialized runtime state and deterministic RNG continuation, NOT by replaying the input sequence from the seed.
4. **Deterministic Reproduction:** For debugging or replays, a new run started with the same seed + identical player input sequence must reproduce the exact same board state and combat outcome. (Do not use `dart:math` Random in core logic without a strict seed).

---

## 12. Configuration Strategy

- **game_constants.dart:** A static file containing all hardcoded mathematical limits (e.g., `maxBoardSize = 8`, `maxTraySize = 3`, `comboCap = 4`).
- **SettingsManager:** Handles the mutable player settings (Volume, Motion, Drag Offset) defined in `16` using SharedPreferences.

---

## 13. Extensibility Strategy

- **Relic Architecture:** Do not use a massive `switch` statement for relics. Use an interface/composition pattern: `RelicEffect`. 
  - Each relic subscribes to the `CombatEngine`'s calculation pipeline via events (e.g., `onBeforeDamageCalculated`, `onLineCleared`).
- **Enemy Behaviors:** Use a Strategy pattern (`EnemyAction`). 

---

## 14. Error Handling Philosophy

- **Domain Layer:** Fail Fast. If a piece tries to place out of bounds, throw a hard `ArgumentError` or `RangeError`. This ensures bugs are caught during automated testing.
- **Presentation Layer:** Degrade Gracefully. If a VFX asset is missing, log a warning and continue the game. A missing particle should never crash the puzzle loop.

---

## 15. Debugging Architecture

- **Cheat Console:** Implement a hidden developer menu (e.g., tap version number 5 times) to instantly add Stardust, kill enemies, spawn specific shapes, or skip floors.
- **Visual Debugging:** Use Flame's debug mode or a custom debug overlay to show the mathematical grid indices on top of the visual board to ensure alignment.

---

## 16. Testing Architecture

- **Unit Tests:** Rely heavily on `dart test`. The Domain layer (Board math, line clearing, combat output) must have high test coverage.
- **Widget/Integration Tests:** Used for Flutter UI interaction validation (`flutter test`) and end-to-end flows (`integration_test`).
- *AI Note:* Because the Domain is decoupled from Flutter/Flame, AI can easily generate comprehensive test suites for the core logic.

---

## 17. Save/Persistence Boundary

- **SaveService:** An abstraction layer. The game logic requests `SaveService.save(runData)`.
- **Implementation:** Uses Hive CE for fast, local NoSQL persistence.
- **State Checkpointing:** The game saves at the start of every Battle, and immediately after a piece is placed (to prevent save-scumming line clears).

---

## 18. Platform Boundary

- **IAP / Ads:** Abstracted behind `MonetizationService`. 
- **Implementation:** Dummy/Mock services are used in development. Platform plugins (e.g., `in_app_purchase`, `google_mobile_ads`) are used in production. Core logic never references platform specifics.

---

## 19. Third-Party Dependency Policy

- **Allowed:** 
  - Essential SDKs (IAP, Ads, Analytics).
  - Frameworks: Riverpod (state), GoRouter (navigation), Hive CE (persistence).
  - Animation: `flutter_animate` for UI, Flame Effects for game entities.
- **Forbidden:**
  - Heavy dependency injection frameworks (e.g., GetIt if Riverpod is already handling DI) - obscures the call stack. 
  - Overly complex architectural packages that hide standard Flutter/Dart paradigms.

---

## 20. Anti-Pattern Catalog

To maintain codebase health, the following are strictly forbidden:

1. **The God Class:** E.g., `GameManager.dart` doing board math, playing audio, and tracking score.
2. **Logic in the UI:** A Flutter Widget's `onPressed` resolving combat damage. (It should only send a command to the Application layer).
3. **Overusing Global State:** Pass dependencies explicitly or use Riverpod scoping securely.
4. **Flame Magic for Logic:** Do not use collision callbacks to determine if a block is on the grid. Use the mathematical integer array.
5. **Magic Numbers:** E.g., `if (score > 1000)`. Must be `if (score > GameConstants.bossThreshold)`.

---

## 21. Naming Principles

Code must read like a standardized manual for both humans and AI.

- **Files:** `snake_case.dart` for all Dart files.
- **Classes/Enums/Typedefs:** `PascalCase` (e.g., `BoardModel`, `GameState`).
- **Variables/Methods:** `lowerCamelCase` (e.g., `hp`, `calculateDamage`).
- **Private Fields:** Prefix `_` (e.g., `_currentScore`).

---

## 22. Performance Principles

Target: 60fps on minimum spec devices.

- **Zero Allocation Gameplay (Where Possible):** During the core puzzle loop, avoid creating massive amounts of temporary objects that trigger Dart Garbage Collection (GC) spikes, causing micro-stutters.
- **Object Pooling:** All blocks, particle systems, and audio sources should be pooled or reused in Flame if instantiation is expensive.
- **Widget Rebuilds:** Minimize Flutter widget rebuilds by carefully scoping Riverpod watchers (`ref.watch`).
- **Flame Optimizations:** Use batched rendering or standard Flame rendering best practices for the board components.

---

## 23. Security / Integrity Considerations

Even for a single-player offline MVP:
- Store Stardust and Unlocks securely if needed, but primarily avoid trivial manipulation.
- IAP validation must rely on standard secure receipt verification.

---

## 24. AI Coding Compatibility

To maximize the effectiveness of AI coding tools (Copilot, ChatGPT, Antigravity):

1. **Small Files:** Keep classes under 300 lines. AI context windows degrade on massive files.
2. **Explicit Interfaces:** Always define the abstract class/interface *before* asking the AI to write the implementation.
3. **Code over Configuration:** Everything is written in code. There is no hidden configuration layer. AI can read and write the entire app because the UI, Engine, and Logic are entirely code-based.

---

## 25. Technology Stack & Layer Responsibilities

### Tech Stack
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
| Testing | flutter test / dart test / integration_test |
| Release | Flutter AAB (App Bundle) |

### Layer Responsibilities
| Layer | Responsibility |
|-------|----------------|
| Domain (Pure Dart) | Rules and deterministic game logic |
| Data | Persistence/content repositories |
| Application | Use cases/orchestration/services |
| Riverpod | Reactive application state/dependency wiring |
| Flame | Real-time game world/presentation/input |
| Flutter | App UI/navigation/settings |
| Hive CE | Persistent local game data |
| SharedPreferences | Lightweight preferences |
| Flame Audio | Game audio |

---

## 26. Architecture Decision Records (ADRs)

Any deviation from this document or major structural addition must be recorded as an ADR in a `/docs/ADR/` folder. 

**Format:**
1. Context (The problem)
2. Options (Considered solutions)
3. Decision (What we chose and why)
4. Consequences (Trade-offs)

---

## 27. Final Architecture Contract

> **Logic is permanent; Presentation is temporary.**
>
> This architecture demands that Block Battles could be played entirely in a text-based terminal console. 
> 
> Flame and Flutter are treated merely as a high-fidelity renderer and input device that sits on top of our bulletproof, mathematically pure, fully deterministic puzzle engine. We do not let Flame dictate how our game works; we tell Flame what to draw.

---

**End of `18_TECHNICAL_ARCHITECTURE.md`.**
