---
id: k2gwp2
slug: h6-visualservo-tta-gate
type: experiment
status: completed
date-completed: 2026-06-03
hypothesis: |
  VisualServo-TTA (sole survivor of the h6-bold-math workflow, the only idea that escapes the training-fit
  trap via TEST-TIME adaptation): at inference, read WHERE the named object is from the live frame and steer
  the action's bearing toward it. The corrective signal exists ONLY off-distribution (clean: object where
  memorized -> zero correction; swap: object moved -> correction), so clean is held by construction.
  Detectors are blocked on LIBERO sim (OWL-ViT too weak; OWLv2/GDINO crash in-env), so per the user the
  bearing comes from the in-model TargetLocalizer (real-transferable, no detector). GATE before building the
  full TTA: does the localizer's target_pred TRACK the moved object on swap, or memorize?
parents: ["[[2026-06-5wtdmq-h5-equivariance]]"]
links: ["[[2026-06-ih0sga-h2-grasp-anchored-grounding]]", "[[2026-06-j38ajw-h4-proprio-shortcut]]"]
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/k2gwp2-h6-visualservo-tta-gate
github-commit: caead13
date-created: 2026-06-03
date-started: 2026-06-03
tags: [position-swap, test-time-adaptation, localizer, generalization-gate, decisive-negative]
metrics: {tp_shift: 0.08, obj_shift: 0.23, cos_xy_task0: -0.99, cos_xy_task1: -0.81, cos_xy_task2: -0.52, cos_xy_task3: 0.02}
---

# k2gwp2 — h6-visualservo-tta-gate

## Hypothesis
See frontmatter.

## Parents
[[2026-06-5wtdmq-h5-equivariance]] — H5 null; the h6-bold-math workflow then produced VisualServo-TTA as the
only trap-escaping idea, gated on the localizer providing a position-invariant bearing.

## Method
Retrained the in-model `TargetLocalizer` (H2c machinery, grasp-target aux loss, warm-start 2500; grasp_loss
0.079→0.007, localizer trained weights verified merged into the eval checkpoint). Exposed `target_pred` in
the inference path (`_regression_or_discrete_prediction`). Probe (`scripts/probe_localizer_generalization.py`):
for matched clean vs swap scenes of the same task, render via the LIBERO env, read the localizer's
`target_pred`, and compare its shift to the object's true position shift (frame-offset-invariant).

## Results — the GATE
| task | ‖target_pred shift‖ | ‖object shift‖ | cos_xy(pred_shift, obj_shift) | verdict |
|---|---|---|---|---|
| 0 (alphabet soup) | 0.119 | 0.185 | **−0.99** | memorizes |
| 1 (cream cheese) | 0.034 | 0.244 | −0.81 | memorizes |
| 2 (salad dressing) | 0.119 | 0.253 | −0.52 | memorizes |
| 3 (bbq sauce) | 0.092 | 0.251 | +0.02 | memorizes |

The localizer's prediction barely moves (0.03–0.12) while objects move 0.18–0.25, and the direction is wrong
(cos ≤ 0). For task 0 (target = most-moved object) it predicted the **opposite** direction.

## Conclusion
**Decisive NEGATIVE, and the sharpest result of the program.** A head trained EXPLICITLY to localize the
object (grasp_loss→0.007 on train) **cannot localize the object at a new position** — the model's learned
visual features encode object position by MEMORIZATION, not by content. This is the bedrock reason every
in-model / data-free intervention (H1–H5) failed on swap: there is no internal, position-invariant
"where is the named object" signal to exploit. ⇒ Fixing swap **requires an EXTERNAL position-invariant
grounding** (open-vocab detector reading the live pixel) — which is exactly the real-transferable signal
that works on real robots, blocked here only by the sim domain-gap + this env's detector CUDA issues.
VisualServo-TTA is sound but **gated out** with the in-model bearing; it (and H6 rank-2 reframe / rank-3
detector-composite augmentation) all need the external detector that LIBERO-sim denies.

## Next directions
- [ ] To realize VisualServo-TTA: get an open-vocab detector working on LIBERO sim (compile GDINO kernel /
      pin transformers / fine-tune a detector), then the localizer bearing is replaced by the detector.
- [ ] Or validate the TTA MECHANISM with an oracle (sim object pose) bearing — de-risk only (sim-GT).
- [ ] Write-up: the negative arc (H0→H6) + the root finding "VLA visual features memorize object position;
      external open-vocab grounding is necessary for position-swap robustness".
