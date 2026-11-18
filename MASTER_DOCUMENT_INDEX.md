# MASTER_DOCUMENT_INDEX.md
## Block Battles — Master Document Index & Knowledge Architecture

**Author:** Technical Architecture & Documentation Team  
**Governing document:** `00_MASTER_GAME_VISION.md`  
**Document status:** Living Index — Updated with every document addition or modification  
**Target:** Engineering, Design, Art, Audio, and AI Coding Agents

---

## 1. Documentation Philosophy

**"The documentation is the design; the code is the implementation."**

Every discrete design and engineering decision belongs to exactly one document. If the scoring formula exists in `05`, it exists NOWHERE else — not in comments, not in chat history, not in a developer's head.

1. **Single Source of Truth:** Every value, formula, behavior, and visual property has one authoritative document.
2. **Explicit Cross-References:** Documents reference other documents by ID and title.
3. **Inheritance & Hierarchy:** Layer A documents govern Layer B, which govern Layer C. No child document may contradict its parent.
4. **Zero Ambient Knowledge:** If a rule or constraint is not written in these documents, it does not exist.

---

## 2. Document Hierarchy

```text
LAYER A — PRODUCT & GAMEPLAY FOUNDATIONS (Vision & Rules)
  ├── 00_MASTER_GAME_VISION.md
  ├── 01_GAMEPLAY_SPECIFICATION.md
  ├── 02_SHAPE_LIBRARY.md
  ├── 03_BOARD_ENGINE_AND_RULES.md
  ├── 04_SPAWN_RNG_AND_DIFFICULTY.md
  ├── 05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md
  ├── 06_INPUT_DRAG_AND_DROP_UX.md
  └── 07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md

LAYER B — PRESENTATION & SENSORY DESIGN (Look & Feel)
  ├── 08_UI_UX_MASTER_SPECIFICATION.md
  ├── 09_VISUAL_DESIGN_SYSTEM.md
  ├── 10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md
  ├── 11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md
  ├── 12_ANIMATION_MOTION_AND_GAME_FEEL.md
  ├── 13_VFX_PARTICLE_AND_IMPACT_SPEC.md
  ├── 14_AUDIO_SFX_MUSIC_AND_MIXING.md
  ├── 15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md
  ├── 16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md
  └── 17_MONETIZATION_REWARD_AND_AD_UX.md

LAYER C — ENGINEERING SPECIFICATIONS (Code Architecture)
  ├── 18_TECHNICAL_ARCHITECTURE.md
  ├── 19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md
  ├── 20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md
  ├── 21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md
  ├── 22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md
  ├── 23_UI_IMPLEMENTATION_ARCHITECTURE.md
  ├── 24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md
  ├── 25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md
  ├── 26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md
  └── 27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md

LAYER D — PRODUCTION & EXECUTION (Delivery)
  ├── 28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md
  ├── 29_AI_CODING_AGENT_DEVELOPMENT_RULES.md
  └── 30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md
```

---

## 3. Master Document Registry

| Doc ID | Layer | Title | Authoritative Domain |
|---|---|---|---|
| `00` | A | Master Game Vision | Core fantasy, pillars, MVP scope, dual-mode definition |
| `01` | A | Gameplay Specification | Turn loop, placement rules, game states |
| `02` | A | Shape Library | 34 canonical shapes, coordinates, families, categories |
| `03` | A | Board Engine & Rules | 8×8 grid state, collision checks, line clears |
| `04` | A | Spawn RNG & Difficulty | Spawner weights, solvability protection, Mulberry32 |
| `05` | A | Scoring, Combo & Streak Formulas | Placement score, line bonuses, streak multipliers |
| `06` | A | Input Drag & Drop UX | Finger-lift offset, ghost preview, snapping |
| `07` | A | Stage Objectives & Level Design | Mode 2 stage definitions, goals, obstacles, stars |
| `08` | B | UI/UX Master Specification | Screen hierarchy, HUD layouts, modal flows |
| `09` | B | Visual Design System | Typography, glassmorphism, depth, visual hierarchy |
| `10` | B | Color System & Block Art | 8-theme color palette, contrast ratios |
| `11` | B | Block Visual Material Spec | Beveled shading, borders, stone and ice tiles |
| `12` | B | Animation Motion & Feel | Interpolation curves, shake durations, clear timings |
| `13` | B | VFX Particle & Impact Spec | Particle count, spark bursts, line dissolve effects |
| `14` | B | Audio SFX, Music & Mixing | SFX asset inventory, harmonic chimes, loudness |
| `15` | B | Screen Content & Copy | Player-facing strings, modal text, terminology |
| `16` | B | Settings & Accessibility | Preferences, volume sliders, reduced motion |
| `17` | B | Monetization & Ad UX | Rewarded revive, premium ad-free unlock |
| `18` | C | Technical Architecture | Clean architecture, domain isolation, dependency rules |
| `19` | C | Project Structure | Folder layout, Flutter/Flame component mapping |
| `20` | C | Data Models & Schemas | Dart data definitions, stage schemas, state classes |
| `21` | C | System Managers & Services | Service lifecycles, Riverpod providers, pure engines |
| `22` | C | Event Bus & State Machine | Dart stream events, state orchestration |
| `23` | C | UI Implementation | Flutter widget stack, HUD layering, reactive binding |
| `24` | C | Save Data & Persistence | Hive CE schemas, atomic writes, versioning |
| `25` | C | Asset Pipeline & Naming | Asset paths, snake_case standards, image resolutions |
| `26` | C | Performance & Mobile Opt. | 60 FPS budgets, memory pooling, GC avoidance |
| `27` | C | Testing, QA & Automation | Unit tests, widget tests, deterministic verification |
| `28` | D | Build & Release Config | Android app bundle, signing, store configuration |
| `29` | D | AI Coding Agent Rules | Behavior guidelines, scope discipline, test contracts |
| `30` | D | Implementation Roadmap | 5 Master Phases for pure puzzle execution |

