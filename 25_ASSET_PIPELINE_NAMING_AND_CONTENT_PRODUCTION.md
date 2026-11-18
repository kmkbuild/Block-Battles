# 25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md
## Block Battles — Asset Pipeline, Naming, and Production

**Governing documents:** `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`, `14_AUDIO_SFX_MUSIC_AND_MIXING.md`, `19_FLUTTER_FLAME_PROJECT_STRUCTURE_AND_APP_GAME_ARCHITECTURE.md`
**Document status:** Layer C — Production Authority

**Inheritance confirmation:** This document establishes the rigid production pipeline required to execute the visual and auditory identity of Block Battles for both **Mode 1 (Classic Endless)** and **Mode 2 (Goal-Driven Stage Challenge)**. It ensures that a solo developer, alongside AI content-generation tools, can maintain a pristine, highly organized Flutter project without breaking references or suffering from asset bloat.

---

## 1. Asset Pipeline Philosophy

**"Predictability over Creativity in the Project Structure."**
While the art itself is creative, filenames, folder structures, and pubspec declarations must be ruthlessly standardized. An AI coding agent or build script must be able to resolve an asset path simply by knowing the project's naming conventions.

**Core Rule:** Ensure all assets are properly declared in `pubspec.yaml` and rely on canonical naming. Proper `pubspec.yaml` asset declarations are the only registration mechanism. Missing or misnamed assets fail at runtime.

---

## 2. Asset Categories

The pipeline formally recognizes these specific categories:
1. **Sprites (Blocks):** Gameplay block textures (8 palette themes) and special board obstacles (stone, ice).
2. **Sprites (UI):** Menus, buttons, streak badges, star icons, stage cards, dialog panels.
3. **Sprites (Backgrounds):** Board grid backgrounds and subtle ambient gradient textures.
4. **VFX:** Particle textures (geometric sparkles, line collapse shards, starbursts) for Flame.
5. **Audio:** Calming background music loops and crisp tactile SFX for Flame Audio.
6. **Fonts:** Typography assets for HUD and score displays.

---

## 3. Naming Convention (Canonical)

All assets MUST adhere to `snake_case`. No spaces, no dashes, no uppercase letters.

**Format:** `[prefix]_[category]_[name]_[variant].[ext]`

| Asset Type | Prefix | Category | Example |
|---|---|---|---|
| Block Sprite | `spr` | `blk` | `spr_blk_amber_normal.png` |
| Obstacle Sprite | `spr` | `obs` | `spr_obs_ice_tier1.png` |
| UI Sprite | `ui` | `btn`, `pnl`, `icn`, `str` | `ui_btn_play.png`, `ui_icn_star_gold.png` |
| Background | `bg` | `board`, `menu` | `bg_board_grid_dark.png` |
| Particle Tex | `vfx` | `tex` | `vfx_tex_shard_square.png` |
| Audio (SFX) | `sfx` | `ui`, `brd`, `str` | `sfx_brd_line_clear_01.wav`, `sfx_str_streak_x2.wav` |
| Audio (BGM) | `mus` | `loop`, `menu` | `mus_loop_ambient_zen.wav` |
| Font | `fnt` | `display`, `body` | `fnt_display_bold.ttf` |

---

## 4. Folder Conventions

Reaffirming the structure from `19`.
All imported assets live within the standard Flutter `assets/` root directory:
- `assets/images/blocks/`
- `assets/images/obstacles/`
- `assets/images/ui/`
- `assets/images/backgrounds/`
- `assets/audio/sfx/`
- `assets/audio/music/`
- `assets/fonts/`

Every directory must be declared in `pubspec.yaml`:
```yaml
flutter:
  assets:
    - assets/images/blocks/
    - assets/images/obstacles/
    - assets/images/ui/
    - assets/images/backgrounds/
    - assets/audio/sfx/
    - assets/audio/music/
```

---

## 5. Source vs. Export Assets

**Golden Rule:** NEVER put raw editable source files (`.psd`, `.ai`, `.blend`, `.afdesign`) inside the Flutter `assets/` folder.
- Source files bloat the repository and slow down Flutter compilation.
- **Workflow:** Maintain a separate workspace folder for design sources. Export only flattened, optimized `.png`, `.webp`, or `.wav` files into the Flutter `assets/` directory.

