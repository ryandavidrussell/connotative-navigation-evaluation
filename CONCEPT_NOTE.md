# Connotative Navigation Evaluation

**Authority Laundering, Judge Calibration, and Action-Sensitive Retrieval Contexts**

Status: Concept Note v0.1
Project: Kaleidoworks / Soraya governance research
Current artifact state: Running calibrated evaluation pipeline with a frozen PermBias sanity-check report and CNE Annotation Pack v0.1.

---

## 1. Problem

LLM-powered search and question-answering systems increasingly compress many visible content objects into a single fluent answer: advertisements, official sources, user comments, testimonials, benchmarks, citations, redirects, and calls to action. This compression creates a governance problem that ordinary semantic retrieval does not fully capture.

A source can be semantically relevant without being authorized to play the evidentiary role the answer assigns to it.

For example, a visible interaction may contain:

- a sponsored course advertisement,
- an organic comment expressing enthusiasm about the broader topic,
- a commercial redirect recommending a cheaper paid alternative.

A conventional model may summarize this as apparent consensus: people are excited, the advertised program is relevant, and a cheaper alternative is available. But the correct interpretation is more constrained: the advertisement establishes only that something is being promoted; the organic comment establishes topical interest, not product validation; the commercial redirect is another transactional offer, not independent evidence.

This is the failure mode I call **authority laundering**: semantic adjacency, fluency, or repetition converts weak, promotional, or role-limited evidence into apparent validation or action authority.

---

## 2. Relation to AuthorityBench

AuthorityBench is the closest identified prior art. It asks whether LLMs can perceive source authority and use that authority to improve retrieval-augmented generation. Its datasets include domain-level authority, entity-level authority, and a downstream RAG authority benchmark.

Connotative Navigation Evaluation asks a different question.

AuthorityBench asks:

> Is this source authoritative enough to trust or filter for RAG?

CNE asks:

> What evidentiary role is this visible content object permitted to play in this answer, under this user request?

That distinction matters. Domain authority can tell us that a university, government site, vendor, or publisher may be broadly authoritative. It does not tell us that a sponsored university advertisement is independent evidence of its own program value, or that a user comment near that ad validates the advertised product.

CNE therefore focuses on role preservation, action boundaries, and authority transfer under adjacency pressure, rather than authority perception alone.

---

## 3. Core Framework

CNE represents visible content as typed navigation objects rather than undifferentiated semantic text. Each item can carry independent descriptors:

- **Origin type:** first-party, third-party, user-generated, unknown
- **Communication mode:** official description, sponsored promotion, organic testimony, commercial recommendation, editorial summary, unknown
- **Action tier:** read, compare, transact, disclose, execute
- **Structural relation:** comments on, replies to, links to, co-presented with, calls to action

The central rule is:

> Retrieval may surface relevant content, but the answer must preserve what each content object is permitted to establish.

This is directly aligned with Soraya's governance principle: semantic relevance is not operational authority.

---

## 4. First Experiment: C0 vs. C1

The first experiment is deliberately not a retrieval experiment. Both conditions receive the same fixed visible context. The difference is whether the model receives connotative typing.

| Condition | Description |
|-----------|-------------|
| **C0** raw-context answerer | The model receives raw visible content only. Source roles are absent or collapsed. |
| **C1** oracle-typed answerer | The model receives the same visible content plus human-supplied descriptive metadata: source role, action tier, and visible structure. |

The first experimental question is narrow:

> Given the same visible interaction context, does oracle-supplied connotative metadata reduce authority-laundering errors compared with raw context alone?

This avoids overclaiming. C1 does not yet prove automatic role detection. It tests whether typed connotative context helps a model preserve evidentiary boundaries.

Later conditions can add:

- predicted typing,
- interpreted graph relations,
- actual retrieval/reranking,
- repetition stress tests,
- harmonic or other anti-amplification features.

---

## 5. Evaluation Metrics

CNE separates generation, annotation, judge calibration, and final reporting.

Primary annotation fields include:

- Role Preservation Score, 1–5
- Contains Authority Laundering
- Contains Prohibited Action
- Remains Useful

Derived metrics include:

**Authority Laundering Error Rate (ALER)**
The fraction of answer items marked as laundering authority.

