# AI_START_HERE.md
## Block Battles — AI Orientation Document

**Read this file first. Always.**
This is the single entry point for any AI coding agent, language model, or automated tool interacting with this repository. It provides project orientation and authoritative document routing. Do not skip it.

---

## 1. Project Identity

**Name:** Block Battles

**One-sentence definition:** Block Battles is a mobile puzzle-combat game where players place Tetromino-style blocks onto an 8x8 grid to clear lines and deal damage to handcrafted enemies.

**One-paragraph definition:**
Block Battles fuses the satisfying infinite loop of a block-placement puzzle (similar in feel to Woodoku or 1010!) with a turn-based roguelite combat system. The player places pieces from a 3-slot tray onto an 8x8 board. When a full row or column is cleared, damage is dealt to the current enemy. As the player wins battles, they collect passive upgrades called Relics that fundamentally change how their damage, combos, and board interactions work. Each encounter has a specific Objective (defeat the enemy, clear N lines, survive N moves). The game is designed for Android-first, offline, solo-developer production with a deterministic engine fully compatible with AI-assisted coding.

---

## 2. Core Game Loop (Visual)

```
START RUN (seeded)
     │
     ▼
┌─────────────────────────────────────┐
│ BATTLE BEGINS                       │
│  Enemy appears with HP + Objective  │
└──────────────────┬──────────────────┘
                   │
                   ▼
         ┌─────────────────┐
         │  PLAYER TURN    │◄──────────────┐
         │  Pick a piece   │               │
         │  Drag to board  │               │
         │  Drop & Snap    │               │
         └────────┬────────┘               │
                  │                        │
                  ▼                        │
         Rows/Cols full?                   │
          YES │     NO │                   │
              │        └──────────────────►┤
              ▼                            │
      LINE CLEAR + DAMAGE                  │
      Relics evaluate & modify             │
      Enemy takes damage                   │
              │                            │
              ▼                            │
     Objective complete?     No more valid placements?
          YES │     NO │          YES │
              │        └─────────────┘│
              │                       ▼
              │                   DEFEAT ──► MENU
              ▼
        VICTORY
        Select 1 Relic
              │
              ▼
        Next Floor or
        Run Complete
```

---

## 3. What Makes the Game Different

| Feature | What It Does |
|---|---|
| **Block Puzzle Foundation** | Players feel the deeply satisfying loop of placing pieces and clearing lines — no decay, no timer, pure zen decision-making. |
| **Combat Integration** | Line clears deal real damage. More lines cleared simultaneously = higher L-Value multiplier = more damage. Combat is a natural consequence of good puzzle play. |
| **Relic Builds** | After each victory, the player selects 1 of 3 random Relics. These are passive modifiers (e.g., "+20% damage on L3+ clear", "Restore 1 move when filling a corner"). Builds snowball and create unique playthroughs. |
| **Handcrafted Objectives** | Each battle has a specific win condition (Defeat enemy, Clear N lines, Survive N moves). This prevents every battle feeling identical. |
| **Roguelite Progression** | Runs begin fresh but Stardust (soft currency) carries over between runs to unlock Relic compendium entries. No energy gates, no pay-to-win. |

---

## 4. Core Player Experience

The game should feel: **calm, focused, satisfying — with moments of explosive reward.**

The player should feel clever, not lucky. The board is a puzzle. The enemy is a constraint. The Relics are a build. Placing a piece should feel physically *right* — a tactile snap with gentle camera feedback. A 4-line clear should feel like a celebration. Losing should feel meaningful, not unfair.

---

## 5. MVP Scope

### Included
- 8x8 board, 3-piece tray, no rotation mechanic.
- 15+ unique shapes.
- Line clearing: rows + columns.
- L-Value damage multiplier (L1 to L4+ capped).
- 5 unique enemies with differing HP and attack patterns.
- 10 unique Relics.
- 3 Objective types: Defeat, Clear Lines, Survive Moves.
- Run structure: multiple floors, Relic selection between battles.
- Core VFX, SFX, and BGM.
- Save/Load (offline JSON, Atomic writes).
- Settings (Volume, Haptics, Reduced Motion).
- Monetization: Rewarded Ad (Revive) + Premium Unlock (Remove Ads).

### Excluded from MVP
- Cloud Save or server backend.
- Leaderboards or social features.
- Energy/stamina system.
- Battle Pass or complex economy.
- Multiple currencies.
- Character skins or cosmetics.

### Deferred (Post-MVP)
- Daily Challenges.
- Boss encounters.
- Additional Relic tiers.
- Seasonal events.

---

## 6. Core Rules (Non-Negotiable)

| Rule | Value |
|---|---|
| Board Size | 8×8 grid |
| Tray Slots | 3 (refills when all 3 placed) |
| Piece Rotation | NONE. Pieces have a fixed orientation. |
| Line Clear | Full row OR full column clears. Both can clear simultaneously. |
| L-Value | Number of lines cleared in a single piece placement (1-4+). |
| Damage Formula | `BaseDamage × L_Multiplier(L)` — see `05` for table. |
| Game Over | When no piece in the tray can legally fit anywhere on the board. |
| Relics | Passive. Selected between battles. Cannot be removed mid-run. |
| Objectives | One primary per battle. Completion = Battle Victory. |
| Stardust | Soft currency earned per run. Persists across runs. |

