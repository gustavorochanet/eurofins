# 3.4 Architecture Decision Records

**Status:** Proposed decisions for assessment; not yet approved.  
**Related:** [Assumptions and Open Questions](01-assumptions-open-questions.md).

## ADR-001: Modular Monolith for MVP

**Context:** Four months, one small cross-functional team, moderate initial scale, and atomic booking decisions.  
**Options:** Microservices; serverless function-per-capability; modular monolith.  
**Decision:** Modular monolith with explicit domain modules and shared relational transaction boundary. Deploy one API and a background worker; extract only for demonstrated scale, ownership, or availability need.  
**Consequences:** Faster coherent delivery and straightforward consistency; module boundaries, migrations, and shared deployment failure domain need governance.

## ADR-002: Azure App Service and Azure SQL

**Context:** Azure is standard; the team knows .NET and SQL Server; target run cost is under EUR 3,000/month.  
**Options:** AKS; Azure Container Apps; App Service; Azure SQL versus Cosmos DB.  
**Decision:** Containerised ASP.NET Core on Azure App Service with at least two production instances, Azure SQL Database with a supported general-purpose configuration, Bicep, and zone redundancy where supported and affordable.  
**Consequences:** Low operations burden and familiar relational constraints; Azure SQL/App Service are managed dependencies. Validate exact tier, region, network, licensing, price, and zone support before commitment.

## ADR-003: PWA Instead of Native Mobile Apps

**Context:** Mobile skills are limited and web/phone access is needed in four months.  
**Options:** Native iOS/Android; cross-platform native; responsive web/PWA.  
**Decision:** React/TypeScript responsive PWA with installability and browser push where supported; email fallback; no native app in MVP.  
**Consequences:** One product surface and lower maintenance; push/install behavior varies. Booking and check-in must work in the browser without installation.

## ADR-004: SQL-Authoritative Booking and Asynchronous Side Effects

**Context:** No double booking is mandatory under bursty concurrent load.  
**Options:** Distributed lock/cache; serialized booking service; relational constraints and transactions.  
**Decision:** SQL transaction, filtered unique active-allocation index, idempotency key, and transactional outbox. Use Service Bus Standard for retriable post-commit work; queue delivery never controls booking correctness.  
**Consequences:** Strong invariant with a simple operational model; contention and quota semantics require integration/load tests.

## ADR-005: Vendor-Neutral Edge Boundary; Defer Runtime

**Context:** At least two LPR vendors are future scope and barriers must work during cloud loss.  
**Options:** Direct vendor calls in domain; vendor-specific modules; canonical access-event contract and local adapter.  
**Decision:** Define canonical `AuthorisationSnapshot`, `ArrivalEvent`, and `AccessDecision` contracts; defer edge runtime until vendor, offline policy, hardware, and support are selected. Before barrier pilot, use encrypted local storage, signed/expiring snapshots, local decision capability, event replay, and operator override.  
**Consequences:** Vendor detail stays outside booking logic; offline revocation, hardware, clock integrity, and physical safety need site-specific design and test.

## ADR-006: Tenant-Aware Data Without Premature SaaS Productisation

**Context:** External SaaS is possible later; MVP serves one employer.  
**Options:** No tenant boundary; tenant-aware shared schema; full database-per-tenant SaaS now.  
**Decision:** Include `TenantId` and tenant-scoped authorization/query patterns from the first migration; seed one tenant. Defer onboarding, billing, self-service, and database-per-tenant operations.  
**Consequences:** Lowers migration risk but makes tenant filtering security-critical. Enforce at API/repository boundaries and test cross-tenant isolation; reassess before external customers.
