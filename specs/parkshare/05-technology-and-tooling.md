# 3.5 Technology Stack, Frameworks, and Tooling

**Status:** Proposed stack; validate versions, licences, and corporate approvals before implementation.  
**Related:** [Assumptions and Open Questions](01-assumptions-open-questions.md), questions 5 and 9.

## Application Stack

| Layer | Proposal | Rationale |
|---|---|---|
| API | .NET 10 LTS, ASP.NET Core REST controllers, versioned OpenAPI | Matches in-house skills; stable support horizon and clear HTTP contracts. Pin SDK/runtime and follow Microsoft's LTS policy. |
| Domain / validation | C# domain types; FluentValidation or approved equivalent; explicit mapping | Test rules in domain; avoid implicit mapping in sensitive booking commands. |
| Data | Azure SQL Database, EF Core 10, reviewed migrations and targeted indexes | Relational constraints and SQL Server expertise; inspect generated SQL on search/booking paths. |
| Background work | Transactional outbox, Azure Service Bus Standard, .NET worker/Azure Functions | Retry notifications and scheduled work outside booking commit. Runtime choice is open. |
| Resilience | `Microsoft.Extensions.Http.Resilience` for HR/Graph calls | Bounded timeouts, retries, and circuit breakers; do not blindly retry non-idempotent writes. |
| Identity | Entra OIDC; Microsoft identity platform/MSAL as needed; managed identity service-to-service | Corporate SSO and fewer stored credentials; exact library depends on hosting model. |
| Web | React, TypeScript strict mode, Vite, approved component library such as Fluent UI | Fits stated skills; confirm approved versions and accessibility requirements. |
| Frontend state | Small server-state query/cache library only if needed; URL for search filters; local form state | Avoid duplicated state and client-side booking authority. |
| Mobile | Responsive PWA/service worker; push where supported; online-only booking/check-in | Avoid native delivery. Do not cache plate or booking details offline. |
| Observability | OpenTelemetry-compatible traces/metrics/logs to Application Insights/Azure Monitor | Correlate booking, database, outbox, and worker without PII. |
| Edge (future) | .NET worker/container on approved site hardware; encrypted local cache/vendor adapter | Reuse team skills; choose hardware after vendor discovery. |

## Engineering Toolchain

| Concern | Proposal |
|---|---|
| Source / branching | Azure Repos Git; trunk-based short-lived branches, PR to main, required reviewers and linked work item. |
| CI/CD | Azure Pipelines YAML; build, tests, scans, signed artifact, approval-gated dev/test/stage/prod promotion; workload federation/managed identity where supported. |
| IaC / environments | Bicep modules; isolated dev/test/stage/prod identities/resources; never copy production personal data to lower environments. |
| Containerisation | OCI API/worker images in ACR; pin and scan base images; local Docker Compose for API and SQL dependencies. |
| Tests | xUnit unit; SQL-compatible integration; OpenAPI contract; Playwright E2E; k6 peak/contention tests; synthetic plate fixtures. |
| Quality | .NET analyzers, `dotnet format`, nullable enabled, ESLint, TypeScript strict, architecture/module-boundary tests. |
| Security / supply chain | Approved SAST/SCA, NuGet/npm audit, secret/container scanning, SBOM per release, vulnerability triage SLA. |
| API docs | Generated OpenAPI, reviewed breaking-change diff, authenticated non-production Swagger UI only. |
| Observability | Azure Monitor dashboards/alerts, restricted central workspace, synthetic checks; control ingestion/retention. |
| Feature flags | Azure App Configuration or approved equivalent; owner, expiry and rollback instructions; never use flags as authorization. |

Choose technology by in-house skills and corporate standards first, then security/support maturity, total cost, operational burden, licence, long-term maintenance, and portability. Avoid AKS, microservices, separate cache/search cluster, native apps, and unapproved dependencies in MVP without a demonstrated need. Accept Azure managed-service dependencies under the corporate Azure constraint; isolate email, queue, and future vendor integrations behind interfaces where switching is plausible.

Maintain an approved technology list/tech radar with owner/review date. Pin dependencies; review monthly and expedite critical security updates. Upgrade .NET and Node on supported LTS cycles; review frontend packages quarterly or sooner for security. Generate an SBOM per release. Require owner, maintenance/support evidence, licence review, vulnerability scan, and business reason for each new dependency; track abandoned or critical-vulnerable packages to remediation.
