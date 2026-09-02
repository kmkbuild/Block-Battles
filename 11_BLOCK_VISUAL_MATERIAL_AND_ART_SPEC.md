# 11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md
## Block Battles — Block Visual Material and Art Specification

**Governing documents:** `02_SHAPE_LIBRARY.md`, `06_INPUT_DRAG_AND_DROP_UX.md`, `09_VISUAL_DESIGN_SYSTEM.md`, `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`, `00_MASTER_GAME_VISION.md`, `03_BOARD_ENGINE_AND_RULES.md`
**Document status:** Layer B — Block Art Authority

**Inheritance confirmation:** This document does not contradict any Layer A or Layer B document above it. It owns the complete visual specification for every gameplay block state. It implements the visual design language defined in `09_VISUAL_DESIGN_SYSTEM.md` and uses only color tokens registered in `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`. It does not introduce new gameplay mechanics. It does not introduce new color values. The animation properties of block states are noted here as targets; their exact timing curves and sequences are owned by the Animation doc.

---

## 1. Block Art Philosophy

### 1.1 The Block Is the Game

Blocks are the most-seen, most-touched, most manipulated visual element in Block Battles. Over the course of a single play session, a player will look at and drag dozens of blocks. Over a run, hundreds. The block's visual design is therefore the single most important asset investment in the game.

The block must achieve five things simultaneously:

1. **Instant readability** — at a glance, the player must be able to distinguish an occupied cell from an empty one, a placed block from a ghost block, a valid drop zone from an invalid one.
2. **Tactile satisfaction** — the block must feel like a physical object that you are picking up and placing, not a 2D icon that you are dragging a cursor around.
3. **Polished consistency** — every block, regardless of its piece shape or color identity, must look like it belongs to the same visual family. A 1x1 square and a 4-cell L-shape must be recognizably the same kind of object.
4. **Efficient rendering** — this is a mobile game. Every block must be renderable in batches without GPU complexity that reduces framerate below the 60fps target (`00_MASTER_GAME_VISION.md` Section 12.3).
5. **Distinctive identity** — the block style must be a recognizable visual signature of this game specifically, not a generic puzzle-game aesthetic.

### 1.2 The Fundamental Design Decision: Painted Dimensional Flat

The block art style is **Painted Dimensional Flat** — a deliberately chosen middle path between:

- Pure flat vector (too sterile, loses tactility)
- Full 3D render (too heavy, too realistic, mobile performance concern)
- Isometric pixel art (wrong tone, period aesthetic)

**Painted Dimensional Flat** means:
- The block is rendered as a **flat 2D sprite** (no 3D geometry, no normal maps, no real-time lighting).
- Dimensionality is achieved entirely through **painted-in lighting**: a top-left highlight face, a main body face, and a bottom-right shadow face.
- The painting style is clean, deliberate, and slightly idealized — as if a skilled human painted each face with an airbrush at low opacity, not as if it was procedurally generated.
- The result should feel like **a premium physical game piece** — think the satisfying weight and finish of a high-quality board game tile, translated to 2D mobile.

### 1.3 What the Block Should NOT Look Like

| Forbidden aesthetic | Reason |
|---|---|
| Pure flat single-color square with a dark border | No dimensionality, no tactility — reads as a UI element, not a game object. |
| Fully rendered 3D block with PBR materials, normal maps, and real reflections | Inconsistent with the 2.5D design language; mobile performance concern; risks looking like a different game. |
| Beveled tile with an inner border and a drop shadow on every cell individually | Creates visual noise at 8x8 density; the inter-cell gaps become too visually heavy. |
| Candy-gloss sphere / jewel | Match-3 anti-reference (`09_VISUAL_DESIGN_SYSTEM.md` Section 1.7). |
| Pixel art with visible pixel grid | Wrong tone; also inconsistent with the smooth corner radius system. |
| Wood, stone, or textile texture on the block face | Skeuomorphic-kitsch anti-keyword (`09` Section 1.6). |

---

## 2. Base Geometry

### 2.1 Proportions

Each block cell occupies one square grid unit on the 8x8 board. The grid unit size is calculated as:

```
grid_unit = floor(board_pixel_width / 8)
```

where `board_pixel_width` is the rendered board width in pixels at the current device resolution. The board is always square (`09_VISUAL_DESIGN_SYSTEM.md` Section 17.3).

**Block cell visual size within the grid unit:**

The block does not fill its entire grid unit. It occupies **88% of the grid unit on each axis**, centered:

```
block_pixel_size = grid_unit * 0.88
block_inset = (grid_unit - block_pixel_size) / 2   [= grid_unit * 0.06 per side]
```

This inset creates the **cell gap** — the narrow breathing room between adjacent blocks on the board that ensures individual blocks remain visually distinct even at full board density.

**Example at a common board size of 320px:**
```
grid_unit = 320 / 8 = 40px
block_pixel_size = 40 * 0.88 = 35.2px (rendered at 35px, inset 2.5px per side)
```

### 2.2 Silhouette

The block silhouette is a **rounded square** — corner-radius-M (10dp) per `09_VISUAL_DESIGN_SYSTEM.md` Section 8. At the rendered block sizes typical for mobile, this produces a gentle, approachable rounding that reads as "physical object" rather than "UI chip."

**Silhouette rule:** The silhouette shape is identical for every block, regardless of color identity, placement state, or piece shape membership. A block cell in a 1x1 piece and a block cell in a 4x4 piece are the same visual object. The piece shape is constructed from multiple identical block cells placed adjacently.

**Adjacent block gap rendering:** When two blocks in the same piece are adjacent (e.g., two cells of an L-shape sharing an edge), the gap between them is rendered at the inset distance defined in Section 2.1. The gap is never zero — adjacent cells in a piece always have a visible (though narrow) gap between them. This ensures the individual cell count of any piece is always readable.

### 2.3 Corner Treatment

