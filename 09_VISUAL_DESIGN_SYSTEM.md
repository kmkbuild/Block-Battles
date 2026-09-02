# 09_VISUAL_DESIGN_SYSTEM.md
## Block Battles — Visual Design System

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `06_INPUT_DRAG_AND_DROP_UX.md`, `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`, `08_UI_UX_MASTER_SPECIFICATION.md`
**Document status:** Layer B — Visual Design Authority

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23. It is the **Visual Design System** referenced in `08_UI_UX_MASTER_SPECIFICATION.md` Section 25's Cross-Document Contract. It introduces no new gameplay mechanics. It owns the complete visual language, production standards, and art direction. Every future visual document (Color System, Block Art, Animation, VFX, Audio) must implement this document without contradicting it.

---

## 1. Visual Identity

### 1.1 Personality

Block Battles is a **tactile puzzle game with attitude**. It feels handcrafted — not procedurally generated, not algorithmically assembled, not templated. Every screen should feel like a human made deliberate decisions about it. The personality sits at the intersection of:

- **Confident simplicity** — everything on screen is there because it needs to be.
- **Playful precision** — the game is serious about its rules but not serious about itself.
- **Satisfying weight** — pieces feel like they have physical presence; placing them should feel like pressing a thumb into clay.
- **Clean energy** — not minimalist sterility, but a purposeful openness that lets the puzzle breathe.

### 1.2 Mood

| Context | Mood |
|---|---|
| Home screen / menus | Calm, inviting, slightly ambient — a quiet space before the action. |
| Active battle (low pressure) | Focused, clean, rhythmic — the puzzle state. |
| Active battle (high pressure — Move Limit low, Board filling) | Heightened, urgent but never chaotic — pressure comes from board state, not from the UI screaming. |
| Line clear / combo | Momentary joy, earned — sharp, fast, satisfying. |
| Enemy defeat / victory | Distinct celebration, short-lived — a beat, not a parade. |
| Defeat / Board Lock | Neutral, informative — calm resignation, never punishing. |

### 1.3 Emotional Tone

The player should feel:

1. **In control** — the UI never crowds them.
2. **Competent** — feedback rewards correct decisions visibly.
3. **Curious** — relic cards and enemy reveals have enough visual character to spark interest.
4. **Respected** — the game presents information clearly; it does not hide things or bury them in decoration.

### 1.4 Design Character

Three words that must apply simultaneously to every visual element produced:

> **Clear. Crafted. Alive.**

- **Clear** — the purpose of every element is legible at a glance.
- **Crafted** — it shows a considered hand; not over-polished, not under-designed.
- **Alive** — it has warmth, personality, and just enough imperfection to feel human.

### 1.5 Visual Keywords

These are the permitted vocabulary of visual decisions. If a design choice cannot be traced back to at least one of these, it requires justification:

`tactile` - `weighted` - `readable` - `warm` - `precise` - `bold` - `clean` - `expressive` - `confident` - `grounded` - `layered` - `purposeful`

### 1.6 Anti-Keywords (Explicitly Prohibited)

| Anti-Keyword | What to avoid |
|---|---|
| `neon-generic` | Dark background, electric cyan/pink, glowing everything, no hierarchy — the default look of thousands of mobile puzzle games. |
| `AI-wallpaper` | Hyper-detailed fantasy art, airbrushed textures, photorealistic lighting used decoratively where it competes with gameplay. |
| `cheap-casual` | Flat primary colors with no refinement, rounded-everything-with-no-reasoning. |
| `clinical` | Pure white backgrounds, system-font UI, no personality — looks like a wireframe that shipped. |
| `overloaded` | Every element fighting for attention. Shadows everywhere. Gradients on gradients. Particles every second. |
| `skeuomorphic-kitsch` | Wood textures, felt-table backgrounds, stitched leather panels — genre tropes used without purpose. |
| `tryhard-epic` | Gold-foil-everything, self-serious tone that the game's tone cannot support. |

### 1.7 What This Game Should NOT Look Like

- A generic match-3 mobile game (candy/jewel aesthetic, pastel explosions, coin rain).
- A generic neon-arcade block game (Tetris clone with glowing LED aesthetic).
- A low-effort roguelite (programmer art, 9-patch panels everywhere, placeholder icons shipped as final).
- A premium mobile game trying to look like a PC title (too dense, too much chrome, information overload).
- A children's educational game (infantile shapes, exaggerated animations, primary-only colors).

---

## 2. Reference Philosophy

### 2.1 How to Use Visual References

References are **directional, not prescriptive**. A reference tells us what emotional register or craft quality to aim for. It does not grant license to copy visual systems.

**Three-tier classification for any reference:**

| Tier | Definition | Usage |
|---|---|---|
| **Genre Convention** | Visual norms players of this genre expect (boards are grids, pieces have distinct colors, HP bars deplete). | Adopt freely — deviating requires a strong reason. |
| **Inspiration** | A quality, system, or feeling from another work that we want to evoke. | Extract the principle; rebuild from scratch. Never trace or clone. |
| **Anti-reference** | A work that defines what we are deliberately NOT doing. | Keep in the brief as a rejection signal, not a guide. |

### 2.2 Valid Inspiration Sources (Principle-Level Only)

| Source (type, not named product) | Principle to extract |
|---|---|
| Premium mobile puzzle games (board-first, 2019-2024 era) | Board readability as absolute hierarchy. Pieces with satisfying visual weight. Animations timed to sound. |
| Modern independent card games | Relic/card presentation with character per card, legible at small sizes, no rarity-color noise. |
| Physical board game components | Tactile depth suggestion — pieces feel like objects, not flat sprites. |
| Brutalist UI design (applied softly) | Purposeful whitespace, bold typographic hierarchy, nothing decorative unless earned. |
| Japanese puzzle games (contemporary) | Restraint. Exact spacing. Every pixel chosen. Visual polish applied uniformly. |

### 2.3 Original Design Decisions

The following are Block Battles-specific visual inventions:

