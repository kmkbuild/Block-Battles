# 18_TECHNICAL_ARCHITECTURE.md
## Block Battles — Technical Architecture

**Governing documents:** All Layer A (`01`-`07`) and Layer B (`08`-`17`) documents.
**Document status:** Layer C — Engineering Authority (Primary)

**Inheritance confirmation:** This document translates the mechanical, visual, and UX requirements into a concrete Unity/C# software architecture. It prioritizes the solo-developer constraint (`00`), AI-assisted development paradigms, and long-term maintainability over over-engineered enterprise patterns.

---

## 1. Technical Purpose

The purpose of this architecture is to build a robust, deterministic, and testable foundation for Block Battles. It acts as the "constitution" for all subsequent code generation, ensuring that logic remains decoupled from presentation, bugs are easily isolated, and new content (Relics, Enemies, Shapes) can be added purely via data.

## 2. Architectural Goals

- **Strict Separation of Concerns:** Gameplay logic must compile and run even if the Unity rendering engine is disabled.
- **Determinism:** The same player input and the same random seed must ALWAYS produce the exact same board state and combat outcome.
- **AI-Readability:** Classes must be small, single-responsibility, and explicitly bounded so AI coding agents can generate, refactor, and review them without hallucinating dependencies.
- **Data-Driven Scalability:** Adding a new Enemy or Relic should require zero changes to core systems, only the creation of a new ScriptableObject or JSON configuration.
- **High Performance:** Maintain a rigid 60fps on low-end Android devices.

## 3. Architectural Non-Goals

- **Enterprise Over-engineering:** We will not use bloated frameworks (e.g., Zenject, UniRx, DOTS/ECS) that add massive learning curves and obscure control flow from AI agents unless strictly necessary.
- **Multiplayer/Network Sync:** The architecture does not need to support rollback netcode or server authority.
- **Physics-Driven Gameplay:** The board relies on discrete mathematical arrays, not Unity 2D Physics or Rigidbodies.

---

## 4. High-Level System Map

```text
[ PLATFORM LAYER ]
   ├── AnalyticsService
   ├── MonetizationService
   └── Save/Load IO (FileSystem/JSON)

[ PRESENTATION LAYER (Unity MonoBehaviours) ]
   ├── View Controllers (BoardView, HUDView)
   ├── VFX & Audio Managers (Object Pools)
   ├── Input Handlers (DragDropController)
   └── Animation Sequencers

[ APPLICATION LAYER (Orchestration) ]
   ├── RunManager (State Machine)
   ├── BattleController (Turn Pipeline)
   ├── CombatEngine (Damage Calculation)
   └── ProgressionSystem

[ DOMAIN LAYER (Pure C# Logic) ]
   ├── BoardModel (8x8 int array math)
   ├── PieceModel (Shape matrix, L-value logic)
   ├── RNGProvider (Seeded generation)
   └── RelicRules (Modifier mathematics)
```

---

## 5. Layering Model (Clean Architecture Variant)

The codebase is strictly divided into three primary layers:

### 5.1 Domain Layer (The Core)
- **What it is:** Pure C# classes (`.cs`). Absolutely zero references to `UnityEngine` (except `UnityEngine.Debug` for logging if abstracted). 
- **Responsibility:** Contains the 8x8 grid matrix, piece placement math, line-clear detection, scoring formulas, and HP tracking. **Note: There is no runtime piece rotation. Piece orientation is static and authored per `ShapeData` asset.** The board cell model carries two logically distinct attributes — Occupancy and Cell Modifier — stored compactly but semantically independent (e.g., a `Frozen` cell is Occupied but does not count toward Line Clear). Full cell-state authority is owned by `03_BOARD_ENGINE_AND_RULES.md` Section 3.
- **Why:** Allows lightning-fast unit testing without launching the Unity Editor.

### 5.2 Application Layer (The Orchestrator)
- **What it is:** Pure C# or lightweight MonoBehaviours functioning as controllers.
- **Responsibility:** Manages the State Machine (Menu -> Battle -> Post-Match). Evaluates combat sequences (e.g., "Line cleared -> Trigger Relics -> Apply Damage -> Return Result").

### 5.3 Presentation Layer (The View)
- **What it is:** Unity MonoBehaviours, Canvas elements, Animators, AudioSources.
- **Responsibility:** Reads the data from the Domain layer and makes it visible/audible. Captures user touch input and forwards it to the Application layer.
- **Rule:** The Presentation layer NEVER modifies the Domain layer directly. 

---

## 6. Dependency Direction

**THE GOLDEN RULE: Dependencies always point INWARD.**

- `Presentation` knows about `Application` and `Domain`.
- `Application` knows about `Domain`.
- `Domain` knows about **NOTHING** but itself.

*Example:* `BoardModel` does not know `BoardView` exists. When a piece is placed, `BoardModel` fires an event (`OnPiecePlaced`). `BoardView` listens to this event and triggers the Unity animations.

