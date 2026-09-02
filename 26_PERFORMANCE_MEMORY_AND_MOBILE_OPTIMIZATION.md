# 26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md
## Block Battles — Performance, Memory, and Mobile Optimization

**Governing documents:** `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`, `14_AUDIO_SFX_MUSIC_AND_MIXING.md`, `18_TECHNICAL_ARCHITECTURE.md`, `23_UI_IMPLEMENTATION_ARCHITECTURE.md`
**Document status:** Layer C — Engineering Authority

**Inheritance confirmation:** This document establishes the boundaries within which all previously designed art, audio, and UI must operate. It protects the solo-developer scope by focusing exclusively on high-ROI optimizations, avoiding theoretical micro-optimizations in favor of solving actual mobile bottlenecks.

---

## 1. Performance Philosophy

**"Smoothness is a Mechanic."**

Block Battles relies on "Tactile Snap" (`12_ANIMATION_MOTION_AND_GAME_FEEL.md`). If the game drops a frame precisely when a block snaps to the grid, the physical illusion shatters. 
- **Rule 1:** Optimize what happens 100 times a minute (Placement, UI updates). Do not obsess over what happens once per run (Menu loading).
- **Rule 2:** Avoid the garbage collector like the plague during gameplay. 
- **Rule 3:** Overdraw is the silent killer of mobile GPUs.

---

## 2. Target Devices

- **Minimum Spec:** 3GB RAM, Snapdragon 660 / Apple A10 (e.g., iPhone 7 / mid-range 2018 Android).
- **Target Spec:** 4GB+ RAM, Snapdragon 845 / Apple A12 or newer.
- **Why:** Puzzle games appeal to a broad, casual demographic. The game must run flawlessly on 5-year-old hardware to succeed in the mobile market.

---

## 3. FPS Target

- **Gameplay Target:** 60 FPS locked.
- **Menu Target:** 30 FPS or 60 FPS (depending on battery save mode).
- **Rationale:** 30 FPS for drag-and-drop gameplay feels sluggish and disconnects the player's finger from the piece. 60 FPS is non-negotiable for the core loop.

---

## 4. Frame-Time Budget

To achieve 60 FPS, a frame must render in under **16.6ms**.

| Subsystem | Budget | Notes |
|---|---|---|
| **CPU (Scripts & Logic)**| < 3.0 ms | Board math is instant. Time is spent in UI updating. |
| **CPU (Physics)** | 0.0 ms | Physics is strictly forbidden (`19` Sec 15). |
| **GPU (Rendering)** | < 10.0 ms| Particle systems and UI overdraw. |
| **Overhead (OS, Audio)** | < 3.6 ms | Unity internal overhead. |

---

## 5. Memory Budget

- **Target Active RAM:** < 250 MB.
- **Maximum App Size (Download):** < 100 MB.
- **Texture Budget:** < 50 MB in VRAM (Easily achievable via Sprite Atlases).
- **Audio Budget:** < 20 MB in RAM (Compressed SFX).

---

## 6. CPU Considerations

- **Cache Misses:** Iterating through the 8x8 Board array must be done in a 1D array (`int[64]`) rather than a `List<int>` or nested objects, maximizing CPU cache coherency.
- **`Update()` Calls:** Minimize MonoBehaviours using `Update()`. A single `BattleOrchestrator` calling `Tick()` on its subsystems is significantly faster than 64 `CellView` scripts running empty `Update()` loops.

---

## 7. GPU Considerations

- **Overdraw Limit:** Mobile GPUs die on fill-rate. Transparent pixels cost just as much as opaque pixels. Ensure all UI elements are trimmed tightly, and disable the `Image` component entirely if it has an Alpha of 0.
- **Draw Calls:** Batch the board and pieces into a single Sprite Atlas. The entire board layer MUST render in <= 3 draw calls.

---

## 8. UI Performance

As mandated in `23_UI_IMPLEMENTATION_ARCHITECTURE.md`:
1. **Canvas Splitting:** Modifying one text element forces Unity to rebuild the entire Canvas. Keep static UI (Backgrounds) and dynamic UI (HP Bars, Damage Text) on separate Canvases.
2. **Raycast Targets:** Uncheck `Raycast Target` on EVERY `Image` and `TextMeshPro` component unless it is a clickable button. 
3. **No Animators:** Do not use Unity `Animator` on UI. Use DOTween to tween `RectTransforms` directly, avoiding heavy Canvas dirtiness.

---

## 9. VFX Performance

- **Max Particles:** Capped at 300 active particles globally (`13` Sec 16.1).
- **Render Mode:** Avoid massive soft-edged additive sprites overlapping. Use crisp, opaque geometric fragments.
- **Culling:** Ensure particle systems have `Culling Mode` set to `Pause and Catch-up` or `Stop` when off-screen/invisible.

---

## 10. Animation Performance

- **Sprite Animation:** For the few enemies that require frame-by-frame animation, use a lightweight script to swap the `SpriteRenderer.sprite` array index instead of the heavy Unity `Animator` if blending/state-machines are not required.

---

## 11. Audio Memory

- **SFX:** Must be set to `Force To Mono` (halves memory) and `Decompress On Load` (prevents CPU spikes on playback).
- **Music:** Must be set to `Streaming` or `Compressed In Memory`. Never decompress a 3-minute BGM track into RAM.

