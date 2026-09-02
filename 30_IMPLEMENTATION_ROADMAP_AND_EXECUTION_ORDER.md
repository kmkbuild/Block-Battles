# 30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md
## Block Battles — Implementation Roadmap and Execution Order

**Governing documents:** ALL `00_MASTER_GAME_VISION.md` through `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md`
**Document status:** Layer D — Production Authority

---

## Governing Principle

This roadmap is NOT organized by document number. It is organized by **technical dependency** and **playable milestone**. A feature cannot be built until its foundations exist. No foundation should ever be built twice.

---

# PART 1: IMPLEMENTATION PHASES

---

## Phase 0 — Project Setup & Infrastructure

**Goal:** An empty Unity project that compiles, runs, and has the structural skeleton in place.
**Reason for Order:** Nothing can be built until the folder structure, ASMDEFs, and services exist.

**Prerequisites:** None.

**Tasks:**
- Create Unity project (Unity 2022.3+ LTS) with 2D URP template.
- Build the folder hierarchy (`_Project/Scripts/Core`, `Application`, `Presentation`, `Services`, `Utils`) per `19` Sec 2.
- Create 5 Assembly Definitions per `19` Sec 3.
- Create the `00_Boot` scene with a `[Services]` GameObject.
- Stub out `ISaveService`, `IAudioService`, `IMonetizationService` interfaces.
- Add `Newtonsoft.Json` and `DOTween` Unity packages.
- Create `GameConstants.cs` with all numeric constants from Layer A docs.
- Set up Git repository. Initial commit.

**Files to Create:**
- `Assets/_Project/Scripts/Core/GameConstants.cs`
- `Assets/_Project/Scripts/Services/ISaveService.cs`
- `Assets/_Project/Scripts/Services/IAudioService.cs`
- `Assets/_Project/Scripts/Services/IMonetizationService.cs`
- `Assets/_Project/Scripts/Application/ServiceLocator.cs`
- `Assets/_Project/Scenes/00_Boot.unity`

**Tests:** Verify the project compiles with zero errors. All 5 ASMDEFs resolve correctly.
**Acceptance Criteria:** The Boot scene runs and logs "Services Initialized" to the console.
**Playable State:** Not playable. White screen.
**Common Failure Modes:** ASMDEF circular references. Package version conflicts.
**Rollback Strategy:** Delete the project. Restore from the initial Git commit.

---

## Phase 1 — Board Model (Domain)

**Goal:** A pure C# 8x8 grid model with collision checking.
**Reason for Order:** Every other gameplay system depends on the Board.

**Prerequisites:** Phase 0.

**Tasks:**
- Implement `BoardModel.cs` (1D array encoding Occupancy + Cell Modifier compactly, indexing logic `row * 8 + col`).
- Implement `IsValidPlacement(ShapeMatrix, int x, int y)` method.
- Implement `CommitPlacement(ShapeMatrix, int x, int y)` method.
- Write NUnit tests covering: bounds checking, overlap detection, full board state reset.

**Files to Create:**
- `Assets/_Project/Scripts/Core/Board/BoardModel.cs`
- `Assets/_Project/Tests/Core/BoardModelTests.cs`

**Tests:** 10+ Unit Tests (Edit Mode). All pass before Phase 2.
**Acceptance Criteria:** Tests prove a piece placed at (0,0) writes to cell index 0. A piece placed at (8,0) is rejected.
**Playable State:** Not playable. No visual.
**Common Failure Modes:** Off-by-one errors in indexing. Incorrect coordinate mapping.

---

## Phase 2 — Shape System (Domain)

**Goal:** A data structure for all 24 defined shapes (`02_SHAPE_LIBRARY.md`).

**Prerequisites:** Phase 1.

