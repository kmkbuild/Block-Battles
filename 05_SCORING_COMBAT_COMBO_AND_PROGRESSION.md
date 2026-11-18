# 05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md
## Block Battles — Scoring, Combo Streaks, and High-Score Persistence

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`  
**Document status:** Implementation-level system design  
**Core Authority:** Pure Arcade Puzzle Scoring, Multi-Line Bonuses, Streak Multipliers, and Device Record Persistence.

---

## 1. Purpose and Ownership

This document defines the mathematical scoring rules for Block Battles: **piece placement points → line clear points → multi-line clear bonuses → consecutive streak multipliers → final score tally → local device High Score persistence.**

**Ownership boundary:** This document owns all point formulas, streak rules, and record persistence triggers. It consumes `PlacementCommitted` and `LinesCleared` events from `03_BOARD_ENGINE_AND_RULES.md`.

---

## 2. Scoring Philosophy

1. **Every action rewards skill:** Placing shapes awards points per block; completing rows and columns awards exponential line clear bonuses.
2. **Rewarding simultaneity:** Clearing multiple lines at once (e.g., 2, 3, or 4+ lines via cross-clears) pays out significantly higher points than clearing single lines sequentially.
3. **Rewarding momentum (Streaks):** Clearing lines on consecutive placement turns builds a visible, audio-rising Combo Streak that multiplies line clear points up to $2.5\times$.
4. **Legible and immediate:** Every point gain spawns a floating pop-up (`+120`, `STREAK x2!`) over the board so the player instantly understands their point gain.
5. **Competitive self-mastery:** The player's score is continuously compared against their device's all-time High Score.

---

## 3. Mathematical Formula

On each player turn:

$$\text{TurnScore} = \text{PlacementPoints} + \left( \text{BaseLineClearPoints}(L) \times \text{StreakMultiplier} \right)$$

Where:
- $\text{PlacementPoints} = \text{shape.blockCount} \times 1$ point.
- $L$ is the total count of lines cleared this turn ($|rows| + |cols|$).
- If $L = 0$: Line clear points = 0, streak counter resets to 0.
- If $L > 0$: Line clear points are computed from the Table below, streak counter increments by 1.

---

## 4. Base Line Clear Point Table

| Lines Cleared ($L$) | Base Line Clear Points | Rationale |
|---|---|---|
| **0 Lines** | 0 pts | Placement only; streak resets. |
| **1 Line (L1)** | 100 pts | Standard single line clear. |
| **2 Lines (L2)** | 300 pts | Double clear bonus ($1.5\times$ per line). |
| **3 Lines (L3)** | 600 pts | Triple clear bonus ($2.0\times$ per line). |
| **4 Lines (L4)** | 1,000 pts | Quadruple clear / cross-clear jackpot. |
| **5+ Lines** | $1,000 + (L - 4) \times 500$ pts | Massive mega-clear milestone. |

---

## 5. Consecutive Turn Streak / Combo Multiplier

A streak is built by clearing at least one line on consecutive piece placement turns.

| Streak Step | Consecutive Turns with Clears | Multiplier | Visual / Audio Feedback |
|---|---|---|---|
| **0** | No prior turn clear | $1.0\times$ | Standard clear flash + chime. |
| **1** | 2 consecutive turns | $1.5\times$ | "STREAK x1.5" badge, chime pitch $+2$ semitones. |
| **2** | 3 consecutive turns | $2.0\times$ | "STREAK x2.0" badge, chime pitch $+4$ semitones, subtle camera shake. |
| **3+** | 4+ consecutive turns | $2.5\times$ (cap) | "MEGA STREAK!" golden badge, chord arpeggio, glowing particle ring. |

**Streak Reset Rule:** As soon as a piece is placed that clears **0 lines**, the streak counter resets immediately back to 0 (multiplier reverts to $1.0\times$).

---

## 6. Local High Score Persistence

- **Storage Key:** `classicHighScore` stored in Hive CE `progressionBox` via `SaveRepository` (Doc 24).
- **Default value:** 0.
- **In-Game Display:** The active in-game HUD displays:
  - `SCORE`: Current running game score (e.g., `1,420`).
  - `BEST`: Device all-time high score with a golden trophy icon (e.g., `8,650`).
- **New High Score Event:**
  - The moment `CurrentScore > BestScore`, the HUD highlights the BEST score with a golden glow pulse.
  - On **Board Lock (Game Over)**, if a new high score was set, the Game Over dialog displays a celebratory "NEW BEST SCORE!" banner with fanfare particle confetti.
- **100% Offline:** Requires zero internet connection or server login.

---

## 7. Score Resolution Pipeline

```text
1. PlacementCommitted (from BoardEngine)
   └── PlacementPoints = shape.blockCount * 1 point
   └── Score += PlacementPoints

