---
id: bekxnt
slug: h0-stablevla-libero-pro-baseline
type: experiment
status: active
hypothesis: 'StableVLA''s corruption-robustness (vision-only FusedFAN gating) does NOT transfer to LIBERO-PRO grounding robustness: it should help the environment dimension but not object/position/semantic. Diagnose by running released StableVLA checkpoints (spatial/object/goal/long) across all 5 LIBERO-PRO dimensions + clean LIBERO + ImageNet-C corruption, broken down per-dimension, with VLA-Adapter as the same-arch-minus-FAN attribution control.

  '
parents: []
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/bekxnt-h0-stablevla-libero-pro-baseline
github-commit: e74390f9f9d4af5ec2ff140b8a2ccee6ca11492d
date-created: 2026-06-02
date-started: 2026-06-02
date-completed: null
tags: []
metrics:
  smoke_clean: 1.0
  smoke_object: 0.0
  smoke_swap: 0.0
  smoke_lan: 1.0
  smoke_task: 0.0
---

# bekxnt — h0-stablevla-libero-pro-baseline

## Hypothesis
StableVLA's corruption-robustness (vision-only FusedFAN gating) does NOT transfer to LIBERO-PRO grounding robustness: it should help the environment dimension but not object/position/semantic. Diagnose by running released StableVLA checkpoints (spatial/object/goal/long) across all 5 LIBERO-PRO dimensions + clean LIBERO + ImageNet-C corruption, broken down per-dimension, with VLA-Adapter as the same-arch-minus-FAN attribution control.

## Parents
(none — root)

## Method
Fresh isolated env `gibvla` (py3.10.16, torch2.2.0+cu121, flash-attn2.5.5, LIBERO+LIBERO-PRO); headless EGL render verified on H100. Wired LIBERO-PRO into `experiments/robot/libero/run_libero_eval.py`: 5 perturbation suites (object/swap/lan/task/env), `base_suite_of()` suffix-strip, `check_unnorm_key` remaps `<base>_<dim>` → base `_no_noops`. **Correctness fix:** for `lan`/`task` dims the instruction is read from the bddl `(:language)` block (task.language is filename-derived and would leak the *original* instruction). `scripts/run_libero_pro_battery.sh` runs dims in parallel + aggregates to JSON. H0 model = released StableVLA `object` checkpoint (`beikui12345/stablevla`) on `libero_object`. Train-free diagnostic.

## Results
**PRELIMINARY smoke** (task 0, 3 episodes/dim) with `object` checkpoint on `libero_object`:

| dim | meaning | SR |
|---|---|---|
| clean | — | **1.00** |
| object | visual identity | **0.00** |
| swap | position | **0.00** |
| lan | semantic/paraphrase | **1.00** |
| task | procedural goal | **0.00** |

Full per-dimension battery (10 tasks × 15 ep = 150/dim) RUNNING → final metrics recorded on completion.

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
Pipeline validated end-to-end (model + sim + EGL + success detection). Preliminary smoke already shows the thesis pattern: StableVLA holds **clean (1.0)** and **paraphrase/lan (1.0)** but collapses on **object (0.0)**, **position/swap (0.0)**, and **task (0.0)** — i.e. perceptual robustness does NOT transfer to grounding. Full battery in progress; status kept **active** until complete. Next: record full metrics, then `/exp-branch` H1 (language-conditioned gate).

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
