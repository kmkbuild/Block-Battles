# 25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md
## Block Battles — Asset Pipeline, Naming, and Production

**Governing documents:** `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`, `14_AUDIO_SFX_MUSIC_AND_MIXING.md`, `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`
**Document status:** Layer C — Production Authority

**Inheritance confirmation:** This document establishes the rigid production pipeline required to execute the visual and auditory identity of Block Battles. It ensures that a solo developer, alongside AI content-generation tools, can maintain a pristine, highly organized Flutter project without breaking references or suffering from asset bloat.

---

## 1. Asset Pipeline Philosophy

**"Predictability over Creativity in the Project Structure."**
While the art itself is creative, the filenames, folder structures, and pubspec declarations must be ruthlessly standardized. An AI coding agent or script must be able to guess the filename of a sprite simply by knowing the project's naming conventions. 

**Core Rule:** Ensure all assets are properly declared in `pubspec.yaml` and rely on canonical naming. Proper `pubspec.yaml` asset declarations are the only registration mechanism. Missing or misnamed assets will fail at runtime.

---

## 2. Asset Categories

The pipeline formally recognizes these specific categories:
1. **Sprites (Blocks):** Gameplay pieces.
2. **Sprites (Enemies):** Combat portraits.
3. **Sprites (UI):** Icons, panels.
4. **Sprites (Backgrounds):** Static environment art.
5. **VFX:** Particle textures (geometric shapes) for Flame.
6. **Audio:** Music and SFX for Flame Audio.
7. **Fonts:** Typography assets.
8. **Data:** JSON content definitions (if not using Dart files).

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
| Font | `fnt` | `type` | `fnt_body_regular.ttf` |
| Data | `dat` | `cat` | `dat_cat_relics.json` |

---

## 4. Folder Conventions

Reaffirming the structure from `19`.
All imported assets live within the standard Flutter `assets/` root directory.
- `assets/images/blocks/`
- `assets/images/enemies/`
- `assets/images/ui/`
- `assets/audio/sfx/`
- `assets/audio/music/`
- `assets/fonts/`
- `assets/data/` (Optional JSON)

Every directory must be declared in the `pubspec.yaml`:
```yaml
flutter:
  assets:
    - assets/images/blocks/
    - assets/images/enemies/
    - assets/images/ui/
    - assets/audio/sfx/
    - assets/audio/music/
    - assets/data/
```

---

## 5. Source vs. Export Assets

**Golden Rule:** NEVER put source files (`.psd`, `.ai`, `.blend`, `.afdesign`) inside the Flutter `assets/` folder. 
- Source files bloat the repository, drastically slow down Flutter builds, and bloat the final APK.
- **Workflow:** Maintain a separate directory (e.g., `C:/Projects/BlockBuster_Source/`). Export only flattened `.png`, `.jpg`, or `.wav` files into the `assets/` directory.

---

## 6. Content Definition Files (Dart Catalogs)

All content data is authored as pure Dart files (e.g., `shape_catalog.dart`, `enemy_catalog.dart`, `relic_catalog.dart`).

```dart
// lib/domain/catalogs/enemy_catalog.dart
class EnemyCatalog {
  static const EnemyData iceGoblin = EnemyData(
    id: 'enm_ice_goblin',
    name: 'Ice Goblin',
    baseHp: 100,
    assetPath: 'enemies/spr_enm_ice_goblin.png',
  );
}
```
If using JSON for massive data sets, store them in `assets/data/` and load via `rootBundle.loadString('assets/data/dat_cat_relics.json')`. However, compile-time Dart constants are preferred for speed and type safety.

---

## 7. Flame and Flutter Loading Standards

### Flame Sprites
- Loaded via: `Flame.images.load('blocks/spr_blk_ember_normal.png')`
- For Sprite Atlases / Sheets, use Flame's `SpriteSheet` class.

### Flame Audio
- SFX loaded and played via: `FlameAudio.play('sfx/sfx_cmb_damage_crit_01.wav')`
- BGM loaded via: `FlameAudio.bgm.play('music/mus_loop_battle_base.wav')`

### Typography
- Prefer `google_fonts` package if using standard open web fonts.
- If using custom local fonts, declare them in `pubspec.yaml` and reference via `TextStyle(fontFamily: 'fnt_body_regular')`.

---

## 8. Sprite Standards

- **Dimensions:** Power of Two (POT) preferred (e.g., 64x64, 128x128, 256x256). If an asset is 112x112, pad the canvas to 128x128 to ensure mobile GPUs compress it efficiently.
- **Bleed:** Ensure 2 pixels of edge padding to prevent artifacts (white lines) when rendering at different resolutions.
- **Alpha:** Ensure clean transparent backgrounds. Flame handles standard PNG alpha perfectly.

---

## 9. UI Asset Standards

