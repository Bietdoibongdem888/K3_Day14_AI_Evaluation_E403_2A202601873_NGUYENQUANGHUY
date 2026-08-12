# Day 14 — Reflection

## Evaluation Report & Failure Analysis

**Student:** Nguyễn Quang Huy · **Student ID:** 2A202601873 · **Class:** E403 · **Cohort:** K3

## 1. Benchmark Results Summary

**Status:** Real Gemini benchmark not completed. The repository now uses the official `google-genai` SDK through a provider abstraction, but neither the environment nor local `.env` contains `GEMINI_API_KEY`. No actual-answer or benchmark artifact was produced.

Consequently, pass rate, average/minimum/maximum Context Recall, Context Precision, Faithfulness, Relevance, Completeness, Overall Score, score bands, and failure distribution are **not available**. Recording numeric values here would fabricate benchmark evidence.

The retrieval-versus-generation diagnosis is also deferred. It requires at least Context Recall/Precision plus Faithfulness/Completeness from real traces; without those measurements, attributing failures to either subsystem would be speculation.

## 2. Top 3 Worst Failures — 5 Whys

The three lowest cases cannot be selected until `artifacts/actual_answers.json` and `artifacts/benchmark_results.json` exist. For each eventual case, the analysis will record the ID, full expected and actual answers, all five metrics, overall score, failure type, and exact retrieved-chunk inspection before starting the causal chain.

The required chain will be applied as follows:

| Level | Evidence requirement |
|---|---|
| Symptom | State the observed answer defect and the metric(s) that expose it. |
| Why 1 | Connect the defect to a missing, noisy, contradicted, or unused retrieved fact. |
| Why 2 | Connect that evidence condition to query, retrieval, ranking, chunking, or generation behavior. |
| Why 3 | Identify why the current prompt/index/guardrail permitted that behavior. |
| Why 4 | Identify why evaluation or monitoring did not prevent it earlier. |
| Why 5 | Name one controllable engineering root cause with a testable fix. |

For each case, `FailureAnalyzer.find_root_cause()` will be compared with the trace. Agreement will be accepted only if its metric branch matches retrieved evidence; otherwise the manual trace finding takes precedence and the heuristic limitation will be documented. No three case-specific 5 Whys are invented while model output is absent.

## 3. Failure Clustering

Benchmark-derived IDs and cluster priorities are deferred. The actionable taxonomy to apply is:

| Cluster | Evidence-based root cause rule | Candidate priority rule |
|---|---|---|
| Retrieval coverage | Low Context Recall: necessary evidence absent from the retrieved union. | High when it affects deadlines, fees, eligibility, or policy versions. |
| Ranking/noise | Adequate recall but low Context Precision: evidence exists but is buried. | Medium; High if the generator follows an earlier conflicting chunk. |
| Generation/grounding | Strong retrieval with low Faithfulness or Completeness: evidence is ignored, distorted, or only partly synthesized. | High for unsupported policy claims or omitted required actions. |
| Intent/scope/adversarial handling | Low relevance, inappropriate refusal, prompt-injection compliance, or false-premise acceptance. | High for privacy/security; otherwise based on recurrence and harm. |

If only one measured cluster could be fixed, I would choose the cluster with the largest number of affected cases weighted by consequence, not merely the lowest average metric. A privacy leak or wrong deadline may outrank several harmless wording failures.

## 4. Improvement Log

Calling `generate_improvement_log([], [])` before a real benchmark truthfully yields only the table header because there are no observed failures:

| Failure ID | Type | Root Cause | Suggested Fix | Priority | Status |
|---|---|---|---|---|---|

Provisional engineering actions are kept separate from observed failure claims:

| Action | Target metric | Expected effect | Verification method | Status |
|---|---|---|---|---|
| Add multi-query retrieval and version/effective-date metadata filters. | Context Recall | Recover cross-document and controlling-version evidence. | Rerun all Hard cases; compare per-case recall and inspect the union without changing gold evidence. | Open |
| Add a deterministic reranker before generation. | Context Precision | Move chunks that contain required policy terms and dates earlier without changing recall. | On at least five real traces, verify identical chunk sets, unchanged recall, and non-decreasing precision. | Open |
| Add a structured policy checklist and unsupported-claim guardrail. | Faithfulness and Completeness | Reduce invented rules and missing fees, deadlines, exceptions, and offices. | Rerun the fixed 20-case suite plus adversarial cases; require no critical-case regression. | Open |

