# 16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md
## Block Battles — Settings, Accessibility, and Presentation

**Governing documents:** `08_UI_UX_MASTER_SPECIFICATION.md`, `09_VISUAL_DESIGN_SYSTEM.md`, `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md`, `12_ANIMATION_MOTION_AND_GAME_FEEL.md`, `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`, `14_AUDIO_SFX_MUSIC_AND_MIXING.md`, `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md`, `00_MASTER_GAME_VISION.md`
**Document status:** Layer B — Accessibility and Settings Authority

**Inheritance confirmation:** This document consolidates all presentation and accessibility features previously mandated in Layer B documents. It strictly adheres to the solo-developer scope defined in Doc 00 by omitting bloated account systems and complex cloud setups. It guarantees the game remains playable, readable, and comfortable for the widest possible audience.

---

## 1. Settings Philosophy

**"Zero-Friction Inclusion."**

1.  **Do not hide accessibility.** A player should not have to dig through three sub-menus to stop the screen from shaking.
2.  **No bloat.** Every setting must solve a specific friction point. If a setting exists just because "other games have it" (e.g., a dedicated slider for UI sounds separate from SFX), it is removed.
3.  **Default to Accessible Design.** The game is built so that colorblindness, low contrast, and screen readability are solved by the *default* art direction. Settings are for overriding physical presentation (volume, motion, input).
4.  **Immediate Application.** Adjusting a setting applies it instantly. There are no "Apply" or "Save Changes" buttons.

---

## 2. Settings Screen Architecture

The Settings screen is a single scrolling modal panel accessible from the Home Screen and the Battle Pause Menu. 

**Categorization Order:**
1.  **Audio** (Most frequently adjusted)
2.  **Gameplay & Input** (Drag offset, Haptics)
3.  **Accessibility** (Motion, Language)
4.  **System** (Privacy, Data)

The architecture deliberately places volume sliders at the very top, as they are the primary reason players open this menu during a run.

---

## 3. Audio Settings

As specified in `14_AUDIO_SFX_MUSIC_AND_MIXING.md` Section 19.

| Setting | Type | Range | Default | Note |
|---|---|---|---|---|
| **Master Volume** | Slider | 0 - 100% | 100% | Scales all audio buses logarithmically. |
| **Music Volume** | Slider | 0 - 100% | 80% | Controls the background music bus. |
| **SFX Volume** | Slider | 0 - 100% | 100% | Controls combat, placement, and UI buses. |

**Exclusions:** UI sounds do not get their own slider. The MVP scope does not justify separating UI clicks from piece placements; both are grouped under SFX.
**Mute Behavior:** Dragging a slider to 0% acts as a hard mute, suspending the audio processing for that bus to save CPU.

---

## 4. Haptics

As specified in `14_AUDIO_SFX_MUSIC_AND_MIXING.md` Section 20.

| Setting | Type | Default | Note |
|---|---|---|---|
| **Vibration / Haptics** | Toggle | ON | Enables/disables all haptic feedback. |

**Exclusions:** No "Haptic Intensity" slider. Implementing consistent intensity scaling across disparate Android and iOS haptic APIs is a massive engineering overhead for a solo developer. It is On or Off. The OS-level haptic settings govern the baseline strength.

---

## 5. Motion

As specified in `12_ANIMATION_MOTION_AND_GAME_FEEL.md` Section 19 and `13_VFX_PARTICLE_AND_IMPACT_SPEC.md` Section 17.

| Setting | Type | Default | Note |
|---|---|---|---|
| **Reduced Motion** | Toggle | OFF | A single master toggle for photosensitivity and motion sickness. |

**When "Reduced Motion" is ON, the game executes the following overrides:**
- **Screen Shake:** Clamped to 0 (disabled entirely).
- **Flashes:** Full-screen combat flashes and additive line-clear glows are disabled; replaced with simple linear opacity fades.
- **Scale Blooms:** Block scale pulses (1.05x, 1.08x) during placement and clears are disabled.
- **Particles:** VFX particle emission counts are reduced by 80%. Boss ambient particles disabled.

*Why a single toggle?* Splitting this into "Disable Shake", "Disable Flash", and "Disable Bloom" bloats the UI. A player sensitive to one is highly likely to be sensitive to the others. 

---

## 6. Visual Accessibility (Baked-In Defaults)

These are **not settings**. They are non-negotiable design standards enforced by `10_COLOR_SYSTEM_AND_BLOCK_ART_DIRECTION.md` and `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`.

- **Color Independence:** Valid ghost preview uses a solid outline; Invalid-Occupied uses a cross-hatch pattern + dashed outline; Invalid-Out-of-Bounds uses edge clipping. No state relies on color alone.
- **Contrast:** All primary text meets WCAG 2.1 AA (4.5:1) against its background.
- **Semantic Indicators:** Danger states (Move Limit, HP) are always paired with an icon (warning triangle/cross) and an animation (pulsing), never just a red color shift.

Because these are baked into the core design, the settings menu does not need a "Colorblind Mode" or "High Contrast Mode" toggle. The base game is already compliant.

---

## 7. Gameplay Accessibility

As established in `06_INPUT_DRAG_AND_DROP_UX.md`.

| Setting | Type | Default | Note |
|---|---|---|---|
| **Drag Offset** | Segmented Control (Low / Med / High) | Medium | Controls how far *above* the player's finger the dragged piece renders. |
| **Touch Assistance** | Toggle | OFF | Expands the hitbox of tray pieces and UI buttons by 20% for players with motor tremors. |

**Rationale:** Drag Offset is critical because finger sizes and device screens vary wildly. A player on a small phone with large thumbs needs a "High" offset to see the board.

---

