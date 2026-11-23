# 3.8 Developer and AI Guidance

**Status:** Proposed guidance for a future ParkShare repository; no application repository exists yet.  
**Related:** [Assumptions and Open Questions](01-assumptions-open-questions.md), question 9.

## Documentation and Enforcement

Maintain an architecture overview/diagrams, module ownership, ADRs, API/error conventions, data classification, runbooks, glossary, local setup, testing, and release guides. Put concise global rules in `.github/copilot-instructions.md`, with path-specific `.instructions.md` files for API/domain, data access, frontend, and infrastructure. Keep a one-command local start using synthetic data.

Update guidance in the same PR as a changed contract or architectural boundary. Give sections owners and review dates; review quarterly. CI checks formatting, tests, OpenAPI diffs, dependency boundaries, IaC, secret/dependency/container scans, and SBOM. PR template records affected modules, data classification, tests, telemetry, migrations, and documentation. Human reviewers remain accountable for AI-assisted changes.

## Sample Repository Guidance

```markdown
# ParkShare Engineering Guide

## System Shape
- ParkShare is a modular monolith: Identity, Inventory, Booking,
  SecurityAccess, Notifications, Reporting, and Audit.
- Keep domain rules in their owning module. Do not write another module's
  tables directly; use its application interface or a documented event.
- Booking correctness is owned by Azure SQL transactions and the unique
  active-allocation index. Search results are advisory, never a reservation.
- The MVP has no barrier/LPR integration. Do not add vendor SDKs or infer
  presence from a plate list. Check-in is the proposed pilot signal.

## Security and Privacy
- Treat employee IDs, licence plates, bookings, and check-ins as personal data.
- Enforce TenantId, role, site scope, and resource ownership in the API.
  A hidden UI control is not authorization.
- Never log licence plates, tokens, secrets, or raw HR payloads. Use opaque
  correlation IDs. Use managed identity; never hardcode credentials.
- Use parameterized SQL and validate external input on the server. Do not
  weaken authentication, authorization, TLS, CSRF, or audit controls.
- Do not change retention periods without DPO/works-council approval.

## API and Data
- Follow versioned REST/OpenAPI conventions in the architecture guide.
- Return stable problem-details errors; use 409 for allocation conflicts.
- Booking commands require an idempotency key and must be transactional.
  External calls/notifications must not run inside the booking transaction.
- Persist UTC instants and site-local service dates with an IANA time-zone ID.
- Schema changes require reviewed migrations and SQL integration tests.

## Quality Gates
- Add behavior-focused unit and SQL integration tests for changed rules.
- Booking changes require a race test proving one active allocation wins.
- Run formatting, tests, analyzers, dependency/security scans, and OpenAPI
  checks. Do not disable or weaken a gate to pass.
- Add structured telemetry without personal data; update dashboards/runbooks
  when operational behavior changes.
- Prefer approved libraries. Explain and review any new dependency.
- Do not invent business policy; raise unresolved rules and mark assumptions.

## AI Assistant Boundaries
- Do not commit, deploy, change cloud resources/permissions, or create tickets
  unless explicitly requested and reviewed by a human.
- Do not claim tests, security reviews, or privacy approvals passed unless run
  and evidenced. Summarize changed files, tests, and remaining risks.
```

## AI Security, IP, and Privacy

Use only the enterprise-approved Copilot tenant/settings. Never paste real plates, HR records, secrets, credentials, or incident data into prompts. Review licensing/IP of generated dependencies and code under corporate policy. Require human review for authorization, SQL migrations, cryptography, retention, infrastructure, and generated tests. AI output is not evidence of security or compliance. Track AI-originated defects through the normal quality process.
