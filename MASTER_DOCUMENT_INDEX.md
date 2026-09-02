# MASTER_DOCUMENT_INDEX.md
## Block Battles — Complete Documentation Navigation System

**Read after:** `AI_START_HERE.md`
**Document status:** Meta-Layer — Navigation Authority
**Total files indexed:** 32 Markdown documents

---

## 1. Project Documentation Philosophy

### One Source of Truth Per Topic
Every discrete design and engineering decision belongs to exactly one document. If the damage formula exists in `05`, it exists NOWHERE else — not in comments, not in chat history, not in a developer's head.

### Dependency Ordering
Documents are ordered by dependency. Layer A (Design) must be stable before Layer B (Presentation) can finalize. Layer B must be stable before Layer C (Engineering) can fully lock. Reading a document before its dependencies leads to incomplete understanding.

### Conflict Resolution
If two documents appear to contradict each other:
1. The **lower-numbered** document (closer to Layer A) wins for design decisions.
2. The **higher-numbered** Layer C document wins for implementation decisions within its domain.
3. When in doubt: `18_TECHNICAL_ARCHITECTURE.md` is the supreme engineering authority.
4. If still unclear: flag the conflict, do NOT resolve it silently in code.

### Update Responsibility
When any system changes, the developer (or AI agent) must update the governing document. Code is allowed to evolve; the specification must evolve with it. A document that no longer reflects the code is worse than no document.

---

## 2. Complete File Tree

```text
E:\Block Buster\
├── AI_START_HERE.md                                    ← Read first
├── MASTER_DOCUMENT_INDEX.md                            ← Read second (this file)
│
├── [LAYER A — GAME DESIGN]
│   ├── 00_MASTER_GAME_VISION.md
│   ├── 01_GAMEPLAY_SPECIFICATION.md
│   ├── 02_SHAPE_LIBRARY.md
│   ├── 03_BOARD_ENGINE_AND_RULES.md
│   ├── 04_SPAWN_RNG_AND_DIFFICULTY.md
│   ├── 05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md
│   ├── 06_INPUT_DRAG_AND_DROP_UX.md
│   └── 07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md
│
├── [LAYER B — PRESENTATION DESIGN]
│   ├── 08_UI_UX_MASTER_SPECIFICATION.md
│   ├── 09_VISUAL_DESIGN_SYSTEM.md
│   ├── 10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md
│   ├── 11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md
│   ├── 12_ANIMATION_MOTION_AND_GAME_FEEL.md
│   ├── 13_VFX_PARTICLE_AND_IMPACT_SPEC.md
│   ├── 14_AUDIO_SFX_MUSIC_AND_MIXING.md
│   ├── 15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md
│   ├── 16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md
│   └── 17_MONETIZATION_REWARD_AND_AD_UX.md
│
├── [LAYER C — ENGINEERING]
│   ├── 18_TECHNICAL_ARCHITECTURE.md
│   ├── 19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md
│   ├── 20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md
│   ├── 21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md
│   ├── 22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md
│   ├── 23_UI_IMPLEMENTATION_ARCHITECTURE.md
│   ├── 24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md
│   ├── 25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md
│   ├── 26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md
│   ├── 27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md
│   └── 28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md
│
└── [LAYER D — META / AGENT DIRECTIVES]
    ├── 29_AI_CODING_AGENT_DEVELOPMENT_RULES.md
    └── 30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md
```

---

## 3. Layer Overview

### Layer A — Game Design (Documents 00–07)
These documents define **what the game is and how it plays**. They are the ground truth for all game rules, mechanics, and content. No code or presentation decision may contradict Layer A. These files must be locked before any coding begins.

### Layer B — Presentation Design (Documents 08–17)
These documents define **what the player sees, hears, and feels**. UI layouts, animation timings, color values, audio behavior, and player-facing copy. Layer B inherits from Layer A and may not introduce mechanics.

### Layer C — Engineering (Documents 18–28)
These documents define **how the game is built**. Unity project structure, C# architecture, data schemas, event systems, persistence, performance, QA, and release. Layer C translates all design into precise technical contracts.

### Layer D — Meta / Agent Directives (Documents 29–30 + Context Files)
These documents govern **how the project is navigated and executed**. They exist to orient new agents, prevent rework, and enforce discipline in solo development.

---

## 4. Complete Document Registry

