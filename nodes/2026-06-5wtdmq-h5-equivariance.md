---
id: 5wtdmq
slug: h5-equivariance
type: experiment
status: completed
hypothesis: 'Encode the symmetry the swap benchmark literally measures: TRANSLATIONAL EQUIVARIANCE of the action.

  For two same-instruction demos (same target object) whose object sits at different positions, the

  DIFFERENCE of predicted 8-step reach-displacements must equal the DIFFERENCE of GT displacements (which

  carries the object-location shift Δ). Loss is a PURE FUNCTION of predicted_actions => it reaches the

  ACTION objective itself (the property H1/H2/H4 all lacked — they shaped a representation the L1-optimal

  action could route around). Data-free (existing same-object/different-position RLDS demos), real-

  transferable, warm-start, identity-at-init. Sole survivor of the h5-math-design adversarial workflow

  (6 formulated → 1 alive); the vetter killed the naive grasp_target-anchored form (the 8-step chunk only

  reaches part-way to the grasp point + unit mismatch) and fixed it to the pairwise GT-displacement form.

  '
parents:
- '[[2026-06-j38ajw-h4-proprio-shortcut]]'
links:
- '[[2026-06-ih0sga-h2-grasp-anchored-grounding]]'
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/5wtdmq-h5-equivariance
github-commit: a287242
date-created: 2026-06-03
date-started: 2026-06-03
date-completed: 2026-06-03
tags:
- position-swap
- equivariance
- symmetry
- mathematical
- data-free
metrics:
  clean_base: 1.0
  clean_h5: 0.99
  swap_base: 0.0
  swap_h5: 0.0
  task_base: 0.19
  task_h5: 0.19
  eval_episodes: 49
---

# 5wtdmq — h5-equivariance

## Hypothesis
See frontmatter. Objective:
`L = L1(â, a) + λ · mean_{i≠j} W_ij ‖(d̂_i − d̂_j) − (d_i − d_j)‖²`, where `d̂_i = Σ_{k=0..7} â_i[k,0:3]`
(predicted 8-step translational displacement), `d_i` = same for GT, `W_ij = 1[same instruction]` (prompt
portion of input_ids), λ linearly warmed up. Full vetted spec + the 6-way design report in
`.experiments/attachments/h5-math-design-2026-06-03.json`.

## Parents
[[2026-06-j38ajw-h4-proprio-shortcut]] — H4 ruled out proprio; the elimination motivated a
mathematically-principled objective that acts on the action itself.
Related: [[2026-06-ih0sga-h2-grasp-anchored-grounding]] (reuses the grasp_target object-location label idea).

## Method
`--equiv_weight 1.0 --equiv_warmup_steps 500` (env `GIBVLA_EQUIV_WEIGHT`). Warm-start from `outputs/object`,
2500 steps, batch 16, lr 1e-4, `libero_object_no_noops`. Same-instruction pairs formed within-batch from
the prompt portion of `input_ids` (no Sampler dependency; loss inert when no pairs — logs `equiv_npairs`).
Eval (matched, 100 ep/dim) vs baseline on swap + task + clean. Code `gib-vla@a287242`.

## Results
| dim | base | H5-EQUIV-PAIR (equiv_weight 1.0) | Δ |
|---|---|---|---|
| clean | 1.00 (92/92) | 0.99 (88/89) | −0.01 |
| swap | 0.00 (0/49) | 0.00 (0/49) | 0.00 |
| task | 0.19 (10/53) | 0.19 (10/54) | 0.00 |
(eval stopped ~49 ep/dim; swap 0/49 conclusive.)

**Training dynamics:** the equiv term was clearly ACTIVE (curr 0.02→~0.13 under λ ramp, then recovered
to ~0.05) — it reshaped the action, then converged to a solution satisfying BOTH L1 and the pairwise
equivariance. Clean held (0.99).

## Plots
<!-- ![[attachments/...]] -->

## Conclusion
**NULL on swap (clean held).** The pairwise-equivariance constraint is satisfiable by the memorized
solution *on the training pairs* (where L1 already constrains the function), so the model converges to
something equivariant where it has data but **does not extrapolate** to the unseen swap positions —
confirming the a-priori redundancy concern at the level of *generalization* (not training dynamics).
Key lesson for the whole program: **any data-free fine-tune whose constraint is satisfied by the
training-data fit cannot break position-memorization** — the method must act at TEST TIME, impose
equivariance BY CONSTRUCTION, or construct genuinely off-distribution positions. Pivoting (per user)
to a bolder mathematical sweep under the same data-free + real-transferable constraints.

## Next directions
- [ ] If swap moves: ablate `equiv_weight` (recover clean while keeping swap), report full battery + ≥3 seeds.
- [ ] If swap stays 0 despite active reshaping: pairwise-on-training-pairs doesn't transfer → implement the
      **#2 counterfactual** (swap the VISION between same-instruction samples, require predicted displacement
      to follow the swapped object) — the non-redundant off-distribution version.
