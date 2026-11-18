# 30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md
## Block Battles — Streamlined Master Roadmap & Execution Order
### Pure Arcade Engine: Classic Endless High-Score + Goal-Driven Architecture

**Governing documents:** ALL `00_MASTER_GAME_VISION.md` through `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md`  
**Document status:** Layer D — Production Authority  
**Architecture:** 5 Streamlined Master Phases (Minimum Phases for God-Level Result)

---

## Governing Principle

This roadmap is organized by **strict technical dependency** and **instant visual/gameplay excellence**.

Instead of deferring visual polish, juice, audio, and game feel to late stages while building extraneous bloat, **visual richness and physical tactile juice are core first-class citizens from Day One.**

Every single phase produces a verified, visually stunning, test-passing deliverable.

---

# PART 1: 5 MASTER PHASES

---

## Phase 1 — Pure Engine Foundations (Domain Layer)

**Goal:** Establish a rock-solid, 100% pure Dart mathematical foundation for the 8×8 board, 34 canonical shapes, deterministic seeded RNG, and 3-piece tray generation with solvability protection.

**Why First:** The spatial grid, polyomino geometry, and piece generation are the fundamental mathematical rules everything else builds on.

**Prerequisites:** Flutter baseline established (`lib/`, `test/`).

**Micro-Tasks:**
1. **BoardModel (Domain):**
   - 8×8 grid with 1D flat array storage (64 cells, index `row * 8 + col`).
   - Occupancy (`empty`, `filled`) and Modifiers (`normal`, `blocked`, `frozen`).
   - Pure semantic predicates: `canReceiveBlock(cell)` and `countsTowardLineClear(cell)`.
   - Read-only placement validation: `validatePlacement(shape, row, col)`.
   - Atomic placement commit: `commitPlacement(shape, row, col)` (all-or-nothing mutation).
2. **Canonical Shape System (Domain):**
   - Author all 34 canonical shapes (`SHP_001`–`SHP_034`) from `02_SHAPE_LIBRARY.md` as immutable Dart constants.
   - Geometry validation: bounding box, normalization to `(0, 0)`, orthogonal edge connectivity via BFS.
   - Strict static orientation (no runtime rotation).
3. **Deterministic RNG (`GameRNG`):**
   - Mulberry32 32-bit PRNG with seed tracking, monotonic draw index, and state reconstruction.
4. **Tray System & Solvability Protection (`TrayEngine` & `ShapeGenerator`):**
   - 3-piece tray slots (`TraySlot`).
   - Two-stage generation: Family draw → Orientation draw.
   - Duplicate shape ID prevention within a batch.
   - Table 17.3 Solvability Protection: 3 redraw retries; fallback to `SHP_001` Single Block if board space is critical.
   - Slot consumption and automatic 3-slot batch replenishment.

**Files to Create:**
- `lib/core/game_constants.dart`
- `lib/domain/board/board_cell.dart`
- `lib/domain/board/board_model.dart`
- `lib/domain/shapes/shape_matrix.dart`
- `lib/domain/shapes/shape_definition.dart`
- `lib/data/definitions/shape_catalog.dart`
- `lib/domain/rng/game_rng.dart`
- `lib/domain/tray/shape_generator.dart`
- `lib/domain/tray/tray_engine.dart`

**Verification:**
- `dart test` passes 100% of domain tests (bounds, collision, solvability, RNG determinism).
- `flutter analyze` passes with 0 issues.
- Zero Flutter/Flame imports in `lib/domain/`.

---

## Phase 2 — Tactile Visual Presentation & Drag-and-Drop UX ("Killing AI Slop" Part 1)

**Goal:** Transform the raw board and tray into a physical, responsive, luxury game surface. Eliminate generic flat squares with rich beveled materials, elastic drag pickup, and live ghost previews.

**Prerequisites:** Phase 1.

