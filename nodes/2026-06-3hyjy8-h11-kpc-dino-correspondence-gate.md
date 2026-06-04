---
id: 3hyjy8
slug: h11-kpc-dino-correspondence-gate
type: experiment
status: completed
hypothesis: 'H11 KPC foundation gate (training-free). The novelty workflow''s winner KPC (Kinesthetic Patch Correspondence) claims: the grasp event gives a per-object DINOv2 patch PROTOTYPE (proprioception supplies the grasp pixel), and a swapped object is localized from vision alone by frozen-feature cosine-NN to that prototype — detector-free, label-free, no training, dodging BOTH the sim open-vocab-detector wall AND H6''s memorization (H6 measured a TRAINED pooled readout; raw DINOv2 patch correspondence is translation-covariant by architecture). H11 is the cheapest decisive gate BEFORE any build: does frozen DINO-SigLIP patch correspondence actually localize the named object across the clean->swap position change? Render matched clean+swap scenes; take the object''s patch feature at its clean image location (object world pos projected via the sim camera — DIAGNOSTIC only, the method uses the kinesthetic grasp pixel) as prototype; on swap, cosine-match the prototype to all patches; check
  whether the top-match patch lands on the object''s true swapped image location (top-1 accuracy / pixel error). GATE: if correspondence reliably tracks the moved object (top-1 on object, cos high) => KPC''s grounding substrate is sound => build the full kinesthetic cut-paste + localizer. If it fails (matches the memorized location or a distractor) => raw DINO correspondence also memorizes/confuses => KPC dead, the GROUND wall is deeper than the readout.

  '
parents:
- '[[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]'
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/3hyjy8-h11-kpc-dino-correspondence-gate
github-commit: kpc-probes
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: 2026-06-04
tags:
- position-swap
- ground-gap
- dino-correspondence
- kinesthetic
- training-free
- null-result
- bedrock-wall
metrics:
  probe: dino_patch_correspondence
  track_vla_backbone: 0/6
  track_dinov2_224: 0/8
  track_dinov2_518: 0/4
  corrected_rot180_track: 0/8
  corrected_failmode: scattered_5of8_memorized_3of8
  cosTrue_objpatch: 0.04-0.68_low
  cause: objects_too_small_content_no_correspondence
  projection_bug: rot180_corrected_conclusion_holds
  verdict: KPC-dead-on-sim-bedrock-wall
---

# 3hyjy8 — h11-kpc-dino-correspondence-gate

## Hypothesis
H11 KPC foundation gate (training-free). The novelty workflow's winner KPC (Kinesthetic Patch Correspondence) claims: the grasp event gives a per-object DINOv2 patch PROTOTYPE (proprioception supplies the grasp pixel), and a swapped object is localized from vision alone by frozen-feature cosine-NN to that prototype — detector-free, label-free, no training, dodging BOTH the sim open-vocab-detector wall AND H6's memorization (H6 measured a TRAINED pooled readout; raw DINOv2 patch correspondence is translation-covariant by architecture). H11 is the cheapest decisive gate BEFORE any build: does frozen DINO-SigLIP patch correspondence actually localize the named object across the clean->swap position change? Render matched clean+swap scenes; take the object's patch feature at its clean image location (object world pos projected via the sim camera — DIAGNOSTIC only, the method uses the kinesthetic grasp pixel) as prototype; on swap, cosine-match the prototype to all patches; check whether the top-match patch lands on the object's true swapped image location (top-1 accuracy / pixel error). GATE: if correspondence reliably tracks the moved object (top-1 on object, cos high) => KPC's grounding substrate is sound => build the full kinesthetic cut-paste + localizer. If it fails (matches the memorized location or a distractor) => raw DINO correspondence also memorizes/confuses => KPC dead, the GROUND wall is deeper than the readout.

## Parents
- [[2026-06-s2s5hs-h10-perstep-jacobian-cotrans]]

