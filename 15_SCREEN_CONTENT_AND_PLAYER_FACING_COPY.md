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
- **Combat Tone:** Punchy and dynamic. (e.g., "Full Combo!" instead of "Excellent line clearance.")
- **Reward Tone:** Triumphant but brief.
- **Failure Tone:** Objective and clinical. A puzzle was unsolved; a tragedy did not occur. (e.g., "Board Lock" or "Move Limit Reached," never "You Died" or "Wasted.")

---

## 2. Terminology Rules

To prevent player confusion, terminology must be absolutely rigid.

| Concept | Canonical Term | Forbidden Terms | Reason |
|---|---|---|---|
| A single encounter | **Battle** | Fight, Level, Stage, Match | Consistency. |
| The 8x8 play area | **Board** | Grid, Field, Map, Table | Simplest noun. |
| The 3 available items | **Tray** | Deck, Hand, Queue, Bench | Fits the physical board game aesthetic. |
| A draggable item | **Piece** | Block, Shape, Tetromino | "Block" refers to the 1x1 sub-units. |
| The 1x1 grid unit | **Cell / Block**| Square, Tile, Pixel | "Block" is the material, "Cell" is the space. |
| Losing by running out of space | **Board Lock** | Stuck, Blocked, Game Over | Specific mechanical failure state. |
| Losing by running out of turns| **Out of Moves** | Time Up, Too Slow | Accurate description of the limit. |
| Full run from start to finish | **Run** | Campaign, Journey, Arcade | Standard roguelite terminology. |
| The passive modifiers | **Relics** | Items, Artifacts, Upgrades | Standardized in Doc 05. |

---

## 3. Home Screen Copy

| Element | Copy |
|---|---|
| **Game Title** | Block Battles |
| **Primary Action** | Start Run |
| **Resume Action** | Continue Run |
| **Secondary Navigation** | Relic Compendium |
| **Secondary Navigation** | Settings |
| **Run Status Meta** | Battle {0} of {1} |

---

## 4. New Run & Flow Control

| Context | Copy | Note |
|---|---|---|
| **Abandon Confirmation Title** | Abandon Run? |
| **Abandon Confirmation Body**| Current progress will be lost. You will return to the Home screen. | Neutral, factual consequence. |
| **Confirm Action** | End Run | Danger register color (`10_COLOR_SYSTEM`). |
| **Cancel Action** | Cancel |
| **Next Battle Transition** | Enter Battle | Button on Relic selection screen. |

---

## 5. Battle HUD

| Element | Copy | Note |
|---|---|---|
| **Enemy Title** | [Enemy Name] | Inherits from Section 8. |
| **Enemy HP** | {0} / {1} | e.g., 450 / 500. Always tabular. |
| **Primary Objective** | [Objective Name] | Inherits from Section 9. |
| **Primary Progress** | {0} / {1} | e.g., 2 / 5 Lines. |
| **Bonus Objective** | Bonus: [Condition] | Collapsed by default on small screens. |
| **Score / Currency** | {0} | Top-right indicator. |
| **Move Limit (if active)** | {0} Moves Left | Alert state triggers when <= 5. |

---

## 6. Tutorial Copy

Tutorial text must be the shortest copy in the game. It uses high-contrast tooltips pointing directly to the UI.

| Step | Trigger | Copy |
|---|---|---|
| **1. Movement** | First battle starts | **Drag a piece to the board.** |
| **2. Clearing** | Piece placed | **Fill a row or column to attack.** |
| **3. Enemy Intention**| Enemy turn starts | **Watch the enemy's next move.** |
| **4. Objectives** | First non-defeat objective | **Complete the objective to win.** |

---

## 7. Relic Cards

### 7.1 Strict Formatting Rules
- **Syntax:** `[Trigger condition]: [Effect]`
- **Numbers:** Always use digits (5, not five).
- **Math:** Use explicit signs (`+5 Damage`, `x1.5 Multiplier`).
- **Avoid:** "Increases your damage." (Too vague). Use "+X Base Damage."

### 7.2 The MVP 8 Relics (From Doc 05)

| ID | Title | Description (Trigger + Effect) |
|---|---|---|
| `RLC-001` | Iron Mallet | **Every line cleared:** +10 Base Damage. |
| `RLC-002` | Combo Capacitor | **L=2 or higher cleared:** Damage Multiplier +0.5x. |
| `RLC-003` | Glass Dagger | **First turn of Battle:** Damage Multiplier x2.0. |
| `RLC-004` | Patience Charm | **Placing a 1x1 Piece:** Restore 1 Move to Move Limit. |
| `RLC-005` | Heavy Corners | **Clearing an edge row/column:** +25 Base Damage. |
| `RLC-006` | Momentum Engine| **Clearing 3 turns in a row:** Next attack deals double damage. |
| `RLC-007` | Executioner | **Enemy below 30% HP:** +50 Base Damage. |
| `RLC-008` | Minimalist | **Placing an I-shape (4-line) piece:** +20 Base Damage. |

