# 00_MASTER_GAME_VISION.md
## Block Battles — Master Game Vision

**Document status:** Foundational / highest authority
**Applies to:** All downstream design, systems, content, art, audio, and production documents

---

## 1. Document Purpose

This document exists to define the single source of truth for what Block Battles *is*, before any system, content, or production document is written. It exists because a solo-developer project has no design team to resolve ambiguity through discussion — ambiguity must be resolved once, here, in writing.

**What it controls:** product identity, core fantasy, design pillars, high-level architecture, MVP scope boundaries, complexity budget, and non-negotiable design rules.

**What it does not control:** exact numerical values (damage, HP, RNG weights), UI layouts, art direction specifics, code architecture, or monetization mechanics. Those belong to downstream documents that must not contradict this one.

**DESIGN DECISION:** In any conflict between this document and a downstream document, this document wins. A downstream document may narrow or elaborate a decision made here; it may not reverse it without an explicit revision to this file.

**Inheritance rule:** Every later Markdown file (shapes, board, combat, objectives, relics, UI, audio, etc.) must open by confirming it does not contradict Sections 5, 12, 13, and 23 of this document.

---

## 2. Product Definition

**One sentence:** Block Battles is a premium, tactile 8×8 block-placement arcade puzzle game featuring an endless Classic High-Score mode and an extensible Goal-Driven challenge architecture.

**One paragraph:** Block Battles takes the moment-to-moment satisfaction of placing polyomino pieces on an 8×8 grid and clearing full rows and columns, elevating it with world-class tactile game feel, physical block materials, and explosive arcade feedback. The player launches the game instantly into a clean, clutter-free experience: App Icon → Animated Splash → Minimalist Home → Classic Mode. In Classic Mode, the player plays an endless high-score loop competing against their own device's all-time best record until Board Lock is reached. The game engine is built on deterministic board-aware RNG and clean architecture, preserving dedicated extension hooks for a future level-based Goal-Driven mode without cluttering the pure arcade loop.

**Elevator pitch:** "Pure block puzzle mastery: place the grid, chase the record, feel the juice."

**30-second explanation:** You get three pieces at a time and place them on an 8×8 grid. Full rows and columns clear simultaneously, scoring points and building combo streaks. High-juice particle bursts, screen shake, and melodic chimes celebrate your clears. When none of your tray pieces can fit on the board, Board Lock triggers Game Over, saving your new local High Score and offering an instant one-tap replay. A future Goal-Driven mode will introduce level hurdles on top of this same rock-solid engine.

**Detailed product definition:** Block Battles is a single-player, session-based mobile puzzle game focused on the timeless satisfaction of spatial organization and line clearing. The puzzle layer (placement, clearing, streak multipliers) is the core gameplay; the tactile presentation layer (beveled materials, lift offset, snap haptics, particle bursts) transforms basic flat blocks into a juicy, premium mobile experience; the persistence layer saves your local personal best on device; and the extensible architecture reserves clean hooks for level-based puzzle hurdles.

---

## 3. Core Fantasy

**DESIGN DECISION:** The core fantasy is: *"I'm in a pure zen flow state, mastering spatial placement, building massive combo streaks, and smashing my personal best high score."*

Expanded into player-facing emotions:

- **Competence:** "I'm good at reading this board" — spatial intuition and forward planning are rewarded with clean, open board states.
- **Escalation:** "My combo is heating up" — consecutive line clears ramp up pitch-rising audio chimes, glowing visual multipliers, and explosive particle bursts.
- **Juice & Physicality:** "This feels incredible in my hands" — blocks lift with natural finger offsets, snap with micro-haptics, and dissolve in white-hot flashes.
- **Tension under control:** "Space is tight, but I have a way out" — board tension is always visible, transparent, and fair.
- **Earned mastery:** "One more game to beat my record" — game-over on board lock feels 100% fair and immediately prompts an instant restart.

**DESIGN DECISION:** The fantasy is deliberately a systems and tactile arcade fantasy. It is not an RPG, not an enemy battler, and not an ad-bloated free-to-play grinder. It is a world-class, premium puzzle toy.

---

## 4. Target Player Experience

