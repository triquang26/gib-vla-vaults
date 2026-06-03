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
         └─ H5 (5wtdmq) translational equivariance ..... NULL (training-only; clean held)
      └─ H6 (k2gwp2) VisualServo-TTA gate .......... DECISIVE: in-model localizer MEMORIZES (can't localize moved object)
         └─ H7 (5i0j20) HARD residual coord injection . CONDITIONAL GREENLIGHT: 1st mechanism to MOVE swap; needs off-dist data
            └─ H8 (ocdaey) translation-equivariance aug . NULL: forces coord (loc_proj 2×) but swap unchanged; proprio-OOD
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
| **H5** | translational-equivariance loss on the action | 0.00 | 0.19 | 0.99 | NULL (fit-satisfiable → no transfer) |
| **H6** | in-model localizer generalization gate | — | — | — | DECISIVE: features MEMORIZE position |
| **H7** | HARD residual coord injection (oracle) | see↓ | — | — | **CONDITIONAL GREENLIGHT** |

\*partial-episode counts differ across runs; see each node's attachment for exact n.

**H7 reach-metric** (min xy-dist eef→true target; binary success uninformative under blank vision): blank=1.0
**clean 0.029 (reached) / swap 0.105 (drop 0.106)** — the hard residual coordinate channel is the **first
mechanism in H0→H7 to measurably move swap** (every prior left swap fully stuck). Proprio-memory masks the
rest; dropping proprio to force coord breaks closed-loop reach (clean 0.146). See the H7 finding below.

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

## Bedrock finding (H6 gate, the sharpest result)

A head trained EXPLICITLY to localize the named object (grasp_loss→0.007 on train) **cannot localize it at a
new position** on swap (prediction shift 0.03–0.12 vs object shift 0.18–0.25; direction cos ≤ 0). The model's
learned visual features encode object position by **memorization, not content** — so there is NO internal,
position-invariant "where is the object" signal for any in-model / data-free method to exploit. This is WHY
H1–H6 all null. ⇒ **Position-swap robustness requires an EXTERNAL position-invariant grounding** (open-vocab
detector on the live pixel) — the real-transferable signal that works on real robots, blocked here only by the
LIBERO sim-to-real detector gap.

## H7 finding (the first crack in swap) — hard residual coordinate injection

After H6, the workflow `swap-fix-design` adversarially vetted 5 method families (JEPA / slot / copy-paste /
detector / wildcard) against the actual code and killed all five soft-path designs on the **T3 hard-path
test** (position reaches the action only via attention K/V the head zeros — `action_heads.py:347/389-393`),
crowning a hybrid **JEPA-Localizer-HARD**: a content-honest localizer (EMA stop-grad target + arg-softmax over
a fixed grid, structurally unable to emit a memorized constant like H6's free regressor) feeding a coordinate
into the action through an **un-nullable residual-stream channel** (`output + x + loc_proj(coord)`, the
mechanical inverse of every soft path), trained on **off-distribution re-rendered** object positions.

H7 is the oracle gate that validated the *consumption mechanism* before building the localizer: inject a
correct coordinate (oracle `grasp_target`) via the residual channel, blank vision, and measure the
coordinate-driven reach (min xy-dist eef→true target; binary success is 0 under blank vision). Result:

- **The hard residual channel IS consumed** (loc_proj trains zero→nonzero; the eef measurably moves toward the
  injected coordinate). Under blank=1.0: **clean min-xy 0.029 (reached), swap 0.105 (drop 0.106)** — the eef
  is pulled ~halfway to the *swapped, off-distribution* object, and some swap episodes reach grasp range.
  **This is the first intervention in the entire program to move swap at all** (H1–H6 left it fully stuck).
- **But in-distribution training can't make coord OVERRIDE proprio-memory on swap.** Proprio (current-eef
  trajectory) and the coordinate only *disagree* off-distribution; on in-dist demos they agree, so no gradient
  teaches "trust coord over proprio." Removing proprio (dropout) to force coord **breaks closed-loop reaching**
  (clean min-xy 0.146 — no current-eef feedback). ⇒ **Catch-22:** proprio is needed for the closed-loop reach
  yet is the very shortcut that masks the coordinate on swap.

⇒ **The missing ingredient is OFF-DISTRIBUTION training data** (re-render the object at new positions +
supervise the action toward it): there proprio-memory is wrong and the coordinate is right, forcing
coord-over-proprio while keeping proprio for control. The residual injection (validated) + a learned
EMA-stop-grad localizer (real-transferable coord) + off-dist re-render is the path to a genuine swap win.
This also reconciles the H6 "external grounding required" finding: the external signal enters the *action*
through the residual channel, not a representation the head can ignore.

**H8 (ocdaey) — cheap data-free forcing is insufficient.** Tried to supply the off-distribution signal
WITHOUT a re-render via translation-equivariance augmentation: shift the injected coordinate AND the proprio
eef-xyz by the same random Δ, action unchanged (a delta-eef reach is translation-invariant). This forced the
head to commit harder to the coordinate (`loc_proj` std doubled 0.0033→0.0065) — but swap reach did *not*
improve (0.100 vs blank=1.0's 0.105) while clean degraded (0.060 vs 0.029). Cause: the shift pushes **proprio
off-manifold** (Δ=0.10 m in x ⇒ ±0.83 in normalized proprio-x, scale only 0.12), so train (shifted proprio)
≠ eval (real proprio) — a distribution mismatch that wrecks reach calibration. **Lesson:** a clean
off-distribution signal cannot be faked by perturbing proprio; it needs **real proprio at relocated object
positions** = an actual env re-render (object physically moved → in-range proprio + matching scene + a
scripted/expert or demo-geometry action). That re-render route + the validated residual coord channel + a
learned localizer is the remaining path to an actual swap win.

## (superseded) earlier frontier — H5 (mathematically-principled)

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