- **White/Grayscale Base:** UI Panels, Buttons, and progress bar fills MUST be exported as pure white or grayscale. 
- **Why:** This allows Flutter's `ColorFilter` (or `color` parameter on Image widgets/Flame Sprites) to tint the sprite dynamically via code/data, saving us from exporting a Red, Blue, and Green button separately. (Massive memory savings).
- **9-Slicing:** Use Flame's `NineTileBox` or Flutter's `centerSlice` on Image widgets for perfectly scalable UI panels.

---

## 10. Audio Standards

- **Loudness:** All SFX must be exported with a normalized peak of `-3.0 dBFS`. (Never 0 dB, to prevent digital clipping in the mix).
- **Silence Trimming:** Ensure absolutely zero milliseconds of silence at the start of an audio file (`14` Sec 24 requires 0ms latency). 
- **Tails:** Leave a natural decay tail, but do not export 3 seconds of silence after a sound effect.
- **Format:** Use standard `.wav` for short SFX and `.mp3` or `.ogg` for background music to save space.

---

## 11. VFX Asset Standards

- **Shapes:** Do not export a custom 32x32 white square. Use Flutter/Flame drawing primitives (`Canvas.drawRect`, `Canvas.drawCircle`) or standard geometric sprites to save texture memory.
- **Color:** Particle textures must be pure white. Coloring is handled exclusively by the Flame `ParticleSystemComponent` or custom renderers.

---

## 12. Generated Asset Review Process (AI Constraints)

Because a solo developer will likely use AI (Midjourney, DALL-E, Stable Diffusion) to generate enemy portraits, relics, or backgrounds, every generated asset must pass a strict filter before entering the project.

**The AI Review Checklist:**
1. **Style Consistency:** Does it match the "Painted Dimensional Flat" or abstract aesthetic?
2. **Lighting Match:** Does the light source come from Top-Left (315°)?
3. **Artifacts:** Zoom to 200%. Are there random floating pixels, jagged edges, or AI signature watermarks?
4. **Resolution:** AI outputs non-standard resolutions (e.g., 1024x1024). Downscale appropriately for mobile (e.g., 256x256 for a Relic Card icon).
5. **Transparency:** AI rarely outputs clean PNG transparency. The asset MUST have its background cleanly masked and removed via Photoshop/Affinity before export.
6. **Text:** NEVER allow AI-generated text inside an image asset. It will be illegible and impossible to localize.

---

## 13. Asset Versioning (Git)

- All final assets in `assets/` must be committed to source control. No `.gitignore` for game assets.
- We have no GUID system (no `.meta` files). Filenames are the ultimate source of truth.
- **LFS:** (Large File Storage) is technically recommended for images/audio, but for a minimalist 2D MVP, a standard Git repo is usually sufficient. Keep total repo size < 2GB.

---

## 14. Texture Compression

The Flutter and Android build system handle texture compression during the APK/AAB build process. Source PNGs should be properly downscaled before inclusion to keep build size low.

---

## 15. Unused Asset Detection

- **Policy:** Bloat kills mobile load times and APK sizes.
- **Workflow:** Before every major Milestone build, verify that all assets in `pubspec.yaml` are actually utilized. Dart linters will not catch unused assets cleanly, so periodic manual reviews of the `assets/` directory against `catalog.dart` files are necessary.

---

## 16. Asset QA Checklist

Run this on Friday afternoons during production:
- [ ] Are all new UI sprites 9-sliced (if applicable) and base-white?
- [ ] Are all new audio files prefixed with `sfx_` or `mus_`?
- [ ] Did any source files (`.psd`, `.wav` project files) accidentally get dragged into the project folder?
- [ ] Are all new assets declared in `pubspec.yaml` if not using directory wildcards?
- [ ] Is snake_case strictly adhered to?

---

## 17. Solo Developer Production Workflow

The pipeline to maximize solo efficiency:
1. **Define:** Add data to `enemy_catalog.dart` (Needs: "Ice Goblin").
2. **Generate:** Use AI to generate 4 variations of an Ice Goblin.
3. **Select & Clean:** Pick the best. Remove the background. Adjust lighting to top-left.
4. **Export:** Export as `spr_enm_ice_goblin.png` to the `assets/images/enemies/` folder.
5. **Assign:** Verify the asset path string in `enemy_catalog.dart` matches exactly.
6. **Done.**

---

## 18. Final Asset Contract

> **Discipline in the pipeline creates freedom in the gameplay.**
>
> A messy project folder slows down the developer, confuses AI coding tools, and bloats the final mobile APK. By adhering strictly to this naming convention, managing our asset resolutions, and aggressively filtering AI-generated art for consistency, we ensure the game remains professional, performant, and incredibly easy to update in Flutter.

---

**End of `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md`.**
