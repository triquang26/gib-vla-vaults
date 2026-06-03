---
id: 5lsz9w
slug: h9-reach-commit-oracle
type: experiment
status: active
hypothesis: |
  H9 REACH-COMMIT oracle (gates the whole CPGC no-env program). H7 showed the hard residual coord channel moves swap only HALFWAY (0.105) because in-dist training never decorrelates the coordinate from proprio (they agree on in-dist; H8 tried to decorrelate by shifting proprio but went off-manifold and died). H9 isolates the load-bearing unknown BEFORE any localizer/SAM work: keep proprio REAL/in-range, and on a fraction of frames SHIFT the injected oracle coord (grasp_target xy, COORD_DIM=2) by a random Δ AND co-translate the action's lateral xy by the matching amount (gain estimated by accumulated OLS of cumulative-action-xy on target displacement). Now the triple (real proprio, off-dist coord, co-translated action) is one where proprio-memory is WRONG and the coordinate is the ONLY signal consistent with the action -> the trust-coord-over-proprio gradient must flow through the un-nullable loc_proj. GATE (reach metric, non-blank eval): if swap reach drops below H7's 0.105 toward grasp range (~0.03) AND clean holds => REACH-COMMIT is solvable data-free with in-range proprio => build the full CPGC GROUND head. If swap stays ~0.105 => the commit mechanism is dead and the whole program is gated on it => redesign commit (coord-dropout / up-weight loc_proj) before spending anything on SAM/cut-paste GROUND.
parents: ["[[2026-06-5i0j20-h7-coord-residual-injection]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/5lsz9w-h9-reach-commit-oracle
github-commit: null
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: null
tags: []
metrics: {}
---

# 5lsz9w — h9-reach-commit-oracle

## Hypothesis
H9 REACH-COMMIT oracle (gates the whole CPGC no-env program). H7 showed the hard residual coord channel moves swap only HALFWAY (0.105) because in-dist training never decorrelates the coordinate from proprio (they agree on in-dist; H8 tried to decorrelate by shifting proprio but went off-manifold and died). H9 isolates the load-bearing unknown BEFORE any localizer/SAM work: keep proprio REAL/in-range, and on a fraction of frames SHIFT the injected oracle coord (grasp_target xy, COORD_DIM=2) by a random Δ AND co-translate the action's lateral xy by the matching amount (gain estimated by accumulated OLS of cumulative-action-xy on target displacement). Now the triple (real proprio, off-dist coord, co-translated action) is one where proprio-memory is WRONG and the coordinate is the ONLY signal consistent with the action -> the trust-coord-over-proprio gradient must flow through the un-nullable loc_proj. GATE (reach metric, non-blank eval): if swap reach drops below H7's 0.105 toward grasp range (~0.03) AND clean holds => REACH-COMMIT is solvable data-free with in-range proprio => build the full CPGC GROUND head. If swap stays ~0.105 => the commit mechanism is dead and the whole program is gated on it => redesign commit (coord-dropout / up-weight loc_proj) before spending anything on SAM/cut-paste GROUND.

## Parents
- [[2026-06-5i0j20-h7-coord-residual-injection]]

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
