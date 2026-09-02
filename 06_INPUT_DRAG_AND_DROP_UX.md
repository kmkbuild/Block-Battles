# 06_INPUT_DRAG_AND_DROP_UX.md
## Block Battles — Input, Drag-and-Drop, and Touch UX

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`
**Document status:** Implementation-level interaction design

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23. It is the **Input & Controls doc** referenced by `01_GAMEPLAY_SPECIFICATION.md` Section 25 — it fills in the exact gesture/drag handling beneath the `PIECE_SELECTED`/`DRAGGING` states that Gameplay Spec Section 4 already names, without altering that state machine's transitions. It consumes `03_BOARD_ENGINE_AND_RULES.md`'s `validatePlacement`/coordinate contract as its sole legality authority and treats `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`'s Damage Pipeline as something to be *triggered*, never *previewed*, during a drag (Section 9).

---

## 1. Input Philosophy

**DESIGN DECISION:** Input exists to make Master Vision Section 5.1 (Instant Clarity) physically true, not just visually true. A player must be able to form a correct mental model of "where will this piece land" within their first drag, with zero instructional text, and every subsequent drag must confirm that model rather than surprise them.

Five binding philosophy rules, each traced to a Master Vision pillar and enforced by name throughout this document:

1. **Placement clarity outranks everything else on screen.** Directly restates `00_MASTER_GAME_VISION.md` Section 23, rule 1 ("Puzzle clarity always outranks visual effects") at the input layer — no Combat, Enemy, or Objective visual may compete with the dragged piece for attention (Section 9).
2. **Input is never blocked by feedback.** Restates Master Vision Section 23, rule 2 ("Player input must remain responsive at all times; no feedback effect may block or delay the next input") — every animation in this document is interruptible or non-blocking (Section 17).
3. **The board is read, not memorized.** The player should never need to remember where a piece "actually" is versus where it visually appears — Section 6's visual/logical distinction exists precisely to prevent this gap from ever being felt by the player, even though it exists in implementation.
4. **Forgiveness over precision-demand.** Master Vision Section 19's "Touch forgiveness: generous hit/drop targets and placement-snapping tolerant of imprecise mobile touch input" is treated as load-bearing, not aspirational — every threshold in this document (Sections 4, 7, 14) defaults toward generosity.
5. **Every gesture outcome is predictable before it happens.** Ghost Preview (Section 8) exists so that by the time the player releases, they already know the outcome — release (Section 10) should never produce a result the Ghost Preview didn't already promise.

---

## 2. Touch Model

**DESIGN DECISION:** Block Battles is a **single-active-touch** interaction model at Absolute MVP — only one Piece may be selected/dragging at any time, and only one finger's input is authoritative for board interaction while a drag is active.

- **Primary input:** touch (mobile, per Master Vision Section 2's mobile-first product definition). Mouse/trackpad input (for any desktop/testing context) maps onto the identical model, with "touch down" = mouse down, "touch move" = mouse move while held, "touch up" = mouse release — no separate desktop interaction spec is created, per Master Vision Section 5.8 (solo-developer feasibility).
- **Multi-touch:** a second concurrent touch starting while a drag is active is **ignored** for board interaction purposes (it does not cancel the active drag, does not start a second drag, and does not register as a tap elsewhere) — this prevents accidental two-finger gestures (e.g., a resting palm edge) from corrupting an in-progress Placement. This is the Section 14 (Accidental Input) baseline rule, stated here as the foundational touch model.
- **Touch target vs. visual size:** every interactive element (Tray Piece, its constituent cells) has a touch-target hitbox **larger than its rendered visual bounds** (Section 16, Accessibility) — generosity is a touch-model property, not a per-feature add-on.
- **Coordinate space:** all touch coordinates are captured in screen space and converted to Board logical coordinates only at the points defined in Section 6 — the touch model itself never assumes screen pixels map 1:1 to Board cells.

---

## 3. Pointer Lifecycle

The complete lifecycle a single touch/pointer passes through, mapped directly onto `01_GAMEPLAY_SPECIFICATION.md` Section 4's Battle State Machine (`ACTIVE → PIECE_SELECTED → DRAGGING → PLACEMENT_RESOLUTION`) without introducing any new Battle-level state — this document only elaborates the *pointer-level* sub-states beneath those.

| Pointer stage | Trigger | Battle state entered/confirmed | What happens |
|---|---|---|---|
| **Down** | Finger contacts a Tray Piece's touch target (Section 2). | Remains `ACTIVE` until pickup threshold met (Section 4). | Piece is marked as the candidate for pickup; no visual change yet beyond an immediate micro-feedback (Section 4). |
| **Selection** | Down is registered on a valid Tray Piece and the pointer has not yet moved past the pickup threshold. | `ACTIVE → PIECE_SELECTED` (Gameplay Spec Section 4). | The Piece is visually "picked up" in place (Section 4) — lifted, scaled, ready to follow the finger, but not yet moved from its Tray slot's screen position. |
| **Drag start** | Pointer moves past the pickup threshold (Section 4) while in `PIECE_SELECTED`. | `PIECE_SELECTED → DRAGGING` (Gameplay Spec Section 4). | The Piece begins following the pointer's visual position (Section 6); Ghost Preview (Section 8) becomes live the instant the pointer is over/near the Board. |
| **Drag** | Continuous pointer movement while held, still over-board or off-board. | Remains `DRAGGING`. | Visual position updates every frame (Section 6); Snap-to-Grid (Section 7) and Ghost Preview (Section 8) re-evaluate continuously. |
| **Hover** | A sub-state of Drag specifically describing the pointer being positioned over a candidate Board cell long enough for Snap-to-Grid to settle on a specific origin. | Remains `DRAGGING`. | The Ghost Preview locks to the current best-candidate origin (Section 7); this is a continuous re-evaluation, not a timed "hover delay" — see Section 7's exact snapping rule. |
| **Release** | Finger lifts while in `DRAGGING`. | `DRAGGING → PLACEMENT_RESOLUTION` (valid) or `DRAGGING → PIECE_SELECTED/ACTIVE` (invalid), per Gameplay Spec Section 4. | Section 10 defines the exact branching logic. |
| **Cancel** | Finger lifts while still in `PIECE_SELECTED` (never crossed the pickup threshold into `DRAGGING`), OR an explicit cancel gesture during `DRAGGING` (Section 10). | `PIECE_SELECTED → ACTIVE` (deselect) or `DRAGGING → ACTIVE` (drag cancel). | The Piece visually returns to its Tray slot (Section 11's return behavior, applied identically whether the cause was cancel or invalid release). |

**DESIGN DECISION:** This document treats "Selection" and "Drag start" as two distinct pointer sub-stages specifically because Gameplay Spec Section 4 already names `PIECE_SELECTED` and `DRAGGING` as two distinct Battle states with their own legal-transition table — this document's pointer lifecycle is deliberately isomorphic to that table, not a competing model.

---

## 4. Piece Pickup

- **Pickup threshold — DESIGN DECISION:** a small, fixed screen-space movement threshold (Section 17, Table 17.1) must be exceeded from the initial Down point before a pointer transitions `PIECE_SELECTED → DRAGGING`. Below this threshold, the pointer remains in `PIECE_SELECTED` (or resolves as a tap/no-op on release, Section 10). This threshold exists so that a light, imprecise finger-down (common on mobile) is never misread as an intentional drag start — directly serving Section 1's forgiveness rule.
- **Visual movement:** the instant Down registers on a valid Piece (before threshold is even crossed), the Piece plays an immediate small "lift" animation (scale + slight vertical offset, see below) — this happens at **Down**, not at threshold-crossing, so the player gets instant confirmation their touch registered (Section 1, rule 2: input is never left unacknowledged) even if they end up not dragging.
- **Scale:** the picked-up Piece renders at a modestly increased scale (Section 17, Table 17.1) relative to its Tray-resting size — large enough to read as "lifted off the tray" but not so large that it obscures more Board context than necessary once dragged over the grid.
- **Offset:** the Piece's rendered position during Drag is offset **upward** from the raw pointer position by a fixed vertical amount (Section 5) — never rendered directly centered under the fingertip.
- **Responsiveness:** pickup visual state (lift, scale) must apply within a single frame of Down registering — there is no eased "ramp up" to the lifted state that could read as input lag (Section 17's latency targets apply here specifically).

---

## 5. Finger Occlusion

**DESIGN DECISION:** the dragged Piece is always rendered **above and offset from** the finger's actual contact point, never directly beneath it, so the player can see both their fingertip's general area and the Piece's exact cells simultaneously.

- **Vertical offset:** a fixed screen-space offset (Section 17, Table 17.1) lifts the Piece's rendered anchor point above the raw touch Y-coordinate — large enough that an average adult fingertip does not visually cover any part of the Piece on a typical phone screen, small enough that the offset doesn't make the Piece feel disconnected from the touch.
- **The offset is constant in screen space**, not Board-cell space — it does not scale with Piece size, so a 1-cell Piece and a 3×3 Piece both clear the fingertip by the same visual margin (avoiding the situation where a large Piece's bottom-most cells are still occluded).
- **Ghost Preview cells (Section 8) are drawn on the Board itself**, independent of the offset dragged-Piece rendering — the player's primary placement feedback loop is "look at the Ghost Preview on the grid," not "look at the floating Piece under my finger," which is what makes the offset's exact tuning non-critical to correctness (it is a comfort/clarity feature, not the sole source of placement truth).

---

## 6. Drag Position

**DESIGN DECISION — two distinct, explicitly named position concepts, never conflated in implementation or in this document's own language:**

1. **Visual position:** the screen-space coordinate at which the dragged Piece is currently rendered — continuous, sub-pixel, following the pointer with the Section 5 offset applied. Visual position updates every frame and has no relationship to Board cell boundaries.
2. **Logical board position:** the discrete `(boardRow, boardCol)` origin candidate — the same coordinate space `03_BOARD_ENGINE_AND_RULES.md` Section 2 defines — that the current visual position *resolves to* via Snap-to-Grid (Section 7). Logical board position is what `validatePlacement` (Board Engine Section 4) is ever called with; it is never called with a raw visual/pixel position.

**The conversion boundary:** visual position is converted to a logical board position candidate on every frame during `DRAGGING`, using the resolution rule in Section 7. This conversion is **read-only against the Board** — evaluating a candidate origin via `validatePlacement` for Ghost Preview purposes (Section 8) never mutates Board state, matching `03_BOARD_ENGINE_AND_RULES.md` Section 4's explicit read-only guarantee for validation calls. Only a successful Release (Section 10, Section 12) ever calls `commitPlacement`.

**Why this distinction is load-bearing:** it is what allows the Ghost Preview (Section 8) to always show the player the *exact* outcome of releasing right now, even though the finger itself is never precisely over any single cell — the player's mental model can stay entirely in "does the ghost look right," never needing to reconstruct where their finger literally is relative to the grid (Section 1, rule 3).

---

## 7. Snap-to-Grid

- **Candidate coordinate — DESIGN DECISION:** on every frame during `DRAGGING`, the current visual position (Section 6) is converted to a candidate logical origin by mapping the Piece's *anchor cell* (its local `(0,0)`, per `02_SHAPE_LIBRARY.md` Section 2's normalized origin) to whichever Board cell's screen-space bounds currently contain the offset visual anchor point. This is a direct nearest-cell mapping, not a physics-based or velocity-based prediction.
- **Snapping:** once a candidate origin is computed, the Ghost Preview (Section 8) renders the Piece's full `local_coordinates` set at that candidate origin, snapped exactly to Board cell boundaries — there is no "partial" or "between-cells" render state ever shown; the Piece is always shown as if already resting in whole cells, even mid-drag.
- **Edge behavior:** if the computed candidate origin would place any part of the Piece out-of-bounds (per `03_BOARD_ENGINE_AND_RULES.md` Section 2's per-cell bounds check), the candidate origin is **clamped** to the nearest in-bounds origin along the violated axis/axes, rather than left unresolved — the Ghost Preview always shows a concrete candidate placement (even if that candidate will show as `INVALID` per Section 8, e.g. due to occupancy) rather than disappearing or freezing at the boundary. This keeps Section 1 rule 5 (predictable outcome before release) true even at the grid's edges.
- **Movement:** the candidate origin updates continuously and immediately as the visual position changes — there is no hysteresis, debounce, or "commit to nearest cell after settling" delay. A fast, sweeping drag across the Board updates the Ghost Preview at the same frame rate as the visual drag itself (Section 17's latency targets apply identically to snap updates and to raw visual-position updates).

---

## 8. Ghost Preview

A translucent rendering of the dragged Piece's full shape at its current candidate logical origin (Section 6–7), shown directly on the Board grid throughout `DRAGGING`.

**DESIGN DECISION — four distinct visual states, each unambiguous and never relying on color alone (Section 16):**

| State | Trigger | Visual treatment (conceptual — exact styling owned by a future Art Direction doc) | Icon/pattern cue (non-color-dependent) |
|---|---|---|---|
| **Valid** | `validatePlacement(shape, candidateOrigin, board) == VALID` (`03_BOARD_ENGINE_AND_RULES.md` Section 4). | Solid-ish translucent fill in the Piece's normal color family, calm/affirming treatment. | A subtle "ready" outline weight distinct from the invalid states' outline. |
| **Invalid — Occupied** | `validatePlacement` returns `INVALID: CELL_OCCUPIED` for at least one cell. | Distinctly different translucent treatment (e.g., a cross-hatch or dimmed fill) from Valid — never merely a color swap, since color alone must not carry the meaning (Section 16). | A small blocked-cell glyph/pattern overlaid on the specific occupied cell(s) that caused the failure, not just a uniform whole-Piece treatment — this tells the player *which* cell is the problem. |
| **Invalid — Out of Bounds** | The clamped candidate origin (Section 7) still results in `validatePlacement` returning `INVALID: OUT_OF_BOUNDS` for at least one cell (can occur at Board edges/corners for larger Pieces). | Same non-color-dependent "invalid" family treatment as Occupied, but the specific off-board cells are rendered as truncated/clipped at the Board boundary rather than overlaid with the occupied-glyph, so the player can tell the two invalid reasons apart. | A clipped/truncated edge render, distinct from the Occupied glyph. |
| **No Board contact** | The pointer's visual position (Section 6) is not currently over or near the Board at all (e.g., still within the Tray area). | No Ghost Preview rendered on the Board at all. | N/A — absence of a preview is itself the signal; the dragged Piece under the finger (Section 4–5) is the only visible feedback in this state. |

**DESIGN DECISION:** Occupied and Out-of-Bounds are visually distinguished from each other, not merged into one generic "invalid" state, because they call for different corrective player actions (move to a different spot vs. move fully onto the board) — collapsing them would reduce, not increase, clarity, contradicting Section 1's core philosophy.

The Ghost Preview is the **sole real-time placement-legality signal** the player receives during `DRAGGING` — no other UI element (Section 9) is permitted to also signal legality, to avoid split attention.

---

## 9. Combat Feedback During Placement

**DESIGN DECISION — explicit SHOULD / SHOULD NOT list, binding on every system this document depends on:**

**SHOULD remain active/visible, unchanged, during `DRAGGING`:**
- The current Enemy HP bar/value, Enemy sprite, and any already-shown Telegraph (`01_GAMEPLAY_SPECIFICATION.md` Section 10) — visible but **static**; nothing about them animates or changes state while a drag is in progress.
- The Board itself, including any already-placed Blocked/Frozen/Hazard cells (`03_BOARD_ENGINE_AND_RULES.md` Section 3.2, once implemented) — these are part of the puzzle surface the Ghost Preview must be evaluated against, so they must remain visible and legible throughout.
- Score/Combo-related HUD elements, in their pre-Turn resting state — visible, but not yet updating (there is nothing to update until Release/Commit, Section 12).

**SHOULD NOT trigger, animate, or otherwise draw attention during `DRAGGING`:**
- **No Enemy idle animation escalation, attack wind-up, or reactive animation** may play in response to drag movement, hover position, or Ghost Preview state — the Enemy is inert from the player's input perspective until `COMBAT_RESOLUTION`/`ENEMY_ACTION` actually occurs (Gameplay Spec Section 4), which never happens mid-drag.
- **No Damage Pipeline preview number** (projected `FinalDamage`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 9) is computed or shown during `DRAGGING` — Damage is only ever computed after a real `LinesCleared` event post-Release (Section 12); showing a "you would deal X damage" preview would require running the full pipeline speculatively against a hypothetical commit, adding both technical complexity and a second competing signal to the Ghost Preview, violating Section 1's single-source-of-truth rule for placement feedback.
- **No sound effect or haptic pulse tied to Enemy state** may fire during `DRAGGING` — the only sounds/haptics permitted mid-drag are the Section 4 pickup acknowledgment (one-time, on Down) and continuous drag-following audio, if any, which must be Piece-sourced, never Enemy-sourced.
- **No screen-wide flash, shake, or color-grade shift** referencing Enemy/Combat state may occur mid-drag — these are reserved exclusively for post-Release Combat Feedback (Section 12), where they are contextually earned and cannot be mistaken for placement guidance.

**Rationale, restated from the Master Prompt's own instruction:** "Combat and objectives must NEVER compromise the clarity of placement" and "Do not make the enemy distract the player" are treated as hard constraints, not tuning preferences — this section's SHOULD NOT list is exhaustive of every plausible Combat-layer feedback surface this document's dependencies define (Enemy HP/Telegraph from Gameplay Spec Section 10–11, Damage Pipeline from Scoring doc Section 9), specifically so no future document can introduce a violation by adding a new Combat surface without also amending this list.

---

## 10. Release Behavior

Exact behavior for every release condition, branching directly off Gameplay Spec Section 4's `DRAGGING → {PLACEMENT_RESOLUTION | PIECE_SELECTED/ACTIVE}` transition:

| Release condition | Candidate origin's `validatePlacement` result | Resulting Battle state | Behavior |
|---|---|---|---|
| **Valid release** | `VALID` | `DRAGGING → PLACEMENT_RESOLUTION` | Section 12's full feedback flow triggers; the Piece is consumed from the Tray. |
| **Invalid release — Occupied** | `INVALID: CELL_OCCUPIED` | `DRAGGING → PIECE_SELECTED` (or `ACTIVE` if released off the Tray-return zone entirely, Section 11) | Section 11's return behavior triggers; Piece is not consumed. |
| **Invalid release — Out of Bounds** | `INVALID: OUT_OF_BOUNDS` | `DRAGGING → PIECE_SELECTED`/`ACTIVE` | Same as above — Section 11 does not distinguish return behavior by invalid *reason*, only the live Ghost Preview does (Section 8). |
| **Release with no Board contact** (finger lifts while pointer is over the Tray area or elsewhere off-board) | N/A — no candidate origin was ever computed (Section 8, "No Board contact" state) | `DRAGGING → ACTIVE` (deselect) | Piece returns to Tray identically to an invalid release (Section 11); this is not treated as an error state, simply a non-attempt. |
| **Cancel gesture** (e.g., a defined off-board "drop zone" or a platform-level gesture such as an edge-swipe-back, if the platform reserves one) | N/A | `DRAGGING → ACTIVE` | Identical return behavior to "no Board contact" release — cancellation is never a punished action. |
| **Release during `PIECE_SELECTED`** (never crossed pickup threshold, Section 4) | N/A | `PIECE_SELECTED → ACTIVE` | Treated as a tap-deselect; Piece settles back to its resting Tray pose with the reverse of the Section 4 lift animation, no error feedback (this was never an attempted placement). |

**DESIGN DECISION:** every non-valid release path converges on the same visual return behavior (Section 11) — the player is never shown a harsher penalty for an out-of-bounds attempt versus an occupied-cell attempt versus simply changing their mind mid-drag. This is a deliberate forgiveness choice (Section 1, rule 4): an invalid attempt costs the player nothing but the moment itself.

---

## 11. Invalid Placement

- **Return behavior:** the Piece animates back from its release visual position to its original Tray slot position, over a short, non-blocking duration (Section 17) — the animation is purely cosmetic continuity, not a delay gating the next input; the player may immediately pick up the same or a different Tray Piece before the return animation finishes (Section 1, rule 2).
- **Visual feedback:** at the instant of an invalid release, the Ghost Preview's last-shown Invalid state (Occupied or Out-of-Bounds, Section 8) briefly holds/pulses once before the Piece begins its return animation — this gives a single clear "this is why it failed" beat rather than the Piece simply vanishing and reappearing in the Tray with no acknowledgment.
- **Sound concept:** a short, low-emphasis negative-feedback sound (conceptually: a soft "thud" or "denied" tone, not harsh or punishing in tone) — distinct from the Valid Placement sound (Section 12) but deliberately not aversive, since invalid attempts are an expected, normal part of exploring the board (Section 1, rule 4) and should not feel like a mistake being scolded.
- **Haptic concept:** a single short, light haptic pulse (conceptually: one "tap"-weight impact, not a buzz or sustained vibration) — distinguishable from the Valid Placement haptic (Section 12) by duration/pattern rather than intensity, so repeated invalid attempts during exploratory dragging never feel harsh even if triggered often.

---

## 12. Valid Placement

Conceptual feedback flow, exactly as specified by the Master Prompt, with each stage's owning document identified:

```text
Release
 → Confirm      — the Ghost Preview's Valid state (Section 8) is the confirmation; no separate
                   "are you sure" step exists — release itself is the confirming action, since
                   the Ghost Preview already showed the exact outcome before release (Section 1, rule 5)
 → Commit        — commitPlacement(shape, origin, board) executes (03_BOARD_ENGINE_AND_RULES.md
                    Section 5); Battle state PLACEMENT_RESOLUTION → LINE_CLEAR_RESOLUTION
                    (Gameplay Spec Section 4); this document's Piece visually "locks" into place
                    at the exact Ghost Preview position with zero positional snap/jump, since the
                    Ghost Preview and the committed position are, by construction (Section 7),
                    identical coordinates
 → Board Feedback — 03_BOARD_ENGINE_AND_RULES.md Section 12's LinesCleared/CellsCleared events
                     (if any) drive a Board-layer visual response (line-clear flash/collapse,
                     owned by a future Juice/VFX doc); this document's obligation is only to ensure
                     this feedback begins immediately after Commit with no input-blocking delay
                     (Section 1, rule 2) — the player may begin their next pickup (Section 4) while
                     this animation is still playing
 → Combat Feedback — 05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md Section 9's DamageEvent (if L≥1)
                      drives Enemy HP bar movement / Damage number popup (owned by a future Juice/VFX
                      doc); this is the FIRST point in the entire flow where any Combat-layer visual
                      is permitted to react (directly enforcing Section 9's SHOULD NOT list — nothing
                      Combat-related reacts before this stage)
 → Objective Evaluation — 01_GAMEPLAY_SPECIFICATION.md OBJECTIVE_EVALUATION state runs; if it
                           produces a victory/defeat transition, that transition's own feedback
                           (owned by Gameplay Spec Section 4 / a future UI doc) begins only after
                           this flow's Combat Feedback stage has begun, never interrupting it
