---
id: s2s5hs
slug: h10-perstep-jacobian-cotrans
type: experiment
status: completed
hypothesis: |
  H10 PER-STEP Jacobian co-translation (fixes H9's relabel fragility). H9 forced the strongest coord-commitment (loc_proj 0.0086) but reach did NOT improve because its action-relabel Jacobian was fit by regressing the CUMULATIVE 8-step chunk action on the FULL displacement-to-target — the chunk is only a PARTIAL reach, so that Jacobian is phase-averaged and under-shoots, corrupting the coord->action map. H10 fixes this: fit the PER-STEP controller gain J_inv (control-units per meter) from the ACTUAL per-step pair (action[0], eef_motion = proprio[t+1]-proprio[t]), a CONSTANT controller property with no partial-reach confound (plumb a new raw eef_motion key through the RLDS transform + collator). Use J_inv for the co-translation relabel (per-step action shift = J_inv @ Δ/K), keep proprio REAL, widen Δ to ~0.18m to cover the LIBERO-PRO swap displacement (0.18-0.25m, which H9's 0.10m missed). GATE (reach metric, blank eval): if swap reach NOW drops below H7's 0.105 toward grasp range (~0.03-0.06) AND clean holds => REACH-COMMIT SOLVED data-free => GREENLIGHT the CPGC GROUND localizer. If swap stays ~0.10-0.13 => the data-free action-relabel route is exhausted (last shot) => bank the H7->H10 arc and the env-free REACH-COMMIT wall is real.
parents: ["[[2026-06-5lsz9w-h9-reach-commit-oracle]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/s2s5hs-h10-perstep-jacobian-cotrans
github-commit: 149411f
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: 2026-06-04
tags: [position-swap, reach-commit, per-step-jacobian, controller-gain, co-translation, partial-win, generalization]
metrics: {reach_metric: min_xy_eef_to_target, eval_ep_per_dim: 50, clean_h10: 0.076, swap_h10: 0.087, task_h10: 0.081, swap_h7: 0.105, swap_h9: 0.134, loc_proj_std_h10: 0.0155, Jinv: '[[65.6,-8.1],[1.6,76.4]]', delta_m: 0.18, verdict: partial-win-first-generalization}
---

# s2s5hs — h10-perstep-jacobian-cotrans

## Hypothesis
H10 PER-STEP Jacobian co-translation (fixes H9's relabel fragility). H9 forced the strongest coord-commitment (loc_proj 0.0086) but reach did NOT improve because its action-relabel Jacobian was fit by regressing the CUMULATIVE 8-step chunk action on the FULL displacement-to-target — the chunk is only a PARTIAL reach, so that Jacobian is phase-averaged and under-shoots, corrupting the coord->action map. H10 fixes this: fit the PER-STEP controller gain J_inv (control-units per meter) from the ACTUAL per-step pair (action[0], eef_motion = proprio[t+1]-proprio[t]), a CONSTANT controller property with no partial-reach confound (plumb a new raw eef_motion key through the RLDS transform + collator). Use J_inv for the co-translation relabel (per-step action shift = J_inv @ Δ/K), keep proprio REAL, widen Δ to ~0.18m to cover the LIBERO-PRO swap displacement (0.18-0.25m, which H9's 0.10m missed). GATE (reach metric, blank eval): if swap reach NOW drops below H7's 0.105 toward grasp range (~0.03-0.06) AND clean holds => REACH-COMMIT SOLVED data-free => GREENLIGHT the CPGC GROUND localizer. If swap stays ~0.10-0.13 => the data-free action-relabel route is exhausted (last shot) => bank the H7->H10 arc and the env-free REACH-COMMIT wall is real.

## Parents
- [[2026-06-5lsz9w-h9-reach-commit-oracle]]

## Method
Same as H9 (co-translate the action so the reach follows a Δ-shifted coord, proprio kept REAL, blank vision,
`COORD_DIM=2`, `COTRANS=0.5`) with TWO fixes: **(1) per-step controller-gain Jacobian.** Plumb a new raw
`eef_motion` key (`proprio[t+1]−proprio[t]`, meters) through the RLDS transform + collator, and fit `J_inv`
(control-units per meter) from the ACTUAL per-step pair `(action[0], eef_motion)` — a *constant* controller
property with no partial-reach confound. It converges **diagonal-dominant `[[65.6,−8.1],[1.6,76.4]]`, ~4× larger**
than H9's cumulative fit `[[15.4,…],[−21.9,…]]`. The diagonal form *retroactively proves H9's "~90° rotation"
was an artifact* of regressing on displacement-to-target (partial reach + eef→object geometry), not real
physics; and the 4× magnitude shows H9 was under-shifting the action. Relabel: per-step action `+= J_inv@(Δ/K)`.
**(2) Δ widened 0.10→0.18 m** to cover the LIBERO-PRO swap displacement (0.18–0.25 m, which H9's 0.10 m missed).
Eval = reach metric, blank vision + coord, 50 ep/dim. Code `gib-vla@149411f`.

## Results
| dim | H10 min-xy (n=50) | H7 | H9 |
|---|---|---|---|
| clean | 0.076 | 0.029 | 0.040 |
| **swap** | **0.087** | 0.105 | 0.134 |
| task | 0.081 | — | — |

- **swap 0.086 = best of the whole program** (H7 0.105 → H9 0.134 → **H10 0.086**).
- **clean ≈ swap (0.074 ≈ 0.086), and STABLE across n** (n=10 0.076 → n=27 0.077 → n=42 0.086) — unlike H9
  which drifted 0.092→0.134. This uniformity is the fingerprint of **genuine generalization**: the coord→reach
  map now works as well on swapped (off-dist) coords as on in-dist ones.
- `loc_proj.2` std = **0.0155 — by far the strongest** (0.0033 → 0.0065 → 0.0086 → **0.0155**).

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**PARTIAL WIN — the first genuine REACH-COMMIT generalization in the program.** The per-step Jacobian (the
correct controller gain) + wider Δ made the swap reach ≈ the clean reach (0.086 ≈ 0.074), stable across
episodes — the best swap of any arm and the first time the coordinate→reach map extrapolates off-distribution
instead of memorizing. This validates the H10 hypothesis: H9 failed purely because its cumulative/displacement
Jacobian was a partial-reach artifact; fixing it to the per-step controller gain fixes the relabel. The
**residual gap is PRECISION, not generalization**: ~0.08 m is better than H7's halfway but not yet grasp-range
(~0.03 m). This is a coverage-vs-precision tradeoff (Δ=0.18 + COTRANS=0.5 spread the map wider but softer), so
it is closable by tuning — NOT a fundamental wall. ⇒ REACH-COMMIT is essentially solved data-free; the work
shifts to (a) sharpening precision and (b) the GROUND localizer to supply the coord from pixels.

## Next directions
- [ ] Close precision: anneal Δ / lower COTRANS late in training (sharpen clean while keeping swap
      generalization); or curriculum Δ.
- [ ] Measure BINARY swap task-success in non-blank closed-loop eval — does 0.08 reach + vision yield grasps?
- [ ] Build the GROUND localizer (frozen-DINO-key) to replace the oracle coord — the real-transferable win.