- **First 3 seconds:** Player taps app icon. A clean, smooth animated Splash Screen loads effortlessly into a minimalist, clutter-free Home Screen.
- **First 10 seconds:** Player taps "Play Classic" and is instantly on the 8×8 board with 3 tray pieces. No tutorial text walls. Touch pickup feels elastic; live ghost preview snaps beneath the finger.
- **First minute:** Player completes their first line clear. An immediate white-hot flash and crisp audio chime trigger dopamine. The score updates with a floating pop-up.
- **First streak:** Player clears lines on consecutive turns. A combo multiplier badge illuminates with rising musical pitches.
- **First difficult board:** The board fills up. The player carefully scans the grid for open pockets. Board-aware RNG ensures fair, solvable draws without arbitrary dead-ends.
- **First game over:** Board Lock occurs when no tray piece can fit. The board pauses cleanly, evaluates the final score against the device's all-time High Score, and presents a celebratory Game Over dialog with an instant one-tap "Play Again".
- **Second session:** The player's device remembers their personal best score. They jump straight back into the loop to beat it.

---

## 5. Core Design Pillars

### 5.1 Instant Clarity
- **Definition:** Any player must understand the placement/clear loop within seconds, with zero required reading.
- **Player benefit:** No onboarding friction; mobile players do not tolerate long tutorials.
- **Implementation implication:** Visual grammar (grid, piece shapes, fill states) must be self-explanatory; current score and best score are legible at a glance.
- **Failure condition:** A first-time player hesitates on what action to take next.
- **Acceptance criteria:** A player with zero instructions places a piece and clears a line correctly within their first 3 attempts.

### 5.2 Tactical Placement
- **Definition:** Piece placement is a genuine spatial puzzle with deep replayability.
- **Player benefit:** The core loop remains engaging for hundreds of hours on its own spatial merits.
- **Implementation implication:** 24 canonical shapes and an 8×8 board size preserve meaningful spatial tension.
- **Failure condition:** Optimal play becomes "place anywhere, clears happen automatically."
- **Acceptance criteria:** Skilled players consistently achieve higher scores through strategic pocket management.

### 5.3 Tactile Juice & Zero "AI Slop"
- **Definition:** Every interaction feels weighted, physical, responsive, and aesthetically premium.
- **Player benefit:** Moment-to-moment physical satisfaction that elevates a digital game into a luxury digital toy.
- **Implementation implication:** Beveled gradient block materials, frosted glass board framing, elastic pickup lift, micro-snap haptics, particle bursts, and screen shake.
- **Failure condition:** The game looks like flat, generic colored squares on a plain background.
- **Acceptance criteria:** Players describe the physical feel as "addictive", "satisfying", and "crisp".

### 5.4 Satisfying Feedback
- **Definition:** Every successful action (placement, clear, combo, new high score) is confirmed with immediate, proportionate audiovisual response.
- **Player benefit:** Moment-to-moment dopamine that sustains long sessions.
- **Implementation implication:** Feedback intensity scales with clear size (L1 through L4+) and streak count.
- **Failure condition:** A 3-line cross-clear feels the same as a single 1-line clear.
- **Acceptance criteria:** Players can distinguish clear magnitude by sound and visual impact alone.

### 5.5 Controlled Randomness
- **Definition:** RNG creates spatial decisions, not unfair helplessness.
- **Player benefit:** Replayability without cheap losses.
- **Implementation implication:** Seeded Mulberry32 `GameRNG`, solvability protection, duplicate shape prevention, and board occupancy reweighting.
- **Failure condition:** A player loses due to an unavoidable RNG dead-end with no prior decision that could have prevented it.
- **Acceptance criteria:** Losses can always be traced to prior placement decisions.

### 5.6 Fair Board Lock
- **Definition:** Game Over occurs strictly when no legal moves remain on the board.
- **Player benefit:** Complete trust in game rules; losses feel earned.
- **Implementation implication:** Exhaustive validation checks all 3 tray pieces across all 64 coordinates before declaring Board Lock.
- **Failure condition:** Game ends prematurely while a piece could have legally fit.
- **Acceptance criteria:** Game Over fires if and only if zero legal placements exist.

