# 10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md
## Block Battles — Color System and Block Art Direction

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`, `04_SPAWN_RNG_AND_DIFFICULTY.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `06_INPUT_DRAG_AND_DROP_UX.md`, `07_BATTLE_OBJECTIVES_AND_LEVEL_DESIGN.md`, `08_UI_UX_MASTER_SPECIFICATION.md`, `09_VISUAL_DESIGN_SYSTEM.md`
**Document status:** Layer B — Color Authority

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23. It is the **Color System doc** referenced in `09_VISUAL_DESIGN_SYSTEM.md` Section 22's Cross-Document Contract. It introduces no new gameplay mechanics. It owns all canonical color values, the full block color family, semantic color usage rules, accessibility specifications, and the master color token registry. Every future document that references a color value must use the tokens defined in Section 18 of this document — never invent a new color value without a revision here first.

---

## 1. Color Philosophy

### 1.1 Core Mandate

Color in Block Battles has **one primary job**: make the board state instantly readable. Every other use of color — enemy identity, feedback intensity, relic presentation, UI chrome — is secondary to that job and must never compromise it.

Three principles govern every color decision in this document:

1. **Color serves function, not aesthetics.** A color is justified when it makes a gameplay-critical distinction clearer. It is unjustified when it merely makes something look more visually interesting. Both can coexist, but when they conflict, function wins.

