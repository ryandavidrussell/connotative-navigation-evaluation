# Connotative Navigation Evaluation

**Authority Laundering, Judge Calibration, and Action-Sensitive Retrieval Contexts**

Status: Concept Note v0.1 · Seed scaffold — not a benchmark release
Project: Kaleidoworks / Soraya governance research

---

## The Problem

LLM-powered search and question-answering systems increasingly compress many visible content objects into a single fluent answer: advertisements, official sources, user comments, testimonials, benchmarks, citations, redirects, and calls to action. This compression creates a governance problem that ordinary semantic retrieval does not fully capture.

**A source can be semantically relevant without being authorized to play the evidentiary role the answer assigns to it.**

For example, a visible interaction may contain:

- a sponsored course advertisement,
- an organic comment expressing enthusiasm about the broader topic,
- a commercial redirect recommending a cheaper paid alternative.

A conventional model may summarize this as apparent consensus: people are excited, the advertised program is relevant, and a cheaper alternative is available. But the correct interpretation is more constrained: the advertisement establishes only that something is being promoted; the organic comment establishes topical interest, not product validation; the commercial redirect is another transactional offer, not independent evidence.

This is the failure mode CNE calls **authority laundering**: semantic adjacency, fluency, or repetition converts weak, promotional, or role-limited evidence into apparent validation or action authority.

---

## What CNE Is

CNE represents visible content as **typed navigation objects** rather than undifferentiated semantic text. Each item carries independent descriptors:

- **Origin type:** first-party, third-party, user-generated, unknown
- **Communication mode:** official description, sponsored promotion, organic testimony, commercial recommendation, editorial summary, unknown
- **Action tier:** read, compare, transact, disclose, execute
- **Structural relation:** comments on, replies to, links to, co-presented with, calls to action

The central rule: retrieval may surface relevant content, but the answer must preserve what each content object is permitted to establish.

This aligns with the Soraya governance principle: **semantic relevance is not operational authority.**

---

## Relation to AuthorityBench