### 5.7 Build Identity
- **Definition:** Relic combinations should produce distinct playstyles across runs.
- **Player benefit:** Long-term replay motivation.
- **Implementation implication:** Relics should be grouped into synergy families (e.g., combo-focused, single-clear-focused, defensive) rather than isolated stat bumps.
- **Failure condition:** All runs converge on identical optimal play regardless of relics drawn.
- **Acceptance criteria:** Player-facing surveys/testing can identify at least 3 distinguishable build identities at MVP scale.

### 5.8 Solo-Friendly Production
- **Definition:** Every system must be buildable, content-able, and maintainable by one developer using AI-assisted tooling.
- **Player benefit:** Indirect — a shippable, sustainable game rather than an abandoned one.
- **Implementation implication:** Prefer data-driven, reusable systems over bespoke, one-off content (see Sections 12, 15, 20).
- **Failure condition:** A feature requires bespoke code or hand-authored content per instance at a rate the solo developer cannot sustain.
- **Acceptance criteria:** Every new content item (relic, enemy, objective) can be authored primarily through data/config, not new systems code.

---

## 6. Product Differentiation

**ASSUMPTION:** The following comparisons describe design intent, not verified market claims.

- **Vs. generic puzzle clones ("AI slop"):** Delivers luxury tactile feel, beveled materials, glassmorphism, responsive finger offsets, particle explosions, and screen shake from day one.
- **Vs. ad-bloated casual games:** Pure, clutter-free flow: App Icon → Animated Splash → Minimalist Home → Instant Play. No intrusive pop-up spam, no fake playable ads, no energy timers.
- **Vs. complex RPG puzzle hybrids:** Focuses on the pure, timeless spatial puzzle loop. Eliminates tedious enemy health bars, damage math, and relic drafting in favor of crisp high-score chasing.
- **Vs. traditional endless puzzles:** Features deterministic fair RNG, board-aware solvability protection, and an extensible architecture ready for level-based hurdle goals.

---

## 7. Core Gameplay Architecture

```text
APP LAUNCH
    ↓
SPLASH SCREEN
    ↓
MINIMALIST HOME
    ↓
MODE SELECTION (Classic High Score vs Goal-Driven Hurdles)
    ↓
[CLASSIC MODE]
    ↓
8×8 PUZZLE BOARD (3-Slot Tray)
    ↓
PIECE PLACEMENT (Tactile Drag & Drop)
    ↓
LINE CLEAR (Simultaneous Row/Col Scan)
    ↓
SCORING & COMBO STREAKS (Visual & Audio Juice)
    ↓
TRAY REFILL (3 New Shapes, Deterministic Fair RNG)
    ↓
BOARD LOCK CHECK (Any legal moves remaining?)
    ↓ (No moves left)
GAME OVER
    ↓
LOCAL HIGH SCORE PERSISTENCE & EVALUATION
    ↓
INSTANT REPLAY / RETURN HOME
```

---

## 8. Two Game Modes: Classic Mode + Goal-Driven Mode

**DESIGN DECISION:** The project unifies two distinct modes on one shared, rock-solid 8×8 board engine:

### Mode 1: Classic Mode (Primary Active Focus)
- Pure, endless block puzzle play.
- No timers, no enemies, no distractions.
- Real-time scoring: Placement points + Line clear points + Multi-line bonus + Streak multipliers.
- Compete against own device: All-time best High Score persisted locally.
- Ends strictly on **Board Lock** (when none of the 3 tray pieces can fit anywhere on the board).
- Celebratory Game Over dialog with instant "Play Again" or "Home".

### Mode 2: Goal-Driven Mode (Reserved Architecture)
- Level-based puzzle challenges and hurdles.
- Authored objectives: Reach target score, clear frozen blocks, clear pre-filled patterns within a move limit.
- Cleanly decoupled: shares the exact same board, shapes, and input engine, with objective parameters configured as pure data.
- Built after Classic Mode achieves god-level visual and tactile perfection.

---

## 9. Game Modes

**DESIGN DECISION:** Launch scope is intentionally narrow.

### Main Run
The primary roguelite experience described in Section 7: sequential battles, relic choices, periodic bosses, run-end, persistent reward.

