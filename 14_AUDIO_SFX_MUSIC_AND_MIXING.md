# 14_AUDIO_SFX_MUSIC_AND_MIXING.md
## Block Battles — Audio, SFX, Music, and Mixing

**Governing documents:** `00_MASTER_GAME_VISION.md`, `05_SCORING_COMBAT_COMBO_AND_PROGRESSION.md`, `06_INPUT_DRAG_AND_DROP_UX.md`, `11_BLOCK_VISUAL_MATERIAL_AND_ART_SPEC.md`, `12_ANIMATION_MOTION_AND_GAME_FEEL.md`, `13_VFX_PARTICLE_AND_IMPACT_SPEC.md`
**Document status:** Layer B — Audio Authority

**Inheritance confirmation:** This document builds upon the Tactile Snap identity defined in Doc 12. It ensures every visual event has a synchronized auditory counterpart. It operates within the solo-developer scope defined in Doc 00, favoring a restrained, highly polished library over sprawling complexity.

---

## 1. Audio Identity

### 1.1 Personality
The audio identity of Block Battles is **"Premium Tactile Synth."** It combines the physical, satisfying thuds and clacks of a high-end physical board game with clean, harmonic synthesized tones for magical and combat elements. It is modern, uncluttered, and rhythmic.

### 1.2 Texture
- **Physical elements (Blocks, Placements):** Wooden blocks, acrylic tiles, heavy stones, deep percussion. 
- **Magical elements (Clears, Damage, Relics):** FM synthesis, sine wave plucks, glass marimbas, pure musical tones.
- **UI:** Crisp, dry clicks and soft pops.

### 1.3 Emotional Tone
Focused, satisfying, and rewarding. The audio should induce a state of "flow." It must never feel punishing or grating, even in defeat.

### 1.4 Realism vs. Stylization
Highly stylized. A block dropping does not sound literally like a piece of plastic hitting a table; it sounds like the *idealized, hyper-satisfying memory* of a physical object locking into a grid. 

### 1.5 Reference Philosophy
Avoid: 8-bit chiptune (too retro/casual), hyper-realistic Foley (too dry), orchestral epic (wrong genre).
Target: Modern premium puzzle titles (e.g., *Monument Valley*, *Threes*, *Tetris Effect*) merged with punchy, satisfying UI sounds.

---

## 2. Audio Hierarchy

Audio events are prioritized by their importance to gameplay. If the engine maxes out its voice count, lower-priority sounds are culled first.

1. **Critical Gameplay Feedback:** Invalid placement warnings, Board Lock triggers, Move Limit warnings. (Must always be heard).
2. **Combat/Clear Events:** Line clears, Combos (L=1 to L=4), Enemy damage, Defeat/Victory.
3. **Interaction (Board):** Piece pickup, drag grid-ticks, piece placement.
4. **Interaction (UI):** Button clicks, menu transitions.
5. **Music:** Background gameplay loop.
6. **Ambience:** Environmental/theme background noise (e.g., wind, cave drips).

---

## 3. SFX Categories

The complete taxonomy of sound effects required for the game:

1. **UI:** Click, Back, Modal Open, Modal Close, Toggle, Error.
2. **Selection (Pickup):** Piece lift, Drag grid-tick.
3. **Placement:** Drop/Lock, Return to tray.
4. **Invalid:** Rejection buzz, Out-of-bounds warning.
5. **Clear:** L=1 line collapse, Flash anticipation.
6. **Combo:** L=2, L=3, L=4 escalation chords.
7. **Damage:** Impact, Crit impact, Armor deflection, Hit point ticker.
8. **Enemy:** Attack telegraph, Hit reaction, Defeat shatter.
9. **Objective:** Progress tick, Goal complete, Bonus complete.
10. **Relic:** Card deal, Card select, Combat trigger/proc.
11. **Victory:** Success sting, Reward reveal.
12. **Defeat:** Board lock sequence, Game Over sting.
13. **Boss:** Entrance, Phase shift, Boss defeat.

---

## 4. Placement Sound System

The core loop of the game. A player will hear this thousands of times. It cannot be annoying.

### 4.1 Base Sounds
- **Pickup (`sfx_piece_pickup`):** A hollow, resonant "thwomp" or "click" suggesting a piece being lifted off a magnetic surface. Contains mid-high frequencies for clarity.
- **Drag Tick (`sfx_piece_drag_tick`):** A very quiet, dry "tick" played *only* when the Ghost Preview snaps to a new valid grid cell. 
- **Valid Placement (`sfx_piece_place`):** A heavy, satisfying "clack" with a low-end thump. Must contain harmonics in the 200-400Hz range so the "weight" translates on mobile phone speakers (which cannot reproduce sub-bass).