1. **Board-as-stage** — the board is the game's physical stage, given active spatial presence.
2. **Piece depth language** — blocks have a consistent physical depth metaphor (depth-implied through light and shadow, not 3D rendered).
3. **Combat through clarity** — enemy damage feedback integrates into the board zone's visual rhythm rather than overlaying it with spectacle.
4. **Relic card personality** — each relic card is a primary decorative investment; the player's collected objects should feel individually crafted.

---

## 3. Art Direction

### 3.1 Shape Language

The game's visual DNA is built from **rounded rectangles and soft squares** — never hard 90-degree corners at the macro level, never perfectly circular.

- **Primary shape:** Rounded rectangle. Applies to: board cells, UI panels, buttons, relic cards, HP bars, piece blocks.
- **Secondary shape:** Soft hexagonal accent (used sparingly, only for iconography and badges — never for primary containers).
- **Forbidden macro shapes:** Sharp triangles as primary containers. Perfect circles as primary containers. Irregular polygons as UI surfaces.

Shape language should feel like **physical objects that could exist in the real world**.

### 3.2 Scale Language

| Element | Relative Scale |
|---|---|
| Board | Dominant — occupies the largest visual footprint on screen at all times. |
| Piece blocks (placed) | Medium — clearly readable as individual units. |
| Enemy HP bar | Prominent — second-largest interactive element in the Top Zone. |
| Objective chip | Medium — always legible, never decorative-only. |
| Relic icons | Small — recognizable at icon size without zooming. |
| Score text | Small — present but never demanding. |
| Decorative elements | Micro — exists in negative space only, never competing with scale. |

**DESIGN RULE:** Nothing decorative may approach the scale of Priority 1 or Priority 2 elements as defined in `08_UI_UX_MASTER_SPECIFICATION.md` Section 3.

### 3.3 Detail Density

**Medium-low density by default.**

- **Board cells:** Low internal detail. Cell identity is communicated through color and depth, not surface texture.
- **Enemy presentation:** Medium detail — enough personality to read as a character at the Top Zone's constrained size.
- **Relic cards:** Medium-high detail — the highest-detail surface in the game.
- **HUD chrome:** Low detail — structural purpose only.

### 3.4 Softness vs Sharpness

| Context | Softness Level | Reasoning |
|---|---|---|
| Board cell edges | Medium-soft (radius per Section 8) | Hard edges on a full 8x8 grid create visual harshness. |
| Piece block edges (in tray) | Slightly softer than placed blocks | Unplaced pieces feel lighter; placed pieces feel settled. |
| Button edges | Medium-soft | Tactile impression. |
| Ghost preview | Very soft | Semi-transparency + softened edge communicates "not yet real." |
| Damage number pop | Sharp, bold | Contrast against soft surroundings; demands brief attention. |
| Enemy portrait | Medium — character edges clean, background soft | Character reads against any board state behind it. |

### 3.5 Depth

Block Battles uses a **two-and-a-half-dimensional depth model**: elements are flat or near-flat, but depth is implied through:

1. **Layering** (z-order: background, board, placed pieces, active piece, UI overlays).
2. **Drop shadows** (per Section 9) — soft, offset, low-opacity.
3. **Surface highlight** — a single, consistent top-edge or top-left highlight on every raised element.
4. **Value shift** — placed pieces are slightly darker/more saturated than the board surface.

This is **NOT** a full 3D rendering approach. No normal maps, no PBR materials, no real-time shadows.

### 3.6 Lighting Philosophy

Single light source. Consistent across all assets.