**Tasks:**
- Create `ShapeData.cs` ScriptableObject (5x5 `bool[25]` matrix, `OnValidate` auto-sizing) per `20` Sec 4.
- Create `ShapeMatrix.cs` POCO (computed width, height, normalized occupied cells from the authored `bool[25]` array).
- Author **all 24 canonical MVP shape assets** (SHP_001–SHP_024) as defined in `02_SHAPE_LIBRARY.md` Section 12. Begin with the first 5 simple shapes to verify the pipeline, then author the remaining 19 to complete the MVP catalog.
- **Important:** Do NOT implement a `ShapeRotator.cs`. Piece orientation is static and fixed at authoring time. Runtime rotation is explicitly prohibited (`01`, `AI_START_HERE`).

**Files to Create:**
- `Assets/_Project/Scripts/Core/Shapes/ShapeMatrix.cs`
- `Assets/_Project/Data/Shapes/ShapeData.cs`
- `Assets/_Project/Data/Shapes/shape_square_1x1.asset` *(SHP_001)*
- `Assets/_Project/Data/Shapes/shape_line_1x2.asset` *(SHP_002)* ... through `shape_tetromino_t_d.asset` *(SHP_024)*

**Tests:** Assert that `ShapeMatrix` built from a `ShapeData` asset produces the correct width, height, and occupied cell list. Assert that the `ShapeData.blockMatrix` array cannot be modified at runtime (the field is read-only by convention; no setter is exposed).
**Acceptance Criteria:** All 24 canonical MVP shapes (SHP_001–SHP_024) are authored, validated by `OnValidate`, and can be placed into a `BoardModel` without errors. No rotation function exists anywhere in the `BlockBattles.Core` assembly.

---

## Phase 3 — Placement Logic & Input (Application + Presentation)

**Goal:** A player can drag a piece and drop it onto the board.

**Prerequisites:** Phase 1, Phase 2. First phase to require Unity Scene work.

**Tasks:**
- Create `02_Battle.unity` scene with a camera and placeholder board background.
- Implement `InputController.cs` (Touch/Mouse drag detection, delta tracking).
- Implement `DragDropController.cs` (maps world-space drag position to board grid coordinate).
- Implement `BoardView.cs` (spawns 64 `CellView` GameObjects in a grid pattern).
- Implement `PieceView.cs` (a draggable visual sprite parented to a `ShapeData`).
- Implement Ghost Preview: On hover, highlights valid cells Green and invalid cells Red.
- Wire `DragDropController` -> `BoardModel.IsValidPlacement` -> `BoardModel.CommitPlacement`.

**Files to Create:**
- `Assets/_Project/Scripts/Presentation/InputController.cs`
- `Assets/_Project/Scripts/Presentation/DragDropController.cs`
- `Assets/_Project/Scripts/Presentation/Board/BoardView.cs`
- `Assets/_Project/Scripts/Presentation/Board/CellView.cs`
- `Assets/_Project/Scripts/Presentation/Pieces/PieceView.cs`
- `Assets/_Project/Prefabs/Board/pf_Board.prefab`
- `Assets/_Project/Prefabs/Board/pf_Cell.prefab`
- `Assets/_Project/Prefabs/Pieces/pf_Piece.prefab`

**Tests:** (PlayMode) Simulate a pointer-down on cell (3,3) and pointer-up on a valid board position. Assert `BoardModel[target]` is non-zero.
**Acceptance Criteria:** A colored square can be dragged from the Tray area and dropped onto the 8x8 grid.

---

## Phase 4 — Tray System

**Goal:** Three pieces appear at the bottom of the screen. Placing all three spawns a new set.

**Prerequisites:** Phase 2, Phase 3.

**Tasks:**
- Implement `TrayEngine.cs` (maintains array of 3 `ShapeData` references; removes one on commit).
- Implement `TrayView.cs` (spawns 3 `PieceView` instances, handles spacing).
- Implement `PieceSpawner.cs` (requests pieces from `GameRNG`).
- Implement `GameRNG.cs` (seeded `System.Random`, bag-based draw for shapes).

**Files to Create:**
- `Assets/_Project/Scripts/Core/Tray/TrayEngine.cs`
- `Assets/_Project/Scripts/Core/RNG/GameRNG.cs`
- `Assets/_Project/Scripts/Presentation/Tray/TrayView.cs`

