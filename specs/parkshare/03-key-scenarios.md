# 3.3 Key Scenarios

**Status:** Proposed flows; no implementation exists.  
**Related:** [Assumptions and Open Questions](01-assumptions-open-questions.md), especially questions 1–3 and 7–8; [proposed booking API contract](booking-api.yaml).

## Owner Releases, Seeker Books, Vehicle Arrives

For MVP, the flow ends with a site-scoped security list and employee PWA check-in. Direct barrier control is future scope; its local decision path below is illustrative only.

```mermaid
sequenceDiagram
    actor Owner
    actor Seeker
    participant UI as ParkShare PWA
    participant API as ParkShare API
    participant DB as Azure SQL
    participant Worker as Background worker
    participant Security as Security user / daily list
    participant Edge as Site edge adapter (future)
    Owner->>UI: Release assigned slot for local date(s)
    UI->>API: POST release request
    API->>API: Verify identity, slot ownership, site policy
    API->>DB: Save release and audit record
    DB-->>API: Commit
    API-->>UI: Release confirmed
    Seeker->>UI: Search site and date
    UI->>API: GET availability
    API->>DB: Query eligible released allocations
    DB-->>API: Available slots
    Seeker->>UI: Book slot and choose registered plate
    UI->>API: POST booking + idempotency key
    API->>DB: Atomic policy, quota and unique-allocation transaction
    DB-->>API: Commit or conflict
    API-->>UI: Booking confirmed or 409 unavailable
    API->>DB: Outbox event committed with booking
    Worker->>DB: Read outbox and send notification
    Security->>UI: Request today's site authorisation list
    UI->>API: Site-scoped list request
    API->>DB: Query active bookings and selected plates
    API-->>Security: Minimal, timestamped daily list
    Seeker->>UI: Check in by site cutoff
    UI->>API: POST check-in
    API->>DB: Record check-in and audit
    Note over Edge: Future: edge syncs signed, expiring allowlist before arrival
    Seeker->>Edge: Vehicle arrives at barrier
    Edge->>Edge: Match plate against cached local allowlist
    Edge-->>Edge: Grant or deny locally and queue event for later sync
```

## No Double Booking Under Concurrent Load

Availability displayed to a user is advisory. Only the booking transaction grants the slot; SQL is the final arbiter, including at booking-window opening.

1. Each requested slot/date/period has a `BookingAllocation` with an `IsActive` flag. A filtered unique index on `(TenantId, SlotId, LocalDate, Period) WHERE IsActive = 1` enforces one active allocation. Cancellation/no-show deactivates the allocation in the same transaction that changes the booking.
2. In one SQL transaction, recheck release state, booking window, employee eligibility, site policy, and quota. Multi-date requests reserve all rows or roll back all rows; use short transactions and stable lock ordering.
3. Require a caller-scoped idempotency key. A retry returns the prior outcome, not a second booking. Bound retries to transient failures and idempotent commands.
4. Translate a unique-index conflict into a stable `409 Conflict`; do not retry genuine unavailability. Test races against SQL and assert exactly one active allocation.
5. Load-test forecast peak plus agreed headroom. Tune indexes and API/connection scaling before adding cache or queueing. Client-side locks and cached search results are not correctness controls.

For weekly quotas, update/check a `UserSiteWeekQuota` row in the same transaction or use isolation/locking that prevents oversubscription. The week boundary and treatment of cancellations are unresolved; see [3.1 question 2](01-assumptions-open-questions.md).

## No-Show Detection and Release

Proposed MVP assumption: authenticated PWA check-in is the arrival signal. At each site's configured local cutoff (10:00 is a candidate, not an approved value), an idempotent scheduled job selects eligible active bookings without check-in/valid arrival, transitions the booking to `NoShow`/`Released`, deactivates the allocation, records reason and policy version, and queues notification. A conditional update prevents overlapping/retried jobs from releasing twice. Facility-manager corrections are audited.

Enable automatic release only after the site approves the signal, cutoff, grace period, reminders, appeals, and override process. “Car not seen” cannot be inferred without LPR, barrier, or a manual security signal. Future vendor events require deduplication and explicit late/offline-event rules; see [3.1 questions 3 and 8](01-assumptions-open-questions.md).
