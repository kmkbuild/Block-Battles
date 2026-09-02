# 13_VFX_PARTICLE_AND_IMPACT_SPEC.md
## Block Battles — VFX, Particle, and Impact Specification

**Governing documents:** `09_VISUAL_DESIGN_SYSTEM.md`, `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`, `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `12_ANIMATION_MOTION_AND_GAME_FEEL.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`
**Document status:** Layer B — VFX and Feedback Authority

**Inheritance confirmation:** This document inherits the color tokens from Doc 10 and the timing/motion parameters from Doc 12. It does not alter game mechanics. It defines the particle systems, post-processing flashes, and procedural effects that accompany the animations defined in Doc 12. 

---

## 1. VFX Philosophy

VFX in Block Battles serves as **visual punctuation**. 

If the board is a sentence and animation is the rhythm of speaking, VFX is the exclamation point. It must be sharp, legible, and immediately vanish to let the next sentence begin.

**Core Tenets:**
1. **Geometric over Organic:** We use crisp, geometric particles (squares, fragments, lines) that match the "Painted Acrylic" block art. No realistic smoke, no volumetric fire, no soft glowing lens flares.
2. **Subtractive Polish:** A great effect in this game is one that communicates impact and then gets out of the way in under 400ms. Lingering particle trails obscure the board and are forbidden.
3. **Causality over Spectacle:** VFX must always explain *why* something happened. If a relic modifies damage, a particle must bridge the relic to the damage number. 
4. **Performance is Aesthetic:** A dropped frame ruins game feel more than a missing spark. Effects are budgeted strictly for 60fps mobile execution.

---

## 2. VFX Intensity Scale

Effects are classified by their necessary screen presence and particle budget.

| Intensity | Particle Budget | Screen Space | Usage |
|---|---|---|---|
| **Subtle** | < 10 | Cell-local | UI hovers, Invalid placement rejections, minor HUD updates. |
| **Normal** | 10 - 25 | 1-2 Grid Cells | Block placement impact, standard Enemy hit, L=1 Line Clear. |
| **Elevated** | 30 - 50 | Affected Rows/Cols | L=2 and L=3 Line Clears, major Relic triggers, Objective completion. |
| **Major** | 60 - 100 | Board-wide + HUD | L=4 Full Combo, Enemy Defeat, Run-ending Board Lock. |
| **Boss** | 100 - 150 | Full Screen | Boss phase transition, Boss defeat. |

---

## 3. Placement Effects

Triggered precisely on the `Haptic_Impact` / 0.98x scale bounce frame of a valid placement (Doc 12, Section 9).

- **Impact Ring:** A single, crisp 2px un-filled square stroke expands from the piece's bounding box center, scaling from 1.0x to 1.2x while fading to 0 opacity. (Duration: 150ms).
- **Micro-Dust:** 4 to 8 tiny square particles (2x2 dp) emit outward from the bottom edge of the piece. They travel 5-10dp downward/outward and fade instantly.
- **Color:** Uses `CLR-BRD-EMPTY-BORDER` (subtle board color). Placements do not spark in bright colors; they kick up "board dust."

---

## 4. Invalid Placement Effects

Triggered on the rejection shake (Doc 12, Section 8).

- **Rejection Spark:** 2 to 3 sharp, fast-moving linear particles (sparks) eject horizontally from the point of conflict (the occupied cell boundary).
- **Color:** `CLR-SEM-DANGER` (Red). 
- **Duration:** 100ms.
- **Why:** The spark visually reinforces the "collision" that caused the rejection without muddying the board with lingering smoke.

---

## 5. Line Clear Effects (L=1)

Triggered on the "Clear Poof" phase (Doc 12, Section 10).

- **The Flash:** Handled by animation (Doc 12), cells turn `CLR-CMB-CLEAR-FLASH`.
- **The Burst:** As the cell scales to 0, it emits 4-6 geometric fragments (triangles and small squares). 
- **Fragment Behavior:** Fragments inherit the original BASE color of the block that was cleared. They burst outward radially from the cell center, rotating rapidly, and shrinking to 0 scale over 200ms.
- **The Glow:** A very subtle, additive blend underlay glows briefly behind the cleared line. Color: `CLR-CMB-CLEAR-FLASH` at 30% opacity, fading linearly over 150ms.
- **Debris Limits:** Debris must never travel further than 1.5 grid units from its origin cell to prevent board clutter.

---

## 6. Multi-Line Effects (Combos L=2 to L=4)

In Block Battles, a "Combo" is defined by the L-value (simultaneous lines cleared). VFX scales non-linearly to reward high L-values.

| Element | L=2 (Double) | L=3 (Triple) | L=4 (Full Combo) |
|---|---|---|---|
| **Fragment Density** | 6-8 per cell | 8-12 per cell | 15+ per cell |
| **Fragment Speed** | Normal | Fast (+30%) | Explosive (+60%), higher friction (stops faster) |
| **Glow Underlay** | `CLR-CMB-DAMAGE-2` | `CLR-CMB-DAMAGE-3` | `CLR-CMB-DAMAGE-4` |
| **Screen Edge** | None | `CLR-CMB-COMBO-GLOW` vignette flash (200ms) | `CLR-CMB-FULLCOMBO-GLOW` hard edge flash (250ms) |
| **Connecting Arcs** | None | Subtle energy arcs jump between the intersecting clear lines | Heavy, bright arcs tracing the cross-sections of cleared lines |

---

## 7. Damage Effects

Occurs exactly as the Damage Number pops (Doc 12, Section 12).

- **Normal Hit:** A directional burst of 8-10 fragments away from the center of the enemy portrait. Color: `CLR-CMB-ENEMY-HIT`.
- **Critical Hit / High Damage:** (Triggered if damage exceeds 25% of Enemy Max HP in one blow, or modified by a crit relic). Adds a prominent "X" or starburst slash across the enemy portrait. Fragments are larger and travel further.
- **Armor / Defense Hit:** If enemy defense mitigates >30% of incoming damage (Doc 05), the hit VFX changes. Fragments are dull gray (`CLR-TXT-SECONDARY`), speed is halved, and they bounce downward as if deflected. A sharp, metallic ring effect expands from the hit point.
- **Special / Relic Damage:** If a relic adds flat/multiplier damage, a distinct colored trail (matching the Relic's primary color) flies from the Relic HUD chip to the enemy immediately prior to the hit spark.

---

## 8. Enemy Effects

- **Enemy Hit:** See Section 7 above.
- **Enemy Stagger:** (Triggered by specific stun/delay mechanics, if added). Portrait emits a dizzying ring of 3-4 orbiting dots that fade over 500ms.
- **Enemy Defeat (Major):** Portrait freezes, turns pure white (`CLR-CMB-ENEMY-DEFEAT`), and shatters into 30-50 large geometric shards. Shards fly outward in 3D space (scaling up slightly to simulate flying toward the camera) and fade over 400ms. A screen-space shockwave distortion briefly ripples outward from the enemy center.
- **Status Effect Application:** When an enemy freezes or blocks a board cell, a sharp, colored line connects the enemy portrait to the target cell (100ms), followed by a localized burst of particles at the cell (e.g., ice crystals for FROZEN, heavy dust for BLOCKED).

---

## 9. Objective Completion Effects

- **Subtle Increment:** When progress is made but not completed (e.g., 2/5 lines cleared), a tiny spark `CLR-SEM-SUCCESS` traces the edge of the objective progress bar. (Intensity: Subtle).
- **Major Completion:** When the objective reaches its target (5/5). A bright radial flare bursts from the center of the Objective UI chip. 15-20 small confetti-like squares (`CLR-SEM-SUCCESS` and `CLR-TXT-ACCENT`) shoot outward in screen-space and fall via gravity, fading before they cross the board zone. (Intensity: Elevated).

---

## 10. Relic Effects (Causality)

Relics are passive but must feel active when they contribute to a turn.

- **Trigger Flash:** When a relic applies its modifier during the Damage Pipeline, its card/chip in the HUD flashes white briefly (100ms).
- **Causality Trail:** A thin, fast-moving comet trail fires from the HUD chip to the point of impact (the enemy or the board). This makes invisible math visible. 
- **Style:** Clean lines, no soft smoke. Uses the accent color associated with that specific relic's artwork.

---

## 11. Boss Effects

Bosses require grander feedback but must not obscure the board.

- **Entrance:** Boss drops into the UI zone from above. A heavy screen shake (Doc 12) triggers, and dust particles eject horizontally from the UI dividing line into the board area, dissipating quickly.
- **Phase Transition:** `CLR-BOSS-ACCENT-1` sweeps across the background layer. The boss portrait pulses aggressively. Ambient particles (small floating squares) drift upward in the background for 2-3 seconds to signal heightened tension.
- **Boss Defeat:** Scales the Enemy Defeat effect up by 2x. Multiple secondary explosions trigger across the portrait area before the final shatter. The full screen flashes white.

---

## 12. Particle Design Language

| Property | Rule |
|---|---|
| **Shapes** | Perfect squares, 45-degree rotated squares (diamonds), sharp triangles, straight lines. **No circles, no soft blobs, no smoke puffs.** |
| **Scale** | Ranges from 2x2 dp (dust) to 12x12 dp (major fragments). |
| **Lifetime** | Maximum 600ms. Standard is 200-300ms. Nothing lingers. |
| **Speed/Friction** | Particles emit at high velocity and have high drag/friction. They burst outward fast and stop quickly before fading, rather than drifting endlessly. |
| **Rotation** | Fragments rotate at constant speeds (90 to 360 deg/sec) to simulate tumbling. |
| **Emission** | Burst emission almost exclusively. Very few continuous emitters exist in the game to avoid clutter. |
| **Consistency** | All particles use flat, unshaded colors. No gradients on individual particles. |

---

## 13. Motion + VFX Synchronization

VFX is a slave to Animation (Doc 12). 

- **Hook Rule:** A particle system must never define its own timing for *when* an event happens. It is spawned by an Animation Event trigger.
- **Duration Alignment:** If an animation (like a Line Clear collapse) takes 150ms, the primary VFX burst must occur exactly on frame 1 of that 150ms window.
- **Cleanup:** All particle systems are set to auto-destroy or return to the pool immediately upon completion of their lifetime.

---

## 14. Color Integration

VFX materials must use unlit shaders drawing exclusively from the Doc 10 token registry.

- **Additive Blending:** Used only for flashes, laser trails, and glows (`CLR-CMB-CLEAR-FLASH`, `CLR-CMB-COMBO-GLOW`). 
- **Alpha Blending (Opaque):** Used for physical debris, blocks, and dust. A red fragment from an Ember block uses `CLR-PC-EMB-BASE` with standard alpha blending so it retains its material identity and doesn't blow out to white when overlapping other fragments.

---

## 15. Screen-Space vs World-Space

| Space | Usage | Why |
|---|---|---|
| **World-Space (Board Layer)** | Block fragments, placement dust, cell highlights, invalid placement sparks. | These effects are physical interactions with the board and must obey board masking and scaling. |
| **Screen-Space (UI Layer)** | Combo screen-edge flashes, Damage numbers, Objective bursts, Relic causality trails. | These are meta-feedback mechanisms communicating with the player directly, unconstrained by the board grid. |

---

## 16. Performance Budget

### 16.1 Hard Limits
- **Max Active Particles:** 300 total across the entire screen at any given frame.
- **Overdraw Limit:** Additive glows must not overlap more than 3 layers deep. High overdraw on mobile GPUs kills fill-rate instantly.

### 16.2 Pooling
- All VFX prefabs must be instantiated at load time into an Object Pool. 
- Instantiating particle systems at runtime during a 4-line combo will cause frame stutter (garbage collection/allocation spikes).

### 16.3 Mobile Fallback
If the device profiles as low-end (e.g., < 3GB RAM, older SoC):
- Disable the additive glow underlays entirely.
- Cap particle burst counts at 30% of normal values (e.g., a 15-particle burst becomes a 5-particle burst).
- Disable Boss ambient drifting particles.

---

## 17. Accessibility (Reduced Motion)

When the OS or in-game "Reduced Motion / Reduced Flashes" setting is enabled:

1. **Disable all Additive Flashes:** Screen-edge flashes (`CLR-CMB-FULLCOMBO-GLOW`), full-screen defeat flashes, and cell clear flashes are disabled. Replaced with simple opacity fades.
2. **Remove Screen Shake:** All camera trauma is clamped to 0.
3. **Minimize Debris:** Burst counts are reduced to 1-2 particles purely for confirmation, removing the chaotic visual noise of shattering blocks.
4. **Disable Distortion:** Screen-space shockwaves (Enemy Defeat) are disabled to prevent nausea.

---

## 18. VFX Asset Registry

Canonical list of particle system prefabs required for production:

| ID | Description | Intensity | Space |
|---|---|---|---|
| `vfx_place_dust` | Micro dust at block base | Subtle | World |
| `vfx_invalid_spark` | Sharp red rejection spark | Subtle | World |
| `vfx_clear_burst_base` | The scalable line clear debris burst | Normal - Major | World |
| `vfx_clear_glow` | The additive underlay for line clears | Normal | World |
| `vfx_enemy_hit_standard`| Standard damage spark | Normal | Screen |
| `vfx_enemy_hit_armor` | Deflected armor spark (gray) | Normal | Screen |
| `vfx_enemy_defeat` | Shard explosion + distortion | Major | Screen |
| `vfx_objective_complete`| Confetti burst for UI chips | Elevated | Screen |
| `vfx_relic_trail` | Fast causality line from UI to board | Normal | Screen |
| `vfx_screen_edge_flash` | Full Combo intensity overlay | Major | Screen |

---

## 19. VFX QA Checklist

- [ ] **Frame Rate:** Triggering an L=4 Full Combo does not drop the framerate below 60fps on the target minimum spec device.
- [ ] **Visibility:** Debris from line clears fades before a new piece can be dragged into that space.
- [ ] **Overdraw:** Glowing effects do not blow out to pure white when multiple lines clear adjacently.
- [ ] **Color Accuracy:** Fragments generated by clearing an Ember block match the Ember BASE hex code perfectly.
- [ ] **Pooling:** Playing a 10-minute session shows 0 particle system instantiations after the loading screen (confirming pool usage).
- [ ] **Accessibility:** Toggling "Reduced Motion" successfully suppresses all full-screen flashes and camera shakes.

---

## 20. Final VFX Contract

> **Visual Effects in Block Battles serve the game's mechanics, not the artist's reel.**
>
> We use sharp, geometric debris instead of soft, muddy smoke. We use high-velocity, high-friction movement to make impacts feel heavy and tactile. We scale our rewards, ensuring that a 4-line clear feels exponentially more powerful than a 1-line clear.
>
> Above all, we protect the board. No effect is permitted to linger, obscure, or confuse the player's reading of the puzzle state. The game is clean; the VFX must be cleaner.

---

**End of `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`.**