**Tests:** Assert `GameRNG` with seed `12345` always produces the same piece sequence.
**Acceptance Criteria:** At game start, 3 pieces appear. Dragging a piece to the board removes it from the tray. When all 3 are placed, 3 new pieces spawn. (M1 Milestone Achieved)

---

## Phase 5 — Line Clearing

**Goal:** Full rows and columns clear and award points.

**Prerequisites:** Phase 4.

**Tasks:**
- Implement `LineScanner.cs` (iterates `BoardModel`, detects full rows and full columns).
- Integrate `LineScanner` call inside `TrayEngine.OnCommit`.
- Implement `LineClearResolver.cs` (clears cells from `BoardModel`, calculates L-Value).
- Fire `LinesClearedEvent` via `EventBus`.
- Wire `BoardView` to listen to `LinesClearedEvent` and flash cleared cells white.

**Files to Create:**
- `Assets/_Project/Scripts/Core/Board/LineScanner.cs`
- `Assets/_Project/Scripts/Core/Board/LineClearResolver.cs`

**Tests:** Assert placing 8 consecutive 1x1 blocks across row 0 triggers a `LinesClearedEvent` with `lineCount = 1`. Assert a 4-line simultaneous clear triggers `lineCount = 4`.
**Acceptance Criteria:** Full rows and columns disappear, clearing animation plays.

---

## Phase 6 — Game Over Detection

**Goal:** The game ends when no piece from the tray can legally fit anywhere on the board.

**Prerequisites:** Phase 5.

**Tasks:**
- Implement `GameOverChecker.cs` (checks all 3 tray pieces against all valid board positions).
- Called after each `TrayEngine.Refill` (when new pieces spawn).
- Fire `BoardLockedEvent` if no placement is valid.
- Wire `BattleOrchestrator` state machine to handle `BoardLockedEvent` -> `DEFEAT` state.

**Files to Create:**
- `Assets/_Project/Scripts/Core/Board/GameOverChecker.cs`
- `Assets/_Project/Scripts/Application/BattleOrchestrator.cs` (State Machine skeleton)

**Tests:** Assert a completely full board (64 cells occupied) triggers `BoardLockedEvent`. Assert a board with one open 1x1 cell and a tray containing only 1x1 pieces does NOT trigger game over.
**Acceptance Criteria:** Filling the board ends the session. (M2 Milestone Achieved — pure puzzle loop is complete)

---

## Phase 7 — Combat System (Domain)

**Goal:** Line Clears deal damage to an enemy.

**Prerequisites:** Phase 5, Phase 6. This is the first "BLOCK BATTLES" specific feature.

**Tasks:**
- Implement `CombatEngine.cs`: input is `lineCount` (L-value), output is `DamageDealtEvent`.
- Implement damage formula from `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` (Base * L_Multiplier).
- Implement `ComboTracker.cs` (tracks consecutive multi-line clears within a turn).
- Wire `CombatEngine` to listen to `LinesClearedEvent`.

**Files to Create:**
- `Assets/_Project/Scripts/Core/Combat/CombatEngine.cs`
- `Assets/_Project/Scripts/Core/Combat/ComboTracker.cs`

**Tests:** Assert L=1 produces `BaseDamage * 1.0`. Assert L=3 produces `BaseDamage * 2.0`. Assert L=4 is capped.
**Acceptance Criteria:** Line Clears produce a logged damage number.

---

## Phase 8 — Enemy System

**Goal:** A visible enemy with HP receives damage, reacts, and can be defeated.

**Prerequisites:** Phase 7.

**Tasks:**
- Create `EnemyData.cs` ScriptableObject per `20` Sec 5.
- Author 1 dummy test enemy asset.
- Implement `EnemyEngine.cs` (owns `currentHP`, exposes `TakeDamage`, fires `EnemyDefeatedEvent`).
- Implement `EnemyHUDView.cs` (portrait, HP bar, subscribes to `DamageDealtEvent`).
- Wire `CombatEngine.DamageDealtEvent` -> `EnemyEngine.TakeDamage`.

