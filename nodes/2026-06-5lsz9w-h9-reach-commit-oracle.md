---
id: 5lsz9w
slug: h9-reach-commit-oracle
type: experiment
status: completed
hypothesis: |
  H9 REACH-COMMIT oracle (gates the whole CPGC no-env program). H7 showed the hard residual coord channel moves swap only HALFWAY (0.105) because in-dist training never decorrelates the coordinate from proprio (they agree on in-dist; H8 tried to decorrelate by shifting proprio but went off-manifold and died). H9 isolates the load-bearing unknown BEFORE any localizer/SAM work: keep proprio REAL/in-range, and on a fraction of frames SHIFT the injected oracle coord (grasp_target xy, COORD_DIM=2) by a random Δ AND co-translate the action's lateral xy by the matching amount (gain estimated by accumulated OLS of cumulative-action-xy on target displacement). Now the triple (real proprio, off-dist coord, co-translated action) is one where proprio-memory is WRONG and the coordinate is the ONLY signal consistent with the action -> the trust-coord-over-proprio gradient must flow through the un-nullable loc_proj. GATE (reach metric, non-blank eval): if swap reach drops below H7's 0.105 toward grasp range (~0.03) AND clean holds => REACH-COMMIT is solvable data-free with in-range proprio => build the full CPGC GROUND head. If swap stays ~0.105 => the commit mechanism is dead and the whole program is gated on it => redesign commit (coord-dropout / up-weight loc_proj) before spending anything on SAM/cut-paste GROUND.
parents: ["[[2026-06-5i0j20-h7-coord-residual-injection]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/5lsz9w-h9-reach-commit-oracle
github-commit: 496f8bc
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: 2026-06-04
tags: [position-swap, reach-commit, co-translation, jacobian, oracle, null-result, relabel-fragility]
metrics: {reach_metric: min_xy_eef_to_target, eval_ep_per_dim: 50, clean_h9: 0.040, swap_h9: 0.134, task_h9: 0.119, clean_h7: 0.029, swap_h7: 0.105, loc_proj_std_h9: 0.0086, J: '[[15.4,-1.7],[-21.9,1.5]]'}
---

# 5lsz9w — h9-reach-commit-oracle

## Hypothesis
H9 REACH-COMMIT oracle (gates the whole CPGC no-env program). H7 showed the hard residual coord channel moves swap only HALFWAY (0.105) because in-dist training never decorrelates the coordinate from proprio (they agree on in-dist; H8 tried to decorrelate by shifting proprio but went off-manifold and died). H9 isolates the load-bearing unknown BEFORE any localizer/SAM work: keep proprio REAL/in-range, and on a fraction of frames SHIFT the injected oracle coord (grasp_target xy, COORD_DIM=2) by a random Δ AND co-translate the action's lateral xy by the matching amount (gain estimated by accumulated OLS of cumulative-action-xy on target displacement). Now the triple (real proprio, off-dist coord, co-translated action) is one where proprio-memory is WRONG and the coordinate is the ONLY signal consistent with the action -> the trust-coord-over-proprio gradient must flow through the un-nullable loc_proj. GATE (reach metric, non-blank eval): if swap reach drops below H7's 0.105 toward grasp range (~0.03) AND clean holds => REACH-COMMIT is solvable data-free with in-range proprio => build the full CPGC GROUND head. If swap stays ~0.105 => the commit mechanism is dead and the whole program is gated on it => redesign commit (coord-dropout / up-weight loc_proj) before spending anything on SAM/cut-paste GROUND.

## Parents
- [[2026-06-5i0j20-h7-coord-residual-injection]]

## Method
Warm-start from `outputs/object`. On `GIBVLA_COTRANS=0.5` of frames: keep proprio REAL/in-range, shift the
injected coord (`grasp_target` xy, `COORD_DIM=2`) by Δ ~ U(±0.10 m) AND co-translate the action's lateral xy
by `J @ Δ`, distributed over the 8-step chunk. **J is a FULL 2×2 image/world→action Jacobian** (control-units
per meter) fit by accumulated least-squares of cumulative-raw-action-xy on target displacement — the diagonal
per-axis gain (H8 + first H9) gave a broken negative y because the **eef/action frame is ~90° rotated vs world**
(converged J ≈ [[15.4, −1.7], [−21.9, 1.5]] — the −21.9 off-diagonal is the rotation). Blank vision (isolates
proprio-vs-coord), keep proprio. Eval = reach metric (min xy eef→true target), blank vision + coord
(`COORD_DIM=2`), 50 ep/dim. Code `gib-vla@496f8bc`.

## Results
| dim | H9 min-xy (n=50) | H7 blank=1.0 | Δ |
|---|---|---|---|
| clean | 0.040 | 0.029 | +0.011 (worse) |
| **swap** | **0.134** | 0.105 | **+0.029 (worse)** |
| task | 0.119 | — | — |

- `loc_proj.2` std = **0.0086 — strongest of all arms** (H7 0.0033 → H8 0.0065 → **H9 0.0086**): the
  co-translation DID force the hardest coordinate-commitment yet.
- **But reach did NOT improve — it got slightly worse on every dim.** Early-episode variance was misleading
  (n=9 clean 0.003 / swap 0.092 drifted to n=50 clean 0.040 / swap 0.134).

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**NULL / slightly NEGATIVE.** The co-translation forced the strongest coordinate-commitment of any arm
(`loc_proj` std 0.0086) — confirming the *mechanism* (decorrelate coord from proprio with real, in-range
proprio) is sound — but reach did **not** improve and slightly worsened. Diagnosed cause: the **analytic action
relabel is too imprecise**. The 8-step chunk is only a *partial* reach toward the object, so the
cumulative-action↔displacement Jacobian is phase-averaged and under-shoots; fitting it as a constant 2×2 (even
with the correct ~90° rotation) corrupts the coord→action map, which the now-hard-committed `loc_proj`
faithfully reproduces → worse. Combined with H8 (shift proprio → off-manifold death): **every data-free
REACH-COMMIT push that perturbs/relabels the action is too fragile to beat H7's simple coord-injection
halfway (swap 0.105).** A clean off-distribution *action* label fundamentally needs either the env (forbidden)
or a precise **per-step** Jacobian (action[t] ↔ eef-motion[t], requires the proprio sequence, not the chunk).
The coordinate *channel* works (H7 = best swap, halfway); pushing past halfway data-free is the wall.

## Next directions
- [ ] Precise per-step Jacobian from the proprio SEQUENCE (not the partial chunk) + wider Δ — less fragile,
      but uncertain; ~1 h build.
- [ ] Or accept H7's halfway-coord and pivot to GROUND (frozen-DINO-key localizer) — does correct-coord +
      halfway-reach still partially help swap? (also answers the SAM question).
- [ ] Or bank the H7→H9 arc: coord channel moves swap halfway; data-free action-relabel cannot push further.