```

- **Sound concept (Valid):** a distinctly positive, higher-clarity confirmation tone than the Invalid sound (Section 11) — conceptually brighter/more resolved, with its intensity/layering scalable by `L` (more lines = fuller sound, matching `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 6/8's escalating value structure) so Master Vision Section 5.4's "players can distinguish clear magnitude by feedback alone" is true at the audio layer specifically, not just visually.
- **Haptic concept (Valid):** a firmer, single confirming pulse distinct from Invalid's lighter tap; for `L≥3` ("Full Combo," `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 8) a distinct escalated haptic pattern (e.g., a short double-pulse) is used rather than simply a longer single pulse, so combo-tier haptics remain a recognizable *pattern*, not just "more of the same," in line with Section 16's non-intensity-dependent accessibility stance.
- **DESIGN DECISION:** every stage in this flow is strictly sequential in *trigger order* but not necessarily in *animation completion* — Board Feedback and Combat Feedback may visually overlap/run concurrently once both have started (their triggers are sequential; their animations are not required to be), keeping the whole flow feeling fast rather than a rigid waterfall, per Section 1 rule 2.

---

## 13. Rapid Interaction

**DESIGN DECISION:** the player may pick up a new Tray Piece the instant the previous Turn's `PLACEMENT_RESOLUTION → ... → OBJECTIVE_EVALUATION → ENEMY_ACTION → ACTIVE` loop (Gameplay Spec Section 4) re-enters `ACTIVE`, even if Section 12's Board/Combat Feedback animations are still visually resolving — animation completion is never a gate on the next pointer Down (Section 1, rule 2).

