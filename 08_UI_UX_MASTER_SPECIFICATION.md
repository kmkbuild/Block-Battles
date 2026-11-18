# 08_UI_UX_MASTER_SPECIFICATION.md
## Block Battles — UI/UX Master Specification (Pure Arcade Experience)

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `06_INPUT_DRAG_AND_DROP_UX.md`  
**Document status:** Player-Facing Presentation & Navigation Architecture  
**Core Authority:** Minimalist, clutter-free UI flow, instant playability, tactile game feel, zero "AI slop".

---

## 1. UX Vision & Guiding Principles

1. **Instant Clarity & Zen Focus:** No cluttered side-menus, no forced tutorials, no distracting non-puzzle overlays. The player opens the app and plays.
2. **Tactile Feedback & Juice:** Every interaction (touch, drag, hover, drop, line clear) provides crisp haptic, audio, and visual feedback.
3. **Ergonomic Finger-Lift Offset:** Pieces lifted from the tray appear with a 48dp vertical offset above the finger so the player's thumb never obstructs the placement target.
4. **Local Competition:** The player always sees their all-time High Score on the Home screen and In-Game HUD, fostering satisfying self-improvement.

---

## 2. Global Navigation Flow

```text
APP LAUNCH
    ↓
SPLASH SCREEN (Smooth branded logo reveal, 1.2s auto-transition)
    ↓
MINIMALIST HOME SCREEN
    ├── [ PLAY CLASSIC ]  (Primary Action) ──────────┐
    ├── High Score Trophy Card (Local Best)          │
    ├── [ GOAL MODE ] Portal (Reserved Mode 2)       │
    └── Settings Icon (Audio, Haptics, Theme)        │
                                                     ▼
                                            CLASSIC IN-GAME HUD
                                            (Top Bar + 8x8 Board + Tray)
                                                     │
                                            Board Lock Detected
                                                     │
                                                     ▼
                                            GAME OVER DIALOG
                                            ├── Final Score & Best Score
                                            ├── "NEW BEST!" Fanfare (if beaten)
                                            ├── [ PLAY AGAIN ] (Instant restart)
                                            └── [ HOME ] (Return to Home)
```

---

## 3. Screen Inventory & Specifications

### 3.1 Animated Splash Screen
- **Duration:** 1.2s to 1.5s max, or tap to skip.
- **Visuals:** Deep slate background (`#0F172A`), animated geometric block assembly forming the "BLOCK BATTLES" monogram, accompanied by a subtle ascending chime.
- **Auto-transition:** Seamlessly fades into the Home Screen.

### 3.2 Minimalist Home Screen
- **Layout:** Centered, generous white-space, premium dark aesthetic.
- **Header:** Clean typographic logo with subtle neon glow.
- **High Score Display:** Elegant golden trophy card showing `BEST: 12,450`.
- **Primary CTA:** Large, glowing, rounded [ PLAY CLASSIC ] button with tactile press animation.
- **Secondary Card:** [ GOAL-DRIVEN MODE ] (Stage hurdles & stars — labeled "Stage Mode").
- **Top-Right Actions:** Settings cog icon (opens audio/haptics sheet).

### 3.3 Classic In-Game HUD
- **Top App Bar (Height: 56dp):**
  - Left: Pause button (opens `PauseOverlay`).
  - Center: Current Running Score (`1,420`) in large, bold numbers.
  - Right: Best Score with miniature trophy icon (`BEST: 12,450`).
- **Streak / Combo Badge:**
  - Floats subtly beneath the score bar when streak $\ge 1$ (`STREAK x1.5!`, `MEGA STREAK x2.5!`).
- **Center Canvas:**
  - 8×8 Board inside a rounded slate container with subtle inner shadows.
  - Live ghost preview on drag (soft green translucent glow for valid drops; soft red for invalid drops).
- **Bottom Tray Surface:**
  - Clean horizontal container hosting up to 3 shape previews.
  - Fixed aspect ratio preview containers with generous spacing.

### 3.4 Game Over Overlay (GameOverOverlay)
- **Trigger:** Transition to `BOARD_LOCK_GAME_OVER` when zero legal moves exist (Mode 1 Classic).
- **Content:**
  - Title: "BOARD LOCKED" / "GAME OVER".
  - Final Score counter with count-up animation.
  - High Score comparison:
    - If new high score: Golden particle confetti, celebratory fanfare, and glowing "NEW RECORD!" ribbon.
    - If below high score: Difference displayed ("340 pts behind best").
  - Statistics: Total lines cleared, Max streak achieved.
- **Actions:**
  - [ PLAY AGAIN ] (High-contrast button, restarts board and tray instantly).
  - [ HOME ] (Secondary button, returns to Home Screen).

### 3.5 Pause Overlay (PauseOverlay)
- **Trigger:** Tapping pause icon on top bar (Both modes).
- **Options:**
  - [ RESUME ]
  - [ RESTART ] (Prompts confirmation to prevent accidental loss)
  - [ SETTINGS ] (Quick audio & haptics sliders)
  - [ QUIT TO HOME ]

### 3.6 Settings Screen (SettingsScreen)
- **Route:** `/settings`
- **Controls:**
  - Sound Effects (SFX) Volume Slider / Toggle.
  - Music Volume Slider / Toggle.
  - Haptic Feedback Toggle (On/Off).
  - Reduced Motion Toggle (Disables camera shake for accessibility).

### 3.7 Stage Clear Overlay (StageClearOverlay)
- **Trigger:** All stage goals met in Mode 2.
- **Content:**
  - Title: "STAGE CLEAR!".
  - Star Rating: 1, 2, or 3 animated stars bursting in with sparkling chimes.
  - Score Breakdown: Base score + Remaining moves bonus.
- **Actions:**
  - [ NEXT STAGE ] (Advances to next level).
  - [ REPLAY ] (Try for 3 stars).
  - [ STAGES ] (Returns to Stage Select grid).

### 3.8 Stage Failed Overlay (StageFailedOverlay)
- **Trigger:** Remaining moves reach 0 before completing objectives (Mode 2).
- **Content:**
  - Title: "OUT OF MOVES".
  - Unmet objective tally.
- **Actions:**
  - [ TRY AGAIN ] (Restarts current stage).
  - [ STAGES ] (Returns to Stage Select grid).

---

## 4. Design Guidelines to Kill "AI Slop"

1. **Depth & Tactility:** Blocks must never look like flat colored rectangles. They require subtle 1px top-highlight bevels, soft ambient drop shadows, and responsive squash-and-stretch on pickup.
2. **Intentional Spacing:** Maintain generous margins (16dp-24dp) so UI never feels crowded or cheap.
3. **Color Harmony:** Strict palette using deep navy/slate backgrounds (`#0B0F19`, `#0F172A`), electric gemstone accents for blocks (Emerald, Sapphire, Amber, Ruby, Amethyst, Coral), and crisp white typography.
4. **Responsive Animations:** Buttons compress with spring physics on touch (`Curves.easeOutBack`), and ghost previews snap crisply to grid cells.

---

## 5. Acceptance Criteria

- Splash screen smoothly transitions to Home within 1.5 seconds.
- Home screen clearly displays local High Score and prominent Play button.
- In-Game HUD displays Score and Best Score with real-time updates.
- Ghost preview snaps accurately under drag with finger offset.
- GameOverOverlay opens promptly on Board Lock with clear Play Again and Home actions.
- Zero clutter: No non-puzzle banners, no complex sub-menus, no distracting overlays.

---

**End of `08_UI_UX_MASTER_SPECIFICATION.md`.**
