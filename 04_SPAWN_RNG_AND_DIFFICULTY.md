# 04_SPAWN_RNG_AND_DIFFICULTY.md
## Block Battles — Spawn RNG and Difficulty

**Governing documents:** `00_MASTER_GAME_VISION.md`, `01_GAMEPLAY_SPECIFICATION.md`, `02_SHAPE_LIBRARY.md`, `03_BOARD_ENGINE_AND_RULES.md`
**Document status:** Implementation-level system design

**Inheritance confirmation:** This document does not contradict `00_MASTER_GAME_VISION.md` Sections 5, 12, 13, or 23. It implements Tray Generation (`01_GAMEPLAY_SPECIFICATION.md` Section 6, Section 25) and the solvability safeguards referenced by Master Vision Sections 5.5 and 18, using `02_SHAPE_LIBRARY.md`'s registry and metadata contract and `03_BOARD_ENGINE_AND_RULES.md`'s query/event surface as its only inputs. It does not redefine Board rules, Shape geometry, or Turn structure — it only decides *which* Shapes enter the Tray and *when*.

---

## 1. RNG Philosophy

**DESIGN DECISION:** RNG in Block Battles exists to create *decisions*, never *helplessness* (Master Vision Section 5.5, Controlled Randomness; Section 5.6, Fair Difficulty).

Three non-negotiable philosophy rules, binding on every mechanic in this document:

1. **Solvability first.** A Tray that cannot be placed anywhere on the current Board (Board Lock, `03_BOARD_ENGINE_AND_RULES.md` Section 10) must never be generated when a solvable alternative exists within the same weighting pass (Section 5).
2. **No manufactured loss.** This document must not introduce a mechanic whose sole function is to end a Run — every generation rule here either preserves player agency or makes a threat legible in advance (Master Vision Section 18). Difficulty (Section 10–12) rises by *shrinking the margin for error*, never by *removing the possibility of correct play*.
3. **Randomness is diegetic, not adversarial.** The system does not "know" the player is doing well and punish them for it, nor reward them for doing poorly. Any board-awareness (Section 4) or difficulty scaling (Section 12) responds to *state*, not to a hidden performance judgment of the player.

**MASTER DESIGN RULE (inherited, Master Vision Section 23, rule 4):** Randomness must create decisions, not frustration. Every mechanic below is written to satisfy this rule explicitly, not just in spirit.

---

## 2. Base Shape Selection

**Inputs:** the full 34-entry Shape Registry (`02_SHAPE_LIBRARY.md` Section 12), grouped by `family`, `category`, and `tags`.

**DESIGN DECISION:** Base selection is a two-stage draw, not a flat uniform pick across all 34 entries:

1. **Family draw:** one of the 15 Shape Families (Shape Library Section 4) is selected first, using Family Weights (Section 17, Table 17.1).
2. **Orientation draw:** within the selected Family, one registered orientation entry is selected uniformly at random (e.g., Tetromino L has 4 equally-likely orientations once the L Family itself is chosen).

**Rationale:** Weighting at the Family level (not the individual-entry level) keeps the tuning surface small — 15 tunable weights instead of 34 — while guaranteeing that orientation is never itself a strategic signal (a player should not be able to infer "the T piece is rare" from orientation count alone, since Tetromino T's 4 orientations sum to the same Family weight as Tetromino Square's 1 orientation).

This two-stage draw is the base distribution that Sections 3–9 subsequently modify; it is never used unmodified in live play once Board-Aware Generation (Section 4) and Solvability Protection (Section 5) are applied.

---

## 3. Weighted Generation

**DESIGN DECISION:** Family Weights (Section 17, Table 17.1) are the single tunable surface for baseline shape frequency. They are informed by `block_count` and `strategic_classification` (Shape Library Section 3, Section 7) but are hand-authored values, not a derived formula — this keeps balance changes auditable as data edits, not code changes (Master Vision Section 20, Data-Driven Content).

Weighting considerations baked into the Table 17.1 baseline (not re-derived at runtime):

- **Filler-classified** Shapes (`SINGLE`, `LINE ENABLER` per Shape Library Section 7) are weighted higher — they are low-risk, board-friendly, and support Instant Clarity (Master Vision Section 5.1) for new players.
- **Finisher/High-Risk Finisher** Shapes (large, irregular, e.g. Pentomino Plus, Square 3×3) are weighted lower — they carry higher variance and are meant to be occasional high-value/high-risk draws, not a Tray staple.
- **Precision Piece** Shapes (S/Z/T/L/J tetrominoes) sit at a moderate middle weight — these are Master Vision Section 5.2's "meaningful spatial tension" workhorses.

The Weighted Generation stage draws against Table 17.1 **before** any Board-Aware (Section 4) or Solvability (Section 5) adjustment is applied — those are later filtering/reweighting stages, not replacements for this base table.

---

## 4. Board-Aware Generation

