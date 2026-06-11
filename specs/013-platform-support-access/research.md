# Research: Platform-Wide Reporting and Support Access

## Decision: Expand OpenAPI before backend exposure

**Decision**: All platform-wide reporting, school summary, support drill-down, support access decision, audit summary, approval, revocation, expiration, denial, conflict, and redaction behavior must be added to `api/openapi.yaml` and any affected platform contract before backend routes expose the behavior.

**Rationale**: The constitution requires API-first contract governance. Platform/support access creates intentional cross-tenant visibility exceptions, so undocumented backend behavior would be a security and privacy risk.

**Alternatives considered**:

- Backend-local routes first: rejected because it would create undocumented public API behavior.
- Feature-local contract notes only: rejected because backend and frontend consume the aggregate OpenAPI contract.

## Decision: Platform-wide views expose minimized summaries only

**Decision**: Platform-wide operational views expose approved minimized school identity/status, aggregate counts, report health, lifecycle state summaries, and support diagnostics. They do not expose school-owned detail records by default.

**Rationale**: The roadmap asks for platform administrator or support-user visibility across schools for operational oversight, while existing rules intentionally prevent implicit school-scoped permission bypass.

**Alternatives considered**:

- Reuse existing school-scoped endpoints for platform actors: rejected because it would bypass tenant authorization semantics.
- Expose full detail records with redaction: rejected because the default product need is operational oversight, not unrestricted investigation.
- Delay platform summaries until frontend planning: rejected because backend contract behavior must be established first.

## Decision: Suppress protected aggregate counts below 5

**Decision**: Protected platform-visible aggregate counts below 5 are suppressed, and responses must not replace suppressed values with uniquely identifying detail.

**Rationale**: Small-count summaries can identify students, guardians, teachers, report activity, or administrative records in small schools or narrow filters. A concrete threshold gives contract and test coverage an objective privacy rule.

**Alternatives considered**:

- Suppress below 3: rejected because it leaves higher re-identification risk in small cohorts.
- Suppress below 10: rejected for v1 because it may hide too much operational signal across small schools.
- Use only status bands: rejected because platform operators need some aggregate operational counts.

## Decision: Support drill-down is read-only

**Decision**: V1 support drill-down may return only approved redacted diagnostics and must not perform account reactivation, report retry/cancel, school status changes, or any other write action against school-owned operational records.

**Rationale**: The roadmap item is about visibility for operational oversight. Write authority would create separate lifecycle, authorization, rollback, and audit requirements that belong in a later explicit specification.

**Alternatives considered**:

- Allow limited account recovery writes: rejected because account lifecycle is a separate security-sensitive workflow.
- Allow limited report lifecycle writes: rejected because report lifecycle actions are school-scoped and already governed by school permissions.
- Allow limited school status writes: rejected because school lifecycle changes are platform administration behavior, not support drill-down.

## Decision: Support drill-down requires target-school opt-in and internal platform approval

**Decision**: Support drill-down requires both a school-scoped target-school opt-in and an internal platform approval before school-owned diagnostics are returned. Target-school opt-in is represented by dedicated school-scoped support opt-in operations, not by implicit school-admin access or platform-only approval. Target-school opt-ins expire after 24 hours so they cannot outlive the support access window.

**Rationale**: Cross-school support diagnostics are more sensitive than aggregate platform summaries. Requiring both gates reduces privacy risk and makes access decisions auditable.

**Alternatives considered**:

- Permission and reason code only: rejected because it lacks explicit approval for a high-risk cross-tenant access path.
- Internal platform approval only: rejected because it does not reflect school participation or visibility.
- Target-school opt-in only: rejected because it does not provide internal operational oversight of support access.
- Platform-only opt-in modeling: rejected because it would not capture a school-side approval gate.
- Manual revocation only: rejected because stale school-side opt-ins could be reused unexpectedly.

## Decision: Support access decisions expire after 24 hours

**Decision**: Approved support drill-down decisions and target-school opt-ins expire after 24 hours. Attempts using approvals or opt-ins older than 24 hours, stale approvals, revoked approvals, or mismatched target-school approvals are denied before diagnostics are returned.

**Rationale**: A 24-hour window is short enough to avoid lingering cross-school access while supporting investigations that may span a working day.

**Alternatives considered**:

- Four-hour expiration: rejected because common support investigations may require handoff or follow-up within the same day.
- Seventy-two-hour expiration: rejected because it leaves cross-school access open longer than needed for v1.
- Manually selected expiration up to seven days: rejected because v1 should keep approval semantics simple and predictable.

## Decision: Exclude generated report downloads and emergency access

**Decision**: V1 support access excludes generated report downloads, raw report outputs, private file metadata, and emergency access. Support users cannot bypass normal approval gates through emergency access in this slice.

**Rationale**: Report outputs and emergency access are high-risk access paths that require separate approval, retention, audit, and redaction rules. This slice can satisfy operational oversight with redacted diagnostics.

**Alternatives considered**:

- Allow generated report downloads with approval: rejected because report output contents can contain sensitive school data and are outside support diagnostics.
- Allow emergency access with approval: rejected because emergency workflows need independent governance and break-glass controls.
- Allow both under strict audit: rejected because it materially expands scope beyond the roadmap item's first platform/support access slice.

## Decision: Platform support audit metadata is minimized and tenant-safe

**Decision**: Platform support audit events include actor, action, outcome, target school where applicable, correlation ID, tenant-safe reason code, timestamp, and minimized metadata. Audit events must not store credentials, bearer tokens, private file paths, raw report outputs, private content, full student/guardian records, or unauthorized cross-tenant details.

**Rationale**: Cross-tenant visibility needs traceability, but audit trails must not become a secondary leak of protected data.

**Alternatives considered**:

- Store full request and response payloads: rejected because diagnostics and denials may contain protected school-owned data.
- Store only actor/action/outcome: rejected because support investigation and compliance review need target-school, reason, and correlation context.

## Decision: Backend implementation structure

**Decision**: Implement through Laravel controllers, Form Requests, Policies, API Resources, Services, DTOs, and repositories/query objects only for complex cross-school aggregation, small-count suppression, support access decision lookup, and audit-safe reads.

**Rationale**: This follows the constitution and prior backend slices while keeping controllers thin, authorization explicit, and cross-tenant data access reviewable.

**Alternatives considered**:

- Controller-level support logic: rejected by constitution and because cross-tenant authorization needs dedicated policy/service boundaries.
- Repository abstraction for every model: rejected because repositories are reserved for genuinely complex data access patterns.