---

## 6. Content Definition Files (Dart Catalogs)

All static content data is authored as pure Dart constant files:
- `shape_catalog.dart` (34 canonical shapes across 9 families)
- `stage_catalog.dart` (Mode 2 Stage definitions, goals, move limits, initial obstacles)

```dart
// lib/domain/catalogs/stage_catalog.dart
class StageCatalog {
  static const StageDefinition stage1 = StageDefinition(
    stageNumber: 1,
    stageId: 'stage_001_intro',
    displayName: 'First Steps',
    moveLimit: 20,
    goals: [
      StageGoal(type: GoalType.clearTotalLines, targetValue: 3),
    ],
    star1Threshold: 500,
    star2Threshold: 1000,
    star3Threshold: 1500,
  );
}
```

Compile-time Dart constants are preferred over external JSON files for instant loading, strong typing, and zero file I/O latency.

---

## 7. Flame and Flutter Loading Standards

- **Blocks & VFX:** Loaded via Flame's `Images` cache during `FlameGame.onLoad()`.
- **UI & Menus:** Loaded via Flutter's `Image.asset()` or SVG rendering widgets.
- **Audio:** Pre-cached via `FlameAudio.audioCache.loadAll()` during the app bootstrap sequence.

---

## 8. Resolution and DPI Standards

Mobile screens require crisp rendering across different pixel densities:
- Standard Block Cell: $128 \times 128\text{ px}$ source PNG (rendered scaled to screen density).
- UI Icons: $64 \times 64\text{ px}$ or $128 \times 128\text{ px}$.
- Stage Star Icons: $96 \times 96\text{ px}$.

---

## 9. 9-Slice and Slicing Guidelines

For scalable UI panels, dialog boxes, and buttons, use Flutter's `centerSlice` on `Image` widgets or Flame's `NineTileBox` to achieve distortion-free border scaling.

---

## 10. Audio Standards

- **Loudness:** All SFX must be exported normalized to a peak of `-3.0 dBFS` to prevent digital clipping in simultaneous line clear chimes.
- **Silence Trimming:** Zero milliseconds of silence at the start of audio files to ensure instantaneous haptic and auditory synchronization.
- **Tails:** Leave a natural decay tail (e.g., chime fade out), but trim trailing dead space.
- **Format:** Uncompressed `.wav` for zero-latency SFX; compressed `.ogg` or `.mp3` for ambient music loops.

---

## 11. VFX Asset Standards

- **Shapes:** Leverage Flutter/Flame canvas primitives (`Canvas.drawRect`, `Canvas.drawCircle`, `Canvas.drawRRect`) wherever possible to minimize texture memory.
- **Particle Color:** Base particle sprites should be pure white PNGs; color tinting is applied dynamically in Flame based on the cleared block's color palette.

---

## 12. Asset Quality Checklist

Every new asset must satisfy:
1. **Palette Harmony:** Colors strictly follow the 8-theme color palette from `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`.
2. **Clean Transparency:** Alpha channels cleanly masked with zero halo artifacts or dark fringing.
3. **No Embedded Text in Art:** UI text is rendered via Flutter typography for localization support.
4. **Strict snake_case:** Filename matches canonical naming convention.

---

## 13. Asset Versioning (Git)

- All final assets in `assets/` are committed to source control.
- Filenames and folder locations are authoritative.
- Keep repository size lean (< 100 MB for MVP assets) by optimizing PNG compression.

---

## 14. Unused Asset Detection

- Prior to every milestone release, review the `assets/` directory against `shape_catalog.dart` and `stage_catalog.dart` to prune orphaned files.

---

## 15. Final Asset Contract

> **Discipline in the pipeline creates freedom in the gameplay.**
>
> A structured asset pipeline ensures rapid development, seamless AI code generation, and minimal mobile build sizes. By enforcing canonical naming, proper sound normalization, and clean separation between assets and code definitions, Block Battles guarantees an ultra-responsive, polished mobile experience.

---

**End of `25_ASSET_PIPELINE_NAMING_AND_CONTENT_PRODUCTION.md`.**
