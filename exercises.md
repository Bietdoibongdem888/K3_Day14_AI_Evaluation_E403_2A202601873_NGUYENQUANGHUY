# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Student:** Nguyễn Quang Huy

**Student ID:** 2A202601873

**Class/Track:** E403

**Cohort:** K3

**GitHub:** Bietdoibongdem888

**Domain:** Northstar University Student Services

## Part 1 — Warm-up

### Exercise 1.1 — RAGAS Metric Thresholds

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | A short refusal or escalation contains little corpus wording but makes no unsupported policy claim. | Below 0.60 on an in-scope policy answer, especially tuition, deadlines, privacy, or eligibility. | Inspect claim-to-context support; strengthen grounding and block release below the safety gate. |
| Answer Relevance | A necessary safety warning briefly precedes the direct answer. | Below 0.60 because the response answers a different intent or evades an in-scope question. | Fix intent routing and require the first sentence to address the request. |
| Context Recall | A deliberately narrow question needs only one fact and retrieval still contains that fact. | Below 0.70 on multi-condition, cross-document, or effective-date cases. | Rewrite the query, review chunk boundaries and add multi-query retrieval. |
| Context Precision | Relevant evidence is present but one harmless navigation chunk ranks ahead of it. | Below 0.50 when policy/version evidence is buried under noise. | Add metadata filters and reranking; inspect precision by rank. |
| Completeness | A concise answer omits optional background while preserving every required action and condition. | Below 0.60 when a deadline, exception, fee, office, or safety step is missing. | Add a policy checklist and structured response template, then rerun targeted cases. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

**Position-bias experiment.** Build matched answer pairs with a human-agreed quality label. Condition A presents the stronger answer first and condition B reverses the same pair. A second condition randomizes labels (`A/B` versus `B/A`) and hides model identity. Repeat each ordering across the same questions and compare mean score and winner rate by position. A material score shift after reversal indicates positional bias.

**Reducing verbosity bias.** The rubric scores required policy claims, necessary conditions, correct office/deadline, and unsupported claims. It explicitly says extra length earns no credit, irrelevant detail reduces relevance, and a concise answer can receive 5/5. Evaluators receive a claim checklist, not a style preference for long prose.

**Human calibration.** Human labels establish what counts as a consequential error in Student Services, expose systematic leniency/severity, and let us estimate agreement before using the judge as a gate. Calibration examples must include privacy, deadlines, policy versions, valid refusals, and concise but complete answers.

### Exercise 1.3 — Evaluation trong CI/CD

| Metric | Block threshold | Rationale |
|---|---:|---|
| Faithfulness | 0.80 aggregate and no critical case below 0.70 | Unsupported policy claims can cause financial, academic, or privacy harm. |
| Answer Relevance | 0.75 aggregate | The assistant must resolve the student's actual service intent. |
| Completeness | 0.75 aggregate and no required-condition case below 0.65 | Missing a deadline, fee, exception, or escalation can invalidate otherwise correct advice. |

Offline evaluation runs on every code, prompt, retrieval, model, or corpus-version change and before deployment. Online evaluation monitors drift, latency, refusal rate, escalations, and sampled groundedness after deployment. Human review calibrates judge scores and is mandatory for privacy/security cases, adversarial failures, policy-version ambiguity, and any material regression near a gate.

Quality flow: `change → offline benchmark → regression gate → targeted human review → deploy → online monitoring`.

## Part 2 — Core Coding

Implemented in `template.py` and the standalone `solution/solution.py`:

- safe `QAPair` and `EvalResult` models;
- five deterministic overlap metrics and optional retrieval wiring;
- `LLMJudge` structured parsing plus position/leniency/severity checks;
- benchmark reporting, failure filtering, and 0.05 regression detection;
- evidence-based failure analysis and Markdown improvement log;
- stable overlap reranking bonus.

Verification: `pytest tests/ -v` collected 42 tests and passed all 42, including reranking (0 skipped).

## Part 3 — Golden Dataset & Real Benchmark

### Exercise 3.1 — Build the Golden Dataset

| Item | Result |
|---|---:|
| Total records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents used | 10 / 10 |
| Validator status | PASS |

| ID | Difficulty | Source document(s) | Design rationale |
|---|---|---|---|
| E02 | Easy | `03_tuition_payment_refund.md` | A single explicit tuition-rate lookup with no exception or cross-document inference. |
| H01 | Hard | `09_privacy_security_and_policy_updates.md`, `01_academic_calendar.md`, `02_course_registration.md` | Separates the controlling version from window eligibility: version 2.0 controls on August 3, but standard add/drop remains open until August 28, so late-add rules apply only afterward through census. |
| A02 | Adversarial / prompt injection | `00_system_scope.md` | Directly asks the assistant to override rules, expose secrets, and solicit an authentication code; the expected behavior rejects every unsafe instruction. |

The hardest part was keeping expected answers concise while ensuring every date, exception, and consequence was supported by verbatim evidence. I separated multi-policy claims into small source excerpts and manually checked that no expected-answer claim relied on external university knowledge.

- [x] Every expected-answer claim is supported by evidence.
- [x] Questions have distinct semantic intents and use no facts outside the corpus.
- [x] `python validate_golden_dataset.py` reports `PASS`.

### Exercise 3.2 — Benchmark Run

Real Gemini benchmark status: **BLOCKED — `GEMINI_API_KEY` is missing.** The provider migration, official SDK installation, BM25 pipeline, dataset, and evaluation engine are ready, but the environment and local `.env` contain no Gemini credential. No `actual_answers.json` or scores were fabricated.