- **Queued input is not buffered/replayed** — if a Down occurs before `ACTIVE` is re-entered (e.g., during `ENEMY_ACTION`'s brief resolution), that Down is simply ignored (not queued for later execution); the player must re-touch once `ACTIVE` is live. This avoids surprising "delayed" input execution, which would itself violate predictability (Section 1, rule 5) more than a dropped early touch would.
- **Back-to-back Placements** (all 3 Tray Pieces placed in rapid succession) are fully supported — each Placement independently runs the full Section 12 flow; there is no artificial pacing/cooldown inserted between Turns beyond whatever `ENEMY_ACTION`'s own resolution requires (owned by Gameplay Spec Section 10, not this document).
- **Rapid re-pickup of the same Piece slot after a Board Lock partial state** (e.g., 2 of 3 Tray Pieces placed, the 3rd has no legal placement — Gameplay Spec Section 21's "dead Piece" edge case) is handled identically to any other pickup — the Ghost Preview will simply never show a Valid state anywhere on the Board for that Piece (Section 8), which is itself sufficient player-facing signal; no special "this piece is dead" messaging is triggered by this document (that determination belongs to `03_BOARD_ENGINE_AND_RULES.md` Section 10's Board Lock check, evaluated only across the *whole* Tray, not per-Piece).

---

## 14. Accidental Input

- **Multi-touch suppression:** per Section 2, a second touch during an active drag is ignored entirely — it neither cancels nor interferes with the first touch's drag.
- **Edge-of-screen / palm-rejection zone:** the Tray area and Board area both maintain a small inset margin (Section 17) from the physical screen edge where Down events are not registered as Piece-pickup candidates, reducing accidental pickups from a resting palm or grip during one-handed phone use.
- **Micro-jitter tolerance:** the pickup threshold (Section 4) doubles as accidental-input protection — a Down followed by sub-threshold movement and release is treated as a tap-deselect/no-op (Section 10), never as an attempted (and failed) drag, so a shaky or imprecise tap never produces Invalid Placement feedback (Section 11) it didn't actually attempt.
- **Accidental drag off-screen:** if the pointer's visual position moves beyond the playable screen bounds entirely (e.g., a drag gesture that continues past the device edge in a way the OS still reports coordinates for), the candidate origin clamps per Section 7's edge behavior exactly as it would at the Board's own edge — there is no separate off-screen-specific rule.
- **Rapid double-tap on a Tray Piece:** resolves as two independent Selection/Cancel cycles (Section 3) if both are sub-threshold, never as a special "double-tap" gesture — Block Battles defines no double-tap-triggered behavior at Absolute MVP (e.g., no "double-tap to auto-place"), keeping the gesture vocabulary minimal per Master Vision Section 5.8.

---

## 15. Pause / Background Behavior

Directly implements `01_GAMEPLAY_SPECIFICATION.md` Section 19 (Pause/Resume) and Section 20 (Backgrounding) at the pointer-input layer specifically:

- **Pausing mid-drag is not permitted** — matching Gameplay Spec Section 19's rule that pausing is only honored while `ACTIVE` or `PIECE_SELECTED`, a pause request received during `DRAGGING` is queued and only takes effect once the current pointer lifecycle reaches a resting state (Release resolved, Section 10, and any resulting Turn fully processed) — this document does not invent a mid-drag pause; it simply confirms no pointer-lifecycle stage in Section 3 is ever interrupted by one.
- **Backgrounding mid-drag:** if the app is backgrounded while a pointer is mid-`DRAGGING` (the OS delivers no further touch-move/touch-up events), on foreground return the drag is **not resumed** — the incomplete gesture is treated as an implicit cancel (Section 10's cancel-gesture row), and the Piece returns to its Tray slot (Section 11's return behavior) the instant the app resumes. This is a deliberate simplification: attempting to resume a stale, potentially-stale-coordinate drag across a backgrounding event risks placing a Piece somewhere the player never actually intended, which would violate Section 1's predictability rule far more than simply asking the player to re-attempt the drag.
- **This does not conflict with Gameplay Spec Section 20's mid-resolution autosave guarantee** — that guarantee applies to a Turn already past `PLACEMENT_RESOLUTION` (a committed Placement whose downstream resolution chain must complete on resume); a mid-`DRAGGING` gesture, by definition, has not yet committed anything (Section 6, Section 12), so there is no partially-resolved Turn state to protect here — only an incomplete *gesture*, which this document's own rule (implicit cancel) fully and simply resolves.

---

## 16. Accessibility

- **Touch target sizing:** every Tray Piece's touch target (Section 2) meets or exceeds standard mobile platform minimum touch-target guidance regardless of the Piece's rendered visual size — a `SHP_001` Single Block's touch target is padded well beyond its 1-cell visual footprint so it remains as easy to pick up as any larger Piece (directly serving Master Vision Section 19's "generous hit... targets").
- **Reduced motion:** every non-essential animation defined in this document (pickup lift/scale — Section 4; return-to-tray tween — Section 11; Board/Combat Feedback flourish — Section 12) has a reduced-motion variant that shortens or removes easing/scale/travel distance while preserving the *state change itself* (a Piece still visibly moves from A to B, just with less embellishment) — no animation in this document is ever the sole carrier of essential information (Ghost Preview state, Section 8, is always also communicated via the non-color pattern cues already specified there), so reduced motion never causes information loss.
- **Haptics:** every haptic cue in this document (Sections 4, 11, 12) is an **enhancement, never a sole information channel** — every haptic-paired event also has a simultaneous visual and/or audio cue, so a player with haptics disabled (platform-level setting) loses zero placement-relevant information, only the tactile flourish.
- **Color dependence:** explicitly ruled out already in Section 8's Ghost Preview table — Valid/Occupied/Out-of-Bounds states are each defined with a non-color pattern/glyph cue in addition to any color treatment, satisfying the Master Prompt's explicit accessibility requirement directly at the point where color-only signaling would otherwise be most tempting (a simple green/red Ghost Preview).
- **Motion sensitivity:** the continuous, high-frequency Ghost Preview/Piece-following updates (Sections 6–7) are visually smooth but deliberately **non-parallax and non-camera-based** — nothing in this document's interaction model moves the camera, zooms the Board, or introduces screen-shake during normal placement (screen-shake, if any, is reserved for a post-Commit Combat Feedback flourish, Section 12, owned by a future Juice/VFX doc, and even there must respect a platform/in-app reduced-motion setting) — this keeps the core drag loop safe for motion-sensitive players by construction, not by opt-out.

