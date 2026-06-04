---
id: niz154
slug: h14-cotrans-vision-on-coord-override
type: experiment
status: active
hypothesis: |
  H14 coord-OVERRIDE training (co-translation with vision PRESENT). Motivated by H13: the VLA's grounding head localizes on clean but MEMORIZES on swap, and the blank-vision line (H7-H10) cannot grasp by construction (no vision => no fine grasp; 0/50 binary even at reach 0.026). The single change vs H10: GIBVLA_COORD_BLANK_VISION 1.0 -> 0.0 — keep vision ON during co-translation. Mechanism: on the 50% co-translated samples the coord is shifted to P+delta and the action lateral xy is co-shifted by J_inv@delta, but the IMAGE still shows the object at P => the model is trained on (vision says P, coord says P+delta, eef should go to P+delta). This is the missing supervision that teaches POSITION-FOLLOWS-COORD while VISION provides identity + the fine grasp — H10 never saw it because vision was blanked. On the other 50% (non-aug) coord=vision=true position (consistent), preserving clean grasping. GATE (BINARY success, real vision, oracle coord injected, DIM=2): swap > 0 AND clean stays high => the action head CAN execute a real grasp at an off-distribution position when the coordinate overrides the memorized visual prior => REACH-COMMIT works given grounding => the contribution is a real-transferable method (coord from a real detector at deploy) validated end-to-end with oracle grounding on sim. KILL: swap still 0 binary => either the coord channel is ignored when vision is present (prior still wins) or the head cannot grasp off-distribution => the COMMIT wall is real and swap is unreachable on sim by this family. Risk: with vision present the model may learn to ignore the coord (vision-position prior dominates) — the 50% co-translation disagreement is the countermeasure.
parents: ["[[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/niz154-h14-cotrans-vision-on-coord-override
github-commit: null
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: null
tags: []
metrics: {}
---

# niz154 — h14-cotrans-vision-on-coord-override

## Hypothesis
H14 coord-OVERRIDE training (co-translation with vision PRESENT). Motivated by H13: the VLA's grounding head localizes on clean but MEMORIZES on swap, and the blank-vision line (H7-H10) cannot grasp by construction (no vision => no fine grasp; 0/50 binary even at reach 0.026). The single change vs H10: GIBVLA_COORD_BLANK_VISION 1.0 -> 0.0 — keep vision ON during co-translation. Mechanism: on the 50% co-translated samples the coord is shifted to P+delta and the action lateral xy is co-shifted by J_inv@delta, but the IMAGE still shows the object at P => the model is trained on (vision says P, coord says P+delta, eef should go to P+delta). This is the missing supervision that teaches POSITION-FOLLOWS-COORD while VISION provides identity + the fine grasp — H10 never saw it because vision was blanked. On the other 50% (non-aug) coord=vision=true position (consistent), preserving clean grasping. GATE (BINARY success, real vision, oracle coord injected, DIM=2): swap > 0 AND clean stays high => the action head CAN execute a real grasp at an off-distribution position when the coordinate overrides the memorized visual prior => REACH-COMMIT works given grounding => the contribution is a real-transferable method (coord from a real detector at deploy) validated end-to-end with oracle grounding on sim. KILL: swap still 0 binary => either the coord channel is ignored when vision is present (prior still wins) or the head cannot grasp off-distribution => the COMMIT wall is real and swap is unreachable on sim by this family. Risk: with vision present the model may learn to ignore the coord (vision-position prior dominates) — the 50% co-translation disagreement is the countermeasure.

## Parents
- [[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]

## Method
<điền khi /exp-record>

## Results
<điền khi /exp-record>

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
<điền khi /exp-record>

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
