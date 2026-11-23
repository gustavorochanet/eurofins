# 3.2 Architecture Overview

**Status:** Proposed for assessment; no implementation exists.  
**Source:** [3.1 Assumptions and Open Questions](01-assumptions-open-questions.md) and [integrated response](../parkshare-architecture-challenge.md).

## System Context (C4)

```mermaid
flowchart LR
    employee[Employee: owner or seeker]
    facility[Facility manager]
    security[Security / reception]
    ops[IT Operations]
    hr[HR system]
    entra[Microsoft Entra ID]
    exchangeGraph[Exchange Online / Graph]
    barrier[Barrier and LPR vendors: future]
    parkshare[ParkShare]
    employee -->|Web/PWA: release, search, book, check in| parkshare
    facility -->|Manage site, slots, rules, reports| parkshare
    security -->|Read scoped daily authorised-plate list| parkshare
    ops -->|Operate, monitor, support| parkshare
    parkshare <-->|OIDC sign-in and role claims| entra
    parkshare -->|Read employee and cost-centre data| hr
    parkshare -->|Send transactional email| exchangeGraph
    parkshare -.->|Vendor-neutral adapter boundary; future| barrier
```

## Container View (C4)

```mermaid
flowchart TB
    users[Employees, facility managers, security]
    entra[Microsoft Entra ID]
    hr[HR REST API]
    exchangeGraph[Microsoft Graph]
    subgraph azure[Azure landing zone: one regional deployment]
      web[React TypeScript PWA\nAzure Static Web Apps or App Service static hosting]
      api[ASP.NET Core modular monolith API\nAzure App Service, 2+ instances]
      worker[Background worker\nAzure Functions or worker in App Service]
      sql[(Azure SQL Database\ntransactional system of record)]
      queue[Azure Service Bus Standard\nnotifications and scheduled work]
      obs[Application Insights / Azure Monitor\ncentral Log Analytics]
      config[Key Vault + App Configuration\nsecrets and feature flags]
    end
    users --> web
    web -->|OIDC token, HTTPS API| api
    api <--> sql
    api -->|transactional outbox publisher| queue
    queue --> worker
    worker <--> sql
    api <-->|OIDC| entra
    worker <-->|app identity / OIDC| entra
    worker --> hr
    worker --> exchangeGraph
    api --> obs
    worker --> obs
    api --> config
    worker --> config
```

The PWA is a presentation client; authorization is enforced by the API. The API and worker share a codebase/domain modules and the SQL transaction boundary. HR calls and notifications do not run inside booking transactions. The hosting alternatives shown are not selected; validate the worker runtime, database tier, region, networking, and monitoring against the corporate landing zone.

## Booking Components

```mermaid
flowchart LR
    client[Booking UI]
    endpoint[Booking API endpoint]
    auth[Identity and site-scope authorization]
    policy[Eligibility and site policy evaluator]
    availability[Availability query]
    booking[Booking command handler]
    tx[(SQL transaction)]
    unique[Unique active allocation index]
    quota[Weekly quota guard]
    outbox[Transactional outbox]
    worker[Notification / scheduled worker]
    client --> endpoint --> auth
    endpoint --> availability
    endpoint --> booking --> policy
    booking --> tx
    tx --> unique
    tx --> quota
    tx --> outbox
    outbox --> worker
```

Availability is advisory; the booking transaction is authoritative. The filtered unique active-allocation index prevents two active allocations for the same tenant, slot, local date, and period. Site policy and quota are rechecked at commit.

## Domain Model and Bounded Contexts

Deploy as a modular monolith with explicit module boundaries, no direct cross-module table writes, and synchronous booking decisions in one SQL transaction.

| Context | Main entities and responsibility |
|---|---|
| Identity & Access | `Tenant`, `Employee`, `RoleAssignment`, `SiteScope`; Entra subject is the login key and HR identifier is mapped separately. |
| Site & Inventory | `Site` (IANA time-zone ID and business calendar), `Slot` (attributes, assigned owner, active state), `SlotRelease` (owner, local date/period, state). |
| Booking | `Booking` (requester, selected plate reference, state, idempotency key), `BookingAllocation` (slot/date/period uniqueness), `BookingPolicy` (window, quota, priority rules), `CheckIn`. |
| Security Access | `AuthorisedPlateExport` (site, local date, version, minimum plate set), `AccessEvent` (future adapter-normalised arrival/denial event). Do not store raw LPR imagery. |
| Notifications | `OutboxMessage`, `NotificationAttempt`, delivery status and retry metadata; notification failure does not roll back a booking. |
| Reporting & Audit | `AuditRecord` for administrative/security-sensitive actions; aggregate occupancy without unnecessary person-level history. |

Every business row is tenant-scoped; site-specific authorization also requires site scope. Protect plate values separately from routine booking/reporting fields; never emit them in logs. Store site service dates as local dates with an IANA zone and event timestamps in UTC. Entity-level detail and isolation controls remain subject to review in 3.1.
