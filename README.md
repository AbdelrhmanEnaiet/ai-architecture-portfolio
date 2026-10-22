# AI Architecture Portfolio

An AI solution architecture for **Khaznet Kotob**, an Egyptian book-broker platform, and **Nerdawy**, its bilingual (Egyptian Arabic / English) AI chat agent. The portfolio is built session by session: governance first, then building blocks, ending in a capstone architecture.

## Scenario

Khaznet Kotob aggregates stock from bookstores and warehouses so users can find any book, order delivery, or locate the nearest physical copy. Nerdawy helps users:

1. **Search and order**: find books, reserve them for pickup, or request ones that aren't available.
2. **Check compatibility**: read trusted and user reviews, and check a book against the user's preferences.
3. **Get recommendations and alerts**: discover new books, and get notified (with a temporary pre-booking) when an unavailable book comes back in stock.

## Key constraints

- **Egyptian PDPL (Law No. 151/2020)**: personal data stays in Egypt; only PII-scrubbed conversations without user IDs reach external LLM providers.
- **Arabic, English and mixed-language** understanding is required, which currently favours external frontier models over self-hosting.
- **Grounded answers only**: Nerdawy answers from platform data, shows stock freshness, or says it doesn't know.
- **Targets**: faithfulness ≥ 95%, answer relevance ≥ 90%, time to first token p50 < 1s / p95 < 5s, 99.9% availability.

## Reading guide

1. Start with the [governance pack](docs/governance-pack.md): business goals, principles P1–P5, the ADR summary, and the ARB checklist.
2. Look at the [C2 container diagram](diagrams/C2-diagram.png) for the system layout and the Egypt data boundary.
3. Read the ADRs for the reasoning behind each decision:

| ADR | Decision | Status |
|-----|----------|--------|
| [0001 LLM hosting and routing](adr/0001-llm-hosting-and-routing.md) | External API via an adaptor layer, with primary + fallback providers | Accepted |
| [0002 Database sync](adr/0002-database-sync.md) | Sync tiered by data type and store capability | Accepted |
| [0003 PII scrubbing](adr/0003-pii-scrupper.md) | Regex + NER + catalog allow-list, CPU-only inside Egypt | Accepted |
| [0004 Evaluation](adr/0004-valuation-method.md) | DeepEval for standard metrics + pure Python for scrubber metrics | Accepted |

## Progress

| Session | Deliverable | Status |
|---------|-------------|--------|
| 1 | Governance pack, ADRs 0001–0004, C2 diagram | Done |
| 2 | Building blocks | Not started |
| Capstone | Final architecture PDF and slides | Not started |

## Repository structure

```text
├── README.md                          # Scenario, constraints, and reading guide
├── docs/
│   └── governance-pack.md             # Session 1: goals, principles, ARB checklist
├── diagrams/
│   ├── C2-diagram.png                 # C2 container diagram (export)
│   └── C2-diagram.xml                 # draw.io source
├── adr/                               # One file per architecture decision
│   ├── 0001-llm-hosting-and-routing.md
│   ├── 0002-database-sync.md
│   ├── 0003-pii-scrupper.md
│   └── 0004-valuation-method.md
└── capstone/                          # Final architecture PDF and slides
```