---

## 4. Topic Lookup

**"What determines line clearing?"**  
→ Rules: `03_BOARD_ENGINE_AND_RULES.md`. Scoring: `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`. Implementation: `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md`.

**"How are stage objectives authored for Mode 2?"**  
→ Design: `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`. Schemas: `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md`. Engine: `StageObjectiveEngine` per `21`.

**"How are colors defined and where do they live?"**  
→ `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`. Implemented via `23_UI_IMPLEMENTATION_ARCHITECTURE.md`.

**"How are shapes stored and generated?"**  
→ Definitions: `02_SHAPE_LIBRARY.md` and `20`. Generator: `04_SPAWN_RNG_AND_DIFFICULTY.md` and `TrayEngine` per `21`.

**"How does a session start?"**  
→ Flow: `01_GAMEPLAY_SPECIFICATION.md`. Lifecycle: `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md`. Code: `ClassicGameService` / `StageGameService` per `21`.

**"Where should I add a new Stage?"**  
→ Add `StageDefinition` to `stage_catalog.dart` per schema in `20` and level design in `07`. No engine modification required.

**"Where should I add a new Shape?"**  
→ Add to `shape_catalog.dart` per `02` and `20`. Update `04` spawn weights.

**"What should happen on Board Lock?"**  
→ Game rule: `01`, `03`. State transition: `22`. UI presentation: `08`, `23`.

**"What controls save data?"**  
→ Schema: `24`. Runtime owner: `SaveRepository` per `21`. Data classes: `20`.

**"What is the drag offset value for pieces?"**  
→ `06_INPUT_DRAG_AND_DROP_UX.md`. Saved to settings per `16`. Applied in code per `23`.

---

## 5. Locked Architectural Decisions

| ID | Decision | Authoritative Doc | Status |
|---|---|---|---|
| D-001 | Board size is 8×8, fixed geometry. | `01`, `03` | LOCKED |
| D-002 | No piece rotation mechanic. | `01`, `02` | LOCKED |
| D-003 | Tray holds exactly 3 pieces, batch replenishment. | `01`, `04` | LOCKED |
| D-004 | Multi-line scoring scales $L=1\dots 4+$; Streak multipliers scale $1.0\times, 1.5\times, 2.0\times, 2.5\times$ cap. | `05` | LOCKED |
| D-005 | Android-first, offline, no server requirement. | `00`, `28` | LOCKED |
| D-006 | Domain Layer has zero Flutter/Flame references. | `18` | LOCKED |
| D-007 | Deterministic seeded RNG (`GameRNG`). | `04`, `18` | LOCKED |
| D-008 | Object pooling for Pieces, Cells, and VFX in Flame. | `26` | LOCKED |
| D-009 | Zero heavy allocations in tight placement loop. | `26` | LOCKED |
| D-010 | Event payloads are pure Dart classes/records. | `22` | LOCKED |
| D-011 | Flutter strictly handles UI, Flame handles the board. | `23` | LOCKED |
| D-012 | Two-Tier Persistence: Hive CE for game progression and sessions, SharedPreferences for user settings. | `24` | LOCKED |
| D-013 | Monetization: Rewarded Ad + Premium Unlock only. | `17` | LOCKED |
| D-014 | 34 canonical shapes in 15 families (9 categories) from Day One. | `02`, `20`, `30` | LOCKED |
| D-015 | Pure two-mode taxonomy (Classic Endless & Stage Challenge). | `00`, `07`, `20` | LOCKED |

---

## 6. Open Decision Registry

| ID | Question | Resolution Milestone | Relevant Docs |
|---|---|---|---|
| O-001 | Exact Flutter/Flame stable SDK versions? | Project setup. | `19` |
| O-002 | Ad Network choice (e.g., AdMob, Unity Ads)? | Phase 5. | `17`, `28` |
| O-003 | Crashlytics integration strategy? | Release Phase. | `28` |
| O-004 | Final Package Identifier (`com.company.blockbattles`)? | Release Phase. | `28` |
| O-005 | Stage count for MVP launch (30 or 50)? | Stage Content Phase. | `07`, `20` |
| O-006 | Exact ambient BGM track selection for MVP? | Audio Polish. | `14` |

---

## 7. Final Documentation Contract

> **No major gameplay, UX, visual, architectural, or data decision should exist only inside code or chat history.**
>
> It belongs in the appropriate source-of-truth document. This repository is its own specification.

---

**End of `MASTER_DOCUMENT_INDEX.md`.**
