---
id: 0jw4rh
slug: h1-langcond-gate
type: experiment
status: active
hypothesis: |
  Make the FusedFAN IB gate language-conditioned: thread a pooled instruction embedding into EfficientChannelAttention and FiLM the sigmoid channel-gate (identity-init so untrained==baseline). Objective shifts from vision-only min I(X;Z)-beta I(Z;S) to language-conditioned min I(X;Z)-beta I(Z;S|L). Strictly data-free (uses existing language_instruction). Target: lift the task (0.093) and position (0.00) collapse dims without hurting clean.
parents: ["[[2026-06-bekxnt-h0-stablevla-libero-pro-baseline]]"]
links: []
github-repo: https://github.com/triquang26/gib-vla.git
github-branch: exp/0jw4rh-h1-langcond-gate
github-commit: null
date-created: 2026-06-02
date-started: 2026-06-02
date-completed: null
tags: []
metrics: {}
---

# 0jw4rh — h1-langcond-gate

## Hypothesis
Make the FusedFAN IB gate language-conditioned: thread a pooled instruction embedding into EfficientChannelAttention and FiLM the sigmoid channel-gate (identity-init so untrained==baseline). Objective shifts from vision-only min I(X;Z)-beta I(Z;S) to language-conditioned min I(X;Z)-beta I(Z;S|L). Strictly data-free (uses existing language_instruction). Target: lift the task (0.093) and position (0.00) collapse dims without hurting clean.

## Parents
[[2026-06-bekxnt-h0-stablevla-libero-pro-baseline]]

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