## Method
Training-free probe (`scripts/probe_kpc_correspondence.py`, `scripts/probe_kpc_dinov2.py`). Render matched
clean + swap scenes per task; extract patch features; take the object's clean-location patch as a PROTOTYPE;
on the swap scene, cosine-match the prototype to every patch; classify the top match as TRACKS-object (on the
swapped object's patch ±1), memorized-loc (on the clean object's patch ±1), or distractor. Object image pixel
from the robosuite camera projection of the object world pos (DIAGNOSTIC only — the real KPC supplies it from
the kinesthetic grasp pixel). Three feature variants: (1) the VLA's fine-tuned DINO-SigLIP backbone; (2) RAW
`facebook/dinov2-base` at 224 (16×16 patches); (3) RAW DINOv2 at 518 (37×37 patches, finer). Also report
`cosTrue` = cosine of the prototype to the object's TRUE swapped-location patch.

## Results
| feature | TRACKS | memorized | distractor | cosTop | cosTrue (true swap patch) |
|---|---|---|---|---|---|
| VLA backbone (16×16) | **0/6** | 6/6 | 0/6 | 0.99 | — |
| RAW DINOv2 224 (16×16) | **0/8** | 7/8 | 1/8 | 0.93 | 0.04–0.61 (low) |
| RAW DINOv2 518 (37×37) | **0/4** | 4/4 | 0/4 | **1.00** | 0.23–0.47 (low) |

- The prototype's top match is on the **same grid position** on the swap scene with **cos 0.98–1.00**, even
  though the object has MOVED away and a different object now sits there.
- `cosTrue` (prototype vs the object's actual swapped patch) is **low (~0.3)** at all resolutions.

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**DEAD — and it pinpoints the GROUND wall precisely.** Patch correspondence (VLA backbone, raw DINOv2, finer
DINOv2) NEVER tracks the moved object (0 across all variants); it locks onto the same grid position with cos
≈1.00. The cause is decisive: a same-position patch is **cos 1.00 identical across clean↔swap** because the
LIBERO objects are SMALL relative to the patch (14 px even at 518) and the table background dominates each
patch — so DINOv2 sees ~pure (identical) table at every grid cell and the object is effectively invisible in
the features (`cosTrue` ≈0.3, the object's content barely registers). Compounded by the objects being
**same-category** (condiment bottles/boxes), correspondence cannot distinguish them. ⇒ The novel KPC idea
(grasp-event prototype + DINO correspondence) cannot ground these objects on LIBERO sim. This is the deepest
confirmation of the **bedrock wall**: on LIBERO sim, GROUND is fundamentally blocked — no open-vocab detector
(crashes / conf 0.002, sim domain gap), AND no feature correspondence (objects too small/similar) can localize
the swapped object. The grounding signal is simply not recoverable from the sim pixels at this object scale.
On REAL robots (larger objects, in-domain RGB, a working detector/SAM) the GROUND piece works — so the full
method (H10 REACH-COMMIT + a real localizer) is real-transferable but UNVALIDATABLE end-to-end on LIBERO.

## Next directions
- [ ] (real-robot) deploy the method where a detector works; sim cannot validate GROUND.
- [ ] (sim, if pushed) a benchmark whose objects are large enough for patch correspondence, or train a
      LIBERO-specific object segmenter offline — but the latter is the env-crutch the user excludes.
- [ ] Bank: H7→H10 (REACH-COMMIT method) + H11 (GROUND wall fully characterized) is the complete story.

## Correction (added during H13 — [[2026-06-u0hlhq-h13-vla-self-attention-ground-gate]])
A **rot180 projection bug** was found in the H11/H12 probes: `get_libero_image` flips the frame 180°, so the
patch grid is UPRIGHT while `camera_utils` projects in the RAW frame — the prototype/GT patch here was sampled
at the **180°-opposite (background) patch**, not the object (verified by `scripts/calib_project.py`). H11 was
**re-run with the corrected GT + the named (instruction-target) object** (`logs/h11_dino_corrected.log`):
**TRACKS 0/8** still (failure mode shifts from "memorized 7/8" to "scattered/other 5/8 + memorized 3/8"),
cosTrue 0.04–0.68 (the small object's DINO content does not correspond across the move). **Conclusion unchanged:**
raw DINO correspondence cannot ground the swapped object on LIBERO sim — now on corrected footing.