---

## 7. Runtime Lifecycle

The strict state machine flow:

1. **Boot:** Initialize Services (Audio, Save, IAP). Load Player Data.
2. **Menu:** Enter `HomeState`. Display UI.
3. **Run Start:** Generate Seed. Initialize `RunModel` (HP, current Relics, Floor number).
4. **Battle Start:** Initialize `BattleController`. Load Enemy data. Clear `BoardModel`.
5. **Turn Loop (Player):** Input -> Ghost Preview -> Drop -> Board Update -> Combat Calc -> Resolve.
6. **Turn Loop (Enemy):** Enemy Telegraph -> Enemy Action -> Board Modification.
7. **Battle End:** Detect Win/Loss -> Grant Stardust -> Choose Relic -> Next Battle.
8. **Run End:** Save currency -> Display results -> Return to Menu.

---

## 8. Core Gameplay Ownership

- **BoardEngine:** Owns the 2D integer array. Handles collision detection, valid placement checks, and line-clear mathematics.
- **TrayEngine:** Owns the 3-piece Tray state. Serves pieces in their fixed, authored orientation. Runtime rotation is prohibited; orientation diversity is managed via the shape catalog, not runtime transforms.
- **CombatEngine:** Pure math calculator. Takes `BaseDamage` + `L-Value` + `RelicModifiers` -> Outputs `FinalDamage` and `DamageEvents` for the UI.
- **TurnOrchestrator:** A state machine managing whose turn it is and blocking input while animations resolve.

---

## 9. Presentation Ownership

- **BoardView:** Manages the visual rendering of the 8x8 grid. Spawns and colors `CellView` prefabs.
- **PieceView:** Manages the drag-and-drop visuals, scale bounces, and ghost previews.
- **VFXManager:** Owns the Object Pool for all particle systems defined in `13`.
- **AudioManager:** Owns the AudioSource pools and mix buses defined in `14`.

---

## 10. Data-Driven Architecture

Content must be authored without touching C# scripts.

- **ScriptableObjects (SOs):** Used for all static game data.
  - `ShapeData` (Matrix arrays for pieces)
  - `EnemyData` (HP, Portrait, Behavior list)
  - `RelicData` (Name, ID, Base Multipliers, visual icon)
  - `EncounterData` (What enemies spawn on Floor N)
- **JSON:** Used only for Player Save Data (Settings, Stardust, Unlocked Relics, Active Run state).

---

## 11. Determinism Strategy

1. **RNG Ownership:** All gameplay randomness is controlled by a single `GameRNG` class.
2. **Seeded Runs:** At the start of a Run, a master seed is generated and saved. All piece spawns, enemy selections, and relic drops draw from this seed.
3. **Exact Save Restoration:** Mid-run saves restore the exact logical state immediately, relying on serialized runtime state and deterministic RNG continuation (`rngStateCalls`), NOT by replaying the input sequence from the seed (`24` Section 9).
4. **Deterministic Reproduction:** For debugging or replays, a new run started with the same seed + identical player input sequence must reproduce the exact same board state and combat outcome. (Do not use `UnityEngine.Random` in core logic).

---

## 12. Configuration Strategy

- **GameConstants.cs:** A static class containing all hardcoded mathematical limits (e.g., `MAX_BOARD_SIZE = 8`, `MAX_TRAY_SIZE = 3`, `COMBO_CAP = 4`).
- **SettingsManager:** Handles the mutable player settings (Volume, Motion, Drag Offset) defined in `16`.

---

## 13. Extensibility Strategy

- **Relic Architecture:** Do not use a massive `switch` statement for relics. Use an interface/composition pattern: `IRelicEffect`. 
  - Each relic subscribes to the `CombatEngine`'s calculation pipeline via events (e.g., `OnBeforeDamageCalculated`, `OnLineCleared`).
- **Enemy Behaviors:** Use a Strategy pattern (`IEnemyAction`). 

---

## 14. Error Handling Philosophy

- **Domain Layer:** Fail Fast. If a piece tries to place out of bounds, throw a hard `ArgumentOutOfRangeException`. This ensures bugs are caught during automated testing.
- **Presentation Layer:** Degrade Gracefully. If a VFX prefab is missing, log a warning and continue the game. A missing particle should never crash the puzzle loop.

---

## 15. Debugging Architecture

- **Cheat Console:** Implement a hidden developer menu (e.g., tap version number 5 times) to instantly add Stardust, kill enemies, spawn specific shapes, or skip floors.
- **Visual Debugging:** Use Unity Gizmos or a debug overlay to show the mathematical grid indices on top of the visual board to ensure alignment.

---

## 16. Testing Architecture

