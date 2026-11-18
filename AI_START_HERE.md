# AI_START_HERE.md
## Block Battles — AI Orientation Document

**Read this file first. Always.**
This is the single entry point for any AI coding agent, language model, or automated tool interacting with this repository. It provides project orientation and authoritative document routing. Do not skip it.

---

## 1. Project Identity

**Name:** Block Battles (Pure Arcade & Stage Puzzle Engine)

**One-sentence definition:** Block Battles is a tactile, premium mobile block-placement puzzle game featuring an endless Classic High-Score mode and a Goal-Driven Stage Challenge mode.

**One-paragraph definition:**
Block Battles fuses the deeply satisfying, tactile loop of an 8×8 block-placement puzzle with state-of-the-art audiovisual polish, physical game feel, and competitive local device scoring. The player places geometric polyomino shapes from a 3-slot tray onto an 8×8 board. Completing full rows and columns clears lines, scores points, and triggers escalating streak multipliers with rich particle VFX, subtle screen shake, and melodic chimes. The app features an instant, distraction-free flow: App Icon → Snappy Splash → Mode Selection → Instant Play. Mode 1 offers an endless high-score loop competing against local device records until board lock; Mode 2 offers a curated progression of bite-sized stage puzzles with target scores, color quotas, and obstacle clears. Designed mobile-first, offline, and solo-developer ready.

---

## 2. Core Game Loop (Visual)

```text
[APP LAUNCH]
     │
     ▼
[ANIMATED SPLASH SCREEN] (Smooth branded animation)
     │
     ▼
[MODE SELECTION MENU]
     │
     ├─────────────────────────────────────────┐
     ▼                                         ▼
[MODE 1: CLASSIC ENDLESS]                 [MODE 2: STAGE CHALLENGE]
     │                                         │
     ▼                                         ▼
┌─────────────────────────┐               ┌─────────────────────────┐
│ 8×8 PUZZLE BOARD        │               │ 8×8 BOARD WITH GOALS    │
│ 3-Slot Shape Tray       │               │ Target Score, Color Qty │
│ Endless High Score Loop │               │ Move Limit, Obstacles   │
└────────────┬────────────┘               └────────────┬────────────┘
             │                                         │
             ▼                                         ▼
   ┌───────────────────┐                     ┌───────────────────┐
   │ PLAYER TURN       │◄────────────────┐   │ PLAYER TURN       │
   │ Pick piece (lift) │                 │   │ Place piece       │
   │ Drag (ghost glow) │                 │   │ Track move budget │
   │ Drop & Snap       │                 │   └─────────┬─────────┘
   └─────────┬─────────┘                 │             │
             │                           │             ▼
             ▼                           │     Goals Completed?
     Rows/Cols full?                     │     YES: STAGE CLEAR (1-3 Stars)
      YES │     NO │                     │     NO: Move Limit Reached?
          │        └────────────────────►┤         YES: STAGE FAILED
          ▼                              │
  LINE CLEAR & JUICE                     │
  Simultaneous Row/Col Scan              │
  White-hot Flash + Particles            │
  Melodic Chime + Screen Shake           │
  Score + Streak Multiplier (1.0x-2.5x)  │
          │                              │
          ▼                              │
  Tray empty? ──YES──► Refill 3 Shapes   │
          │                              │
          ▼                              │
  Any legal placement for tray pieces?   │
     YES │      NO (Board Lock)          │
         └─────────────┬─────────────────┘
                       │
                       ▼
               [GAME OVER DIALOG]
          Final Score & High Score Check
          Celebratory New Best Fanfare
          One-Tap "Play Again" or "Home"
```

---

## 3. What Makes the Game Different

| Feature | What It Does |
|---|---|
| **Zero "AI Slop" Visuals** | Beveled, rich gradient block materials, frosted glass board frame, glowing ghost previews, and white-hot particle dissipation. |
| **Tactile Physical Game Feel** | Finger-occlusion lift, elastic drag curves, micro-snap haptics, and proportionate camera shake on multi-line clears. |
| **Pure Zen Endless Loop** | No timers, no stress, no energy gates. Pure spatial puzzle mastery and chasing your own device high score. |
| **Two Cohesive Modes** | Mode 1 (Endless High Score) for endless flow; Mode 2 (Goal-Driven Stages) for curated puzzle challenges with move limits. |
| **Deterministic Fair RNG** | Board-aware shape generation ensures solvable, intelligent piece generation with duplicate prevention. |
| **Clean Decoupled Architecture**| Pure Dart domain logic decoupled from Flame rendering and Flutter UI for instantaneous unit testing. |