2. LinesCleared (from BoardEngine)
   ├── If L == 0 (no lines cleared):
   │   └── Streak counter resets to 0 (Multiplier = 1.0x)
   │   └── Emit ScoreUpdated { turnScore: PlacementPoints, totalScore, streak: 0 }
   │
   └── If L >= 1 (1 or more lines cleared):
       ├── Increment streak counter (capped at 3+)
       ├── StreakMultiplier = lookup(streak) [1.0x, 1.5x, 2.0x, 2.5x]
       ├── BasePoints = Table_4_Lookup(L)
       ├── LinePoints = round(BasePoints * StreakMultiplier)
       ├── Score += LinePoints
       └── Emit ScoreUpdated { turnScore: (PlacementPoints + LinePoints), totalScore, streak }
       └── Emit StreakUpdated { streak, multiplier: StreakMultiplier }

3. High Score Comparison
   ├── If Score > HighScore:
   │   ├── HighScore = Score
   │   └── Save locally via SaveRepository (field: classicHighScore in progressionBox)
   │   └── Emit NewHighScoreAchieved { newHighScore: Score }
```

---

## 8. Audiovisual Juice & Escalation

1. **Floating Score Text Popups:**
   - On line clear, floating text spawns at the geometric center of the cleared lines (e.g. `+100`, `+300`).
   - If streak > 0, an accompanying badge spawns (e.g. `STREAK x1.5!`, `MEGA STREAK x2.5!`).
2. **Audio Pitch Escalation:**
   - Clearing lines on consecutive turns shifts the clear chime pitch upward by +2 semitones per streak tier.
   - Streak 3+ triggers a full musical major chord arpeggio.
3. **Screen Juice:**
   - L = 1: Subtle 100ms haptic tap, soft board shake (2px).
   - L = 2: Medium 150ms haptic buzz, moderate board shake (4px).
   - L >= 3: Heavy 250ms haptic feedback, 6px camera shake, golden particle explosion.

---

## 9. Board Wipe (Full Clear) Bonus

If a line clear results in an entirely empty 8x8 board (0 filled cells remaining):
- **Bonus Award:** +500 flat points.
- **Audio/Visual:** "BOARD CLEAR!" banner with dual-side golden fireworks and resonant gong chime.

---

## 10. Local Persistence Contract

- **Technology:** Hive CE (`hive_ce` via `SaveRepository`).
- **Entity & Field:** `ProgressionData.classicHighScore` in `progressionBox` (Doc 24).
- **Default:** `0`.
- **Safety:** Updated immediately on new record and flushed on app backgrounding to prevent data loss.
- **Offline:** Absolutely no network requests or external dependencies.

---

## 11. Mode 2: Goal-Driven Score Targets (Reserved Architecture)

In Mode 2, scoring rules remain identical, but add a goal condition:
- `targetScore`: Target integer required to clear the stage.
- `movesRemaining`: Decremented on each placement.
- Victory triggered if `currentScore >= targetScore` before `movesRemaining == 0`.

---

## 12. Acceptance Criteria

- Placement points awarded for every valid piece drop (1 point per block in shape).
- Single line clear (L=1) awards exactly 100 points at 1.0x streak.
- Simultaneous multi-line clears award the exact values from Section 4 (L=2 -> 300, L=3 -> 600, L=4 -> 1000).
- Consecutive clears correctly escalate the streak multiplier up to 2.5x.
- Any turn with 0 lines cleared resets the streak multiplier back to 1.0x.
- New high score updates persistent storage immediately.
- Works 100% offline.

---

## 13. Worked Numerical Examples

**Example 1: Single Placement, No Lines Cleared**
- Shape: 3-cell L-tromino placed.
- L = 0.
- PlacementPoints = 3 * 1 = 3.
- Streak resets to 0 (1.0x).
- TurnScore = 3.

**Example 2: Single Line Clear, Fresh Streak**
- Shape: 4-cell I-tetromino placed, completing 1 row.
- L = 1. Streak was 0 -> becomes 1 (1.5x).
- PlacementPoints = 4.
- BaseLinePoints = 100.
- LinePoints = 100 * 1.5 = 150.
- TurnScore = 4 + 150 = 154.

**Example 3: Double Cross-Clear with Streak Tier 2**
- Shape: 1-cell Monomino placed at row/col intersection, completing 1 row and 1 column.
- L = 2. Prior streak was 1 -> becomes 2 (2.0x).
- PlacementPoints = 1.
- BaseLinePoints = 300.
- LinePoints = 300 * 2.0 = 600.
- TurnScore = 1 + 600 = 601.

---

**End of `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`.**
