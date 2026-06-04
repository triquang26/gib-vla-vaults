---
id: niz154
slug: h14-cotrans-vision-on-coord-override
type: experiment
status: completed
hypothesis: 'H14 coord-OVERRIDE training (co-translation with vision PRESENT). Motivated by H13: the VLA''s grounding head localizes on clean but MEMORIZES on swap, and the blank-vision line (H7-H10) cannot grasp by construction (no vision => no fine grasp; 0/50 binary even at reach 0.026). The single change vs H10: GIBVLA_COORD_BLANK_VISION 1.0 -> 0.0 — keep vision ON during co-translation. Mechanism: on the 50% co-translated samples the coord is shifted to P+delta and the action lateral xy is co-shifted by J_inv@delta, but the IMAGE still shows the object at P => the model is trained on (vision says P, coord says P+delta, eef should go to P+delta). This is the missing supervision that teaches POSITION-FOLLOWS-COORD while VISION provides identity + the fine grasp — H10 never saw it because vision was blanked. On the other 50% (non-aug) coord=vision=true position (consistent), preserving clean grasping. GATE (BINARY success, real vision, oracle coord injected, DIM=2): swap > 0 AND clean
  stays high => the action head CAN execute a real grasp at an off-distribution position when the coordinate overrides the memorized visual prior => REACH-COMMIT works given grounding => the contribution is a real-transferable method (coord from a real detector at deploy) validated end-to-end with oracle grounding on sim. KILL: swap still 0 binary => either the coord channel is ignored when vision is present (prior still wins) or the head cannot grasp off-distribution => the COMMIT wall is real and swap is unreachable on sim by this family. Risk: with vision present the model may learn to ignore the coord (vision-position prior dominates) — the 50% co-translation disagreement is the countermeasure.

  '
parents:
- '[[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]'
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/niz154-h14-cotrans-vision-on-coord-override
github-commit: 04d0d93a6e57425cbb5241869d30df74a277daf6
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: 2026-06-04
tags: []
metrics:
  eval: binary_success_oracle_coord_real_vision
  change_vs_h10: blank_vision_1.0_to_0.0
  clean: 48/50 (0.96)
  swap: 0/50 (0.00)
  task: ~0.13
  swap_reach_med_m: 0.13
  clean_reach_med_m: 0.004
  verdict: KILL-on-swap; clean-PRESERVED; additive-coord-ignored-when-vision-conflicts
---

# niz154 — h14-cotrans-vision-on-coord-override

## Hypothesis
H14 coord-OVERRIDE training (co-translation with vision PRESENT). Motivated by H13: the VLA's grounding head localizes on clean but MEMORIZES on swap, and the blank-vision line (H7-H10) cannot grasp by construction (no vision => no fine grasp; 0/50 binary even at reach 0.026). The single change vs H10: GIBVLA_COORD_BLANK_VISION 1.0 -> 0.0 — keep vision ON during co-translation. Mechanism: on the 50% co-translated samples the coord is shifted to P+delta and the action lateral xy is co-shifted by J_inv@delta, but the IMAGE still shows the object at P => the model is trained on (vision says P, coord says P+delta, eef should go to P+delta). This is the missing supervision that teaches POSITION-FOLLOWS-COORD while VISION provides identity + the fine grasp — H10 never saw it because vision was blanked. On the other 50% (non-aug) coord=vision=true position (consistent), preserving clean grasping. GATE (BINARY success, real vision, oracle coord injected, DIM=2): swap > 0 AND clean stays high => the action head CAN execute a real grasp at an off-distribution position when the coordinate overrides the memorized visual prior => REACH-COMMIT works given grounding => the contribution is a real-transferable method (coord from a real detector at deploy) validated end-to-end with oracle grounding on sim. KILL: swap still 0 binary => either the coord channel is ignored when vision is present (prior still wins) or the head cannot grasp off-distribution => the COMMIT wall is real and swap is unreachable on sim by this family. Risk: with vision present the model may learn to ignore the coord (vision-position prior dominates) — the 50% co-translation disagreement is the countermeasure.

## Parents
- [[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]

## Method
Single change vs H10 ([[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]): `GIBVLA_COORD_BLANK_VISION` 1.0 -> 0.0
(vision PRESENT during co-translation). Warm-start outputs/object, 2500 steps, b16, lr1e-4, lora r64, DIM=2,
COTRANS=0.5 — co-translation Jinv converged to the H10 value [[64.98,-8.55],[2.19,76.27]]. EVAL = BINARY
success (not the reach proxy) with the oracle coord injected at inference (GIBVLA_COORD_INJECT=1, DIM=2), REAL
vision (no blank), proprio natural (no xyz-zero), on clean/swap/task, 50 ep each (`scripts/watch_eval_h14.sh`).

## Results
| dim | binary success | median reach to target |
|---|---|---|
| **clean** | **48/50 = 0.96** | 0.004 m |
| **swap** | **0/50 = 0.00** | **0.130 m** |
| task | ~0.13 (5/~40) | 0.060 m |

- Clean grasping is **fully preserved** (0.96) — the decisive contrast with the blank-vision line (H7-H10:
  0/50 even clean). Vision-present training keeps the policy intact.
- Swap is **0** and the eef reach stays at **0.13 m** — i.e. it drives to the MEMORIZED location (the swap
  displacement is ~0.13 m), not toward the oracle coord. The additive coord channel produces ~0 correction.
- Task (positions unchanged, goal changed) is partially rescued (0 -> ~0.13) because there coord AGREES with
  vision — confirming the failure is specifically the coord-vs-vision CONFLICT on swap.

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
KILL on swap — but the most informative KILL of the program. With vision PRESENT the oracle coord cannot
override the learned vision->position pathway: on swap the eef goes to the memorized spot (reach 0.13 m, the
swap displacement) and never moves toward the handed-in target. Yet clean grasping is **intact (0.96)**. This
proves two things at once: (1) the additive zero-init coord residual is structurally too weak to beat the
visual prior at inference (a stronger-magnitude retrain is unlikely to fix a ~0 effect — it is being ignored,
not merely under-weighted); (2) **the VLA grasp machinery is fully functional** — descend, gripper timing, lift
all work when the eef is at the object. So the swap problem REDUCES to a single residual: redirect the
approach-xy to the target. Combined with the blank-vision result (H7-H10: reaches via coord but cannot grasp
with no vision), the in-network coord-injection family is closed both ways. ==> The fix must be done OUTSIDE
the network at test time, where the prior cannot veto it (next: [[2026-06-h15...]] test-time controller-VLA
hybrid: override only the action xy with the analytic controller Jinv*(coord-eef), keep the VLA z/grip/grasp,
distance-gated; sim-validatable with oracle coord, no retrain, real-transferable via a detector at deploy).

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