**Files to Create:**
- `Assets/_Project/Scripts/Core/Enemy/EnemyEngine.cs`
- `Assets/_Project/Data/Enemies/EnemyData.cs`
- `Assets/_Project/Data/Enemies/enm_test_golem.asset`
- `Assets/_Project/Scripts/Presentation/HUD/EnemyHUDView.cs`

**Tests:** Assert `EnemyEngine.TakeDamage(50)` on an HP=100 enemy sets HP=50. Assert `TakeDamage(200)` sets HP=0, not -100, and fires `EnemyDefeatedEvent`.
**Acceptance Criteria:** A visible enemy portrait has an HP bar. Line Clears visibly reduce the HP bar. (M3 Milestone Achieved — combat is playable)

---

## Phase 9 — Objective System

**Goal:** Each battle has a specific win condition beyond simply killing the enemy.

**Prerequisites:** Phase 8.

**Tasks:**
- Create `ObjectiveData.cs` ScriptableObject per `20` Sec 7.
- Author 2 test objectives: `DefeatEnemy`, `ClearLines(10)`.
- Implement `ObjectiveEngine.cs` (tracks progress, fires `ObjectiveProgressedEvent` and `ObjectiveCompletedEvent`).
- Implement `ObjectiveHUDView.cs` (bar or counter).

**Files to Create:**
- `Assets/_Project/Scripts/Core/Objectives/ObjectiveEngine.cs`
- `Assets/_Project/Data/Objectives/ObjectiveData.cs`

**Tests:** Assert that clearing 10 lines triggers `ObjectiveCompletedEvent` on a `ClearLines(10)` objective. Assert progress halts at 100%.
**Acceptance Criteria:** The HUD shows objective progress. Completing an objective triggers the Victory flow. (M5 Milestone Achieved)

---

## Phase 10 — Relic System

**Goal:** Persistent passive modifiers activate during combat.

**Prerequisites:** Phase 9 (Relic effects often interact with Objectives and Combat).

**Tasks:**
- Create `RelicData.cs` ScriptableObject per `20` Sec 6.
- Implement `RelicEngine.cs` (subscribes to domain events, evaluates trigger conditions, returns multipliers to `CombatEngine`).
- Author 2 relics: `BaseDamageBoost` and `OnLineClearMultiplier`.
- Implement `RelicSelectionView.cs` (the post-battle Relic choice modal from `23`).

**Files to Create:**
- `Assets/_Project/Scripts/Core/Relics/RelicEngine.cs`
- `Assets/_Project/Data/Relics/RelicData.cs`
- `Assets/_Project/Scripts/Presentation/Modals/RelicSelectionView.cs`

**Tests:** Assert a `+50% Damage` Relic correctly modifies `CombatEngine.CalculateDamage` output.
**Acceptance Criteria:** After beating an enemy, the player is offered 3 relics. The chosen relic activates in the next battle. (M4 Milestone Achieved)

---

## Phase 11 — Run Structure

**Goal:** Multiple battles chain together with persistent Relic and Stardust progression.

**Prerequisites:** Phase 10.

**Tasks:**
- Implement `RunModel.cs` POCO (seed, floor index, stardust, active relic list).
- Implement `RunManager.cs` (manages floor progression, calls `BattleOrchestrator` to init each battle).
- Implement `EncounterData.cs` ScriptableObject per `20` Sec 8.
- Wire: Enemy Defeated -> Show Relics -> Next Floor.

**Files to Create:**
- `Assets/_Project/Scripts/Core/Run/RunModel.cs`
- `Assets/_Project/Scripts/Application/RunManager.cs`
- `Assets/_Project/Data/Encounters/EncounterData.cs`

**Acceptance Criteria:** The player defeats an enemy, selects a relic, and a new (slightly harder) enemy appears. After defeat, the Run ends. (M6 Milestone Achieved — full run loop is playable)

---

## Phase 12 — Core UI Screens

**Goal:** The game has a real Main Menu, HUD, Pause, and Game Over screen.

**Prerequisites:** Phase 11 (we need the run lifecycle to wire menus correctly).

