# 00_MASTER_GAME_VISION.md
## Block Battles — Master Game Vision

**Document status:** Foundational / Highest Authority
**Applies to:** All downstream design, systems, content, art, audio, and production documents

---

## 1. Document Purpose

This document defines the single source of truth for what Block Battles *is*. It exists because a solo-developer project has no design team to resolve ambiguity through discussion — ambiguity must be resolved once, here, in writing.

**What it controls:** Product identity, core fantasy, design pillars, dual-mode game architecture, MVP scope boundaries, complexity budget, and non-negotiable design rules.

**What it does not control:** Exact numerical parameters (RNG weights, fine particle counts), specific UI layout code, or asset filenames. Those belong to downstream engineering documents that must strictly adhere to this vision.

**DESIGN DECISION:** In any conflict between this document and a downstream document, this document wins. A downstream document may elaborate or implement a decision made here; it may not contradict or reverse it.

**Inheritance rule:** Every later Markdown file must confirm that it does not contradict the principles, pillars, and boundaries set forth in this document.

---

## 2. Product Definition

**One sentence:** Block Battles is a premium, tactile 8×8 block-placement arcade puzzle game featuring an endless Classic High-Score mode and a Goal-Driven Stage Challenge mode.

**One paragraph:** Block Battles takes the timeless, universal satisfaction of placing geometric block pieces on an 8×8 grid and clearing full rows and columns, elevating it with world-class tactile game feel, beveled materials, and explosive arcade feedback. The player launches the game instantly into a clean, distraction-free experience: App Icon → Minimalist Splash → Mode Selection. The game offers two distinct ways to play on a single shared engine: **Mode 1 (Classic Endless)**, an endless high-score pursuit against personal best records ending only on Board Lock; and **Mode 2 (Goal-Driven Stage Challenge)**, a progression of curated puzzle levels featuring target scores, color block clearing quotas, line goals, limited moves, and obstacles (stone, ice).

**Elevator pitch:** "Pure block puzzle mastery: place the grid, chase the record, conquer the stages, feel the juice."

**30-second explanation:** You receive three pieces at a time in your tray and place them strategically onto an 8×8 grid. Full rows and columns clear simultaneously, scoring points and building consecutive streak multipliers. High-juice particle bursts, subtle screen shake, and melodic chimes celebrate every clear. In Classic Mode, you play endlessly until no tray piece can fit on the board, saving your personal best score with instant replay. In Stage Mode, you solve crafted puzzle challenges within limited moves to earn 1 to 3 stars.

---

## 3. Core Fantasy

**"I'm in a pure zen flow state, mastering spatial placement, building massive combo streaks, and smashing puzzle challenges."**

Expanded into player-facing emotions:
- **Competence:** "I read the board cleanly" — spatial intuition and forward planning are rewarded with open board space and massive combo potential.
- **Escalation:** "My streak is heating up" — consecutive line clears ramp up pitch-rising audio chimes, glowing visual multipliers ($1.0\times \to 2.5\times$), and vibrant particle bursts.
- **Juice & Physicality:** "This feels incredible in my hands" — blocks lift with natural finger offsets, snap with crisp micro-haptics, and dissolve in radiant sparkles.
- **Tension Under Control:** "Space is tight, but I have a way out" — board tension is always visible, transparent, and fair.
- **Earned Mastery:** "One more game to beat my record" — game-over on board lock feels 100% fair and immediately prompts an instant restart.

**DESIGN DECISION:** The fantasy is deliberately a pure spatial puzzle and tactile arcade fantasy. It is not an ad-bloated free-to-play grinder. It is a world-class, premium puzzle toy.

---

## 4. First 60 Seconds & First Session Experience

- **0–3s:** App launches with an elegant, snappy splash.
- **3–5s:** Clean Home Screen appears immediately with two clear mode options: "Classic Endless" and "Stage Challenge".
- **5–15s:** Tapping "Classic" presents the 8×8 board with 3 inviting pieces in the tray.
- **15–30s:** Player drags a piece. It lifts smoothly above the touch point, previews a semi-transparent ghost over valid grid cells, and snaps down with a responsive tactile thud.
- **30–45s:** Completing a row or column triggers a crisp particle dissolve, a harmonic chime, and a glowing score popup.
- **45–60s:** Placing consecutive clears triggers the streak multiplier badge ($1.2\times, 1.4\times$), intensifying audio pitch and visual feedback.
- **End of session:** On board lock, a celebratory Game Over dialog presents the final score, compares it to the local High Score, and provides an instant one-tap "Play Again".

---

## 5. Core Design Pillars

### 5.1 Instant Clarity
- Any player must understand the placement and line-clearing loop within seconds, with zero forced tutorials.
- Visual grammar (grid lines, block geometry, fill states, score counters) is self-explanatory.

### 5.2 Tactical Spatial Placement
- Piece placement is a genuine spatial puzzle with deep replayability across 34 canonical shapes across 15 families (9 categories).
- Skilled players achieve higher scores and longer runs through forward planning and pocket management.

