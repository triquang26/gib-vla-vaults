Based on the five adversarial verification verdicts, here is the honest trustworthiness assessment of the H0 baseline.

## (1) OVERALL VERDICT: TRUSTWORTHY — gate the project, build H1 on it

All five dimensions pass independent adversarial verification with verdict `trustworthy: yes`. The critical chain is intact end-to-end:
- **Perturbations are real** at the BDDL/init-state level (verified by file diffs, md5, and qpos decode), not near-no-ops.
- **Model inputs are correct** for every dim: the instruction-source fix fires for `_lan`/`_task` (paraphrase/relabel reaches `get_action`) and correctly does NOT fire for `clean`/`object`/`swap` (0 bddl-instruction lines each). `unnorm_key` resolves identically (`libero_object_no_noops`) across all dims, so no per-dim norm-stat divergence.
- **Failures are genuine** task failures, not crashes: 0 tracebacks/exceptions/NaN/OOM anywhere; all 150 episodes/dim completed; failures run to `max_steps=280` timeouts (confirmed by swap's 67-70s/it vs clean's 40-49s early-termination timing); success is gated strictly on env `done=True`, exceptions force `success=False`.

The numbers tell a coherent, mechanistically-explained story (recolor-robust / scale-fragile object; total spatial-grounding collapse on swap; ceilinged paraphrase-robustness; partial referent-grounding on task). This is a sound gate and a valid foundation for H1.

## (2) CAVEATS / THREATS-TO-VALIDITY (record in the experiment node)

1. **Task/lan numbers are NOT comparable to the leaderboard — instruction-source fix changes the protocol.** The harness reads the perturbed instruction from the BDDL `(:language)` block and overrides the filename-derived instruction. The leaderboard ~0.00 on `task` is *inferred* to come from feeding the ORIGINAL instruction while grading the NEW goal (unwinnable). Our 0.093 is correctly higher because we remove that instruction/goal mismatch; grading (env goal) is unchanged. **Any leaderboard delta is partly protocol, not model.** Record `task=0.093` and `lan=1.0` as "post-fix, not leaderboard-comparable."

2. **Single seed (seed=7), 15 episodes/task vs the official 50 — CIs unknown, violates the project's own ≥3-seed rule.** Init states are deterministic, so the 15 ep/task are FIXED initial conditions: numbers are exact-on-these-150-inits, not averaged. Adds variance (no directional bias), but `swap=0` and `lan=1.0` carry no confidence interval.

3. **Headline aggregates mask all-or-nothing per-task patterns.**
   - `object=0.90` = 9/10 tasks at ceiling (1.0) + 1 task at floor (0.0, the *rescaled* alphabet_soup). Not a uniform 90%.
   - `task=0.093` rests entirely on ONE of 10 tasks (task1 = 0.933, other 9 = 0.0). High-variance, single-task-driven.

4. **`object` dim tests geometry robustness with N=1.** 9/10 perturbations are color recolors (a vision backbone may be largely invariant); only alphabet_soup has a true scale change — and that is the one that breaks. The "robust to visual change" claim is well-supported for recolor, weakly supported (1 case) for geometry.

5. **Ceiling saturation limits `lan`.** Clean is already 1.0, so `lan=1.0` proves only unchallenging lexical/syntactic paraphrase-robustness on a saturated suite — no headroom to detect a small gap, and no test of adversarial/referent-swapping paraphrases.

6. **Reproducibility fragility (not an active confound here):** the eval relies on `env.sh` exporting `LIBERO_CONFIG_PATH` to override the global `~/.libero/config.yaml` (which points at a CLEAN Evo-1 LIBERO). If a future run forgets to source `env.sh`, the env silently loads clean BDDLs. Also ~15 `config.json.back.*` backups indicate the checkpoint config is mutated at eval-time — pin/snapshot it.

7. **Spurious-success risk minimally audited.** Success = env `done=True` (object-in-basket); individual rollout videos were not frame-audited. Mitigated because clean=1.0 (policy at ceiling), so degradation reflects the perturbation, not luck — but `object`/`task` successes were not video-confirmed.

## (3) MINIMAL ADDITIONAL CONTROLS before ANY cross-model claim

1. **Run VLA-Adapter (and any second model) through the IDENTICAL harness** — same `env.sh` override, same instruction-source fix, same 15×10 fixed inits, same seed. No cross-model claim is valid against the leaderboard's different protocol; only same-harness numbers are comparable.

2. **`task`: add a "feed-original-instruction" control variant.** Run `task` BOTH ways — (a) fixed instruction (current, 0.093) and (b) original filename-derived instruction graded against new goal. This directly *measures* (rather than infers) the leaderboard's ~0.00 cause and isolates protocol effect from model effect. Mandatory before attributing any task-dim delta to the model.

3. **≥3 seeds (or vary init states) on at least `swap`, `object`, `task`** to attach CIs — especially since `task` hangs on a single task and `object`'s floor hangs on a single rescaled object.

4. **Report per-task SR, not just the aggregate**, for every model, so the all-or-nothing structure is visible and cross-model comparison isn't masked by a single driving task.

5. **Add ≥1 more geometry/scale perturbation** to the `object` dim so the geometry-robustness claim rests on more than N=1.

6. **Snapshot the eval config + assert `LIBERO_CONFIG_PATH` override** at run start (fail loudly if clean BDDLs load), and record the checkpoint `config.json` hash, to make the cross-model runs reproducible.

Relevant paths: `eval_logs/h0-object/results.json`, `eval_logs/h0-object/EVAL-*.txt`, `run_libero_eval.py` (L84 `INSTRUCTION_PERTURBING`, L104-105/231-241 `base_suite_of`/unnorm_key, L366 max_steps, L410-416 success/exception gating, L452-460 bddl-instruction override), `scripts/run_libero_pro_battery.sh` (L29-35), `env.sh` (`LIBERO_CONFIG_PATH` override), `LIBERO-PRO/libero/libero/{bddl_files,init_files}/`.