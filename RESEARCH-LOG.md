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
            ├─ H8 (ocdaey) translation-equivariance aug . NULL: forces coord (loc_proj 2×) but swap unchanged; proprio-OOD
            └─ H9 (5lsz9w) co-translation (2×2 Jacobian) ... NULL: hardest coord-commit (loc_proj 0.0086) but relabel too imprecise
               └─ H10 (s2s5hs) per-step Jacobian + wide Δ ... PARTIAL WIN: first REACH-COMMIT GENERALIZATION (swap 0.086 ≈ clean, best ever)
                  └─ H11 (3hyjy8) KPC DINO-correspondence gate . DEAD: features can't content-localize small objs on sim (bedrock wall)
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

**H9 (5lsz9w) — data-free action-relabel is too fragile (the REACH-COMMIT wall).** After H8 ruled out
perturbing proprio, H9 kept proprio REAL and instead CO-TRANSLATED the action: on a fraction of frames, shift
the injected coord by Δ and shift the action's lateral xy by `J@Δ`, where J is a full 2×2 image/world→action
Jacobian (the eef/action frame is ~90° rotated vs world — J off-diagonal −21.9 — so a diagonal gain gave a
broken negative-y; only the full 2×2 works). This forced the **hardest coordinate-commitment of any arm**
(`loc_proj` std 0.0033→0.0065→**0.0086**) — proving the *mechanism* (decorrelate coord from proprio with
in-range proprio) is sound — **but reach did NOT improve** (swap 0.134 vs H7 0.105; clean 0.040 vs 0.029).
Cause: the 8-step chunk is only a *partial* reach, so the cumulative-action↔displacement Jacobian is
phase-averaged and under-shoots; a constant 2×2 corrupts the coord→action map, which the hard-committed
`loc_proj` faithfully reproduces. **Verdict across H7→H9: the coordinate channel reliably moves swap HALFWAY
(H7 0.105 = best), but every data-free push that perturbs/relabels the ACTION (H8 proprio-shift, H9
action-relabel) is too fragile to beat it.** A clean off-distribution *action* label needs the env (forbidden)
or a precise per-step Jacobian from the proprio SEQUENCE (not the chunk). The GROUND gap (content localizer)
is untouched and orthogonal — H7's halfway-coord could still be paired with a real localizer.

**H10 (s2s5hs) — the per-step Jacobian breaks the REACH-COMMIT wall (first generalization).** H9's diagnosed
fix, executed: plumb a raw `eef_motion` (proprio[t+1]−proprio[t]) key, fit the **per-step controller gain
`J_inv`** from `(action[0], eef_motion)` instead of the cumulative-chunk/full-displacement regression. `J_inv`
converges **diagonal-dominant `[[65.6,−8.1],[1.6,76.4]]`, ~4× larger** than H9's `[[15.4,…],[−21.9,…]]` — its
diagonal form *proves H9's "~90° rotation" was a partial-reach + eef→object-geometry artifact*, and the 4×
magnitude shows H9 under-shifted the action 4×. With the correct gain + Δ widened to 0.18 m (covering the
0.18–0.25 m swap), **swap reach = 0.086 ≈ clean 0.074, stable across episodes** — the best swap of the program
(H7 0.105 → H9 0.134 → **H10 0.086**) and the **first arm where the coordinate→reach map genuinely generalizes
off-distribution** (clean≈swap, vs H9 clean-perfect/swap-fail). `loc_proj` std 0.0155 (strongest). The residual
gap is **precision** (~0.08 m vs grasp ~0.03 m), a coverage-vs-precision tradeoff from Δ=0.18/COTRANS=0.5 — NOT
a generalization failure → closable by tuning. **REACH-COMMIT is essentially solved data-free**; the remaining
work is precision-sharpening + the GROUND localizer (frozen-DINO-key, real-transferable) to supply the coord
from pixels. This is the path to an actual swap win with no env.

**H11 (3hyjy8) — the GROUND wall is fundamental on LIBERO sim (KPC dead).** A novelty workflow proposed
**Kinesthetic Patch Correspondence** (grasp-event gives a per-object DINOv2 patch prototype; localize the
swapped object by frozen-feature cosine-NN; detector-free, training-free). The training-free gate KILLED it:
patch correspondence (VLA backbone, raw DINOv2 @224 and @518) NEVER tracks the moved object (0/6, 0/8, 0/4) —
it locks onto the SAME grid position with cos≈1.00, because the small LIBERO objects are background-dominated
in every patch (a same-position patch is cos 1.00 identical across clean↔swap; `cosTrue`≈0.3, the object
barely registers) and the objects are same-category. The workflow ALSO corrected a key over-claim: re-checking
eval_logs, **binary grasp success is 0/50 even with a PERFECT sim-oracle coordinate** (clean too) under the
blank-vision reach regime — so H7–H10's 0.03–0.09 "reach" is only a min-distance proxy, never a grasp.
**Net bedrock (re-confirmed at the feature level):** on LIBERO sim the position-swap GROUND signal is NOT
recoverable from pixels — no open-vocab detector (crash / conf 0.002, sim domain gap) and no feature
correspondence (objects too small/similar). REACH-COMMIT is solved data-free (H10, novel) but the swap
binary win is unvalidatable end-to-end on sim; it is real-transferable (works where a detector works).

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