### 5.3 Tactile Juice & Zero "AI Slop"
- Every interaction feels weighted, physical, responsive, and aesthetically premium.
- Beveled block materials, frosted glass board framing, elastic pickup lift, micro-snap haptics, particle bursts, and screen shake elevate digital blocks into a luxury toy.

### 5.4 Satisfying Feedback
- Every successful action (cell placement, line clear, streak combo, stage star earned) is confirmed with immediate, proportionate audiovisual response.
- Feedback intensity scales cleanly with clear size ($L=1$ to $L=4+$) and consecutive streak count.

### 5.5 Controlled Deterministic Randomness
- Seeded `GameRNG` creates spatial decisions, not unfair helplessness.
- Solvability protection guarantees at least one placeable piece in every tray roll, preventing cheap RNG dead-ends.

### 5.6 Fair Board Lock
- Game Over in Classic Mode occurs strictly when none of the 3 tray pieces can legally fit onto the board.
- Exhaustive validation checks all 3 pieces across all 64 coordinates before declaring Board Lock.

### 5.7 Solo-Friendly Clean Architecture
- Every system is buildable, testable, and maintainable by a solo developer using AI-assisted tooling.
- Pure Dart domain logic is completely decoupled from Flame rendering and Flutter UI.

---

## 6. Product Differentiation

- **Vs. generic puzzle clones ("AI slop"):** Delivers luxury tactile feel, beveled materials, glassmorphism, responsive finger offsets, particle explosions, and screen shake from day one.
- **Vs. ad-bloated casual games:** Pure, clutter-free flow: App Icon → Animated Splash → Minimalist Home → Instant Play. No intrusive pop-up spam, no fake playable ads, no energy timers.
- **Vs. overcomplicated hybrids:** Focuses purely on the timeless spatial puzzle loop. Zero distraction from non-puzzle systems, delivering a laser-focused, deeply satisfying puzzle experience.
- **Vs. traditional endless puzzles:** Features deterministic fair RNG, board-aware solvability protection, and two cohesive modes (Endless & Goal-Driven Stages) on one rock-solid engine.

---

## 7. Two Game Modes

The game unifies two distinct gameplay modes on a single shared 8×8 board engine:

```text
               ┌─────────────────────────────────┐
               │           APP LAUNCH            │
               │   (Splash Screen → Main Menu)   │
               └────────────────┬────────────────┘
                                │
                ┌───────────────┴───────────────┐
                ▼                               ▼
     ┌─────────────────────┐         ┌─────────────────────┐
     │   MODE 1: CLASSIC   │         │   MODE 2: STAGES    │
     │   (Endless Loop)    │         │ (Goal-Driven Levels)│
     └──────────┬──────────┘         └──────────┬──────────┘
                │                               │
                ▼                               ▼
     ┌─────────────────────┐         ┌─────────────────────┐
     │  8×8 Endless Board  │         │  Curated Stage Grid │
     │  - Chase High Score │         │  - Limited Moves    │
     │  - Streak Multip.   │         │  - Score Milestones │
     │  - Board Lock Exit  │         │  - Color Quotas     │
     │  - 1-Tap Play Again │         │  - Obstacles (Ice)  │
     │                     │         │  - 1-3 Star Ratings │
     └─────────────────────┘         └─────────────────────┘
```

### Mode 1: Classic Endless Mode
- Pure, endless block puzzle play.
- No move limits, no timers, no distractions.
- Real-time scoring: Placement points ($1\text{ pt/cell}$) + Line clear points ($L=1\to 100, L=2\to 300, L=3\to 600, L=4\to 1000$) + Consecutive streak multipliers ($1.0\times \to 2.5\times$) + Board clear bonus (+500 pts).
- Compete against own device: All-time best High Score persisted locally via Hive CE.
- Ends strictly on **Board Lock** (when none of the 3 tray pieces can fit anywhere on the board).
- Celebratory Game Over dialog with instant "Play Again" or "Home".

### Mode 2: Goal-Driven Stage Mode
- Level-based puzzle challenges and curated hurdle progression.
- Authored objectives:
  - Reach target score within move limits (e.g., score 3,500 points in 20 moves).
  - Clear specific block colors (e.g., collect 16 Sapphire blocks).
  - Clear total lines (e.g., clear 8 lines).
  - Clear board obstacles (stone tiles, multi-hit ice blocks).
- Star rating evaluation: 1, 2, or 3 stars awarded based on efficiency and bonus score.
- Progression map: Unlocking stages sequentially with star ratings saved locally.

---

## 8. Player Decision Hierarchy

- **Moment-to-moment:** "Where should I place this piece to keep options open?"
- **Tactical:** "Should I clear a single line now, or build for a simultaneous multi-line clear to score bigger?"
- **Streak Management:** "Can I clear a line this turn to keep my $2.0\times$ multiplier alive, or must I place safely and reset?"
- **Objective Strategy (Mode 2):** "How many moves do I have left, and which color lines do I need to prioritize to fulfill the stage goal?"
- **Session Goals:** "Can I beat my all-time high score, or earn 3 stars on this challenging stage?"