**Tasks:**
- Build `MainMenuView.cs` and `pf_MainMenu.prefab`.
- Finalize `BattleHUDView.cs` (Score, Move Count, Run Floor).
- Build `PauseModal.cs` and `pf_PauseModal.prefab`.
- Build `DefeatView.cs` and `pf_DefeatModal.prefab`.
- Build `VictoryView.cs` and `pf_VictoryModal.prefab`.
- Implement `UIOrchestrator.cs` (Modal stack, Back button).
- Build `00_Boot` to `01_MainMenu` to `02_Battle` scene transition flow.

**Files to Create:** All UI prefabs and View scripts as per `23` Sec 2.

**Acceptance Criteria:** The full loop (Menu -> Start Run -> Battle -> Win/Lose -> Menu) functions with polished UI screens and correct data display.

---

## Phase 13 — Visual Implementation

**Goal:** The game looks like the visual spec (`11`, `09`, `10`).

**Prerequisites:** Phase 12. UI must exist before visual polish is applied.

**Tasks:**
- Replace placeholder board with the polished Block visual (bevel, gradient, specular) per `11`.
- Apply color palette tokens from `10` to all Cell states (empty, occupied, hover, ghost).
- Import and configure Sprite Atlas for Block assets.
- Apply background art from `09_VISUAL_DESIGN_SYSTEM.md`.
- Apply typography (TextMeshPro fonts) from `09`.
- Apply color themes to Enemy HUD, HP bars.

**Files to Create/Modify:** Art assets, Material assets, Board/Cell visual configs.

**Acceptance Criteria:** The game visually matches the style-guide. The block surface has depth and bevel. The board has the correct dark grid. (M7 Milestone Achieved — visually polished prototype)

---

## Phase 14 — Animation & Game Feel

**Goal:** All defined motion from `12_ANIMATION_MOTION_AND_GAME_FEEL.md` is implemented.

**Prerequisites:** Phase 13 (visual assets must exist to animate).

**Tasks:**
- Implement `AnimationController.cs` (DOTween orchestration layer).
- Piece pickup scale bounce, DragOffset lift.
- Piece placement Thud + Grid Ripple.
- Invalid placement Shake Rejection.
- Line Clear Flash sequence (50ms flash -> 150ms dissolve per `12`).
- Enemy Hit Shake and HP bar smooth lerp.
- All UI Panel entrance/exit DOTween animations.

**Files to Create:**
- `Assets/_Project/Scripts/Presentation/Animation/AnimationController.cs`

**Acceptance Criteria:** The game generates the "Tactile Snap" feeling. Placing pieces feels physical. Clearing a line feels satisfying. The `UIAnimationCompleteEvent` is fired reliably after every sequence.

---

## Phase 15 — VFX

**Goal:** Particle effects from `13_VFX_PARTICLE_AND_IMPACT_SPEC.md` are implemented.

**Prerequisites:** Phase 14 (particle systems augment, not replace, animation).

**Tasks:**
- Implement `VFXManager.cs` (Object Pool for all VFX prefabs).
- Create and wire the following particle systems: `pf_vfx_place_dust`, `pf_vfx_line_clear_burst`, `pf_vfx_line_clear_shard`, `pf_vfx_enemy_hit`, `pf_vfx_enemy_defeated`.
- Wire Particle System playback to EventBus events.

**Files to Create:**
- `Assets/_Project/Scripts/Presentation/VFX/VFXManager.cs`
- All `pf_vfx_*.prefab` files.

**Acceptance Criteria:** A line clear spawns a visible burst of colorful particles. Enemy HP reaching 0 triggers a shatter effect. No VFX causes a frame drop below 55 FPS on a Cascade Test.

---

## Phase 16 — Audio

**Goal:** All SFX and Music from `14_AUDIO_SFX_MUSIC_AND_MIXING.md` are implemented.

**Prerequisites:** Phase 15.

