# 3.7 Delivery Plan and Work Breakdown

**Status:** Indicative assessment plan.  
**Related:** [Assumptions and Open Questions](01-assumptions-open-questions.md), especially questions 3–9.

## Epics, Features, Dependencies, and Size

| Epic | Features | Dependencies | Size |
|---|---|---|---|
| E1: Secure platform foundation | Entra sign-in, tenant/site authorization, CI/CD, telemetry, database migrations | Landing zone, Entra app registration, HR API access | L |
| E2: Sites and inventory | Site/slot CRUD, attributes, owner assignment, audit | E1; facility data and role model | M |
| E3: Release and discover | Owner releases; site/date search; local-time rules | E2; calendar and eligibility policy | M |
| E4: Book and manage | Atomic booking, weekly limits, cancellation, idempotency | E3; confirmed quota and priority semantics | L |
| E5: Daily operations | Check-in, no-show release, notifications, security plate export | E4; privacy/security approval; Graph access | L |
| E6: Reporting and rollout | Occupancy dashboard, performance/security hardening, pilot and staged rollout | E2–E5; site champions, pen-test | M/L |

## Example Stories and Acceptance Criteria

**Release an assigned slot.** As a slot owner, I can release my assigned slot for eligible future dates so another employee can use it. Acceptance criteria: only the authenticated owner or scoped facility manager can release; invalid/out-of-policy dates fail clearly; multi-date release is atomic; release/reversal is audited; search reflects release within five seconds.

**Book an available slot.** As an eligible employee, I can book a released slot and know immediately whether I succeeded. Acceptance criteria: API revalidates availability, eligibility, window, and quota at commit; two concurrent requests yield exactly one active booking and one conflict; repeating the same idempotency key returns the original result; notification is queued only after commit; errors disclose no other user's personal data.

**Check in and release a no-show.** As an employee, I can confirm arrival for today's booking; as a facility manager, I can configure cutoff and review exceptions. Acceptance criteria: check-in requires the employee's active booking at the correct site/date; reminder precedes cutoff; unconfirmed booking releases once only when site auto-release is enabled; employee is notified; manager correction is audited; site-local/DST boundaries are tested.

## Sprint and Release Plan (Eight Two-Week Sprints)

| Sprint | Outcome and exit evidence | Primary allocation |
|---|---|---|
| 0 | Walking skeleton: repo/pipeline, Bicep environments, API/PWA deploy, Entra proof, SQL migration, logging, threat model, policy/privacy workshops, cost estimate. | Architect + DevOps lead platform; backend pair; frontend skeleton; QA baseline; PO resolves policy questions. |
| 1 | Entra sign-in, tenant/site authorization, employee/plate profile, audit foundation; deploy to test. | Backend: identity/API; frontend: shell/profile; QA: auth/security tests; architect: boundaries/reviews. |
| 2 | Facility site/slot master data and ownership; HR sync contract stubbed and tested. | Backend: inventory/HR adapter; frontend: manager screens; QA: API/role tests; PO: pilot data. |
| 3 | Owner release and site/date search; local-time/holiday baseline. | Backend: release/search; frontend: owner/seeker flows; QA: boundary/DST tests. |
| 4 | Booking transaction, quota, idempotency, cancellation; concurrency test is a release gate. | Backend pair: booking; frontend: booking UX; QA: SQL contention suite; architect: invariant review. |
| 5 | Check-in, no-show job, notifications, security daily list, retention deletion job. | Backend: worker/Graph/security export; full-stack: PWA check-in; QA: failure/retry/privacy tests. |
| 6 | Dashboard, load/security/accessibility testing, pen-test remediation, pilot rehearsal; pilot go-live target at sprint end. | Whole team hardens; DevOps production readiness; PO/facility/security UAT; architect coordinates risks. |
| 7 | Observe pilot, fix blocking defects, measure cost/SLO, roll out in waves to remaining sites with rollback gates. | Developers swarm on defects; QA validates waves; DevOps monitors; PO/site champions approve readiness. |

This targets 16 weeks, pilot by week 12 and a four-week pilot/rollout window. Dates depend on timely identity, HR, policy, privacy, site data, and platform access. If a release-blocking risk appears, protect safety/privacy and reduce scope rather than waive a gate.

## Sprint 0, Team Allocation, and Scope Control

Sprint 0 is a production-shaped walking skeleton: authenticated request through pipeline; Entra/HR connectivity proof; SQL transaction and unique-index race test; module/API conventions; dashboards, alerts, environments, threat model, DPIA owners, and measured cost estimate. Record short ADRs and revise them when implementation evidence changes a proposal.

The two senior backend developers own identity/inventory and booking/integration, pairing on the transaction invariant. The frontend developer owns responsive flows/accessibility. The full-stack/mobile-capable developer owns PWA/check-in. QA owns automation, test data, concurrency and release evidence. Reserve the shared 0.5 FTE DevOps capacity early. Architect (50% hands-on) owns boundaries, ADRs, privacy/security design coordination, critical booking reviews, fitness functions, and cross-team decisions without becoming a delivery bottleneck. PO owns policy, acceptance, and stakeholder access.

If the date is at risk, defer half-day booking, Teams, advanced reports, calendar auto-release, and visual polish. Keep identity/access control, inventory correctness, no-double-booking, cancellations, approved no-show policy, security list, privacy/retention, accessibility basics, and production observability. Narrow pilot breadth before weakening an invariant or release gate.