### 4.2 Repetition Control & Variation
To prevent fatigue, the Placement sound uses:
- **Round-Robin Pooling:** A pool of 4 slightly different recordings of the placement impact. The engine cycles through them sequentially.
- **Pitch Randomization:** Every time a piece is placed, the pitch is randomized by ±0.5 semitones. 

---

## 5. Clear Sound System

Triggered on the Line Clear frame (Doc 12, Section 10).

- **Anticipation (The Flash):** A quick, reverse-cymbal or suction sound (`sfx_clear_charge`) that plays for the 60ms prior to the collapse.
- **The Collapse (L=1):** A bright, harmonic chime (e.g., a Major 3rd interval on a synth marimba) mixed with a crisp shattering sound (glass or crystal). 
- **Decay:** The sound must have a short tail. Long reverbs will muddy the mix when multiple actions happen in sequence.

---

## 6. Combo Audio (Simultaneous Lines)

Block Battles defines combos by the L-value (simultaneous lines cleared). The audio escalates musically with the L-value. 

### 6.1 The Chord Progression
Instead of raising the pitch arbitrarily, higher L-values trigger richer musical chords.
- **L=1 (Single):** Root + Major 3rd (e.g., C + E). Light, pleasant.
- **L=2 (Double):** Root + Major 3rd + Perfect 5th (Major triad). Fuller, more resonant.
- **L=3 (Triple):** Major Triad + Major 7th. Accompanied by a deep synth bass layer. 
- **L=4 (Full Combo):** Massive Major 9th chord spanning multiple octaves. Includes a heavy, saturated sub-bass impact and a bright white-noise "crash." 

### 6.2 Repetition Protection
Because players will clear L=1 and L=2 frequently, they share the musical key of the background track (Section 13) to feel like they are contributing to the music, rather than interrupting it.

---

## 7. Damage Audio

Damage audio plays concurrently with the Clear audio. They occupy different frequency bands (Clears are high/mid-high; Damage is mid/low).

- **Normal Hit:** A punchy, energetic "zap" or "thwack".
- **Critical Hit / High Damage:** (Damage > 25% enemy HP). A sharper, crackling impact with a prominent high-frequency transient (like lightning or shattering glass).
- **Armor / Mitigation Hit:** A dull, metallic "thud." The high frequencies are explicitly rolled off (low-pass filter) to communicate that the attack was muffled or absorbed.
- **Damage Number Ticker:** A rapid, quiet mechanical ticking sound (`sfx_ui_tick_fast`) plays as the enemy HP bar drains.

---

## 8. Enemy Audio

Enemies do not have complex voice acting. They use synthesized or highly processed abstract sounds.

- **Telegraph / Action Prep:** A rising, ominous synth swell that plays when the enemy locks in a dangerous telegraph.
- **Hit Reaction:** A short, glitched vocalization or digital "grunt" mixed with the damage impact.
- **Enemy Defeat:** A massive, reverberant "shatter" followed by a satisfying downward electronic sweep, clearing the auditory space for the Victory sting.

---

## 9. Relic Audio

Relics represent passive modifiers, but the player must *hear* them working.

- **Relic Trigger (`sfx_relic_proc`):** When a relic applies its math to a hit, a distinct, ethereal "ping" (like a tuning fork or triangle) plays. 
- **Frequency:** This sound sits in the 2kHz - 5kHz range to easily cut through the heavier bass of block placements and damage.
- **Restraint:** If a relic triggers on *every single placement*, the sound is reduced in volume by -6dB to prevent it from becoming the dominant sound of the game.

---

## 10. Objective Audio

Objectives must clearly signal their state without requiring the player to look away from the board.

- **Progress Tick (`sfx_obj_progress`):** A soft, affirmative "pop" every time the objective counter increases (e.g., 2/5 -> 3/5).
- **Completion (`sfx_obj_complete`):** A triumphant, brassy or synth-horn chord. Significantly louder and longer than a progress tick.
- **Bonus Completion:** Similar to Completion, but pitched up a Perfect 4th to differentiate it.
- **Failure / Warning:** A descending minor-key arpeggio or a digital "klaxon" for when the Move Limit is critically low.

---

## 11. UI Audio

UI sounds must be instantaneous (0ms latency). 

