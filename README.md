# Block Battles — Pure Arcade & Stage Block Puzzle Engine

[![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?logo=flutter)](https://flutter.dev)
[![Flame](https://img.shields.io/badge/Flame-Engine-FF6F00?logo=flame)](https://flame-engine.org)
[![Platform](https://img.shields.io/badge/Platform-Android%20%7C%20Touch-brightgreen)](https://developer.android.com)
[![Architecture](https://img.shields.io/badge/Architecture-Clean%20Domain%20%2B%20Event%20Bus-blueviolet)](file:///18_TECHNICAL_ARCHITECTURE.md)
[![License](https://img.shields.io/badge/License-Proprietary-red)](#)

> **Pure Spatial Mastery. Tactile Arcade Feel. Zero "AI Slop".**  
> Block Battles is a premium mobile spatial block puzzle game built with Flutter + Flame + Dart for mobile devices. It delivers an addictive, highly responsive, tactile arcade loop focused on zen spatial placement, explosive line clears, and curated stage challenges.

---

## 1. Project Vision & Core Pillars

Block Battles is engineered from the ground up as a pure, premium block puzzle experience:

1. **Tactile Juice from Day One:** Blocks are physical gemstones with crisp top-highlight bevels, vibrant color depth, and ambient drop shadows.
2. **Ergonomic Finger-Lift Offset:** Dragged pieces float **48dp above the player's touch point**, ensuring the target board cells are always 100% visible.
3. **Escalating Feedback:** Ascending chime arpeggios, screen shake, and radiant particle explosions celebrate multi-line clears and consecutive streaks.
4. **Local Self-Competition:** 100% offline gameplay competing against the device's all-time High Score record with instant one-tap replay.

---

## 2. The Two Game Modes

```text
                                  [ APP LAUNCH ]
                                        ↓
                         [ ANIMATED SPLASH SCREEN (1.2s) ]
                                        ↓
                           [ MINIMALIST HOME SCREEN ]
                                  │           │
                ┌─────────────────┘           └─────────────────┐
                ▼                                               ▼
     [ MODE 1: CLASSIC ENDLESS ]                    [ MODE 2: STAGE CHALLENGE ]
        (Continuous Loop)                            (Curated Level Puzzles)
   • Endless spatial puzzle                        • Level-based stages & star ratings
   • 8×8 grid + 3-piece replenishing tray          • Target Score hurdles (e.g. 3,500 pts)
   • Consecutive streak multipliers (1.0x-2.5x)    • Obstacle clears (Stone / Ice blocks)
   • Local High Score competition                  • Move limits (turn countdown budget)
   • Board Lock failure condition                  • Pure declarative StageDefinition data
```

### Mode 1: Classic Endless Mode
- **Grid:** Standard 8×8 grid (64 cells).
- **Tray:** Exactly 3 pieces per batch; fully replenishes only when all 3 slots are consumed.
- **Scoring Pipeline:**
  $$\text{TurnScore} = \text{PlacementPoints} + \left( \text{BaseLineClearPoints}(L) \times \text{StreakMultiplier} \right)$$
  - *Placement Points:* $1 \text{ pt} \times \text{blockCount}$.
  - *Multi-Line Bonuses ($L$ simultaneous lines):* $L1 = 100$, $L2 = 300$, $L3 = 600$, $L4 = 1,000$.
  - *Consecutive Streak Multipliers:* Turn 1 clear = $1.0\times$, Turn 2 consecutive clear = $1.5\times$, Turn 3 consecutive clear = $2.0\times$, Turn 4+ consecutive clear = $2.5\times$ (cap).
  - *Board Wipe:* Clearing all 64 cells awards a $+500$ flat jackpot with golden fanfare.
- **Game Over:** Board Lock triggers the instant zero valid placements exist for all remaining tray pieces.

### Mode 2: Goal-Driven Stage Mode
- Curated progression of puzzle stages with explicit completion hurdles.
- Objective types:
  - Reach target score within move limits (e.g., 3,500 points in 20 moves).
  - Clear specific block colors (e.g., collect 16 Amber blocks).
  - Clear board obstacles (stone blocks, multi-hit ice tiles).
- Earn 1 to 3 stars per stage based on move efficiency and bonus score.

---

## 3. The 5 Streamlined Master Phases

The roadmap executes in **5 Master Phases** prioritizing visual polish and tactile game feel:

```text
┌────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: Pure Engine Foundations (Board, Shapes, RNG & Tray)           │
│ • Pure Dart 8x8 BoardEngine, 34 Shapes, GameRNG, TrayEngine (Tests)    │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│ PHASE 2: Tactile Visuals & Drag-and-Drop UX ("Killing AI Slop" Part 1) │
│ • Flame canvas, beveled gem materials, 48dp finger lift, ghost snap    │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│ PHASE 3: Explosive Line Clearing & Game Feel ("Killing AI Slop" Part 2)│
│ • Simultaneous clears, white-hot flash, particle bursts, pitch chimes  │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│ PHASE 4: Classic Endless Mode, Score Engine & Local High Score         │
│ • Scoring rules, streak multipliers, Board Lock detection, Game Over   │
└──────────────────────────────────┬─────────────────────────────────────┘
                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│ PHASE 5: App Shell, Animated Splash, Clean Home & Stage Mode Hooks     │
│ • Splash screen, minimalist home menu, settings sheet, StageDefinition │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Technical Architecture

```text
[ FLUTTER APP SHELL & NAVIGATION ]
   ├── SplashScreen (Snappy branded animation)
   ├── HomeScreen (Minimalist: Mode Selection, High Score, Settings)
   ├── ClassicGameScreen & StageGameScreen (Host GameWidget and HUD overlays)
   └── Overlays: GameOverOverlay, StageClearOverlay, PauseOverlay, SettingsSheet

[ FLAME PRESENTATION LAYER (Game Engine & Feel) ]
   ├── BlockBattlesGame (Flame-Flutter bridge)
   ├── BoardComponent & CellComponent (8x8 visual grid, beveled gemstones)
   ├── TrayComponent & PieceComponent (Interactive shapes, 48dp lift offset)
   ├── GhostPreviewComponent (Snaps to grid with valid green / invalid red)
   ├── VFX & Animation (LineClearBurst, WhiteFlashEffect, ScreenShake)
   └── AudioManager (Pitch escalation chimes, tactile haptics)

[ APPLICATION LAYER (Dart Services & Orchestration) ]
   ├── ClassicGameService (Mode 1 turn loop orchestrator)
   ├── StageGameService (Mode 2 stage progression orchestrator)
   ├── SaveRepository (Local persistence via Hive CE)
   └── SettingsService (Audio & haptic preferences via SharedPreferences)

[ DOMAIN LAYER (Pure Dart — Zero Flutter/Flame Imports) ]
   ├── BoardEngine (8×8 integer array math, placement commits)
   ├── LineScanner (Simultaneous row/col checks)
   ├── ShapeCatalog & ShapeDefinition (34 canonical shape geometries)
   ├── TrayEngine (3-piece batch generation & solvability checks)
   ├── ScoreService (Placement points, line clear points, streak multipliers)
   ├── StageObjectiveEngine (Goal progress evaluation)
   └── GameRNG (Deterministic seeded pseudo-random generator)
```

---

## 5. Repository & Documentation Structure

All source-of-truth documents are located in this repository (`E:\Block Buster`), strictly organized by architectural layers:

```text
E:\Block Buster\
├── README.md                                          ← Project Overview (this file)
├── AI_START_HERE.md                                   ← Mandatory orientation for AI agents
├── MASTER_DOCUMENT_INDEX.md                           ← Complete navigation & dependency map
│
├── [LAYER A — GAME DESIGN SPECIFICATIONS]
│   ├── 00_MASTER_GAME_VISION.md                       ← Product definition, pillars, two modes
│   ├── 01_GAMEPLAY_SPECIFICATION.md                   ← Turn lifecycle, board rules, board lock
│   ├── 02_SHAPE_LIBRARY.md                            ← 34 canonical shape matrices & tags
│   ├── 03_BOARD_ENGINE_AND_RULES.md                   ← 8×8 grid math, cell predicates
│   ├── 04_SPAWN_RNG_AND_DIFFICULTY.md                 ← Seeded RNG, piece generation weighting
│   ├── 05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md     ← Pure scoring formulas & high score persistence
│   ├── 06_INPUT_DRAG_AND_DROP_UX.md                   ← Touch gestures, 48dp lift, ghost preview
│   └── 07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md       ← Mode 2: Stage Objectives & Level Design
│
├── [LAYER B — PRESENTATION & UX DESIGN]
│   ├── 08_UI_UX_MASTER_SPECIFICATION.md               ← Screen flows: Splash, Home, HUD, Game Over
│   ├── 09_VISUAL_DESIGN_SYSTEM.md                     ← Typography, spacing, container styling
│   ├── 10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md     ← Gemstone color palette, contrast ratios
│   ├── 11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md       ← Bevel highlights, glass sheen, shadows
│   ├── 12_ANIMATION_MOTION_AND_GAME_FEEL.md           ← Spring curves, drag squash, snap timings
│   ├── 13_VFX_PARTICLE_AND_IMPACT_SPEC.md             ← Line clear flash, particle burst physics
│   ├── 14_AUDIO_SFX_MUSIC_AND_MIXING.md               ← Chimes, pitch escalation, haptic patterns
│   ├── 15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md    ← Clean UI strings & microcopy
│   ├── 16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md ← Haptics, audio, reduced motion
│   └── 17_MONETIZATION_REWARD_AND_AD_UX.md            ← Ethical non-intrusive monetization principles
│
├── [LAYER C — TECHNICAL & ENGINEERING ARCHITECTURE]
│   ├── 18_TECHNICAL_ARCHITECTURE.md                   ← Pure Dart Domain, Flame Presentation, Flutter UI
│   ├── 19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md ← File layout, routing
│   ├── 20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md ← Dart data models & schemas
│   ├── 21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md ← Service locator & Riverpod
│   ├── 22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md   ← Strongly-typed domain events
│   ├── 23_UI_IMPLEMENTATION_ARCHITECTURE.md           ← Widget tree & Flame boundary
│   ├── 24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md     ← Local save schemas & Hive CE
│   ├── 25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md ← SFX, font, and icon pipelines
│   ├── 26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md ← 60fps budget, zero-allocation loops
│   ├── 27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md      ← Unit, widget, and integration testing
│   └── 28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md ← Android APK / App Bundle config
│
└── [LAYER D — EXECUTION & AGENT DIRECTIVES]
    ├── 29_AI_CODING_AGENT_DEVELOPMENT_RULES.md        ← Strict AI development laws & hierarchy
    └── 30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md ← 5 Master Phases & execution order
```

---

## 6. Tech Stack & Platform Matrix

| Category | Specification |
|---|---|
| **Language** | Dart 3.x (Sound null-safety, strict analysis) |
| **Framework** | Flutter 3.x (Material 3, dark slate baseline) |
| **Game Engine** | Flame 1.x (Component tree, drag callbacks, particle systems) |
| **State Management** | Riverpod |
| **Persistence** | Hive CE (Game Progress & Sessions) + SharedPreferences (Settings) — 100% offline |
| **Audio** | Flame Audio (Sound pool, pitch-shifting clear chimes) |
| **Target Platform** | Android 8.0+ (API 26+) / iOS — Portrait orientation, 60fps locked |
| **Primary Code Workspace** | `E:\BLOCK BATTLES DEV` |
| **Documentation Workspace**| `E:\Block Buster` (`https://github.com/kmkbuild/Block-Battles.git`) |

---

## 7. Verification & Quick Commands

To test and verify the project:

```bash
# Analyze code health (must remain at 0 issues)
flutter analyze

# Run pure Dart domain unit tests
dart test

# Run Flutter UI & integration tests
flutter test

# Build debug APK for phone testing
flutter build apk --debug
```

---

*Authored by the Block Battles Engineering & Design Team.*