- **Corner radius:** 10dp (M tier, `09` Section 8). Applied at full radius — the corner is a true quarter-circle, not a chamfer or a partial bevel.
- **Corner consistency:** The same 10dp radius applies to the block in every state (Normal, Dragged, Ghost, Placed). The radius never changes between states.
- **Inner corner:** Where the highlight face meets the main face (inside the block's depth rendering), there is no hard inner corner. The face transition is a soft painted gradient, not a geometric edge.

### 2.4 Bevel Treatment

The block uses **painted bevel** — not a geometric bevel extruded from the silhouette.

A geometric bevel (like CSS `box-shadow: inset` or a physical extrusion) would require separate geometry or a complex shader. Instead, the bevel is an artistic convention painted into the sprite's flat surface:

- **Top face (highlight bevel):** The topmost ~18% of the block height is painted 1.5 stops brighter than the BASE color, using a subtle soft-edge gradient from the top edge inward. This simulates a slightly angled top surface catching the light.
- **Bottom face (shadow bevel):** The bottommost ~18% of the block height is painted 1.5 stops darker than the BASE color, using a soft-edge gradient from the bottom edge inward.
- **Left face (secondary highlight):** The leftmost ~10% of the block width is painted 0.75 stops brighter. The top-left light source `09` Section 18.1 also catches the left face, but less strongly than the top face.
- **Right face (secondary shadow):** The rightmost ~10% of the block width is painted 0.75 stops darker.
- **Main face (body):** The central area (~62% width x 64% height) is the BASE color, flat.

**Percentage reference for a 35px block at standard density:**

| Face | Approximate extent |
|---|---|
| Top highlight bevel | Top 6px |
| Bottom shadow bevel | Bottom 6px |
| Left highlight bevel | Left 3-4px |
| Right shadow bevel | Right 3-4px |
| Main face | ~22px x 22px center |

### 2.5 Internal Padding

The block content area (where any future symbol, label, or overlay would be placed) is the main face region defined above. All state overlays (cross-hatch for invalid, phase markers for MVP+) are confined to this interior region and do not extend to the bevel areas.

### 2.6 Cell Spacing

**Gap between adjacent block cells in the same piece (inset-to-inset):** The visual gap = `grid_unit * 0.12`. At a 40px grid unit, this is 4.8px (rendered at 5px). This is the only gap — blocks in the same piece share the same gap as blocks in different pieces. The piece's unity is communicated by color identity, not by a zero-gap shared border.

**Gap between the board outer edge and the first cell column/row:** The board has a uniform inset of `grid_unit * 0.10` from its visual border to the first cell position, ensuring the outermost cells do not touch the board frame.

---

## 3. Material Language

### 3.1 The Chosen Material: Painted Acrylic

The block material is **Painted Acrylic** — specifically, the visual character of a flat acrylic tile with a matte-to-semi-matte finish. This is the single chosen direction. No other material is mixed into the block system.

**What "Painted Acrylic" means in visual terms:**

| Property | Character |
|---|---|
| **Base finish** | Matte-to-semi-matte. Not glossy. Not completely flat — there is a very slight sheen at the highlight bevel, but the main face has no specular. |
| **Opacity** | Fully opaque (Normal state). The block has physical presence. |
| **Edge quality** | Clean, smooth, slightly soft. Not a hard vector edge, not a pixel-jagged edge. Anti-aliased at all sizes. |
| **Surface character** | Smooth with very slight micro-imperfection — not a sterile digital surface, but not visibly textured. Think the surface of a high-quality matte-finish arcade token. |
| **Light interaction** | Diffuse dominant. The light catches the bevel faces and slightly rakes across the left face. No sharp specular highlights, no reflection. |
| **Color richness** | Saturated but not electric. The BASE color feels rich and confident — the color of a well-made physical object, not a screen-glowing icon. |

### 3.2 Why Painted Acrylic

| Alternative considered | Why rejected |
|---|---|
| **Glossy plastic / candy** | Specular highlights require either pre-baked per-color sprites or a shader. Gloss reads as match-3 (anti-reference). Fingerprint smudge aesthetic conflicts with the clean design direction. |
| **Soft rubber / matte foam** | Too soft-looking — loses the "weighted" and "precise" visual keywords from `09` Section 1.5. |
| **Ceramic / porcelain** | The fragility/delicacy associations are wrong for a combat-adjacent puzzle game. |
| **Flat vector (no material)** | Loses tactility entirely. Fails the "physical" and "satisfying" design target. |
| **Wood / stone** | Skeuomorphic-kitsch (`09` Section 1.6). Also inconsistent with smooth corner radius language. |

### 3.3 Material Consistency Requirement

All seven color identities (Ember, Amber, Leaf, Tide, Iris, Rose, Slate) use the identical material. The material does not vary between colors. The only difference between an Ember block and a Tide block is the color values (BASE/LIGHT/SHADOW per `10` Section 3.4) applied to the same painted-acrylic geometry and surface treatment.

---

## 4. Lighting

This section operationalizes `09_VISUAL_DESIGN_SYSTEM.md` Sections 3.6 and 18 for the block specifically.

### 4.1 Global Light Direction

**Single top-left light source at 315 degrees** (measuring clockwise from right/east). Equivalent to a light coming from the upper-left corner of the screen, at roughly a 45-degree elevation angle.

This direction is fixed and does not change with:
- Piece shape or orientation
- Color identity
- Board position
- Theme
- Device orientation (the game is portrait-locked at Absolute MVP)

### 4.2 Highlight Position and Intensity

| Face | Position | Intensity relative to BASE |
|---|---|---|
| Top bevel | Top 18% of block height | +20% lightness (HSL). +5 degrees hue shift toward warm (toward 60 degrees). |
| Left bevel | Left 10% of block width | +10% lightness (HSL). +3 degrees hue shift toward warm. |
| Specular point | None — matte material has no specular point. | N/A |

The transition from highlight bevel to main face is a **soft gradient edge** — the highlight fades into the BASE color over approximately 25% of the bevel width (approximately 1.5px at standard mobile density). This softness is critical to the "painted" quality of the material.

### 4.3 Shadow Direction and Intensity

| Face | Position | Intensity relative to BASE |
|---|---|---|
| Bottom bevel | Bottom 18% of block height | -22% lightness (HSL). -5 degrees hue shift toward cool (away from 60 degrees). |
| Right bevel | Right 10% of block width | -12% lightness (HSL). -3 degrees hue shift toward cool. |

Same soft gradient edge applies — shadow fades to BASE over ~25% of the bevel width.

### 4.4 Drop Shadow (Block-Cast Shadow)

A cast drop shadow is rendered beneath/behind the block sprite, per the elevation system in `09` Section 9.1:

**Placed block (Mid elevation, tier 2):**
- Blur: 6dp
- Opacity: 20%
- Offset: X +2dp, Y +3dp (bottom-right, consistent with light source)
- Color: `CLR-BG-100` darkened 20% (a very dark near-black warm tone — never pure black)

**Dragged block (Active elevation, tier 4):**
- Blur: 20dp
- Opacity: 30%
- Offset: X +4dp, Y +6dp (more pronounced offset at higher elevation)
- Color: Same dark tone as above

Drop shadows are rendered at the piece level (the full piece casts one composite shadow from the dragged piece sprite) rather than at the individual cell level, to avoid shadow stacking across adjacent cells.

### 4.5 Lighting Consistency Requirement

Every block in every state must have its highlight on the top-left bevel and its shadow on the bottom-right bevel, with no exceptions. A block that has its shadow on the top or its highlight on the bottom has a lighting error and fails QA.

---

## 5. Surface Detail

### 5.1 Gradient

The main face of the block uses an **extremely subtle radial gradient** — barely perceptible, centered approximately 30% from the top-left:

- Center point of the radial: approximately (30% x, 30% y) of the main face area.
- Center color: BASE color lightened by 4%.
- Edge color: BASE color exactly.
- Gradient spread: covers the full main face area.

This is a maximum 4% brightness variance. At rendered block sizes (35px), this gradient is not consciously visible — it reads as a slightly "alive" surface compared to a pure flat fill. If at any target device size the gradient becomes visible as a distinct shape, it should be reduced further or removed.

### 5.2 Highlight

The bevel highlight is the primary surface detail — fully specified in Section 4.2. It is painted, not a separate overlay layer.

### 5.3 Texture

**No explicit surface texture at Absolute MVP.** The "micro-imperfection" quality of the painted acrylic is achieved by the softness of the painted bevel gradients, not by a noise texture, grain overlay, or surface normal variation.

**Reserved for MVP+:** A very subtle grain overlay (a noise texture at 3-5% opacity, semi-transparent over the block face) may be added in MVP+ to increase the "handcrafted" quality. This would be a single shared noise texture tiled across all blocks, applied in a post-process step. It must not reduce block-to-block color distinction.

### 5.4 Specularity

**No specular highlights on the main face.** The material is matte. The only brightness variation is:
1. The bevel gradients (Sections 4.2, 4.3)
2. The subtle main-face radial (Section 5.1)

If a specular-looking element appears on the block during production, it is a rendering error.

### 5.5 Subtle Imperfections

At Absolute MVP: none — the block surface is clean and idealized. The "human-made" quality comes from the soft brush-quality of the painted gradients, not from surface irregularities. Visible scratches, chips, dents, or smudges are explicitly forbidden at MVP — these would read as damaged or low-quality.

---

## 6. Normal State

The block at rest in the tray, awaiting player interaction.

### 6.1 Visual Properties

| Property | Value |
|---|---|
| **Opacity** | 100% |
| **Scale** | 1.0x — exactly one grid unit (with the 88% inset per Section 2.1) |
| **Corner radius** | 10dp (M tier) |
| **Surface** | Three-face painted acrylic per Section 5 — BASE/LIGHT/SHADOW |
| **Drop shadow** | Mid/2 elevation per Section 4.4: blur 6dp, opacity 20%, offset (+2dp, +3dp) |
| **Outline** | None — no drawn border |
| **Color** | Assigned identity BASE, LIGHT, SHADOW tokens from `10` Section 3.4 |

### 6.2 Tray Block vs. Placed Block — Normal State Differences

| Property | Tray block | Placed block |
|---|---|---|
| Scale | 1.0x | 1.0x |
| Corner radius | 10dp | 10dp |
| Shadow | Mid/2 elevation | Mid/2 elevation |
| Highlight intensity | Standard | Standard |

**There is no visual difference between a tray block and a placed block at rest.** A piece in the tray looks identical to the same piece placed on the board. The physical context (tray zone vs. board zone) and the grid surface provide the spatial distinction — the block object itself does not visually distinguish "available" from "placed."

**Rationale:** Changing the block's appearance between tray and placed states (e.g., making tray blocks lighter/glowing) would add visual noise and imply a semantic distinction that doesn't exist in the game model — both are the same physical object at different stages.

---

## 7. Dragged State

The dragged block is the highest-priority visual element during active input. It must feel elevated, physical, and under the player's finger.

### 7.1 Elevation Effect

Per `09_VISUAL_DESIGN_SYSTEM.md` Section 9.1 Active/4 tier:

| Property | Value | Change from Normal |
|---|---|---|
| **Scale** | 1.05x — 5% larger than Normal | +5% scale: makes the piece feel "lifted off" the surface |
| **Opacity** | 100% | No change |
| **Drop shadow** | Blur 20dp, opacity 30%, offset (+4dp, +6dp) | Significantly larger, softer shadow simulates elevation |
| **LIGHT face** | +5% additional lightness beyond standard LIGHT value | Stronger top-left catch — the elevated block is closer to the (conceptual) light source |
| **SHADOW face** | No change from Normal SHADOW value | The shadow face remains the same; the increased cast shadow handles the visual weight below |
| **Corner radius** | 10dp (unchanged) | — |

### 7.2 Position Offset from Finger

Per `06_INPUT_DRAG_AND_DROP_UX.md` Section 6 (finger occlusion offset):

The dragged piece is rendered at a **vertical offset upward** from the actual touch point, so the player's finger does not obscure the piece during drag. The exact offset is defined in `06` Section 6 and is not duplicated here. This document only confirms: the dragged piece must always be fully visible above the touch contact point.

### 7.3 Scale Transform Origin

The 1.05x scale applied on pickup is centered on the **piece's logical anchor cell** (`02_SHAPE_LIBRARY.md` Section 2, anchor definition), not on the bounding box center. This ensures that the scale-up feels like the piece lifting toward the player's finger rather than expanding away from a geometric center.

### 7.4 Transition from Normal to Dragged

The scale and shadow change is **immediate on the DRAG_BEGIN event** (per `06` Section 4 drag lifecycle). There is no easing on the pickup scale change — the piece must feel responsive, not sluggish.

The transition back to Normal (on invalid placement and return, per `06` Section 10) uses a brief ease-out — the piece "settles" back into the tray. The exact curve is owned by the Animation doc; this document specifies that the transition exists and that it must complete within the timing window defined in `06` Section 17.

---

## 8. Valid Placement State

The Ghost Preview in the Valid state, as defined in `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` Section 6.1, rendered as a block visual.

### 8.1 Visual Properties

| Property | Value |
|---|---|
| **Fill** | Piece's assigned `CLR-PC-[identity]-BASE` at 45% opacity — flat fill, no three-face model |
| **Outline** | 1.5px solid at the piece's `CLR-PC-[identity]-LIGHT` at 80% opacity |
| **Corner radius** | 10dp (same as all states) |
| **Drop shadow** | None — ghosts have no material, therefore no cast shadow |
| **Highlight faces** | None — the ghost is a semi-transparent silhouette, not a material object |
| **Scale** | 1.0x — the ghost sits at the grid position exactly, same size as placed blocks |

### 8.2 Relationship to the Dragged Piece

The Ghost Preview and the dragged piece are rendered simultaneously:
- The **Ghost Preview** sits at the snapped grid position, at Mid/2 z-order (above the board surface, below the dragged piece).
- The **dragged piece** sits at Active/4 z-order, above the ghost.

The ghost is always slightly below the dragged piece in z-order and lower in visual intensity (semi-transparent vs. fully opaque). Together they communicate: "this is where it will land, and this is what it looks like in your hand."

### 8.3 Non-Color Cue

The solid outline on the Valid ghost is the primary non-color validity signal. In any colorblind simulation, the outline remains visible as a distinct 1.5px border around the ghost cells, distinguishing Valid from both Invalid states (which use dashed or absent outlines).

---

## 9. Invalid Placement State

Two variants: Occupied and Out-of-Bounds. Both are fully specified in `10` Section 6.2 and 6.3. This section translates those color specs into block visual terms.

### 9.1 Invalid — Occupied

| Property | Value |
|---|---|
| **Fill** | `CLR-BRD-EMPTY` at 60% opacity on all ghost cells |
| **Cross-hatch overlay** | `CLR-SEM-DANGER` at 30% opacity, applied only to the specific cells that are already occupied. Pattern: diagonal lines at 45 degrees, line weight 1px, spacing 4px. Covers only the occupied-conflict cells, not the full piece footprint. |
| **Outline** | Dashed 1px at `CLR-SEM-DANGER` at 60% opacity. Dash pattern: 4px dash, 3px gap. |
| **Drop shadow** | None |
| **Highlight faces** | None |
| **Scale** | 1.0x |

**Non-color cues:** Dashed outline (vs. solid in Valid), cross-hatch pattern on specific cells (absent in Valid).

### 9.2 Invalid — Out of Bounds

| Property | Value |
|---|---|
| **In-bounds cells** | `CLR-BRD-EMPTY` at 60% opacity, flat fill |
| **Out-of-bounds cells** | Not rendered — clipped at the board boundary. The piece footprint simply ends at the board edge. |
| **Outline** | Solid 1px at `CLR-SEM-WARNING` at 70% opacity — amber-warning, not danger-red |
| **Drop shadow** | None |
| **Highlight faces** | None |
| **Scale** | 1.0x |

**Non-color cues:** Visible truncation of the piece at the board edge (absent in all other states); amber vs. red outline color (color-level distinction between OOB and Occupied).

---

## 10. Filled Board State

A block that has been successfully placed on the board.

### 10.1 Visual Properties at Placement

At the moment a `COMMIT_PLACEMENT` event fires (`03_BOARD_ENGINE_AND_RULES.md` Section 10):

1. The Ghost Preview disappears instantly.
2. The placed blocks appear at their final positions at the full Normal state visual — three-face painted acrylic, 100% opacity, Mid/2 elevation shadow.
3. A brief settle animation plays (owned by Animation doc). During this animation, the placed blocks momentarily scale from 1.05x down to 1.0x, then hold at 1.0x. The color and material do not change during the settle.

### 10.2 Long-Term Placed State

After the settle animation, placed blocks remain at **Normal state visual** indefinitely:

| Property | Value |
|---|---|
| **Opacity** | 100% |
| **Scale** | 1.0x |
| **Surface** | Full three-face painted acrylic |
| **Shadow** | Mid/2 elevation |
| **Color** | Unchanged from when the piece was placed — the board is a mosaic of the colors of all placed pieces |

### 10.3 Board Mosaic Readability

Because each placed block retains its color identity indefinitely, a full or mostly-full board becomes a mosaic of the seven piece color identities. This mosaic must remain readable — the board must not become an indistinct color blob.

**Readability is guaranteed by:**
- The consistent 12% inter-cell gap (Section 2.6) — every cell boundary is visible regardless of color adjacency.
- The three-face depth model — adjacent blocks of the same color identity are still visually separated because their shared edge shows one block's shadow face and the adjacent block's highlight face.
- The 8x8 maximum density — at this density with the chosen block size, individual cells remain distinguishable at arm-length on all supported device classes.

---

## 11. Line-Clear State

This section defines the visual target state for line-clearing blocks — what the clearing visuals should achieve, independent of animation timing (which is owned by the Animation doc).

### 11.1 Pre-Clear Flash Target

When a row or column is determined to be cleared (after a `COMMIT_PLACEMENT` event), the affected cells must transition through a clearly distinct visual state before disappearing. The visual target for this flash state:

| Property | Value |
|---|---|
| **Fill** | `CLR-CMB-CLEAR-FLASH` (`#F0EAE0`) at 90% opacity, replacing the block's color identity entirely |
| **Three-face model** | Suspended during the flash — the block becomes a near-flat bright warm-white shape (the flash washes out the depth faces) |
| **Scale** | Momentarily expands to 1.08x at the peak of the flash (the block "blooms" before vanishing) — owned by Animation doc |
| **Outline** | None during flash |
| **Shadow** | Opacity reduces to 0% during the flash (the block is luminous; it no longer casts a shadow) |

### 11.2 Post-Clear Empty State

After the flash and the clear animation completes, the cleared cells return to the `CLR-BRD-EMPTY` empty cell visual. The transition is immediate (no fade) — the cell is either filled or empty, not in between.

### 11.3 Simultaneous Multi-Line Clear Appearance

When L >= 2 (multiple lines clear simultaneously, per `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 4), all affected cells flash simultaneously at the same instant. There is no sequential "waterfall" effect. The visual reads as a single, synchronous clear event — all cells bloom and vanish together.

For L=4 (Full Combo), the flash brightness is additionally intensified — the clearing cells briefly reach `CLR-CMB-CLEAR-FLASH` at 100% opacity with the scale-bloom at 1.12x (larger than the standard 1.08x). This visual differentiation of the Full Combo clear is the block-level contribution to the Tier 4 feedback event defined in `09` Section 13.

---

## 12. Special Blocks

### 12.1 MVP vs. Later

Per `03_BOARD_ENGINE_AND_RULES.md` Section 3.2's confirmation that `INDESTRUCTIBLE`, `FROZEN`, and `BLOCKED` cell states are locked to "Later/MVP+," **no special block variants are implemented at Absolute MVP.** All board cells at MVP are either EMPTY or standard FILLED (one of the seven color identities).

This section pre-defines the visual language for each special type so that when they are activated, they implement a consistent art direction without requiring a new material system.

### 12.2 Frozen Block (MVP+)

**Intent:** A standard filled block that has had its cell frozen by an enemy action. It cannot be cleared until it is unfrozen.

| Property | Target direction |
|---|---|
| **Base visual** | The block's original color identity is heavily desaturated — BASE color drops to approximately 20% saturation, color shifts toward `CLR-CELL-FROZEN` (pale blue-white, to be defined in a future revision) |
| **Surface treatment** | An ice-crystal SVG pattern overlay at 50% opacity on the main face — a sharp, angular pattern (distinct from the soft painted-acrylic language of standard blocks, which signals "this is not a normal block") |
| **Highlight/shadow** | Reduced highlight intensity — ice appears more evenly lit than a warm acrylic surface |
| **Outline** | A 1px solid pale blue-white outline |
| **Non-color cue** | The angular crystal pattern on the face distinguishes FROZEN from FILLED without color |
| **Animation target** | A very slow, subtle pulse (brightness oscillates 3-5% at ~1Hz) to signal an active status effect |

### 12.3 Blocked Cell (MVP+)

**Intent:** A cell that has been blocked by an enemy action. No piece can be placed on it.

| Property | Target direction |
|---|---|
| **Base visual** | Dark, desaturated warm-grey fill — significantly lower brightness than any piece color |
| **Surface treatment** | A diagonal line hatch pattern (distinct from the Occupied-Invalid ghost hatch — heavier weight, higher opacity) |
| **Icon** | A small "no entry" glyph (or X) centered on the main face — the non-color cue |
| **Highlight/shadow** | Greatly reduced — the block reads as inert, sunken, heavy |
| **Outline** | A slightly heavier border than standard cells to reinforce the "sealed" feeling |

### 12.4 Indestructible Cell (MVP+)

**Intent:** A cell that cannot be cleared under any condition.

| Property | Target direction |
|---|---|
| **Base visual** | Near-black with a very subtle dark metallic warm tint — the darkest element on the board |
| **Surface treatment** | A stone-grain or hammered-metal texture pattern (very fine, very low contrast — more of a surface quality than a visible texture) |
| **Icon** | A lock or diamond glyph centered on the face |
| **Highlight/shadow** | Minimal highlight — suggests a dense, non-reflective material |

### 12.5 Hazard Cell (MVP+)

**Intent:** A cell that applies a penalty when a piece occupies it (specific mechanic to be defined at activation).

| Property | Target direction |
|---|---|
| **Base visual** | Deep red-brown — distinct from DANGER red by being significantly more desaturated and brownish, distinct from Ember piece by being noticeably darker and less vibrant |
| **Surface treatment** | A warning chevron or hazard stripe pattern at reduced opacity |
| **Non-color cue** | The stripe pattern (same visual language as real-world hazard markings) |
| **Animation target** | A slow pulsing outline when the hazard is "active" |

### 12.6 Future Special Piece Types

Any future piece type that is visually different from a standard block must:
1. Remain on the 10dp corner radius grid.
2. Use only tokens from `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` (or tokens added in a revision).
3. Include a non-color cue (pattern, icon, or shape variant) to distinguish the special type.
4. Be defined in a revision of this document before implementation.

---

## 13. Shape Cohesion

### 13.1 The Multi-Cell Piece as a Unified Object

A piece made of multiple cells (e.g., a 4-cell L-shape from `02_SHAPE_LIBRARY.md`) must read as **one cohesive object**, not as separate independent blocks that happen to be adjacent.

**Cohesion is achieved by:**

1. **Shared color identity** — all cells in a piece share the same BASE/LIGHT/SHADOW values. No cell in a piece ever has a different color identity from its siblings in the same piece.

2. **Consistent lighting direction** — the top-left highlight is on the top-left bevel of every cell in the piece, regardless of where in the piece the cell is located. This means internal cells (surrounded by other cells) still have their highlight on their top-left face — the lighting does not simulate "internal cells being in shadow from neighboring cells." This simplification is intentional: it reduces visual noise and maintains the consistent lighting rule.

3. **Gap size consistency** — the gap between adjacent cells within a piece is the same as the gap between cells of different pieces at adjacent positions on the board (both = 12% of grid unit). The gap never disappears for "adjacent cells in the same piece."

4. **Drop shadow at the piece level** — during dragging, the cast drop shadow is generated for the entire piece bounding shape, not cell by cell. This makes the piece read as one object casting one shadow. For the Shadow to be accurate, it is pre-baked per piece shape in the asset pipeline.

### 13.2 Irregular Shapes — Special Cases

Irregular shapes (IRREGULAR tag in `02_SHAPE_LIBRARY.md` Section 5) may have cells that are diagonally connected or have protruding single cells. These must remain visually readable as distinct units:

- A single-cell protrusion from a larger body must have its own complete bevel highlight/shadow — it is visually self-contained even though it is part of the larger piece.
- The shape identity of an irregular piece must be readable in the Ghost Preview, even when the piece is at the Valid state's reduced opacity. If any Ghost Preview makes the shape unreadable at 45% opacity (because the cells are too small or too close), the minimum opacity for that state should be increased to 50% for that specific piece size — this is the one exception to a uniform opacity rule.

### 13.3 1x1 Pieces

The 1x1 single block (SMALL tag, per `02` Section 5) is the simplest piece. It has exactly one cell, no adjacent cells, and its full bevel is visible on all four faces. The 1x1 is the block's purest form and should function as the reference for correct lighting, proportions, and material treatment. All other pieces are extensions of the 1x1 block.

---

## 14. Block-to-Cell Relationship

### 14.1 Grid Alignment

Every block cell, in every state, snaps to the integer grid position defined by the board's cell coordinates. There is no sub-pixel positioning during settled states (placed blocks, empty cells). During drag, the piece follows the finger continuously (sub-pixel allowed), but the ghost preview always snaps to the nearest valid grid position (integer cell coordinates).

### 14.2 Inset and Margin (Recap)

| Measurement | Value |
|---|---|
| Cell visual fill | 88% of grid unit per axis |
| Inset per side | 6% of grid unit per side |
| Gap between adjacent cells | 12% of grid unit (sum of two 6% insets) |
| Board outer inset | 10% of grid unit |

These values are design-fixed. They must not be altered per-device or per-theme. They define the visual rhythm of the board and altering them would change the perceived density of the game.

### 14.3 Alignment Precision

Every block cell must be aligned to the pixel grid at every integer zoom/scale level that the target device may render at. Anti-aliasing at sub-pixel positions is acceptable during drag (the piece is moving), but must not produce pixel-crawling artifacts in the placed/settled state. The production pipeline should bake placed-block sprites at the exact pixel dimensions that correspond to the grid unit sizes at target device classes (Section 15).

### 14.4 Board Surface Relationship

The board surface (`CLR-BRD-SURFACE`) is the lowest layer. Empty cells (`CLR-BRD-EMPTY`) sit directly on the surface. Placed blocks (Mid/2 elevation) sit above the surface and cast their shadow onto it. The visual stack within the board:

```
(top) Dragged piece (Active/4)
      Ghost Preview (between Mid/2 and High/3)
      Placed block sprites (Mid/2)
      Empty cells (Low/1)
      Board surface (Ground/0)
(bottom)
```

---

## 15. Scale Rules

### 15.1 Grid Unit Sizes by Device Class

The grid unit (the pixel size of one board cell) is derived from the board's rendered width at each device class. The board's rendered width is determined by the responsive layout rules from `09_VISUAL_DESIGN_SYSTEM.md` Section 17.

| Device class | Target board width | Grid unit | Block pixel size (88%) | Min readable at arm-length? |
|---|---|---|---|---|
| Small phone (<= 360dp) | ~288dp (80% of screen width, safe area) | 36dp | ~31.7dp (~32dp) | Yes — 32dp block is above the minimum legibility threshold for this genre |
| Standard phone (360-430dp) | ~320-360dp | 40-45dp | ~35-40dp | Yes |
| Large phone (430-500dp) | ~380-420dp | 47-52dp | ~41-46dp | Yes |
| Tablet (if supported) | ~480-540dp | 60-67dp | ~53-59dp | Yes |

**Minimum block size:** 32dp. Below this, the three-face bevel detail becomes imperceptible and the block reads as a flat colored square. If any device class produces a grid unit smaller than 36dp, the board outer inset (Section 14.2) is reduced from 10% to 5% to recover space before any block size reduction occurs.

### 15.2 Bevel Extent Scaling

The bevel extents defined in Section 2.4 are **percentage-based** and therefore automatically scale with the grid unit. At 32dp block size:
- Top/bottom bevel: 18% of 32dp = 5.76dp (~6dp) — still visible and functional.
- Left/right bevel: 10% of 32dp = 3.2dp (~3dp) — just visible; at this minimum it reads as a subtle edge, which is sufficient.

At tablet sizes (53dp+):
- Top/bottom bevel: 18% of 53dp = 9.54dp (~10dp) — more pronounced; may need slight reduction to 14-15% to avoid the bevel becoming too dominant relative to the main face at larger sizes. Production test at each device class is required.

### 15.3 Corner Radius at Different Scales

The corner radius is **fixed at 10dp (M tier)** per `09` Section 8, not percentage-based. This means:

| Block size | Corner radius | Radius-to-size ratio |
|---|---|---|
| 32dp | 10dp | 31.25% |
| 40dp | 10dp | 25% |
| 53dp | 10dp | 18.9% |

At 32dp, the 10dp radius means nearly one-third of each corner is rounded — the block reads as quite soft/rounded. This is acceptable and intentional — at small sizes, rounder edges improve readability by reducing the harsh visual density of small square corners. At 53dp, the radius produces a more moderately rounded look. Both are within the acceptable range of the "rounded square" target silhouette.

### 15.4 Drop Shadow Scaling

Drop shadow values (blur, offset) from Sections 4.4 and 7.1 are in dp and scale with device density automatically. No per-device shadow adjustment is needed.

---

## 16. Asset Production Specification

### 16.1 Source Artwork Dimensions

All block assets are produced at **@4x source resolution** — four times the baseline grid unit. This provides sufficient resolution for any current or near-future mobile display density.

| Asset | Source dimension (4x of 40dp baseline) |
|---|---|
| Single block cell sprite | 160 x 160 px |
| All other states (ghost, dragged, clearing) are derived from the same source geometry |

Source files are vector-origin: drawn in a vector tool (Figma, Illustrator, or equivalent) at 160x160px artboard, then rasterized for export. The vector source must be maintained for future re-export at different scales.

### 16.2 Export Dimensions

| Export scale | Pixel size | Target |
|---|---|---|
| @1x | 40 x 40 px | Low-density reference (not shipped as primary; used for icon/thumbnail contexts) |
| @2x | 80 x 80 px | Standard-density mobile targets (160dpi range) |
| @3x | 120 x 120 px | High-density mobile targets (320-480dpi range — the primary modern mobile target) |

All three scales are exported for every block variant.

### 16.3 Transparency

Block sprites use **premultiplied alpha** (per `09` Section 20.4). Ghost Preview sprites require alpha channels (they are semi-transparent). Normal/Placed/Dragged block sprites are fully opaque — their alpha channel is still present in the file format but set to 100%.

Export on a neutral mid-grey matte (#808080) before final premultiplied alpha generation to prevent fringing artifacts at the corner radii.

### 16.4 Naming Convention

Follows `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` Section 20.1 (from `09`). Canonical pattern for block assets:

```
piece_block_[identity]_[state]@[scale]x.png
```

**Identity codes:** `emb`, `amb`, `lea`, `tid`, `irs`, `ros`, `slt`

**State codes:**
- `normal` — Normal state (used for both tray and placed)
- `dragged` — Dragged state (elevated highlight, larger shadow — if pre-baked; see Section 16.5)
- `ghost_valid` — Valid Ghost Preview
- `ghost_occ` — Invalid-Occupied Ghost Preview
- `ghost_oob` — Invalid-Out-of-Bounds Ghost Preview
- `clearing` — Line-clear flash state (pre-baked peak frame, if used)

**Full examples:**
```
piece_block_emb_normal@2x.png
piece_block_emb_normal@3x.png
piece_block_tid_ghost_valid@2x.png
piece_block_tid_ghost_valid@3x.png
piece_block_amb_dragged@2x.png
piece_block_slt_ghost_occ@3x.png
piece_block_lea_clearing@2x.png
```

### 16.5 Dragged State — Pre-Baked vs. Runtime Shader

Two implementation approaches are acceptable:

**Option A (Pre-baked):** The dragged state has its additional LIGHT brightness and its larger shadow baked into a separate sprite variant. This eliminates any runtime shader requirement.

**Option B (Runtime tint):** The dragged state applies a subtle brightness-boost shader or tint to the normal block sprite at runtime, and the shadow is handled by the engine's shadow system. This reduces asset count but requires a shader.

**Recommendation:** Option A (pre-baked) at Absolute MVP, to minimize rendering complexity. The additional asset cost (7 color identities x 1 dragged variant x 2 scales = 14 additional sprites) is justified by the zero shader overhead.

### 16.6 Piece-Shape Drop Shadow

The drop shadow during dragging is a **separate pre-baked shadow layer** per piece shape, not per individual cell. For each of the 34 piece orientations in `02_SHAPE_LIBRARY.md`:

```
piece_shadow_[shape_id]_dragged@2x.png
piece_shadow_[shape_id]_dragged@3x.png
```

These are simple semi-transparent dark shapes at the piece's footprint, with a gaussian blur applied (blur 20dp equivalent at each scale). They are rendered beneath the dragged piece sprite and above the board surface.

The placed-block shadow (Mid/2 elevation) is either:
- Applied per-cell via a small drop shadow shader (preferred for simplicity), or
- Pre-baked as a shadow layer per cell (simpler if shaders are not available in the chosen engine).

### 16.7 Sprite Atlas Organization

All block sprites for the same identity are atlased together:

```
atlas_blocks_[identity].png
```

This produces 7 atlas files, one per color identity. Each atlas contains all states and all scales for that identity. Atlas dimensions target 1024x1024px or 2048x2048px depending on content density. Sprite atlasing is critical for mobile GPU draw call reduction.

Ghost preview sprites (which are shared across all color identities in their pattern/outline treatment but vary in fill color) may be atlased separately:

```
atlas_blocks_ghost.png
```

---

## 17. Performance Constraints

### 17.1 Target Framerate

60fps sustained during all input states, including drag (the most GPU-intensive moment). Per `00_MASTER_GAME_VISION.md` Section 12.3.

### 17.2 Draw Call Budget for Block Layer

At full 8x8 board (64 placed blocks + 3 tray pieces + 1 dragged piece + ghost preview):

| Element | Max draw calls |
|---|---|
| 64 placed board blocks | 1 draw call per atlas (7 atlases max, but in practice fewer color identities will be present) — ideally 2-4 calls for the full board if atlas-combined with instancing |
| 3 tray pieces (up to 3 x max piece size cells) | 1-2 additional calls |
| 1 dragged piece | 1 call |
| Ghost preview | 1 call |
| Board drop shadows | Combined into 1 call if pre-baked per cell |
| Dragged piece shadow | 1 call (pre-baked per piece shape) |
| **Total target** | **Under 12 draw calls for the entire block layer** |

This target is achievable with proper sprite atlasing and GPU instancing.

### 17.3 Prohibited Complexity at MVP

| Visual feature | Status | Reason |
|---|---|---|
| Per-block real-time normal mapping | Prohibited at MVP | GPU cost; not justified by visual gain |
| Per-block real-time shadow casting (rasterized) | Prohibited at MVP | Draw call cost; pre-baked shadows achieve the same visual |
| Particle systems on block placement | Prohibited at MVP | Frame budget; placement feedback is animation-based, not particle-based |
| Animated textures on placed blocks | Prohibited at MVP | Bandwidth and GPU cost |
| Per-piece unique geometry (3D mesh) | Prohibited at MVP | Entire block language is 2D sprite-based |
| Screen-space reflections or bloom on block surfaces | Prohibited at MVP | Post-process cost |

### 17.4 Texture Memory Budget

All block sprites combined (7 identities x all states x @2x and @3x):
- **Target: under 8MB of uncompressed texture memory for block atlases.**
- Mobile texture compression (ASTC on iOS, ETC2 on Android) applies after export — final GPU memory footprint should be under 3MB with compression.

---

## 18. Visual QA

### 18.1 Alignment QA

- [ ] Every placed block is pixel-aligned to the grid at every target device density.
- [ ] The gap between any two adjacent cells is visually uniform across the board — no cell appears closer or farther from its neighbors than the defined 12% inset.
- [ ] The board outer inset (10% of grid unit) is uniform on all four sides.
- [ ] No block sprite overflows its cell boundary (the block pixel size must be <= 88% of the grid unit at every device class).
- [ ] The dragged piece's anchor cell aligns with the ghost preview's anchor cell — no visual jump between the dragged piece position and where the ghost indicates it will land.

### 18.2 Color Consistency QA

- [ ] All seven color identities use only the HEX values registered for their BASE, LIGHT, and SHADOW tokens in `10` Section 18.
- [ ] No block asset contains a color value not found in the `10` token registry.
- [ ] Adjacent blocks of the same color identity are still visually separable (the shadow face of one block reads as darker than the highlight face of the adjacent block).
- [ ] The Ghost Preview Valid fill is visually distinct from a fully placed block — the opacity difference (45% vs. 100%) is clearly perceptible at arm-length.
- [ ] CLR-SEM-DANGER (Invalid ghost) is not confused with any piece color identity (specifically: Ember BASE on the board, Rose BASE on the board).

### 18.3 Lighting Consistency QA

- [ ] Every block in the scene has its highlight on the top-left bevel (no blocks with highlight on top-right, bottom-left, or bottom-right).
- [ ] Every block in the scene has its shadow on the bottom-right bevel (no exceptions).
- [ ] The LIGHT face value of the Dragged state block is perceptibly brighter than the LIGHT face of the same block in the Normal state.
- [ ] Drop shadows fall to the bottom-right for both placed blocks and the dragged piece.
- [ ] No block exhibits a hard specular point or glossy reflection on the main face.

### 18.4 Shape Readability QA

- [ ] Every piece orientation from `02_SHAPE_LIBRARY.md`'s 34 registered orientations is tested in both Normal and Ghost Preview Valid states at small phone (32dp block size) and standard phone (40dp block size).
- [ ] At minimum block size (32dp), the piece shape is distinguishable — individual cells are clearly separate units.
- [ ] At maximum board density (64/64 cells filled), individual cell boundaries remain visible via the gap — the board does not read as a solid color field.
- [ ] Irregular shapes (IRREGULAR tag in `02`) remain recognizable as distinct from the board surface in the Ghost Preview Valid state.

### 18.5 State Consistency QA

- [ ] The transition from Ghost Preview Valid (45% opacity) to Placed (100% opacity, full three-face model) requires zero visual re-alignment — the piece lands exactly where the ghost indicated.
- [ ] The Invalid-Occupied ghost cross-hatch pattern appears only on the cells that are actually occupied, not on unoccupied cells of the same piece.
- [ ] The Invalid-Out-of-Bounds truncation clips at the board boundary precisely — no ghost cell fragments appear outside the board boundary.
- [ ] The Line-Clear flash completely covers the block's color identity — no BASE color remains visible during the peak flash frame.
- [ ] The Normal state and Placed state are visually identical (verified: a placed block from tray should look unchanged on board).

### 18.6 Performance QA

- [ ] Total block layer draw calls at full board density (64 blocks + 3 tray + 1 dragged + ghost): <= 12.
- [ ] Total block atlas texture memory (uncompressed): <= 8MB.
- [ ] Frame time during drag on the minimum supported device does not exceed 16.67ms (60fps target).
- [ ] No frame drops during the line-clear flash animation on the minimum supported device.

---

## 19. Future Extensibility

### 19.1 The Theme Compatibility Guarantee

The block material system (Painted Acrylic, three-face depth model, 10dp radius) is designed to remain visually consistent across all four MVP themes and any future themes added per `09` Section 16.

**Why the block material is theme-agnostic by design:**

Blocks are placed on the board — they exist on top of the theme's board surface, against the theme's background. The block's material must read clearly against any board surface tint. Since the board surface tint changes only slightly between themes (per `10` Section 12.2), and the blocks use fully saturated color identities that are high-contrast against any of the defined board tints, no per-theme block variant is needed.

**Verification:** For any new theme added, confirm that all seven piece color identities produce a luminance contrast ratio of at least 3:1 against the new board surface color. If any identity fails this test, the theme's board surface tint must be adjusted, not the block.

### 19.2 New Color Identity Extension

If an eighth (or more) color identity is added in the future (e.g., for a special game mode or seasonal content), the extension process is:

1. Define the new identity's BASE, LIGHT, and SHADOW HEX values following the HSL derivation rules in `10` Section 14 (LIGHT = BASE +20% lightness +5° warm, SHADOW = BASE -22% lightness -5° cool).
2. Register the new tokens in `10` Section 18.
3. Produce block sprites for the new identity in all states and all scales.
4. Verify the new identity is visually distinct from all seven existing identities and from all semantic colors (Section 16 rules of `10`).
5. Update the rotation system in `10` Section 5.1 to include the new identity.

### 19.3 Special Block Extension

When a special block type (FROZEN, BLOCKED, INDESTRUCTIBLE, HAZARD) is activated:

1. Register the color tokens in `10` Section 18 (the reserved token IDs exist — they need HEX values).
2. Add a full visual specification to Section 12 of this document (the target directions in Sections 12.2-12.5 are starting points, not final specs — they require production artwork).
3. Verify the special block visual is: (a) distinct from all standard piece colors, (b) readable at minimum block size, (c) distinguishable without color alone (non-color pattern/icon is always required).

### 19.4 Material System Resilience

The Painted Acrylic direction is resilient to future additions because:

- It does not require engine-specific shaders (it is pre-baked, not computed).
- It does not depend on a specific lighting model (the light is painted in).
- It scales cleanly from small phone to tablet.
- The three-face model (BASE/LIGHT/SHADOW) is a simple system that any new piece type or color identity can slot into without modifying the underlying art pipeline.

---

## 20. Final Art Contract

This is the binding visual specification for every block asset produced for Block Battles. Every block, in every state, must be traceable to one or more of the following commitments:

---

> **One Material. Seven Colors.** Every block, regardless of color identity or piece shape, uses the identical Painted Acrylic material treatment. The system's power comes from consistency — a player who understands one block understands all blocks.

> **Light is Painted. Physics is Simulated.** The block has no 3D geometry, no real-time lighting, no normal map. Its depth and dimensionality are entirely achieved through the three-face depth model (BASE face, top-left LIGHT face, bottom-right SHADOW face). This is efficient, scalable, and must be applied uniformly to every block in every state.

> **The Gap is Sacred.** The 12% inter-cell gap and the 10dp corner radius are non-negotiable at every device size. They are the visual heartbeat of the board — they ensure a player can always count cells, always read piece shapes, and always distinguish a full board from a nearly-full one.

> **The Ghost is Not a Block.** The Ghost Preview is a semi-transparent silhouette — it has no material, no highlight, no cast shadow. It communicates position and validity without creating the illusion of a placed block. Its opacity (45%) and flat fill deliberately distinguish it from the fully placed block it predicts.

> **State Changes are Instant or Intentional.** Color and material state changes (Normal to Placed, Ghost to Placed) are instant — they communicate certainty and commitment. Animated state changes (pickup scale, settle, clear flash) are brief, tightly timed, and owned by the Animation doc. The art provides the visual target; the animation provides the transition.

> **Performance is a Design Constraint, Not a Compromise.** Under 12 draw calls for the full block layer. Under 8MB of atlas texture. 60fps on all supported devices. These are first-class design requirements, not afterthoughts. The Painted Acrylic sprite system exists specifically because it achieves premium visual quality within these constraints.

---

**Every block asset produced for this project inherits this contract. A deviation from any of the six commitments above requires a revision to this document — not a silent exception in production.**

---

**End of `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`.**