- **Primary Click:** Short, wooden "tap".
- **Secondary Click:** Lighter, higher-pitched "tap".
- **Modal Open/Close:** A soft "whoosh" or "paper slide" accompanying the visual animation.
- **Cancel/Error:** A muted, dull "boop" (not a harsh buzzer).
- **Relic Card Select:** A crisp, heavy paper flick.
- **Relic Confirm:** A solid, locking "kachunk."

---

## 12. Victory / Defeat Audio

### 12.1 Victory Flow
1. **Enemy Defeat shatter** clears the audio space.
2. **Silence** for 400ms (dramatic pause).
3. **Victory Sting:** A 3-4 second uplifting musical phrase that resolves on a major chord.
4. **Music Transition:** The gameplay music fades out, transitioning to the calmer Menu/Relic Selection ambient track.

### 12.2 Defeat Flow (Board Lock)
1. **The Sweep:** A descending, resonant synth tone that plays over 800ms, matching the visual red sweep across the board. 
2. **The Lock:** A heavy, final "clank" like a vault door shutting.
3. **Music Transition:** Gameplay music slows down slightly (pitch/time stretch) and fades to silence.

---

## 13. Music Philosophy

The music must establish a flow state. It should not be a repetitive 30-second loop with an annoying melody. 

- **Genre:** "Focus Groove." Low-fi hip-hop beats, ambient synthwave, or rhythmic IDM. 
- **Frequency Management:** The music is explicitly composed to have a "hole" in the mid-range frequencies. The bass and drums carry the rhythm, and high airy pads carry the chord progression. The mid-range is left empty so the SFX (placements, damage, clears) can be heard clearly.
- **Menu Music:** Ambient, beatless, calming. 
- **Gameplay Music:** Introduces the beat. Forward momentum.
- **Boss Music:** Faster tempo, heavier bass, more aggressive synth textures.

---

## 14. Dynamic Music (Scope Decision)

**Decision:** Minimal Dynamic Music for Absolute MVP. 

Complex interactive music systems (tempo matching, branching paths) require immense solo-developer overhead. 

**MVP Implementation (Stem Muting):**
The gameplay music consists of 3 synchronized stereo stems:
1. Base (Pads/Atmos)
2. Rhythm (Drums/Bass)
3. Tension (Arpeggios/Leads)

- **Normal Play:** Stems 1 + 2 are unmuted.
- **High Pressure (Move limit < 5):** Stem 3 is unmuted, adding tension. 
- **Victory:** Stems 2 and 3 mute; Stem 1 fades out into the victory sting.

---

## 15. Audio Variation (Fatigue Protection)

Any sound played more than 5 times per minute MUST have variation.
- **Block Placements:** 4 variations + pitch shifting (Section 4).
- **UI Clicks:** 3 variations.
- **Damage Impacts:** 3 variations. 

Failure to include these will result in "machine-gunning" (the exact same waveform repeating rapidly), which is highly abrasive to the human ear.

---

## 16. Volume Hierarchy (Mix Buses)

The game audio is routed through the following bus structure:

1. **Master Bus** (0.0 dB limit, with a soft-knee limiter to prevent clipping during L=4 combos).
   ├── **SFX Bus** (-3.0 dB)
   │   ├── Combat/Clear
   │   ├── Board Interaction
   │   └── UI
   ├── **Music Bus** (-12.0 dB)
   └── **Ambience Bus** (-18.0 dB)

*Note: Decibel values are starting reference points for the mix engineer.*

---

## 17. Ducking Rules (Sidechain Compression)

To ensure critical sounds are heard, the engine will use audio ducking:

- **When a Tier 4 (Full Combo) clear plays:** The Music Bus is ducked (reduced) by -6dB for 500ms, recovering smoothly over 1 second.
- **When an Enemy Defeated sound plays:** The Music Bus is ducked by -10dB.
- **When Modal UI opens (Pause Menu, Settings):** The Music Bus is low-pass filtered (muffled) and reduced by -4dB to shift focus to the UI layer.

---

## 18. Master Mix

The final output must be mixed specifically for **mobile device speakers**.
- **High-Pass Filtering:** Any sub-bass below 80Hz in the music should be rolled off to save headroom, as phone speakers cannot play it.
- **Harmonic Saturation:** Bass impact SFX (like the placement thump) must have upper harmonics (200-500Hz) added so the brain perceives the "bass" even when the fundamental frequency is lost on a phone speaker.
- **Mono Compatibility:** The mix must sound good when collapsed to Mono, as many players use single-speaker devices or play with one earbud.

---

## 19. Audio Settings

