---
id: u0hlhq
slug: h13-vla-self-attention-ground-gate
type: experiment
status: completed
hypothesis: 'H13 VLA SELF-ATTENTION GROUND gate — the last untested grounding modality, the one that addresses BOTH prior failure causes at once. H11 (raw DINO correspondence) failed for being visual-only; H12 (off-the-shelf CLIP patch-text) failed for the sim domain gap. The fine-tuned VLA''s OWN LLM self-attention from the instruction-noun tokens to the vision-patch tokens is BOTH in-domain (no CLIP gap) AND language-conditioned (disambiguates same-category condiments). It also asks a genuinely new question: does the model ALREADY localize the object internally (a grounding head) while the L1 action head ignores it -> a non-blank-vision coupling fix that would NOT regress clean? Method: faithfully replicate predict_action''s multimodal sequence ([tok0]+[512 vision patches]+[prompt+64 action tokens]) but with output_attentions=True; for each (layer,head) read the noun->agentview-patch attention, argmax -> 16x16 patch, and over 10 matched clean/swap tasks count clean-localize / track-swap
  / memorize-old. CRITICAL CORRECTNESS FIX uncovered here: get_libero_image rotates the frame 180, so the model''s patch grid is UPRIGHT while the camera projection is RAW — H11/H12 compared attention/features against the 180-opposite (background) patch. Re-derived the GT with rot180 (verified visually by scripts/calib_project.py: the basket only projects to the correct side under rot180) and matched the GT to the NAMED instruction-target object (not the most-moved object, which is often the distractor swapping into the target''s OLD slot -> memorization mislabeled as tracking). GATE: a head with clean>=0.6n AND track>=0.5n => the VLA internally grounds the swapped object => coupling-fix path opens; else the position prior is baked into the attention too => GROUND wall holds across ALL modalities.

  '
parents:
- '[[2026-06-1jr7vm-h12-dense-openvocab-ground-gate]]'
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/u0hlhq-h13-vla-self-attention-ground-gate
github-commit: db3fc914db12e3873779f9922d33b4784740d99d
date-created: 2026-06-04
date-started: 2026-06-04
date-completed: 2026-06-04
tags: []
metrics:
  probe: vla_llm_self_attention_perhead
  n: 10
  chance_at_tol2: 0.5/10
  noun_best_clean_head: L2h7
  noun_clean: 7/10
  noun_track_swap: 4/10
  noun_memorize_old: 6/10
  action_query_attends: fixed_arm_sink_row1
  dino_corrected_track: 0/8
  projection_bug_found: rot180_get_libero_image_affected_H11_H12
  verdict: localizes-clean-via-prior-but-memorizes-on-swap; GROUND-wall-holds-across-all-4-modalities
---

# u0hlhq — h13-vla-self-attention-ground-gate

## Hypothesis
H13 VLA SELF-ATTENTION GROUND gate — the last untested grounding modality, the one that addresses BOTH prior failure causes at once. H11 (raw DINO correspondence) failed for being visual-only; H12 (off-the-shelf CLIP patch-text) failed for the sim domain gap. The fine-tuned VLA's OWN LLM self-attention from the instruction-noun tokens to the vision-patch tokens is BOTH in-domain (no CLIP gap) AND language-conditioned (disambiguates same-category condiments). It also asks a genuinely new question: does the model ALREADY localize the object internally (a grounding head) while the L1 action head ignores it -> a non-blank-vision coupling fix that would NOT regress clean? Method: faithfully replicate predict_action's multimodal sequence ([tok0]+[512 vision patches]+[prompt+64 action tokens]) but with output_attentions=True; for each (layer,head) read the noun->agentview-patch attention, argmax -> 16x16 patch, and over 10 matched clean/swap tasks count clean-localize / track-swap / memorize-old. CRITICAL CORRECTNESS FIX uncovered here: get_libero_image rotates the frame 180, so the model's patch grid is UPRIGHT while the camera projection is RAW — H11/H12 compared attention/features against the 180-opposite (background) patch. Re-derived the GT with rot180 (verified visually by scripts/calib_project.py: the basket only projects to the correct side under rot180) and matched the GT to the NAMED instruction-target object (not the most-moved object, which is often the distractor swapping into the target's OLD slot -> memorization mislabeled as tracking). GATE: a head with clean>=0.6n AND track>=0.5n => the VLA internally grounds the swapped object => coupling-fix path opens; else the position prior is baked into the attention too => GROUND wall holds across ALL modalities.

