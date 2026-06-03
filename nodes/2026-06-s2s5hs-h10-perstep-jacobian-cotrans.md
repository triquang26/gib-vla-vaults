---
id: s2s5hs
slug: h10-perstep-jacobian-cotrans
type: experiment
status: active
hypothesis: |
  H10 PER-STEP Jacobian co-translation (fixes H9's relabel fragility). H9 forced the strongest coord-commitment (loc_proj 0.0086) but reach did NOT improve because its action-relabel Jacobian was fit by regressing the CUMULATIVE 8-step chunk action on the FULL displacement-to-target — the chunk is only a PARTIAL reach, so that Jacobian is phase-averaged and under-shoots, corrupting the coord->action map. H10 fixes this: fit the PER-STEP controller gain J_inv (control-units per meter) from the ACTUAL per-step pair (action[0], eef_motion = proprio[t+1]-proprio[t]), a CONSTANT controller property with no partial-reach confound (plumb a new raw eef_motion key through the RLDS transform + collator). Use J_inv for the co-translation relabel (per-step action shift = J_inv @ Δ/K), keep proprio REAL, widen Δ to ~0.18m to cover the LIBERO-PRO swap displacement (0.18-0.25m, which H9's 0.10m missed). GATE (reach metric, blank eval): if swap reach NOW drops below H7's 0.105 toward grasp range (~0.03-0.06) AND clean holds => REACH-COMMIT SOLVED data-free => GREENLIGHT the CPGC GROUND localizer. If swap stays ~0.10-0.13 => the data-free action-relabel route is exhausted (last shot) => bank the H7->H10 arc and the env-free REACH-COMMIT wall is real.
parents: ["[[2026-06-5lsz9w-h9-reach-commit-oracle]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/s2s5hs-h10-perstep-jacobian-cotrans
github-commit: null
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: null
tags: []
metrics: {}
---

# s2s5hs — h10-perstep-jacobian-cotrans

## Hypothesis
H10 PER-STEP Jacobian co-translation (fixes H9's relabel fragility). H9 forced the strongest coord-commitment (loc_proj 0.0086) but reach did NOT improve because its action-relabel Jacobian was fit by regressing the CUMULATIVE 8-step chunk action on the FULL displacement-to-target — the chunk is only a PARTIAL reach, so that Jacobian is phase-averaged and under-shoots, corrupting the coord->action map. H10 fixes this: fit the PER-STEP controller gain J_inv (control-units per meter) from the ACTUAL per-step pair (action[0], eef_motion = proprio[t+1]-proprio[t]), a CONSTANT controller property with no partial-reach confound (plumb a new raw eef_motion key through the RLDS transform + collator). Use J_inv for the co-translation relabel (per-step action shift = J_inv @ Δ/K), keep proprio REAL, widen Δ to ~0.18m to cover the LIBERO-PRO swap displacement (0.18-0.25m, which H9's 0.10m missed). GATE (reach metric, blank eval): if swap reach NOW drops below H7's 0.105 toward grasp range (~0.03-0.06) AND clean holds => REACH-COMMIT SOLVED data-free => GREENLIGHT the CPGC GROUND localizer. If swap stays ~0.10-0.13 => the data-free action-relabel route is exhausted (last shot) => bank the H7->H10 arc and the env-free REACH-COMMIT wall is real.

## Parents
- [[2026-06-5lsz9w-h9-reach-commit-oracle]]

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