**Tasks:**
- Implement `AudioService.cs` (Audio Source pools, Volume mixer groups for Music/SFX/UI).
- Implement `MusicController.cs` (Plays looping BGM, fades on Scene change).
- Wire all EventBus events to SFX playback (`sfx_cmb_damage_crit_01`, `sfx_brd_lineclear_L2`, etc.).
- Implement placeholder audio assets.
- Wire Settings volume sliders to Unity Audio Mixer.

**Files to Create:**
- `Assets/_Project/Scripts/Services/AudioService.cs`
- `Assets/_Project/Scripts/Services/MusicController.cs`
- `Assets/_Project/Audio/` assets.

**Acceptance Criteria:** The game sounds cohesive and reactive. Muting the Master Volume in Settings silences everything. Backgrounding the app pauses the music.

---

## Phase 17 — Persistence (Save/Load)

**Goal:** Player progress survives app close and reopen.

**Prerequisites:** Phase 11 (Run system must exist for save data to be meaningful).

**Tasks:**
- Implement `SaveService.cs` using `Newtonsoft.Json` + Atomic Save pattern (`24` Sec 7).
- Implement `PlayerProfileSave.cs` + all child data classes.
- Implement `BootstrapService.cs` to load save on boot and corrupt-file recovery.
- Hook `OnApplicationPause` emergency save.
- Implement save on Turn End + Battle Start per `24` Sec 8.

**Tests:** Serialization roundtrip test. Corruption recovery test. Migration V1->V2 test.
**Acceptance Criteria:** Closing the app mid-run and reopening resumes from the correct floor with correct HP and board state. Stardust survives a force-quit.

---

## Phase 18 — Settings & Accessibility

**Goal:** All settings from `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` are functional.

**Prerequisites:** Phase 16, Phase 17.

**Tasks:**
- Implement `SettingsManager.cs`.
- Build `pf_SettingsModal.prefab` with Volume sliders, Haptics toggle, Reduced Motion toggle.
- Implement `ReducedMotion` flag suppressing all DOTween animation durations.
- Implement `SafeAreaFitter.cs` on all full-screen canvases.

**Tests:** Assert `ReducedMotion = true` causes the `pf_MainMenu` panel to snap without animation.
**Acceptance Criteria:** All settings persist. Reduced Motion eliminates all non-essential animations. Notch/safe area is correctly padded on all test devices.

---

## Phase 19 — Monetization

**Goal:** Opt-in Rewarded Ads and Premium Unlock are functional.

**Prerequisites:** Phase 18.

**Tasks:**
- Implement `MonetizationService.cs` with IAP and Ad wrappers.
- Implement `IMonetizationService` Mock for Editor and QA builds.
- Wire Rewarded Ad button in Defeat modal.
- Wire IAP "Remove Ads" purchase to permanently disable ad prompts.
- Integrate GDPR/ATT consent popup in `BootstrapService`.

**Tests:** Simulate an IAP purchase in Internal Testing with a mock account.
**Acceptance Criteria:** A Rewarded Ad can be watched for a Revive. The Premium Unlock removes all ad prompts. Ad SDKs use Test IDs in QA builds. (M8 Milestone Achieved — content-complete MVP)

---

## Phase 20 — QA, Performance, and Polish

**Goal:** The game meets all Release Gates from `27` and `26`.

**Prerequisites:** All 19 phases complete.

**Tasks:**
- Profile on minimum-spec Android device. Fix any frame time violations.
- Run "Zero Allocation" profiler test on combat loop.
- Run 10-battle memory leak test.
- Execute full test suite and achieve >90% Domain coverage.
- Fix all S0 and S1 bugs.
- Conduct external playtest session (5 people).

**Acceptance Criteria:** All automated tests pass. Zero S0 bugs. Performance gates cleared.

---

## Phase 21 — Release Preparation

**Goal:** Build is submitted to Google Play.

**Prerequisites:** Phase 20.

**Tasks:**
- Generate Keystore (`28` Sec 11). Back it up securely.
- Compile Release Build (`.aab`) with Production Ad Unit IDs.
- Upload `symbols.zip` to Google Play Console.
- Complete Store Listing (Screenshots, Description, Feature Graphic) per `28` Sec 18.
- Set initial Staged Rollout to 10%.

