# 3.6 Cross-Cutting Concerns

**Status:** Proposed controls and targets; privacy and platform approvals remain open.  
**Related:** [Assumptions and Open Questions](01-assumptions-open-questions.md), questions 3 and 5–9.

## Security and Identity

Use Entra OIDC for workforce sign-in, MFA/Conditional Access per corporate policy, and short-lived tokens. API authorization combines role and resource scope: employees manage their own plates/releases/bookings; facility managers are restricted to assigned sites; security reads only the site's daily access list; administrators receive least privilege. Validate issuer, audience, tenant, scopes, and ownership on every API operation. Use managed identity, Key Vault, TLS, encryption at rest, private endpoints where required, and no secrets in source/pipeline variables. Complete threat modelling, OWASP ASVS L2 control mapping, dependency review, and an independent pre-go-live pen-test; remediate critical/high findings before production.

## GDPR and Data Retention

Complete a DPIA and works-council review before pilot. Document purpose, lawful basis, access, data-subject rights, processors, cross-border transfers, and retention. Keep only the selected plate on a booking and the minimum daily list needed by security. No continuous location tracking, raw plate imagery, or person-level occupancy analytics. Restrict plate access, mask it in admin views where possible, and audit reads/exports without copying plate values into audit text.

The architecture response offers provisional retention values for DPO review only: check-in/arrival event details 30 days; booking-identifying data 90 days after service date; security/audit metadata 12 months; de-identified aggregate occupancy for trend reporting. These are not approved policy. Implement no retention schedule until DPO/legal/security/works-council owners approve it. Define deletion across replicas, backups, logs, dead-letter queues, exports, legal holds, and restored backups.

## Availability, Resilience, and Scalability

Assignment targets: 99.5% during business hours per site time zone; search and booking under one second p95; 8,000 potential users, approximately 2,500 daily active, and opening-window/07:00–09:00 peaks. Define measurement boundaries and test realistic load plus contention.

Proposed hosting uses two or more API instances, zone redundancy where supported/affordable, SQL backups and point-in-time restore, health probes, graceful shutdown, bounded queues, dead-letter alerts, dependency timeouts, and circuit breakers. RPO 15 minutes/RTO 4 hours are provisional targets only. A single-region MVP requires recovery after regional outage; future barrier decisions must be local during cloud loss. Bookings fail closed if SQL is unavailable; stale search cannot grant a booking. Keep HR/Graph out of synchronous booking transactions and retry notification delivery independently.

## Deployment, Testing, and Operations

Separate dev/test/stage/prod identities and environments; promote immutable scanned artifacts. Review backward-compatible migrations; test SQL constraints, isolation, and concurrency. Feature flags stage pilot rollout but never replace authorization. Maintain change/release checklist, support contact, rollback/forward-fix plan, and runbooks.

Test domain policy with unit tests; SQL uniqueness/isolation/concurrency with integration tests; API auth/errors with contract tests; release-book-check-in-cancel end-to-end with Playwright; localization/accessibility/DST boundaries; opening-window peaks with k6; and restore/failure behavior for SQL, queue, Graph, and HR. Pen-test and privacy approval are release gates.

## Indicative Monthly Run Cost

| Service | Planning range (EUR/month) |
|---|---:|
| App Service production capacity (2 instances) and non-prod footprint | 350–850 |
| Azure SQL Database, backups, and storage | 400–1,000 |
| Static hosting, registry, queue, functions/background work | 50–250 |
| Key Vault, App Configuration, networking/private endpoints | 100–450 |
| Application Insights / Log Analytics ingestion and retention | 100–350 |
| **Indicative total** | **1,000–2,900** |

This is a planning envelope, not a quote. Region, redundancy, SQL tier, networking, log volume, non-production/shared services, tax, and discounts may materially change it. Validate against the approved landing zone, keep budget headroom, alert on spend/ingestion, and review pilot actuals. Do not weaken privacy or availability controls to fit an unvalidated estimate.
