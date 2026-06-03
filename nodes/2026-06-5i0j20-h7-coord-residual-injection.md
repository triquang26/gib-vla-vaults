---
id: 5i0j20
slug: h7-coord-residual-injection
type: experiment
status: active
hypothesis: |
  H7 HARD residual-stream coordinate injection (oracle gate). The 6 prior nulls + H6 gate show object position reaches the L1 action head only via SOFT paths (attention K/V on h_a/p/h_t, aux losses) the L1-optimal head routes around, and the in-model localizer MEMORIZES. H7 isolates the load-bearing question with an oracle: inject a CORRECT absolute 3D target coordinate (grasp_target object-location proxy) into EVERY MLPResNet block residual (output+x+loc_proj(coord), zero-init => warm-start byte-identical) — an additive, un-nullable channel, the mechanical inverse of every soft path — while BLANKING vision at both train and eval to remove the memorization shortcut, making the policy a pure reach-to-coordinate controller. GREENLIGHT if swap-approx-clean (coordinate->reach extrapolates to swapped/off-dist positions) => build the JEPA-honest localizer to supply the coordinate; KILL if swap~0 even with a perfect coordinate handed in => the head cannot turn a coordinate into a generalizing reach, the external-grounding fix is dead (stronger negative than all 6 priors).
parents: ["[[2026-06-k2gwp2-h6-visualservo-tta-gate]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/5i0j20-h7-coord-residual-injection
github-commit: null
date-created: 2026-06-03
date-started: 2026-06-03
date-completed: null
tags: []
metrics: {}
---

# 5i0j20 — h7-coord-residual-injection

## Hypothesis
H7 HARD residual-stream coordinate injection (oracle gate). The 6 prior nulls + H6 gate show object position reaches the L1 action head only via SOFT paths (attention K/V on h_a/p/h_t, aux losses) the L1-optimal head routes around, and the in-model localizer MEMORIZES. H7 isolates the load-bearing question with an oracle: inject a CORRECT absolute 3D target coordinate (grasp_target object-location proxy) into EVERY MLPResNet block residual (output+x+loc_proj(coord), zero-init => warm-start byte-identical) — an additive, un-nullable channel, the mechanical inverse of every soft path — while BLANKING vision at both train and eval to remove the memorization shortcut, making the policy a pure reach-to-coordinate controller. GREENLIGHT if swap-approx-clean (coordinate->reach extrapolates to swapped/off-dist positions) => build the JEPA-honest localizer to supply the coordinate; KILL if swap~0 even with a perfect coordinate handed in => the head cannot turn a coordinate into a generalizing reach, the external-grounding fix is dead (stronger negative than all 6 priors).

## Parents
- [[2026-06-k2gwp2-h6-visualservo-tta-gate]]

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