**DESIGN DECISION:** Generation may read Board state via `03_BOARD_ENGINE_AND_RULES.md` Section 11 Board Queries — specifically `getFilledCellCount()`, `getRowFillCount(row)`, `getColFillCount(col)`, and `hasAnyLegalPlacement(candidateShape)` — but only as a **filter/reweight** applied on top of the Section 3 base draw, never as a full replacement of randomness.

Board-awareness operates in two ways:

1. **Soft reweighting (always active):** if the Board's average `getRowFillCount`/`getColFillCount` across all 8 rows/columns is high (a "tight" Board), the weight of `SMALL`-tagged Shapes (Shape Library Section 6) is boosted and `LARGE`-tagged Shapes is reduced, *before* the draw — increasing the odds of a board-friendly Shape without guaranteeing one. Exact thresholds and multipliers: Section 17, Table 17.2.
2. **Hard filtering (Solvability Protection only, Section 5):** a candidate Shape is discarded and redrawn *only* if including it would violate the minimum fairness guarantee (Section 5) — this is the sole point where Board-Aware Generation can remove a specific Shape from consideration entirely, and it is bounded and logged (Section 6).

**MASTER DESIGN RULE:** Soft reweighting (mechanism 1) must never reduce a Shape's weight to zero — a "tight" Board makes large Shapes *less likely*, not *impossible*, preserving Controlled Randomness (Section 1). Only mechanism 2 (Solvability Protection) may remove a Shape outright, and only under the narrow condition defined in Section 5.

---

## 5. Solvability Protection

**Minimum fairness guarantee — DESIGN DECISION:** At the moment a full 3-Shape Tray batch is generated (`01_GAMEPLAY_SPECIFICATION.md` Section 6, "all 3 are generated together as a new batch"), **at least one of the 3 Shapes must have at least one legal Placement on the current Board**, verified via `03_BOARD_ENGINE_AND_RULES.md` Section 10's `hasAnyLegalPlacement` logic applied per-candidate-Shape.

**Algorithm — `generateTray(board)`:**

```text
1. Draw 3 candidate Shapes via Section 2–4 (family draw → orientation draw → board-aware reweight).
2. For each of the 3 candidates, check hasAnyLegalPlacement([candidate], board) individually.
3. If at least 1 of 3 candidates has a legal placement: accept the batch as-is. STOP.
4. If none of the 3 candidates has a legal placement:
   a. Discard the single lowest-weighted candidate (per its Section 3–4 effective weight).
   b. Redraw a replacement candidate from the full weighted pool.
   c. Re-check hasAnyLegalPlacement for the new candidate.
   d. Repeat up to a bounded retry cap (Section 17, Table 17.3) before falling back to Step 5.
5. Fallback (retry cap exhausted): deterministically substitute the smallest-`block_count` Shape Family member (SHP_001, Single Block) as the final candidate — a 1-cell Shape is placeable on any Board with at least one Empty cell, which is guaranteed to exist unless the Board is entirely full (an unreachable state, since a full Board always triggers a clear per `03_BOARD_ENGINE_AND_RULES.md` Section 5/8 before the next Tray generation point).
```

**DESIGN DECISION — what this guarantee does *not* promise:** It guarantees the *Tray* is not dead on arrival. It does **not** guarantee the player can survive many further Turns, clear a line that Turn, or avoid Board Lock later in the same Battle — later Board Lock remains a legitimate, earned failure state (`01_GAMEPLAY_SPECIFICATION.md` Section 15) resulting from the player's own prior placements, not from this generation system. This distinction is what keeps Section 1's "no manufactured loss" rule compatible with "losses must be explainable and traceable to a prior decision" (Master Vision Section 23, rule 9) — the system only promises fairness *at the moment of the deal*, matching real player agency for everything after.

---

## 6. Pity / Recovery Mechanics

**DESIGN DECISION:** Block Battles uses exactly one Pity mechanic at Absolute MVP: **Consecutive Large-Shape Suppression.**

- **Trigger:** if the previous 2 consecutive Tray batches (6 Shapes total) each contained at least one `LARGE`-tagged Shape (Shape Library Section 6), the next batch's Section 4 soft-reweight additionally suppresses `LARGE`-tagged weight by a fixed factor (Section 17, Table 17.4) for that single batch only.
- **Reset:** the suppression counter resets to zero the instant a batch is generated with zero `LARGE`-tagged Shapes.
- **Rationale:** prevents a run of "bad luck" (Master Vision Section 5.5's "unavoidable RNG dead-end") from compounding across multiple consecutive Trays, without ever guaranteeing a specific Shape (which would violate Section 8's anti-cheating stance).

**Explicitly not implemented at MVP:**
- No "guaranteed X after N draws" hard pity counter for any specific Shape or Family — this would make outcomes too predictable and reduce the tension Master Vision Section 3 wants ("tension under control," not "tension eliminated").
- No comeback mechanic tied to Enemy HP, player performance, or Battle length — Board-Aware Generation (Section 4) and this Pity rule are the only two feedback loops, and both react to *Board state*, never to *combat outcome*, keeping the puzzle and combat layers cleanly separated (Master Vision Section 20).