---

## 12. Asset Size

- **ASTC Compression:** All sprites must use ASTC compression (4x4 or 6x6). ASTC offers superior visual quality at tiny file sizes on modern mobile devices.
- **Sprite Atlases:** Group all `spr_blk_` assets into a single `BlockAtlas`. Group all UI icons into a `UIAtlas`. This eliminates texture switching on the GPU.

---

## 13. Garbage Collection (The GC Spike)

GC spikes cause frame hitches. They are forbidden during `BATTLE_IDLE` and `RESOLVING_COMBAT`.

- **No `new` during gameplay:** Do not instantiate arrays or objects dynamically when a piece is placed.
- **No LINQ:** `IEnumerable`, `.Where()`, and `.ToList()` allocate garbage. Use standard `for` loops when iterating over the 8x8 grid.
- **Strings:** Do not build strings in combat (e.g., `"Damage: " + amt`). Use `TextMeshPro`'s `SetText(char[])` or pre-allocate string buffers if numbers change rapidly, OR accept the minor string allocation for damage numbers *only* if they are pooled.
- **Event Payloads:** All `EventBus` payloads (`22` Sec 5) MUST be `structs`, not `classes`, to avoid heap allocation.

---

## 14. Object Pooling

Object Pooling is mandatory. Calling `Instantiate` or `Destroy` during gameplay is an architectural failure.

**What MUST be pooled:**
1. Block Sprites (Pieces)
2. Board Cells
3. Damage Numbers (Floating Text)
4. VFX Particle Systems (`vfx_clear_burst`, `vfx_enemy_hit`)
5. AudioSources (A generic pool of 24 AudioSources for SFX playback).

*Warmup:* All pools must instantiate their required initial count during the loading screen before the Battle begins.

---

## 15. Loading

- **Synchronous Loading:** Banned.
- **Workflow:** Use `SceneManager.LoadSceneAsync`. While loading, display a fade-to-black or loading screen. Wait for pooling warmup to finish before fading in.

---

## 16. Scene Transitions

- All transitions are masked by a `TransitionCanvas` (sorting order 100).
- Loading a new run deletes the old Battle domain, forces `Resources.UnloadUnusedAssets()`, forces a `System.GC.Collect()`, and then builds the new Battle domain. 
- *Note:* Forcing GC during a loading screen is safe and recommended. Forcing GC during gameplay is disastrous.

---

## 17. Battery

- High framerates drain batteries.
- If the game is paused or waiting in the Main Menu, use `Application.targetFrameRate = 30` to preserve battery.
- When `02_Battle` loads, restore to `Application.targetFrameRate = 60`.

---

## 18. Thermal Considerations

- If the OS reports thermal throttling, the game must not crash.
- Implement an emergency fallback: If frame rate drops below 45 FPS for more than 10 seconds, automatically trigger `SettingsManager.EnableReducedMotion(true)` and cap framerate to 30 FPS to let the device cool down.

---

## 19. Low-End Fallbacks

If the device fails a baseline hardware check on boot (e.g., < 3GB RAM):
- **VFX:** Disable additive glows (`vfx_clear_glow`).
- **Post-Processing:** Disable all Unity Post-Processing (Bloom, Vignette) entirely.
- **Resolution:** Limit rendering resolution scale to 0.8x native.

---

## 20. Profiling Workflow

**Never profile in the Unity Editor and assume the results are valid for mobile.**
1. Build to a physical Android device.
2. Connect via WiFi / USB.
3. Use the **Unity Profiler** to check CPU ms/frame and GC Allocations.
4. Use the **Memory Profiler** to detect memory leaks.
5. Use the **Frame Debugger** to verify draw call batching is working.

---

## 21. Performance Test Scenarios

### "The Cascade Test"
- **Setup:** A full 8x8 board. The player drops an I-Piece that clears 4 lines simultaneously (L=4). The player has 3 Relics active that trigger on line clear. 
- **Requirement:** The game must calculate the board math, trigger all 3 relics, calculate enemy damage, and trigger the `vfx_clear_burst_L4` particle systems WITHOUT dropping below 55 FPS.

---

## 22. Release Performance Gate

A build cannot be submitted to the App Store / Google Play unless it passes:
1. **The Leak Test:** Play 10 consecutive battles. Memory usage must not grow by more than 5MB from Battle 1 to Battle 10.
2. **The Allocation Test:** Playing a standard turn (Drag, Drop, Clear 1 Line) must generate exactly **0 bytes** of GC allocation in the Unity Profiler.
3. **The Size Test:** Final compiled `.apk` / `.aab` is under 100 MB.

---

## 23. Final Performance Contract

> **Performance is respect.**
>
> Dropped frames, device overheating, and massive download sizes are forms of disrespect to the player. As a solo developer, we cannot fix performance by rewriting the rendering pipeline—we fix performance by being disciplined. 
> 
> We pool our objects. We squash our textures. We split our canvases. We do not use LINQ. We do not use physics. We build a mathematically pure simulation, and we render it as cheaply and beautifully as possible.

---

**End of `26_PERFORMANCE_MEMORY_AND_MOBILE_OPTIMIZATION.md`.**