### Challenge/Objectives
Standalone handcrafted Block Quest encounters using the same engine and rule-modifier system described in Section 8, accessible outside the main run structure (e.g., as discrete, replayable challenges).

**MASTER DESIGN RULE:** No additional game mode is added at launch unless it can be justified against Section 12 (Complexity Budget) and does not require new core systems beyond the placement/clear/combat/objective engine already defined.

**OPEN DECISION:** Whether Challenge/Objectives content is delivered as a static curated set or a rotating/seasonal set is deferred.

---

## 10. Battle Philosophy

- **Player board:** The single shared surface for both puzzle-solving and combat output; there is no separate "combat screen."
- **Enemy:** Represented primarily by an HP value and a small set of legible mechanics/telegraphed actions.
- **Enemy HP:** Decreases via line-clear-generated damage; the primary win condition for a standard battle.
- **Player HP:** **ASSUMPTION, pending Open Decision below.**
- **Damage:** Derived from clear size/combo, modified by active relics.
- **Relics:** Passive modifiers that alter how clears generate damage/effects; do not require separate input during battle.
- **Objectives:** Layer additional win/loss conditions or constraints on top of standard battle rules (Section 8).
- **Turns:** **OPEN DECISION**, see below.
- **Enemy mechanics:** Simple, telegraphed effects (e.g., adding obstacles to the board, applying a debuff) that pressure the player's placement options without requiring direct-control combat input.

**CRITICAL ASSESSMENT — Does player HP belong in the MVP?**

**DESIGN DECISION:** Player HP is **not required for the Absolute MVP.** A simpler, board-pressure-based loss condition (e.g., the board fills up / becomes unsolvable, or a turn/time limit expires before the enemy is defeated) can create equivalent stakes with less system surface: no HP economy to balance, no defensive relic category to design, no UI for a second resource bar. This is a strict application of Section 5.8 (Solo-Friendly Production) and the "Most Important Design Principle": HP as a resource only earns its place if it creates decisions that a board-pressure loss condition cannot.

**OPEN DECISION:** Whether player HP is introduced in MVP+ once the core loop is validated, and if so, what damages it (enemy mechanics vs. failing to clear in time), is deferred to a dedicated Combat design document.

**OPEN DECISION:** Whether battles are turn-based (enemy acts after N player placements) or continuous/real-time-with-telegraphs is deferred to the same document. Both must satisfy Section 5.6 (Fair Difficulty)'s telegraphing requirement.

---

## 11. Player Decision Hierarchy

- **Moment-to-moment:** "Where should I place this piece?"
- **Tactical:** "Should I clear now, or set up a stronger future clear?"
- **Build:** "Which relic best fits my strategy?"
- **Battle:** "How should I approach this enemy's specific mechanic?"
- **Run:** "What kind of build am I becoming, and what should I prioritize next?"
- **Meta:** "What did I learn, and how should I approach the next run / spend persistent rewards?"

**DESIGN DECISION:** Every system added to the game must be traceable to at least one tier of this hierarchy. A system that does not affect any of these six decision levels fails the Most Important Design Principle test and should be cut.

---

## 12. Complexity Budget

**DESIGN DECISION:** The following are explicit launch ceilings, not targets to fill for their own sake. Exact final numbers are refined in Section 14; this section sets the *type* of budget discipline required.

| Category | Launch Ceiling (approximate) |
|---|---|
| Currencies | 1 run-time (n/a persistent) + 1 persistent meta-currency (2 total) |
| Status effects | Small, single-digit set |
| Game modes | 2 (Main Run, Challenge/Objectives) |
| Persistent (meta) upgrade categories | Small, single-digit set |
| Relics (launch) | See Section 14 |
| Enemies (launch) | See Section 14 |
| Bosses (launch) | See Section 14 |

**MASTER DESIGN RULE:** Any proposal to exceed a ceiling in this table requires an explicit revision to this document with justification tied to Section 11 (Player Decision Hierarchy) — not merely "more content is better."

---

## 13. MVP Scope

