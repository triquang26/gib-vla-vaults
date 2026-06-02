---
id: bekxnt
slug: h0-stablevla-libero-pro-baseline
type: experiment
status: completed
hypothesis: 'StableVLA''s corruption-robustness (vision-only FusedFAN gating) does NOT transfer to LIBERO-PRO grounding robustness: it should help the environment dimension but not object/position/semantic. Diagnose by running released StableVLA checkpoints (spatial/object/goal/long) across all 5 LIBERO-PRO dimensions + clean LIBERO + ImageNet-C corruption, broken down per-dimension, with VLA-Adapter as the same-arch-minus-FAN attribution control.

  '
parents: []
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/bekxnt-h0-stablevla-libero-pro-baseline
github-commit: e74390f9f9d4af5ec2ff140b8a2ccee6ca11492d
date-created: 2026-06-02
date-started: 2026-06-02
date-completed: 2026-06-02
tags: []
metrics:
  clean: 1.0
  object_visual: 0.9
  position_swap: 0.0
  semantic_lan: 1.0
  task_procedural: 0.093
  protocol: object-ckpt x libero_object, 10 tasks x 15 ep, seed 7, post-instruction-fix
  verified: trustworthy
---

# bekxnt — h0-stablevla-libero-pro-baseline

## Hypothesis
StableVLA's corruption-robustness (vision-only FusedFAN gating) does NOT transfer to LIBERO-PRO grounding robustness: it should help the environment dimension but not object/position/semantic. Diagnose by running released StableVLA checkpoints (spatial/object/goal/long) across all 5 LIBERO-PRO dimensions + clean LIBERO + ImageNet-C corruption, broken down per-dimension, with VLA-Adapter as the same-arch-minus-FAN attribution control.

## Parents
(none — root)

## Method
Fresh isolated env `gibvla` (py3.10.16, torch2.2.0+cu121, flash-attn2.5.5, LIBERO+LIBERO-PRO); headless EGL render verified on H100. Wired LIBERO-PRO into `experiments/robot/libero/run_libero_eval.py`: 5 perturbation suites (object/swap/lan/task/env), `base_suite_of()` suffix-strip, `check_unnorm_key` remaps `<base>_<dim>` → base `_no_noops`. **Correctness fix:** for `lan`/`task` dims the instruction is read from the bddl `(:language)` block (task.language is filename-derived and would leak the *original* instruction). `scripts/run_libero_pro_battery.sh` runs dims in parallel + aggregates to JSON. H0 model = released StableVLA `object` checkpoint (`beikui12345/stablevla`) on `libero_object`. Train-free diagnostic.

## Results
**Final H0 baseline** — released StableVLA `object` checkpoint on `libero_object`, 10 tasks × 15 ep = 150/dim (seed 7). See `attachments/bekxnt-h0-object-results.json`.

| dim | meaning | SR | succ/eps | note |
|---|---|---|---|---|
| clean | — | **1.00** | 150/150 | ceiling |
| object | visual identity | **0.90** | 135/150 | 9 tasks @1.0 + 1 (rescaled soup) @0.0 — recolor-robust, geometry-fragile |
| semantic (lan) | paraphrase | **1.00** | 150/150 | ceiling (model DID receive paraphrase) |
| task | procedural goal | **0.093** | 14/150 | rides on 1/10 tasks; post-instruction-fix |
| position (swap) | object relocated | **0.00** | 0/150 | total collapse, **zero confounds** |

**Adversarially verified** (6-agent workflow, `attachments/bekxnt-h0-verification-synthesis.md`): all dims `trustworthy=yes` — perturbations real (bddl+init-qpos diffs), model inputs correct (instruction-fix fires only for lan/task), failures genuine (run to max_steps=280, no crashes; swap 67-70s/it vs clean 40-49s).

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**GATE = GO.** Premise confirmed: StableVLA's perceptual robustness does NOT transfer to grounding. It is at **ceiling on clean, paraphrase, and object-recolor**, but **collapses on POSITION (swap=0.00, rock-solid) and TASK (0.093)**. The worst, cleanest failure is **spatial grounding** — the policy replays a memorized trajectory to the familiar target spot and grasps the wrong/empty object when the named object is relocated.

**Threats-to-validity (must address before cross-model claims):**
1. task/lan are **post-instruction-fix → NOT leaderboard-comparable** (leaderboard ~0.00 on task likely fed the original instruction → unwinnable; ours fixes that mismatch so 0.093 is correctly higher, not inflated).
2. **single seed (7), 15 ep/task** (vs official 50) → no CIs; violates the ≥3-seed rule for the final claim.
3. headline SRs **mask all-or-nothing per-task** (object & task each driven by 1 task); object geometry-robustness tested with N=1; lan ceiling-saturated.

**Queued controls:** (a) run **VLA-Adapter** through the identical harness (same-arch-minus-FAN attribution); (b) `task` "feed-original-instruction" variant to *measure* the protocol effect; (c) ≥3 seeds on swap/object/task; (d) per-task SR reporting.

**Opens up → next:** position is the worst + cleanest target. Lean **H2 (object presence+position grounding head, data-free, targets position)** as the first pilot; H1 (language-conditioned gate) targets task/semantic but task is fragile (1-task) and lan is ceiling. Decide via /exp-plan.

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
