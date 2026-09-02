# 24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md
## Block Battles — Save Data, Persistence, and Versioning

**Governing documents:** `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md`, `17_MONETIZATION_REWARD_AND_AD_UX.md`, `20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the precise mechanisms for writing and retrieving player data to disk. It adheres strictly to the offline-first, solo-developer constraints (Doc 00) and explicitly avoids backend server databases (BaaS/mBaaS).

---

## 1. Persistence Philosophy

**"Trust the Client, Protect the File."**

For a single-player, offline puzzle game, attempting to build server-authoritative anti-cheat is a massive waste of solo-dev resources. If a player hacks their local JSON file to give themselves 1,000,000 Stardust, they are only ruining their own experience. 

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
- All values defined in `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` (Volumes, Haptics, Drag Offset, Language).

---

## 3. What Must NOT Be Saved

- **Mid-Animation States:** Because the Domain resolves the entire turn's math in Frame 1 and immediately saves the result (`Section 8`), the game never "saves during an animation". If the game crashes while a piece is visually exploding, it reloads to the state *after* the turn resolved (the animation is simply skipped). This prevents save-scumming.
- **Ghost Preview Data:** Position of a dragged piece is discarded.
- **Particle System Data:** Do not serialize VFX transforms.

---

## 4. Save Schema

The root save file is a single JSON object encompassing the entire player profile.

```csharp
[Serializable]
public class PlayerProfileSave {
    public int saveVersion; 
    public string lastPlayedTimestamp;
    
    public SettingsSaveData settings;
    public MetaSaveData meta;
    public RunSaveData activeRun; // Null if no run is in progress
}

[Serializable]
public class MetaSaveData {
    public int stardust;
    public bool hasCompletedTutorial;
    public bool isPremiumUnlocked;
    public List<string> unlockedRelicIDs;
}
```
*(RunSaveData defined previously in Doc 20).*

---

## 5. Serialization Strategy

- **Format:** JSON.
- **Library:** `Newtonsoft.Json` (Json.NET for Unity).
- **Why:** Unity's built-in `JsonUtility` does not support generic Lists, Dictionaries, or nullability cleanly. `Newtonsoft` handles schema evolution elegantly (e.g., ignoring missing fields, populating new fields with defaults).

---

## 6. Local Storage Strategy

- **Location:** `Application.persistentDataPath + "/player_profile.json"`
- **PlayerPrefs:** Explicitly BANNED for saving progression. `PlayerPrefs` is insecure, prone to being wiped by OS cleanup tools on some Android forks, and has harsh size limits. It may ONLY be used to store a tiny flag like `HasAgreedToTOS=true`.
- **Cloud Sync:** Relies entirely on the native OS auto-backup mechanisms (Google Play Cloud Save / Apple iCloud Device Backup) mapping to `persistentDataPath`. No custom backend code required.

---

## 7. Atomic Save (Corruption Prevention)

To prevent a crash mid-write from wiping the player's history, ALL disk writes must use the **Atomic Save Pattern**:

1. Serialize data to a temporary file: `profile.tmp`
2. Verify the temporary file has a size > 0 bytes.
3. Overwrite the backup file: Copy `profile.json` to `profile.bak`
4. Commit the save: Move `profile.tmp` to `profile.json` (File replacement is generally atomic at the OS level).

---

## 8. Save Frequency

The save system must explicitly checkpoint at the following times to prevent save-scumming and guarantee exact mid-run restoration:
1. **At Run Start:** Immediately after generating the Run Seed and initializing the RunManager.
2. **At Battle Start:** Immediately after the enemy is spawned and the first tray is generated.
3. **After Every Successful Placement (Turn End):** The Domain resolves the ENTIRE turn instantly in Frame 1 (piece placed, lines cleared, damage calculated, enemy action executed) and transitions back to `ACTIVE` (or `BATTLE_IDLE`). The game MUST save immediately at this exact moment, *before* Presentation animations finish. This prevents save-scumming line clears. If the app is closed during Presentation animations, reopening it produces the exact logical state that existed *after* that placement resolved.
4. **After Every Battle (Pre-Reward):** When an enemy is defeated, before the Relic reward screen is shown.
5. **After Relic Selection:** Immediately after the player chooses a Relic and it is added to the active set.
6. **At Run End:** Immediately upon `RUN_DEFEAT` or `RUN_VICTORY` to serialize stardust payouts and wipe the active run save.
7. **On Settings/Meta-Transaction:** Immediately on slider release or IAP.

---

## 9. Restore Order (Exact Restoration)

When restoring a mid-run save, the system must execute the following sequence to guarantee exact restoration without triggering new piece generation or unwanted events:

1. **Load save:** Read and deserialize `profile.json`.
2. **Validate schema:** Ensure save version matches or run migrations (Section 12).
3. **Restore Run State:** Set seed, floor, active relics.
4. **Restore RNG State:** Re-seed `GameRNG` and advance it by `rngStateCalls` to reach the exact mid-run state (`04` Section 13).
5. **Load Encounter:** Instantiate the correct Enemy and Objective based on floor.
6. **Restore Board:** Apply the saved `int[64]` to the `BoardEngine`.
7. **Restore Enemy:** Set current HP, action counter, and telegraphed action.
8. **Restore Objective:** Set current progress.
9. **Restore Tray:** Explicitly load the saved `currentTrayShapeIDs` without asking `GameRNG` to spawn new ones.
10. **Restore Combo/Turn State:** Apply remaining moves, combo count, turn counter.
11. **Enter Gameplay:** Transition `BattleOrchestrator` directly into `BATTLE_IDLE` (or `ACTIVE` per `01` Section 4). *Do not emit "Game Started" or "Piece Spawned" events that might inadvertently progress counters.*

---

## 10. Background / Crash Handling

Mobile OS environments aggressively kill background apps to free RAM.
The `SaveService` must hook into Unity's lifecycle events:

```csharp
private void OnApplicationPause(bool pauseStatus) {
    if (pauseStatus) {
        ForceEmergencySave();
    }
}

