# ADR-0004: Evaluation method
Status: Accepted
Date: 2026-10-22

## Context
P1, P2, and ADR-0003 set measurable targets: faithfulness ≥ 95%, answer relevance ≥ 90% with a refusal cap, and PII scrubber recall and false-positive rates. P3 requires both LLM providers to pass the same evals. We need a way to measure all of these across English, Arabic, and mixed-language messages.

Because the eval judges will be external frontier models, the golden dataset must not contain actual user data, or the evals themselves would break P5.

## Decision drivers
1. Covers relevance, faithfulness, and PII scrubber metrics (P1, P2, P5).
2. Works on English, Arabic, and mixed-language messages.
3. No real user data reaches external judge models (P5).
4. Effort to build and maintain.
5. Uses industry-standard metrics where possible.

## Options considered
### A. Pure Python
**Pros:**
1. Full control over every metric, including the PII scrubber's recall and false positives.
2. No framework dependency.

**Cons:**
1. Faithfulness and relevance still need a judge model, which we would wire up and validate ourselves.
2. More code to build and maintain.

### B. DeepEval
**Pros:**
1. An industry standard, with ready-made faithfulness and answer relevancy metrics.
2. The judge model is configurable per metric.

**Cons:**
1. Its PII leakage metric checks whether the LLM's output leaks PII. It does not measure our scrubber's recall or false positives against labeled data.

### C. DeepEval + pure Python
**Pros:**
1. DeepEval covers faithfulness and relevance with standard metrics.
2. Pure Python covers the PII scrubber as we defined it: recall and false positives against labeled examples.

**Cons:**
1. Two eval paths to maintain.

## Decision
We choose Option C: DeepEval for relevance and faithfulness, and pure Python for the PII scrubber metrics.

## Golden dataset
- Split into three subsets: English, Arabic, and mixed language.
- About 100 samples per subset at launch.
- We write 10 examples per subset; an LLM synthesizes the rest; humans review and curate them.
- Each subset also includes 10 human-created malicious samples (e.g. prompt injection, attempts to extract other users' data).
- No actual user data at launch. PII examples use fake names, phone numbers, and IDs.
- Targets must hold on each subset, not just on average.
- Three months after launch, the set is expanded with real conversations: human reviewers curate them and replace all sensitive information with fake values before they enter the set. Only conversations from users who consented to quality review are used, retained in Egypt until curated (P5).

## LLM judge
- The judge must be from a different model family than the provider being evaluated.
- Before use, the judge is benchmarked against our own human labels on the 10 hand-written examples per subset.
- The benchmark set is expanded after three months, alongside the golden dataset.

## Consequences
**Positive:**
- Standard metrics where they fit, custom code only where needed.
- External judges never see real user data.
- Language-specific weaknesses show up per subset.
- Malicious samples test Nerdawy's resistance to prompt injection and data-extraction attempts.

**Negative:**
- Two eval paths to maintain.
- Human curation takes time, and reviewers must be fluent in Egyptian Arabic.

**Risks accepted:**
- At launch, synthetic data may not match how real users write; the 3-month expansion closes this gap.
- With about 100 samples per subset, each failure moves a score by 1%, so results near the targets are only roughly measured.
- A reviewer may miss sensitive information during curation, which would then reach the external judge.
- The judge benchmark starts small (10 examples per subset), so early judge validation is coarse.

## Revisit when
- Real usage shows patterns the golden set misses.
- DeepEval adds a metric that fits our scrubber evaluation.
- Scores sit close to a target, needing a larger subset to judge.
- The judge's agreement with human labels drops.