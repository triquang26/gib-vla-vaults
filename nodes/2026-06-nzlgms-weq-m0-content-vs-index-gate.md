---
id: nzlgms
slug: weq-m0-content-vs-index-gate
type: experiment
status: completed
hypothesis: 'WEQ-M0 gate: on object position-swap, StableVLA''s failure is content-based (it tracks WHERE IT SEES the target object), not pure patch-index (task bound to an image cell regardless of content). Test data-free on the frozen released checkpoint: using the oracle segmentation mask, hard-crop the scene to show ONLY the target object at its NEW (swapped) position plus the gripper, black out everything else, keep the instruction; roll out and log the eef trajectory. If the eef converges to the NEW position -> content-based -> the where-equivariant (WEQ) direction is alive -> proceed to M1. If it still goes to the memorized OLD (now empty) position -> pure patch-index -> WEQ and every implicit-visual fix is dead -> pivot to a diagnostic-only paper (brief Appendix A). Report the full distribution of final-eef xy-distance to NEW vs OLD, not just the mean.

  '
parents: []
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/nzlgms-weq-m0-content-vs-index-gate
github-commit: 89f45dcc5cfce7afe837c7256f6ef21d64371b54
date-created: 2026-06-13
date-started: 2026-06-13
date-completed: 2026-06-13
tags: []
metrics:
  swap_SR: 0.0
  clean_full_SR: 1.0
  clean_blackav_SR: 0.85
  swap_crop_frac_to_OLD: 1.0
  swap_crop_frac_to_NEW: 0.025
  swap_nocrop_frac_to_OLD: 1.0
  swap_blackav_frac_to_OLD: 1.0
  delta_frac_to_NEW_crop_vs_nocrop: 0.0
  d_old_min_median_m: 0.006
  N_per_condition: 40
  verdict: patch_index_proprio_prior
---

# nzlgms — weq-m0-content-vs-index-gate

## Hypothesis
WEQ-M0 gate: on object position-swap, StableVLA's failure is content-based (it tracks WHERE IT SEES the target object), not pure patch-index (task bound to an image cell regardless of content). Test data-free on the frozen released checkpoint: using the oracle segmentation mask, hard-crop the scene to show ONLY the target object at its NEW (swapped) position plus the gripper, black out everything else, keep the instruction; roll out and log the eef trajectory. If the eef converges to the NEW position -> content-based -> the where-equivariant (WEQ) direction is alive -> proceed to M1. If it still goes to the memorized OLD (now empty) position -> pure patch-index -> WEQ and every implicit-visual fix is dead -> pivot to a diagnostic-only paper (brief Appendix A). Report the full distribution of final-eef xy-distance to NEW vs OLD, not just the mean.

## Parents
(none — root)

## Method
Data-free probe on the frozen StableVLA checkpoint (`outputs/object`) over `libero_object_swap`. For each genuine target-swap episode (N=40/condition = 10 tasks x 4 trials, kept only ||NEW-OLD||_xy >= 5cm), we rewrite **only the input image** using the LIBERO oracle instance segmentation, roll out, and log the eef xy-distance to the NEW (swapped) vs OLD (memorized clean-layout) target position.

Conditions: `swap_nocrop` (full image) · `swap_crop` (agentview masked to target@NEW + gripper, OLD location blacked) · `swap_blackav` (agentview fully blank). Clean control: `clean_full` vs `clean_blackav` (N=20 each). Harness: `scripts/weq_m0_content_vs_index.py` (+ `weq_m0_probe.py`, `weq_m0_analyze.py`).

## Results
| condition | frac eef→OLD (<6cm) | frac→NEW | d→OLD min (median) | SR |
|---|---|---|---|---|
| swap_nocrop | **1.00** | 0.025 | 6mm | 0.00 |
| swap_crop (target only@NEW) | **1.00** | 0.025 | 6mm | 0.00 |
| swap_blackav (agentview black) | **1.00** | 0.025 | 16mm | 0.00 |
| clean_full | — (→target 4mm) | 1.00 | — | **1.00** |
| clean_blackav (agentview black) | — (→target 6mm) | 1.00 | — | **0.85** |

Δ frac_near_NEW (crop − nocrop) = **+0.00**. Causal probe (single forward, `eval_logs/m0_probe/`): net xy-action under {full, crop, blackav} all point +x,−y → OLD; blacking agentview barely changes the action.

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
VERDICT = **PATCH-INDEX / proprio-prior, NOT content-based.** `swap_crop` == `swap_nocrop` == `swap_blackav`: 100% of episodes the eef goes to OLD (median 6mm), 2.5% reach NEW; crop changes frac_near_NEW by +0.00. Blanking the whole agentview does not change OLD-going behavior → the third-person scene is **non-causal** for the approach. Clean agentview-blackout drops SR only 1.00→0.85 → agentview is **near-vestigial**; localization is driven by a proprio+wrist+instruction memorized prior.

Sharper than the brief anticipated: the position prior lives in **proprio+instruction, not in the visual `h_what`** → WEQ`s `L_what_invariance` (visual-only) could not remove it even if built. Per the brief`s M0 rule, the implicit-visual where-equivariant direction is **dead**. Pivot: Appendix-A diagnostic + a de-confounding constructive fix (random-spawn retrain / external controller). Consistent with vault priors [[2026-06-j38ajw-h4-proprio-shortcut]], [[2026-06-5i0j20-h7-coord-residual-injection]], [[2026-06-niz154-h14-cotrans-vision-on-coord-override]].

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