**Discordant Pair Rate (DPR)**
The fraction of C0/C1 pairs where human annotation identifies a meaningful difference between raw and typed conditions.

**Judge Inversion Rate (JIR)**
Computed only on discordant gold pairs:

> Of the pairs where humans identify one answer as role-preserving and the other as laundered, how often does an LLM judge prefer the laundered answer?

This matters because a fluent but structurally wrong answer may look more helpful than a governed answer that preserves uncomfortable distinctions.

---

## 6. PermBias: Judge Calibration Before JIR

CNE does not report Judge Inversion Rate directly from an uncalibrated LLM judge. That would be methodological quicksand with a nice dashboard.

The companion harness, PermBias, tests whether rubric-based judge scores are sensitive to score-order or presentation-order effects. It implements balanced permutation aggregation inspired by recent work on rubric-position bias ([Xu et al., arXiv:2602.02219](https://arxiv.org/abs/2602.02219)), and uses position-calibration logic from prior pairwise judge-bias work ([Wang et al., ACL 2024](https://aclanthology.org/2024.acl-long.511/)).

PermBias reports:

- raw rank correlation with human labels (Spearman ρ, Kendall τ-b)
- permutation-aggregated correlation
- correlation delta (Δ)
- per-item bias cost
- JSON export for reproducibility

The current frozen sanity-check report shows that PermBias can expose judge instability in CNE-style synthetic seed cases. That report is not yet a benchmark result. Its role is methodological: it establishes the calibration layer required before JIR can be trusted.

The key rule:

> PermBias is not the result; PermBias is the calibration gate that determines whether a JIR result is reportable.

---

## 7. Current Annotation Pack

CNE Annotation Pack v0.1 contains a sponsored/organic/commercial-redirect family:

- 4 cases,
- 4 query routes per case,
- 16 C0/C1 pairs,
- 32 annotation slots.

The query routes increase in action sensitivity:

1. **Describe** — summarize visible content
2. **Evaluate** — assess credibility or quality
3. **Decide** — recommend a course of action
4. **Act** — produce or authorize a specific action

Health is intentionally excluded from v0 to avoid safety-tuning confounders. The initial domains focus on education, employment, finance, and professional decision-making.

The 25-pair reporting target has not yet been reached. The current pack is an annotation scaffold, not a completed benchmark.

---

## 8. Why This Matters

Search and QA systems do not merely retrieve facts. They route attention, compress source landscapes, and shape perceived authority. In LLM interfaces, this routing becomes more powerful because many sources are collapsed into one fluent answer.

A system that cannot distinguish "this source is relevant" from "this source is authorized to support this claim" will repeatedly make structurally unsafe mistakes:

- treating ads as evidence,
- treating topic interest as endorsement,
- treating commercial redirects as validation,
- treating repeated promotional signals as consensus,
- treating a user's action request as justified by insufficient source roles.

CNE is designed to make those failures measurable.

---

## 9. Near-Term Plan

1. Generate real C0/C1 answers from Annotation Pack v0.1
2. Collect at least two independent human annotations per item
3. Compute ALER and Discordant Pair Rate
4. Run PermBias against human labels with bootstrap CI
5. Report JIR only after the PermBias gate passes
6. Expand from 16 to at least 25 C0/C1 pairs
7. Add predicted typing and controlled repetition stress tests

---

## 10. Closing Claim

Existing authority-aware RAG work studies whether a source is authoritative enough to improve retrieval-augmented generation. Connotative Navigation Evaluation studies whether a retrieved or visible content object is being used according to the evidentiary role its visible function permits, especially under action-sensitive user requests.

That is the contribution: not just better retrieval, but governed interpretation of retrieved context.

We have a running calibrated evaluation pipeline.

---

## References

- Xu, Hirasawa, Kozuno, and Ushiku (2026). "Am I More Pointwise or Pairwise? Revealing Position Bias in Rubric-Based LLM-as-a-Judge." arXiv:2602.02219. https://arxiv.org/abs/2602.02219
- Wang et al. (2024). "Large Language Models are not Fair Evaluators." ACL 2024. https://aclanthology.org/2024.acl-long.511/
