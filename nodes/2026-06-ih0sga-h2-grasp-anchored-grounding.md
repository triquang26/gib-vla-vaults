---
id: ih0sga
slug: h2-grasp-anchored-grounding
type: experiment
status: completed
date-completed: 2026-06-03
hypothesis: |
  Position-swap collapse (swap=0.00) is a spatial-grounding failure: the policy grasps the
  remembered location, not the named object. Fix data-free with a real-transferable supervision
  signal no prior work uses — the demonstration's own GRASP PROPRIOCEPTION. The named target sits
  where the gripper closes, so eef xyz at the grasp moment (fingers most closed) localizes it
  (free, exists on real robots; verified clean+discriminative on libero_object). Add an explicit
  localization head (image+instruction -> target xyz) trained jointly with the action loss
  (H2a). If the head merely memorizes object->position (some libero objects are near-fixed), add
  counterfactual instruction-rebinding (wrong instruction -> push prediction off the grasp loc)
  to force instruction-conditioned visual localization (H2b). Target: lift swap off 0.00 (and
  task) without hurting clean. Novelty vs ReconVLA (latent reconstruction, detector labels,
  object-level), ObjectVLA (object-id, VL pairs), RoboPoint (sim-GT, separate VLM), LIBERO-Plus
  (20k new trajectories, 7B): explicit head + grasp-proprio label + position-swap eval, small VLA.
parents: ["[[2026-06-0jw4rh-h1-langcond-gate]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/ih0sga-h2-grasp-anchored-grounding
github-commit: null
date-created: 2026-06-03
date-started: 2026-06-03
date-completed: null
tags: [grounding, position-swap, grasp-pose, data-free, real-transferable]
metrics: {h2a_clean_base: 1.00, h2a_clean: 0.94, h2a_swap_base: 0.00, h2a_swap: 0.00, h2a_task_base: 0.10, h2a_task: 0.09, h2b_clean: 0.95, h2b_swap: 0.00, h2b_task: 0.24, h2a_eval_ep: 100, h2b_eval_ep: 34}
---

# ih0sga — h2-grasp-anchored-grounding

## Hypothesis
See frontmatter. Data-free, real-transferable grounding via grasp-proprioception localization label;
warm-start from outputs/object; eval only the collapse dims (swap, task) + clean guard.

## Parents
[[2026-06-0jw4rh-h1-langcond-gate]] — H1 (FiLM on vision IB gate) was NULL on swap; representation
tweak insufficient → move the intervention to a supervision-side localization objective.

## Method
Free, real-transferable localization label: `batch["grasp_target"]` = eef xyz at the grasp moment
(gripper fingers most closed = where the named object is), added to the RLDS pipeline (env-gated
`GIBVLA_GRASP_TARGET=1`), verified clean+discriminative. Code `gib-vla` branch `exp/ih0sga`.

**H2a (this result):** an explicit, instruction-conditioned localization aux head
(`prismatic/util/grasp_head.py`: pooled instruction *queries* the projected visual patches via
cross-attention → MLP → 3D), trained jointly with the action loss (`weight·MSE(pred, grasp_target)`,
weight 1.0). Warm-start from `outputs/object`, 2500 steps, batch 16, lr 1e-4. Eval (`scripts/eval_compare.sh`,
matched 100 ep/dim) vs baseline on swap+task+clean.

## Results
| dim | base (`outputs/object`) | H2a (grasp head, 2500) | Δ |
|---|---|---|---|
| clean | 1.00 (100/100) | 0.94 (94/100) | −0.06 |
| swap (position) | 0.00 (0/100) | 0.00 (0/100) | 0.00 |
| task | 0.10 (10/100) | 0.09 (9/100) | −0.01 |

## Plots
<!-- ![[attachments/...]] -->

## Conclusion
**H2a NULL on swap** (and mild clean regression). The aux head *grounds the visual representation*
(forces it to predict the named object's location) but the **action policy does not use that
grounding** — swap stays exactly floored at 0.00, task unchanged, clean −0.06. Diagnosis (matches the
data): many libero-object objects sit at near-fixed training positions, so the head can satisfy the
loss by memorizing object→position, and even when it localizes, the grounding only reaches the action
indirectly via the shared LoRA representation. ⇒ the intervention must **reach the action explicitly**.

## Next directions
- [x] **H2b — NULL** (counterfactual instruction-rebinding, warm-start 2500, 34 ep/dim): clean 0.95,
      **swap 0.00**, task 0.24 (vs base 1.00/0.00/0.26). The localizer's grasp_loss fell 0.031→0.012
      (it *did* learn to localize), yet swap stays floored ⇒ instruction-conditional grounding of the
      *representation* still does not change the action. Stopped early; swap 0/34 is conclusive.
- [x] **H2c — coded, SUPERSEDED (not cleanly evaluated).** In-model `TargetLocalizer` adds a zero-init
      modulation to `action_queries` (full PEFT integration: explicit LoRA target list excluding the
      localizer + merge-copy of the trained module; identity-at-init verified, grasp_loss decreased). The
      run got tangled by rapid relaunches, and the re-hypothesize workflow then showed H2c is the SAME
      failure class (zero-init modulation has no L1 gradient to activate while proprio/position stays a
      sufficient statistic) → not worth a clean re-run. Code is on the branch for reuse by H5.
- [x] **Conclusion of the H2 family: NULL on swap.** Grasp-proprio is a clean, novel, real-transferable
      localization label and the head provably learns it (grasp_loss↓), but representation-grounding never
      reaches the action. This motivated [[2026-06-j38ajw-h4-proprio-shortcut]] (H4, also null → proprio
      ruled out) and the mathematically-principled **H5** phase (position-equivariance / conditional-MI).
- [ ] Reusable assets for H5: `grasp_target` data label (object 3D location, free), `TargetLocalizer`
      (visual object-location read), `scripts/train_warmstart.sh` + `eval_compare.sh`.