---

## 8. Enemy Descriptions

Enemy names are visible in the HUD. Descriptions are visible in a pre-battle modal (if implemented) or bestiary.

| Archetype | Name | Description | Telegraph Tag |
|---|---|---|---|
| Basic Melee | Goblin | Attacks steadily. No board interference. | None |
| Basic Caster | Slime | Weak attacks, but leaves hazard residue. | Drops Hazard |
| Tank | Stone Golem | High defense, reduces incoming damage. | Defends |
| Controller | Ice Elemental | Freezes board cells. | Freezes Cells |
| Boss 1 | Chaos Weaver | Alters piece shapes in your tray. | Corrupts Tray |

---

## 9. Objective Text

Mapped to the 10 MVP objective types from `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`.

| Objective Type | HUD Short Title | HUD Progress | Expanded Description (if tapped) |
|---|---|---|---|
| Defeat | Defeat Enemy | {0} / {1} HP | Reduce the enemy's HP to zero to win. |
| Line Count | Clear Lines | {0} / {1} Lines | Clear the specified number of lines. |
| Row Count | Clear Rows | {0} / {1} Rows | Clear horizontal rows to progress. |
| Col Count | Clear Columns | {0} / {1} Cols | Clear vertical columns to progress. |
| Combo | Perform Combos | {0} / {1} Times | Clear 2 or more lines simultaneously. |
| Move Limit | Survive Moves | {0} Moves | Survive until the move limit reaches zero. |
| Threshold | Deal Damage | {0} / {1} Dmg | Deal the required total damage in one hit. |

---

## 10. Victory Copy

| Context | Copy |
|---|---|
| **Battle Win Header** | Battle Complete |
| **Battle Win Sub** | Enemy defeated. |
| **Reward Prompt** | Choose a Relic |
| **Run Win Header** | Run Complete! |
| **Run Win Sub** | You have conquered the gauntlet. |

---

## 11. Bonus Objective Copy

Bonus objectives use an identical structure to Primary objectives, but are prefaced with "Bonus" and offer a specific currency/score reward.

- **State: Active:** `Bonus: Clear 3 lines simultaneously.`
- **State: Complete:** `Bonus Complete! (+50 Score)`

---

## 12. Game Over Copy

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
| Sound Effects | Controls UI and combat sounds. | Slider (0-100%) |
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
| **Relic Inventory (Mid-run)** | You haven't collected any Relics yet. |
| **Relic Compendium (Home)** | Discover Relics during runs to unlock them here. |

---

## 17. Accessibility Labels (Screen Readers)

Crucial for visually impaired players using OS-level screen readers.

| UI Element | Semantic `aria-label` equivalent |
|---|---|
| **Pause Button** | "Pause the game" |
| **Settings Gear** | "Open Settings Menu" |
| **Relic Card (Grid)** | "Relic: [Name]. [Description]. Tap to select." |
| **Enemy HP Bar** | "Enemy Health: [Current] out of [Max]" |
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
| **Relic Names** | 18 | 1 |
| **Relic Desc** | 80 | 3 |
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
| `ui.btn.relics` | Relic Compendium | Main menu secondary |
| `ui.btn.settings`| Settings | Main menu secondary |
| `ui.btn.pause` | Pause | HUD button |
| `ui.btn.abandon` | End Run | Dialog confirmation |
| `ui.btn.cancel` | Cancel | Dialog cancellation |
| `battle.obj.lines`| Clear Lines | HUD objective title |
| `battle.obj.rows` | Clear Rows | HUD objective title |
| `battle.obj.survive`| Survive Moves | HUD objective title |
| `battle.hud.moves`| {0} Moves Left | HUD limit indicator |
| `battle.hud.bonus`| Bonus: {0} | HUD bonus description |
| `result.win.title`| Battle Complete | Victory screen |
| `result.lose.lock`| Board Lock | Defeat screen (no moves) |
| `result.lose.turn`| Out of Moves | Defeat screen (limit hit) |
| `relic.select.title`| Choose a Relic | Relic screen header |

---

## 21. Copy QA Checklist

Before any string freeze or build submission:
- [ ] **Variable check:** Do all strings containing `{0}` successfully receive their variables without rendering "undefined" or "[Object object]"?
- [ ] **Truncation check:** Set language to German (or a pseudo-localization script that adds +30% length). Do any buttons clip text? Do any Relic cards require scrolling?
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
