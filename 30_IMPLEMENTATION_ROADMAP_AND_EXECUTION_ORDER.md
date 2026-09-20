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

**Goal:** An empty Flutter project that compiles, runs, and has the structural skeleton in place.
**Reason for Order:** Nothing can be built until the folder structure and foundational packages exist.

**Prerequisites:** None.

**Tasks:**
- Install Flutter stable, Android Studio, run `flutter doctor`.
- Run `flutter create block_battles` to initialize the project.
- Add dependencies in `pubspec.yaml` (`flame`, `flutter_riverpod`, `go_router`, `hive_ce`, `hive_ce_flutter`, `shared_preferences`, `flame_audio`).
- Build the directory structure (`lib/domain`, `lib/application`, `lib/game`, `lib/presentation`, `lib/data`) per doc 19.
- Stub out interfaces or empty classes for services.
- Create `core/game_constants.dart` with all numeric constants from Layer A docs.
- Create an initial `domain/board/board_model.dart` (stub).
- Set up Git repository. Initial commit.

**Files to Create:**
- `lib/main.dart`
- `lib/app/app.dart`
- `lib/app/app_router.dart`
- `lib/core/game_constants.dart`
- `lib/domain/board/board_model.dart`
- `pubspec.yaml`
- `analysis_options.yaml`

**Tests:** Verify `flutter analyze` passes with 0 issues.
**Acceptance Criteria:** `flutter run` launches a blank screen with no errors.
**Playable State:** Not playable. White screen.
**Common Failure Modes:** Package version conflicts in pubspec.yaml.
**Rollback Strategy:** Delete the project. Restore from the initial Git commit.

---

## Phase 1 — Board Model (Domain)

**Goal:** A pure Dart 8x8 grid model with collision checking.
**Reason for Order:** Every other gameplay system depends on the Board.

**Prerequisites:** Phase 0.

**Tasks:**
- Implement `board_model.dart` (1D array encoding Occupancy + Cell Modifier compactly, indexing logic `row * 8 + col`). Ensure it supports MVP states: `EMPTY`, `FILLED`, `BLOCKED`, and `FROZEN`.
- Implement `board_cell.dart`.
- Implement `canReceiveBlock(cell)` and `countsTowardLineClear(cell)` semantic predicates.
- Implement `isValidPlacement(ShapeMatrix, int x, int y)` method using `canReceiveBlock`.
- Implement `commitPlacement(ShapeMatrix, int x, int y)` method.
- Implement `setCellModifier` and Line-Clear evaluation using `countsTowardLineClear`.
- Write Dart tests covering bounds checking and the cell state rules.

**Tests (Must Verify):**
- **Placement:** `EMPTY + NORMAL` works. `EMPTY + BLOCKED` fails. `FILLED + NORMAL` fails. `FILLED + FROZEN` fails.
- **Line clearing:** `FILLED + NORMAL` counts toward line completion. `FILLED + FROZEN` does not count. `EMPTY + NORMAL` does not count. `EMPTY + BLOCKED` does not count.
- **Clear reset:** Any cell that is actually cleared becomes: `EMPTY + NORMAL`.

**Files to Create:**
- `lib/domain/board/board_model.dart`
- `lib/domain/board/board_cell.dart`
- `test/domain/board/board_model_test.dart`

**Tests:** 10+ Unit Tests (`dart test`). All pass before Phase 2.
**Acceptance Criteria:** Tests prove a piece placed at (0,0) writes to cell index 0. A piece placed at (8,0) is rejected.
**Playable State:** Not playable. No visual.
**Common Failure Modes:** Off-by-one errors in indexing. Incorrect coordinate mapping.

---

## Phase 2 — Shape System (Domain)

**Goal:** A data structure for all 24 defined shapes (`02_SHAPE_LIBRARY.md`).

**Prerequisites:** Phase 1.

**Tasks:**
- Create `shape_definition.dart` and `shape_matrix.dart` (computed width, height, normalized occupied cells from an authored `bool` matrix or array).
- Create `shape_catalog.dart` (contains all 24 shapes as const `ShapeDefinition` objects).
- Author **all 24 canonical MVP shape definitions** (SHP_001–SHP_024) as defined in `02_SHAPE_LIBRARY.md` Section 12. Begin with the first 5 simple shapes to verify the pipeline, then author the remaining 19 to complete the MVP catalog.
- **Important:** Do NOT implement a shape rotator. Piece orientation is static and fixed at authoring time. Runtime rotation is explicitly prohibited (`01`, `AI_START_HERE`). Shapes are purely Dart const objects.

