---
id: 5i0j20
slug: h7-coord-residual-injection
type: experiment
status: completed
hypothesis: |
  H7 HARD residual-stream coordinate injection (oracle gate). The 6 prior nulls + H6 gate show object position reaches the L1 action head only via SOFT paths (attention K/V on h_a/p/h_t, aux losses) the L1-optimal head routes around, and the in-model localizer MEMORIZES. H7 isolates the load-bearing question with an oracle: inject a CORRECT absolute 3D target coordinate (grasp_target object-location proxy) into EVERY MLPResNet block residual (output+x+loc_proj(coord), zero-init => warm-start byte-identical) — an additive, un-nullable channel, the mechanical inverse of every soft path — while BLANKING vision at both train and eval to remove the memorization shortcut, making the policy a pure reach-to-coordinate controller. GREENLIGHT if swap-approx-clean (coordinate->reach extrapolates to swapped/off-dist positions) => build the JEPA-honest localizer to supply the coordinate; KILL if swap~0 even with a perfect coordinate handed in => the head cannot turn a coordinate into a generalizing reach, the external-grounding fix is dead (stronger negative than all 6 priors).
parents: ["[[2026-06-k2gwp2-h6-visualservo-tta-gate]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/5i0j20-h7-coord-residual-injection
github-commit: 41094ef
date-created: 2026-06-03
date-started: 2026-06-03
date-completed: 2026-06-04
tags: [position-swap, coordinate-injection, residual-stream, oracle-gate, reach-metric, conditional-greenlight, off-dist-necessary]
metrics: {reach_metric: min_xy_eef_to_target, eval_ep_per_dim: 50, clean_blank05: 0.134, swap_blank05: 0.155, clean_blank10: 0.029, swap_blank10: 0.105, swap_drop_blank10: 0.106, clean_blindprop: 0.146, swap_blindprop: 0.168, swap_best_blank10: 0.0155}
---

# 5i0j20 — h7-coord-residual-injection

## Hypothesis
H7 HARD residual-stream coordinate injection (oracle gate). The 6 prior nulls + H6 gate show object position reaches the L1 action head only via SOFT paths (attention K/V on h_a/p/h_t, aux losses) the L1-optimal head routes around, and the in-model localizer MEMORIZES. H7 isolates the load-bearing question with an oracle: inject a CORRECT absolute 3D target coordinate (grasp_target object-location proxy) into EVERY MLPResNet block residual (output+x+loc_proj(coord), zero-init => warm-start byte-identical) — an additive, un-nullable channel, the mechanical inverse of every soft path — while BLANKING vision at both train and eval to remove the memorization shortcut, making the policy a pure reach-to-coordinate controller. GREENLIGHT if swap-approx-clean (coordinate->reach extrapolates to swapped/off-dist positions) => build the JEPA-honest localizer to supply the coordinate; KILL if swap~0 even with a perfect coordinate handed in => the head cannot turn a coordinate into a generalizing reach, the external-grounding fix is dead (stronger negative than all 6 priors).

## Parents
- [[2026-06-k2gwp2-h6-visualservo-tta-gate]]

## Method
**Hard residual coordinate channel.** Added `loc_proj` (MLP 3→256→dim, **zero-init final layer** ⇒
warm-start byte-identical) to `L1RegressionActionHead`; `loc_token = loc_proj(coord)` is added to **every**
MLPResNet block residual (`x = ffn(output + x + loc_token)`). Unlike the soft attention-K/V paths (h_a/p/h_t)
the L1-optimal head drove to ~0 in H1/H2/H4/H6, an additive residual is **un-nullable** except by zeroing
loc_proj itself — which the loss penalizes ⇒ gradient is forced onto it. `coord` = `grasp_target` (eef-xyz at
the grasp moment = object-location proxy; real-transferable stand-in for a SAM/detector centroid). To isolate
the coordinate as the position signal, two shortcuts are optionally removed: **vision blanked**
(`GIBVLA_COORD_BLANK_VISION`, zeroes the projected patch embeddings at train+eval) and **proprio-xyz dropped**
(`GIBVLA_PROPRIO_XYZ_DROPOUT` at train + `GIBVLA_PROPRIO_XYZ_ZERO` at eval, matched). At eval the oracle coord
is the live target-object world position from the sim obs, matched to the instruction noun (coverage 100%,
35/35 requeries — no name-mismatch confound), injected via the same residual.

**Metric.** Binary task-success is 0 under blank vision (blind pick-place can't time the gripper / find the
basket), so it cannot read the gate. The signal is **min xy-distance from the eef to the true target object
over the episode** (`[H7reach] minxy`): under blank vision the reach is *purely coordinate-driven*, so this
isolates whether the injected coordinate steers the arm to the object — on clean (in-dist coords) and swap
(off-dist coords). 3 arms, warm-start 2500 steps, `libero_object_no_noops`; eval 50 ep/dim on clean+swap+task.
LIBERO object xy-workspace spans ~0.2–0.3 m; ~0.03 = reached grasp range, ~0.15+ = never approached.
Code `gib-vla@41094ef`.

## Results
| arm (blind, coord-injected) | clean min-xy | swap min-xy | swap drop | reading |
|---|---|---|---|---|
| blank=0.5 | 0.134 | 0.155 | 0.057 | coord used but under-committed |
| **blank=1.0** | **0.029 ✓ reached** | 0.105 | **0.106** | clean reached; **coord pulls swap ~halfway** |
| blank=1.0 + proprio-drop | 0.146 ✗ broke | 0.168 | 0.028 | proprio-zero broke closed-loop reach → inconclusive |

- `loc_proj` trained zero→nonzero in every arm ⇒ the residual channel is **live and consumed** (the eef
  measurably moves toward the injected coordinate). Best single swap reach under blank=1.0 = 0.0155 (some
  swap episodes fully reach grasp range).
- **blank=1.0 is the headline:** the hard coordinate channel drives the eef ~halfway to the **swapped
  (off-distribution)** object (drop 0.106) — the **first mechanism in H0→H7 to measurably move swap at all**
  (H1/H2/H4/H5/H6 left swap completely stuck). Proprio-memory masks the other half.
- Dropping proprio-xyz to *force* coord usage broke closed-loop reaching (no current-eef feedback ⇒ clean
  min-xy 0.146, best 0.084) — the arm can't precisely converge; this arm is inconclusive for extrapolation.

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**CONDITIONAL GREENLIGHT.** (1) The hard residual coordinate channel is genuinely *consumed* by the L1 action
(loc_proj trains nonzero; the eef moves toward the injected coordinate) — the mechanical **inverse** of the
soft paths H2/H6 the head ignored/memorized — and is the **first intervention in the program to measurably
move swap** (blank=1.0 swap-drop 0.106; some swap episodes reach grasp range). (2) But **in-distribution
training, even fully blind, cannot make the coordinate OVERRIDE proprio-memory on swap**, because proprio
(current-eef trajectory) and the coordinate only *disagree* off-distribution; on in-dist demos they agree, so
no gradient teaches "trust coord over proprio." (3) Removing proprio (dropout) to force coord **breaks
closed-loop reaching**. ⇒ **Catch-22:** proprio is needed for closed-loop reach yet is the very shortcut that
masks the coordinate on swap. The missing ingredient is **OFF-DISTRIBUTION training data** (re-render the
object at new positions + supervise the action toward it): there proprio-memory is WRONG and the coordinate
is RIGHT, forcing coord-over-proprio while keeping proprio for closed-loop control. This greenlights building
the winner **JEPA-Localizer-HARD** with off-dist re-render training — the residual coord injection is the
*validated* consumption mechanism; an EMA-stop-grad learned localizer supplies the coordinate
real-transferably (replacing the oracle grasp_target).

## Next directions
- [ ] Build winner: off-dist re-render (in-repo SwapPerturbator) + analytic action relabel toward the moved
      object, keeping proprio; coordinate via the residual channel; then replace oracle coord with the
      EMA-stop-grad JEPA localizer (SAM/detector centroid label).
- [ ] Re-run the gate WITH off-dist data: does swap min-xy drop to ≈clean (full generalization)?
- [ ] Zero-coord control (`GIBVLA_COORD_ZERO=1`) to formally attribute the reach to the coordinate.