**Acceptance Criteria:** The game is live in Google Play Internal Testing and passes Google's automated pre-launch report with zero critical device incompatibilities. (M9 Milestone Achieved — Release Candidate)

---

# PART 2: CRITICAL MILESTONES

| Milestone | What Must Work | What Can Be Rough |
|---|---|---|
| **M0: Project Launches** | Project compiles. Boot scene runs. | Everything else. |
| **M1: Playable Puzzle** | Drag-and-drop. Board placement. Tray refill. | Visuals placeholder. No combat. |
| **M2: Puzzle Loop Complete** | Line clearing. Game over detection. | No combat. No enemy. |
| **M3: Combat Playable** | Damage dealt. Enemy HP tracked. | No run structure. Hardcoded enemy. |
| **M4: Relics Playable** | Relics modify damage. Relic selection after win. | Only 2 relics. No run persistence. |
| **M5: Objectives Playable** | Objective types tracked. Win conditions trigger correctly. | Only 2 objective types. |
| **M6: Full Run Playable** | Multiple floors. Run death. Persistent Relics. | Placeholder art/audio. No settings. |
| **M7: Polished Prototype** | Art applied. Animations. VFX. Audio. | No Save. No Monetization. |
| **M8: Content-Complete MVP** | Save/Load. Settings. Ads/IAP. All planned content. | Minor visual bugs. |
| **M9: Release Candidate** | All gates passed. Store listing ready. Staged rollout set. | Post-launch content. |

---

# PART 3: VERTICAL SLICE

Before proceeding past Phase 10, build a Vertical Slice:
- **1 Enemy:** Test Golem (100 HP, simple attack telegraph).
- **4 Shapes:** 1x1, 1x3, 2x2, L-Shape.
- **1 Relic:** +20% Damage on Line Clear.
- **1 Objective:** Defeat the Enemy.
- **Victory:** Enemy HP = 0.
- **Defeat:** Board Lock.
- **Restart:** "Play Again" button resets the board and enemy.

**Purpose:** The vertical slice is a mini-playable demo used for internal testing to confirm the core puzzle-combat loop is fun BEFORE investing dozens of hours in content production.

---

# PART 4: MVP SCOPE LOCK

**Must Have (MVP cannot ship without this):**
- 8x8 board with drag-and-drop placement.
- Line clearing (rows and columns).
- 24 canonical playable shape definitions (SHP_001–SHP_024, per `02_SHAPE_LIBRARY.md`).
- Combat: L-Value damage multipliers.
- 5 unique enemies with different HP values and at least 1 action.
- 10 unique relics with varied trigger types.
- 3 objective types (Defeat, Clear Lines, Survive Moves).
- Save / Load system.
- Settings (Volume, Haptics, Reduced Motion).
- Core VFX and SFX.
- Rewarded Ad (Revive) and Premium Unlock (Remove Ads).

**Should Have (Ship with MVP if time permits):**
- Post-MVP shape expansion: integrate SHP_025–SHP_034 (10 authored-but-deferred entries from `02_SHAPE_LIBRARY.md`) after balance testing.
- 8+ enemies.
- 20+ relics.
- Boss enemy (Super-structured attacks).
- Tutorial tooltips.
- Stardust soft-currency with a basic unlock shop.

**Nice to Have (Post-MVP):**
- Cloud Save.
- Achievement system.
- Daily Challenges.
- Seasonal content.

**Never Block MVP:**
- Leaderboards.
- Social sharing.
- Multiplayer.
- Character skins.
- A story mode.

---

# PART 5: AI EXECUTION STRATEGY

### 1. Work One Subsystem at a Time
Never ask an AI to implement "the whole combat system and the enemy system and the relics." 
Ask: "Implement `CombatEngine.cs` based on `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 3."

### 2. Provide Full Context
When prompting, paste the relevant Markdown sections as context. Do not assume the AI remembers the correct values from previous sessions.

### 3. Compile After Structural Changes
After any change to `.asmdef` files, interfaces, or class constructors — compile immediately before continuing. Structural errors cascade catastrophically.