| File | Layer | One-Line Purpose | Primary Owner | Depends On | Used By |
|---|---|---|---|---|---|
| `AI_START_HERE.md` | D | Project orientation for any AI agent. | All | None | All agents |
| `MASTER_DOCUMENT_INDEX.md` | D | Navigation map for the full doc set. | All | All | All agents |
| `00_MASTER_GAME_VISION.md` | A | High-level vision, tone, platform goals. | Game Director | None | All docs |
| `01_GAMEPLAY_SPECIFICATION.md` | A | Core rules: board, tray, placement, win/lose. | Game Designer | 00 | 03, 05, 06, 18 |
| `02_SHAPE_LIBRARY.md` | A | 24 canonical MVP shape entries (SHP_001–024) from a 34-entry full catalog. Entries SHP_025–034 are deferred to post-MVP. | Game Designer | 01 | 04, 20 |
| `03_BOARD_ENGINE_AND_RULES.md` | A | 8x8 board collision, line clearing rules. | Game Designer | 01, 02 | 05, 18, 21 |
| `04_SPAWN_RNG_AND_DIFFICULTY.md` | A | Seeded RNG, piece spawn pools, difficulty. | Game Designer | 02, 03 | 18, 20, 21 |
| `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` | A | L-Value formula, damage table, combo logic. | Game Designer | 03 | 07, 18, 20, 21 |
| `06_INPUT_DRAG_AND_DROP_UX.md` | A | Touch input, drag offset, ghost preview. | UX Designer | 01, 03 | 12, 23 |
| `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md` | A | Objective types, enemy definitions, level flow. | Game Designer | 01, 05 | 08, 20, 21 |
| `08_UI_UX_MASTER_SPECIFICATION.md` | B | All screens, flows, and navigation structure. | UX Designer | 00–07 | 15, 23 |
| `09_VISUAL_DESIGN_SYSTEM.md` | B | Typography, grid system, spacing, iconography. | Art Director | 00, 08 | 10, 11, 23 |
| `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` | B | Color palette, tokens, contrast, rarity hues. | Art Director | 09 | 11, 13, 23 |
| `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md` | B | Block face art, bevel, gradients, materials. | Technical Artist | 09, 10 | 13, 25 |
| `12_ANIMATION_MOTION_AND_GAME_FEEL.md` | B | All animation timings, easing, and game feel. | Animation Director | 06, 09 | 23, 26 |
| `13_VFX_PARTICLE_AND_IMPACT_SPEC.md` | B | Particle systems, impact effects, VFX scale. | VFX Artist | 10, 11, 12 | 21, 26 |
| `14_AUDIO_SFX_MUSIC_AND_MIXING.md` | B | SFX events, music structure, mix priorities. | Audio Director | 00, 05 | 16, 21 |
| `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md` | B | All UI strings, tone, terminology glossary. | UX Writer | 08, 07 | 23 |
| `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` | B | All player settings, a11y, reduced motion. | UX/A11y | 08, 12, 14 | 18, 24 |
| `17_MONETIZATION_REWARD_AND_AD_UX.md` | B | Ad flow, IAP, ethical monetization design. | Product | 00, 08 | 28 |
| `18_TECHNICAL_ARCHITECTURE.md` | C | Clean Architecture, layering, anti-patterns. | Tech Lead | 00–17 | 19–28 |
| `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md` | C | Folders, scenes, ASMDEFs, prefab hierarchy. | Unity Architect | 18 | 20–28 |
| `20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md` | C | All SO schemas, runtime POCO definitions. | Data Architect | 18, 01–07 | 21, 24 |
| `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md` | C | Every runtime system, its lifetime, ownership. | Systems Architect | 18, 19, 20 | 22, 23 |
| `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md` | C | EventBus API, state machine, event payloads. | Systems Architect | 18, 21 | 23, 26 |
| `23_UI_IMPLEMENTATION_ARCHITECTURE.md` | C | UI Canvas layers, prefabs, data binding. | UI Engineer | 08, 18, 22 | 26 |
| `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md` | C | Save schema, atomic writes, migration. | Systems Engineer | 16, 20, 21 | 28 |
| `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` | C | Naming convention, import settings, AI art QA. | Technical Artist | 09–17, 19 | 26, 28 |
| `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md` | C | FPS/mem budgets, GC rules, pooling mandates. | Performance Eng | 18–25 | 27, 28 |
| `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md` | C | Test types, device matrix, release gates. | QA Lead | All | 28 |
| `28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md` | C | Build settings, signing, store assets, rollout.| Release Eng | All | None |
| `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md` | D | AI operating manual, forbidden behaviors. | Tech Lead | All | AI agents |
| `30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md` | D | 21-phase build order, milestones, scope lock. | Producer | All | Developer |

