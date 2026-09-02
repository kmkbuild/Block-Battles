# 27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md
## Block Battles — Testing, QA, Debugging, and Automation

**Governing documents:** `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `18_TECHNICAL_ARCHITECTURE.md`, `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document establishes the rigorous testing and validation protocols necessary to uphold the clean architecture defined in Doc 18 and the performance limits in Doc 26. It relies entirely on the testability of decoupled systems to allow a solo developer and AI agents to maintain high quality without manual drudgery.

---

## 1. QA Philosophy

**"Automate the Math, Playtest the Feel."**

Because the core game logic (`Domain Layer`) is completely decoupled from Unity's rendering and physics (`18`), we can automate 100% of the game rules. A solo developer should never have to manually drag a block onto a grid to see if a line clears. Manual testing is reserved strictly for Game Feel (`12`), UI layout (`23`), and device compatibility.

---

## 2. Testing Pyramid

For Block Battles, the effort distribution is:

1. **Unit Tests (70%):** Pure C# (NUnit). Tests Board math, Piece placement (fixed orientation), Combat calculations, and Relic triggers. Executes in milliseconds.
2. **Integration Tests (15%):** Tests Save/Load IO, EventBus broadcasting, and turn state-machine progression.
3. **System / UI Tests (10%):** Unity PlayMode tests. Simulates a mock drag-and-drop to ensure the `TrayEngine` and `BoardEngine` communicate correctly.
4. **Manual / Playtesting (5%):** Real-device testing for haptics, framerate, and subjective fun.

---

## 3. Gameplay Tests

- **State Machine Progression:** Assert that completing an action transitions the `BattleOrchestrator` through `BATTLE_IDLE -> RESOLVING_BOARD -> RESOLVING_COMBAT -> WAITING_FOR_ANIM -> BATTLE_IDLE`.
- **Turn Locking:** Assert that injecting a drag-and-drop command during `RESOLVING_COMBAT` is instantly rejected.

---

## 4. Board Tests

- **Bounds Checking:** Assert that placing any piece outside the 0-7 X/Y coordinates throws an `ArgumentOutOfRangeException` or returns false.
- **Overlap Detection:** Assert that placing a piece on an occupied cell returns false.
- **Line Clearing (Single):** Assert that a full row resets its cells to 0 and raises a `LinesClearedEvent` with L=1.
- **Line Clearing (Multi):** Assert that filling a vertical and horizontal line simultaneously raises an L=2 event, prioritizing intersection cells correctly.

---

## 5. Shape Tests

- **Orientation Immutability:** Assert that a `ShapeData` asset's `blockMatrix` returns the same cell configuration on every read, confirming the orientation is authored and static. No runtime transformation must alter it.
- **Anchor Offset:** Assert that the piece's logical anchor (e.g., bottom-left) correctly translates 5x5 local coordinates to 8x8 global coordinates.

---

## 6. RNG Tests

- **Determinism Guarantee:** Assert that generating 1,000 pieces with `GameRNG` seeded with `12345` produces the exact same sequence of 1,000 pieces every time.
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
- **Completion:** Assert that hitting 0 on an objective raises `ObjectiveCompletedEvent` and prevents further decrements.

---

## 10. UI Tests (PlayMode)

- **Modal Stacking:** Assert that opening Settings, then opening Compendium, then pressing "Close" returns to Settings, not the HUD.
- **Data Binding:** Assert that raising `DamageDealtEvent` updates the EnemyHP text component instantly.
- **Animation Blocking:** Assert that DOTween completes and fires `UIAnimationCompleteEvent`.

---

## 11. Save Tests

- **Serialization Roundtrip:** Create a `RunSaveData` object, serialize to JSON, deserialize back, and assert all fields match perfectly.
- **Migration:** Pass a V1 Save JSON into the V2 Migrator (`24`) and assert it successfully restructures the missing variables.
- **Corruption Handling:** Corrupt a JSON string (remove a bracket) and assert the `SaveService` catches the exception and falls back to `.bak`.

---

## 12. Input Tests

- **Ghost Preview:** Assert that dragging a 3x3 piece over the grid boundary correctly evaluates the clipped coordinates as invalid (Red Crosshatch).
- **Touch Occlusion:** Assert that `DragOffset = High` moves the dragged sprite exactly 120dp above the logical touch point.

---

## 13. Audio / VFX Tests

- **Pool Depletion:** Assert that requesting 50 particles from a pool of 30 safely recycles the oldest particles or rejects the request without throwing a `NullReferenceException`.
- **Event Listening:** Assert that `LinesClearedEvent` successfully calls `AudioManager.PlaySFX`.

---

## 14. Accessibility Tests

