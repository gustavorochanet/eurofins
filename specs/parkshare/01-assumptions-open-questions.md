# ParkShare: Assumptions and Open Questions

**Deliverable:** Architecture Challenge, Section 3 / Deliverable 1  
**Status:** Assessment / discovery draft  
**Date:** 2026-11-20  
**Product Owner / Service Owner:** Eurofins  
**Architect:** Gustavo Rocha  
**Project reference:** None; assessment only

This document captures what the assignment states, what is assumed to make an architecture proposal, and the ten highest-value stakeholder questions. The assignment is the source for stated business constraints; the companion [ParkShare architecture challenge response](../parkshare-architecture-challenge.md) contains the proposed design. Proposed choices below are not approved policy or implementation facts.

## Sourced Facts and Constraints

- Contoso Group has 12 office sites in six countries, approximately 8,000 employees, and approximately 1,500 owned or rented parking spaces. Around 900 spaces are permanently assigned; the remainder are first-come-first-served.
- The assignment estimates that approximately 40% of assigned spaces are empty on an average day. Facility managers currently manage requests using email and spreadsheets.
- MVP is targeted for four months. The first release must go live at two pilot sites before rollout to all sites. Sprints are two weeks.
- The team is one Product Owner, one Application Architect at approximately 50% hands-on, two senior .NET developers, one web frontend developer, one full-stack/mobile-capable developer, one QA/test-automation engineer, and a DevOps engineer shared at 0.5 FTE.
- MVP requires corporate SSO using OIDC/Entra ID; employee profiles with one or more licence plates; facility-managed sites, slots, and attributes; owner release; site/date search and booking without double booking; site-specific booking window, weekly quota, and priority rules; cancellation and automatic no-show release; email and push notifications; daily authorised-plate lists for security; and a basic occupancy dashboard.
- Slot attributes include EV charging, accessible, motorbike, and compact; the assignment also refers to size.
- Half-day granularity and Teams notifications are nice-to-have, not core MVP requirements.
- Future scope includes LPR/barrier integrations from at least two vendors, Outlook calendar/out-of-office release, waiting list/fair allocation, visitor parking, EV session booking/billing, and external-customer SaaS/private paid sharing.
- Target load is 8,000 potential users and approximately 2,500 daily active users, with peaks from 07:00–09:00 local time and when the booking window opens.
- Availability target is 99.5% during business hours per site time zone. Future barrier integration must work if cloud connectivity is unavailable.
- Search and booking target is under one second at p95.
- Security requirements include corporate OIDC SSO, role-based access, OWASP ASVS Level 2, and a pen-test before go-live.
- Licence plates and presence data are personal data; data minimisation and retention limits are required.
- English, German, and French are required at minimum; the service is multi-time-zone.
- Microsoft Azure is the corporate hosting standard; Azure DevOps and central logging are named; on-premises hosting is allowed only at sites for edge devices.
- In-house skills include .NET/C#, Angular or React, and SQL Server; mobile skills are limited.
- MVP run-cost target is below EUR 3,000 per month.
- HR employee/cost-centre data is available through an existing REST API; email is via Exchange Online / Graph.
- Developers use GitHub Copilot, including agent mode; management expects productivity while guarding against inconsistent quality, security, and architecture erosion.

## Planning Assumptions

