# 3.9 Risks and Mitigations

**Status:** Assessment risk register; owners and acceptance thresholds to be assigned.  
**Related:** [Assumptions and Open Questions](01-assumptions-open-questions.md).

| Risk | Likelihood / impact | Mitigation and trigger |
|---|---|---|
| No-show, priority, quota, or reclaim rules remain ambiguous | High / High | PO and facility/works-council decision log in Sprint 0; version policy and test scenarios; block automatic release until site policy is signed off. |
| Plate and presence processing is not accepted by DPO/works council | Medium / High | DPIA and purpose/retention review before pilot; minimise fields and roles; no raw imagery; do not launch without approval. |
| Booking-window burst causes latency or database contention | Medium / High | Race and k6 tests at forecast peak plus headroom; unique index and bounded transactions; tune before adding cache/queueing; alert on p95/errors. |
| HR/Entra/Graph access or API limits arrive late | Medium / High | Name owners, secure test tenant/contracts in Sprint 0; resilient adapters; define stale-profile behavior; escalate by Sprint 1 exit. |
| 99.5% target conflicts with EUR 3,000/month | Medium / High | Validate regional price, zone redundancy, private networking, and log ingestion early; cost alerts and pilot actuals; raise scope/funding decision. |
| Shared DevOps capacity delays environments and release | High / Medium | Reserve 0.5 FTE at kickoff; create standard IaC/pipeline templates in Sprint 0; escalate environment blockers within one sprint. |
| Pilot site data, training, or security process is not ready | Medium / Medium | Name site champions; validate inventory, rehearse daily list/support/rollback, and use staged rollout. |
| Future LPR offline mode creates stale authorisations/vendor lock-in | Medium / High | Defer until vendor/policy selected; signed expiring snapshots, local event queue, offline-age/override policy, and vendor failure tests before site activation. |

## Decision Gates

- **End Sprint 0:** policy owners/open-question plan, Entra/HR connectivity proof, deployment pipeline, cost estimate, and DPIA workstream.
- **Before booking acceptance:** SQL uniqueness/concurrency test, quota semantics, cancellation and reclaim rules approved.
- **Before pilot:** DPO/works-council approval, critical/high pen-test findings closed, site security process rehearsed, support/rollback/restore runbooks tested, p95 and cost measured.
- **Before full rollout:** pilot SLO, support burden, no-show false-positive rate, and privacy handling reviewed by PO, Operations, and site stakeholders.