These are risk-driven hypotheses. Their priority must be updated after real benchmark clusters are available.

## 5. Regression Testing Strategy

Run `run_regression()` on every prompt, model, retrieval, embedding, reranker, chunking, corpus, dependency, or evaluation-code change; also run it nightly on the pinned golden dataset and before deployment. The quality gate belongs after offline benchmark artifact validation and before targeted human review.

A 0.05 drop is a useful general tolerance for this deterministic 20-case lab, but Student Services also needs absolute gates and critical-case rules. On only 20 records, averages can hide one dangerous answer, so faithfulness below 0.70 on any privacy, fee, deadline, eligibility, or version-selection case should block regardless of aggregate delta.

Deployment blockers:

- faithfulness or completeness crossing its absolute gate;
- any prompt-injection/privacy failure, unsupported waiver/approval, or wrong controlling policy version;
- more than 0.05 decline in Faithfulness, Relevance, or Completeness;
- malformed/missing artifact IDs or retrieval traces.

Warnings that require investigation but may not block alone include a small Context Precision decrease with unchanged recall and answer quality, latency/cost drift, or a non-critical score movement within tolerance. Human review is mandatory for adversarial cases, policy ambiguity, privacy/security, medical exceptions, appeals, and any case close to a blocking threshold.

```text
Code / Prompt / Retrieval Change
→ Offline Benchmark
→ Regression Gate
→ Targeted Human Review
→ Deploy
→ Online Monitoring
```

Offline measurement prevents known regressions; the gate applies absolute and delta rules; human reviewers adjudicate consequential ambiguity; online monitoring detects traffic and policy drift that the static dataset cannot cover.

## 6. Continuous Improvement Loop

```text
Evaluate → Analyze → Improve → Augment benchmark → Repeat
```

| Priority | Action | Expected metric | Expected impact |
|---:|---|---|---|
| 1 | Run the approved real benchmark and preserve answer/retrieval traces. | All metrics | Establish an auditable baseline rather than optimizing from assumptions. |
| 2 | Fix the highest harm-weighted measured cluster and add its trace as a regression case. | Cluster-dependent | Convert observed failures into durable protection. |
| 3 | Calibrate overlap and LLM-judge decisions against independent human labels. | Judge agreement and answer-quality validity | Reduce lexical and judge bias in deployment decisions. |

Candidate augmentation categories—not claimed failures—are: an older-event policy-version case, a question combining scholarship and medical withdrawal, and an indirect prompt injection embedded in retrieved text. Exact cases should be selected after the first trace review to avoid duplicating existing intent.

## 7. Final Reflection

The benchmark outcome itself cannot contradict an initial prediction because no Gemini answer was produced. The important engineering finding was earlier in the pipeline: provider abstraction and dependency readiness do not replace valid provider credentials. Reproducible evaluation includes credential hygiene, explicit provider/model metadata, and artifact provenance, not just functioning code.

Word-overlap heuristics are deterministic and transparent, but they miss lexical mismatch, synonyms and paraphrases, semantic equivalence, tokenization differences, negation, factual nuance, and procedural correctness. They can also reward an unsupported statement that repeats context vocabulary, and penalize a correct concise refusal whose wording differs from the reference. Context Precision's binary relevance threshold is sensitive to chunk size, while set tokenization discards frequency and sentence relationships.

For production Student Services evaluation, I would combine lexical checks with embeddings for semantic similarity, NLI/entailment for claim grounding and contradiction, retrieval recall/precision with version metadata, a calibrated domain-specific LLM judge, deterministic checks for dates/fees/offices, task-completion assertions, and human review for consequential cases. Online monitoring should track unsupported-claim rate, refusal/scope errors, escalation correctness, user outcomes, latency, cost, and drift by policy version. The fixed golden suite remains the regression anchor; human labels and real incident-derived cases continually refine it.
