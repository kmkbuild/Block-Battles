# 26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md
## Block Battles — Performance, Memory, and Mobile Optimization

**Governing documents:** `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`, `14_AUDIO_SFX_MUSIC_AND_MIXING.md`, `18_TECHNICAL_ARCHITECTURE.md`, `23_UI_IMPLEMENTATION_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document establishes the boundaries within which all previously designed art, audio, and UI must operate. It protects the solo-developer scope by focusing exclusively on high-ROI optimizations, avoiding theoretical micro-optimizations in favor of solving actual mobile bottlenecks.

---

## 1. Performance Philosophy

**"Smoothness is a Mechanic."**

Block Battles relies on "Tactile Snap" (`12_ANIMATION_MOTION_AND_GAME_FEEL.md`). If the game drops a frame precisely when a block snaps to the grid, the physical illusion shatters. 
- **Rule 1:** Optimize what happens 100 times a minute (Placement, Flame rendering, UI rebuilds). Do not obsess over what happens once per run (Menu loading).
- **Rule 2:** Avoid the Dart garbage collector like the plague during tight gameplay loops (clear resolution). 
- **Rule 3:** Overdraw is the silent killer of mobile GPUs, and unnecessary Flutter widget rebuilds are the silent killer of CPU frame budgets.

---

## 2. Target Devices

- **Minimum Spec:** 3GB RAM, Snapdragon 660 / Apple A10 (e.g., iPhone 7 / mid-range 2018 Android).
- **Target Spec:** 4GB+ RAM, Snapdragon 845 / Apple A12 or newer.
- **Why:** Puzzle games appeal to a broad, casual demographic. The game must run flawlessly on older hardware to succeed in the mobile market.

---

## 3. FPS Target

- **Gameplay Target:** 60 FPS locked on mid-range Android.
- **Menu Target:** 30 FPS or 60 FPS (depending on battery save mode/Flutter's native target).
- **Rationale:** 60 FPS is non-negotiable for the core loop involving dragging pieces and rendering board updates. 

---

## 4. Frame-Time Budget

To achieve 60 FPS, a frame must render in under **16.6ms**.
Flutter separates execution into UI and Raster threads.

| Subsystem | Budget | Notes |
|---|---|---|
| **CPU (Dart Logic/UI)**| < 5.0 ms | Pure Dart Domain math is instant. Time is spent in Flame `update()` and Riverpod UI rebuilds. |
| **GPU (Raster Thread)**| < 10.0 ms| Rendering Flame components, particle systems, and Flutter overlays. Overdraw is the main risk here. |
| **Overhead** | < 1.6 ms | OS and Engine overhead. |

---

## 5. Memory Budget

- **Target Active RAM:** < 250 MB.
- **Maximum App Size (Download):** APK/AAB size < 100 MB.
- **Texture Budget:** < 50 MB in RAM (Leverage Flame SpriteSheets).
- **Audio Budget:** < 20 MB in RAM (Stream large audio, preload small SFX).

---

## 6. CPU and Dart GC Considerations (Zero Allocation Rule)

- **The Zero Allocation Rule:** GC spikes cause frame hitches. They are forbidden during clear resolution. Do not allocate large numbers of new objects, lists, or strings inside tight `update()` loops or during the instantaneous pure Dart board evaluation.
- **Dart AOT:** Release builds use Dart AOT compilation, making them exceptionally fast. However, memory allocation still triggers GC.
- **Iterations:** Iterating through the 8x8 Board array should avoid creating intermediate lists. Use standard `for` loops rather than chained `.where().toList()` when raw speed and zero allocation matter in grid evaluation math.

---

## 7. Flame Render Performance

- **Component Count:** Keep the Flame `Component` tree as flat as possible. Avoid thousands of deep nested components.
- **Sprite Batching:** Use Flame's `SpriteBatch` or `SpriteSheet` to draw multiple identical sprites (like grid tiles) in a single pass rather than instantiating 64 independent complex `SpriteComponent`s if rendering becomes a bottleneck.
- **Update Loop:** Ensure the `update(dt)` overrides in Flame components only do necessary math. Do not poll complex state if it hasn't changed.

---

## 8. Flutter UI Performance (Rebuilds)

As mandated in `23_UI_IMPLEMENTATION_ARCHITECTURE.md`:
1. **const Widgets:** Use `const` on all stateless widgets and constructors. This prevents Flutter from rebuilding them when their parents rebuild.
2. **Selective Watches:** Riverpod `ref.watch()` must be aggressively scoped. Do NOT watch the entire `gameSessionProvider` at the top of the UI tree. Use `ref.watch(gameSessionProvider.select((state) => state.currentScore))` to ensure only the score widget rebuilds when score changes.
3. **Avoid Opacity:** Animating the `Opacity` widget is expensive. Prefer animating colors or using `FadeTransition`.

---

## 9. VFX and Object Pooling

- **Max Particles:** Capped at 300 active particles globally.
- **Flame Component Pooling:** Creating and destroying Flame components frequently (e.g., score popups, clear burst fragments) causes GC pressure.
- **Manual Pool:** Implement a manual or package-based object pool for `DamageNumberComponent` and `ParticleComponent`. When an effect is done, disable it and return it to the pool instead of removing it from the component tree.

---

## 10. Asset and Audio Memory

- **Sprite Reuse:** Load shared sprites once globally or via a centralized `Images` cache, and pass references to components.
- **Audio Streaming vs Preload:**
  - SFX (Short): Preload via `FlameAudio.audioCache.loadAll()` during startup. 
  - BGM (Long): Stream via `FlameAudio.bgm.play()` to prevent large `.mp3` files from ballooning RAM.

---

## 11. Profiling Workflow

**Never profile performance in Debug mode or the IDE and assume the results are valid for mobile.**
1. Build to a physical Android device in **Profile Mode** (`flutter run --profile`).
2. Connect via WiFi / USB.
3. Open **Flutter DevTools**.
4. Use the **Performance Tab** to monitor UI/Raster thread frame times and identify Jank.
5. Use the **Memory Tab** to detect Dart memory leaks and GC spikes.
6. Use the **CPU Profiler** to find bottlenecks in the Flame `update()` loop or Domain logic.

---

## 12. Startup Time

Android-specific startup time matters.
- Defer non-critical initializations. Open the Hive CE `progressionBox` immediately, but defer loading massive audio banks or heavy SpriteSheets until the loading screen or main menu.
- Ensure the splash screen is lightweight.

---

## 13. Battery & Frame Pacing

- High framerates drain batteries. 
- While in deep menus without animations, Flutter is naturally efficient because it only paints when state changes.
- If implementing an explicit power-save mode, capping Flame's internal update/render rate could be considered, though standard Flutter engine pacing generally handles this adequately.

---

## 14. Thermal Considerations & Low-End Fallbacks

If the device reports poor performance (e.g., dropping below 45 FPS consistently):
- **VFX Fallback:** Automatically trigger `ReducedMotion` in preferences, which scales back Flame particle counts by 75% or disables screen shakes.
- **Resolution:** Flutter handles DPI scaling automatically, but avoid extremely high-resolution assets that stress VRAM on older devices.

---

## 15. Performance Test Scenarios

### "The Cascade Test"
- **Setup:** A full 8x8 board. The player drops an I-Piece that clears 4 lines simultaneously (L=4). Consecutive streak multiplier is active at 2.5x. 
- **Requirement:** The Pure Dart Domain must resolve the cascade instantly. Flame must render the 4-line clear burst, and Flutter must update the score/combo UI, WITHOUT the Raster thread dropping below 16ms.

---

## 16. Release Performance Gate

A build cannot be submitted to the App Store / Google Play unless it passes:
1. **The Leak Test:** Play 10 consecutive game sessions. Memory usage in DevTools must plateau and not grow linearly from Session 1 to Session 10.
2. **The Allocation Test:** Playing a standard turn (Drag, Drop, Clear 1 Line) must avoid massive GC pauses in the DevTools timeline.
3. **The Size Test:** Final compiled `.aab` / `.apk` is under 100 MB.

---

## 17. Final Performance Contract

> **Performance is respect.**
>
> Dropped frames, device overheating, and massive download sizes are forms of disrespect to the player. As a solo developer, we cannot fix performance by rewriting the rendering pipeline—we fix performance by being disciplined. 
> 
> We pool our Flame components. We use `const` Flutter widgets. We selectively watch Riverpod state. We do not trigger Dart GC in the placement loop. We build a mathematically pure simulation, and we render it as cheaply and beautifully as possible.

---

**End of `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md`.**
