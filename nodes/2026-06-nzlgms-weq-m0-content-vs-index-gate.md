---
id: nzlgms
slug: weq-m0-content-vs-index-gate
type: experiment
status: active
hypothesis: |
  WEQ-M0 gate: on object position-swap, StableVLA's failure is content-based (it tracks WHERE IT SEES the target object), not pure patch-index (task bound to an image cell regardless of content). Test data-free on the frozen released checkpoint: using the oracle segmentation mask, hard-crop the scene to show ONLY the target object at its NEW (swapped) position plus the gripper, black out everything else, keep the instruction; roll out and log the eef trajectory. If the eef converges to the NEW position -> content-based -> the where-equivariant (WEQ) direction is alive -> proceed to M1. If it still goes to the memorized OLD (now empty) position -> pure patch-index -> WEQ and every implicit-visual fix is dead -> pivot to a diagnostic-only paper (brief Appendix A). Report the full distribution of final-eef xy-distance to NEW vs OLD, not just the mean.
parents: []
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/nzlgms-weq-m0-content-vs-index-gate
github-commit: null
date-created: 2026-06-13
date-started: 2026-06-13
date-completed: null
tags: []
metrics: {}
---

# nzlgms — weq-m0-content-vs-index-gate

## Hypothesis
WEQ-M0 gate: on object position-swap, StableVLA's failure is content-based (it tracks WHERE IT SEES the target object), not pure patch-index (task bound to an image cell regardless of content). Test data-free on the frozen released checkpoint: using the oracle segmentation mask, hard-crop the scene to show ONLY the target object at its NEW (swapped) position plus the gripper, black out everything else, keep the instruction; roll out and log the eef trajectory. If the eef converges to the NEW position -> content-based -> the where-equivariant (WEQ) direction is alive -> proceed to M1. If it still goes to the memorized OLD (now empty) position -> pure patch-index -> WEQ and every implicit-visual fix is dead -> pivot to a diagnostic-only paper (brief Appendix A). Report the full distribution of final-eef xy-distance to NEW vs OLD, not just the mean.

## Parents
(none — root)

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
