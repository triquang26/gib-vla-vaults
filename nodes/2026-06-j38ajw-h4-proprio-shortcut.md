---
id: j38ajw
slug: h4-proprio-shortcut
type: experiment
status: active
hypothesis: |
  ROOT CAUSE (found by the h2-rehypothesize workflow, code-confirmed): the action head is fed
  `proprio` (current eef xyz pose) directly, and L1 on the delta-action chunk is the only action
  signal. Given the current eef pose + a per-task position prior, the action is ~predictable WITHOUT
  localizing the named object -> proprio is a SUFFICIENT STATISTIC for L1 -> every representation-
  grounding attempt (H1 IB-gate, H2a/b grasp head, H2c action_queries modulation) is dead weight the
  action head ignores, which is exactly why swap stays 0.00.
  H4-PROP: dropout the translational proprio (proprio[:,0:3], eef xyz) during warm-start fine-tune so
  the action head CANNOT use current-pose to memorize position -> forced to read the vision tokens for
  WHERE the named object is. Data-free, real-transferable, S-effort. Decisive: swap moves => proprio
  shortcut confirmed; swap stays 0 + clean high => shortcut is in the vision patch-index (-> pixel/patch
  interventions H4-SPOT/H4-MASKAUG, or combine with the H2c localizer so localization is the only
  remaining source of "where").
parents: ["[[2026-06-ih0sga-h2-grasp-anchored-grounding]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/j38ajw-h4-proprio-shortcut
github-commit: null
date-created: 2026-06-03
date-started: 2026-06-03
date-completed: null
tags: [position-swap, root-cause, proprio-shortcut, sufficient-statistic, data-free]
metrics: {}
---

# j38ajw — h4-proprio-shortcut

## Hypothesis
See frontmatter. Attacks the SHORTCUT (proprio sufficient statistic) instead of adding grounding to a
representation the action head can ignore. Full re-hypothesis report + the H4 ladder (H4-PROP/CONTRA/
SPOT/MASKAUG) in `.experiments/attachments/h2-rehypothesize-2026-06-03.json`.

## Parents
[[2026-06-ih0sga-h2-grasp-anchored-grounding]] — H2 family (grasp-anchored grounding) all NULL on swap;
the workflow analysis of WHY produced this root cause and redirected the attack to the action shortcut.

## Method
H4-PROP (running): `--proprio_xyz_dropout 0.5` — per-sample Bernoulli zeroing of `proprio[:,0:3]` (eef
xyz) in `run_forward_pass` during training (eval uses real proprio). Warm-start from `outputs/object`,
2500 steps, batch 16, lr 1e-4, `libero_object_no_noops`. Eval (matched, 100 ep/dim) vs baseline on
swap + task + clean. Pure input ablation — zero new labels.

## Results
<điền khi /exp-record>

## Plots
<!-- ![[attachments/...]] -->

## Conclusion
<điền khi /exp-record>

## Next directions
- [ ] If swap moves: ablate dropout rate (0.5 vs 1.0), add object-relative action frame, then write up
      "position-debiasing fine-tune for small VLAs".
- [ ] If null + clean high: pivot to pixel/patch interventions (H4-SPOT detector spotlight + decoy
      decorrelation; H4-MASKAUG canonical-location patch dropout).
- [ ] Combine H4-PROP + H2c localizer (localized target becomes the only source of "where").
