# ADR-0002: Database sync
Status: Accepted
Date: 2026-10-22

## Context
Nerdawy answers from platform data (P1), so every source it relies on must be synced into the platform: stock from stores and warehouses, holds, the catalog, reviews, and the retrieval index. Stale data in any of them means confident but wrong answers, and a wrong "in stock" wastes a user's trip (P4).

These sources change at very different speeds. Stock can change within minutes, while the catalog and reviews change over days or weeks. Stores also differ in capability: some have a POS system or API, some keep a spreadsheet, and some have no system at all.

## Decision drivers
1. Freshness of the data Nerdawy answers from (P1).
2. Reservations without conflicts, so a held copy isn't sold to someone else (P4).
3. What each store can realistically support.
4. Build and operating effort.

## Options considered
### A. Batch everything
**Pros:**
1. Simple: one sync schedule for all data and all stores.

**Cons:**
1. Stock is only as fresh as the slowest batch, so Nerdawy often reports outdated stock (P1).
2. Holds can conflict with copies sold in between syncs (P4).

### B. Real-time everything
**Pros:**
1. Freshest possible data.

**Cons:**
1. Most stores can't support it; small stores have no system to integrate.
2. Highest build and operating effort, including for data that changes weekly.

### C. Tiered by data type and store capability
**Pros:**
1. Each data type syncs as often as it actually changes.
2. Each store syncs as fast as it can, and users are told honestly what to expect.

**Cons:**
1. Several sync paths and tiers to build and maintain.
2. Lower tiers give users a slower or less certain reservation experience.

## Decision
We choose Option C.

**By data type:**
- **Stock:** synced in less than 5 minutes for Tier 1 stores, so stock is always updated before holds.
- **Holds:** Nerdawy does not make real-time reservations. Orders are batched every 5 minutes and executed in timestamp order to resolve conflicts. The user is updated by email, or can check their account for the result.
- **Catalog:** updated with the bookstores every week.
- **Reviews:** synced every week or month, since they change slowly.
- **Retrieval index:** only catalog descriptions and reviews are free text and need it; everything else is tabular and queried directly. The index is rebuilt after each catalog and review sync.

**By store tier:**
- **Tier 1 (POS or API):** synced every 5 minutes; reservations go through the 5-minute hold batch.
- **Tier 2 (daily upload):** the user can choose to call the store and book with their account ID, or wait a day for our confirmation.
- **Tier 3 (no system):** calls only, or the user places an order and we let them know whether it was received successfully.

## Consequences
**Positive:**
- Stock and holds stay fresh where stores can support it.
- Hold conflicts are resolved deterministically by timestamp.
- Weekly data isn't synced more often than needed.

**Negative:**
- Reservations are pending for up to 5 minutes, even at Tier 1.
- Tier 2 and Tier 3 users get a slower or manual path.
- Several sync paths, tiers, and an index rebuild to maintain.

**Risks accepted:**
- A Tier 2 copy may sell before the next daily sync; the user learns this only at confirmation.
- Stock shown for lower tiers is older, so Nerdawy must show each store's "last updated" time (P1).

**Sets:** P1's staleness threshold per tier: 5 minutes for Tier 1, one day for Tier 2; Tier 3 stock is not shown as confirmed.

## Revisit when
- Enough Tier 2 or Tier 3 stores adopt a POS or API to move up a tier.
- Users often lose holds to timestamp conflicts, suggesting a shorter batch interval.
- Catalog or review data changes faster than the weekly sync.