# 15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md
## Block Battles — Screen Content and Player-Facing Copy

**Governing documents:** `08_UI_UX_MASTER_SPECIFICATION.md`, `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `09_VISUAL_DESIGN_SYSTEM.md`, `00_MASTER_GAME_VISION.md`
**Document status:** Layer B — Content & Copy Authority

**Inheritance confirmation:** This document defines the exact string values for the UI wireframes defined in Doc 08. It applies the game mechanics and scoring rules from Docs 05 and 07 into human-readable text. It adheres to the localization constraints required for a global mobile release. It introduces no new features; it only explains existing ones.

---

## 1. Voice and Tone

### 1.1 Personality
The game's voice is **"The Confident Croupier."** It is a neutral, highly competent facilitator. It explains the rules clearly, rewards success enthusiastically, and reports failure objectively. It does not judge, it does not mock, and it does not waste the player's time.

### 1.2 Tone Guidelines
- **Confidence:** Speak in active voice. Say "Clear 5 lines" not "You need to try to clear 5 lines."
- **Humor Level:** Low. Puns, memes, and meta-jokes distract from puzzle-solving. The game takes its mechanics seriously.
- **Seriousness:** High for mechanics, medium for lore. Flavor text exists to set a mood, not to write a novel.
- **Feedback Tone:** Punchy and dynamic. (e.g., "Quad Clear!" or "STREAK x2.5!" instead of "Excellent line clearance.")
- **Reward Tone:** Triumphant but brief.
- **Failure Tone:** Objective and clinical. A puzzle was unsolved; a tragedy did not occur. (e.g., "Board Lock" or "Move Limit Reached," never "You Died" or "Wasted.")

---

## 2. Terminology Rules

To prevent player confusion, terminology must be absolutely rigid.

| Concept | Canonical Term | Forbidden Terms | Reason |
|---|---|---|---|
| A single stage (Mode 2) | **Stage** | Fight, Match, Quest | Clean puzzle taxonomy. |
| The 8x8 play area | **Board** | Grid, Field, Map, Table | Simplest noun. |
| The 3 available items | **Tray** | Deck, Hand, Queue, Bench | Fits the physical board game aesthetic. |
| A draggable item | **Piece** | Block, Shape, Tetromino | "Block" refers to the 1x1 sub-units. |
| The 1x1 grid unit | **Cell / Block**| Square, Tile, Pixel | "Block" is the material, "Cell" is the space. |
| Losing by running out of space | **Board Lock** | Stuck, Blocked, Game Over | Specific mechanical failure state. |
| Losing by running out of turns| **Out of Moves** | Time Up, Too Slow | Accurate description of the limit. |
| Full run from start to finish | **Run** | Campaign, Journey, Arcade | Standard roguelite terminology. |
| Consecutive line clears | **Streak** | Combo, Chain, Series | Clear arcade terminology. |

---

## 3. Home Screen Copy

| Element | Copy |
|---|---|
| **Game Title** | Block Battles |
| **Primary Action (Mode 1)** | Play Classic |
| **Secondary Action (Mode 2)**| Adventure / Goals |
| **High Score Display** | Best: {0} |
| **Navigation Button** | Settings |
| **Navigation Button** | How to Play |

---

## 4. Game Flow & Dialog Copy

| Context | Copy | Note |
|---|---|---|
| **Quit Confirmation Title** | Return to Menu? |
| **Quit Confirmation Body** | Your current round will end and your score will be saved. | Neutral, factual consequence. |
| **Confirm Action** | End Game | Neutral danger register. |
| **Cancel Action** | Resume | Returns to active board. |
| **Level Complete Action** | Next Level | Advances to next stage in Mode 2. |

---

## 5. Gameplay HUD Copy

| Element | Copy | Note |
|---|---|---|
| **Current Score** | {0} | Primary top anchor. Always tabular. |
| **High Score Indicator** | Trophy: {0} | Mode 1 device record. |
| **Streak Multiplier Badge** | STREAK x{0}! | Dynamic capsule pill (e.g. STREAK x1.5!). |
| **Primary Objective (Mode 2)**| [Objective Title] | Inherits from Section 9. |
| **Objective Progress** | {0} / {1} | e.g., 12 / 20 Blue Blocks. |
| **Move Limit (Mode 2)** | {0} Moves Left | Alert state triggers when <= 5 moves. |

---

## 6. Tutorial & Onboarding Copy

Tutorial copy must be the shortest in the game. It uses high-contrast tooltips pointing directly to the UI.

| Step | Trigger | Copy |
|---|---|---|
| **1. Placement** | First game starts | **Drag a piece to the board.** |
| **2. Clearing** | Piece placed | **Fill a complete row or column to clear.** |
| **3. Streaks** | First clear made | **Clear lines on consecutive turns for score multipliers!** |
| **4. Planning** | Board fills | **Keep space open for larger shapes.** |

---

## 7. Mode 2 Objective Types & Copy

### 7.1 Objective Card Formatting Rules
- **Syntax:** `[Action Verb] [Target Count] [Item]`
- **Numbers:** Always use digits (15, not fifteen).
- **Tone:** Direct, clear, measurable.

### 7.2 Canonical Stage Objective Formats

| ID | Title | Description | HUD Progress |
|---|---|---|---|
| `OBJ-001` | Target Score | Reach {0} points before running out of moves. | {0} / {1} Pts |
| `OBJ-002` | Line Count | Clear {0} total lines (rows or columns). | {0} / {1} Lines |
| `OBJ-003` | Color Clearance | Clear {0} {Color} blocks from the board. | {0} / {1} Blocks |
| `OBJ-004` | Break Ice | Clear all {0} frozen obstacle tiles. | {0} / {1} Ice |
| `OBJ-005` | Multi-Clear | Perform {0} double or triple line clears. | {0} / {1} Clears |

---

## 8. Mode Descriptions (Home / Mode Select)

| Mode | Title | Description |
|---|---|---|
| **Mode 1** | Classic Endless | Pure block puzzle survival. Place shapes, clear lines, build streaks, and set the all-time record. |
| **Mode 2** | Goal Challenge | Curated puzzle levels. Complete color goals, clear obstacles, and beat move limits. |

---

## 9. Objective Text (HUD Labels)

| Objective Type | HUD Short Title | HUD Progress | Expanded Description |
|---|---|---|---|
| Target Score | Reach Score | {0} / {1} Pts | Reach the required score threshold. |
| Line Count | Clear Lines | {0} / {1} Lines | Clear the specified number of rows or columns. |
| Color Clearance| Clear {Color} | {0} / {1} | Clear blocks of the specified color identity. |
| Ice Hurdle | Break Ice | {0} / {1} Ice | Clear lines adjacent to ice tiles to melt them. |
| Move Limit | Moves Left | {0} Moves | Complete objectives before moves expire. |

---

## 10. Victory & Level Clear Copy

| Context | Copy |
|---|---|
| **Level Clear Header** | Stage Complete! |
| **Level Clear Sub** | All objectives achieved. |
| **Star Rating** | {0} of 3 Stars |
| **New Record Header** | New Best Score! |
| **New Record Sub** | All-time device high score surpassed! |

---

## 11. Bonus Objective Copy

Bonus objectives use an identical structure to Primary objectives, but are prefaced with "Bonus" and offer a specific currency/score reward.

- **State: Active:** `Bonus: Clear 3 lines simultaneously.`
- **State: Complete:** `Bonus Complete! (+50 Score)`

---

## 12. Run Defeat / Game Over Copy

There are two primary failure states. The copy must distinguish them.

| Failure State | Header | Subtext | Action |
|---|---|---|---|
| **Board Lock** | Board Lock | No valid placements remain. | End Run |
| **Move Limit** | Out of Moves | You did not complete the objective in time. | End Run |

*Rule:* Never use "You Died" or "Defeated." The player did not die; the puzzle state became unsolvable. Maintain the "Confident Croupier" tone.

---

## 13. Revive / Rewarded Ad Copy (MVP+ Rules)

**BINDING DECISION:** As per `00_MASTER_GAME_VISION.md` Section 5.7 and `08` Section 21, Block Battles at Absolute MVP has **NO monetization surface, NO ads, and NO revive mechanics.**

**Future-Proofing Contract (If added post-MVP):**
If a rewarded ad revive is added later, the copy must adhere to strict transparency rules to prevent dark patterns.

- **Required Copy:** `Watch Ad to Continue`
- **Forbidden Copy:** `Don't give up!`, `Save your run?`, `Revive?` (without mentioning the ad).
- **Refusal Copy:** `End Run`
- **Forbidden Refusal Copy:** `No, I want to lose`, `I give up`, `Lose all progress`.

