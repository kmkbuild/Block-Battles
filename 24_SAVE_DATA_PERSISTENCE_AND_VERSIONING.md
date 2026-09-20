# 24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md
## Block Battles — Save Data, Persistence, and Versioning

**Governing documents:** `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md`, `17_MONETIZATION_REWARD_AND_AD_UX.md`, `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the precise mechanisms for writing and retrieving player data to disk. It adheres strictly to the offline-first, solo-developer constraints (Doc 00) and explicitly avoids backend server databases (BaaS/mBaaS).

---

## 1. Persistence Philosophy

**"Trust the Client, Protect the File."**

For a single-player, offline puzzle game, attempting to build server-authoritative anti-cheat is a massive waste of solo-dev resources. If a player hacks their local Hive database to give themselves 1,000,000 Stardust, they are only ruining their own experience. 

However, we MUST aggressively protect the save file against **accidental corruption** (e.g., the phone battery dying mid-write). Data loss is the #1 cause of 1-star reviews.

---

## 2. What Must Be Saved

### A. Meta-Progression
- **Stardust:** Current balance of the soft currency.
- **Relic Compendium:** List of String IDs representing unlocked Relics.
- **Onboarding State:** `hasCompletedTutorial` boolean.
- **IAP Status:** `hasPurchasedPremiumUnlock` boolean.

### B. Current Run State (Resumability)
- **Run Metadata:** Seed, current floor/battle index, current active Relics, and **RNG State calls consumed**. (The original seed alone does NOT guarantee exact mid-run restoration; the exact number of times the RNG was polled must be saved to advance the RNG back to its exact mid-run state).
- **Battle State:** Current enemy ID, Enemy HP, remaining moves, turn counter, enemy action counter, telegraphed enemy action ID, current combo, objective progress.
- **Board State:** 64-element integer array representing the 8x8 grid contents (both Occupancy and Modifier).
- **Tray State:** The IDs of the currently available shapes in the 3 tray slots.

### C. Settings & Preferences
- All values defined in `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` (Volumes, Haptics, Drag Offset, Language, Reduced Motion).

---

## 3. What Must NOT Be Saved

- **Mid-Animation States:** Because the Pure Dart Domain resolves the entire turn's math instantly and immediately saves the result (`Section 8`), the game never "saves during an animation". If the game crashes while a piece is visually exploding in Flame, it reloads to the state *after* the turn resolved (the animation is simply skipped). This prevents save-scumming.
- **Ghost Preview Data:** Position of a dragged piece is discarded.
- **Particle System Data:** Do not serialize VFX components.

---

## 4. Save Schema and Box Structure

The game utilizes **Hive CE** (`hive_ce`, `hive_ce_flutter`) as its primary local database. The database is partitioned into three distinct boxes:

1. **`progressionBox`**: Stores `PlayerProgressionData` (Stardust, unlocked relics, tutorial status).
2. **`runBox`**: Stores `RunSaveData` (Active run state).
3. **`settingsBox`** (Optional via Hive, but implemented via `SharedPreferences` as detailed below).

```dart
@HiveType(typeId: 1)
class PlayerProgressionData {
  @HiveField(0)
  int schemaVersion = 1;
  
  @HiveField(1)
  int stardust = 0;
  
  @HiveField(2)
  bool hasCompletedTutorial = false;
  
  @HiveField(3)
  bool isPremiumUnlocked = false;
  