- **Direction:** Top-left (roughly 10-11 o'clock position).
- **Quality:** Soft-diffuse. Not a sharp point light.
- **What it affects:** Block highlights (top and left faces brighter), block shadows (bottom and right faces subtly darker), UI card surfaces (subtle top-to-bottom gradient).
- **What it does NOT affect:** Backgrounds (ambient only). Enemy portraits (may use own directional lighting internally, but must be consistent).
- **Forbidden lighting:** Lens flare, bloom on static elements, animated lighting on non-feedback elements, multiple conflicting light directions within one screen.

### 3.7 Dimensionality

Everything lives at **one of four dimensional layers**:

| Layer | What lives here | Visual treatment |
|---|---|---|
| **Ground** | Background, board surface | Flat, low contrast, ambient. |
| **Objects** | Placed pieces, board cells | Depth-implied with top-light highlight and subtle shadow. |
| **Active** | Dragged piece, Ghost Preview, Enemy | Slightly elevated — cast shadow on the board below. |
| **Interface** | HUD chrome, panels, buttons, text | Highest — fully above the board in z-order; may use stronger shadows. |

---

## 4. Visual Hierarchy

Defined in alignment with `08_UI_UX_MASTER_SPECIFICATION.md` Section 3's Priority 1-5 system, with visual properties assigned:

### 4.1 Hierarchy Levels

| Level | Content | Visual Properties |
|---|---|---|
| **1 - Board** | The 8x8 grid and all placed cells | Largest element. Highest surface area. Saturation higher than background. The visual anchor of every battle screen. |
| **2 - Pieces** | Current tray pieces + dragged piece | High contrast vs. board background. Depth-implied. Color-distinct per piece identity. |
| **3 - Enemy / Objective** | Enemy HP bar, Primary Objective progress, Enemy Telegraph | Second-highest visual weight zone. Bold type for numbers. High-contrast HP bar fill. |
| **4 - Critical Feedback** | Line clear flash, Damage number, Combo indicator | Transient, high-intensity — only for the frames they are meant to be read. Must fade or disappear. |
| **5 - Secondary UI** | Relic indicators, Bonus Objective, Score, Run indicator | Lower contrast, smaller type, collapsed by default where possible. |
| **6 - Decoration** | Background elements, ambient patterns, environmental flavor | Barely perceptible at rest. Decoration fails if a first-time player notices it before the board. |

### 4.2 Why This Order

The board is the game. A player who cannot instantly read the board cannot play. Everything else exists in service of that read. This directly implements `00_MASTER_GAME_VISION.md` Section 23 Rule 1 and `08_UI_UX_MASTER_SPECIFICATION.md` Section 3.

---

## 5. UI Shape Language

### 5.1 Buttons

| Type | Visual treatment |
|---|---|
| **Primary action button** | Full-width or large, filled background in a high-contrast accent, bold label centered, radius per Section 8's Large tier. |
| **Secondary action button** | Outlined or ghost variant, same radius, medium contrast. |
| **Destructive action button** | Outlined in a warning-register color; label in that color — never a filled destructive button at normal state. |
| **Disabled button** | Reduced opacity only — not a different visual design. |
| **Icon button** (Pause, Back, etc.) | Compact, fixed size, contained in a lightly surfaced rounded container. Never a bare tap target. |

**BINDING RULE:** No button ever uses a shape that deviates from the corner radius system in Section 8.

### 5.2 Cards

| Property | Standard |
|---|---|
| Background | Slightly elevated surface (Section 9's elevation system). |
| Border | 1px border at a lighter value than the card's background; not a solid black outline. |
| Corner radius | Large tier (Section 8). |
| Shadow | Card-tier shadow (Section 9). |
| Content padding | M spacing (Section 7) on all sides. |
| Icon zone | Top-left or top-center, consistent across all cards of same type. |
| Label zone | Below icon, bold. |
| Description zone | Below label, body weight, capped line count. |

### 5.3 Panels

| Property | Standard |
|---|---|
| Background | Semi-transparent or full-opacity tinted surface — never pure white or pure black. |
| Border | Subtle top or bottom 1px line only (not a full outline). |
| Corner radius | XL tier at panel outer edges; flat where panel meets screen edges. |
| Shadow | Panel-tier shadow, downward only. |
| Content padding | L spacing (Section 7). |

### 5.4 Indicators

| Property | Standard |
|---|---|
| Shape | Compact pill (capsule radius). |
| Background | Mid-elevation surface. |
| Type | Numerical, bold, larger than context text. |
| Color | Inherits semantic color of the data it shows (neutral = default, warning-register = pressure state). |
| Size | Fixed — does not grow with value change. |

### 5.5 Chips

| Property | Standard |
|---|---|
| Shape | Tight capsule. |
| Background | Lightly tinted — 15-20% opacity version of a semantic or accent color. |
| Type | Caption weight, small, all-caps or title-case depending on usage. |
| Border | Optional hairline at the same tint family. |

### 5.6 Badges

| Property | Standard |
|---|---|
| Shape | Circle — the one context where circle is the primary shape (badges are always small and adjacent to another element). |
| Size | Fixed: S icon tier (Section 10). |
| Color | Semantic (completion = positive register; warning = warning register). |
| Position | Always top-right of the parent element. Never overlapping critical text. |

### 5.7 Bars

| Property | Standard |
|---|---|
| Shape | Full-radius capsule (pill). |
| Track background | Low-contrast, desaturated tinted surface value — not white. |
| Fill | High-contrast, semantic color fill. |
| Fill transition | Smooth animated depletion (owned by Animation doc). |
| Segmentation | Boss phase bars use visible tick marks. Standard bars have no tick marks. |
| Numeric label | Displayed adjacent or inside (if bar is wide enough). Bold weight. |

### 5.8 Icons

See Section 10 (Iconography) for full standards. Every icon lives inside a container — never bare.

---

## 6. Typography Philosophy

### 6.1 Font Character

| Role | Character Requirements |
|---|---|
| **Display / Heading** | Geometric or slightly humanist sans-serif. Moderate-to-high x-height. Bold weight primary. Numerals must be tabular. |
| **Body / Label** | Same family as Display if supported, or a harmonizing companion. Optimized for small-size readability. Numerals tabular. |

**ANTI-REQUIREMENTS:** No serif fonts. No illegible display fonts. No mixed-family usage without justification. No system-default fonts shipped as final.

### 6.2 Hierarchy

| Level | Role | Weight | Usage |
|---|---|---|---|
| H1 | Screen title | ExtraBold / Black | Home screen title, Victory/Defeat headers. |
| H2 | Section header | Bold | Relic card title, Objective name. |
| H3 | Subsection | SemiBold | Enemy name, onboarding label. |
| Body | Standard text | Regular / Medium | Relic description, Objective description. |
| Caption | Small label | Regular | Chip text, badge labels, secondary info. |
| Numeric-Large | Big numbers | Bold / ExtraBold | Enemy HP value, Score. |
| Numeric-Display | Combat feedback | Black / ExtraBold | Damage numbers overlaid mid-board. |

### 6.3 Readability

- **Minimum size:** Body text must never render below 12sp on any supported device class.
- **Line spacing:** 1.3-1.5x line height for body text. Display text may use tighter 1.1-1.2x for single-line headers.
- **Line width:** No text block exceeds 60 characters per line.
- **Contrast:** All text meets WCAG 2.1 AA minimum contrast (4.5:1 for body, 3:1 for large/bold display text).

### 6.4 Numbers — Special Treatment

- **Tabular numerals mandatory** for any number that changes live (HP, score, objective progress, turn count).
- **Combat Damage numbers scale with damage magnitude:**
  - 1 line cleared: base size
  - 2 lines cleared: 1.2x base
  - 3 lines cleared: 1.5x base
  - 4 lines cleared (Full Combo): 2.0x base
- **Score:** Small, Numeric-Large weight, top-zone corner.
- **Objective progress:** Bold, inline. Format: current / target or remaining, depending on objective type.

### 6.5 Weight Usage

| Weight | Usage |
|---|---|
| Regular | Descriptions, body text, captions. |
| Medium | Secondary labels, collapsible chips. |
| SemiBold | Third-level headers, button labels on secondary buttons. |
| Bold | Primary button labels, numeric displays, objective names. |
| ExtraBold / Black | Display numerics (damage), screen-level headers, combo indicators. |

**BINDING RULE:** Weight must only vary with hierarchy — never for decoration.

---

## 7. Spacing System

All spacing values derive from a base unit of **4dp**.

| Token | Value | Usage |
|---|---|---|
| **XS** | 4dp | Internal icon padding, tight chip padding, hairline separator gaps. |
| **S** | 8dp | Compact element inner padding, gap between closely related inline elements. |
| **M** | 12dp | Standard inner padding for interactive elements (buttons, compact cards). Gap between related HUD elements. |
| **L** | 16dp | Standard inner padding for panels and cards. Gap between distinct HUD groups. |
| **XL** | 24dp | Section-level separation (Top Zone to Board Zone, Board Zone to Tray Zone). |
| **XXL** | 32dp | Screen-edge outer margin on standard phones. |
| **XXXL** | 48dp | Modal overlay inner padding on large phones/tablets. |

**BINDING RULES:** Every spacing value must map to one of the tokens above. No fractional dp values (6dp, 10dp, 14dp) without an explicit design exception. Safe-area insets are additive to the nearest margin token.

---

## 8. Corner Radius System

| Token | Value | Usage |
|---|---|---|
| **None** | 0dp | Board cell interior grid lines only. No UI element uses 0dp radius. |
| **XS** | 4dp | Internal decorative details only. |
| **S** | 6dp | Board cells (the individual square cells of the 8x8 grid). |
| **M** | 10dp | Piece blocks (tray and placed on board). Badges. Small indicators. |
| **L** | 14dp | Buttons (all types). Chips. Progress bars. |
| **XL** | 20dp | Cards (Relic cards, onboarding tooltip cards). |
| **XXL** | 28dp | Panels (Top Zone, Tray Zone). Modals (Pause overlay, Victory screen). |
| **Full / Capsule** | 50% of height | HP bar, Objective bar, pill-shaped Indicators, Move Limit counter. |

**BINDING RULES:** Piece blocks use M (10dp) — slightly softer than board cells (S/6dp) to distinguish piece objects from the grid surface. All elements of the same type use the same radius.

---

## 9. Depth System

### 9.1 Elevation Tiers

| Tier | Surface | Shadow | Highlight | Examples |
|---|---|---|---|---|
| **Ground (0)** | Darkest surface, no shadow cast. | None. | None. | Background plane. |
| **Low (1)** | Slightly lighter than Ground. | Very subtle ambient shadow (blur: 4dp, opacity: 15%, offset: 0 1dp). | None. | Board surface. |
| **Mid (2)** | Board-cell value. | Soft drop shadow (blur: 6dp, opacity: 20%, offset: 0 2dp). | Subtle 1dp top-left edge highlight at +15% brightness. | Placed piece blocks, filled board cells. |
| **High (3)** | UI panel surfaces. | Standard card shadow (blur: 12dp, opacity: 25%, offset: 0 4dp). | 1dp top-edge highlight at +20% brightness. | HUD panels, cards at rest. |
| **Active (4)** | Dragged piece. | Elevated shadow (blur: 20dp, opacity: 30%, offset: 0 6dp). | Stronger top-left highlight at +25% brightness. | Dragged piece (06 Section 4). |
| **Modal (5)** | Full-screen overlay panels. | Scrim below (semi-transparent dark layer). | Top edge only. | Pause overlay, Relic Selection, Victory screen. |

### 9.2 Rules

- Elevation must follow logical stacking: a UI panel (High/3) cannot appear beneath a placed piece (Mid/2).
- The dragged piece (Active/4) must visually read as elevated above all placed blocks.
- Shadows are always **downward** (offset Y positive). Exception: Tray Zone panel casts shadow upward onto the board edge (deliberate).
- Shadow colors are **dark-tinted** versions of the background palette, never pure black.

### 9.3 Ghost Preview Depth

Ghost Preview lives between Mid (2) and High (3). Achieved by:

- Reduced opacity fill (40-60%).
- No surface highlight (ghosts have no material).
- No shadow.
- Non-color pattern cue (hatching, dashed outline) for Invalid states — never color alone (accessibility requirement).

---

## 10. Iconography

### 10.1 Icon Style

All icons use a **filled icon style with consistent optical weight**.

### 10.2 Filled Philosophy

| Context | Style |
|---|---|
| HUD icons (Relic strip, Pause, Back) | Filled. |
| Onboarding tooltips | Filled for action icons; outline acceptable for decorative arrows/pointers. |
| Objective type indicators | Filled, consistent glyph per objective category. |
| Status indicators (Telegraph warning, Move Limit alert) | Filled. |

### 10.3 Size Hierarchy

| Token | Size | Context |
|---|---|---|
| **XS** | 12x12dp | Badge icons, tiny inline status dots. |
| **S** | 16x16dp | Chip icons, compact HUD elements (Relic strip at rest). |
| **M** | 20x20dp | Standard HUD icons (Pause button, Back button). |
| **L** | 24x24dp | Card icons at rest (Relic card icon). |
| **XL** | 32x32dp | Objective type icon in the Top Zone. |
| **XXL** | 48x48dp | Relic card icon on the Relic Selection screen. |

### 10.4 Alignment

All icons within any single container are optically centered, not mathematically centered. Optical adjustment is always less than or equal to 2dp from mathematical center.

### 10.5 Icon Grid

- **Canvas:** Fixed square per tier.
- **Live area:** 80% of canvas.
- **Safe area inset:** 10% per side — no icon element extends to canvas edge.
- **Stroke weight:** Consistent optical weight across all icons, proportional to icon size.

---

## 11. Panels and Cards

### 11.1 Purpose Distinction

| Element | Purpose |
|---|---|
| **Panel** | Structural container organizing HUD zones (Top Zone, Tray Zone). Zone-level in size. |
| **Card** | Content-carrying surface presenting a single coherent unit of information (one Relic, one Objective, one onboarding tip). Individually interactive. |

### 11.2 Visual Hierarchy Within Panels

Top Zone panel internal hierarchy (top to bottom):
1. Enemy identity (P5 — smallest, flavor-only).
2. Enemy HP bar (P2 — largest, bold).
3. Telegraph indicator (P2 — prominently adjacent to HP).
4. Primary Objective (P2 — large chip or inline bar).
5. Bonus Objective (P3 — collapsed chip, below Primary).
6. Secondary indicators (P4 — small, right-aligned or tucked).

### 11.3 Background Treatment

| Element | Background |
|---|---|
| Panel (Top Zone) | Semi-transparent surface — 85-92% opacity over the background. Frost/blur if performance allows; fallback to opaque tinted surface. |
| Panel (Tray Zone) | Same treatment as Top Zone. |
| Card (Relic Selection) | Fully opaque, elevated surface. |
| Card (Tooltip) | Fully opaque, elevated surface. Slightly warmer tint. |

### 11.4 Border Treatment

- **Panels:** Single 1px top or bottom border at a lighter-than-surface value.
- **Cards:** Full 1px border at a lighter-than-surface value.
- **No element uses a solid dark or black outline at full opacity.** Global rule.

### 11.5 Shadow

- Panels: High (3) tier.
- Cards at rest: High (3) tier.
- Cards selected (Relic pre-confirm): Active (4) tier — the selected card elevates slightly to confirm selection.

### 11.6 Information Density

- **Panels:** Medium density. All information shown is necessary.
- **Cards:** Medium-high density in expanded view. Icon + title + 2-4 line description. Max 5 lines before truncation/scroll.

---

## 12. Enemy Visual Hierarchy

### 12.1 States

| State | Visual Treatment |
|---|---|
| **Normal** | Full-opacity portrait, idle pose. HP bar full or mid-fill. Telegraph indicator shows next action. |
| **Damaged** | Brief hit-reaction: portrait shifts slightly or flashes. HP bar decrements with animated fill. Returns to Normal after the reaction completes. |
| **Dangerous** (Move Limit low, or board-threatening Telegraph) | Telegraph indicator uses a warning-register visual treatment (color + icon change). HP bar itself does not change. |
| **Defeated** | A distinct defeat animation. Portrait changes to defeat pose or fades. HP bar shows zero fill. Visually unambiguous from a mere damage reaction. Transitions to Victory beat. |

### 12.2 What Enemy Visuals Must NOT Do

- Must NOT animate reactively to the player's drag position (enforced by `06` Section 9).
- Must NOT display a Player HP equivalent.
- Must NOT be so visually complex that it competes with the board at normal viewing distance.
- Must NOT use a visual state not defined in this section at Absolute MVP.

---

## 13. Feedback Hierarchy

**Core principle:** Not every event is equally important. If everything is loud, nothing communicates.

### Tier 1 — Minor Interaction (Barely Perceptible)

**Triggers:** Piece pickup lift animation, invalid-placement return, tray piece hover highlight.

**Visual treatment:** Micro-scale change only (scale shift <= 5%, opacity shift <= 10%). No color flash. No screen-layer effect.

**Duration:** 1-2 frames for state change; return animation brief.

### Tier 2 — Normal Placement (Acknowledged)

**Triggers:** A piece placed successfully with no line clear (L=0).

**Visual treatment:** Piece locks into position with zero positional jump. Subtle "settle" scale pulse on placed blocks only (<= 3% scale, <= 100ms). No screen-layer effect.

**Duration:** <= 150ms total.

**BINDING RULE:** A placement that produces no line clear must not produce any visual event that could be mistaken for a line clear. Tier 2 must be visually lower-intensity than Tier 3.

### Tier 3 — Line Clear (Satisfying)

**Triggers:** Any placement resulting in L=1 (one line cleared).

**Visual treatment:** Cleared cells flash briefly before collapsing. Cells above/below collapse with smooth fill animation. Damage number pops at the Feedback Zone edge (near the cleared line, not centered on the board). Enemy HP bar decrements. No full-screen effect.

**Duration:** <= 400ms total for clear + collapse + damage number appearance. Damage number may persist up to 600ms before fading.

### Tier 4 — Combo / Multi-Clear (Rewarding)

**Triggers:** Any placement resulting in L=2, L=3, or L=4 (Full Combo).

**Visual treatment:** All Tier 3 elements apply, escalated. Multiple lines flash simultaneously. Damage number is larger (1.2x, 1.5x, or 2.0x base size per Section 6.4). A brief peripheral screen effect (screen edges only — never center-of-board) for L=3 or L=4. For L=4: brief full-edge flash + distinctly heavier haptic pattern.

**Duration:** <= 800ms total. L=4 may extend to 1000ms. Player input is never blocked.

### Tier 5 — Major Combat / Boss Event (Spectacular)

**Triggers:** Enemy defeat, Boss phase transition, Run-ending Board Lock.

**Visual treatment:**
- Enemy defeat: Full distinct defeat animation. Brief full-screen flash in a positive register color. Transition to Victory screen.
- Boss phase transition: Brief pacing pause — HP bar segmented marker advances, screen offset equivalent plays, Telegraph resets visibly.
- Board Lock: Slow, calm highlight sweeps all unplaceable cells. Tone is resignation, not explosion. Transition to Defeat screen.

**Duration:** Defeat/Victory beat: 600-1200ms. Board Lock sweep: 800-1500ms.

**BINDING RULE:** Tier 5 events may only occur at the moments defined above. Using Tier 5 treatment for anything else dilutes impact.

---

## 14. Background Design

### 14.1 Philosophy

The background is the **least important visual layer.** Its purpose:
1. Provide a neutral contrast surface.
2. Establish the current environment/theme through color register and extremely subtle texture.
3. Disappear from conscious awareness once the player is focused on the board.

A background that a playtester explicitly notices and comments on during play has failed.

### 14.2 Treatment Rules

| Rule | Rationale |
|---|---|
| **No high-contrast backgrounds.** | A busy background competes with the board's Priority 1 status. |
| **No moving/animated backgrounds at Absolute MVP.** | Ambient animation competes with Ghost Preview and piece animations. |
| **No photorealistic textures.** | Inconsistent with the game's 2.5D design language. |
| **No stark white or stark black.** | Both create harsh contrast with UI panels and boards. |
| **No background element with a distinct shape or silhouette competing with board cells.** | Players must not confuse background elements with board state. |
| **Background color register must shift between themes meaningfully but subtly.** | Theme change must be perceptible but not demand the player's attention. |

### 14.3 Composition

Backgrounds may use:
- A base color (defined by the active theme, Section 16).
- A very subtle radial or linear gradient (<= 15% brightness shift from edge to center).
- An optional barely-perceptible geometric micro-pattern (grid dots, subtle diagonal weave) at <= 5% opacity.
- Theme-specific ambient elements restricted to the extreme top or side edges, never behind the board.

---

## 15. Decorative Philosophy

### 15.1 What Decoration Is Permitted

| Category | Rules |
|---|---|
| **Environmental flavor accents** | Small, low-opacity illustrative elements in the far background. Must not overlap any Priority 1-4 element at any screen size. |
| **Board surface texture** | Extremely subtle grid texture below the cells. Must not reduce cell readability. Must not look like a placed block. |
| **Enemy portrait illustration** | The enemy character art carries the most decoration budget in the game. |
| **Relic card illustration** | Each relic card's icon/illustration is a primary decorative investment. |
| **UI micro-detail** | Subtle corner accents, hairline separators, icon micro-fills. Never animated at Absolute MVP. |

### 15.2 What Decoration Is Forbidden

- Any decoration that moves or animates during `ACTIVE` gameplay.
- Any decoration overlapping the board zone.
- Any decoration using the same visual language as gameplay feedback.
- Any decoration introducing a new color not present in the Color System doc.
- Decoration on UI panels/buttons not specified in this document.

---

## 16. Environment / Theming Strategy

### 16.1 Theme Structure

Every battle environment is defined by **four visual parameters**:

| Parameter | Definition |
|---|---|
| **Background color register** | A base hue/value for the background plane. |
| **Accent color** | A theme-specific highlight color used in environmental accents and the enemy's primary color. |
| **Board surface tint** | A very slight tint on the board surface. Near-imperceptible alone; creates cohesion with the environment. |
| **Ambient shape family** | The optional micro-pattern associated with this theme. |

### 16.2 MVP Theme Set

| Theme | Background register | Accent register | Notes |
|---|---|---|---|
| **Field / Default** | Warm neutral (earthy mid-tone) | Warm amber/gold | Tutorial and early-run encounters. |
| **Cave / Stone** | Cool dark neutral (slate/charcoal) | Muted teal/green | Mid-run standard encounters, Stone Golem archetype. |
| **Frost / Ice** | Very cool desaturated light | Pale blue-white | Dependent on FROZEN mechanic activation; may be MVP+ content. |
| **Boss Arena** | High-contrast dark neutral | Saturated accent (specific to Boss archetype) | Used only for Boss encounters. |

### 16.3 Theme Rules

- A theme defines **only the four parameters above** — no new UI chrome, board geometry, or HUD layouts.
- Theme changes between battles are communicated by the Battle Start sequence.
- A future theme can be added by populating the four parameters above — no new code or art systems required.

---

## 17. Responsive Visual Rules

### 17.1 Scaling Principles

| Principle | Rule |
|---|---|
| **Board-first sizing** | The board is sized first (largest perfect square fitting the available content area minus Top Zone and Tray Zone). |
| **Type scales proportionally** | All type sizes scale relative to screen base dp. No text shrinks below minimums in Section 6.3. |
| **Spacing tokens hold** | Spacing tokens are fixed dp values. On larger screens, extra spacing is absorbed by zones' internal padding. |
| **No layout restructure below 360dp width** | Below 360dp, board may reduce safe-area padding before any other element is reduced. |
| **Icons do not scale below their minimum tier** | No icon renders below XS tier (12x12dp). |

### 17.2 Device-Class Variants

| Device class | Visual adjustment |
|---|---|
| **Small phones (<=360dp width)** | Reduce XL/XXL spacing tokens to L where used as outer margins. Collapse Bonus Objective by default. Score element hidden unless expanded. |
| **Standard phones (360-430dp width)** | Baseline layout. No adjustments. |
| **Large phones (430-500dp width)** | Allow Objective descriptions to render expanded by default. |
| **Tablets (if supported)** | Side-docked HUD tentatively specified; deferred as Open Decision per `08` Section 17. |

### 17.3 Aspect Ratio Variance

- **Tall aspect ratios:** Extra vertical space added to Top Zone's padding — never stretches the board taller.
- **Short aspect ratios:** Board Zone sized first; Top and Tray Zones compress to minimum heights.
- **The board never renders non-square** — no exception exists.

---

## 18. Lighting Language

### 18.1 Light Direction

**Single consistent source: top-left at approximately 315 degrees** (measuring from the right edge clockwise). Globally binding for all game assets.

### 18.2 Softness

The light source is **large and soft** — not a point light. Character of illumination: overcast daylight, not direct sunlight.

- Hard specular highlights are forbidden on UI elements.
- Soft gradient highlights on block surfaces are the primary depth cue.

### 18.3 Highlight Placement

- **Block top-left face:** Primary highlight. +15-25% brightness from base block color.
- **Block bottom-right face:** Shadow side. -10-20% brightness from base.
- **UI panel top edge:** Very subtle 1dp highlight in a lighter value.
- **Button surface:** Very subtle inner gradient, top edge to center, <= 10% gradient range.

### 18.4 Shadow Direction

All cast shadows fall to the **bottom-right** (X +2dp, Y +3dp at Mid elevation, scaling with elevation tier). Shadow color: dark tint of the background palette, never pure black or pure grey.

### 18.5 Material Consistency

- **Placed blocks:** Opaque and matte. Low specularity, soft light catch.
- **Ghost Preview:** Semi-transparent, no highlight. Ghosts have no material.
- **Blocked cells:** Dull and desaturated. Communicates "inert."
- Enemy character art is allowed its own material language, but must not conflict with the block language.

---

## 19. Visual Do / Dont System

### 19.1 Board

| DO | DONT |
|---|---|
| Make board cells clearly readable as distinct units from any angle. | Make cells so small they are unreadable at arm-length on a standard phone. |
| Use a subtle, low-contrast grid line to separate cells. | Use a high-contrast or thick grid that visually dominates. |
| Give filled cells a consistent depth treatment per Section 9. | Give different piece types different depth treatments — all filled cells share one material. |
| Make empty cells distinctly different from filled cells. | Make empty cells so dark the board reads as mostly filled when it is not. |
| Use a slight board surface tint matching the current theme. | Change the board's shape, size, or grid count based on theme. |

### 19.2 Pieces

| DO | DONT |
|---|---|
| Give each piece a distinct color identity within a shared palette. | Assign colors indistinguishable under colorblindness without a pattern or shape cue. |
| Make the dragged piece visually elevated (Active/4 elevation tier). | Make the dragged piece look the same as a placed piece. |
| Show the Ghost Preview at reduced opacity with a non-color validity cue. | Make the Ghost Preview the same opacity as a real piece, or omit the validity state. |
| Return a piece to its tray slot with a smooth brief animation on invalid placement. | Leave a piece stranded at the invalid position, or return it with no animation. |

### 19.3 Enemy and Objectives

| DO | DONT |
|---|---|
| Show the enemy HP bar as the largest, most prominent element in the Top Zone. | Let the enemy portrait dominate the Top Zone at the HP bar's expense. |
| Always show the Telegraph indicator at the same prominence as the HP bar. | Hide, collapse, or de-emphasize the Telegraph. |
| Show Objective progress as a live fraction updating per `07` Section 12's timing. | Batch-update Objective progress at Turn end. |

### 19.4 Feedback

| DO | DONT |
|---|---|
| Scale damage numbers with the magnitude of the combo (Section 6.4). | Show the same-sized damage number for a 1-line clear and a 4-line Full Combo. |
| Make line-clear feedback clearly distinct from piece-placement feedback. | Use the same animation or sound for Tier 2 and Tier 3 events. |
| Keep all feedback within the Feedback Zone or the affected board area. | Flash the entire screen for a single-line clear (reserve full-screen effects for Tier 5 only). |
| Let Board Feedback and Combat Feedback play concurrently (per `06` Section 12). | Wait for Board Feedback to fully complete before starting Combat Feedback. |

### 19.5 Typography

| DO | DONT |
|---|---|
| Use tabular numerals for all live-updating numbers. | Use proportional numerals for HP, score, or objective counts. |
| Weight hierarchy: heavier = more important. | Use bold for decoration or emphasis unrelated to information priority. |
| Scale damage number to display weight with combo magnitude. | Use the same weight for a damage number and a body-copy description. |

### 19.6 Spacing and Layout

| DO | DONT |
|---|---|
| Use spacing tokens from Section 7 for every gap and padding. | Set arbitrary spacing values (6dp, 10dp, 14dp) per element. |
| Keep the board centered and square on every device class. | Stretch the board to fill extra horizontal space on wide screens. |
| Respect safe areas: notch, home indicator, gesture zones. | Place interactive elements under any system-reserved zone. |

### 19.7 Color (pending Color System doc)

| DO | DONT |
|---|---|
| Reserve saturated/high-contrast colors for feedback and interactive elements. | Apply saturated colors to decorative or background elements. |
| Pair every color cue with a non-color cue (icon, pattern, shape). | Use color as the sole carrier of critical information. |
| Use the current theme's accent color only for environmental accents. | Apply the accent color to primary board cells, HP bars, or objective progress. |

---

## 20. Production Asset Standards

### 20.1 Naming Convention

All assets follow this naming schema:

```
[category]_[element]_[variant]_[state]@[scale].[format]
```

**Categories:** `board`, `piece`, `ui`, `enemy`, `relic`, `bg`, `fx`, `font`

**Examples:**
```
piece_block_filled_normal@2x.png
piece_block_ghost_valid@2x.png
enemy_goblin_normal@2x.png
enemy_goblin_damaged@2x.png
ui_button_primary_normal@2x.png
ui_button_primary_disabled@2x.png
ui_icon_pause_filled@2x.svg
relic_card_icon_rlc001@2x.png
bg_field_base@2x.png
```

### 20.2 Resolution Standards

| Asset type | Export scales | Rationale |
|---|---|---|
| UI elements (buttons, panels, icons) | @1x, @2x, @3x | Cover standard, retina, and high-density mobile displays. |
| Board and piece blocks | @2x minimum, @3x for flagship devices | Piece blocks are the most-seen element; never appear blurry. |
| Enemy portraits | @2x minimum | Constrained to Top Zone; @3x optional. |
| Relic card art | @2x, @3x | Shown in a modal moment; quality matters for retention. |
| Background layers | @1x + @2x | Backgrounds are not detail elements. |
| Icons | SVG preferred; PNG @2x and @3x fallback | SVG scales without multiple exports. |

### 20.3 Formats

| Format | Usage |
|---|---|
| **SVG** | All icons. All simple geometric UI shapes. |
| **PNG (with transparency)** | Character art, piece blocks, assets with complex edges or gradients. |
| **WEBP** | Background layers on Android targets. PNG fallback always provided. |
| **JPG** | Not used. |
| **Font (TTF/OTF)** | Variable font format preferred. Subset to characters actually used. |

### 20.4 Transparency

- All assets requiring transparency export with **premultiplied alpha** unless the target engine requires straight alpha.
- No white or dark fringes on transparent exports.

### 20.5 Sprite / UI Requirements

- **Nine-patch (9-slice):** All scalable UI containers must be defined with a 9-patch specification.
- **Sprite atlasing:** All assets in the same visual category should be combined into a single sprite atlas.
- **Pivot points:** All animated assets must have pivot/anchor explicitly defined. A piece's anchor is its logical (0,0) origin per `02_SHAPE_LIBRARY.md` Section 2.

### 20.6 Consistency Requirements

- All piece blocks of the same fill state must look identical in depth treatment, regardless of piece shape.
- All enemy portraits must share the same art style (owned by the Character Art doc).
- All relic card icons must share visual weight — heavier-looking icons create false rarity signaling.
- No asset may be shipped at a different style level than its neighboring assets.

---

## 21. Visual QA Checklist

### 21.1 Hierarchy

- [ ] The board occupies the largest visual footprint on the Battle screen.
- [ ] No Priority 5 (decorative) element is larger or more visually prominent than any Priority 1-2 element on the same screen.
- [ ] The enemy HP bar is clearly the most prominent element in the Top Zone.
- [ ] The Primary Objective is visible without any tap or scroll from the moment a Battle begins.
- [ ] No Player Health element appears anywhere on any screen.

### 21.2 Readability

- [ ] Board cells are distinctly readable as individual units at arm-length on a 360dp device.
- [ ] Ghost Preview valid state is clearly distinguishable from Ghost Preview invalid state without relying on color alone.
- [ ] Ghost Preview invalid (Occupied) is visually distinguishable from Ghost Preview invalid (Out-of-Bounds).
- [ ] All text meets WCAG 2.1 AA contrast ratios on all screens, in all states.
- [ ] Tabular numerals are used for all live-updating numeric displays.
- [ ] No text block is truncated or clipped on the smallest supported device class (360dp width).

### 21.3 Depth and Elevation

- [ ] The dragged piece appears visually elevated above the board and placed pieces.
- [ ] Ghost Preview appears below the dragged piece in z-order and at a lower depth cue.
- [ ] UI panels appear above the board in z-order.
- [ ] No two elements at different elevation tiers share the same shadow treatment.

### 21.4 Feedback

- [ ] Tier 2 (placement with no clear) is visually lower-intensity than Tier 3 (line clear).
- [ ] Tier 3 (single line clear) is visually lower-intensity than Tier 4 (multi-line combo).
- [ ] A Tier 5 visual effect never plays for a non-Tier-5 event.
- [ ] No Combat-layer visual fires before a Commit event during a drag.
- [ ] Damage number position does not overlap the active placement area during ACTIVE input.

### 21.5 Consistency

- [ ] All buttons of the same type use the same corner radius on every screen.
- [ ] All spacing values map to the Section 7 spacing token system (no arbitrary values).
- [ ] All icons use the filled style and are drawn on the standard icon grid.
- [ ] All block assets of the same fill state are visually identical regardless of piece shape.
- [ ] The light direction is consistent (top-left) across all assets on a screen.
- [ ] Shadow direction is consistent (bottom-right offset) across all UI elements.

### 21.6 Accessibility

- [ ] Every color cue is paired with a non-color cue for all gameplay-critical states.
- [ ] Reduced-motion toggle removes non-essential animations without removing information.
- [ ] All touch targets meet or exceed platform minimum size recommendations.
- [ ] No interactive element is placed within system-reserved gesture zones.

### 21.7 Production Standards

- [ ] All assets follow the naming convention in Section 20.1.
- [ ] All icons are exported as SVG or as PNG at @2x and @3x.
- [ ] All scalable UI containers are defined with a valid 9-patch specification.
- [ ] No placeholder or watermarked asset is present in the build.
- [ ] All assets are at a consistent production-readiness level on the same screen.

### 21.8 Theme

- [ ] The active theme's background register is clearly distinct from at least two other themes at a glance.
- [ ] No background element is mistakable for a board cell or a placed piece.
- [ ] Theme accent colors do not repeat piece block colors in a way that creates visual ambiguity.

---

## 22. Cross-Document Dependencies

| Dependent Document | What it must respect from this document |
|---|---|
| **Color System doc** (future) | Section 1 visual identity keywords and anti-keywords; Section 3.6 lighting philosophy; Section 9 depth/shadow color language; Section 13 feedback tier color registers; Section 16 theme accent color families. The Color System doc owns exact values — this document owns the usage rules those values must serve. |
| **Block / Piece Art doc** (future) | Section 3.1 shape language (M corner radius for blocks); Section 3.6 lighting philosophy (top-left, soft); Section 9 elevation tiers (Mid/2 for placed, Active/4 for dragged); Section 18 material consistency (matte, no specular); Section 20 production standards. |
| **Animation doc** (future) | Section 13 feedback tier definitions; Section 11 Battle Start sequence timing; Section 12 Victory beat structure; Section 3.4 softness/sharpness; Section 17 responsive layout. |
| **VFX doc** (future) | Section 13 feedback tiers (Tier 4 peripheral screen effect, Tier 5 full-screen flash); Section 14.2 background animation restrictions; Section 15 decoration prohibitions; Section 20.1 asset naming for fx_ category. |
| **Audio doc** (future) | Section 13 feedback tiers (audio intensity must match visual intensity per tier); Section 6.4 damage number magnitude scaling; Section 1.2 mood table. |
| **Content / Copy doc** (future) | Section 6 typography philosophy (plain language, short lines); Section 11.6 information density (card body max line count); Section 9 objective communication (names must fit within chip/label treatments). |
| **Character Art / Enemy doc** (future) | Section 12 enemy visual hierarchy (four states defined); Section 3.6 lighting philosophy; Section 15 decoration budget. |
| **Accessibility / Settings doc** (future) | Section 21.6 accessibility requirements; Section 6.3 readability minimums; Section 17.2 responsive rules for small devices. |

---

## 23. Final Visual Identity Contract

This is the binding visual constitution for Block Battles. Every design decision, every asset, and every future document must be traceable to one or more of the following principles:

---

> **The board is the game. Every visual element exists to make the board more readable, more satisfying, or more contextually meaningful — not to draw attention to itself.**

> **Clarity is not minimalism. The game has character, depth, and warmth. But every element with character has earned its place by making the experience better, not by filling empty space.**

> **Feedback is earned. Not every moment is loud. A single-line clear and a four-line Full Combo must feel qualitatively different — that difference is the reward for skill. If everything is a celebration, nothing is.**

> **Human-made, not generated. This game looks like a person made considered decisions. The spacing is deliberate. The radii are consistent. The type has hierarchy. The depth is logical. These are the signals of craft.**

> **Accessible by construction, not by exception. Color is never the sole carrier of information. Touch targets are generous. Animations can be reduced. These are not add-ons — they are the baseline from which the visual system is built.**

> **The system is the standard. One corner radius token. One spacing scale. One light source. One icon style. Consistency compounds — every element that follows the system makes every other element look more intentional.**

---

**Every future visual contributor to this project inherits this contract. A deviation from any of the six principles above requires a revision to this document — not a silent exception.**

---

**End of `09_VISUAL_DESIGN_SYSTEM.md`.**
