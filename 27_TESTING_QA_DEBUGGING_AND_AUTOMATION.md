# 27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md
## Block Battles — Testing, QA, Debugging, and Automation

**Governing documents:** `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `18_TECHNICAL_ARCHITECTURE.md`, `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document establishes the rigorous testing and validation protocols necessary to uphold the clean architecture defined in Doc 18 and the performance limits in Doc 26. It relies entirely on the testability of decoupled systems to allow a solo developer and AI agents to maintain high quality without manual drudgery.

---

## 1. QA Philosophy

**"Automate the Math, Playtest the Feel."**

Because the core game logic (`Domain Layer`) is completely decoupled from Flame's rendering and Flutter's UI (`18`), we can automate 100% of the game rules. A solo developer should never have to manually drag a block onto a grid to see if a line clears. Manual testing is reserved strictly for Game Feel (`12`), UI layout (`23`), and device compatibility.

---

## 2. Testing Pyramid

For Block Battles, the effort distribution is:

1. **Unit Tests (70%):** Pure Dart (`dart test`). Tests Board math, Piece placement (fixed orientation), Combat calculations, RNG, Objectives, Relics, and Run state. Executes in milliseconds with no Flutter context needed. File location: `test/domain/`.
2. **Widget Tests (15%):** Flutter Widget tests (`flutter test`). Tests `HomeScreen`, `BattleScreen` HUD overlays, settings menus, navigation (`GoRouter`), and data binding. File location: `test/presentation/`.
3. **Integration Tests (10%):** Full app end-to-end tests using the `integration_test` package. Simulates app launch, battle flow, run flow, and Hive IO. File location: `integration_test/`.
4. **Manual / Playtesting (5%):** Real-device testing for haptics, framerate, and subjective fun.

---

## 3. Gameplay Tests

- **Turn State Progression:** Assert that completing an action transitions the `BattleOrchestrator` appropriately based on `01` Section 4 states.
- **Turn Locking:** Assert that injecting a drag-and-drop command during resolution is instantly rejected.
- **Test A (Board Lock):** Assert that when the board has no valid moves for the tray, `BoardService` emits `BoardLockDetected`.
- **Test B (Battle Failure):** Assert that a `BoardLockDetected` event (or failing an Objective) causes the Orchestrator to emit `BattleFailed`.
- **Test C (Run Defeat):** Assert that `BattleFailed` causes the `RunManager` to emit `RunDefeated` (one-life run ends).
- **Test D (Victory):** Assert that when the enemy dies (or Primary Objective completes), the Orchestrator emits `BattleVictory`.

---

## 4. Board Tests

- **Bounds Checking:** Assert that placing any piece outside the 0-7 X/Y coordinates throws a `RangeError` or returns false.
- **Overlap Detection:** Assert that placing a piece on an occupied cell returns false.
- **Line Clearing (Single):** Assert that a full row resets its cells to 0 and yields a line clear result with L=1.
- **Line Clearing (Multi):** Assert that filling a vertical and horizontal line simultaneously yields an L=2 event, prioritizing intersection cells correctly.

---

## 5. Shape Tests

- **Orientation Immutability:** Assert that a `ShapeData` model's block matrix returns the same cell configuration on every read, confirming the orientation is authored and static. No runtime transformation must alter it.
- **Anchor Offset:** Assert that the piece's logical anchor (e.g., bottom-left) correctly translates 5x5 local coordinates to 8x8 global coordinates.

---

## 6. RNG Tests

- **Determinism Guarantee:** Assert that generating 1,000 pieces with `GameRNG` seeded with `12345` produces the exact same sequence of 1,000 pieces every time. (Pure Dart test).
- **Bag Depletion:** Assert that standard shapes use a Bag system (`04` Sec 3) and no shape repeats until the bag is empty.

---

## 7. Combat Tests

- **Damage Formula:** Assert that Base Damage (10) * L=3 Multiplier (2.0) = 20 Damage.
- **Enemy Defense:** Assert that if an Enemy has an Armor modifier of 50%, the final damage applied to HP is 10.
- **Lethality:** Assert that dealing damage >= Enemy HP raises `EnemyDefeatedEvent` and reduces HP to exactly 0 (no negative HP).

---

## 8. Relic Tests

- **Trigger Validation:** Assert that "Combo Capacitor" only triggers if L >= 2.
- **Condition Validation:** Assert that "Executioner" only triggers if Enemy HP < 30%.
- **Modifier Stacking:** Assert that two relics granting +50% multiplier stack additively (1.0 + 0.5 + 0.5 = 2.0x), not multiplicatively, unless specifically coded to do so.

---

## 9. Objective Tests

- **Tracking:** Assert that completing a Line Clear decrements the `LineObjective` target value.
- **Completion:** Assert that hitting 0 on an objective marks it as completed and prevents further decrements.

---

## 10. UI Tests (Widget Tests)

- **Modal Stacking:** Assert that opening Settings, then opening Compendium, then pressing "Close" pops the route to Settings, not the HUD.
- **Data Binding:** Assert that changes to the `enemyProvider` in Riverpod updates the EnemyHP `Text` widget instantly.
- **Animation Blocking:** Assert that Flame animations complete and fire corresponding callbacks before user input is accepted.

---

## 11. Save Tests

- **Serialization Roundtrip:** Create a `RunSaveData` object, serialize to JSON/Map, deserialize back, and assert all fields match perfectly.
- **Migration:** Pass a V1 Save Map into the V2 Migrator (`24`) and assert it successfully restructures the missing variables before saving to Hive.
- **Corruption Handling:** Corrupt a Hive box or JSON string and assert the `SaveRepository` catches the exception and falls back to a backup or starts a fresh state safely.

---

## 12. Input Tests

