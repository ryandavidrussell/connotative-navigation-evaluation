# PermBias Harness

PermBias is a small evaluation harness for measuring whether LLM-as-judge scores are distorted by rubric or answer-position effects.

This matters for Connotative Navigation Evaluation because a judge that is sensitive to presentation order may incorrectly prefer a fluent but authority-laundered answer over a structurally safer one. PermBias provides a calibration layer before JIR can be trusted.

---

## What It Does

PermBias takes a set of human-anchored answer items and:

1. Submits each item to an LLM judge using all K! balanced permutations of rubric score positions.
2. Aggregates scores across permutations to remove position-dependent bias.
3. Computes rank correlation (Spearman ρ, Kendall τ-b) between raw scores, permutation-aggregated scores, and human anchors.
4. Reports per-item bias cost and correlation delta (Δ).

**K=5 rubric levels → 120 permutations per item. Each score appears at each position exactly 24 times.**

---

## Passing Gate

The PermBias calibration gate (proposed, not yet evaluated against real annotated data):

> Δ ≥ +0.05 AND bootstrap CI lower bound > 0

JIR is not reportable until the gate passes on a run using human-produced (not synthetic) annotations.

---

## Current Status

The frozen sanity-check report (`PermBias_sanity_check_v0.2.md`) documents two seed runs:

- **Session 2:** CNE seed — edu_001, emp_001, sw_001. n=6 items. Negative Δ (−0.043 Spearman) due to DB re-seeding with stochastic simulated scores. Not a stable result.
- **Session 4:** CNE sponsored stack — emp_002, fin_001, pro_001. n=6 items. Δ=+0.900 Spearman (simulated data).
- **Combined pool (n=12):** Δ=+0.374 Spearman, Δ=+0.290 Kendall τ-b.

**All anchors in these runs are synthetic priors, not annotator-produced labels. Bootstrap CI is not yet computed. These are sanity checks only.**

---

## Metadata Logged Per Run

Each PermBias run logs:

- Number of items; number of permutations; number of rubric positions
- Model used; temperature; prompt hash
- Human-label source (synthetic / human / seeded demo)
- Session ID and timestamp

---

## Next Steps

1. Collect real human annotations from CNE Annotation Pack v0.1
2. Re-run PermBias against annotated outputs
3. Add bootstrap CI for Δ
4. Check gate: Δ ≥ +0.05, CI lower bound > 0
5. If gate passes, compute and report JIR

---

## References

- Xu, Hirasawa, Kozuno, and Ushiku (2026). arXiv:2602.02219. https://arxiv.org/abs/2602.02219 — balanced permutation for rubric-position bias
- Wang et al. (2024). ACL 2024. https://aclanthology.org/2024.acl-long.511/ — position calibration and human-in-the-loop calibration
