# 24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md
## Block Battles — Save Data, Persistence, and Versioning

**Governing documents:** `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md`, `17_MONETIZATION_REWARD_AND_AD_UX.md`, `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the precise mechanisms for writing and retrieving player data to disk. It adheres strictly to the offline-first, solo-developer constraints (Doc 00) and explicitly avoids backend server databases (BaaS/mBaaS) for both **Mode 1 (Classic Endless)** and **Mode 2 (Goal-Driven Stage Challenge)**.

---

## 1. Persistence Philosophy

**"Trust the Client, Protect the File."**

For a single-player, offline puzzle game, attempting to build server-authoritative anti-cheat is an unnecessary burden for solo development. If a player tampers with their local database, they only alter their personal scores.

However, we MUST aggressively protect the save file against **accidental corruption** (e.g., the phone battery dying mid-write or the OS terminating the app in the background). Data loss is the primary cause of negative mobile game reviews.

---

## 2. What Must Be Saved

### A. Mode 1: Classic Endless Meta-Progression
- **`classicHighScore`:** All-time best score achieved in Classic Endless mode.
- **`totalClassicGamesPlayed`:** Lifetime count of Classic games started.
- **`bestStreakCount`:** Highest consecutive line clear streak achieved.

### B. Mode 2: Goal-Driven Stage Progression
- **`highestUnlockedStage`:** Integer identifying the highest unlocked stage (e.g., Stage 1..100).
- **`stageStarsMap`:** Map of `{stageNumber: starsEarned}` storing 1 to 3 stars earned per stage.
- **`stageHighScoresMap`:** Map of `{stageNumber: bestScore}` storing the player's best score per stage.

### C. Active Mid-Game Session State (Resumability)
- **`activeMode`:** Enum indicating `classicEndless` or `goalStage`.
- **`boardGrid`:** 64-element integer array representing the 8×8 grid state (occupancy and cell color/modifier).
- **`trayShapeIds`:** List of 3 nullable shape IDs currently present in the tray slots.
- **`currentScore`:** Score accumulated in the active session.
- **`currentStreak`:** Active consecutive streak multiplier state.
- **`remainingMoves`:** (Mode 2 only) Number of remaining placement turns allowed.
- **`goalProgressMap`:** (Mode 2 only) Progress counters toward active stage objectives.
- **`rngSeed` & `rngCallsConsumed`:** Deterministic RNG state counters for exact piece roll restoration.

### D. Settings & Preferences
- All values defined in `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` (Master Volume, SFX, Music, Haptics, Drag Offset, Reduced Motion, Language) persisted via `SharedPreferences`.

---

## 3. What Must NOT Be Saved

- **Mid-Animation States:** The pure Dart domain resolves the full placement and clear logic instantly before saving. Visual particle effects and falling block animations are never saved.
- **Ghost Preview Coordinates:** Transient drag pointer coordinates are discarded.
- **Particle System Objects:** Flame VFX and render objects are strictly ephemeral.

---

## 4. Save Schema and Box Structure

The game utilizes **Hive CE** (`hive_ce`, `hive_ce_flutter`) as its primary local database, partitioned into two distinct boxes:

1. **`progressionBox`**: Stores persistent player records (`PlayerProgressionData`).
2. **`sessionBox`**: Stores active mid-game session data (`ActiveSessionData`) for seamless resumption.

```dart
@HiveType(typeId: 1)
class PlayerProgressionData {
  @HiveField(0)
  int schemaVersion = 1;

  @HiveField(1)
  int classicHighScore = 0;

  @HiveField(2)
  int totalClassicGamesPlayed = 0;

  @HiveField(3)
  int highestUnlockedStage = 1;

  @HiveField(4)
  Map<int, int> stageStars = {}; // stageNumber -> stars (1..3)

  @HiveField(5)
  Map<int, int> stageHighScores = {}; // stageNumber -> score

  @HiveField(6)
  bool hasCompletedOnboarding = false;
}

@HiveType(typeId: 2)
class ActiveSessionData {
  @HiveField(0)
  int schemaVersion = 1;

  @HiveField(1)
  int gameModeIndex = 0; // 0 = classicEndless, 1 = goalStage

  @HiveField(2)
  int activeStageNumber = 1;

  @HiveField(3)
  List<int> gridCells = []; // 64 integers

  @HiveField(4)
  List<String?> trayShapeIds = []; // 3 shape IDs

  @HiveField(5)
  int currentScore = 0;

  @HiveField(6)
  int streakCount = 0;

  @HiveField(7)
  int remainingMoves = 0;

  @HiveField(8)
  Map<int, int> goalCurrentValues = {};

  @HiveField(9)
  String rngSeed = '';

