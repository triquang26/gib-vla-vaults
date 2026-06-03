# gib-vla — Research Log (full detail)

**Goal.** Improve a small (~0.5B) VLA (StableVLA: Qwen2.5-0.5B + DINO-SigLIP + "FusedFAN" projector,
L1RegressionActionHead) on the **LIBERO-PRO** grounding/compositional-robustness benchmark, **without extra
teleoperation data**, with maintainable + reproducible infra. Eval focus = the 2 collapse dims **position-
swap + task**, with **clean LIBERO** as a guard.

**Method discipline.** All hypotheses **warm-start** from the released `outputs/object` checkpoint
(~28 min/arm on 1×H100, identity-at-init so clean is preserved), changing **one** thing for clean
attribution; eval matched at 100 ep/dim via `scripts/eval_compare.sh`. Supervision must be **real-
transferable** (works on real robot data later — proprio/detector/VLM OK; sim seg/pose/re-render NOT).
Code repo `triquang26/gib-vla` (private), vault `triquang26/gib-vla-vaults` (public).

---

## The experiment ladder (DAG)

```
H0 (bekxnt) baseline/diagnosis
└─ H1 (0jw4rh) language-conditioned IB gate ........... NULL on swap
   └─ H2 (ih0sga) grasp-anchored grounding
      ├─ H2a aux localization head ................... NULL on swap
      ├─ H2b + counterfactual rebinding .............. NULL on swap
      └─ H2c in-model action_queries modulation ...... coded, superseded
      └─ H4 (j38ajw) proprio-xyz dropout ............. NULL  → proprio RULED OUT
         └─ H5 (math) equivariance / conditional-MI ... ← current frontier
```

## Results so far (swap / task / clean, vs base = released checkpoint)

| H | idea | swap | task | clean | verdict |
|---|---|---|---|---|---|
| **H0** | baseline diagnosis | 0.00 | 0.093 | 1.00 | swap+task are the collapse dims |
| **H1** | FiLM the vision info-bottleneck gate on instruction | 0.00 | 0.07 | 0.97 | NULL |
| **H2a** | aux head: instruction×patches → predict object 3D loc (grasp-proprio label) | 0.00 | 0.09 | 0.94 | NULL (head learns, action ignores) |
| **H2b** | H2a + counterfactual instruction-rebind | 0.00 | 0.24* | 0.95 | NULL (grasp_loss↓ but swap floored) |
| **H2c** | in-model localizer modulates action_queries | — | — | — | coded; superseded (same failure class) |
| **H4-PROP** | drop eef-xyz proprio (kill the "where am I" shortcut) | 0.00 | 0.27 | 0.96 | NULL → **proprio ruled out** |

\*partial-episode counts differ across runs; see each node's attachment for exact n.

## Key findings (what we now KNOW)

1. **Representation-grounding cannot fix swap.** H1 (IB gate), H2a/H2b (aux head, ±counterfactual), and the
   H2c modulation all inject grounding into a *representation* the action head is free to ignore. All NULL.
2. **The grasp-proprioception label is good and the head learns it** (grasp_loss 0.031→0.012) — localization
   is not the bottleneck; *using* it in the action is.
3. **Proprio is NOT the shortcut** (H4-PROP): dropping eef-xyz neither moved swap nor hurt clean (0.96).
4. **Root cause (information-theoretic).** The action head decodes the delta-action chunk where
   `I(action ; object_location | proprio, task) ≈ 0` — position/scene is a *sufficient statistic* for the L1
   objective, so no gradient flows to grounding. **The fix must change the ACTION OBJECTIVE so that reading
   the named object's current location becomes NECESSARY to minimize the loss.**

## Current frontier — H5 (mathematically-principled)

Stop adding signals beside a sufficient statistic; instead encode the **symmetry the swap benchmark
measures**. Candidates under design (workflow `h5-math-design`):
- **Equivariance:** `a(scene, obj@p+Δ) = a(scene, obj@p) ⊕ Δ` enforced on existing same-object/diff-position
  demos (data-free). This *is* the definition of position-robustness.
- **Conditional-MI / InfoNCE at the action:** maximize `I(action ; visual obj-location | proprio)` so the
  action must discriminate the true object location from negatives.
- IRM-causal (position = spurious environment), object-centric canonicalization, hard-pair mining.
The workflow vets each for math-soundness (degenerate minimizers), data-free implementability, and whether
it genuinely reaches the action; top-2 get an implementation spec → implement → train → eval → record.

## Reproducibility pointers

- Code branches (private repo): `exp/<id>-<slug>` per node. Scripts: `scripts/train_warmstart.sh`,
  `scripts/eval_compare.sh`, `scripts/verify_warmstart.py`. Env: `docs/ENVIRONMENT.md`.
- Attachments (this vault): per-node `*-compare.json` (raw SRs), `novelty-research-*.json` (prior art),
  `h2-rehypothesize-*.json` (root-cause analysis + H4 ladder).
- Prior-art boundaries (do not reinvent): ReconVLA, ObjectVLA, RoboPoint, LIBERO-Plus — see H2 node.