| ID | Assumption for the proposal | Why it is reasonable / boundary |
|---|---|---|
| A1 | ParkShare is an internal Eurofins/Contoso assessment for one corporate tenant. | External-customer SaaS is explicitly future scope. Carrying a tenant identifier is a design proposal; do not build SaaS onboarding, billing, or tenant self-service in MVP. |
| A2 | The first client is a responsive web application/PWA; no native iOS/Android app is planned for MVP. | The brief requires web and mobile access and notes limited mobile skills. Browser push support must be verified; email remains the fallback. |
| A3 | MVP booking and release granularity is one full local site day. | Half-day granularity is described as nice-to-have. Store site service dates in the site's local time zone and event timestamps in UTC. |
| A4 | A release makes an assigned owner's slot available without transferring permanent ownership. | This follows the owner-release use case. Until the first-come-first-served model is clarified, do not infer that every unassigned space is bookable through the same release workflow. |
| A5 | A user may register multiple plates and selects one plate for each booking. | This is directly required by the profile scope. Plate verification, shared vehicles, and plate changes after booking are not defined. |
| A6 | For architecture planning only, authenticated PWA check-in is the proposed MVP no-show signal; 10:00 site-local time is a candidate cutoff. | The assignment gives “not confirmed / car not seen by 10:00” as examples but does not choose a detection source. Do not enable automatic release until a site approves the signal, cutoff, grace period, appeal, and override process. |
| A7 | Booking policy is configured per site and evaluated by the server at booking commit. | The assignment requires site-specific rules. Priority groups, quota counting, week boundary, fairness, and tie-breaking remain business decisions. |
| A8 | Azure SQL is the proposed booking system of record, with a relational uniqueness constraint as the final no-double-booking guard. | SQL Server is an in-house skill and booking requires an atomic invariant. This is a proposed technology choice, not a platform approval. |
| A9 | A modular monolith and managed Azure compute are the initial architecture proposal. | The team and four-month MVP favor a small operational footprint. Exact hosting, worker runtime, database tier, queue, region, and recovery topology require landing-zone validation. |
| A10 | HR is treated as the proposed authoritative employee reference source; ParkShare copies only fields needed for operation. | The brief requires HR master-data integration but does not state source-of-truth ownership, sync cadence, or stale-data behavior. Confirm with HR/integration owners. |
| A11 | Email and push are MVP channels; Teams, calendar automation, LPR, visitor workflow, waiting list, EV billing, and SaaS are deferred. | Email and push are required; Teams is a nice-to-have and the other items are post-MVP scope. Push/browser behavior and notification content still need validation. |
| A12 | Do not retain raw LPR images in the proposed platform; future vendor events should be normalised behind an adapter. | This minimises personal data and isolates vendor differences, but final data flow and retention require privacy/security and vendor review. |
| A13 | A single Azure region is acceptable for the MVP proposal; the business-hour 99.5% target and future local barrier continuity are separate concerns. | Active-active multi-region is not specified and may exceed the budget. Operations must approve region, measurement, recovery targets, and cost. |
| A14 | Any suggested retention periods (30 days for check-in/arrival details, 90 days for booking-identifying data after service date, 12 months for security/audit metadata) are placeholders for DPO review only. | These figures originated in the architecture proposal, not the assignment or an approved policy. Do not implement them until approved. |
| A15 | The EUR 3,000/month figure is a planning cap for the MVP run-cost discussion, not a validated quote. | Region, redundancy, licensing, non-production, shared services, log ingestion, tax, and discounts are unspecified. |
| A16 | Facility dashboards should prefer aggregate occupancy data and avoid individual attendance analytics. | This aligns with the works-council data-minimisation requirement; metric definitions and any exceptions need approval. |

## Top 10 Stakeholder Questions

1. **Slot inventory and ownership:** How are permanently assigned and first-come-first-served spaces represented and made bookable? Can an owner withdraw a release after someone books it? Who resolves conflicts, and what happens when an assigned slot changes or becomes unavailable?
2. **Fair allocation and booking policy:** What are the eligible groups, priority order, tie-breaker, booking window, and weekly quota? Is the quota counted by booking, booked day, or attendance day, and how do cancellations/priority bookings affect it?
3. **No-show policy:** Is check-in acceptable, or must the system observe the vehicle? What are the signal, site-local cutoff, grace period, reminder, appeal, manual override, and behavior for late or erroneous arrival events?
4. **Pilot sites and local rules:** Which two sites pilot first? What are their IANA time zones, business hours, holidays, shift patterns, accessibility rules, slot categories, and site champions?
5. **Identity and HR integration:** What are the Entra app/claim/group mappings and HR API contract, identifiers, fields, authentication, rate limits, freshness expectations, deprovisioning event, and behavior during HR/Entra outages?
6. **Privacy and governance:** What lawful basis, DPIA, works-council approval, data classification, retention/deletion schedule, backup expiry, access roles, data residency, and data-subject request process apply to plates, bookings, check-ins, exports, and audit records?
7. **Security and reception workflow:** Who receives the daily authorised-plate list, in what format/channel and at what refresh time? How are incorrect plates corrected, visitors handled, exports audited, and site outages managed?
8. **Barrier/LPR roadmap:** Which two vendors and protocols are likely? Who owns site hardware and support? What are the permitted offline snapshot age, revocation behavior, local decision rule, manual override, event replay, and fail-open/fail-closed policy?
9. **Availability, platform, and cost:** Which Azure region/services and network controls are approved? How is 99.5% measured per site, what are RPO/RTO, what peak load/headroom is required, and which production/non-production/shared-service costs count toward EUR 3,000/month?
10. **Localization and notifications:** Confirm minimum languages (assignment says EN/DE/FR), translation owner, email/push delivery expectations and fallback, supported mobile browsers, local date/time formats, and whether Teams is excluded from MVP.

## Source Gaps to Confirm

No external architecture, HR, Entra, Azure landing-zone, DPO/works-council, site-operating, or barrier-vendor documents were supplied with the assignment. Confirm whether these sources exist and provide owners/links; do not treat their absence from this workspace as proof that no documentation exists.

The companion [assessment-stage service context](../../.github/service-context.md) records the current proposal and points to deferred stakeholder decisions. Revisit both documents as decisions are made; neither represents implemented or production-approved behavior.