As defined in `08_UI_UX_MASTER_SPECIFICATION.md` (Settings), the game requires:
- **Master Volume Slider** (0-100%)
- **Music Volume Slider** (0-100%)
- **SFX Volume Slider** (0-100%)
- **Haptics Toggle** (On/Off)

**System integration:** The game must automatically mute if the device physical mute switch is engaged (iOS) or if the OS media volume is 0.

---

## 20. Haptic Integration

Audio and haptics are designed together. Every major SFX has a paired haptic event (from Doc 12, Section 18).

| Audio Event | Haptic Hook |
|---|---|
| `sfx_piece_drag_tick` | `Haptic_MicroTick` |
| `sfx_piece_place` | `Haptic_Impact` |
| `sfx_clear_L1` | `Haptic_Clear` |
| `sfx_clear_L4` | `Haptic_FullCombo` |
| `sfx_invalid_drop` | `Haptic_Error` |

*Implementation:* Haptics trigger simultaneously with the audio event call. If the audio is delayed due to asset loading, the haptic must delay as well to remain synchronized.

---

## 21. Asset Registry

Canonical list of required audio events for MVP.

**UI:**
- `evt_ui_click`
- `evt_ui_back`
- `evt_ui_modal_open`
- `evt_ui_modal_close`
- `evt_ui_error`

**Board Interaction:**
- `evt_piece_pickup`
- `evt_piece_drag_tick`
- `evt_piece_place`
- `evt_piece_return`
- `evt_placement_invalid`

**Combat / Clear:**
- `evt_clear_flash`
- `evt_clear_L1`
- `evt_clear_L2`
- `evt_clear_L3`
- `evt_clear_L4`
- `evt_damage_normal`
- `evt_damage_crit`
- `evt_damage_armor`

**System / Enemy:**
- `evt_enemy_hit`
- `evt_enemy_telegraph`
- `evt_enemy_defeat`
- `evt_obj_progress`
- `evt_obj_complete`
- `evt_relic_proc`
- `evt_board_lock`

**Music:**
- `mus_menu_loop`
- `mus_battle_loop_base`
- `mus_battle_loop_drums`
- `mus_battle_loop_tension`
- `mus_boss_loop`
- `mus_victory_sting`
- `mus_defeat_sting`

---

## 22. Naming Convention

All raw audio files delivered by the sound designer must follow:
`[type]_[category]_[name]_[variant].wav`

- `type`: `sfx`, `mus`, `amb`
- `category`: `ui`, `board`, `cmb` (combat), `enm` (enemy), `sys` (system)
- `name`: descriptive event name
- `variant`: `01`, `02`, `03` (for round-robin pools)

Example: `sfx_board_place_01.wav`

---

## 23. Mobile Performance

- **Format:** All SFX should be exported and compressed as **ADPCM** or **Vorbis** (depending on engine standard) to save disk space and RAM.
- **Sample Rate:** SFX at 44.1kHz. Music can be 32kHz or 44.1kHz.
- **Voice Limit:** Hard cap the engine at **24 concurrent voices**. If 24 sounds are playing, the quietest/lowest-priority sound is instantly culled.
- **Preloading:** The `UI` and `Board Interaction` sound banks must be loaded into memory at launch and kept resident. They are too latency-sensitive to stream from disk.

---

## 24. Audio QA Checklist

- [ ] **Fatigue Test:** Place 50 blocks rapidly. Does the sound become annoying or "machine-gun"? (If yes, adjust pitch randomization and round-robin).
- [ ] **Mix Clarity:** During an L=3 combo with damage numbers and enemy hit sounds all playing at once, can you hear every distinct element without the audio distorting/clipping?
- [ ] **Mobile Speaker Test:** Does the placement "thud" still sound satisfying on a raw phone speaker without headphones?
- [ ] **Ducking:** Does the music noticeably (but smoothly) lower in volume when an Enemy Defeated sound plays?
- [ ] **Latency:** Is the delay between touching a piece and hearing `sfx_piece_pickup` less than 40ms?
- [ ] **Settings:** Do the Master, Music, and SFX sliders scale logarithmically (not linearly) to ensure a natural volume curve?

---

## 25. Final Audio Contract

> **Audio in Block Battles is physical, rhythmic, and restrained.**
>
> We do not rely on aggressive, noisy sound effects. We rely on musical harmony, punchy transients, and distinct frequencies to communicate board state. 
>
> The sound of a block snapping into the grid must be as satisfying on day 100 as it is on day 1. If a sound demands too much attention, it belongs to a Tier 4 combo or a Boss defeat; everything else must serve the flow state.

---

**End of `14_AUDIO_SFX_MUSIC_AND_MIXING.md`.**