**Micro-Tasks:**
1. **Premium Visual Styling & Design Tokens:**
   - Define color palettes and design tokens (`09_VISUAL_DESIGN_SYSTEM.md`, `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`).
   - Rich block materials: Beveled borders, subtle inner shadows, gradient fills (`11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`).
   - Frosted glass outer board container with rounded corners and glowing border.
2. **Interactive Flame Game Canvas (`BlockBattlesGame` & `BoardComponent`):**
   - Responsive viewport auto-scaling for mobile devices.
   - Pre-allocated cached Paint objects (zero allocations in `render()`).
3. **Tray Component & Piece Presentation (`TrayComponent` & `PieceComponent`):**
   - 3 interactive tray pedestals centered below the board.
   - Auto-scaled piece rendering with drop shadows.
4. **Tactile Drag-and-Drop UX (`DragHandler`):**
   - Single-active-touch multi-touch palm rejection.
   - Touch pickup lift: piece scales up slightly (`1.05x`) with vertical finger-occlusion offset (48dp above touch point).
   - Live snap-to-grid Ghost Preview:
     - Valid placement: Translucent emerald green fill (`#22C55E80`) with bright border.
     - Invalid/Blocked placement: Translucent ruby red fill (`#EF444480`) with color-blind accessible 'X' conflict markers.
   - Drop release: Valid placement commits atomically; invalid release smoothly springs back to tray slot.
5. **Haptic & Audio Touch Hooks:**
   - Haptic click on pickup; light tick on grid-cell snap crossing.

**Files to Create / Modify:**
- `lib/game/block_battles_game.dart`
- `lib/game/components/board_component.dart`
- `lib/game/components/cell_component.dart`
- `lib/game/components/piece_component.dart`
- `lib/game/components/tray_component.dart`
- `lib/game/input/drag_handler.dart`
- `test/game/components/board_component_test.dart`
- `test/game/components/tray_component_test.dart`
- `test/game/input/drag_handler_test.dart`

**Verification:**
- Full touch drag, lift offset, snap preview, and atomic placement work smoothly on Android hardware.
- Unit tests verify touch lifecycle, coordinate mapping, and rejection rules.

---

## Phase 3 — Explosive Line Clearing & Game Feel Juice ("Killing AI Slop" Part 2)

**Goal:** Deliver maximum audiovisual dopamine when rows and columns clear. Implement simultaneous line detection, white-hot flash highlights, particle burst dissipation, screen shake, and harmonic audio chimes.

**Prerequisites:** Phase 2.

**Micro-Tasks:**
1. **Simultaneous Line Detection (`LineScanner`):**
   - Evaluates full rows and full columns simultaneously against post-placement board state.
   - Qualification predicate: `countsTowardLineClear(cell) == true` (`FILLED + NORMAL`).
   - Frozen cells and blocked cells protect lines from completing.
2. **Atomic Resolution (`LineClearResolver`):**
   - Builds deduplicated set union $cellsToClear = \{ (r, c) \mid r \in rows \} \cup \{ (r, c) \mid c \in cols \}$.
   - Intersection cells cleared exactly once; cells reset to canonical `EMPTY + NORMAL`.
   - Returns immutable `LineClearResolution` (L-Value = total lines cleared).
3. **Decoupled Event Dispatcher (`EventDispatcher`):**
   - Dispatches strongly-typed broadcast events: `PlacementCommitted` $\to$ `LinesCleared` & `CellsCleared` $\to$ `BoardChanged`.
4. **Explosive Line Clear Visuals & Game Feel:**
   - White-hot flash overlay along cleared rows and columns with alpha fade.
   - Bouncy cell dissipation animation (cells shrink and pop).
   - Particle burst spawner across cleared coordinates.
   - Micro camera shake on multi-line clears (intensity scales with L-Value: L1 = subtle, L3+ = heavy).
5. **Harmonic Audio Chimes:**
   - Pentatonic chord arpeggios that rise in pitch on multi-clears.