---

## 9. Complexity Budget & Content Ceilings

Launch scope is disciplined and focused:

| Category | Launch Target |
|---|---|
| Game Modes | 2 (Mode 1: Classic Endless, Mode 2: Goal-Driven Stages) |
| Board Grid Size | $8 \times 8$ (64 cells) |
| Tray Capacity | 3 simultaneous shape slots |
| Shape Library | 34 canonical shapes across 15 families (9 categories) |
| Color Themes | 8 vibrant palette themes |
| Obstacle Types | 2 (Stone blocks, Multi-hit Ice blocks) |
| Mode 2 Stages | 30 curated progression stages for MVP launch |
| Persistent Storage | Hive CE (Local on-device save) |

---

## 10. MVP Scope Boundaries

### Absolute MVP
1. Full 8×8 board with smooth drag-and-drop input and finger lift offset.
2. 34 canonical shapes registered in `ShapeCatalog` with fair, solvability-protected RNG replenishment.
3. Complete scoring engine with placement points, multi-line bonuses, streak multipliers ($1.0\times \to 2.5\times$), and board clear bonuses.
4. Mode 1 (Classic Endless) with local high score persistence and board-lock game over.
5. Mode 2 (Goal-Driven Stages) with move counters, score/color goals, and 1–3 star evaluation.
6. Full tactile juice: beveled materials, particle effects, harmonic audio chimes, and haptics.

### Explicitly Forbidden Scope
- No extraneous non-puzzle entities or character overlays.
- No player status meters, attack stats, or conflict pipelines.
- No equipment drafts, card mechanics, or inventory systems.
- No intrusive energy meters, paywalls, or forced ad interruptions mid-game.
- No mandatory online accounts or backend multiplayer server requirements.

---

## 11. Emotional Pacing

Across a play session:
- **Calm & Focus:** Starting a clean board, surveying the initial 3 pieces, establishing spatial rhythm.
- **Engagement:** Assembling rows and columns, watching the board fill strategically.
- **Thrill & Excitement:** Triggering a multi-line clear, hearing the ascending chimes, watching glowing streak multipliers ignite.
- **Tension:** Board space contracting, evaluating remaining slots against tray pieces, executing tactical rescues.
- **Satisfaction:** Achieving a clean board wipe (+500 points) or hitting 3 stars on a stage.
- **Closure & Replay:** Clear Game Over or Stage Clear presentation with immediate, frictionless restart.

---

## 12. Master Terminology

- **Board:** The $8 \times 8$ puzzle grid (64 cells).
- **Piece / Shape:** A geometric polyomino block structure offered in the tray.
- **Tray:** The 3-slot container holding available pieces for the player.
- **Placement:** Dragging and committing a piece to valid board coordinates.
- **Line Clear ($L$):** Completing an entire row or column, removing the blocks ($L=1, 2, 3, 4$).
- **Streak Multiplier:** Consecutive placement moves that clear lines, multiplying line clear points ($1.0\times$ to $2.5\times$).
- **Board Clear:** Clearing all 64 cells on the board, awarding a +500 point bonus.
- **Board Lock:** The state where none of the 3 current tray pieces can legally fit onto the board (triggers Game Over in Classic Mode).
- **Stage (Mode 2):** A curated level with defined move limits and specific completion goals.
- **Obstacle:** Board cells with special clearing conditions (Stone, Ice).
- **Stars (1–3):** Rating awarded upon completing a Stage based on efficiency.

---

## 13. Master Design Rules

1. **Puzzle Clarity First:** Visual effects and animations must never obscure grid state or delay user input.
2. **Instant Input Responsiveness:** Touch input must register immediately; zero frame lag on piece pickup.
3. **Pure Spatial Identity:** Block Battles is strictly a spatial block puzzle game. No extraneous action or fighting mechanics are permitted.
4. **Deterministic Fairness:** RNG must provide interesting spatial options while guaranteeing that at least one tray piece is always placeable.
5. **Clean Architecture:** Core gameplay math lives in a pure Dart domain layer; Flame handles presentation and rendering; Flutter handles UI.
6. **Frictionless Flow:** Navigating between modes, restarting games, and reviewing scores must take one tap with zero loading delays.
7. **Quality over Clutter:** Every feature must serve the core puzzle experience; unnecessary complexity must be aggressively pruned.

---

## 14. Final Product Contract

> **Block Battles is pure puzzle satisfaction.**
>
> It combines the timeless elegance of 8×8 block placement with modern tactile luxury, rewarding streaks, and thoughtful stage challenges. By keeping our vision focused, our rules deterministic, and our architecture decoupled, we deliver a mobile puzzle game that feels extraordinary to play, session after session.

---

**End of `00_MASTER_GAME_VISION.md`.**
