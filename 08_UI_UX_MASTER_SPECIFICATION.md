# 08_UI_UX_MASTER_SPECIFICATION.md
## Block Battles — UI/UX Master Specification

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `06_INPUT_DRAG_AND_DROP_UX.md`, `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`
**Document status:** First Layer B document — player-facing experience architecture

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23, and introduces no new gameplay mechanic — it is a pure presentation layer over the eight Layer A documents. It notably inherits the binding absence of a Player HP resource (`00` Section 10, `01` Section 10, `05` Section 12): no screen, HUD element, or state in this document ever displays player health. It uses Master Terminology (`00` Section 22) without renaming.

---

## 1. Document Purpose

**Scope:** This document owns the complete player-facing information architecture — what screens exist, why, what the player sees on each, information priority, control placement, transitions, and how Battle/Objective/Relic/Board/Progression state is communicated.

**Ownership:** Structure, hierarchy, and behavior only. It does not own final colors, materials, character/block art style, particle/VFX implementation, sound assets, or exact animation curves — those belong to future Layer B documents (Section 25).

**Downstream dependencies:** Every future visual/audio/animation document must implement this document's structure without altering its information hierarchy (Section 3) or screen inventory (Section 5) without a revision here first.

**Precedence:** Where this document's needs would require a gameplay rule that does not exist in Layer A, this document defers to Layer A and flags an Open Decision rather than inventing new gameplay (e.g., Section 10's Relic confirmation step is a UX-only addition — it changes no data).

**Non-goals:** No monetization system is defined anywhere in Layer A (no IAP, ad, or currency-spend mechanic exists in `00`–`07`); this document therefore reserves no dedicated monetization screen or surface at Absolute MVP (Section 4, Section 25).

---

## 2. UX Vision

Mapped directly onto `00_MASTER_GAME_VISION.md` Section 4's Target Player Experience:

- **First launch:** Home screen loads directly to a single clear primary action (Start Run); no forced account creation, no forced tutorial video.
- **First battle:** Board and Tray are visible within one transition; the first Enemy's HP and the Primary Objective are legible before the player's first drag.
- **First victory:** Enemy defeat is unambiguous (HP visibly reaches zero, distinct victory beat) and leads directly into Relic Selection with no extraneous screen between.
- **First relic selection:** Three plain-language options, current build context visible, one confirmed tap.
- **First boss:** Presentation is recognizably elevated (Section 11) but uses the exact same HUD grammar as a standard Battle — nothing new to learn.
- **First failed run:** Defeat is explained (cause is named, Section 13), framed neutrally, and a restart is one tap away.
- **Return to menu:** Persistent currency earned this Run (`05` Section 18/Table 23.5) is visibly reflected on Home without requiring navigation into a separate progression screen.
- **Second run:** The player recognizes every HUD element instantly — no re-learning cost — while the specific Relic offers, Trays, and Battle sequence (per `07` Section 17) differ.

---

## 3. Information Hierarchy

| Priority | Definition | Examples |
|---|---|---|
| **Priority 1 — Immediate gameplay state** | What the player is acting on right now. | Board occupancy, current Tray (3 Pieces), Ghost Preview (`06` Section 8) and its valid/invalid state. |
| **Priority 2 — Objective/combat state** | What determines win/loss, updated continuously. | Enemy HP, Primary Objective + progress, Move Limit remaining (`07` Section 8), Combo/Damage feedback, Enemy Telegraph (next action). |
| **Priority 3 — Strategic information** | Informs planning but isn't needed every second. | Active Relic list, Bonus Objective + progress, Enemy Defense indicator (if non-zero), Boss phase indicator. |
| **Priority 4 — Secondary information** | Useful context, not decision-critical. | Score value, current Battle number / Difficulty Tier indicator (`07` Section 14), Run progress marker. |
| **Priority 5 — Decorative information** | Flavor only; never gameplay-relevant. | Enemy flavor name/portrait art, environment theme art, background treatment, ambient flavor text. |

**MASTER DESIGN RULE:** A Priority 5 element must never visually outrank (in size, motion, or contrast) any Priority 1–2 element on the same screen. This is the concrete enforcement of the Master Prompt's "never let decorative UI compete with gameplay readability."

---

## 4. Global Navigation Architecture

