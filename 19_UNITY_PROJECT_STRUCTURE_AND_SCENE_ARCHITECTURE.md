# 19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md
## Block Battles — Unity Project Structure & Scene Architecture

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md` (and all prior Layer A & B docs).
**Document status:** Layer C — Engineering Authority (Primary)

**Inheritance confirmation:** This document physically enforces the architecture defined in Doc 18. It defines the exact folder structure, Assembly Definitions (ASMDEFs), and Scene loading paradigms to ensure the solo-developer and AI-assisted workflows remain organized, scalable, and decoupled.

---

## 1. Unity Version Policy

- **Requirement:** The project MUST use a Unity Long Term Support (LTS) version. 
- **Target:** Unity 2022.3 LTS or newer.
- **Why:** Beta and TECH stream releases introduce instability and package deprecations that break solo-dev momentum and confuse AI coding agents analyzing standard Unity APIs.

---

## 2. Project Folder Tree

All custom game content must exist within a root `_Project` folder. This isolates our code/assets from imported third-party packages, plugins, and Unity standard assets.

```text
Assets/
├── _Project/
│   ├── Art/
│   │   ├── Materials/
│   │   ├── Sprites/
│   │   │   ├── Board/
│   │   │   ├── UI/
│   │   │   └── Enemies/
│   │   └── VFX/
│   ├── Audio/
│   │   ├── Music/
│   │   └── SFX/
│   ├── Data/
│   │   ├── Enemies/        (ScriptableObjects)
│   │   ├── Relics/         (ScriptableObjects)
│   │   └── Shapes/         (ScriptableObjects)
│   ├── Prefabs/
│   │   ├── Board/
│   │   ├── Systems/
│   │   └── UI/
│   ├── Scenes/
│   ├── Scripts/
│   │   ├── Core/           (Domain Layer)
│   │   ├── Application/    (Orchestration Layer)
│   │   ├── Presentation/   (MonoBehaviours & Views)
│   │   ├── Services/       (Save, Audio, IAP)
│   │   └── Utils/          (Extensions, Math)
│   └── Settings/
├── Plugins/
└── ThirdParty/
```

**Forbidden:** Do NOT use a `Resources/` folder. It bloats application startup time and prevents proper memory management.

---

## 3. Assembly Definition Strategy (.asmdef)

To enforce the dependency rules from `18` Section 6, the project uses Assembly Definitions. This physically prevents the `Core` layer from referencing `UnityEngine.UI` or `Presentation` scripts.

1. `BlockBattles.Core.asmdef` (No Unity references allowed except `UnityEngine.CoreModule` for basic math/vectors).
2. `BlockBattles.Application.asmdef` (References `Core`).
3. `BlockBattles.Presentation.asmdef` (References `Core` and `Application`).
4. `BlockBattles.Services.asmdef` (References `Core` for data models).
5. `BlockBattles.Tests.asmdef` (References `Core` and `Application`).

---

## 4. Scene Inventory

The MVP requires exactly **THREE** scenes. Keeping the scene count low reduces loading times, simplifies testing, and makes state management predictable.

1. `00_Boot`
2. `01_MainMenu`
3. `02_Battle`

---

## 5. Boot Scene (`00_Boot`)

- **Purpose:** The entry point of the application. It initializes all persistent background services, authenticates the user (if OS-level auth exists), and loads the save file.
- **Persistent Objects:** Instantiates a `[Services]` GameObject marked with `DontDestroyOnLoad`.
- **Initialization:** 
  - `SaveService.Initialize()`
  - `AudioService.Initialize()`
  - `MonetizationService.Initialize()`
- **Transition:** Once initialization is complete, it loads `01_MainMenu` additively or destructively.

---

## 6. Main Menu Scene (`01_MainMenu`)

- **Purpose:** Handles the Home Screen (`15` Section 3), Relic Compendium, and Settings.
- **Local Objects:** `MainMenuController`, `SettingsCanvas`, `CompendiumCanvas`.
- **Flow:** When the player taps "Start Run", it generates the Run Seed, instantiates the `RunModel`, and loads `02_Battle`.

---

## 7. Gameplay Scene (`02_Battle`)

- **Purpose:** The core game loop. 
- **Local Objects:** 
  - `BattleOrchestrator`
  - `BoardView`, `TrayView`
  - `HUDCanvas`
  - `VFXManager` (Object Pool)
- **Flow:** Resolves single battles. At the end of a battle, it triggers the Relic Selection overlay (not a separate scene), updates the `RunModel`, and either re-initializes itself for the next battle or loads `01_MainMenu` on Run Over.

---

## 8. Optional Reward / Transition Architecture

**Decision:** Reward screens, Relic selection, and "Next Floor" transitions are **Overlay UI Canvases** within `02_Battle`. 
**Why:** Loading a separate `03_Reward` scene just to tap a card takes longer to load than it does to play. Keep it in the Battle scene to maintain the Flow State (`14_AUDIO_SFX_MUSIC_AND_MIXING.md`).

---

## 9. Persistent Runtime Objects

The `[Services]` GameObject created in `00_Boot` survives forever. It contains:
- `SceneTransitionManager` (Handles fade-to-black overlays).
- `AudioService` (Music continues playing seamlessly between Menu and Battle).
- `GameRNG` (The master seed controller for the run).
- `RunModel` (The active state of the player's campaign).

---

## 10. Scene Dependency Rules

- Scenes **must not** depend on previous scenes to function for testing.
- If a developer presses Play directly in `02_Battle`, a `DevBootstrapper` script must detect that `[Services]` is missing, instantiate a temporary mock version, and inject a dummy `RunModel` so testing can occur instantly.

---

## 11. Prefab Architecture

Prefabs must be granular. Avoid monolithic prefabs (e.g., an entire "Game" prefab).

- **`pf_Board`:** The 8x8 grid visual container.
- **`pf_Cell`:** A single 1x1 visual square. Spawns 64 times inside the Board.
- **`pf_Piece`:** A draggable piece. Contains child sprites generated dynamically based on `ShapeData`.
- **`pf_EnemyHUD`:** The portrait, HP bar, and telegraph UI.
- **`pf_RelicCard`:** The UI component for the selection screen.
- **`pf_VFX_[Type]`:** Particle system prefabs for the Object Pool (`13` Sec 18).

---

## 12. Prefab Ownership

- **BoardView** owns and instantiates `pf_Cell`.
- **TrayView** owns and instantiates `pf_Piece`.
- **VFXManager** owns and instantiates all `pf_VFX_` prefabs.
- **UIManager** owns UI modal prefabs.

*Rule:* The `Core` logic layer NEVER instantiates a prefab. It only fires events saying "A piece spawned." The `Presentation` layer listens and instantiates.

---

## 13. Canvas Architecture

- **`Canvas_Background` (Screen Space - Camera):** The static background art.
- **`Canvas_Board` (World Space):** Used if board elements require UI components, though standard SpriteRenderers are highly preferred for the Board and Pieces to ensure perfect pixel alignment and batching.
- **`Canvas_HUD` (Screen Space - Overlay):** Enemy HP, Move limits, Score, Pause button.
- **`Canvas_Modals` (Screen Space - Overlay):** Settings, Pause Menu, Relic Selection. Highest sort order.

---

## 14. Sorting / Layers / Tags

**Tags:** The use of Unity Tags (e.g., `if (gameObject.tag == "Enemy")`) is **strictly forbidden**. It relies on strings and causes silent failures. Use interfaces (e.g., `if (component is IEnemy)`).

**Sorting Layers:**
1. `Background` (Z = 0)
2. `Board` (Z = 10)
3. `PlacedPieces` (Z = 20)
4. `VFX_Behind` (Z = 25)
5. `DraggedPiece` (Z = 30) - Ensures dragged pieces float over everything.
6. `VFX_Front` (Z = 40)
7. `UI` (Handled by Canvas sorting).

---

## 15. Physics / Collision Policy

**Policy:** ZERO Unity Physics (`Rigidbody2D`, `Collider2D`, `Physics2D.Raycast`).

The board is a mathematical grid. Dragging a piece translates screen coordinates to world coordinates, mathematically divides by the cell size, and checks the 2D array in `BoardModel`. 

Using Unity physics for grid-based puzzle placement introduces float inaccuracies, jitter, and massive performance overhead. It is forbidden.

---

## 16. Addressables / Asset Loading

**Decision:** Standard direct references for Absolute MVP. 
**Why:** Addressables are overkill for a 2D puzzle game that is < 150MB in total size. `Resources.Load` is banned. Use direct inspector references in a `GameConfig` ScriptableObject, or simple AssetBundles if the game exceeds memory limits (unlikely).

---

## 17. Resource Lifetime

- **Pooling:** All pieces, cells, particles, and audio sources are pre-instantiated in the `Start()` method of `02_Battle` (or specific Managers) into object pools.
- **Destruction:** Pools are destroyed naturally when the `02_Battle` scene is unloaded. We do not persist combat VFX pools back to the Main Menu.

---

## 18. Scene Transition Rules

- Never use `SceneManager.LoadScene` directly in gameplay code.
- Always route through a `ISceneTransitionService`.
- **Flow:** Service triggers Fade-Out UI animation -> Async Scene Load -> Service triggers Fade-In UI animation.

---

## 19. Editor Tooling

To accelerate AI and solo-dev workflows, the following custom inspectors/tools are required:
1. **Grid Debugger:** A gizmo script that draws the mathematical 8x8 grid bounds in the Scene View to ensure visual alignment.
2. **Shape Editor:** A custom inspector for `ShapeData` ScriptableObjects that displays a 5x5 grid of toggles, allowing designers to literally "draw" the block shape in the editor instead of typing raw matrices.

---

## 20. Environment-Specific Assets

Keep environment profiles minimal. 
- Ensure a generic Mobile Profile for texture compression (ASTC).
- Do not create different prefabs for iOS vs Android; use runtime device detection if platform-specific UI adjustments are necessary.

---

## 21. Scene Validation Checklist

Before a build, scenes must pass these checks:
- [ ] `00_Boot` has no missing script references.
- [ ] No `GameManager` script exists in the hierarchy (ensure modular architecture).
- [ ] `Physics2D` settings have auto-simulation disabled (to save CPU).
- [ ] The EventSystem exists only in the `[Services]` persistent object or is dynamically spawned. (No duplicate EventSystems in active scenes).
- [ ] Camera Clear Flags are set to Solid Color (no skybox for a 2D game).

---

## 22. Final Project Structure Contract

> **Organized storage yields organized thought.**
>
> This project structure ensures that when a bug occurs in the relic system, the developer knows exactly which folder to open. When the UI needs a tweak, the core game loop is entirely insulated from accidental breakage. 
>
> By enforcing Assembly Definitions and banning Unity Physics for grid logic, we guarantee that AI assistants can write clean, testable C# logic without needing to manipulate undocumented Unity scene properties.

---

**End of `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md`.**
