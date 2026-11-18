# Block Battles — Pure Arcade Block Puzzle Engine

[![Flutter](https://img.shields.io/badge/Flutter-3.x-02569B?logo=flutter)](https://flutter.dev)
[![Flame](https://img.shields.io/badge/Flame-Engine-FF6F00?logo=flame)](https://flame-engine.org)
[![Platform](https://img.shields.io/badge/Platform-Android%20%7C%20Touch-brightgreen)](https://developer.android.com)
[![Architecture](https://img.shields.io/badge/Architecture-Clean%20Domain%20%2B%20Event%20Bus-blueviolet)](file:///18_TECHNICAL_ARCHITECTURE.md)
[![License](https://img.shields.io/badge/License-Proprietary-red)](#)

> **Pure Arcade Spatial Mastery. Tactile Game Feel. Zero "AI Slop".**  
> Block Battles is a premium mobile spatial block puzzle game built with Flutter + Flame + Dart for Android. It strips away all generic RPG bloat to deliver an addictive, highly responsive, tactile arcade loop focused on zen spatial mastery, explosive line clears, and local high score self-competition.

---

## 1. Project Vision & The Strategic Pivot

### Why We Pivoted Away from RPG Combat
Previous prototypes focused heavily on enemy combat, turn telegraphs, and relic drafting before basic board aesthetics and game feel were locked in. The result felt like generic, cluttered "AI slop" on a real device.

We fundamentally pivoted to a **pure, tactile arcade puzzle**:
1. **Zero Combat Baggage:** All enemy HP, goblin hunting, damage calculation pipelines, and relic drafts are completely removed.
2. **Tactile Juice from Day One:** Blocks are glossy, physical gemstones with 1px top-highlight bevels and ambient drop shadows—never flat, boring rectangles.
3. **Ergonomic Finger-Lift Offset:** Dragged pieces float **48dp above the player's thumb**, ensuring the board target is always 100% visible.
4. **Local Self-Competition:** 100% offline gameplay competing against the device's all-time High Score record.

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
     [ MODE 1: CLASSIC ENDLESS ]                    [ MODE 2: GOAL-DRIVEN ]
       (Immediate MVP Focus)                         (Reserved Architecture)
  • Endless spatial puzzle                        • Level-based stages & star ratings
  • 8×8 grid + 3-piece replenishing tray          • Target Score hurdles (e.g. 3,000 pts)
  • Consecutive combo streak multipliers          • Obstacle clears (Frozen/Blocked cells)
  • Local High Score competition                  • Move limits (countdown cap)
  • Board Lock failure condition                  • Decoupled LevelDefinition contracts
```

### Mode 1: Classic Endless Mode (Core Focus)
- **Grid:** Standard 8×8 grid.
- **Tray:** Exactly 3 pieces per batch; fully replenishes only when all 3 slots are consumed.
- **Scoring Pipeline:**
  $$\text{TurnScore} = \text{PlacementPoints} + \left( \text{BaseLineClearPoints}(L) \times \text{StreakMultiplier} \right)$$
  - *Placement Points:* $1 \text{ pt} \times \text{blockCount}$.
  - *Multi-Line Bonuses ($L$ simultaneous lines):* $L1 = 100$, $L2 = 300$, $L3 = 600$, $L4 = 1,000$, $L5+ = 1,000 + (L-4) \times 500$.
  - *Consecutive Streak Multipliers:* $0 \text{ streak} = 1.0\times$, $1 \text{ streak} = 1.5\times$, $2 \text{ streak} = 2.0\times$, $3+ \text{ streak} = 2.5\times$ (cap).
  - *Board Wipe:* Clearing all 64 cells awards a $+500$ flat jackpot with golden fanfare.
- **Game Over:** Board Lock triggers the instant zero valid placements exist for all remaining tray pieces.

### Mode 2: Goal-Driven Mode (Reserved Architecture)
- Architecturally reserved via `LevelDefinition` and `GoalDrivenGameService`.
- Challenges players with pre-set obstacle boards (Frozen and Blocked cells) and strict move limits.
- Kept strictly isolated so Classic Mode remains lean and pure.

---

## 3. The 5 Streamlined Master Phases

The roadmap has been condensed from 22 dragged-out steps into **5 Master Phases** with visual polish and tactile game feel prioritized early:

```text
┌────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: Pure Engine Foundations (Board, Shapes, RNG & Tray)           │
│ • Pure Dart 8x8 BoardModel, ShapeCatalog, GameRng, TrayEngine (Tests)  │
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
│ PHASE 5: App Shell, Animated Splash, Clean Home & Goal Mode Hooks      │
│ • Splash screen, minimalist home menu, settings sheet, LevelDefinition │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Technical Architecture

```text
[ FLUTTER APP SHELL & NAVIGATION ]
   ├── SplashScreen (1.2s logo assembly)
   ├── HomeScreen (Minimalist: Play Classic, Best Score Trophy, Settings)
   ├── GameView (Hosts GameWidget and HUD overlays)
   └── Overlays: GameOverDialog, PauseModal, SettingsSheet

[ FLAME PRESENTATION LAYER (Game Engine & Feel) ]
   ├── BlockBattlesFlameGame (Flame-Flutter bridge)
   ├── BoardComponent & CellComponent (8x8 visual grid, 1px highlight bevels)
   ├── TrayComponent & PieceComponent (Interactive shapes, 48dp lift offset)
   ├── GhostPreviewComponent (Snaps to grid with valid green / invalid red)
   ├── VFX & Animation (LineClearBurst, WhiteFlashEffect, CameraShake)
   └── AudioManager (Pitch escalation chimes, tactile haptics)

[ APPLICATION LAYER (Dart Services & Orchestration) ]
   ├── ClassicGameService (Turn loop orchestrator)
   ├── HighScoreService (Local persistence via SharedPreferences / Hive CE)
   ├── SettingsService (Audio & haptics preferences)
   └── GoalDrivenGameService (Reserved Mode 2 orchestrator)

[ DOMAIN LAYER (Pure Dart — Zero Flutter/Flame Imports) ]
   ├── BoardModel (8×8 integer array math, placement commits)
   ├── LineClearDetector (Simultaneous row/col checks)
   ├── ShapeCatalog & ShapeDefinition (Immutable piece geometries)
   ├── TrayEngine (3-piece batch generation & solvability checks)
   ├── ScoreEngine (Placement points, line clear points, streak multipliers)
   ├── BoardLockDetector (Exhaustive legal-move search)
   ├── GameRng (Deterministic seeded pseudo-random generator)
   └── LevelDefinition (Mode 2 hurdle schemas)
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
│   ├── 02_SHAPE_LIBRARY.md                            ← 24 canonical shape matrices & tags
│   ├── 03_BOARD_ENGINE_AND_RULES.md                   ← 8×8 grid math, cell predicates
│   ├── 04_SPAWN_RNG_AND_DIFFICULTY.md                 ← Seeded RNG, piece generation weighting
│   ├── 05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md     ← Pure scoring formulas & high score persistence
│   ├── 06_INPUT_DRAG_AND_DROP_UX.md                   ← Touch gestures, 48dp lift, ghost preview
│   └── 07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md       ← Mode 2: Goal-Driven Mode & Level Hurdles
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
│   ├── 20_DATA_MODEL_CONTENT_DEFINITIONS_AND_RUNTIME_SCHEMA.md ← Dart data models
│   ├── 21_SYSTEM_MANAGERS_SERVICES_AND_DEPENDENCY_ARCHITECTURE.md ← Service locator & Riverpod
│   ├── 22_EVENT_BUS_AND_GAME_STATE_ORCHESTRATION.md   ← Strongly-typed domain events
│   ├── 23_UI_IMPLEMENTATION_ARCHITECTURE.md           ← Widget tree & Flame boundary
│   ├── 24_SAVE_DATA_PERSISTENCE_AND_VERSIONING.md     ← Local save schemas
│   ├── 25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md ← SFX, font, and icon pipelines
│   ├── 26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md ← 60fps budget, zero-allocation loops
│   ├── 27_TESTING_QA_DEBUGGING_AND_AUTOMATION.md      ← Unit, widget, and integration testing
│   └── 28_BUILD_CONFIGURATION_PLATFORM_AND_RELEASE.md ← Android APK / App Bundle config
│
├── [LAYER D — EXECUTION & AGENT DIRECTIVES]
│   ├── 29_AI_CODING_AGENT_DEVELOPMENT_RULES.md        ← Strict AI development laws & hierarchy
│   └── 30_IMPLEMENTATION_ROADMAP_AND_EXECUTION_ORDER.md ← 5 Master Phases & execution order
│
└── .planning/
    └── sketches/
        ├── themes/default.css                         ← Shared dark arcade styling tokens
        ├── MANIFEST.md                                ← Design exploration registry
        └── 001-app-icon/                              ← UI 1 App Icon interactive prototype & phone preview
```

---

## 6. Live UI Sketches & Figma MCP Integration

We use a **UI-First** development process:
- **Interactive HTML Prototypes:** Testable in any browser at `.planning/sketches/` (e.g. [001-app-icon/index.html](file:///.planning/sketches/001-app-icon/index.html) with a live mobile home screen simulator).
- **Figma MCP Connected:** Direct live canvas synchronization with Figma via Model Context Protocol (`mcp.figma.com/mcp`), enabling the AI agent to draw and update frames directly on your Figma canvas.

---

## 7. Tech Stack & Platform Matrix

| Category | Specification |
|---|---|
| **Language** | Dart 3.x (Sound null-safety, strict analysis) |
| **Framework** | Flutter 3.x (Material 3, dark slate baseline) |
| **Game Engine** | Flame 1.x (Component tree, drag callbacks, particle systems) |
| **State Management** | Riverpod |
| **Persistence** | SharedPreferences / Hive CE (100% offline, local high scores) |
| **Audio** | Flame Audio (Sound pool, pitch-shifting clear chimes) |
| **Target Platform** | Android 8.0+ (API 26+) — Portrait orientation, 60fps locked |
| **Primary Code Repo** | `E:\BLOCK BUSTER DEV` |
| **Documentation Repo** | `https://github.com/kmkbuild/Block-Battles.git` (`E:\Block Buster`) |

---

## 8. Verification & Quick Commands

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