### Absolute MVP
The 8×8 placement/clear board; a small set of piece shapes; one battle loop (player clears lines → enemy HP decreases); a minimal enemy set with simple, telegraphed mechanics; a board-pressure-based loss condition (Section 10); a small relic pool with visible in-run effects; sequential battles culminating in one boss; one persistent meta-reward on run end.

### MVP+
A second, larger relic pool enabling distinguishable build identities (Section 5.7); the Challenge/Objectives mode (Section 8–9); an expanded enemy/boss roster; expanded persistent meta-progression; refined juice/feedback pass (Section 17).

### Post-launch
Additional themes/environments; seasonal or rotating Challenge content; expanded status-effect and relic-synergy depth; deeper meta-progression trees.

### Explicitly forbidden scope
- Player HP as a resource, unless validated per the Section 10 Open Decision — not added by default.
- A second core input method (e.g., direct-control attacks) that competes with placement as the primary action.
- Objective rule-sets that fork into a structurally different game (see Section 8's Master Design Rule).
- Any system whose primary justification is "other games in the genre have it" rather than a traceable player decision (Section 11).
- Multiplayer, PvP, or social/competitive features at launch.

---

## 14. Launch Content Target

**ASSUMPTION:** Figures below are practical starting targets for a solo developer and are expected to be refined once downstream design documents (Shapes, Enemies, Relics, Objectives) are written.

| Content Type | Launch Target |
|---|---|
| Piece shapes | Small, reusable polyomino set (single-digit to low-double-digit count) |
| Enemies (standard) | Low single-digit archetype count, reused with scaling/variation |
| Bosses | 1–3 at Absolute MVP |
| Relics | Low double-digit count at Absolute MVP, expanded in MVP+ |
| Objective templates | Single-digit reusable rule-modifier templates (Section 8) |
| Handcrafted battles/objective instances | Built by combining objective templates with parameter variation, not fully bespoke authorship per instance |
| Environments/themes | 1 at Absolute MVP |
| Persistent progression elements | Single-digit set of meta-unlocks/upgrades |

**MASTER DESIGN RULE:** Favor parameterized reuse (same enemy archetype with different stat/mechanic weights; same objective template with different rule values) over bespoke one-off content, per Section 5.8.

---

## 15. Long-Term Expansion Strategy

**DESIGN DECISION:** Post-launch growth should occur by adding *data*, not rebuilding *systems*:

- **Relics:** New entries added to the existing relic-family framework (Section 5.7).
- **Enemies:** New archetypes built from the same mechanic-telegraph framework (Section 10).
- **Objectives:** New rule-parameter combinations within the existing Block Quest modifier taxonomy (Section 8).
- **Bosses:** Composed from existing enemy-mechanic building blocks at higher complexity/scale, not new mechanic categories by default.
- **Themes:** Visual/audio reskins of the existing board and enemy framework.
- **Modifiers:** New optional rule toggles usable by both Main Run and Challenge/Objectives content.

**MASTER DESIGN RULE:** If a proposed expansion cannot be expressed as new data within the existing frameworks, it must be evaluated against Section 23 before being added — expansion should not silently grow the complexity budget in Section 12.

---

## 16. Emotional Pacing

Mapped across a representative run:

- **Curiosity:** Run start — what pieces, enemies, and relics will this run offer?
- **Tension:** Approaching a difficult battle or a board nearing capacity.
- **Relief:** A clean line clear that resolves board pressure or finishes an enemy.
- **Excitement:** A large combo clear or a powerful relic offer.
- **Greed:** Choosing to delay a clear for a bigger future payoff (Section 11, Tactical tier).
- **Risk:** Committing to a build direction that may not pay off against upcoming enemies.
- **Mastery:** Recognizing and correctly handling a previously-difficult enemy mechanic.
- **Surprise:** An unusual relic synergy or objective rule producing an unexpected outcome — always within the bounds of Section 5.6's fairness requirement.

**DESIGN DECISION:** Emotional pacing should oscillate between tension and relief on a short (per-battle) cycle, while curiosity, greed, and mastery build on a longer (per-run) cycle.

---

## 17. "Juice" Philosophy

**DESIGN DECISION:** Feedback intensity must always scale with and clearly communicate the *magnitude* of the triggering action; it must never obscure board state.

- **Placement:** Subtle, immediate, confirms input registered.
- **Clearing:** Primary feedback moment; intensity scales with lines/combo size.
- **Damage:** Directly and visibly tied to the clear that caused it, so causality is legible.
- **Combos:** Escalating feedback layer distinct from single clears, reinforcing the tactical "delay for a bigger clear" decision (Section 11).
- **Enemy damage (to player, if applicable):** Distinct from player-caused damage, always readable as a discrete, telegraphed event (Section 5.6).
- **Relic selection:** Celebratory but brief; should not gate the player from returning to the board.
- **Boss defeat:** The single largest feedback moment in a run; must be clearly distinguishable from standard enemy defeat.
- **Objective completion:** Distinct acknowledgment separate from standard battle victory, reflecting its handcrafted nature.
- **Game over:** Should feel conclusive but not punishing in tone, in service of immediate-retry motivation (Section 4).

**Note:** Detailed animation timing, particle counts, and audio asset specifics belong to a dedicated Juice/VFX/Audio document, not this file.

---

## 18. Fairness Philosophy

- **Fair randomness:** Piece draws and relic offers must be weighted to avoid unsolvable or clearly-losing states (Section 5.5).
- **Readable enemy mechanics:** Every enemy mechanic must be telegraphed before it resolves (Section 5.6).
- **Transparent objectives:** Challenge/Objectives rules must be stated plainly before the encounter begins, not discovered through trial and error.
- **Predictable rules:** Core placement/clear/damage rules never change silently within a run; only relics and objective parameters modify them, and always visibly.
- **Recoverability:** A single bad placement should rarely be unrecoverable; the system should generally allow a player to play their way out of a mistake.
- **Avoiding cheap deaths:** Loss must always be traceable to a sequence of decisions the player could learn from (Section 5.6 acceptance criteria).

---

## 19. Accessibility Philosophy

- **Readable UI:** High-contrast, legible at small mobile sizes, minimal reliance on fine detail.
- **Color-independent communication:** No game-critical information conveyed by color alone (shapes/icons/patterns as backup).
- **Reduced motion:** A toggle to dampen non-essential animation for motion-sensitive players.
- **Haptics toggle:** On/off control for vibration feedback.
- **Audio toggles:** Independent music and SFX volume/mute controls.
- **Text clarity:** Minimal reliance on dense text; short, plain-language relic/objective descriptions (Section 4).
- **Touch forgiveness:** Generous hit/drop targets and placement-snapping tolerant of imprecise mobile touch input.

---

## 20. Technical-Neutral Architecture Philosophy

**DESIGN DECISION:** The conceptual architecture favors:

- **Data-driven content:** Shapes, enemies, relics, and objective templates defined as data/config, not hardcoded logic, directly supporting Sections 12, 14, and 15.
- **Modular systems:** Placement, clearing, damage, relic-effects, and objective-rule layers should be separable systems that compose rather than a single monolithic combat system.
- **Deterministic core rules:** Given the same board state, piece, and active relics, outcomes must be reproducible — critical for both fairness (Section 18) and testability.
- **Reusable effects:** Relic and enemy-mechanic effects should draw from a shared, extensible effect framework rather than one-off implementations per item.
- **Reusable objective templates:** Per Section 8's Master Design Rule.
- **Reusable enemy abilities:** Enemy mechanics composed from a shared ability framework, supporting Section 15's expansion strategy.
- **Testability:** Deterministic, modular, data-driven systems are inherently easier for a solo developer (often AI-assisted) to test and iterate on.

**Note:** This section is deliberately implementation-neutral; specific engine, language, and code architecture decisions belong to a separate technical design document.

---

## 21. Documentation Ownership Map

| Domain | Owning Document (future) |
|---|---|
| Gameplay (core loop detail) | Core Gameplay Design doc |
| Shapes | Shapes & Pieces doc |
| Board | Board & Grid Rules doc |
| RNG | RNG & Fairness doc |
| Combat | Combat & Battle Systems doc |
| Scoring | Scoring & Damage Formulas doc |
| Input | Input & Controls doc |
| Objectives | Block Quest Objectives doc |
| Future UI | UI/UX Design doc |
| Visuals | Art Direction doc |
| Audio | Audio Direction doc |
| VFX | Juice/VFX doc |
| Monetization | Monetization Design doc |
| QA | QA & Testing Plan doc |

**MASTER DESIGN RULE:** No downstream document may claim authority over a domain outside its row in this table without updating this map.

---

## 22. Master Terminology

- **Run:** One complete playthrough from first battle to run end.
- **Battle:** One enemy encounter resolved on the board.
- **Board:** The 8×8 grid.
- **Piece:** A polyomino shape offered to the player for placement.
- **Placement:** The act of committing a piece to the board.
- **Clear:** The removal of a fully-filled row or column.
- **Combo:** Multiple clears resulting from a single placement or a chained sequence, per rules defined in a future Scoring document.
- **Damage:** The combat effect derived from a clear, applied to an enemy.
- **Relic:** A persistent, passive, run-scoped modifier chosen after a battle.
- **Objective:** A handcrafted encounter using Block Quest rule modifiers (Section 8).
- **Enemy:** A battle opponent defined primarily by HP and telegraphed mechanics.
- **Boss:** A higher-complexity enemy marking a significant run milestone.
- **Persistent Reward / Meta-Progression:** Rewards or unlocks that carry across runs.
- **Block Battles:** The roguelite combat half of the hybrid design (Section 8).
- **Block Quest:** The handcrafted-objective half of the hybrid design (Section 8).

**MASTER DESIGN RULE:** All later documents must use these terms exactly as defined here; new terms introduced downstream must not redefine or conflict with this glossary.

---

## 23. Master Design Rules

1. Puzzle clarity always outranks visual effects.
2. Player input must remain responsive at all times; no feedback effect may block or delay the next input.
3. Combat must enhance the puzzle rather than replace or compete with it as the primary action (Section 3, Section 10).
4. Randomness must create decisions, not frustration (Section 5.5, Section 18).
5. Objectives must reuse existing mechanics/parameters wherever possible rather than introducing bespoke one-off systems (Section 8, Section 15).
6. All content must remain feasible for a solo, AI-assisted developer to build and maintain (Section 5.8, Section 12).
7. Every system must map to at least one tier of the Player Decision Hierarchy (Section 11) or be cut.
8. No feature is added merely because it is common in the genre or "sounds impressive" (Section 13, forbidden scope).
9. Losses must always be explainable and traceable to a prior decision (Section 5.6, Section 18).
10. This document outranks all downstream documents in case of conflict (Section 1).

---

## 24. Open Questions

- Does the MVP require player HP, and if so, in what form? (Section 10)
- Is battle resolution turn-based or continuous-with-telegraphs? (Section 10)
- What is the complete taxonomy of Block Quest rule-modifier parameters? (Section 8)
- Is Challenge/Objectives content static/curated or rotating/seasonal at launch? (Section 9)
- What is the precise combo/scoring formula linking clears to damage? (deferred to Scoring doc)
- What specific persistent meta-progression currencies/unlocks exist beyond the single-digit ceiling in Section 12?
- What is the final numeric content target for each category in Section 14, once downstream documents are drafted?

---

## 25. Final Product Contract

**What the game is:** An 8×8 block-placement puzzle wrapped in a roguelite combat and relic-build structure, extended by handcrafted Block Quest objectives on the same core engine.

**What it is not:** A direct clone of any specific existing block-puzzle title; a stat-heavy deckbuilder; a real-time action-combat game; an endless, structureless score-chaser.

**Who it is for:** Mobile puzzle players who want the placement/clearing loop they already understand, extended with build-driven strategic depth and bounded, replayable runs.

**What makes it different:** The fusion of a familiar, low-friction puzzle input with roguelite stakes and build identity, delivered through a single reusable engine that also powers handcrafted tactical content — designed explicitly for solo, AI-assisted production feasibility.

**What the MVP must prove:** That clearing lines can function convincingly as a combat mechanism, that relic choices produce genuinely distinguishable builds, and that the full loop (battle → relic → next battle → boss → run end → persistent reward) is satisfying enough to motivate an immediate second run — all within the complexity budget defined in Section 12.