---

## 14. Settings Copy

| Label | Description / Subtext (if any) | Values / Toggles |
|---|---|---|
| Master Volume | Controls overall game volume. | Slider (0-100%) |
| Music | Controls background music volume. | Slider (0-100%) |
| Sound Effects | Controls UI and puzzle sounds. | Slider (0-100%) |
| Haptics | Device vibration feedback. | On / Off |
| Reduced Motion | Disables screen shakes and flashes. | On / Off |
| Language | Select game text language. | Dropdown |

---

## 15. Error Messages

Errors must explain *what happened* and *what the player should do next*.

| ID | Title | Body | Action |
|---|---|---|---|
| `err.save` | Save Failed | Could not save your run progress. Check your device storage. | Try Again |
| `err.load` | Load Failed | Could not resume the previous run. The save file may be corrupted. | Start New Run |
| `err.generic` | System Error | An unexpected error occurred. | Return to Menu |

---

## 16. Empty States

When a list or menu has no content, it must not be blank.

| Context | Empty State Copy |
|---|---|
| **Stage Select** | Complete earlier stages to unlock new challenges. |
| **High Scores** | Play a round to record your first score! |

---

## 17. Accessibility Labels (Screen Readers)

Crucial for visually impaired players using OS-level screen readers.

| UI Element | Semantic `aria-label` equivalent |
|---|---|
| **Pause Button** | "Pause the game" |
| **Settings Gear** | "Open Settings Menu" |
| **Score Display** | "Current Score: {0}" |
| **Streak Multiplier** | "Streak Multiplier: {0} times" |
| **Objective Chip** | "Objective: {0}. {1} remaining." |
| **Board Cell** | "Grid cell, column [X], row [Y]. [Empty / Filled with Blue Block]." |