**Files to Create:**
- `lib/domain/shapes/shape_definition.dart`
- `lib/domain/shapes/shape_matrix.dart`
- `lib/data/definitions/shape_catalog.dart`

**Tests:** Assert that `ShapeMatrix` built from a `ShapeDefinition` produces the correct width, height, and occupied cell list using `dart test`.
**Acceptance Criteria:** All 24 canonical MVP shapes (SHP_001–SHP_024) are authored as Dart consts and can be placed into a `BoardModel` without errors. No rotation function exists.

---

## Phase 3 — Placement Logic & Input (Application + Presentation)

**Goal:** A player can drag a piece and drop it onto the board.

**Prerequisites:** Phase 1, Phase 2. First phase to require Flame + Flutter UI work.

**Tasks:**
- Create `BattleScreen` widget, add a `GameWidget` hosting a `FlameGame` (`BlockBattlesGame`).
- Implement `BlockBattlesGame` and component hierarchy.
- Implement Flame `DragCallbacks` on `PieceComponent` for drag detection and positioning.
- Implement Ghost preview logic: render valid cells Green and invalid cells Red in `BoardComponent.render()`.
- Map world-space drag position to board grid coordinate.
- Wire drag drop -> `BoardModel.isValidPlacement` -> `BoardModel.commitPlacement`.

**Files to Create:**
- `lib/game/block_battles_game.dart`
- `lib/game/components/board_component.dart`
- `lib/game/components/cell_component.dart`
- `lib/game/components/piece_component.dart`
- `lib/game/input/drag_handler.dart`

**Tests:** (Widget test) Simulate a pointer-down on cell (3,3) and pointer-up on a valid board position using `flutter test`.
**Acceptance Criteria:** A colored square can be dragged from the Tray area and dropped onto the 8x8 grid in the Flame canvas.

---

## Phase 4 — Tray System

**Goal:** Three pieces appear at the bottom of the screen. Placing all three spawns a new set.

**Prerequisites:** Phase 2, Phase 3.

