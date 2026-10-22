# ADR-0003: PII scrubbing method
Status: Accepted
Date: 2026-10-22

## Context
ADR-0001 sends live conversations to external LLM providers. P5 requires that what leaves Egypt is PII-scrubbed and carries no user IDs. Users type personal details freely: names, phone numbers, addresses, and national ID numbers, in Egyptian Arabic, English, or a mix of both in the same message.

The scrubber runs on every request, so it sits on P3's latency path. It must also run inside Egypt, or it defeats its own purpose. Our domain adds a twist: book conversations are full of person names (authors, characters) that must *not* be masked, or answers lose meaning and break P1 and P2.

## Decision drivers
1. PII recall across Egyptian Arabic, English, and mixed-language messages (P5).
2. Added latency per request (P3).
3. Cost per request, and whether GPUs are needed (P3).
4. False positives: masking authors, titles, or characters would harm answer quality (P1, P2).
5. Runs fully inside Egypt (P5).

## Options considered
### A. Regex and rules
**Pros:**
1. Near-instant and cheap; runs on CPU.
2. Deterministic and auditable.
3. Reliable for structured PII: Egyptian mobile numbers, 14-digit national IDs, emails, card numbers.

**Cons:**
1. Cannot detect names or free-form addresses.
2. Blind to context.

### B. NER model (self-hosted)
**Pros:**
1. Detects names and locations, not just patterns.
2. Runs on CPU in tens of milliseconds.

**Cons:**
1. Accuracy on Egyptian dialect and mixed-language messages is weaker than on English.
2. Cannot tell a user's name from an author's name, so it masks book content (driver 4).
3. Needs labeled data to tune.

### C. Small local LLM
**Pros:**
1. Best context understanding: separates "my name is Ahmed" from "books by Naguib Mahfouz."
2. Handles mixed-language messages better than NER.

**Cons:**
1. Needs GPUs in Egypt, reintroducing the hardware floor ADR-0001 avoided.
2. Adds hundreds of milliseconds or more per request, threatening P3.
3. Non-deterministic; may alter the user's text beyond masking.

### D. Layered: regex + NER + catalog allow-list
**Pros:**
1. Regex catches structured PII deterministically; NER catches names and places.
2. An allow-list built from our own catalog (authors, titles, characters) stops false positives on book content.
3. CPU-only, in-country, and low latency.

**Cons:**
1. Two components plus an allow-list to maintain, and the allow-list must stay synced with the catalog.
2. Still weakest on free-form PII in mixed-language messages.

## Decision
We choose Option D because it reaches good recall on CPU inside Egypt (P5) with low added latency (P3), and the catalog allow-list protects answer quality (P1, P2). Option C was rejected for its GPU cost and latency.

## Consequences
**Positive:**
- No GPUs needed; the scrubber runs on existing in-country infrastructure.
- Structured PII is removed deterministically and can be audited.
- Book names survive scrubbing, so answers stay grounded.

**Negative:**
- We must maintain regex patterns, the NER model, and the allow-list sync.
- We need a labeled PII test set covering Arabic, English, and mixed-language messages.

**Risks accepted:**
- Some free-form PII, especially in mixed-language messages, may reach the provider. Mitigated by zero retention, no user IDs, and storing preference tags instead of transcripts (P5).

**Sets:** scrubber latency ≤ 50 ms at p95 (P3 budget) and 100% recall on structured PII. Recall on free-form PII and the false-positive rate on catalog names are set after a baseline run on the PII test set, then reviewed quarterly.

## Revisit when
- Recall on the PII test set falls below target.
- A small model runs fast enough on CPU to replace NER.
- PDPL guidance requires stronger anonymization.
- ADR-0001 moves to local hosting, making external-path scrubbing unnecessary.