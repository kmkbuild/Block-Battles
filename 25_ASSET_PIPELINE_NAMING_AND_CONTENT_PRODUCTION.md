# 25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md
## Block Battles — Asset Pipeline, Naming, and Production

**Governing documents:** `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`, `14_AUDIO_SFX_MUSIC_AND_MIXING.md`, `19_UNITY_PROJECT_STRUCTURE_AND_SCENE_ARCHITECTURE.md`
**Document status:** Layer C — Production Authority

**Inheritance confirmation:** This document establishes the rigid production pipeline required to execute the visual and auditory identity of Block Battles. It ensures that a solo developer, alongside AI content-generation tools, can maintain a pristine, highly organized Unity project without breaking references or suffering from asset bloat.

---

## 1. Asset Pipeline Philosophy

**"Predictability over Creativity in the Project Window."**
While the art itself is creative, the filenames, folder structures, and import settings must be ruthlessly standardized. An AI coding agent or script must be able to guess the filename of a sprite simply by knowing the project's naming conventions. 

**Core Rule:** Never delete and re-import an asset if it can be overwritten. Deleting destroys the `.meta` file (GUID), instantly breaking every prefab and script referencing it.

---

## 2. Asset Categories

The pipeline formally recognizes these specific categories:
1. **Sprites (Blocks):** Gameplay pieces.
2. **Sprites (Enemies):** Combat portraits.
3. **Sprites (UI):** Icons, panels, 9-slices.
4. **Sprites (Backgrounds):** Static environment art.
5. **VFX:** Particle textures (geometric shapes).
6. **Audio:** Music and SFX.
7. **Fonts:** Typography assets.
8. **Materials:** Unity materials and shaders.

---

## 3. Naming Convention (Canonical)

All assets MUST adhere to `snake_case`. No spaces, no dashes, no uppercase letters.

**Format:** `[prefix]_[category]_[name]_[variant].[ext]`

| Asset Type | Prefix | Category | Example |
|---|---|---|---|
| Block Sprite | `spr` | `blk` | `spr_blk_ember_normal.png` |
| Enemy Sprite | `spr` | `enm` | `spr_enm_iceslime.png` |
| UI Sprite | `ui` | `btn`, `pnl`, `icn` | `ui_btn_primary.png`, `ui_icn_relic.png` |
| Background | `bg` | `theme` | `bg_theme_crystal_cave.png` |
| Particle Tex | `vfx` | `tex` | `vfx_tex_shard_triangle.png` |
| Audio (SFX) | `sfx` | `ui`, `brd`, `cmb`| `sfx_cmb_damage_crit_01.wav` |
| Audio (BGM) | `mus` | `loop`, `stng` | `mus_loop_battle_base.wav` |
| Material | `mat` | `vfx`, `ui` | `mat_vfx_additive_glow.mat` |
| Font | `fnt` | `type` | `fnt_body_regular.asset` |
| Prefab | `pf` | `sys`, `ui`, `vfx` | `pf_vfx_line_clear.prefab` |

---

## 4. Folder Conventions

Reaffirming the structure from `19` Section 2.
All imported assets live within `Assets/_Project/`. 
- `Assets/_Project/Art/Sprites/Blocks/`
- `Assets/_Project/Art/Sprites/Enemies/`
- `Assets/_Project/Art/Sprites/UI/`
- `Assets/_Project/Audio/SFX/`
- `Assets/_Project/Prefabs/`

---

## 5. Source vs. Export Assets

**Golden Rule:** NEVER put source files (`.psd`, `.ai`, `.blend`, `.afdesign`) inside the Unity `Assets` folder. 
- Source files bloat the repository, drastically slow down Unity import times, and create unnecessary `.meta` files.
- **Workflow:** Maintain a separate directory (e.g., `C:/Projects/BlockBuster_Source/`). Export only flattened `.png` or `.wav` files into the Unity `Assets/_Project/` directory.

---

## 6. Import Settings (Unity Specific)

### Sprites (2D & UI)
- **Texture Type:** `Sprite (2D and UI)`
- **Sprite Mode:** `Single` (or `Multiple` if using packed sprite sheets).
- **Pixels Per Unit (PPU):** Exactly `100`. (Must be mathematically consistent globally so grid calculations map perfectly).
- **Mesh Type:** `Full Rect` (Tighter for standard sprites, crucial for UI 9-slicing).
- **Compression:** `ASTC 6x6` or `ASTC 4x4` for mobile. Ensure "Compress using ETC1" is unchecked if alpha channels are present.

### Audio
- **SFX:** `Force To Mono` = TRUE (saves 50% RAM), Load Type = `Decompress On Load` (if < 1 sec) or `Compressed In Memory`.
- **Music:** `Force To Mono` = FALSE, Load Type = `Streaming`. Format = `Vorbis`.

---

## 7. Sprite Standards

- **Dimensions:** Power of Two (POT) preferred (e.g., 64x64, 128x128, 256x256). If an asset is 112x112, pad the canvas to 128x128 to ensure mobile GPUs compress it efficiently.
- **Bleed:** Ensure 2 pixels of edge padding to prevent artifacts (white lines) when rendering at different resolutions.
- **Alpha:** Pre-multiplied alpha (`11` Sec 16) is required for smooth edge rendering on the blocks.

---

## 8. UI Asset Standards