Any future Pity mechanic beyond this one is a Master Vision Section 15 expansion decision, not a default addition, per the same principle Gameplay Spec Section 11 applies to Enemy Mechanics.

---

## 7. Shape Repetition

**DESIGN DECISION — within-batch policy:**

- **Identical Shape entries** (same exact `id`) may not appear twice in the same 3-Shape batch. Enforced as a hard constraint in Section 5's `generateTray` draw loop (Step 1 redraws on a duplicate `id`).
- **Same Family, different orientation** (e.g., two different Tetromino L orientations) *may* co-occur in one batch — this is treated as acceptable variety, not repetition, since the two entries present different placement puzzles despite sharing a Family.
- **Same Category** (e.g., two `LINE`-category Shapes from different Families) may co-occur with no restriction; category-level repetition is not constrained, since Category is a broad grouping (Shape Library Section 5) too coarse to be a meaningful "feels repetitive" signal on its own.

**DESIGN DECISION — cross-batch policy:**

- No hard constraint prevents the same Shape `id` from appearing in consecutive batches — this is intentional; suppressing it would require tracking recent-history state that adds complexity (Master Vision Section 5.8, solo-developer feasibility) for a player-facing effect (three Shapes replaced together per Gameplay Spec Section 6) that is already naturally varied by the every-3-consumed replacement cadence.
- **Diverse-Tray soft bias:** the Section 3 base draw favors, but does not require, at least 2 distinct Families per batch — implemented as a small weight penalty applied to a Family already selected once in the current batch's draw sequence (Section 17, Table 17.5), reducing but not eliminating same-Family repeats within a batch.

---

## 8. Objective-Aware Generation

**This section is critical per the Master Prompt; the design position is stated explicitly and applies without exception.**

**DESIGN DECISION — soft influence only, never invisible guarantees:** Generation may be aware of the current Battle's active Objective (`01_GAMEPLAY_SPECIFICATION.md` Section 12) and may apply a *bounded, table-driven, documented* reweight in response — but it must **never** silently guarantee a specific Shape or Shape property to make an Objective trivially satisfiable.

**Explicit worked example (from the Master Prompt):** an Objective such as "Clear a row using a specific Piece type" or a hypothetical Objective requiring a 1×1 (Single) placement does **not** cause the Single Family's weight to spike to guarantee an appearance. Doing so would let the player win by RNG rather than by decision, which directly contradicts Master Vision Section 3 (Core Fantasy: "I'm solving a simple block puzzle," not "the game handed me the answer") and Section 5.3 (Meaningful Choices).

**What Objective-Aware Generation *is* permitted to do (Section 17, Table 17.6):**

- Apply a **small, fixed, documented weight nudge** (never exceeding the cap in Table 17.6) toward Shapes whose `tags`/`category` are structurally relevant to the active Objective's predicate class — e.g., a cumulative "Clear X Total Lines" Objective may mildly favor `LINE`-tagged Shapes over the run of that Battle, the same way Board-Aware Generation (Section 4) favors `SMALL` Shapes on a tight Board. This is presented as the same *kind* of system the player can learn to read, not a hidden favor.
- Never reweight based on an Objective's specific numeric threshold or turn-limit (e.g., "Defeat Enemy Under Move Limit" does not accelerate generation of high-`block_count` Shapes) — that would cross from "soft influence" into "the RNG is playing the Objective for you."
- Never bypass Section 5's Solvability Protection order-of-operations — Objective-Aware reweighting is applied at the same stage as Board-Aware reweighting (Section 4, mechanism 1), strictly before the Solvability Protection hard-filter (Section 4, mechanism 2; Section 5), and is itself capped so it cannot push Solvability Protection into needing its fallback path more often.

**DESIGN DECISION — auditability requirement:** Every Objective-Aware reweight rule must be entered in Table 17.6 with its exact multiplier and the Objective category it applies to. An Objective category with no entry in Table 17.6 receives **no** reweighting at all — the default is neutral, not favorable.

---

## 9. Relic-Aware Generation

**DESIGN DECISION:** Relics **may** influence Tray generation, but only through the same reweight seam used by Sections 4 and 8 — a Relic is never allowed to introduce a new generation code path, per Master Vision Section 20's "reusable effects... shared, extensible effect framework."

Three sanctioned categories of Relic influence on this document's system, matching the Master Prompt's three named axes:

1. **Shape weight modifiers:** a Relic may declare a `family` or `tags` weight multiplier (e.g., "+30% weight to LARGE-tagged Shapes") applied at the same stage as Section 3's base weights, stacking multiplicatively with Section 4/8 reweights before the final draw.
2. **Special pieces:** **not supported at Absolute MVP.** No Relic may inject a Shape `id` outside the 34-entry Shape Library Section 12 registry, and no Relic may force-include a specific Shape into a batch, since that would bypass Section 5's Solvability Protection logic and the Section 8 anti-guarantee principle by the same mechanism. Reserved as an explicit Extensible hook (Section 20) for a future Relics doc revision, not implemented here.
3. **Piece frequency (Tray size):** **not supported at Absolute MVP.** Tray size is fixed at 3 (`01_GAMEPLAY_SPECIFICATION.md` Section 23); no Relic may alter it, since Tray size is owned by the Gameplay Specification, not this document (Section 20).

