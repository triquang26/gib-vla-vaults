---
id: 0jw4rh
slug: h1-langcond-gate
type: experiment
status: completed
hypothesis: |
  Make the FusedFAN IB gate language-conditioned: thread a pooled instruction embedding into EfficientChannelAttention and FiLM the sigmoid channel-gate (identity-init so untrained==baseline). Objective shifts from vision-only min I(X;Z)-beta I(Z;S) to language-conditioned min I(X;Z)-beta I(Z;S|L). Strictly data-free (uses existing language_instruction). Target: lift the task (0.093) and position (0.00) collapse dims without hurting clean.
parents: ["[[2026-06-bekxnt-h0-stablevla-libero-pro-baseline]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/0jw4rh-h1-langcond-gate
github-commit: 0b866a1
date-created: 2026-06-02
date-started: 2026-06-02
date-completed: 2026-06-03
tags: [null-result, warm-start, collapse-dims]
metrics: {clean_base: 1.00, clean_h1: 0.97, swap_base: 0.00, swap_h1: 0.00, task_base: 0.10, task_h1: 0.07, eval_episodes_per_dim: 100}
---

# 0jw4rh — h1-langcond-gate

## Hypothesis
Make the FusedFAN IB gate language-conditioned: thread a pooled instruction embedding into EfficientChannelAttention and FiLM the sigmoid channel-gate (identity-init so untrained==baseline). Objective shifts from vision-only min I(X;Z)-beta I(Z;S) to language-conditioned min I(X;Z)-beta I(Z;S|L). Strictly data-free (uses existing language_instruction). Target: lift the task (0.093) and position (0.00) collapse dims without hurting clean.

## Parents
[[2026-06-bekxnt-h0-stablevla-libero-pro-baseline]]

## Method
Implementation (clean OOP): `prismatic/util/lang_cond.py::FiLMGate` (zero-init output → identity at step 0),
threaded into `EfficientChannelAttention`/`FusedFANProjector` (`prismatic/util/nn_utils.py`) and the
projector forward (`prismatic/extern/hf/modeling_prismatic.py`): the pre-sigmoid channel-gate logits are
FiLM-modulated by the pooled instruction embedding (`gate' = gate*γ + β`, γ=1+dγ, identity at init).

**Training = warm-start** (adopted as the standard protocol for all hypotheses): load the released converged
`outputs/object` checkpoint (clean=1.0, swap=0.0, task=0.093 at step 0), keep the new `film_gate` params at
identity-init, fine-tune LoRA(r64, all-linear)+film for **2500 steps, batch 16, lr 1e-4** on
`libero_object_no_noops` RLDS (`scripts/train_warmstart.sh`, ~28 min on one H100). Verified byte-correct load
(998 tensors; only the 4 film_gate params new; 0 unexpected). Code: `gib-vla@0b866a1`.

**Eval** (`scripts/eval_compare.sh`, matched): baseline `outputs/object` vs H1 `--2500_chkpt` on the two
collapse dims **swap (position) + task** plus **clean** as guard, 10 tasks × 10 episodes = **100 ep/dim**.

## Results
| dim | base (`outputs/object`) | H1 lang-on (warm-start 2500) | Δ |
|---|---|---|---|
| clean | 1.00 (100/100) | 0.97 (97/100) | −0.03 |
| swap (position) | 0.00 (0/100) | 0.00 (0/100) | 0.00 |
| task | 0.10 (10/100) | 0.07 (7/100) | −0.03 |

H1 lang-on **differs** from baseline (clean 0.97≠1.00, task 0.07≠0.10) → the trained FiLM gate is wired and
active at inference; the null is *real*, not a "language-not-connected" artifact. Binomial SE at 100 ep ≈ 0.03,
so the clean/task deltas are within noise; swap is identical at the 0/100 floor.

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**NULL on the collapse dims** (mild-negative). Language-conditioning the *vision channel-gate* does not lift
position or task grounding: swap stays exactly floored at 0.00 (0/100), task and clean move only within noise
and slightly down. This matches the structural prediction — re-weighting which *vision channels* pass the IB
cannot create the **spatial object↔position binding** that `swap` requires; the gate has no spatial-reasoning
leverage. The conditioning is verified active, so more training of *this* mechanism is unlikely to rescue swap.
Decision: do **not** chase this with longer/higher-lr runs; the lang-zero control is moot (no positive effect to
attribute) and is deferred. The result directly motivates **H2** (a position-targeted auxiliary grounding head).

## Next directions
- [ ] **H2 — auxiliary grounding objective** (next): predict named-object presence+position from sim
      supervision → directly targets `swap`. Branch from this null. See [[2026-06-bekxnt-h0-stablevla-libero-pro-baseline]] for the baseline.
- [ ] (deferred, low priority) lang-zero control + longer/higher-lr H1 to bound the null vs training budget.