---

## 17. Latency Targets

Practical, testable targets — not aspirational language — for every time-sensitive interaction this document defines:

**Table 17.1 — Interaction Timing and Sizing Targets**

| Parameter | Target | Rationale |
|---|---|---|
| Down → pickup visual acknowledgment (Section 4) | Within 1 rendered frame (~16ms at 60fps) | Section 1 rule 2 — input must never feel unacknowledged. |
| Pickup movement threshold (Section 4) | A small, fixed screen-space distance (exact px/dp value owned by a platform-specific implementation doc) — deliberately generous, tuned toward "forgiving" per Master Vision Section 19 | Prevents accidental drag-start from a light tap. |
| Visual position update rate during `DRAGGING` (Section 6) | Every rendered frame, no throttling | A dropped/throttled frame here reads directly as input lag. |
| Ghost Preview re-evaluation rate (Section 7–8) | Matches visual position update rate exactly (no separate, slower cadence) | Prevents the Ghost Preview ever visibly "catching up" to the finger. |
| Release → Commit (valid placement, Section 12) | Within 1–2 rendered frames | The Piece must lock into place with no perceptible delay after release. |
| Invalid-release return-to-tray animation duration (Section 11) | Short (a few hundred ms), non-blocking | Long enough to read as intentional feedback, short enough to never gate the next input (Section 13). |
| Vertical finger-occlusion offset (Section 5) | A fixed screen-space value clearing an average adult fingertip's visual footprint on a typical phone display | Tuned for legibility, not for any Piece-size-dependent scaling. |
| Palm-rejection/edge inset margin (Section 14) | A small fixed inset from each screen edge within the Tray/Board interactive zones | Balances accidental-input protection against not shrinking usable touch area meaningfully. |
| Haptic pulse duration (Sections 11–12) | Short, single discrete pulses (Invalid) vs. a distinguishable short pattern (Valid `L≥3`) — never a sustained/looping vibration | Keeps haptics as punctuation, never a distraction (Section 9's "not distracting" principle extended to touch feedback). |

**DESIGN DECISION:** exact millisecond/pixel values are intentionally left to a platform-specific implementation doc rather than hardcoded here, since optimal values legitimately differ by device class and OS — this document fixes the *relationships* (must-be-same-frame, must-not-block, must-be-short) which are the actual feel-defining constraints, matching this document's own Section 1 philosophy over prescribing numbers that would need per-platform re-tuning anyway.

---

## 18. Input State Machine

The pointer-level state machine, nested inside Gameplay Spec Section 4's Battle State Machine exactly as Section 3 (Pointer Lifecycle) described — presented here as a single canonical diagram for implementation reference:

```text
[ACTIVE] (no pointer active)
   │ Down on valid Tray Piece touch target
   ▼
[PIECE_SELECTED] (pointer down, sub-threshold movement)
   │                                    │
   │ movement > pickup threshold        │ Release (tap) OR sub-threshold hold-release
   ▼                                    ▼
[DRAGGING] ◄────────────┐          [ACTIVE] (deselect, no-op — Section 10)
   │  continuous move    │
   │  (visual position    │
   │   updates, Ghost      │
   │   Preview re-evals,   │
   │   Section 6–8)         │
   │◄──────────────────────┘
   │
   ├── Release, candidate VALID ──────────► [PLACEMENT_RESOLUTION] (Gameplay Spec Section 4)
   │                                          → Section 12 flow → eventually returns to [ACTIVE]
   │
   ├── Release, candidate INVALID ─────────► [ACTIVE] or [PIECE_SELECTED]
   │                                          (Section 10/11 return-to-tray behavior)
   │
   ├── Release, no Board contact ──────────► [ACTIVE]
   │                                          (Section 10/11 return-to-tray behavior)
   │
   ├── Cancel gesture ─────────────────────► [ACTIVE]
   │                                          (Section 10/11 return-to-tray behavior)
   │
   └── App backgrounded mid-drag ───────────► [ACTIVE] (on foreground resume, Section 15)
                                                (implicit cancel, Section 10/11 return-to-tray behavior)
```

**DESIGN DECISION:** this diagram introduces zero Battle-level states beyond what `01_GAMEPLAY_SPECIFICATION.md` Section 4 already defines — `PIECE_SELECTED` and `DRAGGING` are that document's own states; every arrow in this diagram terminates in a state that document's own transition table already permits (Gameplay Spec Section 4's Legal Next States column), confirming this document's full compliance with that authority.

---

## 19. Edge Cases

| Edge Case | Resolution |
|---|---|
| Player drags a Piece over the Board, then back into the Tray area, then back onto the Board, without releasing | Ghost Preview simply follows Section 7–8's continuous re-evaluation the whole time; re-entering the Board area re-shows the Ghost Preview exactly as if the drag had started there — no state is "remembered" from the earlier Board visit. |
| Player releases exactly on a Board cell boundary (ambiguous nearest-cell) | Section 7's candidate-coordinate mapping resolves ties deterministically (e.g., consistent rounding direction) so the same release point always produces the same candidate origin — this document does not leave boundary ties undefined, since undefined behavior here would directly violate Section 1 rule 5. |
| A Piece's Ghost Preview is Valid, but between the last preview update and the actual Release event, the Board changes (theoretically impossible at Absolute MVP since only the dragging player's own future commit can change the Board mid-Turn, per `03_BOARD_ENGINE_AND_RULES.md` Section 13's determinism/no-interleaving guarantee) | Not reachable at Absolute MVP — no Enemy action or other system can mutate Board state mid-`DRAGGING` (Section 9's SHOULD NOT list already forbids Enemy reaction during drag); this row is recorded to confirm the case is structurally excluded, not silently unhandled. |
| Player attempts to drag a Piece that has zero legal placements anywhere on the current Board (a "dead" Piece, Gameplay Spec Section 21) | Fully draggable as normal; Ghost Preview will show Invalid at every candidate origin the player tries; Release anywhere returns the Piece to Tray (Section 11) — no special dead-Piece messaging (Section 13). |
| Very fast "flick" gesture where the pointer moves a large screen distance between two consecutive frames | Section 7's candidate coordinate is computed fresh from the latest visual position each frame — a large single-frame jump simply produces a large single-frame jump in candidate origin; there is no interpolation/tunneling-prevention logic required, since Ghost Preview evaluation has no concept of "passing through" cells, only "currently over." |
| Screen rotation or resize mid-drag (if the platform permits it) | Treated identically to backgrounding mid-drag (Section 15) — implicit cancel, Piece returns to Tray, rather than attempting to re-project a stale drag position into a new layout. |
| Player holds a Piece in `DRAGGING` indefinitely without releasing (idle mid-drag) | No timeout — the drag remains live indefinitely; Gameplay Spec's Pause/Resume rules (Section 15 of this document) are the only mechanism that can end an idle drag externally (backgrounding), never a drag-duration timer, since an arbitrary timeout would violate Section 1's predictability rule (the player didn't do anything to trigger a sudden cancellation). |

---

## 20. Manual Feel Checklist

Separated per the Master Prompt's explicit instruction — functional correctness is necessary but not sufficient; this section exists because a build can pass every Section 21 acceptance criterion and still feel bad.

### 20.1 Functional Correctness (automatable / binary pass-fail)

- [ ] Every Down on a valid touch target produces a visible pickup acknowledgment within Section 17's target frame budget.
- [ ] Ghost Preview state (Valid/Occupied/Out-of-Bounds/None) always matches what `validatePlacement` would return for the current candidate origin, with zero observed mismatches across a manual test pass.
- [ ] Every invalid release returns the Piece to its exact original Tray slot, every time.
- [ ] Every valid release results in the Piece committed at exactly the last-shown Ghost Preview position, with no positional discrepancy.
- [ ] No Combat-layer visual/audio/haptic cue fires before a Commit event, across a manual test pass covering all Section 9 SHOULD NOT items.
- [ ] Backgrounding mid-drag always results in a clean Tray-return on resume, never a stuck/ghost Piece.

### 20.2 "Does This Actually Feel Good?" (subjective, requires human playtesting, not just code review)

- [ ] Does the pickup moment feel *instant* — is there any perceptible hitch between finger-down and the Piece visually responding?
- [ ] Does dragging a large Piece (e.g., Square 3×3, `02_SHAPE_LIBRARY.md` SHP_032) across the Board feel as smooth and light as dragging a Single Block — or does it feel heavier/laggier in a way that isn't intentional?
- [ ] Does the finger-occlusion offset (Section 5) feel natural, or does the Piece feel like it's "floating away" from the touch point?
- [ ] When sweeping the Piece quickly across many cells, does the Ghost Preview feel like it's keeping up, or does it feel like it's chasing the finger?
- [ ] Does an invalid-placement attempt feel like a shrug ("okay, not there") rather than a rebuke ("you did something wrong")?
- [ ] Does a valid placement — especially a multi-line Combo placement — feel *rewarding* the instant it lands, before any Combat Feedback even plays?
- [ ] Does the overall interaction feel "premium" — i.e., does it feel like a crafted, considered touch experience, or does it feel like a generic/default drag-and-drop implementation? (This is intentionally the least specifiable checklist item and is meant to be evaluated holistically, not decomposed further — per the Primary Objective's explicit "premium" requirement.)
- [ ] Does playing one-handed (thumb-only reach) on a large phone still feel comfortable, given the Tray's on-screen position?
- [ ] After 10+ consecutive placements, does any part of the interaction start to feel repetitive/fatiguing in a way that suggests an animation is overstaying its welcome (a candidate for trimming even if it "tested fine" in isolation)?

---

## 21. Acceptance Criteria

- Every pointer lifecycle stage (Section 3) maps onto a Gameplay Spec Section 4 Battle state transition with no invented intermediate Battle state.
- The Ghost Preview's displayed legality state is always literally backed by a live `validatePlacement` call against the current candidate origin — never a cached, approximated, or client-predicted value that could diverge from the Board Engine's own answer.
- No Combat-layer visual, audio, or haptic feedback of any kind fires between Drag start and Commit (Section 9), verified against every item in that section's SHOULD NOT list.
- Every non-valid release path (Occupied, Out-of-Bounds, No Board contact, Cancel, Backgrounding) converges on the identical Tray-return visual/audio/haptic treatment (Section 10–11), with no path producing a harsher or different penalty than another.
- Reduced-motion and haptics-disabled configurations both preserve full placement-relevant information with zero loss (Section 16).
- Ghost Preview legality is communicated through at least one non-color signal in every state (Section 8, Section 16).
- No animation defined in this document blocks or delays the next pointer Down from being accepted once the relevant Battle state (`ACTIVE`) is live (Section 1 rule 2, Section 13).
- A drag interrupted by backgrounding always resolves to a clean Tray-return on resume, with no observable stuck or duplicated Piece state (Section 15).
- All Section 20.1 functional checklist items pass in automated/manual QA before this document is considered implemented; all Section 20.2 items are evaluated via human playtesting as a separate, non-blocking-for-function but required-for-ship quality gate.

---

## 22. Cross-Document Contract

- **Consumes from `01_GAMEPLAY_SPECIFICATION.md`:** the `PIECE_SELECTED`/`DRAGGING` Battle states and their legal-transition table (Section 4) as the exact skeleton this document's Pointer Lifecycle (Section 3) and Input State Machine (Section 18) must nest inside without modification; the Pause/Resume and Backgrounding rules (Sections 19–20) that Section 15 of this document implements at the pointer layer; and this document fulfills Gameplay Spec Section 25's explicit delegation ("Input doc: Owns gesture/drag handling detail beneath the `PIECE_SELECTED`/`DRAGGING` states").
- **Consumes from `02_SHAPE_LIBRARY.md`:** the normalized `local_coordinates` origin convention (Section 2) as the exact definition of a Piece's "anchor cell" used in Section 7's snap-to-grid candidate mapping.
- **Consumes from `03_BOARD_ENGINE_AND_RULES.md`:** `validatePlacement` (Section 4) as the sole, authoritative, read-only legality check driving every Ghost Preview state (Section 8) and every Release branch (Section 10) — this document calls it, never reimplements or approximates its logic; `commitPlacement` (Section 5) as the sole Commit-stage action (Section 12); the coordinate system (Section 2) as the shared logical-position language between the two documents (Section 6).
- **Consumes from `04_SPAWN_RNG_AND_DIFFICULTY.md`:** indirectly — the Tray's contents (whatever Shapes were generated) are simply what this document's Piece Pickup (Section 4) operates on; this document has no direct dependency on generation logic itself, only on the Tray as a already-resolved input.
- **Consumes from `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`:** the `DamageEvent` (Section 9 of that document) as the sole trigger for Section 12's Combat Feedback stage, and that document's Section 9 pipeline as the explicit thing Section 9 of this document forbids previewing or anticipating during `DRAGGING`.
- **Exposes to a future Art Direction / Juice-VFX doc:** the exact trigger points (pickup, drag-follow, Ghost Preview states, return-to-tray, Commit, Board Feedback, Combat Feedback — Sections 4–5, 8, 11–12) as the complete set of moments that document must supply concrete visual/audio treatment for; this document defines *when* and *what kind* of feedback occurs, never the exact art/sound assets themselves.
- **Exposes to a future platform-specific implementation doc:** Section 17's relationship-based latency targets as the constraints any concrete millisecond/pixel tuning must satisfy.
- This document must not be edited to embed content owned by the above; it references them structurally only, per the pattern established across `01`–`05`.

---

**End of `06_INPUT_DRAG_AND_DROP_UX.md`.**
