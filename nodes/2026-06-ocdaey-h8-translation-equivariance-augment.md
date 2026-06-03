---
id: ocdaey
slug: h8-translation-equivariance-augment
type: experiment
status: active
hypothesis: |
  H8 OFF-DISTRIBUTION via TRANSLATION-equivariance augmentation (on top of H7's validated hard coord channel). H7 showed the residual coordinate channel measurably moves swap (~halfway, drop 0.106) but in-dist training lets the head MEMORIZE per-object absolute positions and lean on proprio, so it does not fully generalize. H8: per masked sample, translate the injected coordinate AND the proprio eef-xyz by the SAME random Δ, leaving the action UNCHANGED (a delta-eef reach is translation-invariant: the same deltas from eef+Δ land at object+Δ). This makes instruction X appear at many absolute coords (kills object->position memorization) and decorrelates proprio from trajectory phase (kills the proprio shortcut), so the only L1-fitting function is action=f(coord-proprio,language) — i.e. the translational EQUIVARIANCE the swap benchmark measures, enforced by construction, while preserving closed-loop geometry and the L1 target. Blank vision (the translated scene is not re-rendered). No action relabel, no unit conversion. Real-transferable. GATE (reach metric, blank eval): swap mean_minxy -> ~clean (0.03-0.05) vs blank=1.0 swap 0.105 => coord->reach GENERALIZES => GREENLIGHT building the JEPA localizer to supply coord from real pixels.
parents: ["[[2026-06-5i0j20-h7-coord-residual-injection]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/ocdaey-h8-translation-equivariance-augment
github-commit: null
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: null
tags: []
metrics: {}
---

# ocdaey — h8-translation-equivariance-augment

## Hypothesis
H8 OFF-DISTRIBUTION via TRANSLATION-equivariance augmentation (on top of H7's validated hard coord channel). H7 showed the residual coordinate channel measurably moves swap (~halfway, drop 0.106) but in-dist training lets the head MEMORIZE per-object absolute positions and lean on proprio, so it does not fully generalize. H8: per masked sample, translate the injected coordinate AND the proprio eef-xyz by the SAME random Δ, leaving the action UNCHANGED (a delta-eef reach is translation-invariant: the same deltas from eef+Δ land at object+Δ). This makes instruction X appear at many absolute coords (kills object->position memorization) and decorrelates proprio from trajectory phase (kills the proprio shortcut), so the only L1-fitting function is action=f(coord-proprio,language) — i.e. the translational EQUIVARIANCE the swap benchmark measures, enforced by construction, while preserving closed-loop geometry and the L1 target. Blank vision (the translated scene is not re-rendered). No action relabel, no unit conversion. Real-transferable. GATE (reach metric, blank eval): swap mean_minxy -> ~clean (0.03-0.05) vs blank=1.0 swap 0.105 => coord->reach GENERALIZES => GREENLIGHT building the JEPA localizer to supply coord from real pixels.

## Parents
- [[2026-06-5i0j20-h7-coord-residual-injection]]

## Method
<điền khi /exp-record>

## Results
<điền khi /exp-record>

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
<điền khi /exp-record>

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