---

## 5. Topic Lookup

**"What determines line clearing?"**
→ Rules: `03_BOARD_ENGINE_AND_RULES.md`. Damage: `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`. Implementation: `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md`.

**"What does an enemy do on its turn?"**
→ `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md` (design). `20_DATA_MODEL_SCRIPTABLEOBJECTS_AND_CONTENT_SCHEMA.md` (EnemyAction schema). `21` (EnemyEngine ownership).

**"How are colors defined and where do they live?"**
→ `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` (authoritative palette). Applied to code via `23_UI_IMPLEMENTATION_ARCHITECTURE.md`.

**"Where are Relics stored and how are they read?"**
→ Authoring: `Assets/_Project/Data/Relics/`. Schema: `20`. Runtime: `RelicEngine.cs` per `21`.

**"How does a battle start?"**
→ Flow: `07`. Lifecycle: `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md` (State Machine). Code: `BattleOrchestrator` per `21`.

**"Where should I add a new Enemy?"**
→ 1) Create `EnemyData.asset` per schema in `20`. 2) Author HP + action pattern per design in `07`. 3) Reference in `EncounterData.asset`. No C# changes required.

**"Where should I add a new Relic?"**
→ 1) Create `RelicData.asset` per schema in `20`. 2) Verify Trigger and EffectType enums support it (see `21`). 3) If a new enum is needed, update `22` and the RelicEngine.

**"How do I modify a button's appearance?"**
→ Visual spec: `09`. Color: `10`. Copy: `15`. Unity implementation: `23`.

**"What should happen on a Board Lock or Run Defeat?"**
→ Game rule: `01`, `03`. State transition: `22`. UI screen: `08`. Implementation: `23`.

**"What controls save data?"**
→ Schema: `24`. Runtime owner: `SaveService` per `21`. Data classes: `20`.

**"How are animation durations defined?"**
→ `12_ANIMATION_MOTION_AND_GAME_FEEL.md`. Applied in code via `23`.

**"What is the drag offset value for pieces?"**
→ `06_INPUT_DRAG_AND_DROP_UX.md`. Saved to settings per `16`. Applied in code per `23`.

**"Which file defines the folder where I put Enemy sprites?"**
→ `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md` Sec 2, and `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` Sec 4.

**"How is the Stardust economy balanced?"**
→ `05` (earn rates), `17` (monetization context). Save field defined in `24`.

**"What devices must the game run on?"**
→ `26` Sec 2 (Device Matrix) and `27` Sec 16.

**"What is the naming convention for audio files?"**
→ `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` Sec 3.

---

## 6. Dependency Graph

Layer A documents form the foundation. Nothing higher can be built without them being stable.

```text
                    ┌──────────────────────────┐
                    │  00_MASTER_GAME_VISION   │
                    └────────────┬─────────────┘
                                 │ (Informs All)
         ┌───────────────────────┴───────────────────────┐
         ▼                       ▼                       ▼
   01_GAMEPLAY            07_OBJECTIVES           LAYER B (08-17)
   SPECIFICATION               │                  (Presentation)
         │                     │                       │
    ┌────┴────┐           ┌────┴────┐                   │
    │02_SHAPE │           │05_COMBAT│                   │
    │LIBRARY  │           │SCORING  │                   │
    └────┬────┘           └────┬────┘                   │
         │                     │                        │
    ┌────┴────┐           ┌────┴────┐                   │
    │03_BOARD │           │06_INPUT │                   │
    │ENGINE   │           │DRAG_UX  │                   │
    └────┬────┘           └─────────┘                   │
         │                                              │
    ┌────┴────┐                                         │
    │04_RNG   │                                         │
    │SPAWN    │                                         │
    └─────────┘                                         │
                                                        │
         All Layer A + B feed into →   LAYER C (18-28)
                                       (Engineering)
                                             │
                                             ▼
                                    LAYER D (29-30)
                                    (Meta/Execution)
```

---

## 7. Authority Rules

| Situation | Resolution |
|---|---|
| Code contradicts `01_GAMEPLAY_SPECIFICATION` | Fix the code. |
| `05` formula contradicts `18` description | `05` is authoritative for the math. `18` describes it. Correct `18`. |
| `10` color token contradicts a Prefab color | Correct the Prefab. `10` is the color authority. |
| `24` save schema contradicts actual JSON | Update the schema document and add a migration. |
| Two Layer C docs disagree on architecture | `18_TECHNICAL_ARCHITECTURE.md` is the supreme engineering authority. |
| An AI agent invents a rule not in any doc | Reject it. Flag it. Ask the developer to make a deliberate decision. |