**Files to Create / Modify:**
- `lib/core/events/domain_event.dart`
- `lib/core/events/event_dispatcher.dart`
- `lib/domain/board/line_scanner.dart`
- `lib/domain/board/line_clear_resolver.dart`
- `lib/game/components/board_component.dart` (flash overlay & particle emitters)
- `test/core/events/event_dispatcher_test.dart`
- `test/domain/board/line_scanner_test.dart`
- `test/domain/board/line_clear_resolver_test.dart`
- `test/game/line_clearing_integration_test.dart`

**Verification:**
- Placing pieces that complete rows/cols triggers simultaneous clear, flash feedback, and event sequence.
- All unit and integration tests pass with 100% coverage.

---

## Phase 4 — Classic Endless Mode, Score Engine & Local Device High Score

**Goal:** Complete the full Classic Endless Mode gameplay loop. Score points for placements, line clears, multi-clears, and consecutive streaks; detect Board Lock (game over); persist device High Score; present a celebratory Game Over dialog with instant restart.

**Prerequisites:** Phase 3.

**Micro-Tasks:**
1. **Pure Puzzle Scoring System (`ScoreEngine`):**
   - Block Placement Points: +1 point per placed block.
   - Line Clear Base Points:
     - 1 Line (L1) = 100 pts
     - 2 Lines (L2) = 300 pts
     - 3 Lines (L3) = 600 pts
     - 4 Lines (L4) = 1,000 pts
     - 5+ Lines = $1000 + (L - 4) \times 500$ pts
   - Consecutive Turn Streak / Combo Multiplier:
     - Turn 1 clear = $1.0\times$
     - Turn 2 consecutive clear = $1.5\times$
     - Turn 3 consecutive clear = $2.0\times$
     - Turn 4+ consecutive clear = $2.5\times$ (streak continues until a placement clears 0 lines).
   - Floating score popups (`+120`, `STREAK x2!`) spawned at clear origins.
2. **Board Lock Detection (`BoardLockDetector`):**
   - Runs at the end of each turn after tray consumption / refill.
   - Tests all remaining tray pieces across all 64 board origins using `boardModel.validatePlacement()`.
   - If zero legal placements exist across all tray pieces $\to$ fires `BoardLockDetected` event.
3. **Local High Score Persistence (`SaveRepository`):**
   - Offline storage (Hive CE for progression and session data; SharedPreferences for settings).
   - Stores `classicHighScore` in Hive CE on the local device.
   - Emits `NewHighScoreAchieved` when player exceeds prior record during active gameplay.
4. **Classic Game HUD:**
   - Sleek in-game header: Current Score, Trophy icon with All-Time Best score, active Combo Streak badge, Pause / Audio toggle.
5. **Game Over Modal & Instant Replay Loop:**
   - Polished dialog over frosted gameplay canvas:
     - Final Score tally.
     - "NEW RECORD!" glowing badge with celebratory audio fanfare if high score beaten.
     - One-tap "PLAY AGAIN" button (resets board, RNG, tray instantly).
     - "HOME" button.

**Files to Create / Modify:**
- `lib/domain/scoring/score_engine.dart`
- `lib/domain/board/board_lock_detector.dart`
- `lib/data/persistence/save_service.dart`
- `lib/application/classic_game_service.dart`
- `lib/presentation/game/classic_game_screen.dart`
- `lib/presentation/widgets/game_over_dialog.dart`
- `lib/presentation/widgets/score_hud_widget.dart`
- `test/domain/scoring/score_engine_test.dart`
- `test/domain/board/board_lock_detector_test.dart`
- `test/data/persistence/save_service_test.dart`

**Verification:**
- Full endless loop works seamlessly: play $\to$ score $\to$ board lock $\to$ high score saved $\to$ one-tap replay restarts game instantly.

---

## Phase 5 — App Shell, Animated Splash Screen, Minimalist Home & Goal-Driven Hooks

