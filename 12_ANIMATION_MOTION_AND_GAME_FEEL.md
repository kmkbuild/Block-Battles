# 12_ANIMATION_MOTION_AND_GAME_FEEL.md
## Block Battles — Animation, Motion, and Game Feel

**Governing documents:** `06_INPUT_DRAG_AND_DROP_UX.md`, `09_VISUAL_DESIGN_SYSTEM.md`, `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `00_MASTER_GAME_VISION.md`
**Document status:** Layer B — Animation and Game Feel Authority

**Inheritance confirmation:** This document inherits the feedback tier system from `09_VISUAL_DESIGN_SYSTEM.md` Section 13 and the timing windows from `06_INPUT_DRAG_AND_DROP_UX.md` Section 17. It defines the exact motion curves, durations, and interpolations that satisfy those requirements. It does not alter input handling or core rules. All animations defined here must adhere to the 60fps performance mandate.

---

## 1. Motion Philosophy

### 1.1 Core Tenets

Motion in Block Battles exists to serve four specific goals, in priority order:

1. **Clarity:** Ensure the player understands what state the game is in, what action they just took, and what the consequences of that action are.
2. **Responsiveness:** Eliminate any perception of lag between human intent and system reaction.
3. **Reward:** Make successful actions feel intrinsically satisfying, scaling with the difficulty of the achievement.
4. **Tactile Satisfaction:** Sell the physical illusion of the Painted Acrylic materials (`11` Section 3) interacting with weight, friction, and momentum.

### 1.2 What Motion Must Never Do

- **Block input unnecessarily:** Gameplay input (`ACTIVE` state) must never be locked while waiting for a purely decorative animation to finish.
- **Obscure the board:** The board is Priority 1 (`08` Section 3). Screen-covering sweeps, blinding flashes, or opaque overlays that hide the board state during combat resolution are prohibited.
- **Induce motion sickness:** Rapid full-screen shaking, repetitive high-contrast flashing, or disorienting camera movements must be avoided or clamped.
- **Make the game feel slow:** A 200ms animation that looks beautiful but makes placing three pieces take 3 seconds of waiting is a failed animation. Snappy always beats smooth.

### 1.3 The "Tactile Snap" Identity

The animation style of Block Battles is **Tactile Snap**. It favors:
- **Instant onset:** Animations start at high velocity rather than easing in slowly.
- **Spring-damped settling:** Objects find their resting state with a brief, tight bounce that suggests weight and tension.
- **No floatiness:** Objects move purposefully. They never drift, float, or ease slowly through space without a physical justification.

---

## 2. Timing System

All animations in the game use one of these standardized timing classes. **Never hardcode arbitrary durations (e.g., 173ms).**

| Class | Duration | Primary Use Case | Perception |
|---|---|---|---|
| **Instant** | 0ms (1 frame) | UI state toggles, valid grid snapping, invalid cross-hatch appearance. | Unconscious, immediate. |
| **Micro** | 50 - 80ms | Button press states, pickup scale-up, micro-haptics. | Snappy, tactile response. |
| **Short** | 120 - 150ms | Piece return to tray, valid placement settle, minor UI panel slide. | Fast but perceptible motion. |
| **Medium** | 250 - 350ms | Standard line clear flash, enemy hit reaction, menu transitions. | The standard "event" duration. |
| **Long** | 500 - 800ms | Enemy defeat animation, Boss phase transition, Relic card reveal. | Dramatic, requires attention. |
| **Epic** | 1200 - 2000ms | Game Over sequence, Victory sequence, Run Start sequence. | Full context shift, gameplay is paused. |

---

## 3. Easing System

The game relies on three primary easing curves. Linear easing is prohibited for anything other than color fades or progress bar fills.

### 3.1 Snap-Out (Cubic Out / Quart Out)
The default workhorse ease. Starts at maximum velocity immediately upon trigger, then decelerates smoothly into the target value.
- **Use for:** UI entrances, piece return to tray, damage number pops.
- **Why:** Feels responsive because the initial movement covers the most distance instantly.

### 3.2 Spring (Damped Harmonic Oscillator)
A physics-based spring function simulating mass, stiffness, and damping.
- **Use for:** Piece pickup scale, valid placement settle, button press release.
- **Parameters:** High stiffness, critical or slight under-damping (1 to 1.2 bounces max).
- **Why:** Sells the "physical object" illusion of the blocks.

### 3.3 Anticipation-Overshoot (Back In-Out)
Pulls back slightly before firing forward, overshoots the target, then settles back.
- **Use for:** Major event triggers, Relic selection confirmation, Tier 4 combo numbers.
- **Why:** Draws the eye and adds massive emphasis to a singular event.

---

## 4. Global Motion Rules

- **Scale changes:** Never scale an object from its geometric center if it rests on a surface. Scale from the bottom edge (or logical anchor) so it appears to lift off or press into the surface, not expand outward.
- **Opacity fades:** Fade opacity linearly. Easing opacity usually results in a "dead zone" where the object is barely visible but still rendering.
- **Z-order sorting:** If an object is moving from a low elevation to a high elevation, its Z-order (and drop shadow properties) must jump to the target tier immediately at the start of the animation, not interpolate.
- **Simultaneous movement:** When multiple independent objects move to the same state (e.g., 3 tray pieces sliding in), stagger their start times by a Micro duration (50ms) to create a mechanical, cascading rhythm rather than a flat unified block of movement.

---

## 5. Piece Pickup Animation

Triggered on the `DRAG_BEGIN` event (`06` Section 4).

| Element | Action | Duration | Easing |
|---|---|---|---|
| **Piece Scale** | Scale from 1.0x to 1.05x. Pivot is the touch anchor point. | Micro (80ms) | Spring (tight, slight overshoot) |
| **Y-Offset** | Piece translates upward from finger by the occlusion offset (`06` Sec 6). | Micro (80ms) | Snap-Out |
| **Shadow** | Shadow switches to Active/4 tier, blur and offset increase. | Micro (80ms) | Snap-Out |
| **Highlight** | LIGHT face brightness increases by +5%. | Instant | None |

**Game Feel Note:** The piece must feel like it "snaps" to the finger magnetically the instant the screen is touched, pulling away from the tray surface against gravity.

---

## 6. Drag Motion

Triggered continuously during the `DRAGGING` state.

- **Pointer Tracking:** The piece follows the pointer position exactly.
- **Easing:** None. Adding easing or smoothing to the drag position disconnects the piece from the finger and makes the game feel unresponsive or floaty. It must be 1:1 raw input mapping.
- **Framerate dependency:** Position updates must occur at the display refresh rate.

---

## 7. Snap Motion (Ghost Preview)

Triggered when the dragged piece enters a new valid or invalid grid zone.

- **Ghost Position:** The Ghost Preview object moves to the newly calculated anchor cell.
- **Duration:** Instant (1 frame).
- **Easing:** None.
- **Why:** The ghost is a prediction, not a physical object. If the ghost smoothly slides between grid cells, the player cannot trust exactly which cell it is currently predicting. It must snap instantly.

---

## 8. Invalid Placement Motion

Triggered when a player releases the piece in an Invalid-Occupied, Invalid-Out-of-Bounds, or No Contact state.

| Element | Action | Duration | Easing |
|---|---|---|---|
| **Rejection Shake** | (Occupied only) The piece shakes left/right slightly (±3dp, 2 cycles) while holding position above the board. | 100ms | Sine/Linear |
| **Color Hold** | The Ghost Preview's danger/warning color treatment flashes on the piece itself for one frame. | Instant | None |
| **Return to Tray** | Piece translates from the drop point back to its original tray slot. | Short (150ms) | Snap-Out |
| **Return Scale** | Piece scales back from 1.05x to 1.0x. | Short (150ms) | Spring |
| **Shadow Return** | Shadow returns to Mid/2 tier. | Short (150ms) | Linear |

**Game Feel Note:** The rejection shake must be extremely brief. The return to tray is fast enough that the player isn't waiting for it, but visible enough that they track where the piece went.

---

## 9. Placement Animation

Triggered on a `COMMIT_PLACEMENT` event (L=0). This is the Tier 2 interaction (`09` Section 13).

| Element | Action | Duration | Easing |
|---|---|---|---|
| **Position** | Piece snaps exactly to the Ghost Preview grid position. | Instant | None |
| **Ghost Despawn** | Ghost preview vanishes. | Instant | None |
| **Impact Settle (Scale)**| Piece scales from 1.05x to 0.98x, then rebounds to 1.0x. | 120ms | Spring (under-damped) |
| **Shadow/Light** | Shadow drops to Mid/2 tier; LIGHT face returns to normal brightness. | Instant | None |

**Game Feel Note:** The piece does not slide into place. It slams down. The slight 0.98x scale compression sells the impact force of the acrylic block hitting the board surface.

---

## 10. Line Clear Animation

Triggered when a placement results in L=1 (single clear). This is the Tier 3 interaction.

### 10.1 The Sequence

1. **Placement Settle:** The piece performs its standard Placement Animation (Section 9).
2. **Clear Anticipation (The Flash):** Immediately upon placement, the cleared cells wash out to `CLR-CMB-CLEAR-FLASH` and bloom in scale to 1.08x.
3. **Collapse (The Vanish):** The cells rapidly shrink to 0 scale and opacity 0.
4. **Damage Number:** The damage number pops out of the cleared row/col simultaneously with the collapse.

### 10.2 Timings

| Element | Action | Duration | Easing |
|---|---|---|---|
| **Flash/Bloom** | Color replacement and scale to 1.08x. | Micro (60ms) | Snap-Out |
| **Collapse** | Scale from 1.08x to 0, opacity to 0. | Short (150ms) | Back-In (anticipates slightly before shrinking) |
| **Damage Pop** | Number scales from 0.5x to 1.0x, moves upward 20dp. | Medium (300ms) | Spring (heavy bounce) |
| **Damage Fade** | Number holds, then fades to 0 opacity. | 400ms hold, 200ms fade | Linear |

### 10.3 Simultaneous Behavior
If a line clears, all 8 cells in that line execute the Flash and Collapse **at the exact same time**. There is no cascading or wave effect across a single line. The block feels like a rigid structure that detonates simultaneously.

---

## 11. Multi-Line Clear Animation (Combos)

Triggered when a placement results in L=2, L=3, or L=4. This is the Tier 4 interaction.

### 11.1 Intensity Scaling

The base sequence is identical to the L=1 clear, but parameters scale dramatically based on the L value.

| Parameter | L=2 (Double) | L=3 (Triple) | L=4 (Full Combo) |
|---|---|---|---|
| **Flash Bloom Scale** | 1.10x | 1.15x | 1.25x |
| **Collapse Duration** | 150ms | 180ms | 250ms (slight slow-mo emphasis) |
| **Damage Number Scale**| 1.2x base | 1.5x base | 2.0x base |
| **Damage Number Color**| `CLR-CMB-DAMAGE-2` | `CLR-CMB-DAMAGE-3` | `CLR-CMB-DAMAGE-4` (Red-hot) |
| **Damage Pop Easing** | Spring (heavy bounce) | Anticipation-Overshoot | Massive Anticipation-Overshoot, holds at peak |

### 11.2 Screen-Level Effects

- **L=3:** A subtle screen-edge vignette glow (`CLR-CMB-COMBO-GLOW`) pulses for 300ms.
- **L=4:** A sharp, full-screen edge flash (`CLR-CMB-FULLCOMBO-GLOW`) fires at the exact moment of the Collapse phase. The camera performs a micro-shake (Section 19).

**Game Feel Note:** L=4 must feel like breaking a pane of glass. It is violent, sharp, and intensely rewarding.

---

## 12. Damage and Combat Animation

### 12.1 Enemy Hit Reaction
Triggered when the damage pipeline outputs final damage > 0.

| Element | Action | Duration | Easing |
|---|---|---|---|
| **Portrait Shift** | Enemy sprite translates horizontally (random direction) by 5-10dp, then returns to center. | 150ms | Spring (stiff) |
| **Color Wash** | `CLR-CMB-ENEMY-HIT` flash overlay. | 100ms | Snap-Out (instant on, fade out) |
| **HP Bar Depletion**| Fill track drains to new value. | 300ms | Cubic-Out (fast start, smooth end) |
| **HP Value Ticker** | Numeric value counts down to target. | Sync with bar | Linear interval |

### 12.2 Player Damage?
**Reminder:** The player has no HP. The enemy does not attack the player directly; they affect the board. There is no "player taking damage" screen flash or flinch animation.

---

## 13. Enemy Action Animations

### 13.1 Board Modification (e.g., Freezing a cell)
When the enemy executes a board action (on their turn, per `01` state machine):

- **Telegraph clear:** The telegraph indicator pulses and disappears (100ms).
- **Targeting:** A fast, thin projectile or beam connects the enemy portrait to the target cell (150ms).
- **Cell impact:** The target cell applies its state change (e.g., transitioning to FROZEN). The cell performs a heavy, downward-settling scale (1.0x to 0.95x and back) accompanied by the specific special-block color transition (`11` Section 12).
- **Duration:** The entire enemy action phase must not exceed 600ms total before control returns to the player.

### 13.2 Enemy Defeat
Triggered when enemy HP reaches 0. (Tier 5 interaction).

1. **Impact Hold (Freeze Frame):** Game logic pauses. Enemy sprite locks in the hit-reaction shifted position. Color wash goes pure white. (150ms).
2. **Shatter/Dissolve:** Enemy sprite breaks into geometric shards or dissolves rapidly outward. (400ms, Snap-Out).
3. **Screen Flash:** Full-screen warm white flash at 70% opacity. (Fades over 300ms).
4. **Victory Transition:** The Victory UI (Run Results or Relic Selection) begins its entrance sequence.

---

## 14. Objective Feedback

Triggered when objective progress increments.

- **Increment:** Progress bar fills slightly (200ms, Cubic-Out). Number increments instantly.
- **Completion (Primary):** Bar flashes `CLR-SEM-SUCCESS`, checkmark icon scales up from 0 with a Spring ease (300ms). Bar text updates.
- **Completion (Bonus):** Amber badge pops onto the bonus chip (Spring ease, 250ms).
- **Failure (Move Limit):** Indicator flashes `CLR-SEM-DANGER`, shakes intensely (200ms), and transitions to Board Lock sweep.

---

## 15. UI and System Motion

### 15.1 Relic Selection Entrance
When the Relic Selection modal appears:

1. **Scrim fade:** Background dims (200ms).
2. **Card deal:** The 3 Relic cards slide in from the bottom of the screen, staggered by 80ms each.
3. **Easing:** Anticipation-Overshoot (cards fly up slightly past center, then settle down).
4. **Duration:** 400ms total for all cards to settle.

### 15.2 Relic Confirmation
When a player selects a card:

1. **Elevation:** Card scales to 1.05x, shadow increases to Active/4. (Micro, 80ms).
2. **Selection:** A bright border traces the card edge.
3. **Confirm tap:** Card scales to 1.1x, then collapses into a point of light that flies into the HUD's Relic Strip. (Medium, 350ms).
4. **Unselected cards:** Fade out and drop downward. (Medium, 300ms).

### 15.3 Modal Toggles (Pause Menu)
- **Entrance:** Scale up from 0.9x to 1.0x, opacity 0 to 100. (Short, 150ms, Cubic-Out).
- **Exit:** Scale down to 0.9x, opacity to 0. (Short, 120ms, Cubic-In).
- **Why:** Fast, unobtrusive. Pause menus should not make you wait.

---

## 16. Game Over / Board Lock Motion

Triggered when the Board Engine detects no valid placements exist for the current tray (Board Lock). (Tier 5 interaction).

1. **Discovery Pause:** Game input is locked. Wait 400ms. (Let the player realize they are stuck).
2. **The Sweep:** A slow, resigned wave of `CLR-CMB-BOARDLOCK` (Danger red) sweeps across the unplaceable tray pieces, then across the remaining empty board cells.
3. **Easing:** Linear wave, moving top-left to bottom-right.
4. **Duration:** Long (800ms).
5. **Defeat Modal:** The Defeat modal fades in slowly (500ms).

**Game Feel Note:** Do not explode the board. Do not shake the screen. The player has lost a puzzle game; the appropriate emotional response is a sigh, not a jump scare. The animation should feel final, inevitable, and calm.

---

## 17. Screen Shake

Screen shake is used **only** for the most intense combat events. It is applied to the board and UI layer, not the background.

| Event | Intensity | Duration | Description |
|---|---|---|---|
| L=1, L=2, L=3 Clear | None | 0ms | Do not shake the screen for standard play. |
| **L=4 (Full Combo)** | Medium | 150ms | Tight, rapid, high-frequency vibration (±4dp on X/Y). |
| **Boss Phase Change** | Low | 300ms | Slow, rumbling horizontal shake (±2dp on X). |

**Maximum Sensible Intensity:** Never exceed ±6dp translation. Never apply rotation (camera roll) during a shake — it causes immediate motion sickness on mobile displays.

---

## 18. Haptics Hooks

Haptics are a primary channel for the "Tactile Snap" identity. They must be perfectly synchronized with the visual animation frame.

| Hook Name | Hardware Target | Trigger Event |
|---|---|---|
| `Haptic_MicroTick` | Light, crisp tick | Piece pickup, piece enters new grid cell during drag, UI button tap. |
| `Haptic_Impact` | Medium, sharp thud | Valid placement settle (L=0). |
| `Haptic_Clear` | Heavy, sharp burst | Line clear flash onset (L=1, L=2, L=3). |
| `Haptic_FullCombo` | Maximum intensity vibration | L=4 collapse frame. |
| `Haptic_Error` | Double-pulse (buzz-buzz) | Invalid placement drop, Move Limit warning. |

*Implementation Note:* If the platform API allows, `Haptic_MicroTick` on grid-cell transition during drag should be extremely subtle — just enough to let the player feel the grid through their thumb.

---

## 19. Reduced Motion Mode

Accessibility requirement (`00` Section 19.1). When Reduced Motion is enabled at the OS or game level:

1. **Screen Shake:** Completely disabled (0 intensity).
2. **Line Clear Flash:** The scale bloom (1.08x to 1.25x) is disabled. The color wash remains (required for feedback), but the cells simply disappear without expanding first.
3. **Enemy Hit Reaction:** The positional shift is disabled. Only the color wash flashes.
4. **Relic/Menu Entrances:** Replaced with simple 100ms opacity fades. No sliding or spring scaling.
5. **Ghost Preview:** The ghost still snaps (instant movement is accessible), but invalid rejection shakes are replaced with a simple color flash.

---

## 20. Interruptibility and State Overlap

The game is turn-based, but animations must overlap to prevent the game from feeling sluggish (`06` Section 12, Concurrent Feedback Pipeline).

### 20.1 Allowed Overlaps
- **Board Clear + Combat:** The Line Clear animation (board cells flashing and collapsing) and the Combat animation (damage number moving, enemy HP bar draining) **must play at the same time**. The board does not wait for the enemy to take damage.
- **Combat + Next Turn Input:** The player is granted `ACTIVE` input state (can pick up the next piece) the moment the Combat resolution logic completes, **even if the enemy HP bar is still animating its depletion**. The animation is visual only; the state is already resolved.

### 20.2 Interruptions
- **Piece Return:** If a piece is animating its return to the tray (Invalid placement), the player **can** immediately tap/drag another piece in the tray. The return animation continues independently.
- **UI Animations:** Menu sliding/fading can be interrupted by a closing tap, which reverses the animation from its current interpolation point.

### 20.3 Non-Interruptible (Blocking)
- **Enemy Action:** While the enemy is executing a board modification, player input is blocked. This animation must therefore be fast (Section 13.1).
- **Boss Phase Transition:** Gameplay is paused to ensure the player sees the state change.
- **Relic Reveal:** The initial deal of the 3 cards cannot be bypassed, ensuring the player registers their options.

---

## 21. Performance Rules

Animation complexity directly impacts the 60fps target.

1. **No runtime mask animation:** Do not animate clipping masks (e.g., for wiping effects) on the board layer. They break GPU batching.
2. **Transform and Opacity only:** 95% of animations in this document rely exclusively on `Scale`, `Translate`, `Rotate`, and `Opacity`. These are heavily optimized by mobile game engines.
3. **Avoid animating colors (Tints):** Changing a sprite's tint color per-frame can break sprite batching in some engines. When a color change is required (e.g., Line Clear flash), prefer overlaying a pre-colored white sprite and animating its opacity, rather than animating the tint of the block itself.
4. **Physics simulation:** Do not use a 2D physics engine (Box2D) to calculate the Spring easings. Use mathematical tweening functions (e.g., an underdamped sine wave tween). Real physics introduces non-determinism and massive overhead for simple UI bounce effects.

---

## 22. Animation Timing Matrix (Master Reference)

| Event | Target | Property | Duration | Easing | Audio | VFX | Haptic |
|---|---|---|---|---|---|---|---|
| **Pickup** | Dragged Piece | Scale (1.05x), Z-offset | 80ms | Spring | `sfx_lift` | None | `MicroTick` |
| **Grid Snap** | Ghost Preview | Position X/Y | 0ms | Instant | None | None | `MicroTick` (subtle) |
| **Invalid Drop**| Dragged Piece | X/Y Shake | 100ms | Sine | `sfx_error` | None | `Error` |
| **Return** | Dragged Piece | Position X/Y (to tray) | 150ms | Snap-Out | `sfx_slide` | None | None |
| **Place (L=0)** | Placed Piece | Scale (0.98x -> 1.0) | 120ms | Spring | `sfx_place` | Dust puff | `Impact` |
| **Clear Flash** | Cleared Cells | Opacity wash, Scale up | 60ms | Snap-Out | `sfx_charge` | Glow | `Clear` |
| **Clear Poof** | Cleared Cells | Scale to 0 | 150ms | Back-In | `sfx_clear` | Particles | None |
| **Dmg Number** | UI Text Layer | Scale, Pos Y | 300ms | Spring | None | None | None |
| **Enemy Hit** | Enemy Sprite | Pos X shift | 150ms | Spring | `sfx_hit` | None | None |
| **Enemy Die** | Enemy Sprite | Scale/Dissolve | 400ms | Snap-Out | `sfx_defeat` | Shards | `Impact` |
| **Board Lock** | Board Cells | Color wave | 800ms | Linear | `sfx_loss` | None | `Error` (long) |
| **Modal Open** | UI Panel | Scale, Opacity | 150ms | Cubic-Out | `sfx_ui_in` | None | `MicroTick` |

---

## 23. Animation QA Checklist

Run this checklist at every milestone build:

- [ ] **Pickup snap:** Does the piece scale up and offset the exact frame the finger touches the screen?
- [ ] **Ghost snap:** Does the ghost preview jump instantly between grid cells without smooth interpolation?
- [ ] **Placement settle:** Does placing a piece feel heavy? (Verify the 0.98x scale compression occurs).
- [ ] **Clear synchronicity:** Do all cells in a cleared line flash and disappear on the exact same frame?
- [ ] **Pipeline overlap:** Can the player pick up the next piece while the enemy HP bar is still draining from the previous turn?
- [ ] **Combo scaling:** Is a 4-line combo visually, audibly, and haptically more intense than a 1-line clear?
- [ ] **No frame drops:** Does a 4-line combo clear trigger any frame drops below 60fps on the minimum spec test device?
- [ ] **Accessibility:** Does turning on Reduced Motion disable screen shake and scale-blooms?

---

## 24. Final Motion Contract

> **Block Battles is a game about weight, precision, and impact.**
>
> The animation system respects the player's time. We never lock input to show off an animation. When the player moves, the game snaps to attention. When the player succeeds, the game rewards them with crisp, tactile impacts.
>
> We do not use floaty, linear, or sluggish motion. Everything is driven by tension — springs, snap-outs, and deliberate anticipations.
>
> **If an animation makes the game feel slower, the animation is wrong.**

---

**End of `12_ANIMATION_MOTION_AND_GAME_FEEL.md`.**