[AuthorityBench](https://arxiv.org/abs/2603.25092) (Yao, Zhang, and Bi, 2026) is the closest identified prior art. It asks whether LLMs can perceive source authority and use that authority to improve RAG. Its datasets include domain-level authority (DomainAuth), entity-level authority (EntityAuth), and a downstream RAG authority benchmark (RAGAuth).

CNE asks a different question:

> AuthorityBench: *Is this source authoritative enough to trust or filter for RAG?*
>
> CNE: *What evidentiary role is this visible content object permitted to play in this answer, under this user request?*

Domain authority can tell us that a university, government site, vendor, or publisher may be broadly authoritative. It does not tell us that a **sponsored** university advertisement is independent evidence of its own program value, or that a user comment near that ad validates the advertised product.

CNE focuses on role preservation, action boundaries, and authority transfer under adjacency pressure — rather than authority perception alone.

---

## First Experiment: C0 vs. C1

The first experiment is deliberately not a retrieval experiment. Both conditions receive the same fixed visible context.

| Condition | Description |
|-----------|-------------|
| **C0** raw-context answerer | The model receives raw visible content only. Source roles are absent or collapsed. |
| **C1** oracle-typed answerer | The model receives the same visible content plus human-supplied descriptive metadata: source role, action tier, and visible structure. |

The first experimental question is narrow: given the same visible interaction context, does oracle-supplied connotative metadata reduce authority-laundering errors compared with raw context alone?

C1 does not yet prove automatic role detection. It tests whether typed connotative context helps a model preserve evidentiary boundaries.

---

## Evaluation Metrics

CNE separates generation, annotation, judge calibration, and final reporting.

**Annotation fields:**
- Role Preservation Score, 1–5
- Contains Authority Laundering
- Contains Prohibited Action
- Remains Useful

**Derived metrics:**

| Metric | Definition |
|--------|-----------|
| **ALER** | Authority Laundering Error Rate — fraction of answer items marked as laundering authority |
| **DPR** | Discordant Pair Rate — fraction of C0/C1 pairs where human annotation identifies a meaningful difference |
| **JIR** | Judge Inversion Rate — on discordant gold pairs only: fraction of cases where an LLM judge prefers the laundered answer |

**JIR is not reported from an uncalibrated judge.** See PermBias below.

---

## PermBias: Judge Calibration Gate

CNE does not report Judge Inversion Rate directly from an uncalibrated LLM judge. The companion harness, **PermBias**, tests whether rubric-based judge scores are sensitive to score-order or presentation-order effects.

PermBias implements balanced permutation aggregation and reports:
- raw rank correlation with human labels (Spearman ρ, Kendall τ-b)
- permutation-aggregated correlation
- correlation delta (Δ)
- per-item bias cost
- bootstrap confidence intervals (CI) for Δ

**The passing gate (proposed, not yet evaluated):** Δ ≥ +0.05 AND CI lower bound > 0.

The frozen sanity-check report (`harness/PermBias_sanity_check_v0.2.md`) shows PermBias exposing judge instability in CNE-style synthetic seed cases. That report is a methodological sanity check, not a benchmark result. Bootstrap CI is not yet computed for any run in v0.

---

## Repository Layout

```
connotative-navigation-evaluation/
├── README.md                          ← this file
├── CONCEPT_NOTE.md                    ← full concept note v0.1
├── LICENSE
├── .gitignore
│
├── annotation_pack_v0.1/              ← seed annotation scaffold
│   ├── README.md                      ← pack-specific guide
│   ├── cases/
│   │   └── cne_annotation_cases_v0.1.jsonl   ← 16 C0/C1 pair definitions
│   ├── prompts/
│   │   ├── generate_C0_raw_context.txt
│   │   ├── generate_C1_oracle_typed_context.txt
│   │   └── human_annotation_rubric.txt
│   ├── outputs/
│   │   └── human_annotations_blank.jsonl     ← 32 blank annotation slots
│   └── docs/
│       └── annotation_protocol.md
│
├── harness/
│   └── PermBias_sanity_check_v0.2.md  ← FROZEN. Seed-run sanity check report.
│
└── docs/
    └── (concept notes, design documents — future)
```

---

## Current State

| Item | Status |
|------|--------|
| Concept note | v0.1 — written |
| Annotation pack | v0.1 — 16 pairs, seed scaffold |
| PermBias harness | Running — sanity-check report frozen |
| Human annotations | Not yet collected |
| Bootstrap CI | Not yet computed |
| 25-pair target | Not yet reached (currently 16) |
| JIR reportable | No — PermBias gate not yet passed with real annotations |

**This repository is a research scaffold. The annotation pack contains synthetic human anchors, not annotator-produced labels. No results in this repository should be cited as benchmark results.**

**For annotators:** If you are here to score items, read `annotation_pack_v0.1/prompts/human_annotation_rubric.txt` and complete your independent scores before opening anything in `annotation_pack_v0.1/docs/`. The `calibration_examples.md` file contains gold labels and explained scores — it is calibration material for use *after* independent scoring, not a guide to read first. Opening it before scoring will invalidate your annotation.

---

## Near-Term Plan

1. Generate real C0/C1 answers from Annotation Pack v0.1
2. Collect at least two independent human annotations per item
3. Compute ALER and Discordant Pair Rate
4. Run PermBias against human labels with bootstrap CI
5. Report JIR only after PermBias gate passes
6. Expand from 16 to at least 25 C0/C1 pairs
7. Add predicted typing and controlled repetition stress tests

---

## Key References

- Xu, Hirasawa, Kozuno, and Ushiku (2026). "Am I More Pointwise or Pairwise? Revealing Position Bias in Rubric-Based LLM-as-a-Judge." arXiv:2602.02219. https://arxiv.org/abs/2602.02219
- Wang et al. (2024). "Large Language Models are not Fair Evaluators." ACL 2024. https://aclanthology.org/2024.acl-long.511/

---

*Research scaffold — pre-publication. Synthetic seed data only. Not yet a citable benchmark result.*