---

## 4. Core Player Experience

The game should feel: **zen, tactile, premium, satisfying — with moments of explosive arcade reward.**

The player should feel clever, not lucky. The board is a spatial playground. Placing a piece must feel physically *right* — an elastic pickup lift, a tactile snap, and a clean haptic pulse. A multi-line clear should feel like an explosive celebration. Losing should feel earned and immediately prompt a one-tap retry.

---

## 5. MVP Scope

### Included Systems
- 8×8 grid board with responsive mobile auto-scaling.
- 3-slot tray with deterministic `GameRNG` and solvability protection.
- 34 canonical playable shape definitions across 15 families / 9 categories (`02_SHAPE_LIBRARY.md`).
- Simultaneous line clearing (rows + columns) with single-clear intersection union.
- Pure puzzle scoring system (placement points + line clear points + multi-line bonuses + streak multipliers).
- Mode 1 (Classic Endless): Board lock detection, local high score persistence (`hive_ce`), game over celebration.
- Mode 2 (Stage Challenges): Move limits, target score goals, color block collection quotas, obstacle clearing (stone, ice).
- Local persistence: Hive CE for game progression and active sessions; SharedPreferences for audio/haptic settings.
- Rich audiovisual juice: beveled block shaders, particle bursts, screen shake, audio chimes.
- Offline-first: 100% playable without an internet connection.

### Excluded Scope
- No extraneous non-puzzle entities, fighting mechanics, or character stats.
- No equipment drafts, card mechanics, or inventory systems.
- No cloud save or mandatory external server backend.
- No energy or stamina paywalls.

---

## 6. Core Rules (Non-Negotiable)

| Rule | Value |
|---|---|
| Board Size | 8×8 grid (64 cells) |
| Tray Slots | 3 (refills when all 3 placed) |
| Piece Rotation | NONE. Pieces have fixed authored orientations. |
| Line Clear | Full row OR full column clears. Both clear simultaneously. |
| Intersection Rule | Intersection cells between cleared rows and cols are cleared exactly once. |
| Scoring System | Cell Placement Points (1 pt/cell) + Line Clear Points ($L=1\to 100, L=2\to 300, L=3\to 600, L=4\to 1000$) $\times$ Streak Multiplier ($1.0\times\dots 2.5\times$) + Board Clear Bonus (+500 pts). |
| Game Over (Mode 1) | Triggered when none of the available tray pieces has any legal placement. |
| High Score | Persisted locally on device; evaluated on game over. |
| Input | Touch drag with finger-lift offset and live validity ghost preview. |

All numeric formulas and parameters are authored exclusively in design documents. **Never hardcode them into logic.**

---

## 7. Architecture Summary

### Data-Driven Design
Shapes, Stages, and Objectives are Dart `const` data classes. Adding content requires authoring data assets, not modifying pure logic. See `20`.

### RNG & Determinism
A seeded `GameRNG` class controls all piece spawns.
- **New Sessions:** Deterministic from seed + inputs (the same seed + input sequence always produces the same game state).
- **Resumable Active Sessions:** Restore exact serialized runtime state including deterministic RNG continuation counter. See `04`, `21`, and `24`.

### Event-Driven Communication
Domain systems (`BoardEngine`, `ScoreService`, `StageObjectiveEngine`) emit events via Dart streams. Presentation systems (UI, Audio, VFX) listen to events and react. The Domain NEVER calls into Presentation. See `21`, `22`.

### Clean Architecture Layers
- **Domain Layer** (Pure Dart): Board math, scoring algorithms, RNG, shape models.
- **Application Layer** (Orchestration): Session flow, Classic/Stage controllers.
- **Presentation Layer** (Flutter/Flame): Flame components, Flutter widgets, GoRouter routes.