**Goal:** Wrap the engine into a release-ready, clutter-free mobile app. Add smooth animated Splash Screen, clean Home Screen, audio/haptic toggles, and reserve decoupled data contracts for the future Goal-Driven mode.

**Prerequisites:** Phase 4.

**Micro-Tasks:**
1. **Animated Splash Screen (`SplashScreen`):**
   - Smooth branded animation on app icon launch (fade-in emblem, glow pulse, seamless route to Home).
   - Asset and audio pre-caching during splash.
2. **Minimalist Home Screen (`HomeScreen`):**
   - Clean, uncluttered UI:
     - Prominent "PLAY CLASSIC" button with breathing glow.
     - High Score Trophy card displaying device personal best.
     - Audio & Haptics toggle pills.
     - Sleek "GOAL MODE (Coming Soon / Adventure)" card reserving space for Mode 2.
3. **App Navigation (`AppRouter`):**
   - Clean GoRouter navigation with smooth cross-fade transitions:
     - `/splash` $\to$ `/` (Home) $\to$ `/classic` / `/stages`
4. **Goal-Driven Mode Architecture (Future Foundation):**
   - Define data schema and models for Mode 2 (`LevelObjective`, `HurdleType`, `GoalLevelConfig`):
     - Target Score hurdle
     - Pre-filled obstacle clearing
     - Move limit hurdle
   - Kept strictly decoupled in `lib/domain/goal_mode/` so it never pollutes Classic Mode.
5. **Release Polish & Packaging:**
   - Audio volume sliders and haptic feedback settings.
   - Android back-button handling, safe-area compliance on all screen aspect ratios.
   - Production Android build verification (`flutter build apk --release`).

**Files to Create / Modify:**
- `lib/app/app.dart`
- `lib/app/app_router.dart`
- `lib/presentation/splash/splash_screen.dart`
- `lib/presentation/home/home_screen.dart`
- `lib/domain/goal_mode/goal_level_config.dart`
- `lib/data/persistence/settings_service.dart`
- `test/presentation/home/home_screen_test.dart`

**Verification:**
- Full app lifecycle: Tap app icon $\to$ Splash $\to$ Home $\to$ Play Classic $\to$ Game Over $\to$ Return Home.
- Release APK builds cleanly, passes `flutter analyze` with 0 issues, and passes 100% of test suites.

---

# PART 2: THE 5 CRITICAL MILESTONES

| Milestone | Phase | What Must Work | Playable State |
|---|---|---|---|
| **M1: Pure Engine Verified** | Phase 1 | 8×8 Board, 34 Shapes, RNG, 3-slot Tray, Solvability Protection. | Pure Dart Domain (All tests green). |
| **M2: Tactile Surface Playable** | Phase 2 | Beveled materials, responsive canvas, drag lift, ghost preview, snap placement. | Playable placement toy on device. |
| **M3: Explosive Clear Loop** | Phase 3 | Simultaneous line clear, white-hot flash, particle bursts, screen shake, audio chimes. | Satisfying arcade line clearing loop. |
| **M4: Classic Endless Game** | Phase 4 | Full scoring, combo streaks, board lock game over, local high score, replay dialog. | Complete endless high-score game! |
| **M5: Release-Ready App** | Phase 5 | Splash screen, minimalist home screen, settings, goal-mode hooks, release APK. | Release-ready, polished mobile product. |

---

# PART 3: QUALITY GATES

1. **Architecture Gate:** Zero Flutter/Flame imports in `lib/domain/`.
2. **Visual Gate:** No "AI slop" — all blocks have bevels, shadows, and materials.
3. **Game Feel Gate:** Drag has finger-occlusion lift; line clears have flash and particle bursts.
4. **Test Gate:** 100% of unit and integration tests pass on `flutter test`.
5. **Static Analysis Gate:** `flutter analyze` passes with 0 warnings and 0 errors.

---

**End of `30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md`.**