private void OnApplicationQuit() {
    ForceEmergencySave();
}
```
*Note:* The emergency save must be completely synchronous. Do not use async/await here, as the OS will kill the thread before it yields.

---

## 11. Versioning

Every save file must contain a `saveVersion` integer at the root.
- **MVP Launch:** `saveVersion = 1`.
- If an update radically changes how the Board State array is formatted, the new app will expect `saveVersion = 2`.

---

## 12. Migration System

If the client detects an older `saveVersion` upon loading, it routes the data through an `IMigrationStrategy`.

```csharp
public class MigrationV1toV2 : IMigrationStrategy {
    public PlayerProfileSave Migrate(JObject oldJson) {
        // e.g., In V1, stardust was called "gold". Extract and rename.
        // Return updated PlayerProfileSave.
    }
}
```
*Why:* This ensures players who skip 3 updates and log in a year later do not lose their Relics due to JSON deserialization failures.

---

## 13. Reset Data

As mandated by `16` (Settings), the game must offer a "Delete Save Data" button.
- **Disk:** This deletes `profile.json` and `profile.bak`.
- **Memory:** It must also nullify the active `PlayerProfileSave` in memory and force a soft-reboot of the application state (routing the user back to the `00_Boot` sequence).

---

## 14. Backup / Restore

- **Auto-Restore:** If `BootstrapService` tries to load `profile.json` and `Newtonsoft` throws a parsing exception (indicating file corruption), the system automatically attempts to load `profile.bak`.
- If `profile.bak` is also corrupted, a fresh default profile is generated.

---

## 14. Anti-Corruption Rules

To prevent accidental modification by users poking around in their Android data folders (which often results in broken JSON syntax and app crashes):
- **Base64 Encoding:** The JSON string is encoded to Base64 before writing to disk. 
- *Note:* This is not encryption; it is simply obfuscation to stop casual tampering that results in syntax errors.

---

## 15. Testing

The persistence layer must have robust PlayMode and EditMode tests.
- **Test 1:** Generate a dummy V1 save JSON. Run it through the V2 Migrator. Assert that specific variables were transformed correctly.
- **Test 2:** Force a `Time.timeScale = 0` mid-write to simulate a crash. Assert that `profile.bak` remains intact.
- **Test 3:** Place a piece, trigger a forced application quit, restart the test runner. Assert that the board array still contains the piece.

---

## 16. Failure Recovery (The "Nuke" Option)

If the game encounters an unrecoverable exception during mid-run deserialization (e.g., an Enemy ID exists in the save file but was deleted from the game's ScriptableObjects):
- Do NOT crash to desktop.
- Catch the exception.
- Silently delete the `RunSaveData` (Abandon the run).
- Keep the `MetaSaveData` (Stardust/Relics intact).
- Load the Main Menu with a generic error: "Your run could not be resumed due to a version update."

---

## 17. Final Persistence Contract

> **No player should ever lose progress to a dead battery.**
>
> We write data often, we write it safely, and we always keep a backup. We accept that players might cheat locally, but we vigorously defend them against accidental data loss. 
> 
> By standardizing on Newtonsoft JSON and Base64 obfuscation, we guarantee a flexible, upgradeable, and robust persistence layer that an AI agent can easily extend with new variables.

---

**End of `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md`.**