- **Ghost Preview:** Assert that dragging a 3x3 piece over the grid boundary correctly evaluates the clipped coordinates as invalid.
- **Touch Occlusion:** Assert that the dragged visual representation in Flame is offset accurately above the logical pointer position to avoid finger occlusion.

---

## 13. Audio / VFX Tests

- **Mocking:** Use `mockito` or `mocktail` to mock the `AudioService`. Assert that `LinesClearedEvent` successfully calls `mockAudioService.playSfx(SfxId.lineClear)`.
- **Pool Depletion:** Assert that requesting 50 particles from a pool safely recycles the oldest particles without throwing errors.

---

## 14. Accessibility Tests

- **Reduced Motion:** Assert that setting `ReducedMotion = true` in `SharedPreferences` forces all UI animation durations in the presentation layer to `0.0` (instant snap).
- **Semantic Readability:** (Manual) Ensure Flutter `Semantics` nodes properly enunciate dynamic text updates for screen readers.

---

## 15. CI and Automation

- **Continuous Integration (CI):** Every commit to the `main` branch automatically triggers the testing pipeline.
- **Pipeline Steps:**
  1. `dart format --set-exit-if-changed lib/ test/`
  2. `flutter analyze`
  3. `flutter test`
  4. `flutter test --coverage`
- **Build Failure:** A build cannot be merged or compiled if a single Dart test fails or if `flutter analyze` finds issues. This prevents broken logic from reaching a device.

---

## 16. Device Matrix

| Tier | OS | Example Device | Purpose |
|---|---|---|---|
| **Minimum** | Android 8.0 | Galaxy S8 | Test absolute performance floor, 3GB RAM constraints, thermal throttling. |
| **Minimum** | iOS 12.0 | iPhone 8 | Test lowest-end Apple silicon, safe-area notches. |
| **Recommended**| Android 12+ | Pixel 6 / S21 | Test haptics, 60fps stability. |
| **Edge Case** | Android/iOS | iPad / Galaxy Fold | Test UI anchors, SafeArea, and responsive layouts on 4:3 / 1:1 screens. |

---

## 17. Deterministic Reproduction (Seed Hashing)

Because the game domain is deterministic, bugs are 100% reproducible if the seed and input sequence are known. (Pure Dart test).

- **The Debug Seed:** The Developer Console allows inputting a custom seed (e.g., `BUG_123`).
- **Input Logging:** When in Development mode, the game logs an array of grid coordinates clicked `[ (4,5), (1,2), (3,3) ]`.
- **Replay:** The developer can instantly reproduce the bug by initializing the pure Dart test with that seed and passing the coordinates.

---

## 18. Debugging and Tools

- **Flutter DevTools:** Used for performance profiling, widget inspection, and memory tracking.
- **Logcat / flutter logs:** Used to monitor real-time output from the game on a connected device.
- **Developer Console (Cheat Overlay):** Implemented as a Flutter debug overlay, accessible via rapid taps on the version number in settings.

**Available Commands:**
- `Set_Currency [amount]`
- `Force_Drop [ShapeID]` (Spawns a specific piece in the tray)
- `Force_Relic [RelicID]`
- `Kill_Enemy` (Sets HP to 0)
- `Board_Clear`
- `Toggle_GodMode` (Infinite moves)
- `Log_State` (Dumps active seed and board matrix to clipboard)

---

## 19. Automated Test Requirements

To support AI coding agents writing tests:
- Tests must use standard Dart test framework functions (`test()`, `group()`, `setUp()`, `tearDown()`).
- Tests must adhere to the **Arrange, Act, Assert (AAA)** pattern.
- Mocking is handled using `mockito` or `mocktail` to isolate dependencies like `SaveRepository` or `AudioService`.
- Domain tests must reside in `test/domain/` and never import `flutter/material.dart` or Flame packages.

---

## 20. Bug Severity

| Level | Definition | Rule |
|---|---|---|
| **S0 (Blocker)** | Hard crash, freeze, corrupted Hive box. | **Stop everything.** Cannot release. |
| **S1 (Major)** | Core logic break (piece won't place, combo math wrong). | Must be fixed before RC. |
| **S2 (Minor)** | UI overlap, missing SFX, dropped frames on L4 clear. | Logged. Fix if time permits. |
| **S3 (Cosmetic)**| Typo, off-by-one-pixel padding. | Logged to backlog. |

---

## 21. Bug Report Template

If an issue is found, it must be reported with:
1. **Title:** [Component] Summary (e.g., [Board] 3x3 piece allows out-of-bounds drop).
2. **Device / OS:** (e.g., Pixel 6, Android 13).
3. **Seed:** (e.g., RunSeed: 40192A).
4. **Steps to Reproduce:** (1, 2, 3).
5. **Expected Result:** (Piece is rejected).
6. **Actual Result:** (Piece places, game throws RangeError).

---

## 22. Release Gates

A Release Candidate (RC) build is **REJECTED** if:
1. Automated Test coverage (`flutter test --coverage`) is < 90% for the Domain Layer.
2. A single Dart unit test or Flutter widget test fails.
3. `flutter analyze` returns any warnings or errors.
4. Any S0 or S1 bugs remain open in the tracker.
5. The game fails to compile via the CI/CD pipeline.

---

## 23. Final QA Contract

> **Quality is built in, not bolted on.**
>
> By isolating our logic into a pure, testable Dart Domain layer, we guarantee that the math of Block Battles works flawlessly before a single pixel is drawn by Flame or Flutter. 
> 
> We rely on deterministic seeds to eliminate "ghost bugs," we use clean architecture to eliminate side effects, and we build debug tools to empower rapid iteration. We do not hope the game works; we mathematically prove it works.

---

**End of `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md`.**