### 4. Test After Gameplay Changes
After any Domain Layer modification (Board, Combat, Relics), run the NUnit test suite before proceeding.

### 5. Visual Review After UI Changes
After any Prefab or Canvas modification, deploy to a device (or the Game view) to check layout, safe areas, and font sizing.

### 6. Commit After Stable Milestones
Commit to Git at every Milestone (`M1` through `M9`). Tag the commit (`git tag -a M1 -m "Playable puzzle loop"`).

---

# PART 6: QUALITY GATES

| Gate | Threshold | Tool |
|---|---|---|
| Architecture Gate | Zero Domain Layer Unity references. | ASMDEF compiler errors. |
| Gameplay Gate | All 20+ NUnit domain tests pass. | Unity Test Runner. |
| UX Gate | No modal blocks input unintentionally on any test device. | Manual device test. |
| Visual Gate | Art matches `09`/`10`/`11` color tokens and material spec. | Screenshot review. |
| Audio Gate | All core gameplay events trigger the correct SFX. | Ear test + EventBus log. |
| Performance Gate | 60 FPS on minimum spec. 0-byte GC allocation in combat. | Unity Profiler on device. |
| QA Gate | All `27` Automated Tests pass. Zero S0/S1 bugs. | Unity Test Runner + Bug tracker. |
| Release Gate | Complete `28` Sec 21 checklist item by item. | Manual verification. |

---

# PART 7: PRODUCTION SCHEDULE MODEL

Work units are expressed in "Sessions" (approximately 2-4 focused hours of solo development each).

| Phase | Estimated Sessions | AI Acceleration Potential |
|---|---|---|
| Phase 0 (Setup) | 2 sessions | High (AI generates boilerplate/interfaces) |
| Phase 1 (Board Model) | 2 sessions | High (Pure C# math) |
| Phase 2 (Shape System) | 2 sessions | High (SO definitions + matrix logic) |
| Phase 3 (Placement) | 4 sessions | Medium (Requires manual device feel-testing) |
| Phase 4 (Tray) | 2 sessions | High |
| Phase 5 (Line Clear) | 2 sessions | High (deterministic math) |
| Phase 6 (Game Over) | 1 session | High |
| Phase 7 (Combat) | 2 sessions | High (Pure math, heavily testable) |
| Phase 8 (Enemy) | 3 sessions | Medium (requires portrait art) |
| Phase 9 (Objectives) | 2 sessions | High |
| Phase 10 (Relics) | 4 sessions | High (data-driven, many variations) |
| Phase 11 (Run Structure) | 3 sessions | Medium |
| Phase 12 (UI Screens) | 4 sessions | Medium (requires layout review) |
| Phase 13 (Visual) | 5 sessions | Low (art judgment is human) |
| Phase 14 (Animation) | 3 sessions | Medium (DOTween values require feel-testing) |
| Phase 15 (VFX) | 3 sessions | Medium |
| Phase 16 (Audio) | 3 sessions | Low (audio mixing is subjective) |
| Phase 17 (Persistence) | 3 sessions | High (pure C# serialization) |
| Phase 18 (Settings) | 2 sessions | High |
| Phase 19 (Monetization) | 3 sessions | Medium (requires test account setup) |
| Phase 20 (QA) | 5 sessions | Medium (profiling requires device + judgment) |
| Phase 21 (Release) | 3 sessions | Low (store assets require human creative) |

**Where AI excels:** Generating boilerplate classes, writing NUnit test suites, generating ScriptableObject content, translating Markdown spec into C# code.
**Where human judgment is mandatory:** Game feel tuning (animation timings), visual art direction, audio mixing levels, UX decisions not covered by a spec.

---

## Final Execution Statement

> **Build the foundation before the walls. Build the walls before the roof.**
>
> This roadmap ensures Block Battles is built with maximum confidence at every step. Every mechanic is proven before it is decorated. Every decoration is applied before it is polished. Every polish pass is verified before it is shipped.
>
> The documents you have built define a game. This roadmap converts those documents into a game.

---

**End of `30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md`.**