- **Unit Tests:** Rely heavily on Unity Test Framework (Edit Mode tests). The Domain layer (Board math, line clearing, combat output) must have high test coverage.
- **Play Mode Tests:** Used sparingly for UI interaction validation (e.g., simulating a drag and drop).
- *AI Note:* Because the Domain is decoupled from Unity, AI can easily generate comprehensive NUnit test suites for the core logic.

---

## 17. Save/Persistence Boundary

- **ISaveService:** An abstraction layer. The game logic requests `SaveService.Save(RunData)`.
- **Implementation:** A JSON serializer writing to `Application.persistentDataPath`. 
- **State Checkpointing:** The game saves at the start of every Battle, and immediately after a piece is placed (to prevent save-scumming line clears).

---

## 18. Platform Boundary

- **IAP / Ads:** Abstracted behind `IMonetizationService`. 
- **Implementation:** Dummy/Mock services are used in the Editor. Mobile plugins (e.g., Unity IAP, AdMob) are only instantiated on actual device builds. Core logic never references `UnityEngine.Purchasing` directly.

---

## 19. Third-Party Dependency Policy

- **Allowed:** 
  - Essential SDKs (IAP, Ads, Analytics).
  - Tweening engines (e.g., DOTween) for rapid UI/Motion (`12_ANIMATION_MOTION_AND_GAME_FEEL.md`) implementation.
- **Forbidden:**
  - Visual scripting (Playmaker, Bolt) - corrupts text-based version control and AI coding.
  - Heavy dependency injection frameworks (Zenject) - obscures the call stack. Use a lightweight Service Locator or manual constructor injection instead.

---

## 20. Anti-Pattern Catalog

To maintain codebase health, the following are strictly forbidden:

1. **The God Class:** E.g., `GameManager.cs` doing board math, playing audio, and tracking score.
2. **Logic in the UI:** A Button's `OnClick` event resolving combat damage. (It should only send a command to the Application layer).
3. **GameObject.Find():** Severely degrades performance and breaks when scene hierarchies change. Pass dependencies explicitly.
4. **Unity Magic for Logic:** Do not use `OnTriggerEnter2D` to determine if a block is on the grid. Use the mathematical integer array.
5. **Magic Numbers:** E.g., `if (score > 1000)`. Must be `if (score > GameConstants.BOSS_THRESHOLD)`.

---

## 21. Naming Principles

Code must read like a standardized manual for both humans and AI.

- **Interfaces:** Prefix `I` (e.g., `IBoardModel`).
- **MonoBehaviours:** Suffix `View`, `Controller`, or `Manager` (e.g., `HUDView`, `AudioController`).
- **Pure Data/Logic:** Suffix `Model` or `Data` (e.g., `GridModel`, `EnemyData`).
- **Variables:** camelCase for private (`_hp`), PascalCase for public properties (`MaxHP`).

---

## 22. Performance Principles

Target: 60fps on minimum spec devices.

- **Zero Allocation Gameplay:** During the core puzzle loop, do not use `new`, `LINQ`, or string concatenation (`+`). These trigger Garbage Collection (GC) spikes, causing micro-stutters.
- **Object Pooling:** All blocks, particle systems, and audio sources must be pre-instantiated and pooled on load.
- **Sprite Batching:** Use UI atlases to ensure the entire board draws in 1-2 draw calls (`11` Sec 17).

---

## 23. Security / Integrity Considerations

Even for a single-player offline MVP:
- Store Stardust and Unlocks with a simple checksum hash to prevent trivial manipulation via JSON editing.
- IAP validation must rely on standard secure receipt verification.

---

## 24. AI Coding Compatibility

To maximize the effectiveness of AI coding tools (Copilot, ChatGPT, Antigravity):

1. **Small Files:** Keep classes under 300 lines. AI context windows degrade on massive files.
2. **Explicit Interfaces:** Always define the interface *before* asking the AI to write the implementation.
3. **Code over Configuration:** Prefer wiring up dependencies in C# (e.g., `controller.Init(view, model)`) rather than dragging and dropping 50 references in the Unity Inspector. AI cannot easily edit Unity `.prefab` or `.scene` YAML files reliably, but it can write C# flawlessly.

---

## 25. Architecture Decision Records (ADRs)

Any deviation from this document or major structural addition must be recorded as an ADR in a `/docs/ADR/` folder. 

**Format:**
1. Context (The problem)
2. Options (Considered solutions)
3. Decision (What we chose and why)
4. Consequences (Trade-offs)

---

## 26. Final Architecture Contract

> **Logic is permanent; Presentation is temporary.**
>
> This architecture demands that Block Battles could be played entirely in a text-based terminal console. 
> 
> The Unity engine is treated merely as a high-fidelity renderer and input device that sits on top of our bulletproof, mathematically pure, fully deterministic puzzle engine. We do not let Unity dictate how our game works; we tell Unity what to draw.

---

**End of `18_TECHNICAL_ARCHITECTURE.md`.**
