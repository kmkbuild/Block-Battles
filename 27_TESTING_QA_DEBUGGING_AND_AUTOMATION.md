# 27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md
## Block Battles — Testing, QA, Debugging, and Automation

**Governing documents:** `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `18_TECHNICAL_ARCHITECTURE.md`, `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document establishes the rigorous testing and validation protocols necessary to uphold the clean architecture defined in Doc 18 and the performance limits in Doc 26. It relies entirely on the testability of decoupled systems to allow a solo developer and AI agents to maintain high software quality across **Mode 1 (Classic Endless)** and **Mode 2 (Goal-Driven Stage Challenge)**.

---

## 1. QA Philosophy

**"Automate the Math, Playtest the Feel."**

Because the core game logic (`Domain Layer`) is completely decoupled from Flame's rendering and Flutter's UI (`18`), we can automate 100% of the puzzle rules. A solo developer should never have to manually drag a block onto a grid to see if a line clears. Manual testing is reserved strictly for Game Feel (`12`), UI layout (`23`), and device compatibility.

---

## 2. Testing Pyramid

For Block Battles, the testing effort is distributed as follows:

1. **Unit Tests (70%):** Pure Dart (`dart test`). Tests Board grid math, Piece placement, Scoring formulas, Streak multipliers, Stage objectives, RNG determinism, and Save serialization. Executes in milliseconds with no Flutter context needed. File location: `test/domain/`.
2. **Widget Tests (15%):** Flutter Widget tests (`flutter test`). Tests `HomeScreen`, `ClassicGameScreen`, `StageGameScreen`, HUD overlays, settings menus, navigation (`GoRouter`), and data binding. File location: `test/presentation/`.
3. **Integration Tests (10%):** Full app end-to-end tests using the `integration_test` package. Simulates app launch, session flows, stage transitions, and Hive disk I/O. File location: `integration_test/`.
4. **Manual / Playtesting (5%):** Real-device testing for haptics, framerate, touch feel, and overall player experience.

---

## 3. Gameplay Session Tests

- **Turn State Progression:** Assert that placing a piece transitions state through validation, placement, line clear detection, score addition, and replenishment.
- **Turn Locking:** Assert that injecting drag commands during clear animation resolution is rejected.
- **Board Lock Detection (Mode 1):** Assert that when none of the 3 tray pieces can legally fit onto the 8×8 board, `BoardEngine` emits `BoardLockDetected` and transitions to `GameOver`.
- **Move Limit Exhaustion (Mode 2):** Assert that exhausting allowed moves before meeting all stage goals emits `StageFailedEvent`.
- **Objective Completion (Mode 2):** Assert that satisfying all stage goals emits `StageClearedEvent` and calculates 1–3 stars based on thresholds.

---

## 4. Board Engine Tests

- **Bounds Checking:** Assert that placing any shape outside the 0–7 row/column coordinates throws a `RangeError` or returns false.
- **Overlap Detection:** Assert that placing a piece over an already occupied cell returns false.
- **Line Clearing (Single):** Assert that filling a full horizontal row or vertical column resets its cells to 0 and yields an $L=1$ clear event.
- **Line Clearing (Multi):** Assert that filling a vertical and horizontal line simultaneously yields an $L=2$ event, correctly handling intersecting cells.
- **Full Board Clear:** Assert that clearing all 64 cells yields a `BoardClearedEvent` and awards the +500 bonus.

---

## 5. Shape Library Tests

- **34 Canonical Shapes:** Assert that all 34 shapes are correctly registered in `ShapeCatalog` across the 9 canonical families.
- **Matrix Dimension Invariant:** Assert that every shape definition has a block matrix of exactly 25 elements ($5 \times 5$).
- **Anchor Offset:** Assert that the shape's logical anchor correctly maps local $5 \times 5$ coordinates onto the global $8 \times 8$ board coordinates.

---

## 6. RNG & Spawner Tests

- **Determinism Guarantee:** Assert that generating 1,000 shapes with `GameRNG` seeded with `12345` produces the exact identical sequence every time.
- **Tray Replenishment:** Assert that the tray only draws a new set of 3 shapes after all current 3 shapes have been successfully placed.
- **Unplaceable Safeguard:** Assert that the spawner algorithm guarantees at least one placeable piece in every tray roll.

---

## 7. Scoring Engine Tests

- **Cell Placement Points:** Assert that placing a shape awards exactly 1 point per constituent cell (e.g., a 4-block piece awards +4 points).
- **Line Clear Formulas:** Assert line clear scores match specifications:
  - $L=1 \to 100$ points
  - $L=2 \to 300$ points
  - $L=3 \to 600$ points
  - $L=4 \to 1000$ points
- **Streak Multiplier:** Assert that consecutive placement moves resulting in line clears increment the streak multiplier ($1.0\times, 1.2\times, 1.4\times, \dots, 2.5\times$ cap).
- **Streak Reset:** Assert that a placement move resulting in zero cleared lines resets the streak multiplier back to $1.0\times$.

---

## 8. Stage Objective Tests (Mode 2)