**Ownership boundary:** this document owns the mechanism (a weight-multiplier seam Relics may declare into); the Relics doc owns which specific Relics exist and their exact multiplier values (`01_GAMEPLAY_SPECIFICATION.md` Section 25 assigns Relic content itself to a future doc). A Relic Modifier interacting with generation must still resolve through Section 5's Solvability Protection unchanged — Relic-driven weight changes are subject to the same hard-filter and fallback path as any other reweight source.

---

## 10. Difficulty Model

**DESIGN DECISION:** Difficulty is modeled as a composite of five independently measurable signals, each queryable at any point in a Battle. No single signal is "the" difficulty value; Section 12's curve tunes all five together.

| Signal | Definition | Data source |
|---|---|---|
| **Board occupancy** | `getFilledCellCount() / 64`, current fraction of the Board filled. | `03_BOARD_ENGINE_AND_RULES.md` Section 11 |
| **Legal-move count** | The count of `(shape, origin)` pairs across the current 3-Shape Tray for which `validatePlacement` returns `VALID` — a direct proxy for how much room the player has to maneuver. | `03_BOARD_ENGINE_AND_RULES.md` Sections 4, 10 (same brute-force search, exposed as a count rather than a boolean) |
| **Piece difficulty** | The `strategic_classification` mix of the current Tray (Shape Library Section 7) — a Tray of 3 `Precision Piece`/`High-Risk Finisher` Shapes is harder than 3 `Filler` Shapes. | Shape Library Section 3/7, read per drawn Shape |
| **Objective pressure** | How close the Battle is to an Objective-driven failure clause (e.g., remaining Turns before a Move Limit, Gameplay Spec Section 12) as a fraction of budget consumed. | Gameplay Spec Section 12–13 tracked counters |
| **Enemy pressure** | The severity/frequency of active Enemy Telegraphs (Gameplay Spec Section 11) — measured as Blocked/Frozen cell count currently in effect, since both surface as Board state. | `03_BOARD_ENGINE_AND_RULES.md` Section 3.2 extension states (once implemented) |