---

## 8. Change Impact Rules

When a **gameplay rule** changes (e.g., damage formula):
→ Update `05`. Review `07` (objective balance). Review `18` (architecture impact). Run domain unit tests from `27`.

When a **UI layout** changes:
→ Update `08`. Review `23` (implementation). Check `15` (copy changes). Check `12` (animation changes). Verify safe areas per `19`.

When a **color** changes:
→ Update `10`. Review `11` (block material). Review `09` (design system). Update Prefabs. Run visual QA from `27`.

When **architecture** changes (new system, new manager):
→ Update `18`. Review `21` (ownership). Update `22` (new events if needed). Update `19` (folder if needed). Add to `29`.

When a **data schema** changes (new SO field):
→ Update `20`. Review `24` (save impact). Write a migration if save data is affected. Update tests in `27`.

When **save structure** changes:
→ Update `24`. Increment `saveVersion`. Write `IMigrationStrategy`. Run serialization roundtrip test from `27`.

When **monetization** changes:
→ Update `17`. Review `28` (build separation). Review `16` (consent impact). Review `15` (copy changes).

---

## 9. AI Reading Strategy

### First Read
`AI_START_HERE.md` — Always. No exceptions. This is the compressed orientation layer.

### Second Read
`MASTER_DOCUMENT_INDEX.md` (this file) — To understand where to find specific answers.

### Third Read
Only the **specific document governing your current task**. Do not read all 32 files for a small code change. Use Section 5 (Topic Lookup) or Section 16 of `AI_START_HERE.md` to find the right document.

### Do NOT Read Exhaustively
Reading every document before writing a single line of code is wasteful. Route your questions. Read the source of truth. Make surgical, documented changes.

---

## 10. Task-to-Document Matrix

| Task | Primary Doc | Supporting Docs |
|---|---|---|
| Implement BoardModel.cs | `03` | `18`, `21`, `22` |
| Add a new ShapeData asset | `02` | `20`, `25` |
| Implement CombatEngine.cs | `05` | `18`, `21`, `22` |
| Balance damage values | `05` | `07` |
| Add a new EnemyData asset | `07` | `20`, `25` |
| Add a new RelicData asset | `05`, `07` | `20`, `21` |
| Build TrayView.cs | `06` | `23`, `12` |
| Build EnemyHUDView.cs | `08` | `23`, `10`, `15` |
| Add a new UI Modal | `08` | `23`, `09`, `10` |
| Implement SaveService.cs | `24` | `20`, `21` |
| Configure Build Settings | `28` | `26`, `19` |
| Import a new Sprite | `25` | `19` |
| Write NUnit tests | `27` | `20`, `21`, `22` |
| Fix a performance spike | `26` | `22`, `23` |
| Implement a VFX | `13` | `25`, `22` |
| Add a Settings toggle | `16` | `23`, `24` |
| Implement MigrationV2 | `24` | `20` |
| Set up scenes | `19` | `22` |
| Debug a turn-order bug | `22` | `21`, `27` |

---

## 11. Implementation Mapping

| Document | Primary C# Class(es) |
|---|---|
| `03` | `BoardModel.cs`, `LineScanner.cs`, `BoardLockDetector.cs` |
| `04` | `GameRNG.cs`, `PieceSpawner.cs` |
| `05` | `CombatEngine.cs`, `ComboTracker.cs` |
| `06` | `InputController.cs`, `DragDropController.cs` |
| `07` | `ObjectiveEngine.cs`, `EnemyEngine.cs` |
| `20` | `ShapeData.cs`, `EnemyData.cs`, `RelicData.cs`, `EncounterData.cs` |
| `21` | `RunManager.cs`, `BattleOrchestrator.cs`, `ServiceLocator.cs` |
| `22` | `EventBus<T>.cs`, `BattleOrchestrator.cs` (State Machine) |
| `23` | `BoardView.cs`, `EnemyHUDView.cs`, `UIOrchestrator.cs` |
| `24` | `SaveService.cs`, `PlayerProfileSave.cs`, `MigrationV1toV2.cs` |
| `12` | `AnimationController.cs` (DOTween wrappers) |
| `13` | `VFXManager.cs`, all `pf_vfx_*.prefab` |
| `14` | `AudioService.cs`, `MusicController.cs` |
| `16` | `SettingsManager.cs`, `SettingsView.cs` |
| `17` | `MonetizationService.cs` |