```text
                         ┌───────────────┐
                         │  Title/Home   │
                         └───────┬───────┘
             ┌───────────────────┼───────────────────┐
             ▼                    ▼                    ▼
      ┌─────────────┐     ┌──────────────┐     ┌──────────────┐
      │ Start Run   │     │  Settings    │     │ Tutorial/Help │
      └──────┬──────┘     └──────────────┘     └──────────────┘
             ▼
      ┌─────────────────┐
      │ Battle Start     │◄────────────────────────────┐
      │ (Section 11)      │                             │
      └────────┬──────────┘                             │
               ▼                                        │
      ┌─────────────────┐   pause tap   ┌─────────────┐  │
      │  BATTLE (HUD +   │──────────────►│   Pause     │  │
      │  Board + Tray)    │◄──────────────│  (Section 14)│  │
      └────────┬──────────┘   resume     └─────┬───────┘  │
               │                                 │ Restart/Exit
               │ BATTLE_VICTORY                    (confirm)
               ▼                                    ▼
      ┌─────────────────┐                    ┌──────────────┐
      │ Victory (Sec.12) │                    │ Title/Home    │
      └────────┬──────────┘                    └──────────────┘
               ▼
      ┌─────────────────┐
      │ Relic Selection  │
      │  (Section 10)     │
      └────────┬──────────┘
               └──────────────────────────────────────────┘  (loops to Battle Start for next Battle)

      RUN_DEFEAT (from BATTLE, any Battle)
               ▼
      ┌─────────────────┐
      │ Run Results /     │
      │ Defeat (Sec.13)    │
      └────────┬──────────┘
               ▼
      ┌─────────────────┐
      │  Title/Home       │
      └───────────────────┘
```

**DESIGN DECISION:** There is no separate "Run Start" screen distinct from Battle Start (Section 11) — starting a Run transitions directly into the first Battle's start sequence, avoiding an extra tap/screen per Master Vision Section 5.1 (Instant Clarity).

---

## 5. Screen Inventory

| Screen | Purpose | Entry | Exit | Primary Action | Secondary Actions | Critical Information |
|---|---|---|---|---|---|---|
| **Title/Home** | Entry point; no Run in progress. | App launch, or return from Defeat/Exit. | Start Run, Settings, Tutorial/Help. | Start Run | Settings, Tutorial/Help | Persistent Currency balance (`05` Table 23.5). |
| **Battle Start** | Transition into a new Battle (Section 11). | Run Setup, or post-Relic-Selection Transition. | Board Ready → Battle. | (none — auto-advancing sequence) | Skip (repeat players, Section 11) | Enemy identity, Primary/Bonus Objective, first Telegraph. |
| **Battle (HUD + Board + Tray)** | Core gameplay (`01` Section 4 state machine). | Battle Start complete. | Battle Victory, Run Defeat, or Pause. | Piece placement (drag-drop, `06`) | Pause | Board state, Tray, Enemy HP, Objective progress, Move Limit. |
| **Pause** | Suspend gameplay (`01` Section 19). | Pause tap during `ACTIVE`/`PIECE_SELECTED`. | Resume, Restart (confirm), Exit (confirm), Settings. | Resume | Restart Run, Exit to Menu, Settings, Audio/Haptics toggle | Current Battle is unaffected/frozen. |
| **Victory** | Battle Victory summary (Section 12). | `BATTLE_VICTORY`. | Auto-advances to Relic Selection. | Continue | (none) | Objective/Bonus completion state, reward summary. |
| **Relic Selection** | Choose one of 3 Relics (Section 10). | Post-Victory. | Confirmed choice → next Battle Start. | Choose | View Relic detail | 3 Relic options, current build context. |
| **Run Results / Defeat** | Run-end summary (Section 13). | `RUN_DEFEAT`. | Restart, Return to Menu. | Restart Run | Return to Menu | Cause of defeat, Run summary, currency earned. |
| **Settings** | Audio/haptics/accessibility toggles (Section 21). | Home or Pause. | Back. | (toggles) | — | Current toggle states. |
| **Tutorial/Help** | Replay onboarding tips / rules summary (Section 15). | Home or Pause. | Back. | — | — | Condensed rules reference. |