- **Reduced Motion:** Assert that setting `ReducedMotion = true` forces all UI animation durations in the Application Layer to `0.0f` (instant snap).
- **Semantic Readability:** (Manual) Ensure screen readers properly enunciate dynamic text updates.

---

## 15. Performance Tests

- **Zero Allocation:** (Automated Profiler Test) Run a mocked combat sequence and assert `GC.Alloc` == 0 bytes.
- **Leak Test:** (Automated Profiler Test) Load the Battle Scene 10 times in a row and assert total memory footprint does not climb.

---

## 16. Device Matrix

| Tier | OS | Example Device | Purpose |
|---|---|---|---|
| **Minimum** | Android 8.0 | Galaxy S8 | Test absolute performance floor, 3GB RAM constraints, thermal throttling. |
| **Minimum** | iOS 12.0 | iPhone 8 | Test lowest-end Apple silicon, safe-area notches. |
| **Recommended**| Android 12+ | Pixel 6 / S21 | Test haptics, 60fps stability, ASTC compression. |
| **Edge Case** | Android/iOS | iPad / Galaxy Fold | Test UI anchors and responsive layouts on 4:3 / 1:1 screens. |

---

## 17. Regression Strategy

- **Continuous Integration (CI):** Every commit to the `main` branch automatically triggers the Unity Test Runner in headless mode. 
- **Build Failure:** A build cannot be compiled if a single NUnit test fails. This prevents broken logic from reaching a device.

---

## 18. Deterministic Reproduction (Seed Hashing)

Because the game is deterministic, bugs are 100% reproducible if the seed and input sequence are known.

- **The Debug Seed:** The Developer Console allows inputting a custom seed (e.g., `BUG_123`).
- **Input Logging:** When in Development mode, the game logs an array of grid coordinates clicked `[ (4,5), (1,2), (3,3) ]`.
- **Replay:** The developer can instantly reproduce the bug by initializing the game with that seed and dropping pieces at those coordinates.

---

## 19. Debug Tools (Developer Console)

The game must ship with a hidden debug overlay (accessible via 5 rapid taps on the version number in settings). 

**Available Commands:**
- `Set_Currency [amount]`
- `Force_Drop [ShapeID]` (Spawns a specific piece in the tray)
- `Force_Relic [RelicID]`
- `Kill_Enemy` (Sets HP to 0)
- `Board_Clear`
- `Toggle_GodMode` (Infinite moves)
- `Log_State` (Dumps active seed and board matrix to clipboard)

---

## 20. Automated Test Requirements

To support AI coding agents writing tests:
- Tests must use standard NUnit attributes (`[Test]`, `[SetUp]`, `[TearDown]`).
- Tests must adhere to the **Arrange, Act, Assert (AAA)** pattern.
- Mocking is done manually (via simple mock classes implementing the interface) rather than using heavy reflection-based mocking frameworks like Moq, as they are not supported natively in AOT Unity builds.

---

## 21. Bug Severity

| Level | Definition | Rule |
|---|---|---|
| **S0 (Blocker)** | Hard crash, freeze, corrupted save file. | **Stop everything.** Cannot release. |
| **S1 (Major)** | Core logic break (piece won't place, combo math wrong). | Must be fixed before RC. |
| **S2 (Minor)** | UI overlap, missing SFX, dropped frames on L4 clear. | Logged. Fix if time permits. |
| **S3 (Cosmetic)**| Typo, off-by-one-pixel padding. | Logged to backlog. |

---

## 22. Bug Report Template

If an issue is found, it must be reported with:
1. **Title:** [Component] Summary (e.g., [Board] 3x3 piece allows out-of-bounds drop).
2. **Device / OS:** (e.g., Pixel 6, Android 13).
3. **Seed:** (e.g., RunSeed: 40192A).
4. **Steps to Reproduce:** (1, 2, 3).
5. **Expected Result:** (Piece is rejected).
6. **Actual Result:** (Piece places, game throws IndexOutOfRange).

---

## 23. Release Gates

A Release Candidate (RC) build is **REJECTED** if:
1. Automated Test coverage is < 90% for the Domain Layer.
2. A single Unit Test fails.
3. The GC Allocation Test fails (Allocates > 0 bytes during combat loop).
4. Any S0 or S1 bugs remain open in the tracker.
5. The game fails to compile via the CI/CD pipeline.

---

## 24. Final QA Contract

> **Quality is built in, not bolted on.**
>
> By isolating our logic into a pure, testable Domain layer, we guarantee that the math of Block Battles works flawlessly before a single pixel is drawn on the screen. 
> 
> We rely on deterministic seeds to eliminate "ghost bugs," we use object pools to eliminate lag spikes, and we build debug tools to empower rapid iteration. We do not hope the game works; we mathematically prove it works.

---

**End of `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md`.**