- **White/Grayscale Base:** UI Panels, Buttons, and progress bar fills MUST be exported as pure white or grayscale. 
- **Why:** This allows Unity's `Image.color` property to tint the sprite dynamically via code/data, saving us from exporting a Red, Blue, and Green button separately. (Massive memory savings).
- **9-Slicing:** All scalable UI panels must be perfectly symmetric and configured with Unity's Sprite Editor 9-slice borders immediately upon import.

---

## 9. Audio Standards

- **Loudness:** All SFX must be exported with a normalized peak of `-3.0 dBFS`. (Never 0 dB, to prevent digital clipping in the mix).
- **Silence Trimming:** Ensure absolutely zero milliseconds of silence at the start of an audio file (`14` Sec 24 requires 0ms latency). 
- **Tails:** Leave a natural decay tail, but do not export 3 seconds of silence after a sound effect.

---

## 10. VFX Asset Standards

- **Shapes:** Do not export a custom 32x32 white square. Use Unity's default particle textures or procedurally generate geometric shapes in the Shader Graph to save texture memory.
- **Color:** Particle textures must be pure white. Coloring is handled exclusively by the Particle System or Shader.

---

## 11. Animation Asset Standards

- **UI Animation:** Do NOT use Unity's `Animator` component for UI panels. It dirties the Canvas every frame, destroying performance. Use `DOTween` (code-driven animation) for all UI scaling, fading, and sliding (`23` Sec 15).
- **Sprite Animation:** If enemies require frame-by-frame animation, import as a Sprite Sheet (Multiple) and ensure pivot points are identical for every frame (Bottom Center).

---

## 12. Generated Asset Review Process (AI Constraints)

Because a solo developer will likely use AI (Midjourney, DALL-E, Stable Diffusion) to generate enemy portraits, relics, or backgrounds, every generated asset must pass a strict filter before entering Unity.

**The AI Review Checklist:**
1. **Style Consistency:** Does it match the "Painted Dimensional Flat" or abstract aesthetic? (AI tends to default to hyper-detailed realism or glossy 3D; this must be flattened in Photoshop).
2. **Lighting Match:** Does the light source come from Top-Left (315°)? (Flip the image horizontally in Photoshop if it is reversed).
3. **Artifacts:** Zoom to 200%. Are there random floating pixels, jagged edges, or AI signature watermarks?
4. **Resolution:** AI outputs non-standard resolutions (e.g., 1024x1024). Downscale appropriately for mobile (e.g., 256x256 for a Relic Card icon).
5. **Transparency:** AI rarely outputs clean PNG transparency. The asset MUST have its background cleanly masked and removed via Photoshop/Affinity before export.
6. **Text:** NEVER allow AI-generated text inside an image asset. It will be illegible and impossible to localize.

---

## 13. Asset Versioning (Git/PlasticSCM)

- **.meta Files:** The `.meta` file generated by Unity is as important as the `.png` itself. It must ALWAYS be committed to source control.
- **LFS:** (Large File Storage) is technically recommended for images/audio, but for a minimalist 2D MVP, a standard Git repo is usually sufficient. Keep total repo size < 2GB.

---

## 14. Asset Replacement Procedure

When updating an existing asset (e.g., a better icon for the Iron Mallet relic):
1. Locate the original file (e.g., `ui_icn_iron_mallet.png`).
2. Export the new asset from the design software with the EXACT same filename.
3. Overwrite the file in the OS file explorer.
4. Switch back to Unity. Unity will re-import the pixels but keep the GUID intact, ensuring no references in ScriptableObjects or Prefabs are broken.

---

## 15. Unused Asset Detection

- **Policy:** Bloat kills mobile load times.
- **Workflow:** Before every major Milestone build, the developer must run a dependency checker (e.g., Unity Asset Cleaner plugin) to identify unreferenced sprites/audio files. 
- **Exception:** Assets loaded dynamically via Data (e.g., Relic Icons referenced in `RelicData` ScriptableObjects) might appear "unused" to naive scanners. Verify before deletion.

---

## 16. Asset QA Checklist

Run this on Friday afternoons during production:
- [ ] Are all new sprites set to 100 PPU?
- [ ] Are all new UI sprites 9-sliced (if applicable) and base-white?
- [ ] Are all new audio files prefixed with `sfx_` or `mus_`?
- [ ] Did any source files (`.psd`, `.wav` project files) accidentally get dragged into Unity?
- [ ] Are there any unassigned fields in the newly created Prefabs/ScriptableObjects?

---

## 17. Solo Developer Production Workflow

The pipeline to maximize solo efficiency:
1. **Define:** Add data to `EnemyData` ScriptableObject (Needs: "Ice Goblin").
2. **Generate:** Use AI to generate 4 variations of an Ice Goblin.
3. **Select & Clean:** Pick the best. Remove the background in Photoshop. Adjust lighting to top-left. Simplify excessive details.
4. **Export:** Export as `spr_enm_ice_goblin.png` to the `_Project/Art/Sprites/Enemies/` folder.
5. **Import & Configure:** In Unity, set PPU to 100, Mesh to Full Rect.
6. **Assign:** Drag `spr_enm_ice_goblin.png` into the `EnemyData` ScriptableObject.
7. **Done.**

---

## 18. Final Asset Contract

> **Discipline in the pipeline creates freedom in the gameplay.**
>
> A messy project folder slows down the developer, confuses AI coding tools, and bloats the final mobile APK. By adhering strictly to this naming convention, managing our PPU scaling, and aggressively filtering AI-generated art for consistency, we ensure the game remains professional, performant, and incredibly easy to update.

---

**End of `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md`.**