**Tasks:**
- Implement `tray_engine.dart` (maintains array of 3 shape references; removes one on commit).
- Implement `tray_component.dart` (Flame component rendering 3 pieces, handles spacing).
- Implement piece spawning requests from `GameRNG`.
- Implement `game_rng.dart` (uses Dart's `Random(seed)` with bag-based draw for shapes).

**Files to Create:**
- `lib/domain/tray/tray_engine.dart`
- `lib/domain/rng/game_rng.dart`
- `lib/game/components/tray_component.dart`

**Tests:** Assert `GameRNG` with seed `12345` always produces the same piece sequence via `dart test`.
**Acceptance Criteria:** At game start, 3 pieces appear. Dragging a piece to the board removes it from the tray. When all 3 are placed, 3 new pieces spawn. (M1 Milestone Achieved)

---

## Phase 5 — Line Clearing

**Goal:** Full rows and columns clear and award points.

**Prerequisites:** Phase 4.

**Tasks:**
- Implement `line_scanner.dart` (iterates `BoardModel`, detects full rows and full columns).
- Integrate scanner call inside tray/commit logic.
- Implement `line_clear_resolver.dart` (clears cells from `BoardModel`, calculates L-Value).
- Fire events via Dart `StreamController` (EventDispatcher per doc 22).
- Wire `BoardComponent` to listen to `LinesCleared` stream and play flash effect.

**Files to Create:**
- `lib/domain/board/line_scanner.dart`
- `lib/domain/board/line_clear_resolver.dart`

**Tests:** Assert placing 8 consecutive 1x1 blocks across row 0 triggers a line cleared event with `lineCount = 1` via `dart test`.
**Acceptance Criteria:** Full rows and columns disappear, clearing animation plays.

---

## Phase 6 - Board Lock Detection

**Goal:** The puzzle can end when no valid moves remain.

**Prerequisites:** Phase 4 (Tray).

**Tasks:**
- Implement `board_lock_detector.dart` (checks all 3 tray pieces against all valid board positions).
- Hook lock detector to run at the start of the active state.
- Wire state machine to handle lock detected event -> convert to battle failed.

**New Files:**
- `lib/domain/board/board_lock_detector.dart`

**Tests:** Assert a completely full board (64 cells occupied) triggers lock. Assert a board with one open 1x1 cell and a tray containing only 1x1 pieces does NOT trigger lock via `dart test`.
**Acceptance Criteria:** Filling the board ends the session. (M2 Milestone Achieved — pure puzzle loop is complete)

---

## Phase 7 — Combat System (Domain)

**Goal:** Line Clears deal damage to an enemy.

**Prerequisites:** Phase 5, Phase 6. This is the first "BLOCK BATTLES" specific feature.

**Tasks:**
- Implement `combat_engine.dart`: input is `lineCount` (L-value), output is damage dealt event.
- Implement damage formula from `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` (Base * L_Multiplier).
- Implement `combo_tracker.dart` (tracks consecutive multi-line clears within a turn).
- Wire `combat_engine` to listen to line cleared events.

**Files to Create:**
- `lib/domain/combat/combat_engine.dart`
- `lib/domain/combat/combo_tracker.dart`

**Tests:** Assert L=1 produces `BaseDamage * 1.0`. Assert L=3 produces `BaseDamage * 2.0`. Assert L=4 is capped via `dart test`.
**Acceptance Criteria:** Line Clears produce a logged damage number.

---

## Phase 8 — Enemy System

**Goal:** A visible enemy with HP receives damage, reacts, and can be defeated.

**Prerequisites:** Phase 7.

**Tasks:**
- Create `enemy_catalog.dart` (contains `EnemyDefinition` for test golem as Dart const) per `20` Sec 5.
- Implement `enemy_service.dart` (owns current HP, exposes take damage logic).
- Implement `enemy_hud_widget.dart` (Flutter `ConsumerWidget` watching `EnemyStateNotifier`, portrait, HP bar).
- Wire combat damage events -> `enemy_service`.

**Files to Create:**
- `lib/domain/enemies/enemy_service.dart`
- `lib/data/definitions/enemy_catalog.dart`
- `lib/presentation/battle/widgets/enemy_hud_widget.dart`

**Tests:** Assert damage logic via `dart test`. `flutter test` for `EnemyHudWidget` UI logic.
**Acceptance Criteria:** A visible enemy portrait has an HP bar. Line Clears visibly reduce the HP bar. (M3 Milestone Achieved — combat is playable)

---

## Phase 9 — Objective System

**Goal:** Each battle has a specific win condition beyond simply killing the enemy.

**Prerequisites:** Phase 8.

**Tasks:**
- Create `objective_catalog.dart` and define models.
- Author 2 test objectives: `DefeatEnemy`, `ClearLines(10)`.
- Implement `objective_service.dart` (tracks progress, fires completion events).
- Implement `objective_hud_widget.dart` (bar or counter).

**Files to Create:**
- `lib/domain/objectives/objective_service.dart`
- `lib/data/definitions/objective_catalog.dart`
- `lib/presentation/battle/widgets/objective_hud_widget.dart`

**Tests:** Assert that clearing 10 lines triggers completion on a `ClearLines(10)` objective via `dart test`.
**Acceptance Criteria:** The HUD shows objective progress. Completing an objective triggers the Victory flow. (M5 Milestone Achieved)

---

## Phase 10 — Relic System

**Goal:** Persistent passive modifiers activate during combat.

**Prerequisites:** Phase 9.

**Tasks:**
- Create `relic_catalog.dart` with 2 relics as Dart consts (`BaseDamageBoost` and `OnLineClearMultiplier`).
- Implement `relic_service.dart` (evaluates trigger conditions, returns multipliers).
- Implement `relic_selection_overlay.dart` (Flutter overlay Stack for relic choice).

**Files to Create:**
- `lib/domain/relics/relic_service.dart`
- `lib/data/definitions/relic_catalog.dart`
- `lib/presentation/battle/overlays/relic_selection_overlay.dart`

**Tests:** Assert a `+50% Damage` Relic correctly modifies output via `dart test`.
**Acceptance Criteria:** After beating an enemy, the player is offered 3 relics via Flutter overlay. The chosen relic activates in the next battle. (M4 Milestone Achieved)

---

## Phase 11 — Run Structure

**Goal:** Multiple battles chain together with in-memory Relic and Stardust progression across battles (file persistence is added in Phase 17).

**Prerequisites:** Phase 10.

**Tasks:**
- Implement `run_model.dart` (pure Dart class: seed, floor index, stardust, active relics).
- Implement `run_service.dart` (pure Dart orchestration for floor progression).
- Implement `encounter_catalog.dart`.
- Implement `run_state_notifier.dart` (Riverpod `StateNotifier`).
- Wire: Enemy Defeated -> Show Relics -> Next Floor.

**Files to Create:**
- `lib/domain/run/run_model.dart`
- `lib/application/run_service.dart`
- `lib/data/definitions/encounter_catalog.dart`
- `lib/application/providers/run_state_notifier.dart`

**Acceptance Criteria:** The player defeats an enemy, selects a relic, and a new (slightly harder) enemy appears. After defeat, the Run ends. (M6 Milestone Achieved — full run loop is playable)

---

## Phase 12 — Core UI Screens

**Goal:** The game has a real Main Menu, HUD, Pause, and Run Defeat (Game Over) screen.

**Prerequisites:** Phase 11.

**Tasks:**
- Build `home_screen.dart` (for route `/`) and `battle_screen.dart` (for route `/battle`).
- Build Settings screen for route `/settings`.
- Build overlays for Pause, Victory, Defeat, RelicSelection, and RunComplete. These are shown via Flutter Stack/conditional rendering in `BattleScreen`, driven by state via Riverpod, NOT separate routes.
- Configure GoRouter routes (`/`, `/battle`, `/settings`) in `app_router.dart`.
- Build scene transition logic through GoRouter.

**Files to Create:**
- `lib/presentation/home/home_screen.dart`
- `lib/presentation/battle/battle_screen.dart`
- `lib/presentation/battle/overlays/pause_overlay.dart`
- `lib/presentation/battle/overlays/defeat_overlay.dart`
- `lib/presentation/battle/overlays/victory_overlay.dart`
- `lib/app/app_router.dart`

**Acceptance Criteria:** The full loop (Menu -> Start Run -> Battle -> Win/Lose -> Menu) functions with Flutter routing and overlays.

---

## Phase 13 — Visual Implementation

**Goal:** The game looks like the visual spec (`11`, `09`, `10`).

**Prerequisites:** Phase 12.

**Tasks:**
- Replace placeholder Flame blocks with polished visuals (CustomPainter or Flame PositionComponent with canvas drawing).
- Apply color palette tokens from `10` to all Cell states.
- Configure Flame SpriteSheet / SpriteBatch for visual assets.
- Apply background art from `09`.
- Replace default fonts with custom typography via Flutter Text and `pubspec.yaml` fonts.
- Apply color themes to Enemy HUD, HP bars.

**Files to Create/Modify:** Image assets in `assets/images`, Flame component visual configurations.

**Acceptance Criteria:** The game visually matches the style-guide. The block surface has depth and bevel. The board has the correct dark grid. (M7 Milestone Achieved — visually polished prototype)

---

## Phase 14 — Animation & Game Feel

**Goal:** All defined motion from `12_ANIMATION_MOTION_AND_GAME_FEEL.md` is implemented.

**Prerequisites:** Phase 13.

**Tasks:**
- Implement Flame Effect system (`MoveEffect`, `ScaleEffect`, etc.) and `SpriteAnimationComponent` for game-world logic.
- Implement Flutter `AnimationController` + `Tween` for UI elements.
- Piece pickup scale bounce, DragOffset lift.
- Piece placement Thud + Grid Ripple.
- Invalid placement Shake Rejection.
- Line Clear Flash sequence.
- Enemy Hit Shake and HP bar smooth lerp.

**Files to Create:**
- `lib/game/effects/` contents for Flame effects.
- Use `AnimatedWidget` / `AnimatedBuilder` for UI elements in `lib/presentation/`.

**Acceptance Criteria:** The game generates the "Tactile Snap" feeling. Placing pieces feels physical. Clearing a line feels satisfying.

---

## Phase 15 — VFX

**Goal:** Particle effects from `13_VFX_PARTICLE_AND_IMPACT_SPEC.md` are implemented.

**Prerequisites:** Phase 14.

**Tasks:**
- Implement `vfx_manager.dart` using Flame component reuse patterns.
- Create Flame particle components or custom rendering for dust, line clears, enemy hits.
- Wire Flame effects to stream events.

**Files to Create:**
- `lib/game/effects/vfx_manager.dart`
- `lib/game/effects/place_dust_effect.dart`
- `lib/game/effects/line_clear_effect.dart`

**Acceptance Criteria:** A line clear spawns a visible burst of colorful particles. Enemy HP reaching 0 triggers a shatter effect. No VFX causes major frame drops.

---

## Phase 16 — Audio

**Goal:** All SFX and Music from `14_AUDIO_SFX_MUSIC_AND_MIXING.md` are implemented.

**Prerequisites:** Phase 15.

**Tasks:**
- Implement `audio_service.dart` using `FlameAudio`.
- Setup `FlameAudio.play()` for SFX, `FlameAudio.playLongAudio()` for background music.
- Wire domain events to SFX playback.
- Implement placeholder audio assets in `assets/audio/`.
- Setup `SharedPreferences`-backed volume control.

**Files to Create:**
- `lib/application/audio_service.dart`
- `assets/audio/` files

**Acceptance Criteria:** The game sounds cohesive and reactive. Muting volume in settings silences everything. Backgrounding the app pauses music.

---

## Phase 17 — Persistence (Save/Load)

**Goal:** Player progress survives app close and reopen.

**Prerequisites:** Phase 11.

**Tasks:**
- Implement `save_repository.dart` using Hive CE.
- Setup `runBox` and `profileBox`.
- Implement `run_save_data.dart` and `player_profile_data.dart`.
- Handle exact deterministic state restoration (RNG calls, tray, enemy action).
- Hook Flutter app lifecycle events for emergency saving.

**Tests:**
- Mock Hive boxes in `dart test` for serialization roundtrip.
- Save/Load determinism: Save state, load, and assert exactly matching RNG behavior.
**Acceptance Criteria:** Closing the app mid-run and reopening restores the exact logical state (Seed + RNG state + board + tray + enemy telegraph + objective progress). Stardust survives a force-quit.

---

## Phase 18 — Settings & Accessibility

**Goal:** All settings from `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` are functional.

**Prerequisites:** Phase 16, Phase 17.

**Tasks:**
- Implement `settings_repository.dart` using `SharedPreferences`.
- Build `settings_screen.dart` with Volume sliders, Haptics toggle, Reduced Motion toggle.
- Expose `ReducedMotion` via Riverpod, disabling Flame effects durations and Flutter animation durations when active.
- Utilize Flutter `SafeArea` widget for edge padding.

**Files to Create:**
- `lib/data/persistence/settings_repository.dart`
- `lib/presentation/settings/settings_screen.dart`

**Tests:** Assert `ReducedMotion = true` suppresses duration of animations via `flutter test`.
**Acceptance Criteria:** All settings persist. Reduced Motion eliminates all non-essential animations. Safe area is correctly padded on devices.

---

## Phase 19 — Monetization

**Goal:** Opt-in Rewarded Ads and Premium Unlock are functional.

**Prerequisites:** Phase 18.

**Tasks:**
- Implement `monetization_service.dart`.
- Integrate Google Play Billing via `in_app_purchase` package.
- Integrate Google Mobile Ads via `google_mobile_ads` package.
- Wire Rewarded Ad button in Defeat overlay.
- Wire IAP "Remove Ads" to disable ad prompts.

**Files to Create:**
- `lib/application/monetization_service.dart`

**Acceptance Criteria:** A Rewarded Ad can be watched for a Revive. The Premium Unlock removes all ad prompts. Ad SDKs use Test IDs in QA builds. (M8 Milestone Achieved — content-complete MVP)

---

## Phase 20 — QA, Performance, and Polish

**Goal:** The game meets all Release Gates from `27` and `26`.

**Prerequisites:** All 19 phases complete.

**Tasks:**
- Execute `flutter run --profile` and analyze using Flutter DevTools timeline on minimum-spec device. Fix any frame time violations.
- Run memory leak tests via DevTools.
- Execute full test suite (`dart test` / `flutter test`) and achieve >90% Domain coverage.
- Fix all S0 and S1 bugs.
- Conduct external playtest session (5 people).

**Acceptance Criteria:** All automated tests pass. Zero S0 bugs. Performance gates cleared via DevTools.

---

## Phase 21 — Release Preparation

**Goal:** Build is submitted to Google Play.

**Prerequisites:** Phase 20.

**Tasks:**
- Generate Keystore (`28` Sec 11). Back it up securely.
- Compile Release Build via `flutter build appbundle` with Production Ad Unit IDs.
- Upload to Google Play Console.
- Complete Store Listing (Screenshots, Description, Feature Graphic) per `28` Sec 18.
- Set initial Staged Rollout to 10%.

**Acceptance Criteria:** The game is live in Google Play Internal Testing and passes Google's automated pre-launch report. (M9 Milestone Achieved — Release Candidate)

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
Ask: "Implement `combat_engine.dart` based on `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 3."

### 2. Provide Full Context
When prompting, paste the relevant Markdown sections as context. Do not assume the AI remembers the correct values from previous sessions.

### 3. Analyze After Structural Changes
After any interface updates, file creation, or package modifications — run `flutter analyze` immediately before continuing. Structural errors cascade catastrophically. Ensure package imports from flutter/flame must not appear in `lib/domain/`.

### 4. Test After Gameplay Changes
After any Domain Layer modification (Board, Combat, Relics), run `dart test / flutter test` before proceeding.

### 5. Visual Review After UI Changes
After any Widget or UI modification, run `flutter run` on an Android device to check layout, safe areas, and font sizing.

### 6. Commit After Stable Milestones
Commit to Git at every Milestone (`M1` through `M9`). Tag the commit (`git tag -a M1 -m "Playable puzzle loop"`).

---

# PART 6: QUALITY GATES

| Gate | Threshold | Tool |
|---|---|---|
| Architecture Gate | Zero domain layer flutter/flame imports. | `dart analyze` |
| Gameplay Gate | All 20+ domain dart tests pass. | `dart test` |
| UX Gate | No modal blocks input unintentionally on any test device. | Manual device test. |
| Visual Gate | Art matches `09`/`10`/`11` color tokens and material spec. | Screenshot review. |
| Audio Gate | All core gameplay events trigger the correct SFX. | Ear test + Event stream log. |
| Performance Gate | 60 FPS on minimum spec. | Flutter DevTools on device. |
| QA Gate | All `27` Automated Tests pass. Zero S0/S1 bugs. | `flutter test` + Bug tracker. |
| Release Gate | Complete `28` Sec 21 checklist item by item. | `flutter build appbundle` check. |

---

# PART 7: PRODUCTION SCHEDULE MODEL

Work units are expressed in "Sessions" (approximately 2-4 focused hours of solo development each).

| Phase | Estimated Sessions | AI Acceleration Potential |
|---|---|---|
| Phase 0 (Setup) | 2 sessions | High (AI generates boilerplate/interfaces) |
| Phase 1 (Board Model) | 2 sessions | High (Pure Dart math) |
| Phase 2 (Shape System) | 2 sessions | High (Dart const catalog definitions + matrix logic) |
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
| Phase 14 (Animation) | 3 sessions | Medium (Flame effect timing values require feel-testing) |
| Phase 15 (VFX) | 3 sessions | Medium |
| Phase 16 (Audio) | 3 sessions | Low (audio mixing is subjective) |
| Phase 17 (Persistence) | 3 sessions | High (pure Dart serialization) |
| Phase 18 (Settings) | 2 sessions | High |
| Phase 19 (Monetization) | 3 sessions | Medium (requires test account setup) |
| Phase 20 (QA) | 5 sessions | Medium (profiling requires device + judgment) |
| Phase 21 (Release) | 3 sessions | Low (store assets require human creative) |

**Where AI excels:** Generating boilerplate classes, writing Dart/Flutter test suites, Generating Dart catalog constants, translating Markdown spec into Dart code.
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