---

## 8. Technology Stack

| Concern | Technology |
|---|---|
| Language | Dart |
| App Framework | Flutter |
| Game Engine | Flame |
| State Management | Riverpod |
| Navigation | GoRouter |
| Local Save Data | Hive CE |
| Lightweight Preferences | SharedPreferences |
| Game Audio | Flame Audio |
| Platform | Android-First (with iOS support) |
| Testing | `dart test`, `flutter test`, `integration_test` |

---

## 9. Major Runtime Systems

| System | Responsibility |
|---|---|
| `BoardEngine` | Owns the 8×8 cell-state array. Validates placements and executes line clears. |
| `TrayEngine` | Manages the 3 active pieces. Replenishes pieces from `GameRNG`. |
| `ScoreService` | Calculates placement points, line clear points, and consecutive streak multipliers. |
| `StageObjectiveEngine`| Mode 2 goal progress evaluator (target score, color quotas, line milestones). |
| `ClassicGameService` | Mode 1 session flow orchestrator (high scores, board lock detection, game over). |
| `StageGameService` | Mode 2 stage flow orchestrator (move limits, star evaluation, stage completion). |
| `SaveRepository` | Reads and writes local progression and active session data via Hive CE. |
| `AudioService` | Flame Audio wrapper managing sound effects and ambient music loops. |

---

## 10. Source-of-Truth Routing Table

| Decision | Authority |
|---|---|
| Master Vision, Design Pillars | `00_MASTER_GAME_VISION.md` |
| Core Gameplay Rules & Turn Loop | `01_GAMEPLAY_SPECIFICATION.md` |
| Shape Geometry & 34 Canonical Shapes | `02_SHAPE_LIBRARY.md` |
| Board Engine, Grid Math & Clears | `03_BOARD_ENGINE_AND_RULES.md` |
| Spawn RNG, Solvability & Difficulty | `04_SPAWN_RNG_AND_DIFFICULTY.md` |
| Scoring, Combos & Streak Multipliers | `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` |
| Drag & Drop Input UX, Finger Offset | `06_INPUT_DRAG_AND_DROP_UX.md` |
| Mode 2 Stage Objectives & Levels | `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md` |
| UI/UX Master Specification | `08_UI_UX_MASTER_SPECIFICATION.md` |
| Visual Design System | `09_VISUAL_DESIGN_SYSTEM.md` |
| Color System & Palette Themes | `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` |
| Block Art, Materials & Shaders | `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md` |
| Motion, Animation & Game Feel | `12_ANIMATION_MOTION_AND_GAME_FEEL.md` |
| VFX, Particles & Line Dissolves | `13_VFX_PARTICLE_AND_IMPACT_SPEC.md` |
| Audio SFX, Music & Mix | `14_AUDIO_SFX_MUSIC_AND_MIXING.md` |
| Player-Facing Copy & Localization | `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md` |
| Settings & Accessibility | `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md` |
| Monetization & Ads | `17_MONETIZATION_REWARD_AND_AD_UX.md` |
| Technical Architecture | `18_TECHNICAL_ARCHITECTURE.md` |
| Project Structure & Flutter/Flame | `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md` |
| Data Model & Content Schemas | `20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md` |
| System Managers & Dependency Inj. | `21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md` |
| Event Bus & State Orchestration | `22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md` |
| Flutter UI Implementation | `23_UI_IMPLEMENTATION_ARCHITECTURE.md` |
| Save Data & Persistence | `24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md` |
| Asset Pipeline & Naming | `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md` |
| Performance & Mobile Optimization | `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md` |
| Testing, QA & Automation | `27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md` |
| Build Configuration & Release | `28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md` |
| AI Coding Agent Rules | `29_AI_CODING_AGENT_DEVELOPMENT_RULES.md` |
| Implementation Roadmap | `30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md` |

---

## 11. Final AI Instruction

> **Read this file first.**
> Then read ONLY the documents relevant to your current task.
> Never invent a rule, value, or system that already has a documented owner.
> If a specification is ambiguous, consult the governing documents.
> Keep all code surgical, clean, and testable.

---

**End of `AI_START_HERE.md`.**