  @HiveField(10)
  int rngCallsConsumed = 0;
}
```

---

## 5. Storage Strategy

- **Primary Database (Progression & Session):** Hive CE.
  - **Why:** Hive CE is extremely fast, fully synchronous after opening, and handles strongly typed schemas natively via `@HiveType` adapters.
- **Preferences / Settings:** `SharedPreferences`.
  - **Why:** Simple key-value primitives (`masterVolume`, `hapticsEnabled`, `reducedMotionEnabled`) avoid Hive box overhead for standard OS preferences.
- **Storage Location:** Handled automatically by `Hive.initFlutter()`.
- **Cloud Backup:** Relies on native OS auto-backup mechanisms (Google Play Cloud Backup / Apple iCloud Backup).

---

## 6. Atomic Save Architecture (`SaveRepository`)

Access to disk persistence is abstracted through a clean repository interface:

```dart
abstract class SaveRepository {
  Future<void> init();
  
  // Progression
  Future<void> saveProgression(PlayerProgressionData data);
  Future<PlayerProgressionData> loadProgression();
  
  // Active Session
  Future<void> saveActiveSession(ActiveSessionData data);
  Future<ActiveSessionData?> loadActiveSession();
  Future<void> clearActiveSession();
}
```

---

## 7. Save Boundaries (Frequency)

The save system checkpoints at specific boundaries to guarantee zero progress loss:

1. **On Session Initialization:** Immediately upon starting a new Classic game or Stage.
2. **After Every Successful Placement:** The domain resolves piece placement, line clears, and scoring instantly. The game saves the `ActiveSessionData` immediately at this moment, *before* Flame animations finish. If the app is force-closed during particle animations, reopening loads the exact logical state *after* that placement.
3. **On Session Completion:** When Classic mode triggers game-over or a Stage is cleared/failed, update `PlayerProgressionData` (high scores, stage stars) and delete `ActiveSessionData`.
4. **On Settings Modification:** Immediately write preference changes to `SharedPreferences`.

---

## 8. Exact Restoration Sequence

When resuming an interrupted session, the system executes this sequence:

1. **Load Data:** Deserialize `ActiveSessionData` via `SaveRepository.loadActiveSession()`.
2. **Validate Schema:** Ensure `schemaVersion` matches or execute migration.
3. **Restore Board Grid:** Apply the 64-element cell array to the `BoardEngine`.
4. **Restore Tray:** Reconstruct the 3 tray slots directly from saved `trayShapeIds` without re-rolling RNG.
5. **Restore Score & Multipliers:** Set current score, streak count, and remaining moves.
6. **Restore RNG State:** Seed `GameRNG` and advance its state counter by `rngCallsConsumed`.
7. **Enter Active Play:** Transition UI directly to active gameplay without firing initial piece spawn animations.

---

## 9. Background / Lifecycle Handling

The mobile OS may terminate background apps at any time. The game registers an `AppLifecycleListener` / `WidgetsBindingObserver`:

```dart
@override
void didChangeAppLifecycleState(AppLifecycleState state) {
  if (state == AppLifecycleState.paused || state == AppLifecycleState.detached) {
    // Flush pending Hive writes to disk immediately
    Hive.close();
  }
}
```

---

## 10. Schema Versioning and Migration

Every Hive model carries a `schemaVersion` integer field (starting at `1`). If a future update introduces new fields or alters data formatting:
1. Increment `schemaVersion`.
2. Run a migration routine during repository initialization to map old fields to new schemas without data loss.

---

## 11. Resetting Data

The game provides a "Reset All Progress" button in Settings:
- Wipes `progressionBox` and `sessionBox` using `Hive.deleteBoxFromDisk()`.
- Resets `SharedPreferences`.
- Reinitializes clean default state and routes the player back to the main menu.

---

## 12. Security and Data Integrity

- Hive CE binary storage prevents trivial plain-text file editing.
- Optional simple checksum validation protects against corrupted manual edits on rooted devices.

---

## 13. Testing with Mock Repositories

Unit tests verify persistence logic without touching physical disk I/O:
```dart
class MockSaveRepository implements SaveRepository {
  PlayerProgressionData? progressionData;
  ActiveSessionData? sessionData;

  @override
  Future<void> saveProgression(PlayerProgressionData data) async => progressionData = data;
  
  @override
  Future<PlayerProgressionData> loadProgression() async =>
      progressionData ?? PlayerProgressionData();

  @override
  Future<void> saveActiveSession(ActiveSessionData data) async => sessionData = data;

  @override
  Future<ActiveSessionData?> loadActiveSession() async => sessionData;

  @override
  Future<void> clearActiveSession() async => sessionData = null;
  
  @override
  Future<void> init() async {}
}
```

---

## 14. Failure Recovery

If deserializing an active session throws an unexpected error (e.g., corrupted file after an abnormal device shutdown):
1. Catch the exception safely.
2. Delete the corrupted `ActiveSessionData` via `clearActiveSession()`.
3. Preserve `PlayerProgressionData` (all-time high scores and stage stars remain safe).
4. Launch the main menu smoothly with an informative toast message.

---

## 15. Final Persistence Contract

> **No player should ever lose progress to a dead battery.**
>
> We write data often, we write it safely, and we maintain clear separation between persistent progress and transient session state. By standardizing on Hive CE, `SaveRepository` abstractions, and turn-based save checkpoints, we guarantee a robust, zero-loss persistence system for both Classic Endless and Goal-Driven Stage modes.

---

**End of `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md`.**