All numeric values (BaseDamage, multiplier tables, Stardust rates) are authored exclusively in design documents. **Never hardcode them into logic.**

---

## 7. Architecture Summary

### Data-Driven Design
Enemies, Relics, Shapes, and Objectives are Unity ScriptableObjects. Adding content requires authoring data assets, not modifying C# logic. See `20`.

### Deterministic Gameplay
A seeded `GameRNG` class controls all piece spawns, enemy selections, and relic drops. The same seed + input sequence always produces the same game state. See `04`, `21`.

### Event-Driven Communication
Domain systems (Board, Combat, Enemy) communicate via a type-safe `EventBus<T>`. Presentation systems (UI, Audio, VFX) listen to events and react. The Domain NEVER calls into Presentation. See `22`.

### Presentation Separation (Clean Architecture)
- **Domain Layer** (Pure C#): Board math, damage formulas, RNG.
- **Application Layer** (Orchestration): Turn state machine, Run management.
- **Presentation Layer** (Unity): MonoBehaviours, Canvases, AudioSources.
- Dependencies point **inward only**. Domain knows nothing about Unity. See `18`.

### Persistence
JSON files written to `Application.persistentDataPath` using Atomic Save (tmp → bak → json). Player data never corrupts on crash. See `24`.

---

## 8. Major Runtime Systems

| System | Responsibility |
|---|---|
| `BoardModel` | Owns the 8x8 `int[64]` array. Validates and commits placements. |
| `TrayEngine` | Manages the 3 active pieces. Requests new pieces from `GameRNG`. |
| `LineScanner` | Detects and clears full rows/columns after each placement. |
| `CombatEngine` | Receives L-Value, queries `RelicEngine`, outputs `DamageDealtEvent`. |
| `EnemyEngine` | Tracks enemy HP. Executes enemy turn patterns. |
| `RelicEngine` | Evaluates relic triggers on every domain event. Returns modifiers. |
| `ObjectiveEngine` | Tracks battle win condition progress. |
| `RunManager` | Manages floor progression, stardust, and active relic list across battles. |
| `BattleOrchestrator` | Turn state machine. Blocks input during resolution. |
| `SaveService` | Reads/writes `PlayerProfileSave` JSON. Handles corruption recovery. |
| `AudioService` | Object-pooled AudioSources. Controls mix groups. |
| `VFXManager` | Object-pooled ParticleSystems. Fires effects on EventBus subscriptions. |

---

## 9. Canonical Terminology

| Term | Definition |
|---|---|
| **L-Value** | The number of lines cleared by a single piece placement. L1=1 line, L4=4 lines. |
| **Tray** | The 3-slot panel holding the current available pieces. |
| **Relic** | A passive permanent modifier active for the duration of a run. |
| **Encounter** | A single battle (one enemy, one objective, one board state). |
| **Floor** | An Encounter within a Run. Each floor is harder. |
| **Run** | A full playthrough from Floor 1 to Run Over. |
| **Stardust** | The soft meta-currency earned from runs. Persists. |
| **Board Lock** | The Game Over condition — no piece can be legally placed. |
| **Ghost Preview** | The transparent indicator showing where a dragged piece will land. |
| **Combo** | Consecutive multi-line clears within a turn sequence (see `05`). |
| **Telegraph** | An enemy's declared next action shown N turns in advance. |
| **ShapeData** | A ScriptableObject defining a piece's 5x5 boolean matrix. |
| **EnemyData** | A ScriptableObject defining enemy stats and behavior patterns. |
| **RelicData** | A ScriptableObject defining relic trigger, condition, and effect. |

---

## 10. Source-of-Truth Rules

| Decision | Authority |
|---|---|
| Any gameplay rule | `01_GAMEPLAY_SPECIFICATION.md` |
| Any shape geometry | `02_SHAPE_LIBRARY.md` |
| Any combat formula | `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` |
| Any color value | `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` |
| Any animation timing | `12_ANIMATION_MOTION_AND_GAME_FEEL.md` |
| Any copy/text string | `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md` |
| Any architecture pattern | `18_TECHNICAL_ARCHITECTURE.md` |
| Any folder/file location | `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md` |
| Any data schema | `20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md` |
| Any event definition | `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md` |
| Any save field | `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md` |
| Any naming convention | `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` |
| Any AI behavior rule | `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md` |

**Conflict Resolution:** If existing code disagrees with a document, the document is correct. Fix the code.

---

## 11. Forbidden Behaviors

An AI working on this project MUST NEVER:
1. Write `if (relicName == "IronMallet")` in core logic. Use enum + data.
2. Put game logic in a UI script (`Canvas`, `Button`, `Text`).
3. Use `UnityEngine.Physics2D` or `Collider2D` for board placement.
4. Call `Instantiate()` or `Destroy()` during the combat turn loop.
5. Use `LINQ`, `new List<>()`, or string concatenation during gameplay.
6. Create a `GameManager.cs` that owns more than one responsibility.
7. Delete a `.meta` file and re-create it (destroys Prefab references).
8. Invent a mechanic, balance value, or system not specified in the docs.
9. Wrap a bug in `try { } catch { }` without logging and fixing the root cause.
10. Push to `main` without passing the NUnit test suite.

---

## 12. Development Philosophy

- **Read the spec first.** The Markdown documentation is the source of truth.
- **Make surgical changes.** Fix only what was asked. No unrelated refactors.
- **Data over code.** New enemies and relics are data. New mechanics are not added casually.
- **Test after every Domain change.** If you touched the Board, Combat, or Relics — run the NUnit suite.
- **Ask, don't guess.** If a value is missing from a document, ask the developer. Do not invent defaults.
- **No speculative complexity.** Do not build `AbstractManagerFactoryBase<T>` for a solo mobile game.

---

## 13. Current MVP Content Target

| Category | MVP Target |
|---|---|
| Shapes | 15+ unique shapes |
| Enemies | 5 unique enemies |
| Relics | 10 unique relics |
| Objective Types | 3 (Defeat, ClearLines, SurviveMoves) |
| Encounters / Floors | 5-7 per run |
| Scenes | 3 (Boot, MainMenu, Battle) |
| Audio SFX | ~20 unique clips |
| BGM Tracks | 2-3 looping tracks |

---

## 14. Technical Stack

| Area | Decision |
|---|---|
| Engine | Unity 2022.3 LTS (or newer) |
| Scripting Backend | IL2CPP |
| Language | C# (.NET Standard 2.1) |
| Target Platform | Android-first (Google Play) |
| Min SDK | Android API 26 (Android 8.0) |
| Architecture | ARM64 |
| JSON Serialization | Newtonsoft.Json (Json.NET for Unity) |
| UI Animation | DOTween |
| UI Text | TextMeshPro exclusively |
| Rendering | 2D / URP |
| Physics | NONE (forbidden for gameplay logic) |

---

## 15. Build Order Summary

21 phases in strict dependency order:
`0` Setup → `1` Board Model → `2` Shapes → `3` Placement/Input → `4` Tray → `5` Line Clear → `6` Game Over → `7` Combat → `8` Enemy → `9` Objectives → `10` Relics → `11` Run Structure → `12` Core UI → `13` Visuals → `14` Animation → `15` VFX → `16` Audio → `17` Save → `18` Settings → `19` Monetization → `20` QA → `21` Release.

Do NOT jump ahead. The board must exist before combat. Combat must work before Relics. Relics must work before runs. Full detail in `30`.

---

## 16. Where to Look

| Question | Read |
|---|---|
| What is the overall game vision? | `00_MASTER_GAME_VISION.md` |
| Gameplay rules, piece behavior? | `01_GAMEPLAY_SPECIFICATION.md` |
| Shape geometry and matrices? | `02_SHAPE_LIBRARY.md` |
| Board mechanics and edge cases? | `03_BOARD_ENGINE_AND_RULES.md` |
| RNG, spawning, difficulty? | `04_SPAWN_RNG_AND_DIFFICULTY.md` |
| Damage formulas, combos, scoring? | `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` |
| Drag-and-drop input behavior? | `06_INPUT_DRAG_AND_DROP_UX.md` |
| Objectives and level design? | `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md` |
| UI screens and flows? | `08_UI_UX_MASTER_SPECIFICATION.md` |
| Visual design system? | `09_VISUAL_DESIGN_SYSTEM.md` |
| Color palette and theming? | `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` |
| Block art and material spec? | `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md` |
| Animation timings and feel? | `12_ANIMATION_MOTION_AND_GAME_FEEL.md` |
| Particle and VFX definitions? | `13_VFX_PARTICLE_AND_IMPACT_SPEC.md` |
| Sound effects and music? | `14_AUDIO_SFX_MUSIC_AND_MIXING.md` |
| Player-facing copy and text? | `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md` |
| Settings and accessibility? | `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` |
| Monetization and ads? | `17_MONETIZATION_REWARD_AND_AD_UX.md` |
| Overall engineering architecture? | `18_TECHNICAL_ARCHITECTURE.md` |
| Folder structure, scenes, prefabs? | `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md` |
| ScriptableObject schemas? | `20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md` |
| System managers and ownership? | `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md` |
| EventBus, state machine, events? | `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md` |
| Unity UI implementation? | `23_UI_IMPLEMENTATION_ARCHITECTURE.md` |
| Save data and persistence? | `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md` |
| Asset naming and pipeline? | `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` |
| Performance and optimization? | `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md` |
| Testing and QA strategy? | `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md` |
| Build, signing, and release? | `28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md` |
| AI coding rules and behavior? | `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md` |
| Implementation phase order? | `30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md` |

---

## 17. Final AI Instruction

> **Read this file first.**
> Then read ONLY the documents relevant to your current task.
> Never invent a rule, value, or system that already has a documented owner.
> If a specification is missing, ask the developer.
> When in doubt: be surgical, be explicit, be testable.

---

**End of `AI_START_HERE.md`.**
