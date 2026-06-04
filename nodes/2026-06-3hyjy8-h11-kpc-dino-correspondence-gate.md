---
id: 3hyjy8
slug: h11-kpc-dino-correspondence-gate
type: experiment
status: active
hypothesis: |
  H11 KPC foundation gate (training-free). The novelty workflow's winner KPC (Kinesthetic Patch Correspondence) claims: the grasp event gives a per-object DINOv2 patch PROTOTYPE (proprioception supplies the grasp pixel), and a swapped object is localized from vision alone by frozen-feature cosine-NN to that prototype — detector-free, label-free, no training, dodging BOTH the sim open-vocab-detector wall AND H6's memorization (H6 measured a TRAINED pooled readout; raw DINOv2 patch correspondence is translation-covariant by architecture). H11 is the cheapest decisive gate BEFORE any build: does frozen DINO-SigLIP patch correspondence actually localize the named object across the clean->swap position change? Render matched clean+swap scenes; take the object's patch feature at its clean image location (object world pos projected via the sim camera — DIAGNOSTIC only, the method uses the kinesthetic grasp pixel) as prototype; on swap, cosine-match the prototype to all patches; check whether the top-match patch lands on the object's true swapped image location (top-1 accuracy / pixel error). GATE: if correspondence reliably tracks the moved object (top-1 on object, cos high) => KPC's grounding substrate is sound => build the full kinesthetic cut-paste + localizer. If it fails (matches the memorized location or a distractor) => raw DINO correspondence also memorizes/confuses => KPC dead, the GROUND wall is deeper than the readout.
parents: ["[[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/3hyjy8-h11-kpc-dino-correspondence-gate
github-commit: null
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: null
tags: []
metrics: {}
---

# 3hyjy8 — h11-kpc-dino-correspondence-gate

## Hypothesis
H11 KPC foundation gate (training-free). The novelty workflow's winner KPC (Kinesthetic Patch Correspondence) claims: the grasp event gives a per-object DINOv2 patch PROTOTYPE (proprioception supplies the grasp pixel), and a swapped object is localized from vision alone by frozen-feature cosine-NN to that prototype — detector-free, label-free, no training, dodging BOTH the sim open-vocab-detector wall AND H6's memorization (H6 measured a TRAINED pooled readout; raw DINOv2 patch correspondence is translation-covariant by architecture). H11 is the cheapest decisive gate BEFORE any build: does frozen DINO-SigLIP patch correspondence actually localize the named object across the clean->swap position change? Render matched clean+swap scenes; take the object's patch feature at its clean image location (object world pos projected via the sim camera — DIAGNOSTIC only, the method uses the kinesthetic grasp pixel) as prototype; on swap, cosine-match the prototype to all patches; check whether the top-match patch lands on the object's true swapped image location (top-1 accuracy / pixel error). GATE: if correspondence reliably tracks the moved object (top-1 on object, cos high) => KPC's grounding substrate is sound => build the full kinesthetic cut-paste + localizer. If it fails (matches the memorized location or a distractor) => raw DINO correspondence also memorizes/confuses => KPC dead, the GROUND wall is deeper than the readout.

## Parents
- [[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]

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