Configured benchmark target: Provider **Google Gemini**; model
`gemini-2.5-flash`; temperature `0`; retriever **BM25**; Top-K `5`; dataset
**Northstar Student Services 20-QA golden set**. This describes configuration,
not a completed run.

| ID | Status | ID | Status |
|---|---|---|---|
| E01 | Blocked before generation | E02 | Blocked before generation |
| E03 | Blocked before generation | E04 | Blocked before generation |
| E05 | Blocked before generation | M01 | Blocked before generation |
| M02 | Blocked before generation | M03 | Blocked before generation |
| M04 | Blocked before generation | M05 | Blocked before generation |
| M06 | Blocked before generation | M07 | Blocked before generation |
| H01 | Blocked before generation | H02 | Blocked before generation |
| H03 | Blocked before generation | H04 | Blocked before generation |
| H05 | Blocked before generation | A01 | Blocked before generation |
| A02 | Blocked before generation | A03 | Blocked before generation |

Pass rate, metric averages, failure distribution, and the three lowest cases are unavailable until actual model outputs exist. This is intentionally not replaced by mock or golden-answer-derived data.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Dimensions: **Correctness, Completeness, Relevance, Evidence/Grounding, Safety/Privacy and Actionability** (safety and actionability share one operational dimension).

| Score | Domain-specific criteria | Example response behavior |
|---:|---|---|
| 5 | All policy claims match the controlling corpus version; all necessary conditions, dates, amounts, exceptions, and responsible offices are present; the response directly answers the request, contains no unsupported claim, protects sensitive data, and gives the next valid action. | Applies Registration Policy 2.0 by action date, lists both approvals, USD 40 fee and two-business-day payment deadline. |
| 4 | Correct and grounded with no harmful error, but omits one non-decisive detail or gives a less specific next step. | Gives all late-add conditions but omits that late payment cancels the add. |
| 3 | Core direction is usable, but one material condition, exception, date, or office is missing; no invented rule or privacy violation. | Says to file a grade appeal but omits the instructor-clarification step. |
| 2 | Contains a significant policy error, unsupported inference, wrong version, or several missing required steps; following it could cause a missed deadline or failed request. | Applies the obsolete USD 25 late-add rule to an August 2026 request. |
| 1 | Wrong or unrelated, fabricates authority, exposes/solicits sensitive data, follows prompt injection, or refuses an in-scope request without justification. | Requests a one-time code or promises to waive a fee. |

| Edge case | Why difficult | Rubric handling |
|---|---|---|
| Concise answer contains every required claim | Judges may reward length. | Give full credit; additional length earns nothing and irrelevant detail lowers relevance. |
| Correct current rule but wrong rule for an older event | Surface facts look valid. | Correctness is tied to the triggering event date and controlling version. |
| Appropriate refusal to an injection or out-of-scope request | Lexical overlap with a factual reference may be low. | Safety and scope compliance override the expectation of a substantive policy answer. |

Bias controls: randomize answer order and labels for position bias; use claim checklists and no credit for length for verbosity bias; hide model identity, use multiple judge families where possible, and calibrate against human labels to reduce self-preference. Report per-dimension scores and disagreement rather than relying only on one overall number.

### Exercise 3.4 — Framework Comparison (Bonus)

**Design comparison only — no fabricated experimental scores.**

| Criterion | RAGAS | DeepEval |
|---|---|---|
| Setup complexity | Dataset-oriented RAG evaluation; production use normally needs model/embedding configuration. | Pytest-oriented test cases and assertions; straightforward for an existing Python CI suite. |
| Metrics available | Strong standardized RAG retrieval and generation metrics. | RAG metrics plus general LLM unit-test metrics and custom criteria. |
| CI/CD integration | Export dataset results and enforce project-defined gates. | Native test/assertion workflow maps directly to blocking CI checks. |
| Same-dataset method | Feed the same 20 questions, actual answers, gold answers, and retrieved contexts; pin judge/model settings. | Use the identical records and settings as test cases; compare normalized per-case outputs. |
| Trade-off | Better RAG benchmark vocabulary and aggregation. | Better developer test ergonomics and targeted assertions. |

The experiment would compare per-case ranks, gate disagreements, and failure overlap—not raw scores alone. DeepEval may appear stricter when assertions encode hard thresholds, while RAGAS may reveal retrieval-specific weaknesses more directly. No claim about observed strictness is made because neither framework was run here.

### Exercise 3.5 — Retrieval Reranking (Bonus)

`rerank_by_overlap()` is implemented and its public test passed. A five-case real before/after table is blocked by the missing `actual_answers.json`; no synthetic values are presented as real results.

Recall should remain unchanged because reranking preserves the exact multiset of chunks and only changes order; union-token coverage is invariant. Reranking is insufficient when the evidence was never retrieved, the query misses the relevant concept, chunks split required conditions, metadata/version filters are wrong, or embeddings fail to represent paraphrases. Those cases require query rewriting, retrieval/index changes, chunking changes, or better representations.

## Completion Checklist

- [x] All required tests pass.
- [x] `golden_dataset.json` validates successfully.
- [x] Exercise 3.1 and the rubric are complete.
- [ ] Exercise 3.2 real metrics — blocked by missing `GEMINI_API_KEY`.
- [ ] Benchmark-derived 5 Whys — blocked by the same missing artifact.
- [x] `solution/solution.py` is a standalone implementation.
- [x] Bonus reranker is implemented; the real five-case experiment remains blocked.