---

## 18. Localization Readiness

### 18.1 Variable Interpolation
All dynamic numbers and strings must use index-based interpolation (`{0}`, `{1}`) to allow translators to change word order.
- **BAD:** `"Clear " + count + " more lines."` (Breaks in languages with different grammar structures).
- **GOOD:** `"Clear {0} more lines."` (Allows translation to `"{0} lines remain to be cleared."`).

### 18.2 Expansion Allowances
German and Russian routinely expand English copy by 30-40%.
- UI labels and buttons must have a **30% physical buffer** in the layout before text truncation occurs.
- Buttons should flex in width rather than remaining fixed size.

### 18.3 Number Formats
Do not hardcode commas for thousands. The UI layer must format `1000` as `1,000` or `1.000` based on device locale.

---

## 19. Copy Length Rules

Strict character counts prevent UI breakage. (Values include spaces).

| Element | Max Characters | Max Lines |
|---|---|---|
| **Primary Buttons** | 16 | 1 |
| **Secondary Buttons**| 24 | 1 |
| **Screen Titles** | 24 | 1 |
| **Objective Title** | 20 | 1 |
| **Objective Desc** | 80 | 2 |
| **Objective Title** | 20 | 1 |
| **Dialog Body** | 120 | 4 |

---

## 20. Content Registry (Master String List)

*Excerpt of the primary localization file structure (e.g., `en_US.json`). All code must reference the ID, never the raw string.*

| ID | String | Context |
|---|---|---|
| `ui.menu.title` | Block Battles | Main menu title |
| `ui.btn.start` | Start Run | Main menu primary button |
| `ui.btn.resume` | Continue Run | Main menu resume button |
| `ui.btn.settings`| Settings | Main menu secondary |
| `ui.btn.pause` | Pause | HUD button |
| `ui.btn.end` | End Game | Dialog confirmation |
| `ui.btn.cancel` | Cancel | Dialog cancellation |
| `hud.score` | Score: {0} | HUD score label |
| `hud.high_score` | Best: {0} | HUD high score record |
| `hud.streak` | STREAK x{0}! | HUD streak multiplier |
| `hud.moves` | {0} Moves Left | HUD limit indicator |
| `result.win.title`| Stage Complete! | Level complete screen |
| `result.new_best` | New Best Score! | Record broken banner |
| `result.lose.lock`| Board Lock | Game over screen (no moves) |
| `result.lose.turn`| Out of Moves | Stage failed screen (moves exhausted) |

---

## 21. Copy QA Checklist

Before any string freeze or build submission:
- [ ] **Variable check:** Do all strings containing `{0}` successfully receive their variables without rendering "undefined" or "[Object object]"?
- [ ] **Truncation check:** Set language to German (or a pseudo-localization script that adds +30% length). Do any buttons clip text? Do any dialog cards require scrolling?
- [ ] **Tone check:** Run a grep search for "die", "kill", "fail", "lose". Ensure none of these appear in player-facing UI unless explicitly authorized. (Use "defeat", "end", "board lock").
- [ ] **Case check:** Are all Buttons and Titles in Title Case? Is all body text in Sentence case?

---

## 22. Final Writing Contract

> **Respect the player's intelligence. Respect the player's time.**
>
> In Block Battles, language is an interface component, not a storytelling medium. If a word does not help the player make a decision, understand a rule, or feel a reward, it should be deleted.
>
> We use clear, punchy, active verbs. We do not use manipulative dark patterns to retain players. When they win, we congratulate them cleanly. When the board locks, we state it as a fact of the puzzle, not a personal failure.

---

**End of `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md`.**