  @HiveField(4)
  List<String> unlockedRelicIds = [];
}
```
*(RunSaveData defined previously in Doc 20).*

---

## 5. Storage Strategy

- **Primary Database (Progression & Run):** Hive CE.
- **Why:** Hive CE is extremely fast, fully synchronous after opening, and handles generic collections natively. Hive CE's type adapters (`@HiveType`, `@HiveField`) ensure robust schema enforcement.
- **Preferences / Settings:** `SharedPreferences`.
- **Why:** Simple key-value flags (like `hasAgreedToTos`, `masterVolume`, `hapticsEnabled`, `reducedMotionEnabled`) are best kept in `SharedPreferences`. It avoids unnecessary Hive box overhead for global app properties.
- **Location:** Hive CE automatically manages its storage location via `Hive.initFlutter()`.
- **Cloud Sync:** Relies entirely on the native OS auto-backup mechanisms (Google Play Cloud Save / Apple iCloud Device Backup). No custom backend code required.

---

## 6. Atomic Save (Corruption Prevention)

Hive CE natively handles robust file writing and prevents corruption through its internal append-only storage and compaction strategies. Our responsibility is to wrap reads and writes within a `SaveRepository` abstract class, which the rest of the application interacts with. This repository safely handles initialization and read/write operations.

```dart
abstract class SaveRepository {
  Future<void> init();
  Future<void> saveProgression(PlayerProgressionData data);
  Future<PlayerProgressionData> loadProgression();
  Future<void> saveRun(RunSaveData data);
  Future<RunSaveData?> loadRun();
  Future<void> deleteRun();
}
```

---

## 7. Save Boundaries (Frequency)

The save system must explicitly checkpoint at the following times to prevent save-scumming and guarantee exact mid-run restoration:

1. **At Run Start:** Immediately after generating the Run Seed and initializing the Domain models.
2. **At Battle Start:** Immediately after the enemy is spawned and the first tray is generated.
3. **After Every Successful Placement (Turn End):** The Domain resolves the ENTIRE turn instantly (piece placed, lines cleared, damage calculated, enemy action executed) and transitions back to `active` (or `battle_idle`). The game MUST save immediately at this exact moment, *before* Flame animations finish. This prevents save-scumming line clears. If the app is closed during Presentation animations, reopening it produces the exact logical state that existed *after* that placement resolved.
4. **After Every Battle (Pre-Reward):** When an enemy is defeated, before the Relic reward screen is shown.
5. **After Relic Selection:** Immediately after the player chooses a Relic and it is added to the active set.
6. **At Run End:** Immediately upon `run_defeat` or `run_victory` to serialize stardust payouts and wipe the active run save.
7. **On Settings/Meta-Transaction:** Immediately on slider release (to `SharedPreferences`) or IAP.

---

## 8. Restore Order (Exact Restoration)

When restoring a mid-run save, the system must execute the following sequence to guarantee exact restoration without triggering new piece generation or unwanted events:

1. **Load save:** Read and deserialize via `SaveRepository.loadRun()`.
2. **Validate schema:** Ensure schema version matches or run migrations (Section 10).
3. **Restore Run State:** Set seed, floor, active relics.
4. **Restore RNG State:** Re-seed the Domain RNG and advance its state by `rngCallCount` to reach the exact mid-run state (`04` Section 13).
5. **Load Encounter:** Load the correct Enemy and Objective based on floor.
6. **Restore Board:** Apply the saved `List<int>` (64 elements) to the `BoardEngine`.
7. **Restore Enemy:** Set current HP, action counter, and telegraphed action.
8. **Restore Objective:** Set current progress.
9. **Restore Tray:** Explicitly load the saved `currentTrayShapeIds` without asking RNG to spawn new ones.
10. **Restore Combo/Turn State:** Apply remaining moves, combo count, turn counter.
11. **Enter Gameplay:** Transition `BattleOrchestrator` directly into `battle_idle` (or `active` per `01` Section 4). *Do not emit "Game Started" or "Piece Spawned" events that might inadvertently progress counters.*

---

## 9. Background / Crash Handling

Mobile OS environments aggressively kill background apps to free RAM.
The app should listen to `AppLifecycleState` via `WidgetsBindingObserver`:

```dart
@override
void didChangeAppLifecycleState(AppLifecycleState state) {
  if (state == AppLifecycleState.paused || state == AppLifecycleState.detached) {
    // If a turn is mid-resolution, the Domain should have already saved.
    // We can explicitly flush Hive to disk here.
    Hive.close(); // Forces an immediate sync to disk
  }
}
```

---

## 10. Schema Versioning and Migration

Every Hive class must contain a `schemaVersion` integer field (e.g., `@HiveField(0)`).

- **MVP Launch:** `schemaVersion = 1`.
- If an update radically changes how the Board State array is formatted, the new app will increment this field and utilize a migration strategy upon database initialization.

```dart
Future<void> migrateProgressionBox(Box<PlayerProgressionData> box) async {
  // Read raw data and migrate if necessary
  // e.g. Box data from v1 requires renaming 'gold' to 'stardust'
}
```
*Why:* This ensures players who skip 3 updates and log in a year later do not lose their Relics due to Hive deserialization failures.

---

## 11. Reset Data

As mandated by `16` (Settings), the game must offer a "Delete Save Data" button.
- **Disk:** This calls `Hive.deleteBoxFromDisk()` and clears `SharedPreferences`.
- **Memory:** It must also invalidate the active Riverpod providers in memory (`ref.invalidate()`) and force a soft-reboot of the application state via GoRouter (routing the user back to the `/boot` sequence).

---

## 12. Security (Simple Checksum)

To prevent casual tampering by users altering their save data (which often results in broken types and app crashes):
- **Checksum / Obfuscation:** The Hive CE data for `progressionBox` (stardust/unlocks) can optionally utilize a simple checksum or Hive's built-in AES encryption with a hardcoded local key. 
- *Note:* This is not military-grade encryption; it is simply obfuscation to stop casual tampering that results in syntax errors or trivial save-file editing on rooted Android devices.

---

## 13. Testing with Mock Repositories

The persistence layer must have robust Dart unit tests.
- **MockRepository:** Create a `MockSaveRepository` that implements `SaveRepository` using purely in-memory maps.
- **Test 1:** Generate a dummy V1 save. Run it through the V2 Migrator. Assert that specific variables were transformed correctly.
- **Test 2:** Place a piece, simulate an app pause (force `SaveRepository.saveRun()`), re-initialize the Domain using the mock data. Assert that the board array still contains the piece.

---

## 14. Failure Recovery (The "Nuke" Option)

If the game encounters an unrecoverable exception during mid-run deserialization (e.g., an Enemy ID exists in the save file but was deleted from the game's catalogs):
- Do NOT crash to desktop.
- Catch the exception.
- Silently delete the `RunSaveData` via `SaveRepository.deleteRun()` (Abandon the run).
- Keep the `PlayerProgressionData` (Stardust/Relics intact).
- Load the Main Menu with a generic error dialog: "Your run could not be resumed due to a version update."

---

## 15. Final Persistence Contract

> **No player should ever lose progress to a dead battery.**
>
> We write data often, we write it safely, and we always keep a backup. We accept that players might cheat locally, but we vigorously defend them against accidental data loss. 
> 
> By standardizing on Hive CE, `SaveRepository` abstractions, and strict turn-based save boundaries, we guarantee a flexible, upgradeable, and robust persistence layer.

---

**End of `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md`.**