**DESIGN DECISION:** This document only *measures* these five signals; it does not use them to directly alter Shape weights in real time (that would blur into Board-Aware Generation's Section 4 mechanism, which already reacts to Board occupancy specifically). Their primary consumer is Section 12 (curve tuning, authored per-Encounter) and Section 15 (balance telemetry), not a live per-Turn feedback loop.

---

## 11. Encounter Difficulty

**DESIGN DECISION:** Handcrafted Battles and Block Quest Objectives (`01_GAMEPLAY_SPECIFICATION.md` Section 3, Section 8 per Master Vision) may declare **Encounter Parameters** — a small, explicit config block this document's generation system reads at `PREPARING` (Gameplay Spec Section 4) — rather than relying only on the default global weights (Table 17.1).

Supported Encounter Parameters (Section 17, Table 17.7):

- **Family weight override set:** a per-Encounter replacement or multiplier layer on top of Table 17.1, for the duration of that Encounter only (reverts at the next Transition, Gameplay Spec Section 3).
- **Pre-filled Board state:** a starting Board configuration (`03_BOARD_ENGINE_AND_RULES.md` Section 3.2's future `INDESTRUCTIBLE` state, or simply pre-set `FILLED` cells) — this document does not own Board pre-fill mechanics themselves (Board Engine doc does), only the fact that generation must run its Section 5 Solvability check against whatever starting Board the Encounter provides, not assume an empty Board.
- **Objective-Aware reweight override:** an Encounter may specify a stronger (but still capped, per Section 8's auditability rule) reweight than the default Table 17.6 entry for its specific Objective instance.

**MASTER DESIGN RULE (inherited, Gameplay Spec Section 25):** Encounter Parameters must reuse this document's existing weight/filter mechanism; a Block Quest Objective requiring bespoke generation logic outside this framework is a scope escalation requiring a revision to this document, not a per-Encounter code branch (mirrors Master Vision Section 23, rule 5 — "Objectives must reuse existing mechanics... rather than introducing bespoke one-off systems").

---

## 12. Difficulty Curves

**DESIGN DECISION:** Difficulty scales across a Run in three named bands, tuned via the five Section 10 signals and expressed as Table 17.1/17.2 weight adjustments applied at the Transition boundary between Battles (Gameplay Spec Section 3), not moment-to-moment within a Battle.

| Band | Approximate Battle range (placeholder, Section 23-style ASSUMPTION) | Family weight posture | Board-Aware reweight strength (Section 4) | Enemy pressure (owned by Enemy Content doc, referenced here for curve shape only) |
|---|---|---|---|---|
| **Early-run** | Battles 1–3 | Skewed toward `Filler`/`Line Enabler` classes; `High-Risk Finisher` weight near its floor. | Weak — tight-board reweight multiplier at its minimum (Table 17.2). | Low: simple single-mechanic Telegraphs (Gameplay Spec Section 22, Example A). |
| **Mid-run** | Battles 4–8, including first Boss cadence point (Master Vision Section 14) | Baseline Table 17.1 weights, unmodified. | Standard strength. | Moderate: compound Telegraphs, occasional Objective-modified Encounters (Gameplay Spec Section 22, Example B/C). |
| **Late-run** | Battles 9+ | `Precision Piece`/`High-Risk Finisher` weight raised toward its ceiling; Pity suppression (Section 6) threshold loosened slightly (more consecutive Large Shapes tolerated before suppression triggers) so late-run variance is allowed to be higher. | Strong — tight-board reweight multiplier at its maximum. | High: multi-mechanic Bosses. |

**DESIGN DECISION:** All numeric band boundaries and multiplier values above are Absolute MVP placeholders (matching the tone of `01_GAMEPLAY_SPECIFICATION.md` Section 23), pending simulation-driven tuning (Section 14–15). They exist so the curve is implementable and testable, not as final values.

Curve application never overrides Section 5's Solvability Protection or Section 1's "no manufactured loss" rule — a "late-run" difficulty posture makes the Tray statistically harder to use well, never mathematically un-placeable.

---

## 13. RNG Seeds

**DESIGN DECISION:** Every Run uses a single **Run Seed**, generated once at `RUN SETUP` (Gameplay Spec Section 3) from a non-deterministic source (system entropy). All subsequent RNG draws in this document — Family draw, Orientation draw, Board-Aware/Objective-Aware/Relic-Aware reweight tie-breaks, Solvability Protection redraws, Pity mechanic evaluation — derive from a single deterministic pseudo-random stream seeded by the Run Seed plus a monotonically increasing draw index.

- **Reproducibility:** given the same Run Seed and the same sequence of player decisions (which affects which draw index each generation call occurs at, since generation is event-triggered, not time-triggered), the exact same sequence of Trays is reproduced. This satisfies `03_BOARD_ENGINE_AND_RULES.md` Section 13's Determinism guarantee at the layer above the Board Engine, and directly supports the Simulation Framework (Section 14).
- **Autosave compatibility:** the Run Seed and current draw index are part of the autosaved Run state (`01_GAMEPLAY_SPECIFICATION.md` Section 20), so a resumed Run continues the identical RNG stream rather than reseeding.
- **No per-Battle reseeding:** the stream is continuous for the whole Run; a new Battle does not reset it, preserving the single-stream reproducibility property above across Transitions (Gameplay Spec Section 3).
- **Debug/QA override:** a Run Seed may be explicitly supplied (rather than entropy-generated) for reproducing a specific reported Tray sequence — this is the same mechanism as normal play, just with a caller-supplied seed instead of an entropy-generated one; no separate debug code path exists.

---

## 14. Simulation Framework

**DESIGN DECISION:** Because Section 13 guarantees a deterministic stream per seed, thousands of full or partial Runs can be simulated headlessly (no rendering, no player input) by pairing this document's generation logic with a scripted or heuristic "virtual player" that calls `03_BOARD_ENGINE_AND_RULES.md`'s query/commit functions directly.

**Simulation loop (conceptual):**

```text
for each of N Run Seeds (N in the thousands, per Table 17.8):
    initialize Run with seed
    while Run not in RUN_DEFEAT and Battle budget not exhausted:
        generate Tray (Section 5's generateTray)
        virtual player selects a placement policy (see below) for each of 3 Shapes
        commit each placement via Board Engine Section 5/21 Final Turn Resolution
        record Section 15 metrics after each Turn and each Battle
    record final Run-level metrics (Section 15)
```

**Virtual player policy tiers** (so simulation results distinguish "is the RNG fair" from "is the puzzle beatable only by experts"):

1. **Random-legal policy:** picks any uniformly random legal placement each Turn — establishes a fairness *floor*; if even this policy rarely hits a same-Tray Board Lock (Section 5's guarantee should make immediate-Tray Board Lock impossible, but multi-Turn Board Lock is still possible and expected under this weak policy), that is expected and healthy.
2. **Greedy-clear policy:** always prefers a placement that completes a line if one exists, otherwise the placement leaving the most Empty cells in the most-filled row/column — a naive-but-reasonable heuristic representing an average player.
3. **Heuristic-lookahead policy:** a shallow (1–2 Turn) lookahead minimizing resulting Board fragmentation — represents a skilled player, used to confirm Master Vision Section 5.2's acceptance criterion ("skilled players consistently outperform random placement").

Comparing metrics (Section 15) across these three policy tiers is the primary tool for validating Section 1's philosophy: a healthy design shows a large performance gap between Policy 1 and Policy 3 (skill matters) while Policy 1's failure rate stays within the Anti-Frustration bounds of Section 16 (RNG alone isn't fatal).

---

## 15. Balance Metrics

Metrics recorded per simulated Run/Battle (Section 14), each with a target range to be filled in once simulation data exists (Table 17.8 placeholder columns marked TBD-post-simulation, per the same ASSUMPTION convention as Gameplay Spec Section 23):

| Metric | Definition | Recorded per | Used to validate |
|---|---|---|---|
| **Average run length** | Mean number of Battles completed before `RUN_DEFEAT`, across all simulated Runs at a given policy tier. | Run | Overall pacing against Master Vision Section 14 content cadence. |
| **Average battle duration** | Mean Turn count per Battle. | Battle | Whether Battles feel appropriately paced (too short = shallow puzzle; too long = drags, Master Vision Section 5.4). |
| **Failure rate** | Fraction of simulated Runs ending in `RUN_DEFEAT` due to Board Lock (Section 10 of Board Engine doc) vs. Objective-specific failure (Gameplay Spec Section 15). | Run | Confirms failure causes are attributable and not RNG-dominated (Section 1, Section 16). |
| **Legal move count** | The Section 10 "legal-move count" signal, sampled every Turn. | Turn | Detects if Board occupancy trends toward chronically low legal-move counts too early (a sign generation is too harsh) or never drops (too lenient, Master Vision Section 5.2 "not a formality"). |
| **Score** | Cumulative lines cleared (rows+cols) per Run — a proxy metric, not the Master Vision's final scoring formula (owned by Scoring doc). | Run | Cross-checks difficulty banding (Section 12) against actual player-observable output. |
| **Damage** | Cumulative Damage dealt per Battle (via the Damage Pipeline, Gameplay Spec Section 9) — this document reads it as an input only; it does not own or compute it. | Battle | Confirms Enemy pressure (Section 10) scales sensibly against realistic clear output. |
| **Relic effectiveness** | For simulated Runs with a given Relic active (once Relic-Aware Generation, Section 9, is populated by the Relics doc), the delta in Score/Damage/Run-length vs. a no-Relic control. | Run | Validates Master Vision Section 5.3 (no dominant strategy) at the generation-influence layer specifically. |

**DESIGN DECISION:** All target ranges are intentionally left as **TBD-post-simulation** in Table 17.8 rather than invented — asserting made-up target numbers here would violate this document's own Section 1 principle of not fabricating unverified certainty; the placeholder columns are the mechanism, not the numbers.

---

## 16. Anti-Frustration Rules

Concrete, enforceable rules — not general sentiment — every one traceable to Section 1's philosophy:

1. **No same-Tray Board Lock.** Guaranteed structurally by Section 5; this is the hardest anti-frustration rule in the document.
2. **No back-to-back High-Risk Finisher-heavy batches.** Enforced by the Section 7 diverse-Tray soft bias plus Section 6's Pity mechanic — two consecutive batches each containing a `LARGE`-tagged Shape triggers suppression on the third.
3. **No hidden Objective sabotage.** Objective-Aware Generation (Section 8) is capped and table-driven; it can never *disfavor* Shapes structurally needed to progress an Objective (the Table 17.6 mechanism only ever favors relevant Shapes, never penalizes them — a strict floor of "neutral or better," never "worse than baseline").
4. **Board-Aware reweighting never fights the player's own Board state against them.** Section 4's mechanism 1 always pushes *toward* board-friendly Shapes as occupancy rises, never away from them — there is no "punish a nearly-full Board with more large Shapes" rule anywhere in this document.
5. **Difficulty curve changes (Section 12) apply only at Battle boundaries**, never retroactively mid-Battle — a player is never subjected to a harder generation posture partway through a Battle they started under an easier one.
6. **Every reweight source is capped and enumerable** (Table 17.2, 17.4, 17.5, 17.6, 17.7) — no reweight in this system is allowed to be unbounded, since an unbounded stack of multipliers (Board-Aware + Objective-Aware + Relic-Aware simultaneously) is the most likely accidental route to a soft-locked feeling Tray even under Section 5's hard guarantee.

---

## 17. Probability Tables

Centralized tunable values. **All values below are Absolute MVP placeholder DESIGN DECISIONs**, pending Section 14–15 simulation-driven tuning — matching the explicit ASSUMPTION convention used in `01_GAMEPLAY_SPECIFICATION.md` Section 23.

**Table 17.1 — Base Family Weights (Section 2–3)**

| Family | Base Weight | Rationale class |
|---|---|---|
| Single | 6 | Filler |
| Domino | 8 | Line Enabler |
| Tromino Line | 7 | Line Enabler |
| Tromino Corner | 6 | Setup Piece |
| Tetromino Line | 6 | Line Enabler |
| Tetromino Square (O) | 7 | Flexible Filler |
| Tetromino L | 6 | Precision Piece |
| Tetromino J | 6 | Precision Piece |
| Tetromino T | 5 | Precision Piece |
| Tetromino S | 4 | Precision Piece |
| Tetromino Z | 4 | Precision Piece |
| Pentomino Line | 4 | Finisher |
| Pentomino Plus | 2 | High-Risk Finisher |
| Square 3×3 | 2 | High-Risk Finisher |
| Rectangle 2×3 | 4 | Finisher |

**DESIGN DECISION:** Weights are relative, not percentages; normalize by dividing by the sum (77) at draw time.

**Table 17.2 — Board-Aware Reweight (Section 4)**

| Board occupancy band | `SMALL`-tag weight multiplier | `LARGE`-tag weight multiplier |
|---|---|---|
| 0–33% (low) | ×1.0 | ×1.0 |
| 34–66% (medium) | ×1.15 | ×0.9 |
| 67%+ (high/tight) | ×1.3 | ×0.7 |

**Table 17.3 — Solvability Protection Retry Cap (Section 5)**

| Parameter | Value |
|---|---|
| Max redraw attempts per dead batch | 3 |
| Fallback Shape on cap exhaustion | SHP_001 (Single Block) |

**Table 17.4 — Pity Suppression (Section 6)**

| Parameter | Value |
|---|---|
| Consecutive `LARGE`-containing batches to trigger | 2 |
| Suppression multiplier applied to next batch | ×0.5 on `LARGE`-tag weight |
| Suppression duration | 1 batch, then counter resets per trigger condition |

**Table 17.5 — Diverse-Tray Same-Family Penalty (Section 7)**

| Parameter | Value |
|---|---|
| Weight multiplier for a Family already drawn once in the current batch | ×0.6 |

**Table 17.6 — Objective-Aware Reweight (Section 8)**

| Objective category (Gameplay Spec Section 12) | Reweighted tag/category | Multiplier | Cap |
|---|---|---|---|
| Clear X Rows / Clear X Columns / Clear X Total Lines | `LINE` category | ×1.2 | Hard cap ×1.2, never higher |
| Achieve X Combo | none | ×1.0 (no reweight) | — |
| Survive X Turns | none | ×1.0 (no reweight) | — |
| Defeat Enemy Under Move Limit | none | ×1.0 (no reweight) | — |
| Clear Lines Using a Particular Behavior | the specific `category`/`tags` named by the Objective's predicate | ×1.15 | Hard cap ×1.15 |
| Complete Condition Before Enemy Reaches Phase | none | ×1.0 (no reweight) | — |
| *(any category with no row above)* | none | ×1.0 (neutral default, per Section 8) | — |

**Table 17.7 — Encounter Parameter Bounds (Section 11)**

| Parameter | Bound |
|---|---|
| Max Family weight override multiplier (either direction) | ×0.3 to ×3.0 of Table 17.1 baseline |
| Max Objective-Aware reweight override | ×2.0 of the Table 17.6 entry for that Objective's category |

**Table 17.8 — Simulation Scale (Section 14–15)**

| Parameter | Value |
|---|---|
| Minimum simulated Runs per policy tier for a valid balance pass | 5,000 |
| Policy tiers simulated | 3 (Random-legal, Greedy-clear, Heuristic-lookahead) |
| Metric target ranges | TBD-post-simulation (Section 15) |

---

## 18. Test Cases

| Test | Setup | Expected Result |
|---|---|---|
| G1 | Generate a Tray against Fixture F6 (Board Lock pattern, `03_BOARD_ENGINE_AND_RULES.md` Section 14) using the standard weighted draw. | Section 5's `generateTray` algorithm must not return this exact 3-Shape combination unmodified if all 3 candidates independently fail `hasAnyLegalPlacement`; at least one candidate is redrawn/substituted until the guarantee holds. |
| G2 | Generate 1,000 Trays against an empty Board (Fixture F1) and tabulate Family frequency. | Observed frequency approximates Table 17.1's normalized weights within standard sampling variance. |
| G3 | Generate a Tray against a Board at >67% occupancy (Table 17.2 high band). | `SMALL`-tagged Shape frequency in the resulting sample is measurably higher than the G2 baseline; `LARGE`-tagged frequency is measurably lower. |
| G4 | Generate 2 consecutive batches, each forced (via seeded RNG, Section 13) to include a `LARGE`-tagged Shape. | The 3rd batch's `LARGE`-tag weight reflects the Table 17.4 ×0.5 suppression multiplier. |
| G5 | Generate a Tray with an active "Clear X Total Lines" Objective. | `LINE`-category weight reflects the Table 17.6 ×1.2 multiplier; no other category is affected; the multiplier never exceeds the table's hard cap regardless of stacking with Section 4's reweight. |
| G6 | Generate a Tray with an active "Achieve X Combo" Objective, Board otherwise identical to G2's setup. | Family frequency distribution is statistically indistinguishable from G2 — confirms the "no entry = no reweight" default (Section 8). |
| G7 | Attempt to draw the same Shape `id` twice within one batch (force via seeded RNG). | The duplicate draw is rejected and redrawn per Section 7's within-batch policy; the final batch never contains a repeated `id`. |
| G8 | Run Section 5's `generateTray` fallback path deliberately (force all redraw attempts to fail via a constructed pathological Board). | After Table 17.3's 3 redraw attempts, SHP_001 (Single Block) is substituted as the guaranteed-legal candidate. |
| G9 | Reproduce a Run twice from the same Run Seed with an identical scripted virtual-player policy (Section 14). | Both runs produce byte-identical Tray sequences and identical final metrics (Section 13 determinism). |
| G10 | Simulate 5,000 Runs at the Random-legal policy tier (Section 14, Policy 1). | Zero Runs report a same-Tray-arrival Board Lock (Section 5 guarantee holds under adversarially weak play); multi-Turn Board Lock still occurs at some nonzero rate (expected, not a bug). |

---

## 19. Acceptance Criteria

- Every generated 3-Shape Tray batch has at least one Shape with at least one legal Placement on the Board present at generation time, with zero exceptions across simulation (Section 5, Test G1/G10).
- No Objective, Relic, or Encounter Parameter can force a specific Shape `id` into a Tray at Absolute MVP (Section 8, Section 9, Section 11) — reweighting only ever shifts probability within documented, capped bounds (Table 17.6, 17.7).
- No two identical Shape `id` entries ever appear in the same batch (Section 7, Test G7).
- Given an identical Run Seed and identical player decisions, Tray sequences are perfectly reproducible (Section 13, Test G9).
- All reweight sources (Board-Aware, Objective-Aware, Relic-Aware, Pity, Encounter Parameters) are individually capped per Section 17's tables, and no code path allows them to compound past documented bounds (Section 16, rule 6).
- The Simulation Framework (Section 14) can run at minimum the Table 17.8 scale (5,000 Runs × 3 policy tiers) fully headlessly, using only `03_BOARD_ENGINE_AND_RULES.md`'s existing query/commit/event surface — no simulation-only Board Engine fork is required.
- Every numeric value in Section 17 is traceable to a named Design Decision in this document; no unexplained magic number exists in the generation pipeline.

---

## 20. Cross-Document Dependencies

- **Consumes from `01_GAMEPLAY_SPECIFICATION.md`:** Tray size and batch-replacement timing (Section 6), the Turn/Battle/Transition boundaries this document's curve (Section 12) hooks into (Section 3–4), the Objective category list this document reweights against (Section 12), and the Relic Choice timing this document's Section 9 seam integrates with (Section 17). Tray size itself remains owned there — this document may never alter it (Section 9).
- **Consumes from `02_SHAPE_LIBRARY.md`:** the full Shape Registry, `family`/`category`/`tags`/`strategic_classification`/`block_count` fields (Section 3, Section 12) as the entire input domain for Sections 2–3; explicitly named in Shape Library Section 14 as consumed by "the RNG doc." This document must reference Shapes only by `category`/`tags`, never hardcode a specific `id` in a weighting rule (Shape Library Section 8/9 pattern), with the sole named exception of Section 5's SHP_001 fallback, which is intentionally hardcoded as the universal 1-cell guarantee-of-legality case.
- **Consumes from `03_BOARD_ENGINE_AND_RULES.md`:** `validatePlacement`, `hasAnyLegalPlacement`, and the Board Query surface (Section 4, 10, 11) as the sole mechanism for Solvability Protection (Section 5) and the Difficulty Model's legal-move-count signal (Section 10); this document performs zero direct Board mutation and never bypasses the Board Engine's own algorithms.
- **Exposes to Combat/Scoring doc:** none directly — Section 15's Damage metric is read-only telemetry consumption, not a dependency the Combat doc has on this document.
- **Exposes to Relics doc (future):** the Section 9 weight-multiplier seam and its three sanctioned categories, as the only legal integration point for Relic-driven generation effects; the Relics doc owns which Relics exist and their exact multiplier values, this document owns the mechanism they plug into.
- **Exposes to Objectives/Block Quest doc (future):** the Section 8 Table 17.6 reweight contract and the Section 11 Encounter Parameter schema, as the only legal integration points for Objective/Encounter-driven generation influence; any Objective category added there must receive an explicit Table 17.6 entry (even if that entry is "no reweight") to remain compliant with Section 8's auditability requirement.
- **Exposes to Enemy Content doc (future):** the Difficulty Model's Enemy pressure signal (Section 10) as a read surface for tuning Telegraph severity against generation-driven Board state, without this document needing to know Enemy mechanic specifics.
- This document must not be edited to embed content owned by the above; it references them structurally only, per the pattern established in `01_GAMEPLAY_SPECIFICATION.md` Section 25, `02_SHAPE_LIBRARY.md` Section 14, and `03_BOARD_ENGINE_AND_RULES.md` Section 20.

---

**End of `04_SPAWN_RNG_AND_DIFFICULTY.md`.**