## Parents
- [[2026-06-1jr7vm-h12-dense-openvocab-ground-gate]]

## Method
Faithful replication of `predict_action` (scripts/probe_vla_attention.py): build the multimodal sequence
[tok0]+[512 vision patches (agentview 1..256, wrist 257..512)]+[prompt+64 action tokens+stop], forward the
Qwen LLM with output_attentions=True. For each (layer,head) take the noun-token rows (instruction target) ->
agentview-patch columns [1,257), argmax -> 16x16 patch. Over n=10 matched libero_object / libero_object_swap
tasks count clean-localize (argmax on named obj, clean), track (argmax on named obj NEW pos, swap), memorize
(argmax on named obj OLD pos, swap). Also an action-query variant and a sink-removed contrast map.
TWO correctness fixes vs H11/H12: (1) rot180 the camera projection so the GT patch matches the UPRIGHT grid
the model sees (get_libero_image flips 180; verified by scripts/calib_project.py - the basket only lands right
under rot180); (2) GT = the NAMED object, not the most-moved object (the most-moved one is often the distractor
swapping into the target OLD slot -> memorization mislabeled as tracking; this exact artifact gave a spurious
track 7/10 before the fix). Re-ran the H11 DINO probe (scripts/probe_kpc_dinov2.py) with the SAME corrected GT.

## Results
**Per-(layer,head), n=10, chance ~0.5/10 (±2 patches on 16x16):**

| query | best head | clean | track (swap) | memorize (swap) |
|---|---|---|---|---|
| **noun** | **L2h7** | **7/10** | 4/10 | **6/10** |
| action | (arm sink) | ~1/10 | ~2/10 | — |

- A real grounding head exists: **L2h7 localizes the named object on CLEAN 7/10** (>> chance), proving the
  probe + projection are correct and the model CAN see the object when it sits where training expects.
- But on SWAP the SAME head **memorizes 6/10** (attends the OLD location) and only **tracks 4/10** (the new
  location). track+mem ≈ 10/10 => on swap the head attends one of the two swapped slots, picking the MEMORIZED
  one 60%. The position prior is baked into the attention itself.
- The ACTION-query tokens attend to a FIXED arm/gripper sink (row 1, identical every task) — not the object.
- **DINO re-run with the corrected GT: track 0/8** (was a 180-rotated GT before), cosTrue 0.04–0.68 — confirms
  H11 holds: the small object DINO content does not correspond across the move.
- Earlier (buggy) read showed noun L2h7 "track 7/10"; that was the distractor-into-old-slot artifact. Corrected.

![[attachments/u0hlhq-vla-head-L2h7.png]]

## Plots
<!-- ![[attachments/loss-curve.png]] -->

## Conclusion
KILL (gate not passed: need clean>=6 AND track>=5; got clean 7, track 4). **The memorization is in the
attention, not just the L1 action head** — this closes the "model knows but ignores" coupling path. The VLA
DOES carry a language-conditioned localization head (L2h7) that finds the object on CLEAN (7/10) — but it
localizes by the MEMORIZED position prior, not current visual content: on swap it attends the OLD slot 6/10 and
tracks the new one only 4/10 (above chance but prior-dominated). This is H6 confirmed at the attention level
and unifies the whole GROUND story: GROUND on swap requires reading the object CURRENT position, and NO modality
delivers it on LIBERO sim — (a) open-vocab detector crashes/conf 0.002, (b) DINO correspondence 0/8 (content
does not correspond at this object scale, now on corrected footing), (c) CLIP/SigLIP patch-text 0/8 (domain
gap), (d) the VLA OWN attention localizes clean via prior but memorizes on swap (track 4/10). The model that
could ground (in-domain + language-conditioned) instead memorizes; the models without a prior (DINO/CLIP) cannot
localize at all. ⇒ A coupling fix is not viable (the attention you would couple to is itself 60% memorized).
The remaining lever is to BREAK the prior by training perturbation — but that collides with the COMMIT wall
(blank-vision training that breaks the shortcut also destroys real-vision grasping, h11_visioncoord 0/50 even
clean). METHOD SIDE-RESULT: found + fixed a rot180 projection bug that mislabeled GT in H11/H12; re-ran, their
conclusions hold. On REAL RGB (in-domain, larger objects, working detector/SAM) GROUND works — the method stays
real-transferable but UNVALIDATABLE end-to-end on LIBERO sim.

## Next directions
- [ ] <Claude Code có thể fill khi /exp-plan>