## 8. Language

As established in `15_SCREEN_CONTENT_AND_PLAYER_FACING_COPY.md`.

| Setting | Type | Default | Note |
|---|---|---|---|
| **Language** | Dropdown | System Default | Allows manual override of the OS language. |

- **Fallback:** If the OS language is not supported by the game's localization files, the game defaults to English (`en_US`).
- **Architecture:** Changing this dropdown immediately swaps the active localization JSON dictionary and refreshes all on-screen text. No app restart required.

---

## 9. Data / Privacy

| Setting | Type | Default | Note |
|---|---|---|---|
| **Share Analytics** | Toggle | Prompt at launch | Allows anonymous crash reporting and telemetry (if implemented). |
| **Delete Save Data** | Button | N/A | Wipes the local save file. Requires a red-colored confirmation dialog. |

**Compliance:** Analytics must be opt-in or strictly anonymized depending on regional GDPR/CCPA requirements. A "Delete Save Data" button gives the player total control over their local footprint.

---

## 10. Account / Cloud (Explicit Exclusion)

**BINDING DECISION:** As per the solo-developer constraint in `00_MASTER_GAME_VISION.md`, there is **NO** custom account system, NO email login, and NO cross-platform backend server at Absolute MVP.

- All save data (Run progress, Relic unlocks, Settings) is stored locally on the device.
- If cloud save is supported, it relies entirely and exclusively on native OS solutions (Apple iCloud Key-Value storage / Google Play Games Cloud Save) with zero custom UI. The game simply syncs silently in the background.

---

## 11. Reset / Restore Defaults

| Setting | Type | Behavior |
|---|---|---|
| **Restore Defaults** | Button | Resets all Audio, Motion, Haptic, and Gameplay settings to their default values. Does *not* delete run progress or relic unlocks. |

---

## 12. Settings Persistence

- **Storage:** Settings are saved to a local preferences file (e.g., `PlayerPrefs` in Unity, `UserDefaults` in iOS, `SharedPreferences` in Android).
- **Timing:** Written to disk exactly when the toggle/slider is modified (on pointer release).
- **Scope:** Settings are global. They persist across all Runs and Battles.

---

## 13. Platform Differences

The settings presentation adapts silently based on the compile target:

| Feature | iOS | Android |
|---|---|---|
| **Physical Mute Switch**| Must override game volume instantly, regardless of the slider positions. | N/A (Android relies on media volume). |
| **Audio Ducking** | Must respect iOS audio session category (duck background audio like Spotify only if the game SFX are active, but allow the user to listen to their own music by turning the in-game Music slider to 0%). | Similar audio focus rules apply. |
| **Navigation** | Settings modal closed via 'X' or tapping outside. | Must also close when the physical/virtual Android Back button is pressed. |
| **Haptics API** | CoreHaptics (crisp, precise). | VibrationEffect (may feel muddier on low-end devices). |

---

## 14. Accessibility QA Matrix

Run this test suite before any release candidate is approved.

| Test Case | Condition | Expected Result |
|---|---|---|
| **Colorblind Readability** | Apply Protanopia/Deuteranopia filter to screen. | Valid/Invalid ghost states are still clearly distinguishable. The enemy HP bar danger state is obvious via pulsing/icons. |
| **Low Contrast Readability**| Lower device brightness to 10%. | Primary text and board cells are still legible against the background. |
| **Reduced Motion: Shake** | Toggle Reduced Motion ON. Trigger a Full Combo. | The screen does not shake. |
| **Reduced Motion: Flash** | Toggle Reduced Motion ON. Trigger Enemy Defeat. | The screen does not flash white; it fades out smoothly. |
| **Volume Scaling** | Move Master Volume to 50%. | Both Music and SFX drop by 50% relative to their individual slider positions. |
| **Haptic Silence** | Toggle Haptics OFF. Place a piece. | The device does not vibrate. |
| **Data Deletion** | Press Delete Save Data -> Confirm. | The game immediately resets to the tutorial state. |

---

## 15. Final Settings Registry

This table is the canonical list of user preferences the game code must track.

| Setting ID | UI Label | UI Control | Default | Allowed Values | Persistence |
|---|---|---|---|---|---|
| `pref.vol.master` | Master Volume | Slider | `1.0` | `0.0` to `1.0` | Global |
| `pref.vol.music` | Music Volume | Slider | `0.8` | `0.0` to `1.0` | Global |
| `pref.vol.sfx` | Sound Effects | Slider | `1.0` | `0.0` to `1.0` | Global |
| `pref.sys.haptics` | Haptics | Toggle | `true` | `true`, `false` | Global |
| `pref.acc.motion` | Reduced Motion | Toggle | `false` | `true`, `false` | Global |
| `pref.acc.offset` | Drag Offset | Segmented | `med` | `low`, `med`, `high` | Global |
| `pref.acc.touch` | Touch Assist | Toggle | `false` | `true`, `false` | Global |
| `pref.sys.lang` | Language | Dropdown | `sys` | `sys`, `en`, `de`... | Global |
| `pref.sys.telemetry`| Share Analytics | Toggle | `false` | `true`, `false` | Global |

---

## 16. Final Contract

> **Settings are tools, not content.** 
>
> We do not offer graphic quality sliders for a 2D game; we optimize it so it runs at 60fps everywhere. We do not offer complex HUD customization; we design the HUD correctly the first time. 
>
> Every setting in this document exists because a player's physical reality (their hearing, their vision, their device size, their motor control) demands flexibility. If a setting is not in this document, it is not required for the Absolute MVP.

---

**End of `16_SETTINGS_ACCESSIBILITY_AND_PRESENTATION_OPTIONS.md`.**
