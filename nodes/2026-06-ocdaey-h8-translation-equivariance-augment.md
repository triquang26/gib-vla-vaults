---
id: ocdaey
slug: h8-translation-equivariance-augment
type: experiment
status: completed
hypothesis: |
  H8 OFF-DISTRIBUTION via TRANSLATION-equivariance augmentation (on top of H7's validated hard coord channel). H7 showed the residual coordinate channel measurably moves swap (~halfway, drop 0.106) but in-dist training lets the head MEMORIZE per-object absolute positions and lean on proprio, so it does not fully generalize. H8: per masked sample, translate the injected coordinate AND the proprio eef-xyz by the SAME random Δ, leaving the action UNCHANGED (a delta-eef reach is translation-invariant: the same deltas from eef+Δ land at object+Δ). This makes instruction X appear at many absolute coords (kills object->position memorization) and decorrelates proprio from trajectory phase (kills the proprio shortcut), so the only L1-fitting function is action=f(coord-proprio,language) — i.e. the translational EQUIVARIANCE the swap benchmark measures, enforced by construction, while preserving closed-loop geometry and the L1 target. Blank vision (the translated scene is not re-rendered). No action relabel, no unit conversion. Real-transferable. GATE (reach metric, blank eval): swap mean_minxy -> ~clean (0.03-0.05) vs blank=1.0 swap 0.105 => coord->reach GENERALIZES => GREENLIGHT building the JEPA localizer to supply coord from real pixels.
parents: ["[[2026-06-5i0j20-h7-coord-residual-injection]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/ocdaey-h8-translation-equivariance-augment
github-commit: c26e117
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: 2026-06-04
tags: [position-swap, translation-equivariance, augmentation, off-distribution, reach-metric, null-result, proprio-ood]
metrics: {reach_metric: min_xy_eef_to_target, eval_ep_per_dim: 50, clean_h8: 0.063, swap_h8: 0.126, task_h8: 0.065, clean_blank10: 0.029, swap_blank10: 0.105, loc_proj_std_h8: 0.0065, loc_proj_std_blank10: 0.0033}
---

# ocdaey — h8-translation-equivariance-augment

## Hypothesis
H8 OFF-DISTRIBUTION via TRANSLATION-equivariance augmentation (on top of H7's validated hard coord channel). H7 showed the residual coordinate channel measurably moves swap (~halfway, drop 0.106) but in-dist training lets the head MEMORIZE per-object absolute positions and lean on proprio, so it does not fully generalize. H8: per masked sample, translate the injected coordinate AND the proprio eef-xyz by the SAME random Δ, leaving the action UNCHANGED (a delta-eef reach is translation-invariant: the same deltas from eef+Δ land at object+Δ). This makes instruction X appear at many absolute coords (kills object->position memorization) and decorrelates proprio from trajectory phase (kills the proprio shortcut), so the only L1-fitting function is action=f(coord-proprio,language) — i.e. the translational EQUIVARIANCE the swap benchmark measures, enforced by construction, while preserving closed-loop geometry and the L1 target. Blank vision (the translated scene is not re-rendered). No action relabel, no unit conversion. Real-transferable. GATE (reach metric, blank eval): swap mean_minxy -> ~clean (0.03-0.05) vs blank=1.0 swap 0.105 => coord->reach GENERALIZES => GREENLIGHT building the JEPA localizer to supply coord from real pixels.

## Parents
- [[2026-06-5i0j20-h7-coord-residual-injection]]

## Method
Same as H7 blank=1.0 (`outputs/h7_blind1`) + the translation augmentation `GIBVLA_COORD_CF=0.7`: in
`run_forward_pass`, for ~70% of samples draw Δ ~ U(±0.10, ±0.10, ±0.04) m and add it to BOTH the injected
coordinate (`grasp_target`, raw) and the proprio eef-xyz (normalized, `Δ / proprio_scale`), leaving the
action **unchanged** (a delta-eef reach is translation-invariant). Blank vision (the translated scene is not
re-rendered); keep proprio. Warm-start 2500 steps, `libero_object_no_noops`. Eval = reach metric (min xy-dist
eef→true target) at 50 ep/dim, **blank vision + coord, proprio KEPT** (no proprio-zero). Code `gib-vla@c26e117`.

## Results
| dim | H8 min-xy | blank=1.0 | Δ |
|---|---|---|---|
| clean | 0.063 | 0.029 | **+0.034 (worse)** |
| swap | **0.126** | 0.105 | **+0.021 (worse)** |
| task | 0.065 | 0.066 | ~0 |
(50 ep/dim; binary success 0 under blank vision as expected; coord coverage 100%.)

- `loc_proj.2` std **doubled** (0.0033 → 0.0065): the augmentation DID force the head to commit much harder
  to the coordinate channel (it can no longer memorize a fixed position).
- **But swap reach got WORSE, not better** (0.126 vs 0.105), and clean degraded too (0.063 vs 0.029).

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**NULL / mildly NEGATIVE.** Forcing translational equivariance via blank-vision translation augmentation makes
the head rely more on the coordinate (loc_proj doubles) but does **not** make the coord→reach map generalize
to swap — it degrades both clean and swap. Diagnosed cause: the augmentation shifts the **proprio to
out-of-distribution absolute positions** during training (a Δ of 0.10 m in x maps to ±0.83 in normalized
proprio-x, since the proprio-x scale is only 0.12), while eval uses real in-range proprio — a train/eval
distribution mismatch that wrecks reach calibration. The translation trick cannot give a clean off-distribution
signal because it perturbs proprio off-manifold. ⇒ A genuine off-distribution signal needs **real proprio at
new object positions**, i.e. an actual env re-render (object physically relocated → real/in-range proprio +
matching scene), with the action from a scripted/expert reach or demo geometry. The H7 finding stands (hard
coord injection is the first mechanism to move swap, ~halfway); H8 shows the cheap data-free forcing is
insufficient and bounds the remaining work to the re-render route.

## Next directions
- [ ] Off-dist RE-RENDER variant: relocate the object in the LIBERO env (SwapPerturbator), keep real proprio,
      compute the reach action (scripted P-controller or demo-geometry), mix into warm-start. No proprio-OOD.
- [ ] Or bank the H7→H8 arc as a complete diagnostic (mechanism validated; cheap forcing insufficient).