**MASTER DESIGN RULE:** No screen exists in this inventory for monetization (Section 1's Non-goal) — a future Layer B/monetization document must add a row here explicitly before any such surface may ship.

---

## 6. Gameplay HUD

**DESIGN DECISION, stated explicitly to prevent future drift:** There is **no Player Health element anywhere in the HUD.** This is not an omission — it is a binding inheritance of `00` Section 10 / `05` Section 12. Any future contributor proposing a player HP bar must first revise `01_GAMEPLAY_SPECIFICATION.md`.

| Element | Importance | Location | Size Priority | State Changes | Information Density |
|---|---|---|---|---|---|
| Enemy name | P5 | Top zone | Small | Static per Battle | Low |
| Enemy portrait/icon | P5 | Top zone | Medium | Idle → hit-reaction → defeat | Low |
| Enemy HP (bar + numeric) | P2 | Top zone, prominent | Large | Drains on `DamageEvent` (`05` Section 27) | Medium |
| Player health | **N/A — does not exist** | — | — | — | — |
| Primary Objective (name + progress) | P2 | Top zone, always visible | Medium–large | Progress updates per `07` Section 12 timing | Medium |
| Bonus Objective (name + progress) | P3 | Top zone, visually subordinate to Primary (Section 9) | Small, expandable | Same timing as Primary | Low (collapsed) / Medium (expanded) |
| Score | P4 | Top or side zone, minor | Small | Increments per `05` Section 8's Score value | Low |
| Combo feedback | P2 | Transient overlay near clear location | Scales with `L` (`06` Section 12) | Appears on multi-line Clear, fades | Low, transient |
| Run/Battle indicator | P4 | Top corner, minor | Small | Static per Battle | Low |
| Pause | P1 (interaction), always reachable | Top corner | Small, fixed | Disabled during resolution states (Section 14) | Low |
| Relic indicators | P3 | Side or bottom strip | Small icons, tap for detail | Appends on each Relic Selection | Low (collapsed) |
| Enemy Telegraph indicator | P2 | Adjacent to Enemy HP | Medium | Updates every `ENEMY_ACTION` (`01` Section 10) | Medium |
| Enemy Defense indicator | P3 | Adjacent to Enemy HP, only if non-zero (`05` Section 11) | Small | Static unless Battle-specific effect changes it | Low |
| Move Limit remaining | P2 | Near Objective, only if Battle has a Move Limit Modifier (`07` Section 8) | Medium | Decrements once per Turn (`07` Section 13) | Low |
| Boss phase indicator | P3 | Near Enemy HP, only on Boss Battles | Small segmented marker | Advances per HP-threshold phase (`07` Section 21) | Low |

---

## 7. Board/Screen Composition

```text
┌───────────────────────────────┐
│  TOP ZONE                       │  Enemy identity/HP, Objective(s), Telegraph, Pause
│  (P2/P3 HUD elements)            │
├───────────────────────────────┤
│                                   │
│  BOARD ZONE                       │  8×8 Board (03 Section 5) — dominant surface,
│  (P1 — dominant visual focus)      │  always square, centered, largest element on screen
│                                   │
├───────────────────────────────┤
│  FEEDBACK ZONE (optional,          │  Combo/Damage popups — transient, non-blocking,
│  overlays Board edge, never center) │  never centered over active placement area
├───────────────────────────────┤
│  BOTTOM TRAY ZONE                   │  3-Piece Tray (01 Section 6), drag origin,
│  (P1 — interaction surface)          │  respects bottom safe area (Section 17)
└───────────────────────────────┘
```

- **Battlefield zone:** Conceptually merged with the Board zone at Absolute MVP — there is no separate non-interactive "battlefield" visual layer distinct from the Board itself (Enemy is presented in the Top Zone, not as a separate arena behind the Board), keeping the Board the single dominant surface per the Master Prompt's overriding constraint.
- **Safe areas:** Top Zone content is padded below any notch/dynamic-island region; Bottom Tray Zone respects the home-indicator safe area on devices that have one.
- **Gesture exclusion areas:** A narrow margin along the extreme screen edges is excluded from Piece-drag hit-testing to avoid conflicting with OS edge-swipe (back/home) gestures — inherits `06_INPUT_DRAG_AND_DROP_UX.md`'s touch-model ownership; this document only mandates that the exclusion exists, not its exact pixel width.
- **Adapting to aspect ratios:** The Board Zone always renders as a perfect square sized to the smaller of (available width − margins) or (available height − Top Zone − Tray Zone); Top and Tray Zones absorb any extra vertical space on taller aspect ratios rather than stretching the Board non-square.

---

## 8. Enemy Presentation

| Aspect | Definition |
|---|---|
| **Identity** | Name + portrait/icon (P5) — flavor only, per `07` Section 9's naming caveat; carries no gameplay meaning by itself. |
| **HP** | Numeric value + proportional bar (P2), the single most important Enemy-side readout. |
| **Threat state** | Represented entirely by the Telegraph (Section 6) — never by a generic "threat level" abstraction, since `01` Section 5.6 requires every Enemy action to be concretely previewed, not vaguely signaled. |
| **Phase state** | Segmented HP-threshold markers (P3), only present on multi-phase Bosses (`07` Section 21's "Stone Golem"-style pattern) — absent entirely on standard Battles. |
| **Damage feedback** | Transient hit-reaction on the portrait + HP bar animating down, synchronized with the `DamageEvent` (`05` Section 27) — never delayed past the Turn it occurred in. |
| **Status effects** | At Absolute MVP, limited to the Enemy Defense indicator (Section 6) — Frozen/Blocked/Hazard-driven Enemy-side status presentation is deferred alongside their owning mechanics (`07` Section 8, "Later" rows). |
| **Defeat state** | A distinct, non-reused defeat animation/transition leading directly into the Victory screen (Section 12) — must be visually unambiguous from a mere damage-reaction. |

**Character art style is explicitly out of scope here** (Section 1) — a future Art Direction document owns it.

---

## 9. Objective Presentation

- **Primary Objective:** Always visible in the Top Zone from `PREPARING` onward (`07` Section 10) — never requires opening a menu.
- **Bonus Objective:** Visible but visually subordinate (Priority 3, Section 3) — shown as a collapsed chip that expands on tap for full detail, never competing with the Primary Objective's prominence.
- **Progress:** A live fraction/bar (`current / target`) updated per `07` Section 12's deterministic timing.
- **Success:** A distinct completion animation (checkmark or equivalent) at the moment `success_condition` becomes true, coincident with `BATTLE_VICTORY`.
- **Failure:** For categories with a `failure_condition` (Move Limit, Board Lock — `07` Section 5), a distinct but non-alarming failure state, consistent with Section 13's "do not shame the player" principle.
- **Hidden vs. visible information:** **There is no hidden-objective-information state in this game.** Per `07` Section 10's fairness rule, every Objective field (Section 4 of `07`'s schema) is visible from Battle start — this document defines no "fog of war" or progressive-reveal treatment for Objectives.

**MASTER DESIGN RULE:** An Objective must be fully understandable before the player's first Placement — this is a hard gate on the Battle Start sequence (Section 11), not merely a preference.

---

## 10. Relic Selection UX

- **Reward screen:** A full-screen modal, entered directly from Victory (Section 12), blocking Battle interaction until resolved (matches `01` Section 17's "mandatory, no skip/reroll" rule).
- **3-choice presentation:** Exactly 3 cards, laid out side-by-side on standard/large phones, stacked vertically with scroll on small phones (Section 17).
- **Per-card content:** Relic title (`display_name`), plain-language description of `effect`/`condition` (matching `05` Section 6's schema, written in Master Vision Section 5.3's "immediately understandable" register), and nothing else at Absolute MVP.
- **Rarity:** **Not implemented.** `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 14's 8-Relic launch pool has no rarity/tier field — all cards are presented with uniform visual weight; no card is styled as "rarer" than another.
- **Current-build context:** A small, always-visible strip (below or above the 3 cards) showing the player's currently-held Relics, so synergy (`05` Section 15) can be assessed before choosing — collapsed to icons only, expandable on tap.
- **Selection interaction:** Tap a card to select/highlight it (previewable, reversible); a separate "Choose" confirmation action locks it in — a deliberate two-step to satisfy Section 19's "accidental relic selection" prevention rule, since the choice is permanent for the Run (`01` Section 17).
- **Confirmation:** The "Choose" tap itself is the confirmation — no secondary "Are you sure?" dialog is added on top of it (that would violate Section 23's "too many confirmations" failure mode).
- **Transition to next Battle:** Immediate, into Battle Start (Section 11) for the next encounter.

---

## 11. Battle Start Experience

```text
Battle Loading → Enemy Reveal → Objective Reveal → Board Ready → First Input
```

- **Battle loading:** Brief, non-interactive; shows minimal placeholder (e.g., an Enemy silhouette) rather than a blank/spinner-only screen where feasible.
- **Enemy reveal:** Name/portrait/HP bar populate (Section 8).
- **Objective reveal:** Primary Objective card (and Bonus, if present) slides into the Top Zone (Section 9).
- **Board ready:** Board renders — empty, or with any Section 8 (`07`) pre-fill — followed immediately by the first Enemy Telegraph becoming visible (`01` Section 10's `PREPARING`-time requirement).
- **First input:** Tray populates with the first 3 Pieces; control passes to the player (`ACTIVE`, `01` Section 4).

**DESIGN DECISION:** The full reveal sequence plays on a player's **first encounter with a given Enemy archetype** (Section 8's flavor identity); on subsequent encounters with an already-seen archetype, an **abbreviated version** (Enemy/Objective appear near-simultaneously, shorter transition) plays instead, so experienced players are never repeatedly forced through the same multi-beat animation (Master Prompt's explicit constraint). **ASSUMPTION:** the exact abbreviation threshold ("first encounter" vs. "every encounter") is a placeholder pending playtesting feedback.

---

## 12. Victory Experience

```text
Enemy Defeat → Battle Victory → Objective Completion → Bonus Completion (if met) → Reward Presentation → Relic Selection
```

- **Enemy defeat:** Distinct defeat animation (Section 8), Enemy HP visibly at 0.
- **Battle victory:** A brief, clear victory beat/banner — not a lengthy cutscene.
- **Objective completion:** Primary Objective's success state (Section 9) shown explicitly.
- **Bonus completion:** If met, a separate, distinctly celebratory beat (visually distinguishable from mere Primary completion) acknowledging the supplementary reward (`07` Section 6's currency bonus).
- **Reward presentation:** A brief summary (currency, if Bonus met) immediately preceding Relic Selection — not a separate screen requiring an extra tap to dismiss.
- **Transition to Relic Selection:** Direct, no intermediate "Continue" screen beyond what Section 5's table lists (the Victory screen's own Continue action).

---

## 13. Defeat/Game Over UX

- **Board failure presentation:** When Board Lock (`03` Section 15/`01` Section 15) is detected, the Board itself gives a clear, calm signal (e.g., a brief highlight indicating no legal placement exists) before transitioning to the Run Results screen — this is a direct UX response to the Board Engine's `NoMovesAvailable` event (`03` Section 12).
- **Defeat presentation tone:** **Neutral, never punitive.** No shaming copy, no negative sound stinger disproportionate to the moment — directly per the Master Prompt's explicit instruction and Master Vision Section 16's "relief," not distress, framing.
- **Score/run information:** Battles won this Run, Boss(es) defeated, final Score (`05` Section 8), and currency earned (`05` Table 23.5) are all shown together on one Run Results screen.
- **What caused defeat:** Explicitly named — "Board Lock: no space for your remaining pieces" or "Move Limit reached" (`07` Section 5's `failure_condition` categories) — never a vague "You Lost" with no explanation, directly satisfying `01` Section 5.6's Fair Difficulty acceptance criterion.
- **Continue/revive:** **Does not exist.** `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 22 and `01` Section 3 both fix a Run at exactly one life with no mid-run continue at Absolute MVP — this screen must not present a "Continue?" option that doesn't exist.
- **Restart / Return to Menu:** The two primary actions, both one tap, neither requires confirmation (leaving the Run Results screen forward has no destructive consequence left to protect against — the Run is already over).

---

## 14. Pause UX

- **Layout:** Modal overlay on top of the frozen Battle screen (Section 5).
- **Resume:** Primary action, most prominent.
- **Restart Run:** Secondary, destructive — requires confirmation (Section 19).
- **Settings:** Secondary, non-destructive, opens the Settings screen (Section 5) without leaving Pause.
- **Exit to Menu:** Secondary, destructive — requires confirmation (Section 19); forfeits the current Run.
- **Audio / Haptics toggles:** Directly accessible from Pause (quick-access duplicates of Settings' fuller controls), per Master Vision Section 19's accessibility requirement that these be easy to reach mid-session.
- **Availability constraint:** Per `01_GAMEPLAY_SPECIFICATION.md` Section 19, Pause is only actually honored while in `ACTIVE`/`PIECE_SELECTED`. The Pause button remains tappable at all times (Section 6), but a tap during a resolution chain (`PLACEMENT_RESOLUTION` through `ENEMY_ACTION`) queues the pause and shows a brief "Pausing…" acknowledgment rather than either ignoring the tap silently or freezing gameplay mid-resolution.

---

## 15. New Player Onboarding

**DESIGN DECISION:** No separate tutorial mode, wall of text, or forced walkthrough screens. Onboarding is **embedded directly in Battle 1** (`07` Section 14's Tier 1 — Tutorial), teaching exactly four things: drag pieces, complete lines, damage the Enemy, choose a Relic.

| Trigger | Instructional copy (illustrative, not final) | Behavior |
|---|---|---|
| First Piece pickup | "Drag to place." | Short, non-blocking tooltip near the picked-up Piece. |
| First completed line | "Lines clear — and deal damage!" | Non-blocking tooltip, synchronized with the Combat feedback (Section 6). |
| First Enemy defeat, before Relic Selection first appears | A single short explanatory line on the Relic Selection screen itself (e.g., "Choose one — it stays with you all Run."). | One-time only; never repeats once seen. |
| (No fourth trigger needed) | The Move-drag-clear-relic loop is complete after the above three. | — |

- **Onboarding triggers:** First launch only; state is remembered so none of the above repeats on subsequent Runs.
- **Interactive tutorial behavior:** Tooltips are non-blocking wherever possible; at most one tap-to-dismiss is used across the entire sequence (the Relic Selection explanatory line, since that screen is already modal).
- **Skip behavior:** A single global "Skip Tutorial" affordance available from the very first tooltip; skipping suppresses all remaining onboarding copy for that install.
- **Replay/help access:** The Tutorial/Help screen (Section 5) offers a condensed, on-demand rules summary covering the same four concepts, for players who skipped or want a refresher — this is a static reference, not a re-triggered interactive walkthrough.

---

## 16. Interaction Hierarchy

| Tier | Definition | Examples |
|---|---|---|
| **Primary actions** | The single expected action per screen; always most visually prominent. | Piece drag-drop (Battle), Choose (Relic Selection), Resume (Pause), Start Run (Home). |
| **Secondary actions** | Available, subordinate. | Pause (Battle), View Relic detail (Relic Selection), Settings (Pause/Home). |
| **Destructive actions** | Discards Run progress or leaves a screen with unrecoverable consequence. | Restart Run, Exit to Menu (mid-run). |
| **Confirmation requirements** | Every destructive action requires exactly one explicit confirmation step (Section 19); no primary or secondary action does. |

---

## 17. Responsive Layout Rules

| Device class | Rule |
|---|---|
| **Small phones** | HUD density reduced first — Bonus Objective defaults to collapsed; padding/margins minimized; Board Zone retains full square sizing and touch-target minimums (Section 7) never shrink. |
| **Standard phones** | Baseline layout as specified in Sections 6–7. |
| **Large phones** | Additional breathing room; Objective descriptions may render fully expanded by default rather than collapsed. |
| **Tablets** | **OPEN DECISION / ASSUMPTION:** tablet support is not confirmed as launch scope by `00_MASTER_GAME_VISION.md`; if supported, this document tentatively favors a side-docked HUD (Top/Bottom zones migrate to left/right of a larger centered Board) over simply scaling up the phone layout, but this is not committed at Absolute MVP. |
| **Aspect ratio variance** | The Board Zone is always sized first (Section 7's square-fit rule); Top/Tray Zones absorb remaining space — never the reverse. |
| **Notches / rounded corners** | Top Zone content is inset below any notch/cutout region; Pause (Section 6) is never placed where a notch could occlude it. |
| **Safe areas** | Bottom Tray Zone respects the home-indicator safe area; no drag-drop hit-target is ever placed under a system gesture zone (Section 7). |

---

## 18. UX State Matrix

| Screen | Normal | Loading | Disabled | Success | Failure | Interrupted | Paused | Unavailable | First-use |
|---|---|---|---|---|---|---|---|---|---|
| **Battle** | `ACTIVE`, awaiting input | Battle Start sequence (Section 11) | Input disabled during resolution states (`01` Section 4) | `BATTLE_VICTORY` transition | `RUN_DEFEAT` transition | App backgrounded mid-resolution (`01` Section 20) — resumes automatically, no player-visible interrupted state | Pause overlay active (Section 14) | N/A | Onboarding tooltips active (Section 15) |
| **Relic Selection** | 3 cards shown, none chosen | N/A (instant) | Cards not yet tappable during entry animation | Choice confirmed, transitioning out | N/A (no failure state exists — a choice is always eventually made) | N/A | N/A | N/A | First-ever appearance includes the onboarding line (Section 15) |
| **Pause** | Overlay shown | N/A | N/A | Resume tapped | N/A | N/A | (is itself the paused state) | Unreachable during resolution states (Section 14) | N/A |
| **Victory** | Sequence playing (Section 12) | N/A | Not interactive until sequence completes | Full sequence shown | N/A | N/A | N/A | N/A | N/A |
| **Run Results / Defeat** | Summary shown (Section 13) | N/A | N/A | N/A | (is itself the failure-context screen) | N/A | N/A | N/A | N/A |
| **Home** | Default | App launch loading | N/A | N/A | N/A | N/A | N/A | N/A | First-ever launch may route straight into Battle 1 rather than lingering on Home (`07` Section 14 Tier 1) |

---

## 19. Error Prevention

| Risk | Prevention |
|---|---|
| Accidental purchases | **N/A** — no monetization surface exists (Section 1). |
| Accidental restart | Restart Run (Pause) requires one explicit confirmation dialog. |
| Accidental exit | Exit to Menu (Pause, mid-run) requires one explicit confirmation dialog. |
| Accidental relic selection | Two-step select-then-Choose interaction (Section 10) — a single stray tap cannot lock in a choice. |
| Accidental destructive actions generally | Only actions that discard Run progress require confirmation (Section 16); everything else (Pause/Resume, viewing detail panels) is freely reversible and unconfirmed, per Section 23's "too many confirmations" failure mode. |

---

## 20. Feedback Principles

- **Requires immediate feedback:** Placement legality (Ghost Preview, `06` Section 8), Line Clear, Damage dealt, Objective progress tick, Enemy Telegraph reveal — all within the same Turn they occur (`07` Section 12's timing).
- **Can be delayed:** Currency accrual animation (may batch/count up on the Run Results screen rather than update live), non-critical cosmetic flourishes.
- **May overlap:** Board Feedback and Combat Feedback may play concurrently, per `06_INPUT_DRAG_AND_DROP_UX.md` Section 9's existing allowance — this document does not add a new restriction beyond what `06` already specifies.
- **Must never obscure gameplay:** No feedback element may cover the Board or Tray during `ACTIVE` input (Section 3's Priority 1 rule) — transient popups render at Board edges or in the Feedback Zone (Section 7), never centered over the active placement area.

---

## 21. Accessibility Foundation

Inherits `00_MASTER_GAME_VISION.md` Section 19 directly, restated as UX-level (not settings-implementation-level) requirements:

- **Color-independent communication:** Enemy Telegraph, Objective failure states, and Move Limit warnings (Section 6, Section 9) must each carry an icon/text/shape cue in addition to any color cue — critical here specifically because the absence of a Player HP bar means Board-state pressure, Telegraph, and Objective proximity are the *only* danger signals in the game; none may depend on color perception alone.
- **Reduced motion:** A toggle (owned by Settings, Section 5) must be able to dampen non-essential animation (Section 11's reveal sequences, Section 12's victory beats) without removing any information those animations convey.
- **Haptics toggle:** Directly reachable from Pause (Section 14) in addition to Settings.
- **Audio toggles:** Independent music/SFX controls (Settings).
- **Text clarity:** Objective and Relic descriptions (Sections 9–10) are written in short, plain language by rule, not as a stretch goal.
- **Touch forgiveness:** Board and Tray touch targets meet comfortable minimums on every device class (Section 17) — this document does not specify an exact minimum size, deferring the numeric value to a future Layer B implementation document, but mandates that it is never reduced to fit more HUD density.

The full settings system (toggle persistence, exact controls list) is owned by a future Accessibility/Settings document (Section 25).

---

## 22. UI Consistency Rules

| Rule | Definition |
|---|---|
| **Alignment** | A single shared grid/margin system applies across every screen in Section 5's inventory. |
| **Spacing** | One reusable spacing scale, applied consistently — no screen invents its own spacing values. |
| **Hierarchy** | Section 3's Priority 1–5 system is applied identically on every screen; a Priority 1 element is always the largest/most prominent element present, regardless of which screen it appears on. |
| **Control placement** | Pause is always top-corner on every Battle-adjacent screen; the Primary action (Section 16) is always bottom-center or full-width-bottom, matching one-handed mobile thumb-reach conventions (Master Prompt principle 7). |
| **Terminology** | UI copy uses Master Terminology (`00` Section 22) without silent renaming — "Enemy," "Relic," "Objective," "Combo," "Turn" mean exactly what those documents define; no screen introduces a synonym for an existing canonical term. |
| **Icon use** | One icon = one meaning, enforced globally — no icon is reused for a different meaning on a different screen. |
| **Interaction behavior** | Drag-and-drop is reserved exclusively for Piece placement; tap is reserved exclusively for menu/selection/confirmation actions — no gesture is overloaded with two different meanings anywhere in the app. |

---

## 23. UX Failure Modes

| Failure Mode | Mitigation (cross-referenced) |
|---|---|
| Clutter | Priority hierarchy (Section 3) + Priority 5 elements kept minimal/optional. |
| Excessive overlays | No modal ever stacks on top of another modal (Pause never opens a second overlay in front of itself; inline messaging is used instead of nested dialogs). |
| Unclear objective | Section 9's "always visible, never hidden" rule. |
| Unclear combat | Section 20's mandatory immediate feedback + `06` Section 12's magnitude-scaled Combo/Damage presentation. |
| Poor hierarchy | Section 3 applied identically across all screens (Section 22). |
| Slow transitions | Section 11's skip/abbreviation rule for repeat Battle Starts; no mandatory transition may block input longer than necessary to convey its information. |
| Too many confirmations | Section 19's rule that only true destructive actions are confirmed — everything else is zero-friction. |
| Visual noise | Priority 5 elements never compete with Priority 1–2 zones for space or motion attention (Section 3's Master Design Rule). |

---

## 24. Acceptance Criteria

- No screen in the app displays a Player Health element, under any state.
- A first-time player can identify the Enemy's HP, the Primary Objective, and their current Tray within a few seconds of Battle start, without opening any menu.
- Pause is reachable in exactly one tap from any `ACTIVE`/`PIECE_SELECTED` Battle state, and is visibly (but non-blockingly) deferred if tapped during a resolution chain.
- Every destructive action (Restart Run, Exit to Menu mid-run) requires exactly one confirmation step — never zero, never more than one.
- Relic Selection always presents exactly 3 options and requires exactly one confirmed choice before the next Battle can begin.
- Enemy Telegraph is visible at least one full Turn before the corresponding action executes, on every single Battle without exception.
- The Run Results / Defeat screen always names the specific cause of defeat (Board Lock or Move Limit) — it never shows an unexplained "You Lost."
- No feedback element ever renders over the active Board/Tray area in a way that blocks the next legal input.
- Every onboarding tooltip (Section 15) is shown at most once per install and is fully skippable from its first appearance.

---

## 25. Cross-Document Contract

This document is the structural authority that the following future Layer B documents must implement without contradicting Sections 3–9 (Information Hierarchy, Navigation, Screen Inventory, HUD, Composition, Enemy/Objective Presentation):

- **Visual Design / Color System doc:** Owns final color palette, must respect Section 3's priority weighting and Section 21's color-independence requirement.
- **Block/Piece Art doc:** Owns Shape (`02`) rendering style; must preserve Ghost Preview legibility (`06` Section 8) referenced in Section 3/20 here.
- **Animation doc:** Owns exact easing/timing; must respect Section 11 and Section 23's "no slow mandatory transitions" rule.
- **VFX doc:** Owns particle/effect implementation for Combo/Damage/Victory beats (Sections 6, 12, 20); must never violate the "never obscure gameplay" rule (Section 20).
- **Audio doc:** Owns music/SFX asset design; must respect the audio toggle requirement (Section 21).
- **Content (flavor/copy) doc:** Owns final Enemy names, flavor text, and any copy beyond the plain-language functional strings this document mandates (Sections 9–10, 13, 15).
- **Accessibility/Settings doc:** Owns the full settings screen implementation and toggle persistence (Section 21).
- **Monetization doc (if ever created):** Must add a Section 5 screen-inventory row and a Section 1 scope revision here before any monetization surface may ship — none exists today.

This document must not be edited to embed content owned by the above; it exposes the structure they render into.

---

## 26. Final UX Contract

Block Battles' player-facing experience is organized around one non-negotiable fact: **the Board is the dominant surface, at all times, on every screen where it is present** (Section 3, Section 7). Every other element — Enemy state, Objective, Relics, Score, flavor — is layered around that surface in a fixed five-level priority system (Section 3) that applies identically across all nine inventoried screens (Section 5). The complete absence of a Player HP resource is a structural feature, not a gap: all tension in this game is communicated through Board pressure, Telegraphed Enemy actions, and Objective proximity (Section 9, Section 21) — never through a depleting player-side stat. Onboarding teaches exactly four concepts through embedded, skippable, non-blocking moments rather than a tutorial wall (Section 15). Every destructive action is confirmed exactly once; every non-destructive action is confirmed never (Sections 16, 19, 23). This is the complete information architecture that every future Layer B document renders into.

---

**End of `08_UI_UX_MASTER_SPECIFICATION.md`.**