- **Target Score Objective:** Assert that reaching or exceeding the target score marks the score goal as completed.
- **Color Block Objective:** Assert that clearing lines containing blocks of a specific color decrements the color quota counter.
- **Obstacle Clearance:** Assert that clearing lines adjacent to or intersecting stone/ice obstacles damages or removes the obstacle cells.
- **Star Thresholds:** Assert that finishing with score or moves above defined thresholds awards 1, 2, or 3 stars accurately.

---

## 9. UI Tests (Widget Tests)

- **Route Navigation:** Assert that tapping "Classic Mode" pushes `/classic` and tapping "Stage Mode" pushes `/stages`.
- **Score HUD Binding:** Assert that changes to `classicSessionProvider` update the score and streak display text instantly.
- **Modal Display:** Assert that `GameOverOverlay` or `StageClearOverlay` renders when respective session states trigger completion.

---

## 10. Save & Persistence Tests

- **Serialization Roundtrip:** Create an `ActiveSessionData` instance, serialize to Hive format, deserialize back, and assert all 64 grid cells, score, and tray shape IDs match exactly.
- **High Score Persistence:** Assert that finishing a Classic session with a new high score updates `classicHighScore` in `PlayerProgressionData`.
- **Corruption Recovery:** Assert that corrupting a session box safely clears the broken session and allows the game to boot cleanly without crashing.

---

## 11. Input & Drag Tests

- **Ghost Preview Bounds:** Assert that dragging a piece over grid boundaries highlights valid destination cells and suppresses previews for illegal placements.
- **Touch Offset:** Assert that the visual piece preview renders with an upward vertical offset to maintain visibility above the player's finger.

---

## 12. Audio & Haptic Tests

- **Audio Mocking:** Use `mocktail` to mock `AudioService` and verify that `LinesClearedEvent` invokes `mockAudioService.playSfx(SfxId.lineClear)`.
- **Haptic Mocking:** Verify that placing a piece triggers light haptic feedback and clearing 3+ lines triggers heavy haptic feedback.

---

## 13. Accessibility Tests

- **Reduced Motion:** Assert that setting `ReducedMotion = true` disables shake animations and sets visual transition durations to zero.
- **Contrast Check:** Verify that block color indices adhere to minimum contrast standards against the board background.

---

## 14. CI and Automation

Every commit to the repository triggers the automated CI pipeline:
1. `dart format --set-exit-if-changed lib/ test/`
2. `flutter analyze`
3. `flutter test`
4. `flutter test --coverage`

A build cannot be merged or released if any unit test fails or if `flutter analyze` detects warnings or errors.

---

## 15. Device Matrix

| Tier | OS | Example Device | Validation Focus |
|---|---|---|---|
| **Minimum** | Android 8.0 | Galaxy S8 | Performance floor, memory constraints, thermal throttling |
| **Minimum** | iOS 12.0 | iPhone 8 | Safe area layout, older hardware framerate |
| **Recommended**| Android 12+ | Pixel 6 / S21 | 60fps stability, touch latency, haptic response |
| **Edge Case** | Android/iOS | iPad / Galaxy Fold | Responsive layout across 4:3 and square aspect ratios |

---

## 16. Deterministic Reproduction (Seed Hashing)

Because the game domain is deterministic, all board and tray states are 100% reproducible:
- **Debug Seed:** The Developer Console allows inputting a custom seed (`SEED_TEST_01`).
- **Placement Logging:** In development mode, the game logs placed shape IDs and grid coordinates `[shape_l_3x2 @ (2,3)]`.
- **Replay:** Developers can instantly reproduce any edge case by running a unit test initialized with the recorded seed and move sequence.

---

## 17. Developer Console & Debug Tools

In development builds, tapping the version number in Settings opens the Debug Overlay:
- `Force_Tray_Shape [shapeId]` (Places specific shape in slot)
- `Trigger_Line_Clear [row/col]` (Forces line clear event)
- `Add_Score [amount]` (Increments current score)
- `Complete_Stage_Goal` (Instantly completes active stage objectives)
- `Clear_Board` (Wipes all 64 cells to empty)
- `Toggle_Infinite_Moves` (Disables move counter decrement in Mode 2)

---

## 18. Bug Severity

| Level | Definition | Protocol |
|---|---|---|
| **S0 (Blocker)** | App crash, freeze, corrupted save box | Release blocker; immediate resolution required |
| **S1 (Major)** | Core logic flaw (piece won't place, streak math wrong) | Must be resolved before Release Candidate |
| **S2 (Minor)** | Visual artifact, transient dropped frame on clear | Scheduled for sprint fix |
| **S3 (Cosmetic)**| Minor padding discrepancy or typo | Backlog item |

---

## 19. Final QA Contract

> **Quality is built in, not bolted on.**
>
> By isolating our logic into a pure, testable Dart Domain layer, we guarantee that the math of Block Battles works flawlessly before a single pixel is drawn by Flame or Flutter. 
> 
> We rely on deterministic seeds to eliminate non-reproducible bugs, use clean architecture to avoid side effects, and build automated test suites to empower rapid, safe feature iteration.

---

**End of `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md`.**