2. **Color is never the only signal.** This is a hard constraint inherited from `09_VISUAL_DESIGN_SYSTEM.md` Section 16 (Visual Do/Don't, color rules) and `00_MASTER_GAME_VISION.md` Section 19 (accessibility). Every color cue that carries gameplay-critical information must be accompanied by at least one non-color cue (shape, icon, pattern, position, text). The system is designed so that a player with total color blindness can still distinguish: valid vs. invalid placement, enemy HP status, objective progress state, and board lock warning.

3. **Color has a semantic hierarchy.** Each color register (danger, warning, success, neutral, accent) has a defined meaning and that meaning is consistent across the entire game. A color used for "danger" in one context must not be used decoratively in another context. Semantic drift is the primary failure mode this document is designed to prevent.

### 1.2 Palette Character

The palette is built around a **warm-neutral base** with **purposeful accent saturation**. The goal is a palette that:

- Feels premium and handcrafted — not the flat-primary default of casual games.
- Has enough warmth to feel inviting, without tipping into "children's game."
- Uses saturation sparingly — high-saturation colors are reserved for gameplay-critical feedback and piece identity; background/UI surfaces stay desaturated and warm.
- Reads clearly in both bright and dim ambient lighting conditions (mobile game reality: played in both).

### 1.3 What the Palette Is NOT

- **Not neon.** No fully-saturated electric colors (HSL saturation >= 95%) outside of the specific feedback-flash colors defined in Section 8.
- **Not pastel.** No washed-out, low-saturation piece colors that fail to distinguish from the board surface at a glance.
- **Not monochromatic.** Piece colors must be distinct enough that, even allowing for color-vision deficiency mitigations, a player can tell different pieces apart.
- **Not rainbow-per-turn.** Colors are stable and consistent. A piece that is red is always red, in every tray, in every battle, in every run.

---

## 2. Global Palette Architecture

The entire palette is organized into **seven semantic groups**. All color tokens in Section 18 belong to one and only one group.

### Group 1 — Background

Colors that form the furthest visual layer. Appear behind the board, behind panels, behind everything. Must be low-contrast and non-distracting. Shift subtly between themes (Section 12).

### Group 2 — Surface

Colors for UI panels, cards, buttons, and HUD elements. Sit above the background but below gameplay content. Include elevated and semi-transparent variants.

### Group 3 — Board

Colors for the board grid surface, cell walls/dividers, the board outer border. Distinct from placed blocks but harmonious with them.

### Group 4 — Block / Piece

The piece color family. Seven distinct color identities for the seven active piece color slots (Section 4). These are the highest-saturation colors on the board.

### Group 5 — Semantic Interactive

Colors that carry consistent gameplay meaning: success (green register), warning (amber register), danger (red register). Used for HP states, objective progress, move-limit pressure, board lock warning.

### Group 6 — Text and Icon

Colors for all typographic content. Must meet WCAG 2.1 AA contrast against all surfaces they appear on. Includes primary, secondary, and inverse text.

### Group 7 — Combat and Feedback

Colors used specifically for transient feedback events: damage numbers, line-clear flash, combo flash, enemy hit-reaction, board-lock sweep. These are allowed to be higher-saturation because they are brief and contextually earned.

---

## 3. Exact Color Registry

**BINDING RULE:** This is the single source of truth for all color values in the project. No color value appearing in any other document or in any code file is authoritative unless it matches the HEX value registered here. Discrepancies are resolved in favor of this document.

All color values are specified for the **default (dark-base) environment** (Section 12). Light environment variants are specified in Section 12 where they differ.

### 3.1 Background Colors

| ID | Semantic Purpose | HEX | RGB | HSL | Usage Notes |
|---|---|---|---|---|---|
| `CLR-BG-100` | Background base — deepest layer | `#1A1814` | 26, 24, 20 | 36, 13%, 9% | Default screen background. Warm dark neutral. Never pure black. |
| `CLR-BG-200` | Background elevated — subtle lift | `#221F1B` | 34, 31, 27 | 33, 11%, 12% | Background on content screens where BG-100 needs a slight lift. |
| `CLR-BG-FIELD` | Field theme background | `#1E1C16` | 30, 28, 22 | 43, 15%, 10% | Active when Field/Default theme is loaded. |
| `CLR-BG-CAVE` | Cave/Stone theme background | `#161A1C` | 22, 26, 28 | 199, 12%, 10% | Active when Cave/Stone theme is loaded. |
| `CLR-BG-ICE` | Frost/Ice theme background | `#161C22` | 22, 28, 34 | 214, 21%, 11% | Active when Frost/Ice theme is loaded. MVP+ dependency. |
| `CLR-BG-BOSS` | Boss Arena theme background | `#18141A` | 24, 20, 26 | 285, 13%, 9% | Active when Boss Arena theme is loaded. |

### 3.2 Surface Colors

| ID | Semantic Purpose | HEX | RGB | HSL | Usage Notes |
|---|---|---|---|---|---|
| `CLR-SRF-100` | Surface base — panels, HUD zones | `#2C2820` | 44, 40, 32 | 36, 16%, 15% | Primary surface for Top Zone and Tray Zone panels. |
| `CLR-SRF-200` | Surface elevated — cards | `#342F26` | 52, 47, 38 | 38, 15%, 18% | Relic cards, tooltip cards. Slightly lighter than SRF-100. |
| `CLR-SRF-300` | Surface high-elevated — selected cards | `#3D382D` | 61, 56, 45 | 40, 15%, 21% | Selected/highlighted card state (Relic Selection, pre-confirm). |
| `CLR-SRF-BORDER` | Surface border — panel/card edges | `#4A4438` | 74, 68, 56 | 38, 14%, 25% | 1px borders on panels and cards. Lighter than surface, never dark. |
| `CLR-SRF-SCRIM` | Modal scrim | `#00000099` | 0, 0, 0, 60% | — | Semi-transparent overlay behind modals. 60% black opacity. |

### 3.3 Board Colors

| ID | Semantic Purpose | HEX | RGB | HSL | Usage Notes |
|---|---|---|---|---|---|
| `CLR-BRD-SURFACE` | Board surface — the grid field | `#252018` | 37, 32, 24 | 38, 21%, 12% | The base color of the 8x8 board area, below all cells. |
| `CLR-BRD-EMPTY` | Empty cell — unfilled board cell | `#2E2920` | 46, 41, 32 | 38, 18%, 15% | An unoccupied cell. Must be clearly distinct from BRD-SURFACE and from any placed piece. |
| `CLR-BRD-EMPTY-BORDER` | Empty cell border — grid lines | `#3A3428` | 58, 52, 40 | 38, 18%, 19% | The subtle border/divider between empty cells. Low contrast — present but not dominant. |
| `CLR-BRD-HIGHLIGHT` | Empty cell highlight — valid drop target | `#4A4230` | 74, 66, 48 | 40, 21%, 24% | Highlights the candidate drop zone area (cells the ghost piece would occupy if valid). |

### 3.4 Block / Piece Colors

Seven color identities are defined. Pieces are assigned a color identity at spawn based on the rotation system in Section 5. Each identity has three values: **Base** (the main block face), **Light** (the top/left highlight face), and **Shadow** (the bottom/right shadow face). These implement the lighting model from `09_VISUAL_DESIGN_SYSTEM.md` Sections 3.6 and 18.

| Identity | ID Base | HEX Base | ID Light | HEX Light | ID Shadow | HEX Shadow | Semantic tag |
|---|---|---|---|---|---|---|---|
| **Ember** | `CLR-PC-EMB-BASE` | `#C85A2A` | `CLR-PC-EMB-LIGHT` | `#E8784A` | `CLR-PC-EMB-SHADOW` | `#8A3A18` | Warm red-orange. Primary active energy. |
| **Amber** | `CLR-PC-AMB-BASE` | `#C89A28` | `CLR-PC-AMB-LIGHT` | `#E8BC48` | `CLR-PC-AMB-SHADOW` | `#8A6A18` | Warm gold-yellow. Harvest/treasure feel. |
| **Leaf** | `CLR-PC-LEA-BASE` | `#3A8A48` | `CLR-PC-LEA-LIGHT` | `#58AA66` | `CLR-PC-LEA-SHADOW` | `#265A30` | Mid-value green. Grounded, natural. |
| **Tide** | `CLR-PC-TID-BASE` | `#2A7A9A` | `CLR-PC-TID-LIGHT` | `#48A0BE` | `CLR-PC-TID-SHADOW` | `#1A4E64` | Cool teal-blue. Distinct from Leaf and Iris. |
| **Iris** | `CLR-PC-IRS-BASE` | `#6A3AAA` | `CLR-PC-IRS-LIGHT` | `#8A5AC8` | `CLR-PC-IRS-SHADOW` | `#44246E` | Violet-purple. Mystical, distinct from Tide. |
| **Rose** | `CLR-PC-ROS-BASE` | `#AA3A5A` | `CLR-PC-ROS-LIGHT` | `#C85A7A` | `CLR-PC-ROS-SHADOW` | `#6E2238` | Deep rose-crimson. Warm, distinct from Ember. |
| **Slate** | `CLR-PC-SLT-BASE` | `#4A6A7A` | `CLR-PC-SLT-LIGHT` | `#6A8A9A` | `CLR-PC-SLT-SHADOW` | `#2E4452` | Cool grey-blue. Visually neutral among the six brighter identities. |

**Rationale for seven identities:** The game's piece tray holds 3 pieces at a time. Seven color identities provide sufficient variety that consecutive pieces rarely share a color, while keeping the palette small enough to be memorable. Identity seven (Slate) functions as a visual "rest" among the more vibrant identities.

### 3.5 Semantic Interactive Colors

| ID | Semantic Purpose | HEX | RGB | HSL | Accessibility notes |
|---|---|---|---|---|---|
| `CLR-SEM-SUCCESS` | Success / Positive state | `#4AA86A` | 74, 168, 106 | 143, 38%, 47% | Must pair with a non-color cue (checkmark icon). Checked against protanopia/deuteranopia in Section 11. |
| `CLR-SEM-SUCCESS-DIM` | Success / Dimmed (progress incomplete) | `#2E6642` | 46, 102, 66 | 143, 38%, 29% | Used for incomplete objective progress bar fill at early stages. |
| `CLR-SEM-WARNING` | Warning / Pressure state | `#C88A28` | 200, 138, 40 | 38, 66%, 47% | Move Limit low, HP bar mid-low. Must pair with a non-color cue (icon or numeric). |
| `CLR-SEM-WARNING-DIM` | Warning / Dimmed | `#7A5418` | 122, 84, 24 | 38, 67%, 28% | Track fill for bars in warning register. |
| `CLR-SEM-DANGER` | Danger / Critical state | `#C83A2A` | 200, 58, 42 | 4, 65%, 47% | HP critically low, Move Limit final turns, Board Lock imminent. Always paired with a non-color cue. |
| `CLR-SEM-DANGER-DIM` | Danger / Dimmed | `#7A2218` | 122, 34, 24 | 4, 67%, 28% | Track fill for bars in danger register. |
| `CLR-SEM-NEUTRAL` | Neutral / Default state | `#6A6258` | 106, 98, 88 | 33, 9%, 38% | Default state for bars and indicators not yet in a semantic state. |
| `CLR-SEM-NEUTRAL-DIM` | Neutral / Track fill | `#3A3428` | 58, 52, 40 | 38, 18%, 19% | Track background for neutral-state bars (same as BRD-EMPTY-BORDER — intentional: the board's rhythm carries into the HUD). |

### 3.6 Text and Icon Colors

| ID | Semantic Purpose | HEX | RGB | HSL | Contrast ratio | Usage |
|---|---|---|---|---|---|---|
| `CLR-TXT-PRIMARY` | Primary text — maximum readability | `#F0EAE0` | 240, 234, 224 | 36, 44%, 91% | 15.2:1 vs CLR-SRF-100 | All primary body text, labels, headers. |
| `CLR-TXT-SECONDARY` | Secondary text — reduced emphasis | `#A89E90` | 168, 158, 144 | 36, 14%, 61% | 6.1:1 vs CLR-SRF-100 | Secondary labels, captions, metadata. |
| `CLR-TXT-DISABLED` | Disabled text | `#6A6258` | 106, 98, 88 | 33, 9%, 38% | 3.2:1 vs CLR-SRF-100 | Disabled button labels, inactive state text. Meets 3:1 for large/bold text only. |
| `CLR-TXT-INVERSE` | Inverse text — on light surfaces | `#1A1814` | 26, 24, 20 | 36, 13%, 9% | — | If any light surface is introduced (MVP+ decision, Section 12). |
| `CLR-TXT-ACCENT` | Accent text — highlighted values | `#E8BC48` | 232, 188, 72 | 43, 77%, 60% | 7.8:1 vs CLR-SRF-100 | Highlighted numbers (score flash, bonus label), selected chip text. |
| `CLR-ICN-PRIMARY` | Primary icon | `#F0EAE0` | 240, 234, 224 | 36, 44%, 91% | — | Same as TXT-PRIMARY. Standard filled icon color. |
| `CLR-ICN-SECONDARY` | Secondary icon | `#A89E90` | 168, 158, 144 | 36, 14%, 61% | — | Same as TXT-SECONDARY. Inactive/background icons. |
| `CLR-ICN-ACCENT` | Accent icon | `#E8BC48` | 232, 188, 72 | 43, 77%, 60% | — | Same as TXT-ACCENT. Active objective icon, positive feedback icon. |

### 3.7 Combat and Feedback Colors

| ID | Semantic Purpose | HEX | RGB | HSL | Usage |
|---|---|---|---|---|---|
| `CLR-CMB-DAMAGE-1` | Damage number — 1 line | `#F0EAE0` | 240, 234, 224 | 36, 44%, 91% | Single-line clear damage number. White-warm — clearly readable but not alarming. |
| `CLR-CMB-DAMAGE-2` | Damage number — 2 lines | `#E8BC48` | 232, 188, 72 | 43, 77%, 60% | 2-line combo damage. Amber register — energy escalating. |
| `CLR-CMB-DAMAGE-3` | Damage number — 3 lines | `#E87A28` | 232, 122, 40 | 28, 80%, 53% | 3-line combo damage. Orange-ember — hotter. |
| `CLR-CMB-DAMAGE-4` | Damage number — 4 lines (Full Combo) | `#E83A28` | 232, 58, 40 | 5, 80%, 53% | Full Combo damage. Red-hot — maximum earned intensity. |
| `CLR-CMB-CLEAR-FLASH` | Line clear flash — cells | `#F0EAE0` | 240, 234, 224 | 36, 44%, 91% | The flash color on clearing cells. Warm white — brightness event, not color-coded. |
| `CLR-CMB-COMBO-GLOW` | Combo indicator glow | `#E87A28` | 232, 122, 40 | 28, 80%, 53% | The peripheral glow on L=3+ clear. Same as CMB-DAMAGE-3. |
| `CLR-CMB-FULLCOMBO-GLOW` | Full Combo edge flash | `#E83A28` | 232, 58, 40 | 5, 80%, 53% | The full-edge flash on L=4. Same as CMB-DAMAGE-4. |
| `CLR-CMB-ENEMY-HIT` | Enemy hit-reaction flash | `#E83A28` | 232, 58, 40 | 5, 80%, 53% | Brief flash/tint on enemy portrait when taking damage. |
| `CLR-CMB-ENEMY-DEFEAT` | Enemy defeat flash | `#F0EAE0` | 240, 234, 224 | 36, 44%, 91% | Full-screen flash on enemy defeat. Warm white — a flash of light, not a bloodstain. |
| `CLR-CMB-BOARDLOCK` | Board Lock sweep | `#C83A2A` | 200, 58, 42 | 4, 65%, 47% | The slow sweep highlight on all unplaceable cells on Board Lock. Same as SEM-DANGER. |
| `CLR-CMB-TELEGRAPH` | Enemy Telegraph indicator — normal | `#C88A28` | 200, 138, 40 | 38, 66%, 47% | The Telegraph icon/indicator color when an upcoming action is standard. Same as SEM-WARNING. |
| `CLR-CMB-TELEGRAPH-THREAT` | Enemy Telegraph indicator — severe | `#C83A2A` | 200, 58, 42 | 4, 65%, 47% | The Telegraph indicator when an upcoming action is board-threatening. Same as SEM-DANGER. |

---

## 4. Block Color System

### 4.1 The Seven Color Identities

Every piece that appears in the game's tray is rendered using one of the seven color identities defined in Section 3.4. The identities are:

1. **Ember** — warm red-orange
2. **Amber** — warm gold-yellow
3. **Leaf** — mid-value green
4. **Tide** — cool teal-blue
5. **Iris** — violet-purple
6. **Rose** — deep rose-crimson
7. **Slate** — cool grey-blue

### 4.2 Block Rendering — Three-Face Model

Each block cell in a piece is rendered using **three color values** from the assigned identity (per `09_VISUAL_DESIGN_SYSTEM.md` Section 18):

```
Top-left face (highlight face):     [Identity]-LIGHT
Main face (primary face):           [Identity]-BASE
Bottom-right face (shadow face):    [Identity]-SHADOW
```

This three-face model is applied identically to:
- Tray pieces (unplaced)
- Placed pieces on the board
- The dragged piece (Active/4 elevation — the highlight is slightly brighter and the shadow is slightly deeper to reinforce the elevated feeling)

The Ghost Preview (Section 6) does NOT use the three-face model — it uses a flat, semi-transparent fill.

### 4.3 Block Surface Finish

All blocks use the **matte surface material** defined in `09_VISUAL_DESIGN_SYSTEM.md` Section 18.5:
- Low specularity — the highlight is a painted-on brightness, not a reflective spec.
- Soft edges — corner radius M (10dp) per `09` Section 8.
- No outlines — the depth model and the grid background provide cell separation without a drawn border.

### 4.4 Color Identity vs. Shape Identity — Explicit Relationship

**Shape identity and color identity are fully independent in the piece spawning system.** A piece's shape (`02_SHAPE_LIBRARY.md`'s 34 registered orientations) does not determine its color. Color is assigned per the spawn rotation defined in Section 5.

**Implication:** An L-shape may be Ember in one tray and Tide in the next. A 1x1 single block may be Iris. The player's mental model for piece recognition is primarily **shape** (hardcoded: there are 34 orientations) and secondarily **color** (useful for quick tray scan, but not required for correct placement decisions).

This is a deliberate design decision: tying color to shape would require keeping track of which shapes "belong" to which colors, adding cognitive load. The current model is simpler — any piece of any shape can be any color.

---

## 5. Color Assignment Philosophy

### 5.1 The Rotation System

Colors are assigned to pieces using a **modular rotation through the seven identities**, weighted to prevent immediate repetition.

**Rules (in order of priority):**

1. **No two pieces in the same tray share the same color identity.** When a new tray of 3 pieces is generated, the 3 assigned colors must be 3 distinct identities from the 7.
2. **The color identity of the previous tray's last-placed piece is suppressed for the first position of the new tray.** This prevents the most recently seen color from appearing immediately again in position 1 of the next tray.
3. **Within the constraints above, color selection is random** (uniform distribution across remaining eligible identities, per the RNG rules in `04_SPAWN_RNG_AND_DIFFICULTY.md`).

**Why not a fixed rotation?** A fixed rotation (Ember, Amber, Leaf, Tide, Iris, Rose, Slate, Ember, ...) would be deterministic and predictable, which conflicts with the RNG philosophy of `04` Section 1. The rotation is a constraint on randomness, not a replacement for it.

### 5.2 Special Block Colors

At Absolute MVP, no special-purpose block color is defined beyond the standard seven identities. The following are reserved for future use and must not be repurposed for decorative or arbitrary use at Absolute MVP:

| Reserved purpose | Future color register | When it becomes active |
|---|---|---|
| Frozen cell state | A distinct desaturated blue-white register | When `03_BOARD_ENGINE_AND_RULES.md` Section 3.2's `FROZEN` state is activated (MVP+). |
| Blocked cell state | A distinct dark, low-saturation register | When `BLOCKED` state is activated (MVP+). |
| Indestructible cell state | A very dark, near-black register with a slight metallic sheen suggestion | When `INDESTRUCTIBLE` state is activated (MVP+). |
| Hazard cell state | A distinct warm-red register (distinct from piece Ember by being desaturated and dim, not vibrant) | When Hazard mechanic is activated (MVP+). |

**BINDING RULE:** At Absolute MVP, the board contains only two cell states relevant to color: `EMPTY` (CLR-BRD-EMPTY) and `FILLED` (using one of the seven piece color identities). No other cell color is rendered.

### 5.3 Repeated Piece Same Color?

If the same piece shape appears twice in the same tray (which is possible — the shape library has 34 orientations and tray generation is RNG-based), the two instances will have **different color identities** (enforced by Rule 1 of Section 5.1 — no two pieces in the same tray share the same color). Shape repetition and color repetition are both independently avoided at the tray level.

---

## 6. Ghost Preview Colors

The Ghost Preview renders the dragged piece's footprint on the board to show where it will land. It has **four distinct visual states** per `06_INPUT_DRAG_AND_DROP_UX.md` Section 8. Each state requires a color treatment AND a non-color cue (pattern/outline) to meet accessibility requirements.

### 6.1 Valid State

The placement is legal — `validatePlacement` returns `VALID`.

| Property | Value |
|---|---|
| **Fill color** | The piece's assigned color identity BASE color at **45% opacity** |
| **Fill style** | Flat fill (no three-face model — this is a ghost, not a material object per `09` Section 9.3) |
| **Outline** | 1.5px solid outline at the piece's LIGHT color at 80% opacity |
| **Pattern overlay** | None — the clean translucent fill is itself the "valid" signal |
| **Token reference** | Derived from the piece's assigned `CLR-PC-[identity]-BASE` at 45% opacity |

**Non-color cue:** The outline weight is distinctly heavier than the invalid states' pattern treatments, making the Valid state distinguishable by outline presence/absence independent of fill color.

### 6.2 Invalid — Cell Occupied

At least one cell the piece would occupy is already filled.

| Property | Value |
|---|---|
| **Fill color** | `CLR-BRD-EMPTY` at 60% opacity — the piece's own color identity is suppressed; the ghost "blends with the board" to signal it cannot land there |
| **Fill style** | Cross-hatch pattern overlay at `CLR-SEM-DANGER` at 30% opacity, applied specifically to the OCCUPIED cells only (not all cells) |
| **Outline** | Dashed 1px outline at `CLR-SEM-DANGER` at 60% opacity |
| **Pattern overlay** | Cross-hatch on occupied cells specifically |
| **Token reference** | `CLR-BRD-EMPTY`, `CLR-SEM-DANGER` |

**Non-color cue:** Cross-hatch pattern on the specific cells that caused the failure, plus dashed (not solid) outline treatment, making Occupied distinguishable from Valid by line pattern alone.

### 6.3 Invalid — Out of Bounds

One or more cells of the piece would fall outside the board.

| Property | Value |
|---|---|
| **Fill color** | `CLR-BRD-EMPTY` at 60% opacity, same base as Occupied Invalid |
| **Fill style** | Flat fill on in-bounds cells; clipped/absent on out-of-bounds cells — the out-of-bounds cells simply have no ghost render (they are beyond the board boundary) |
| **Outline** | Solid 1px outline at `CLR-SEM-WARNING` at 70% opacity (amber, not red — this is a positioning issue, not a conflict issue) |
| **Pattern overlay** | None on in-bounds cells; edge clipping at the board boundary |
| **Token reference** | `CLR-BRD-EMPTY`, `CLR-SEM-WARNING` |

**Non-color cue:** The truncation/clipping of ghost cells at the board edge is itself the primary non-color cue — the piece visually "runs out of board." The amber vs. red outline distinction between Out-of-Bounds and Occupied provides a color-level distinction; the clipped rendering provides the non-color-level distinction.

### 6.4 No Board Contact

The piece is being dragged but is not positioned over or near the board (pointer is in the Tray area or off-screen).

| Property | Value |
|---|---|
| **Ghost render** | None — no ghost is drawn on the board at all |
| **Token reference** | N/A |

**Non-color cue:** Absence of the Ghost Preview entirely is the signal.

---

## 7. Board Cell Colors

Every board cell state is defined with its complete color treatment. At Absolute MVP, only EMPTY and FILLED are active. All others are registered here for completeness and future activation.

| State | Status | Base fill | Border treatment | Overlay | Token(s) |
|---|---|---|---|---|---|
| **EMPTY** | MVP — active | `CLR-BRD-EMPTY` | `CLR-BRD-EMPTY-BORDER` at 1px | None | `CLR-BRD-EMPTY`, `CLR-BRD-EMPTY-BORDER` |
| **FILLED (normal occupied)** | MVP — active | Piece's assigned `CLR-PC-[identity]-BASE` | Derived from SHADOW tone — not a separate drawn border, but the bottom-right face of the 3D model | Three-face depth model (BASE/LIGHT/SHADOW) | `CLR-PC-[identity]-BASE/LIGHT/SHADOW` |
| **CLEARING (mid-clear animation)** | MVP — active (transient) | `CLR-CMB-CLEAR-FLASH` at escalating opacity (animation-owned) | None during flash | White wash overlay at 80% opacity | `CLR-CMB-CLEAR-FLASH` |
| **HIGHLIGHTED (valid drop zone)** | MVP — active (transient, during drag) | `CLR-BRD-HIGHLIGHT` | None | None | `CLR-BRD-HIGHLIGHT` |
| **FROZEN** | MVP+ — reserved | Desaturated blue-white (specific HEX to be defined when activated, registered as `CLR-CELL-FROZEN` at activation) | A distinct icy crystalline pattern overlay | Ice-crystal SVG overlay pattern | Reserved: `CLR-CELL-FROZEN` |
| **BLOCKED** | MVP+ — reserved | Dark desaturated warm-grey (specific HEX to be defined at activation, registered as `CLR-CELL-BLOCKED`) | A hatched or solid darker border | No placement allowed marker (non-color: an X glyph or diagonal fill) | Reserved: `CLR-CELL-BLOCKED` |
| **HAZARD** | MVP+ — reserved | Warm deep red-brown, desaturated (distinct from piece Ember by being visibly dimmer and brownish, not vibrant orange-red) | Subtle pulsing border at activation (animation-owned) | Warning glyph (non-color cue) | Reserved: `CLR-CELL-HAZARD` |
| **INDESTRUCTIBLE** | MVP+ — reserved | Near-black with very slight warm metallic tint | A distinct etched/engraved border pattern | No-clear marker (non-color cue: a lock or stone-grain texture) | Reserved: `CLR-CELL-INDESTRUCTIBLE` |

**BINDING RULE:** The four MVP+ cell state colors (FROZEN, BLOCKED, HAZARD, INDESTRUCTIBLE) must be registered in a revision of this document before any implementation begins. The reserved token IDs above are placeholders — no HEX value is assigned until the corresponding state is activated in `01_GAMEPLAY_SPECIFICATION.md` and `03_BOARD_ENGINE_AND_RULES.md`.

---

## 8. Combat Colors

### 8.1 Damage Numbers

Damage numbers are the primary combat feedback visual. They use a 4-tier color escalation that maps directly to the combo multiplier (L value per `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 4):

| Combo tier | L value | Color | Token | Rationale |
|---|---|---|---|---|
| Single clear | L = 1 | Warm white | `CLR-CMB-DAMAGE-1` | Clear but calm — a normal event. |
| Double clear | L = 2 | Amber gold | `CLR-CMB-DAMAGE-2` | Starting to heat up — warmth escalates. |
| Triple clear | L = 3 | Orange-ember | `CLR-CMB-DAMAGE-3` | Hot — player is executing well. |
| Full Combo | L = 4 | Red-hot | `CLR-CMB-DAMAGE-4` | Maximum — the highest achievement in the game's combat. |

This escalation is **color + size** combined (size scaling per `09` Section 6.4, color escalation here). Together they ensure the Full Combo is unmistakably the most impactful event visually, without requiring a new mechanic or sound cue to communicate it.

### 8.2 Enemy HP Bar Color

The enemy HP bar uses a **three-phase color** that transitions as HP depletes. Transitions are smooth animated fades (owned by Animation doc).

| HP remaining | Color | Token |
|---|---|---|
| 66-100% | `CLR-SEM-SUCCESS` — green-positive | `CLR-SEM-SUCCESS` |
| 33-65% | `CLR-SEM-WARNING` — amber-warning | `CLR-SEM-WARNING` |
| 0-32% | `CLR-SEM-DANGER` — red-danger | `CLR-SEM-DANGER` |

**Non-color cues paired with each phase:**
- At 66-100%: HP value is white, no icon.
- At 33-65%: HP value transitions to `CLR-TXT-ACCENT` (amber); a subtle pulsing is added to the bar (animation-owned).
- At 0-32%: HP value transitions to `CLR-CMB-DAMAGE-4` (red-hot); the Telegraph indicator also escalates to `CLR-CMB-TELEGRAPH-THREAT`.

### 8.3 Enemy Hit-Reaction

When the enemy takes damage (DamageEvent fires per `05` Section 27), the enemy portrait plays a brief hit-reaction:

- A color wash of `CLR-CMB-ENEMY-HIT` at 50% opacity is overlaid on the portrait, fading out over the reaction duration.
- The portrait offset/shift is an animation property (owned by Animation doc).
- Both the color wash and the portrait shift must occur simultaneously.

### 8.4 Enemy Defeat

On enemy defeat:

- A full-screen flash of `CLR-CMB-ENEMY-DEFEAT` (warm white) at 70% opacity, fading out over approximately 300ms.
- The board surface briefly illuminates (a brightness overlay of `CLR-CMB-ENEMY-DEFEAT` at 30%, fading out with the full-screen flash).

**Rationale for warm white (not gold/yellow):** A warm white flash reads as a flash of light — earned, clean, triumphant. A gold flash risks reading as "coins dropping" (match-3 association) or "critical hit" (RPG association) — neither of which is the intended tone.

### 8.5 Healing, Shields — Not Implemented

Per `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 12's binding exclusion of player HP, there is no healing mechanic and no shield mechanic at Absolute MVP. No color is assigned to these states. They are noted here only to confirm their explicit absence from the color system.

### 8.6 Telegraph Colors

| Telegraph state | Color | Token | Non-color cue |
|---|---|---|---|
| Normal (standard upcoming action) | Amber | `CLR-CMB-TELEGRAPH` | Standard icon for the action type. |
| Severe (board-threatening, high-pressure action) | Red-danger | `CLR-CMB-TELEGRAPH-THREAT` | Icon changes to a distinct "alert" variant (icon design owned by art doc). |

---

## 9. Objective Colors

Objectives use the **semantic interactive color register** (Section 3.5) to communicate progress and completion states. Objective type does not determine color — progress state does.

| Objective state | Color | Token | Non-color cue |
|---|---|---|---|
| **Not started / 0% progress** | Neutral | `CLR-SEM-NEUTRAL` | Progress bar empty; numeric shows 0/target. |
| **In progress (1-99%)** | Success fill from left | `CLR-SEM-SUCCESS` (fill) + `CLR-SEM-NEUTRAL-DIM` (track) | Progress bar filling; numeric shows current/target. |
| **Nearly complete (>=75%)** | Bright success | `CLR-SEM-SUCCESS` — bar fill brighter/more saturated at animation level | Progress numeric is displayed in `CLR-TXT-ACCENT`. |
| **Complete** | Success — full | `CLR-SEM-SUCCESS` full fill | Checkmark icon overlaid on the bar (non-color cue); bar full; text changes to "Complete." |
| **Failed (Move Limit exceeded)** | Danger | `CLR-SEM-DANGER` | A distinct failure icon (not a checkmark); bar color shifts to danger. |
| **Approaching failure (Move Limit, final turns)** | Warning | `CLR-SEM-WARNING` | Pulsing animation on the Move Limit indicator; warning icon. |

**Bonus Objective:** Uses the same color system as Primary Objective, but at a visually subordinate size (per `08` Section 9). When Bonus is complete, a `CLR-TXT-ACCENT` (amber) badge appears on the chip — amber is used here specifically to distinguish Bonus completion from Primary completion (which uses Success/green).

---

## 10. Relic Rarity Colors

**BINDING DECISION:** Block Battles **does not implement a rarity system** at Absolute MVP. `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md` Section 14 explicitly fixes 8 launch relics with no rarity/tier field. `08_UI_UX_MASTER_SPECIFICATION.md` Section 10 confirms "Rarity: Not implemented" and states that all relic cards are presented with uniform visual weight.

**Therefore: this section defines no rarity colors.** All 8 relic cards are presented identically in visual weight, panel treatment, and color register. The only visual differentiation between relic cards is the per-card icon/illustration (art direction owned by a future Relic Art doc).

**Reserved for future use:** If a rarity system is added in a post-MVP content expansion, the following token IDs are pre-reserved and must be populated in a revision of this document before any rarity-adjacent visual is shipped:

- `CLR-RLY-COMMON` — reserved
- `CLR-RLY-UNCOMMON` — reserved
- `CLR-RLY-RARE` — reserved
- `CLR-RLY-EPIC` — reserved

Until these tokens are defined, no rarity color may appear in any asset.

---

## 11. Accessibility

### 11.1 Core Commitment

Block Battles is designed so that a player with any of the three most common forms of color vision deficiency (protanopia, deuteranopia, tritanopia) can play the game without disadvantage. This section documents how the palette achieves that commitment.

### 11.2 Color Vision Deficiency Analysis

**Protanopia (red-deficient) and Deuteranopia (green-deficient) — most common CVD types, affecting ~8% of males:**

The most critical gameplay distinction at risk: distinguishing **Leaf (green piece)** from **Ember (red-orange piece)** and from the **danger register (red)**.

| Pair | Risk | Mitigation |
|---|---|---|
| Ember vs. Leaf pieces | These two may appear very similar to protanopes/deuteranopes | Mitigation 1: Shape is the primary piece identity signal — not color. Mitigation 2: Amber and Slate pieces, which are less affected by red/green CVD, provide clear contrast anchors in the tray. |
| CLR-SEM-DANGER vs. CLR-SEM-SUCCESS | These may be indistinguishable under protanopia/deuteranopia | **Non-color cue is mandatory:** Danger states always pair with a warning icon or X/alert glyph. Success states always pair with a checkmark or fill-direction cue. |
| CLR-CMB-DAMAGE-4 (red) vs. CLR-CMB-DAMAGE-1 (white) | White vs. red — white is clear under all CVD. The damage number escalation from white to red would lose the red/green distinction but retain the brightness distinction. | The size escalation (Section 6.4 of `09`) provides an additional non-color signal: a Full Combo damage number is 2x the size of a single-clear damage number. |

**Tritanopia (blue-yellow deficient) — affecting ~0.01% of population:**

| Pair | Risk | Mitigation |
|---|---|---|
| Tide (blue) vs. Amber (yellow) | These would appear very similar | These two identities do not need to be distinguished for gameplay — color identity is a visual aid, not a gameplay requirement (shape is the identification system). |
| CLR-SEM-WARNING (amber) vs. neutral | Yellow/amber may appear more grey-ish | Warning states pair with a pulsing animation and icon — the animation and icon remain effective under tritanopia. |

### 11.3 Grayscale Test

All critical state pairs in grayscale (luminosity values):

| Color | Approximate luminance (0-255) | Distinguishable from board background (~32)? |
|---|---|---|
| CLR-BRD-EMPTY | ~43 | Board surface (~32) — barely distinct; this is intentional (the board reads as a low-contrast field) |
| CLR-PC-EMB-BASE (Ember) | ~91 | Yes — clearly distinct from empty cell |
| CLR-PC-AMB-BASE (Amber) | ~154 | Yes — the brightest piece color |
| CLR-PC-LEA-BASE (Leaf) | ~122 | Yes |
| CLR-PC-TID-BASE (Tide) | ~108 | Yes |
| CLR-PC-IRS-BASE (Iris) | ~69 | Yes, but lower — visual distinction relies partly on shape |
| CLR-PC-ROS-BASE (Rose) | ~76 | Yes |
| CLR-PC-SLT-BASE (Slate) | ~96 | Yes |
| CLR-SEM-SUCCESS | ~130 | Yes |
| CLR-SEM-DANGER | ~91 | Yes, but same luminance as Ember — non-color cue mandatory in any context where both appear simultaneously |
| CLR-TXT-PRIMARY | ~233 | Yes — maximum contrast against all surface colors |

**Finding:** Iris is the darkest piece color in grayscale. In a grayscale view it still reads as a distinct filled cell (luminance 69 vs. empty cell luminance 43), but the contrast is moderate. Non-color cue (shape of the piece) remains the primary distinction even in grayscale.

### 11.4 Low-Contrast (Dim-Screen) Test

Mobile games are frequently played in bright sunlight, where display brightness may be perceived as lower. The palette uses no pure-white backgrounds (which cause glare) and no near-black-on-dark surfaces (which disappear in bright light). All critical text meets or exceeds WCAG 2.1 AA (4.5:1) against its surface.

### 11.5 The Redundant Communication Mandate

Every gameplay-critical color in this document is paired with at least one of the following non-color signals:

| Non-color signal type | Applied to |
|---|---|
| **Shape / icon glyph** | All objective states (checkmark, alert icon), all semantic interactive states (warning icon, success icon), Telegraph states (distinct icon per state). |
| **Pattern / texture overlay** | Ghost Preview invalid states (cross-hatch for Occupied, clipping for Out-of-Bounds). |
| **Size / scale** | Damage number magnitude (2x scale for Full Combo vs. single clear). |
| **Position** | No gameplay-critical information is conveyed only by color + position; position always combines with one of the above. |
| **Animation / motion** | Warning-state bars pulse; Board Lock sweep moves (even in reduced-motion mode, the state change itself is visible, just without the easing — per `09` Section 16). |
| **Text / numeric** | All HP states show numeric HP values. All objective states show numeric progress (current/target). Move Limit shows remaining count. |

---

## 12. Dark / Light Environment Behavior

### 12.1 Default Environment

Block Battles uses a **single dark-base environment** at Absolute MVP. The entire color registry in Section 3 is defined for this environment. There is no light-mode variant at Absolute MVP.

**Rationale:** A dark-base environment is appropriate for the game's mood (`09` Section 1.2) and is the standard for premium mobile games in this genre. Adding a light-mode variant doubles the color system's maintenance surface, which is not justified at Absolute MVP given the solo-developer constraint (`00_MASTER_GAME_VISION.md` Section 5.8).

### 12.2 Theme Color Shifts

The four MVP themes (Field/Cave/Stone/Frost Ice/Boss Arena per `09` Section 16.2) shift the background group colors and the board surface tint, while leaving all other color groups (blocks, semantic interactive, text, combat/feedback) entirely unchanged.

| Theme | BG-BASE token active | BRD-SURFACE tint | Accent register |
|---|---|---|---|
| Field / Default | `CLR-BG-FIELD` | Warm amber tint (+3% saturation from BRD-SURFACE base) | Warm amber/gold — aligns with Amber piece identity; uses `CLR-PC-AMB-BASE` at 20% opacity for ambient accents |
| Cave / Stone | `CLR-BG-CAVE` | Cool teal tint (+3% saturation from BRD-SURFACE base, cooler hue) | Muted teal/green — aligns with Leaf/Tide piece identities; uses `CLR-PC-TID-BASE` at 15% opacity for ambient accents |
| Frost / Ice | `CLR-BG-ICE` | Pale blue-white tint | Pale blue-white — a desaturated variant of `CLR-PC-TID-BASE` at high lightness |
| Boss Arena | `CLR-BG-BOSS` | Dark violet tint | Boss-specific saturated accent (defined per Boss archetype in the Enemy Content doc; tokens `CLR-BOSS-ACCENT-1` and `CLR-BOSS-ACCENT-2` are reserved here, values to be defined per archetype) |

**BINDING RULE:** A theme ONLY shifts background and board surface colors. Piece block colors, semantic colors, text colors, and combat/feedback colors are **invariant across themes.** A player's Ember piece is Ember in the Field and Ember in the Boss Arena. Consistency of piece color identity is more valuable than thematic color coherence.

### 12.3 Light Environment (MVP+ Decision)

If a light-mode or high-contrast mode is introduced in the future, it requires:
1. A full revision of Section 3 with a parallel "light" color table.
2. Every token in Section 18 gains a `-LIGHT` variant.
3. A decision on whether the light mode inverts the board/surface relationship or uses an entirely different palette register.

This is explicitly **not defined at Absolute MVP** and is an open decision for a future revision.

---

## 13. Gradients

### 13.1 Where Gradients Are Permitted

| Context | Gradient type | Purpose |
|---|---|---|
| Button surfaces | Very subtle top-to-bottom linear gradient (10% brightness variance from base) | Suggests slight material curvature — the only "depth" on a flat button surface. |
| HP bar fill | Subtle left-to-right linear gradient (5% brightness variance, lighter at left edge) | Reinforces the "draining" reading direction. |
| Background (optional) | Very subtle radial gradient (15% brightness shift, center slightly lighter) | Creates ambient depth without visible texture. Maximum one gradient on the background layer. |
| Screen flash effects | Radial gradient from center at full opacity to transparent at edges | Prevents the flash from appearing as a hard-edged color block. Applies to CLR-CMB-ENEMY-DEFEAT. |

### 13.2 Where Gradients Are Prohibited

| Context | Rationale |
|---|---|
| Block/piece faces | The three-face depth model (BASE/LIGHT/SHADOW) is the piece's depth language. Adding a separate gradient on top of the three-face model creates competing depth signals. |
| Text elements | Any gradient on text reduces legibility. No gradient text anywhere in the game. |
| Icons | Solid fills only. Gradients on icons reduce legibility at small sizes. |
| Board cell fills | Board cells use flat fills for readability. A gradient on filled cells would create visual noise at 64-cell density. |
| Surface borders | Borders are 1px lines. A gradient border has no visible effect at 1px and adds rendering complexity. |
| Any feedback flash effect | Feedback flashes use opacity-animated flat fills (the animation itself provides the transition). A gradient on top of an opacity-animated element creates GPU compositing overhead. |

---

## 14. Highlights and Shadows

### 14.1 How Color Interacts with the Material System

The material depth system from `09_VISUAL_DESIGN_SYSTEM.md` Section 9 and Section 18 is implemented through **color value shifts**, not separate overlay layers.

**For piece blocks (the primary material application):**

The three color values per identity (BASE, LIGHT, SHADOW) implement the lighting model:

- `LIGHT` is the BASE color lightened by **+20% lightness (HSL)** and shifted slightly toward yellow-white (~5° hue shift toward 60°). This simulates the top-left light source catching the block's upper face.
- `SHADOW` is the BASE color darkened by **-22% lightness (HSL)** and shifted slightly toward the complement (~5° hue shift away from 60°, toward the cool side). This simulates the bottom-right face turning away from the light.

**For UI surfaces (panels, cards, buttons):**

- Top-edge highlight: The surface color with +15% lightness applied to a 1dp border on the top edge only.
- Bottom-edge shadow: The surface color with -15% lightness applied to the drop shadow (which is rendered as a separate shadow element, not a border, per the elevation system).

**For the dragged piece (Active/4 elevation):**

- The LIGHT face value is additionally brightened by +5% lightness beyond its standard value.
- The drop shadow opacity increases from 20% to 30%.
- Combined, these changes make the dragged piece read as "higher in the air" than a placed piece without requiring a separate art variant.

### 14.2 Highlight Color Temperature

All highlights trend **warm** (toward the yellow-orange side of the color wheel): this is consistent with the defined light source being warm-toned (an overcast daylight with warm ambient, not a cool fluorescent light). Shadow falls cool.

- Highlight hue shift: +5° toward yellow (60°).
- Shadow hue shift: -5° toward cool complement.

This temperature shift is subtle — at maximum it represents a 10° hue difference between the LIGHT and SHADOW faces. The primary depth signal remains the **lightness shift** (40% range from SHADOW to LIGHT), with the hue shift as a secondary refinement.

---

## 15. State Color Transitions

All color state transitions apply to the same element across different interaction states.

### 15.1 Board Cell States

| State | Visual color | Transition from previous |
|---|---|---|
| **Empty** | `CLR-BRD-EMPTY` | Baseline. |
| **Highlighted** (valid drag-over) | `CLR-BRD-HIGHLIGHT` | Instant switch (per `06` Section 7 — no hysteresis, updates every frame). |
| **Ghost Preview — Valid** | Piece BASE at 45% opacity overlay on highlighted cell | Instant switch per frame. |
| **Ghost Preview — Invalid** | See Section 6.2–6.3 | Instant switch per frame. |
| **Filled (placed)** | Piece BASE/LIGHT/SHADOW three-face model | The piece "locks in" — animation (the Commit moment) is a brief scale-settle, not a color fade. The color is immediately the final placed color. |
| **Clearing** (mid-clear animation) | `CLR-CMB-CLEAR-FLASH` — animated brightness wash | Flash owned by Animation doc. Instantaneous onset, rapid fade (within the Tier 3 feedback window). |
| **Empty again** (post-clear) | `CLR-BRD-EMPTY` | Returns to empty after clearing animation completes. |

### 15.2 HUD / UI Element States

| State | Treatment |
|---|---|
| **Normal** | Base surface color at full opacity. |
| **Hovered / pressed** (touch contact on a button) | Surface lifts to the next elevation tier color (e.g., SRF-200 instead of SRF-100) with an immediate, 1-frame color change. No color fade — the change is instant (input must feel responsive per `06` Section 17). |
| **Selected** (Relic card pre-confirm) | SRF-300 surface + Active/4 shadow elevation + 1px border brightness increase. |
| **Disabled** | Surface color at 40% opacity. Label text at `CLR-TXT-DISABLED`. |
| **Dragging** (dragged piece) | See Section 15.1 Ghost Preview states. The piece itself renders at Active/4 elevation colors (LIGHT face +5%, shadow opacity +10%). |
| **Valid placement** | The piece commits with zero color change — it arrives at the exact color it appeared as in the Ghost Preview Valid state's outline, confirming the prediction. |
| **Invalid placement** | The Ghost Preview briefly holds its invalid color treatment (CMB-DAMAGE-equivalent red/warning tones) for a single beat, then the piece returns to tray. The "invalid hold" is a one-frame brightness flash followed by the return animation. |
| **Damaged** (enemy HP bar) | Three-phase fill color transition (Success, Warning, Danger) — animated depletion per Section 8.2. |
| **Frozen** (MVP+ cell state) | Reserved. Color transition is a slow, cold desaturation sweep from the cell's current fill to `CLR-CELL-FROZEN`. |
| **Cleared** | See "Clearing" in Section 15.1. |

---

## 16. Color Consistency Rules

These rules prevent semantic color collisions — the primary long-term maintenance failure mode for color systems.

### 16.1 Red Is Danger. Only Danger.

`CLR-SEM-DANGER` and `CLR-CMB-DAMAGE-4` share the red register (hue ~5°). **No other element in the game uses this hue at high saturation.** Specifically:

- Piece identity **Rose** (`CLR-PC-ROS-BASE`, hue ~348°) uses a distinct rose-crimson that is visually distinct from the danger red (hue ~5°) by being shifted further toward the warm pink-purple direction. At a glance, Rose reads as pink-red, not danger-red. If in implementation testing they are found to be confusable, Rose shifts further toward magenta.
- **No decorative element, background accent, or non-gameplay icon** uses the danger red register.

### 16.2 Amber / Gold Is Warning and Accent. Distinguish Them.

The warning register (`CLR-SEM-WARNING`) and the text accent (`CLR-TXT-ACCENT`) are both in the amber-gold range. They are distinguished by:

- Warning (`CLR-SEM-WARNING`): hue 38°, saturation 66%. Appears in bars and indicators.
- Accent (`CLR-TXT-ACCENT`): hue 43°, saturation 77%. Appears in text and icons.

These are intentionally close — both are part of the "warm energy" register. The distinction is by context (bars/indicators vs. text/icons), not by requiring them to be visually far apart.

**Rule:** Do not use amber-register colors in pure decorative contexts (e.g., a decorative flourish on a panel border). The amber register is functionally loaded and its decorative use dilutes the warning/accent signal.

### 16.3 Green Is Success. Not Nature, Not Life, Not Go.

`CLR-SEM-SUCCESS` is used **only** for objective completion and positive HP states. It does not appear in:
- Environmental theme accents for the Field theme (which uses amber, not green, even though "field" could suggest green).
- Piece color identity for Leaf (which uses a distinct mid-green that is darker and more olive than the success green).
- Any decorative element.

The Leaf piece identity (`CLR-PC-LEA-BASE`, hue 138°) and the success color (`CLR-SEM-SUCCESS`, hue 143°) are very close in hue. They are distinguished by value: the success color is higher-lightness (47%) and higher-saturation (38%) than the Leaf piece (which would be darker and more muted as a board piece). If in implementation testing this creates confusion, Leaf's hue is shifted ~10° toward yellow (toward 128°) to increase separation.

### 16.4 No Color in the Piece Palette Duplicates a Semantic Color

All seven piece color identities must be visually distinct from all semantic interactive colors at the saturation and lightness levels used in gameplay. Potential conflicts:

| Piece identity | Potential semantic conflict | Resolution |
|---|---|---|
| Leaf (green, hue 138°) | CLR-SEM-SUCCESS (green, hue 143°) | Leaf is darker-lower lightness; success is brighter-higher lightness. Distinguished by value. |
| Ember (red-orange, hue 12°) | CLR-SEM-DANGER (red, hue 4°) | Ember has higher orange bias; Danger is a pure red. On the board, Ember reads as an object; Danger reads as an overlay/warning. |
| Amber (gold, hue 43°) | CLR-SEM-WARNING (amber, hue 38°) | Very close. Warning is used for bars/indicators (functional). Amber piece is used for blocks (board surface). Contexts are separated. If tested as confusing, Amber piece shifts toward hue 50° (more yellow). |

### 16.5 One Color Per Semantic Role

No semantic role may be served by two different colors. If a new semantic need arises (e.g., a "bonus" register for Bonus Objective completion), it must either reuse an existing register (the Bonus completion uses `CLR-TXT-ACCENT` amber badge — already defined in Section 9) or add a new, explicitly-named semantic color to Section 3 via a document revision.

---

## 17. Color QA Checklist

Run this checklist at every build review. Binary pass/fail per item.

### 17.1 Color Registry Integrity

- [ ] Every color value used in any asset in the build traces to a token defined in Section 18 of this document.
- [ ] No color token appears in production code as a raw HEX value — only by its token ID.
- [ ] No new HEX value was introduced without a corresponding new token in Section 18.
- [ ] All MVP+ reserved token IDs (`CLR-CELL-FROZEN`, `CLR-CELL-BLOCKED`, `CLR-CELL-HAZARD`, `CLR-CELL-INDESTRUCTIBLE`, rarity tokens) contain no HEX value (they are still reserved, not defined).

### 17.2 Block Color

- [ ] All seven piece color identities are visually distinct from one another at arm-length on a 360dp screen.
- [ ] No two pieces in any tray share the same color identity.
- [ ] The three-face depth model (BASE/LIGHT/SHADOW) is applied consistently to all placed blocks, with identical treatment regardless of piece shape.
- [ ] The dragged piece's LIGHT face is visually brighter than the same piece when placed.

### 17.3 Ghost Preview

- [ ] Valid ghost preview is visually distinct from Invalid-Occupied ghost preview without relying on color alone (non-color: outline weight and style).
- [ ] Invalid-Occupied is visually distinct from Invalid-Out-of-Bounds without relying on color alone (non-color: occupied uses cross-hatch pattern, out-of-bounds uses truncated rendering).
- [ ] The ghost preview fill never reaches full opacity — it always reads as a ghost, not a placed piece.

### 17.4 Semantic Color

- [ ] `CLR-SEM-DANGER` appears in the game only in the defined danger contexts: HP bar critical phase, Move Limit final turns, Board Lock sweep, Telegraph severe state.
- [ ] `CLR-SEM-SUCCESS` appears only in objective completion and HP bar healthy phase.
- [ ] `CLR-SEM-WARNING` appears only in HP bar mid phase, Move Limit warning, and Telegraph normal state.
- [ ] No decorative element uses any semantic color register.

### 17.5 Accessibility

- [ ] Every danger-register element has a paired non-color cue (icon, pattern, or animation).
- [ ] Every success-register element has a paired non-color cue (icon or text label).
- [ ] The damage number escalation (white to red) is also accompanied by size escalation (base size to 2x).
- [ ] A simulated grayscale screenshot of the Battle screen shows the board cells, pieces, HP bar, and Objective as clearly distinguishable from one another by luminance alone.

### 17.6 Consistency

- [ ] The Leaf piece color and CLR-SEM-SUCCESS are visually distinguishable at arm-length on the board.
- [ ] The Ember piece color and CLR-SEM-DANGER are visually distinguishable in context (Ember on board, Danger in HUD).
- [ ] The Rose piece color is not confused for CLR-SEM-DANGER in player testing.
- [ ] No amber-register element appears in a purely decorative context.

### 17.7 Theme Integrity

- [ ] Switching themes changes only the background and board surface colors — no piece, semantic, text, or feedback color changes on theme switch.
- [ ] The active theme's background is clearly distinct from at least two other themes' backgrounds in a side-by-side comparison.
- [ ] The board remains readable against the theme background (board surface is never closer than 10% luminance to the background).

---

## 18. Canonical Color Token Registry

**This is the master token list.** All code, all design tools, all asset pipelines must reference these tokens by ID. Every token defined in Section 3 appears here with its HEX value and semantic group. This section is the single output that implementation systems consume.

### Group 1 — Background

| Token ID | HEX | HSL | Semantic purpose |
|---|---|---|---|
| `CLR-BG-100` | `#1A1814` | 36, 13%, 9% | Background base — deepest layer |
| `CLR-BG-200` | `#221F1B` | 33, 11%, 12% | Background elevated |
| `CLR-BG-FIELD` | `#1E1C16` | 43, 15%, 10% | Field theme background |
| `CLR-BG-CAVE` | `#161A1C` | 199, 12%, 10% | Cave/Stone theme background |
| `CLR-BG-ICE` | `#161C22` | 214, 21%, 11% | Frost/Ice theme background |
| `CLR-BG-BOSS` | `#18141A` | 285, 13%, 9% | Boss Arena theme background |

### Group 2 — Surface

| Token ID | HEX | HSL | Semantic purpose |
|---|---|---|---|
| `CLR-SRF-100` | `#2C2820` | 36, 16%, 15% | Surface base — panels, HUD zones |
| `CLR-SRF-200` | `#342F26` | 38, 15%, 18% | Surface elevated — cards |
| `CLR-SRF-300` | `#3D382D` | 40, 15%, 21% | Surface high-elevated — selected cards |
| `CLR-SRF-BORDER` | `#4A4438` | 38, 14%, 25% | Surface border — panel/card edges |
| `CLR-SRF-SCRIM` | `#00000099` | — | Modal scrim (60% black) |

### Group 3 — Board

| Token ID | HEX | HSL | Semantic purpose |
|---|---|---|---|
| `CLR-BRD-SURFACE` | `#252018` | 38, 21%, 12% | Board surface — the grid field |
| `CLR-BRD-EMPTY` | `#2E2920` | 38, 18%, 15% | Empty cell — unfilled board cell |
| `CLR-BRD-EMPTY-BORDER` | `#3A3428` | 38, 18%, 19% | Empty cell border — grid lines |
| `CLR-BRD-HIGHLIGHT` | `#4A4230` | 40, 21%, 24% | Empty cell highlight — valid drop target |

### Group 4 — Block / Piece

| Token ID | HEX | HSL | Semantic purpose |
|---|---|---|---|
| `CLR-PC-EMB-BASE` | `#C85A2A` | 19, 64%, 47% | Ember — base |
| `CLR-PC-EMB-LIGHT` | `#E8784A` | 18, 77%, 60% | Ember — light face |
| `CLR-PC-EMB-SHADOW` | `#8A3A18` | 18, 71%, 31% | Ember — shadow face |
| `CLR-PC-AMB-BASE` | `#C89A28` | 43, 67%, 47% | Amber — base |
| `CLR-PC-AMB-LIGHT` | `#E8BC48` | 43, 77%, 60% | Amber — light face |
| `CLR-PC-AMB-SHADOW` | `#8A6A18` | 43, 70%, 31% | Amber — shadow face |
| `CLR-PC-LEA-BASE` | `#3A8A48` | 131, 40%, 38% | Leaf — base |
| `CLR-PC-LEA-LIGHT` | `#58AA66` | 131, 32%, 51% | Leaf — light face |
| `CLR-PC-LEA-SHADOW` | `#265A30` | 131, 38%, 25% | Leaf — shadow face |
| `CLR-PC-TID-BASE` | `#2A7A9A` | 198, 57%, 39% | Tide — base |
| `CLR-PC-TID-LIGHT` | `#48A0BE` | 198, 47%, 51% | Tide — light face |
| `CLR-PC-TID-SHADOW` | `#1A4E64` | 198, 58%, 25% | Tide — shadow face |
| `CLR-PC-IRS-BASE` | `#6A3AAA` | 270, 48%, 44% | Iris — base |
| `CLR-PC-IRS-LIGHT` | `#8A5AC8` | 270, 48%, 57% | Iris — light face |
| `CLR-PC-IRS-SHADOW` | `#44246E` | 270, 51%, 29% | Iris — shadow face |
| `CLR-PC-ROS-BASE` | `#AA3A5A` | 343, 48%, 44% | Rose — base |
| `CLR-PC-ROS-LIGHT` | `#C85A7A` | 343, 48%, 57% | Rose — light face |
| `CLR-PC-ROS-SHADOW` | `#6E2238` | 343, 52%, 28% | Rose — shadow face |
| `CLR-PC-SLT-BASE` | `#4A6A7A` | 200, 24%, 39% | Slate — base |
| `CLR-PC-SLT-LIGHT` | `#6A8A9A` | 200, 19%, 51% | Slate — light face |
| `CLR-PC-SLT-SHADOW` | `#2E4452` | 200, 28%, 25% | Slate — shadow face |

### Group 5 — Semantic Interactive

| Token ID | HEX | HSL | Semantic purpose |
|---|---|---|---|
| `CLR-SEM-SUCCESS` | `#4AA86A` | 143, 38%, 47% | Success / Positive state |
| `CLR-SEM-SUCCESS-DIM` | `#2E6642` | 143, 38%, 29% | Success / Dimmed (progress incomplete) |
| `CLR-SEM-WARNING` | `#C88A28` | 38, 66%, 47% | Warning / Pressure state |
| `CLR-SEM-WARNING-DIM` | `#7A5418` | 38, 67%, 28% | Warning / Dimmed |
| `CLR-SEM-DANGER` | `#C83A2A` | 4, 65%, 47% | Danger / Critical state |
| `CLR-SEM-DANGER-DIM` | `#7A2218` | 4, 67%, 28% | Danger / Dimmed |
| `CLR-SEM-NEUTRAL` | `#6A6258` | 33, 9%, 38% | Neutral / Default state |
| `CLR-SEM-NEUTRAL-DIM` | `#3A3428` | 38, 18%, 19% | Neutral / Track fill |

### Group 6 — Text and Icon

| Token ID | HEX | HSL | Semantic purpose |
|---|---|---|---|
| `CLR-TXT-PRIMARY` | `#F0EAE0` | 36, 44%, 91% | Primary text |
| `CLR-TXT-SECONDARY` | `#A89E90` | 36, 14%, 61% | Secondary text |
| `CLR-TXT-DISABLED` | `#6A6258` | 33, 9%, 38% | Disabled text |
| `CLR-TXT-INVERSE` | `#1A1814` | 36, 13%, 9% | Inverse text (for future light surfaces) |
| `CLR-TXT-ACCENT` | `#E8BC48` | 43, 77%, 60% | Accent text |
| `CLR-ICN-PRIMARY` | `#F0EAE0` | 36, 44%, 91% | Primary icon |
| `CLR-ICN-SECONDARY` | `#A89E90` | 36, 14%, 61% | Secondary icon |
| `CLR-ICN-ACCENT` | `#E8BC48` | 43, 77%, 60% | Accent icon |

### Group 7 — Combat and Feedback

| Token ID | HEX | HSL | Semantic purpose |
|---|---|---|---|
| `CLR-CMB-DAMAGE-1` | `#F0EAE0` | 36, 44%, 91% | Damage number — 1 line |
| `CLR-CMB-DAMAGE-2` | `#E8BC48` | 43, 77%, 60% | Damage number — 2 lines |
| `CLR-CMB-DAMAGE-3` | `#E87A28` | 28, 80%, 53% | Damage number — 3 lines |
| `CLR-CMB-DAMAGE-4` | `#E83A28` | 5, 80%, 53% | Damage number — 4 lines (Full Combo) |
| `CLR-CMB-CLEAR-FLASH` | `#F0EAE0` | 36, 44%, 91% | Line clear flash — cells |
| `CLR-CMB-COMBO-GLOW` | `#E87A28` | 28, 80%, 53% | Combo indicator glow (L=3) |
| `CLR-CMB-FULLCOMBO-GLOW` | `#E83A28` | 5, 80%, 53% | Full Combo edge flash (L=4) |
| `CLR-CMB-ENEMY-HIT` | `#E83A28` | 5, 80%, 53% | Enemy hit-reaction flash |
| `CLR-CMB-ENEMY-DEFEAT` | `#F0EAE0` | 36, 44%, 91% | Enemy defeat full-screen flash |
| `CLR-CMB-BOARDLOCK` | `#C83A2A` | 4, 65%, 47% | Board Lock sweep |
| `CLR-CMB-TELEGRAPH` | `#C88A28` | 38, 66%, 47% | Telegraph indicator — normal |
| `CLR-CMB-TELEGRAPH-THREAT` | `#C83A2A` | 4, 65%, 47% | Telegraph indicator — severe |

### Reserved — MVP+ Cell States

| Token ID | HEX | Status |
|---|---|---|
| `CLR-CELL-FROZEN` | TBD | Reserved — no value until FROZEN state activated |
| `CLR-CELL-BLOCKED` | TBD | Reserved — no value until BLOCKED state activated |
| `CLR-CELL-HAZARD` | TBD | Reserved — no value until HAZARD state activated |
| `CLR-CELL-INDESTRUCTIBLE` | TBD | Reserved — no value until INDESTRUCTIBLE state activated |

### Reserved — MVP+ Rarity

| Token ID | HEX | Status |
|---|---|---|
| `CLR-RLY-COMMON` | TBD | Reserved — no rarity system at Absolute MVP |
| `CLR-RLY-UNCOMMON` | TBD | Reserved |
| `CLR-RLY-RARE` | TBD | Reserved |
| `CLR-RLY-EPIC` | TBD | Reserved |

### Reserved — Boss Accent (per archetype)

| Token ID | HEX | Status |
|---|---|---|
| `CLR-BOSS-ACCENT-1` | TBD | Reserved — defined per Boss archetype in Enemy Content doc |
| `CLR-BOSS-ACCENT-2` | TBD | Reserved |

---

**BINDING FINAL RULE:** The total number of active (non-reserved) color tokens at Absolute MVP is **57**. Any build that uses more than 57 distinct color values has introduced an undocumented color. Any build that uses fewer has removed a semantically-required color. Both are failures requiring a revision here before proceeding.

---

**End of `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`.**
