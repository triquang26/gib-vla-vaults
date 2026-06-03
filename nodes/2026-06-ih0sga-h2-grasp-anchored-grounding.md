---
id: ih0sga
slug: h2-grasp-anchored-grounding
type: experiment
status: active
hypothesis: |
  Position-swap collapse (swap=0.00) is a spatial-grounding failure: the policy grasps the
  remembered location, not the named object. Fix data-free with a real-transferable supervision
  signal no prior work uses — the demonstration's own GRASP PROPRIOCEPTION. The named target sits
  where the gripper closes, so eef xyz at the grasp moment (fingers most closed) localizes it
  (free, exists on real robots; verified clean+discriminative on libero_object). Add an explicit
  localization head (image+instruction -> target xyz) trained jointly with the action loss
  (H2a). If the head merely memorizes object->position (some libero objects are near-fixed), add
  counterfactual instruction-rebinding (wrong instruction -> push prediction off the grasp loc)
  to force instruction-conditioned visual localization (H2b). Target: lift swap off 0.00 (and
  task) without hurting clean. Novelty vs ReconVLA (latent reconstruction, detector labels,
  object-level), ObjectVLA (object-id, VL pairs), RoboPoint (sim-GT, separate VLM), LIBERO-Plus
  (20k new trajectories, 7B): explicit head + grasp-proprio label + position-swap eval, small VLA.
parents: ["[[2026-06-0jw4rh-h1-langcond-gate]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/ih0sga-h2-grasp-anchored-grounding
github-commit: null
date-created: 2026-06-03
date-started: 2026-06-03
date-completed: null
tags: [grounding, position-swap, grasp-pose, data-free, real-transferable]
metrics: {}
---

# ih0sga — h2-grasp-anchored-grounding

## Hypothesis
See frontmatter. Data-free, real-transferable grounding via grasp-proprioception localization label;
warm-start from outputs/object; eval only the collapse dims (swap, task) + clean guard.

## Parents
[[2026-06-0jw4rh-h1-langcond-gate]] — H1 (FiLM on vision IB gate) was NULL on swap; representation
tweak insufficient → move the intervention to a supervision-side localization objective.

## Method
<điền khi /exp-record>

## Results
<điền khi /exp-record>

## Plots
<!-- ![[attachments/...]] -->

## Conclusion
<điền khi /exp-record>

## Next directions
- [ ] H2b counterfactual instruction-rebinding if H2a head-alone memorizes.
