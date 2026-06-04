---
id: dinaz8
slug: h15-testtime-controller-vla-hybrid
type: experiment
status: active
hypothesis: |
  H15 test-time controller-VLA hybrid. H14 proved the VLA grasp machinery is INTACT (clean 0.96) and swap fails PURELY because the approach-xy drives to the memorized location (reach stuck at 0.13 m = the swap displacement); and that the additive coord channel is IGNORED inside the network when vision conflicts (~0 correction). So do the position override OUTSIDE the network at test time, where the prior cannot veto it. At each control step, override ONLY the action xy with a proportional controller toward the oracle target coord: action[0:2] = clip(gain*(coord_xy - eef_xy), -bound, bound) while ||eef-coord|| > eps (approach), and hand xy back to the VLA when within eps (fine grasp), keeping the VLA's z/rotation/gripper throughout. Sign/scale resolved by a quick gain sweep. GATE (BINARY, real vision, oracle coord, on the RELEASED model so no retrain confound): swap > 0 AND clean stays high => the decomposition works — external grounding supplies WHERE, the VLA supplies the GRASP — a real, real-transferable end-to-end fix (coord from a detector at deploy) validated with oracle grounding on sim. KILL: swap still 0 => even a perfect external xy controller cannot land the grasp (the VLA's z/grip depend on the visual-position context being consistent) => the COMMIT coupling is deeper than approach-xy and swap is unreachable on sim by this whole family.
parents: ["[[2026-06-niz154-h14-cotrans-vision-on-coord-override]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/dinaz8-h15-testtime-controller-vla-hybrid
github-commit: null
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: null
tags: []
metrics: {}
---

# dinaz8 — h15-testtime-controller-vla-hybrid

## Hypothesis
H15 test-time controller-VLA hybrid. H14 proved the VLA grasp machinery is INTACT (clean 0.96) and swap fails PURELY because the approach-xy drives to the memorized location (reach stuck at 0.13 m = the swap displacement); and that the additive coord channel is IGNORED inside the network when vision conflicts (~0 correction). So do the position override OUTSIDE the network at test time, where the prior cannot veto it. At each control step, override ONLY the action xy with a proportional controller toward the oracle target coord: action[0:2] = clip(gain*(coord_xy - eef_xy), -bound, bound) while ||eef-coord|| > eps (approach), and hand xy back to the VLA when within eps (fine grasp), keeping the VLA's z/rotation/gripper throughout. Sign/scale resolved by a quick gain sweep. GATE (BINARY, real vision, oracle coord, on the RELEASED model so no retrain confound): swap > 0 AND clean stays high => the decomposition works — external grounding supplies WHERE, the VLA supplies the GRASP — a real, real-transferable end-to-end fix (coord from a detector at deploy) validated with oracle grounding on sim. KILL: swap still 0 => even a perfect external xy controller cannot land the grasp (the VLA's z/grip depend on the visual-position context being consistent) => the COMMIT coupling is deeper than approach-xy and swap is unreachable on sim by this whole family.

## Parents
- [[2026-06-niz154-h14-cotrans-vision-on-coord-override]]

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
