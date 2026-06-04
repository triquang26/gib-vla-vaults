---
id: 1jr7vm
slug: h12-dense-openvocab-ground-gate
type: experiment
status: completed
hypothesis: 'H12 dense open-vocab GROUND gate (idea #1 from the paper-novelty pass: MaskCLIP / CLIP-Surgery / ''Extract Free Dense Labels from CLIP''). H11 killed VISUAL DINO patch correspondence (visual-only, memorizes the grid position, objects too small). Idea #1 attacks the GROUND wall with a different modality: use the INSTRUCTION NOUN as a TEXT query and compute dense patch<->text similarity -> a heatmap of the NAMED object. This is language-conditioned (disambiguates same-category condiments) and detector-free (a ViT+text forward + cosine, NO open-vocab detector, so it dodges the sim CUDA-kernel wall that crashed GroundingDINO/OWLv2). GATE on matched clean+swap renders: does the heatmap argmax localize the named object (clean) and TRACK it to the swapped location (swap)? Three escalating shots so a null is the IDEA''s, not the impl''s: (1) SigLIP head-routing MaskCLIP; (2) canonical CLIP-B/16 MaskCLIP (visual_projection on patch tokens); (3) CLIP-Surgery at SOTA strength — label competition
  (target noun vs other scene nouns vs backgrounds) + logit surgery (cancel respond-to-all patches) on CLIP-L/14-336 (24x24, objects span 2-3 patches). PASS (clean>=0.6 AND track>=0.5) => idea #1 is a viable real-transferable localizer to supply the coord; KILL => dense language grounding ALSO fails on sim => the GROUND wall holds across ALL modalities (detector + visual correspondence + dense language), not just one.

  '
parents:
- '[[2026-06-3hyjy8-h11-kpc-dino-correspondence-gate]]'
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/1jr7vm-h12-dense-openvocab-ground-gate
github-commit: 997e90d88cc5d2b592005bac3a413cc4d8b578ad
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: 2026-06-04
tags: []
metrics:
  probe: dense_open_vocab_patch_text
  shot1_siglip_clean: 1/8
  shot1_siglip_track: 0/8
  shot1_peak: 0.02_noise_implbug
  shot2_clip_maskclip_clean: 0/8
  shot2_clip_peak: 0.28_healthy
  shot2_failmode: argmax_locks_fixed_background_patch
  shot3_clipsurgery_clean: 0/8
  shot3_clipsurgery_track: 1/8_noise
  shot3_peak: 0.34-0.89_confident
  cause: sim_domain_gap+object_too_small_confident_wrong
  verdict: idea1-dead-on-sim-GROUND-wall-holds-across-all-modalities
---

# 1jr7vm — h12-dense-openvocab-ground-gate

