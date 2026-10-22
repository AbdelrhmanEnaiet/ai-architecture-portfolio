# ADR-0001: LLM hosting and routing
Status:Accepted
Date: 2026-10-22

## Context
Nerdawy Need an LLM layer to understand Arabic, English and Mixed language between them. retrive, compare and recommend new books and sext steps based on the books, user data and available options the user can follow

Due to unavailability of frontier LLM models hosted inside Egypt. We can either use an LLM through external APIs. or host a local LLM and avoid exiting the data outside the country. 
P5 Support the Local hosting. but P1 and P2 would beinifit more from the External API models due to stronger Arabic Language understanding and P3 benifits from their availability and lower operating effort


## Decision drivers
1. The cost at the expected launch traffic size. 
2. System Latency on the evaluation dataset.
3. PDPL privacy compliance.
4. Arabic and mixed language quality understanding
5. Availability Support for 99.9% including failover.

## Options considered
### A. Local Hosted Model
**Pros:**
1. Predictable cost: fixed hardware and operations cost, cheaper per conversation at high traffic.
2. No personal data crosses the border at all, so no cross-border permit and no dependence on the PII scrubber being perfect.

**Cons:**
1. High floor: the initial costs will be high until the system traffic breaks even with an external API service.
2. Language understanding: open-weight models that understand Arabic and mixed language have large sizes and do not generally reach the frontier's level. They also require pricey GPUs and handling.
3. Traffic handling and availability: load balancing for requests will be the system's responsibility, as will scaling up and down ourselves.
4. Requires an experienced team to develop the infrastructure around it.

### B. External API Direct Connection
**Pros:**
1. Using the provider SDK allows easier and quicker integration and testing.
2. Model serving and scaling are handled by the provider.
3. Payment per use, with no upfront costs.
4. Nerdawy gets the frontier models' language understanding.
5. Cost per call can be tracked.
6. Latency on p50 can be better than a locally hosted model.

**Cons:**
1. Cost increases with traffic scale and the agent's extra steps.
2. A third party is another point of failure that needs handling.
3. Requires a strong PII scrubber, a cross-border permit, and zero-retention agreements for PDPL compliance.
4. Latency on p95 can be worse than a locally hosted model.
5. Dependent on a specific vendor, and can be affected by rate limiting and behaviour changes.

**Risk note:** the PII scrubber can sometimes miss important details, which can then reach the provider.

### C. External API through a Provider Adaptor Layer
Everything in B applies, plus:

**Pros:**
1. Provider-agnostic: Nerdawy can switch between external providers, or move to a locally hosted model, without being rewritten.
2. Enables P3's failover: routing to a second provider is a configuration change.
3. Removes vendor lock-in, which cancels B's Con 5.
4. Migration path: model serving and scaling are handled by the provider until traffic reaches break-even with local hosting, or regulations tighten. A's PDPL advantage stays available later without paying its upfront floor now.

**Cons:**
1. An extra layer to build and maintain, adding a little latency and one more component that can fail.
2. Lowest common denominator: provider-specific features (prompt caching, tool-calling formats, structured outputs) differ, so the adaptor may hide or limit them.
3. Switching isn't free: each backend needs its own evals and prompt tuning, and moving local still requires A's GPUs and experienced team.
## Decision
We choose Option C because it allow us the benifits of option B with beter cons and allow us control to switch to option A when it is needed in the future.

## Consequences
**Positive:**
- No upfront hardware cost; we pay per use from launch.
- Failover to a second provider is a configuration change, supporting P3's 99.9% availability.
- No lock-in to a single vendor.

**Negative:**
- We must build and maintain the adaptor layer.
- Both providers must pass the same evals (P1, P2), doubling eval and prompt-tuning work.
- Some provider-specific features may be unavailable through the adaptor.

**Risks accepted:**
- The PII scrubber may miss some personal data, which would then reach the provider (mitigated by zero retention; see ADR-003).
- Cost grows with traffic and agent steps.

**Sets:** P3's cost cap of X EGP per conversation, measured on the eval set at expected launch traffic.

## Revisit when
- Monthly API cost reaches the break-even point with self-hosting.
- An open-weight model meets our P1 and P2 eval targets in Egyptian Arabic.
- PDPL guidance on cross-border transfers changes.
- Provider latency repeatedly breaks P3's p95 target.