---

## 12. Documentation Maintenance

### When a File Must Change
A document must be updated when:
- Its governing system's behavior changes.
- A new enum, field, or rule is introduced.
- A decision previously recorded in the document is reversed.
- A new document is added that references it.

### Who Owns Updates
The Developer and any AI agent executing a change are jointly responsible. AI agents MUST NOT change behavior without updating documentation.

### Version / Date Policy
Documents do not carry individual version numbers — the Git commit history serves as the version log. Every commit message must follow Conventional Commits format and reference the affected doc if it was modified.

### Deprecated Decisions
If a design decision is reversed, DO NOT delete the old text. Instead, prepend the section with `[DEPRECATED — See: NewSection]` so the change history is preserved.

---

## 13. Current Decision Registry (Locked)

| ID | Decision | Document | Status |
|---|---|---|---|
| D-001 | Board size is 8x8, no dynamic sizing. | `01`, `03` | LOCKED |
| D-002 | No piece rotation mechanic. | `01` | LOCKED |
| D-003 | Tray holds exactly 3 pieces. | `01` | LOCKED |
| D-004 | L-Value cap is L4 (4 simultaneous lines max). | `05` | LOCKED |
| D-005 | Android-first. No server backend for MVP. | `00`, `28` | LOCKED |
| D-006 | Domain Layer has zero UnityEngine references. | `18` | LOCKED |
| D-007 | All gameplay randomness uses a single seeded GameRNG. | `04`, `18` | LOCKED |
| D-008 | Object pooling mandatory for Pieces, Cells, VFX, AudioSources. | `26` | LOCKED |
| D-009 | No LINQ or dynamic allocation in the combat turn loop. | `26` | LOCKED |
| D-010 | All EventBus payloads are structs (not classes). | `22` | LOCKED |
| D-011 | TextMeshPro exclusively. No Legacy Text. | `23` | LOCKED |
| D-012 | Atomic Save pattern (tmp → bak → json). | `24` | LOCKED |
| D-013 | Monetization: Rewarded Ad + Premium Unlock only for MVP. | `17` | LOCKED |
| D-014 | Unity 2022.3 LTS or newer. | `19` | LOCKED |
| D-015 | IL2CPP scripting backend. ARM64 architecture. | `28` | LOCKED |
| D-016 | PlayerPrefs banned for progression data. | `24` | LOCKED |
| D-017 | Unity Physics (Rigidbody2D, Collider2D) banned for gameplay. | `18`, `19` | LOCKED |
| D-018 | DOTween for all UI animation. No Unity Animator on UI. | `23` | LOCKED |
| D-019 | Newtonsoft.Json for all save serialization. | `24` | LOCKED |
| D-020 | 100 PPU for all game sprites. | `25` | LOCKED |

---

## 14. Open Decision Registry (Unresolved)

| ID | Question | Blocking | Relevant Docs |
|---|---|---|---|
| O-001 | Exact Unity version patch (e.g., 2022.3.X)? | Soft block for setup. | `19` |
| O-002 | AdMob or Unity Ads as the Ad Network? | Phase 19. | `17`, `28` |
| O-003 | Firebase Crashlytics or Unity Cloud Diagnostics? | Phase 20. | `28` |
| O-004 | Final Package Identifier (com.X.Y)? | Phase 21. | `28` |
| O-005 | Studio/Company Name for store listing? | Phase 21. | `28` |
| O-006 | Exact Stardust prices for Relic Unlocks? | Phase 11. | `05`, `17` |
| O-007 | Exact BGM track count for MVP (2 or 3)? | Phase 16. | `14` |
| O-008 | Whether to include a tutorial on first boot? | Phase 12. | `08`, `23` |
| O-009 | Analytics SDK choice (Firebase Analytics vs Unity Analytics)? | Phase 20. | `28` |
| O-010 | Final Enemy count for MVP (5 confirmed, more possible)? | Phase 8. | `07`, `30` |

---

## 15. Final Documentation Contract

> **No major gameplay, UX, visual, architectural, data, or monetization decision should exist only inside code or chat history.**
>
> It belongs in the appropriate source-of-truth document.
>
> This repository is its own specification. When a developer opens any `.cs` file, they should be able to instantly locate the Markdown document that authorized every value, pattern, and rule they see in that code.
>
> A codebase without aligned documentation is a codebase that will be misunderstood, misimplemented, and broken by the next developer — human or AI — who touches it.

---

**End of `MASTER_DOCUMENT_INDEX.md`.**
