# 20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md
## Block Battles — Data Model and Content Schema

**Governing documents:** `18_TECHNICAL_ARCHITECTURE.md`, `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md`, all Layer A design specs.
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document defines the exact C# data structures that fulfill the architecture outlined in Doc 18. It enforces the separation between static design data (ScriptableObjects) and mutable runtime data.

---

## 1. Data Architecture Philosophy

**"Code is the Engine; Data is the Fuel."**
The core gameplay systems must be entirely agnostic to the content they are processing. The `CombatEngine` does not know what an "Iron Mallet" is; it only knows how to process a `RelicData` object that provides a flat damage modifier on a specific trigger. This ensures AI coding agents can add thousands of new items purely by modifying data assets, without ever rewriting core logic.

---

## 2. Runtime Data vs. Authoring Data

There is a strict boundary between what the designer authors and what the game manipulates:

- **Authoring Data (ScriptableObjects):** Static, read-only at runtime. Created in the Unity Editor. Example: `EnemyData` (contains `MaxHP = 100`).
- **Runtime Data (POCOs - Plain Old C# Objects):** Mutable state instantiated at runtime. Example: `EnemyState` (contains `CurrentHP = 75`, initialized from `EnemyData.MaxHP`).

---

## 3. ScriptableObject Rules

1. **Read-Only at Runtime:** Never write code that assigns a value to a ScriptableObject field at runtime. It corrupts the asset in the Editor.
2. **Flat Serialization:** Avoid deeply nested polymorphic serialization (`[SerializeReference]`) where possible. It is fragile for version control and difficult for AI agents to manipulate via YAML. Use Enums + Parameter Structs for composition.
3. **Self-Validation:** Every ScriptableObject must implement `OnValidate()` to catch designer errors immediately (e.g., ensuring a Shape matrix has exactly 25 elements).

---

## 4. Shape Schema (`ShapeData.cs`)

Defines the geometry of a draggable piece.

```csharp
public class ShapeData : ScriptableObject {
    public string shapeID; // e.g., "shape_T_3x3"
    
    [Header("Geometry (5x5 Grid)")]
    // Flat array of 25 bools. True = block, False = empty.
    // Indexing: row * 5 + col. Bottom-Left is 0,0.
    public bool[] blockMatrix = new bool[25]; 
    
    [Header("Metadata")]
    public ShapeComplexityTier tier; // Basic, Advanced, Complex
    public Color overrideColor; // If null/transparent, use standard random spawn color
    
    // Bounds caching (calculated in OnValidate)
    public int width;
    public int height;
}
```

---

## 5. Enemy Schema (`EnemyData.cs`)

Defines an enemy encounter and its behavior pattern.

```csharp
public class EnemyData : ScriptableObject {
    public string enemyID;
    public string displayNameKey; // Localization key
    
    [Header("Stats")]
    public int baseMaxHP;
    public int defenseRating; 
    
    [Header("Behavior Sequence")]
    // Executes in order, looping when reaching the end.
    public EnemyAction[] turnPattern; 
    
    [Header("Presentation")]
    public Sprite portrait;
    public Color themeColor;
}

[System.Serializable]
public struct EnemyAction {
    public EnemyActionType type; // e.g., Attack, Defend, FreezeBoard
    public int powerValue;       // e.g., Damage amount, or number of cells to freeze
    public int telegraphTurns;   // Number of turns before execution
}
```

---

## 6. Relic Schema (`RelicData.cs`)

Defines a passive modifier. Uses the Trigger/Effect architecture required by Doc 05.

```csharp
public class RelicData : ScriptableObject {
    public string relicID;
    public string nameKey;
    public string descriptionKey;
    
    [Header("Mechanics")]
    public RelicTrigger trigger; // Enum: OnLineClear, OnBattleStart, OnDamageDealt, etc.
    public RelicEffectType effectType; // Enum: AddFlatDamage, AddMultiplier, RestoreMoves
    public float effectValue;
    
    [Header("Conditions (Optional)")]
    public ConditionType condition; // Enum: None, TargetBelowHPThreshold, ComboGreaterThan
    public float conditionValue;
    
    [Header("Metadata")]
    public RelicRarity rarity;
    public Sprite icon;
}
```

---

## 7. Objective Schema (`ObjectiveData.cs`)

Defines win conditions for a Battle.

```csharp
public class ObjectiveData : ScriptableObject {
    public string objectiveID;
    public ObjectiveType type; // Enum: DefeatEnemy, ClearLines, SurviveMoves
    public int targetValue;    // e.g., 5 (lines), 0 (enemy HP), 20 (moves to survive)
    
    public string GetProgressText(int current) {
        // Formats string based on type, e.g., "{current} / {targetValue} Lines"
    }
}
```

---

## 8. Battle Schema (`EncounterData.cs`)

Defines a specific floor or node in a Run.

```csharp
public class EncounterData : ScriptableObject {
    public string encounterID;
    public EnemyData enemy;
    public ObjectiveData primaryObjective;
    public ObjectiveData bonusObjective;
    
    [Header("Constraints")]
    public int startingMoveLimit; // 0 if infinite
    
    [Header("Spawn Pool")]
    public ShapeData[] allowedShapes; // Overrides default pool if not empty
    
    public int rewardStardustBase;
}
```

---

## 9. Persistent Save Data Schema (`RunSaveData.cs`)

POCO class serialized to JSON. Does not use Unity `ScriptableObjects`.

```csharp
[System.Serializable]
public class RunSaveData {
    public string runSeed;
    public int currentFloor;
    public int currentStardust;
    public List<string> unlockedRelicIDs;
    public List<string> activeRelicIDs;
    
    // Mid-battle state (if saving mid-run is supported)
    public int[] currentBoardState; // 64 integers representing cell contents
    public int currentEnemyHP;
    public int remainingMoves;
}
```

---

## 10. IDs and Versioning

- **String IDs:** All content relies on unique string IDs (`shape_square_2x2`), NOT object references, when interacting with the Save System.
- **Why:** Unity Object references break when crossing assembly boundaries or when serialized to JSON.
- **Immutability:** Once a Relic ID is shipped, it cannot be renamed. Renaming an ID will corrupt player save files referencing that ID.

---

## 11. Validation Rules

All ScriptableObjects must utilize Unity's `OnValidate()` callback to sanitize data.

```csharp
// Inside ShapeData.cs
private void OnValidate() {
    if (blockMatrix.Length != 25) {
        Debug.LogWarning($"Shape {name} matrix must be exactly 25 elements.");
        Array.Resize(ref blockMatrix, 25);
    }
    // Auto-calculate width/height based on truthy matrix values
}
```

---

## 12. Serialization Rules

- Unity's built-in `JsonUtility` is fast but limited (does not support Dictionaries or deeply nested arrays easily).
- **Rule:** Use `Newtonsoft.Json` (Unity package) for `RunSaveData` to allow robust handling of Lists, Dictionaries, and polymorphic types if they arise.

---

## 13. Null/Missing Data Behavior

- **Graceful Fallback:** If `RunSaveData` references a `relicID` that no longer exists (e.g., removed in a patch), the loading system logs a warning, skips the missing relic, and continues loading. The save file must NOT corrupt.
- **Missing Sprites:** If a `Sprite` reference is null in a `RelicData`, the UI automatically renders a generic "MissingAsset" fallback icon rather than throwing a `NullReferenceException`.

---

## 14. Designer Authoring Workflow

To empower solo-dev and AI content creation:
1. **AI Output:** An AI agent can generate a JSON list of new relics.
2. **Importer Tool:** A simple Editor script reads the JSON and automatically instantiates the `RelicData` ScriptableObjects in the `Assets/Data/Relics/` folder.
3. **No Code Modification:** The AI does not need to edit `CombatEngine.cs` to add the "Executioner" relic, because the Trigger/Effect enums already support its logic.

---

## 15. Content Validation (Automated Testing)

An Editor test suite must verify the integrity of the Data folder before any build:
- Checks for duplicate `relicID` or `enemyID` values.
- Checks for null Sprite/Audio references in every ScriptableObject.
- Ensures all `ShapeData` matrices have at least 1 true value (no invisible pieces).

---

## 16. Example Data Representation

*How the AI should understand an Encounter definition:*

```json
{
  "encounterID": "enc_floor_05_slime",
  "enemyID": "enm_acid_slime",
  "primaryObjective": {
    "type": "ClearLines",
    "target": 10
  },
  "constraints": {
    "moveLimit": 30
  }
}
```

---

## 17. Anti-Pattern Rules

1. **The "Live State" SO:** Never put `public int currentHP` in `EnemyData`. 
2. **Hardcoded IDs in Core:** Never write `if (relic.ID == "IronMallet") { damage += 10; }` inside the `CombatEngine`. The engine must resolve the math generically based on the `effectType` enum.
3. **Circular Asset References:** Do not have `EnemyData` reference an `EncounterData` that references the same `EnemyData`.

---

## 18. Extensibility

If a future update requires a mechanic not supported by the base `Trigger` and `Effect` enums (e.g., "Blocks now bounce"), you do NOT hack it into the existing enums. 
You create a new `SpecialModifier` component or interface and extend the core loop to broadcast a new event hook.

---

## 19. Acceptance Criteria

The Data Architecture is considered complete when:
- [ ] A developer (or AI) can create a completely new Enemy, Objective, and Relic without writing a single line of C# code.
- [ ] The Unity Editor Console throws zero errors when saving/loading `RunSaveData` containing the new content.
- [ ] Automated tests verify that no two data assets share the same string ID.

---

## 20. Final Data Contract

> **Data is the absolute truth; Systems are just interpreters.**
>
> In Block Battles, we lock down our core C# systems early and scale the game purely by injecting wider varieties of data. By maintaining strict discipline over our ScriptableObject schemas, we ensure that adding content is safe, deterministic, and highly compatible with AI-assisted mass generation.

---

**End of `20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md`.**