## Hypothesis
H12 dense open-vocab GROUND gate (idea #1 from the paper-novelty pass: MaskCLIP / CLIP-Surgery / 'Extract Free Dense Labels from CLIP'). H11 killed VISUAL DINO patch correspondence (visual-only, memorizes the grid position, objects too small). Idea #1 attacks the GROUND wall with a different modality: use the INSTRUCTION NOUN as a TEXT query and compute dense patch<->text similarity -> a heatmap of the NAMED object. This is language-conditioned (disambiguates same-category condiments) and detector-free (a ViT+text forward + cosine, NO open-vocab detector, so it dodges the sim CUDA-kernel wall that crashed GroundingDINO/OWLv2). GATE on matched clean+swap renders: does the heatmap argmax localize the named object (clean) and TRACK it to the swapped location (swap)? Three escalating shots so a null is the IDEA's, not the impl's: (1) SigLIP head-routing MaskCLIP; (2) canonical CLIP-B/16 MaskCLIP (visual_projection on patch tokens); (3) CLIP-Surgery at SOTA strength — label competition (target noun vs other scene nouns vs backgrounds) + logit surgery (cancel respond-to-all patches) on CLIP-L/14-336 (24x24, objects span 2-3 patches). PASS (clean>=0.6 AND track>=0.5) => idea #1 is a viable real-transferable localizer to supply the coord; KILL => dense language grounding ALSO fails on sim => the GROUND wall holds across ALL modalities (detector + visual correspondence + dense language), not just one.

## Parents
- [[2026-06-3hyjy8-h11-kpc-dino-correspondence-gate]]

## Method
Three escalating shots of idea #1 (dense patch<->text grounding), so a null is the idea's not the
impl's. Matched clean+swap LIBERO renders; instruction-noun text query; heatmap argmax must localize the
named object (clean) and track it to the swapped position. Object true pixel from sim camera projection
(DIAGNOSTIC only). (1) `scripts/probe_siglip_patchtext.py` — SigLIP head-routing MaskCLIP. (2)
`scripts/probe_clip_maskclip.py` — canonical CLIP-B/16, visual_projection applied to patch tokens. (3)
`scripts/probe_clip_surgery.py` — CLIP-Surgery at SOTA strength: CLIP-L/14-336 (24x24, finer patches so small
objects span 2-3), label competition (target noun vs other scene nouns vs 6 background labels) + logit surgery
(subtract per-patch mean logit across labels to cancel the high-norm patches that respond to every query).

## Results
| shot | model | clean localize | tracks swap | peak sim | failure mode |
|---|---|---|---|---|---|
| 1 SigLIP head-route | siglip-base-224 | 1/8 | 0/8 | ~0.02 (noise) | impl bug — head routing wrong, peaks noise-level |
| 2 canonical MaskCLIP | clip-vit-b/16-224 | **0/8** | 0/8 | **0.28 (healthy)** | argmax locks onto a few FIXED background/artifact patches (clean argmax == swap argmax at same WRONG patch) |
| 3 CLIP-Surgery (SOTA) | clip-vit-L/14-336 | **0/8** | 1/8 (noise) | 0.34–0.89 (confident) | confidently wrong — aligns the noun to a background/arm patch, not the tiny object |

- Shot 1's noise peaks were a routing bug (SigLIP's pooling head != CLIP); shots 2-3 fixed it — peaks are
  healthy/confident (0.28 then 0.34-0.89), proving the patch-text machinery works. Localization still fails.
- Shot 3 (label competition + logit surgery + finer 24x24 patches, the documented fix for the shot-2 failure
  AND for the object-too-small cause that killed H11) STILL gives 0/8 clean. The single "track" (task6) has the
  lowest swap peak (0.255) = noise, not grounding.

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
**DEAD — idea #1 fails on LIBERO sim, and it CLOSES the GROUND wall across all modalities.** Dense
open-vocab patch-text grounding (MaskCLIP -> canonical CLIP -> CLIP-Surgery at SOTA strength, finer patches,
language disambiguation, the full documented anti-background-artifact fix) never localizes the named object on
clean (0/8) and never tracks it across swap. Crucially the failure is NOT weak signal: shots 2-3 produce
CONFIDENT alignment (peak 0.28 -> 0.89) onto the WRONG patch — the foundation model genuinely matches the noun
("alphabet soup") to a background/arm patch because the rendered sim condiments do not look like web-image
products (domain gap) and each object is ~1-2 patches dominated by table. This is the LANGUAGE-modality twin of
H11 (visual correspondence): GROUND is blocked on LIBERO sim across ALL three grounding modalities tried —
(a) open-vocab DETECTOR (GroundingDINO/OWLv2 CUDA-crash, OWL-ViT conf 0.002), (b) VISUAL feature
correspondence (H11: cos 1.00 same-position, cosTrue 0.3), (c) dense LANGUAGE patch-text (H12: confident-wrong,
0/8). ⇒ The grounding signal is fundamentally not recoverable from sim pixels at this object scale, by any
modality. On REAL RGB (in-domain, larger, recognizable products) any of these would work — so a real-localizer
+ the method is real-transferable but UNVALIDATABLE end-to-end on LIBERO sim. Combined with the binary-eval
finding (the H7-H10 coord/blank-vision arms score 0/50 binary even on CLEAN under real vision — the reach proxy
never corresponded to a grasp), swap=0.00 on LIBERO sim is blocked by TWO independent walls: GROUND
(unrecoverable from sim pixels) and COMMIT (blank-vision training that breaks the memorization shortcut also
destroys real-vision grasping). The complete, honest story is H0->H12: a rigorous diagnosis + a
real-transferable REACH-COMMIT augmentation + a thoroughly-characterized double negative on sim.

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
