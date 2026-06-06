# Feature Specification: Platform-Wide Reporting and Support Access

**Feature Branch**: `013-platform-support-access`  
**Created**: 2026-06-06  
**Status**: Draft  
**Input**: User description: "Run speckit-specify for backend roadmap item 8: Platform-Wide Reporting and Support Access. Define the next SchoolMaster backend implementation slice after completed report lifecycle expansion. The feature must specify platform administrator and support-user visibility across schools for operational oversight without creating an implicit bypass of school-scoped student, teacher, guardian, administration, or report permissions.

Specify allowed platform-wide views, support access workflows, cross-school reporting summaries, school lookup or drill-down boundaries, authorization exceptions, role and permission requirements, audit obligations, redaction and data minimization rules, school opt-in or approval requirements if needed, denied-access behavior, conflict rules, response semantics, and tenant-safe error handling.

Preserve contract-first API-only /api/v1 behavior. Require OpenAPI expansion before backend implementation exposes any platform-wide reporting or support access behavior. Keep School as the v1 tenant root and school_id as the concrete tenant column for school-owned records. Preserve explicit platform versus school authorization separation: platform administrators and support users may only access school-scoped data through specifically documented and tested platform support/reporting permissions.

Define audit events for allowed access, denied access, cross-school lookup, report summary access, drill-down access, support escalation, and any school opt-in or approval decisions. Audit entries must include actor, action, outcome, target school when applicable, correlation ID, tenant-safe reason code, and minimized metadata only. Do not store credentials, bearer tokens, private file paths, raw report outputs, private content, full student/guardian records, or unauthorized cross-tenant details in audit metadata.

Define redaction and minimization rules for platform-visible data. Platform-wide views should expose only the minimum data needed for operational oversight, such as school identity/status, aggregate counts, report health/status summaries, lifecycle state summaries, and support diagnostics. Any access to student, guardian, teacher, report-run, custom-report, or school-admin detail must be explicitly allowed by the specification and contract, with separate permission checks and audit coverage.

Clarify whether support access is read-only, whether support users can drill into a single school, whether school opt-in or approval is required before support access, whether support access can include generated report downloads, and whether emergency access is allowed. If these choices are not already documented, mark them as clarification questions instead of inventing behavior.

Do not include frontend implementation, advanced assessment/content types, billing, payroll, accounting, messaging, notifications, live classroom, video conferencing, unrestricted impersonation, permanent purge, legal hold, anonymization workflows, unrestricted raw database search, write actions against school-owned operational records, or undocumented APIs in this slice."

## Clarifications

### Session 2026-06-06

- Q: Should v1 platform support access allow support users to perform write actions such as account reactivation, report retry/cancel, or school status changes? -> A: Read-only support access only; no support writes in this slice.
- Q: What approval model should v1 support drill-down require before school-owned diagnostics are returned? -> A: Both target-school opt-in and internal platform approval.
- Q: Should v1 support access include generated report downloads or emergency access? -> A: Exclude generated report downloads and emergency access from v1.
- Q: What aggregation threshold should platform-visible summary metrics use to prevent small-count identification? -> A: Suppress counts below 5.
- Q: How long should approved support drill-down access remain valid before expiration? -> A: Expire after 24 hours.
- Q: Which school actors may approve or revoke target-school support opt-in? -> A: Only school administrators with explicit support opt-in permission.
- Q: How long should target-school support opt-in remain valid? -> A: Target-school opt-in expires after 24 hours.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Review Platform Operational Health (Priority: P1)

A platform administrator reviews school-level operational health across the platform using minimized cross-school summaries, without receiving school-scoped student, guardian, teacher, administration, or report detail records by default.

**Why this priority**: Platform operators need visibility into tenant status, report health, and support risk signals, but existing rules prevent platform roles from silently bypassing school-scoped authorization.

**Independent Test**: Authenticate as a platform administrator with platform-wide oversight permission, list schools with operational summary fields, and verify the response includes only approved minimized fields, excludes detail records, records an audit event, and denies actors without the explicit platform permission.

**Acceptance Scenarios**:

1. **Given** a platform administrator has platform-wide operational oversight permission, **When** they list platform school summaries, **Then** the response includes only school identity, school status, aggregate counts, report health summaries, lifecycle state summaries, and support diagnostics approved by the OpenAPI contract.
2. **Given** a platform administrator requests the cross-school reporting overview, **When** the request includes supported filters and sorting, **Then** the system returns aggregate report lifecycle and output health summaries without raw report outputs, private file paths, or individual student/guardian/teacher detail.
3. **Given** a platform actor lacks the required platform-wide oversight permission, **When** they request platform school summaries or reporting overview, **Then** access is denied with the documented response envelope and no cross-school data is disclosed.
4. **Given** one or more schools are inactive, suspended, or otherwise restricted, **When** a platform overview is requested, **Then** the response represents the approved status and operational summary without exposing school-owned operational records beyond the allowed minimized fields.

---

### User Story 2 - Perform Authorized Support Drill-Down (Priority: P2)

A platform support user investigates a single school through an explicitly authorized read-only support access path that is narrower than school administrator access and fully audited.

**Why this priority**: Support staff need enough context to diagnose operational issues, but support access must be a documented exception instead of an implicit cross-tenant bypass.

**Independent Test**: Authenticate as a support user with support drill-down permission, target-school opt-in approved by a same-school administrator with explicit support opt-in permission, and internal platform approval, request a permitted school support view, verify only approved redacted diagnostics are returned, verify target school and reason metadata are audited, and verify unrelated school-owned records remain inaccessible.

**Acceptance Scenarios**:

1. **Given** a support user has support drill-down permission, target-school opt-in approved by a same-school administrator with explicit support opt-in permission, internal platform approval, and submits the required target-school and reason metadata, **When** they request a school support view, **Then** the system returns only approved redacted school diagnostics and writes an audit event with actor, action, outcome, target school, correlation ID, and tenant-safe reason code.
2. **Given** a support user attempts to access student, guardian, teacher, report-run, custom-report, or school-admin detail not explicitly allowed by the contract, **When** the request is processed, **Then** access is denied without disclosing whether the hidden record exists.
3. **Given** a support access request lacks required target-school, target-school opt-in, internal platform approval, purpose, or reason metadata, **When** the request is submitted, **Then** validation rejects the request before school-owned data is accessed.
4. **Given** support access has been granted for one school, **When** the same actor attempts to reuse that context for another school, **Then** the system requires a separate authorization decision and audit trail.
5. **Given** an approved support drill-down decision or target-school opt-in is older than 24 hours, revoked, stale, or mismatched to the requested school, **When** the support user requests school diagnostics, **Then** access is denied before school-owned data is returned and the outcome is audited.

---

### User Story 3 - Govern Support Exceptions and Audit Review (Priority: P3)

Authorized compliance or platform administrators review support-access decisions and audit events to confirm that cross-school visibility remains justified, minimized, and traceable.

**Why this priority**: Cross-tenant visibility introduces privacy and security risk; the platform needs verifiable auditability before expanding support access beyond aggregate summaries.

**Independent Test**: Create allowed and denied platform/support access attempts, then retrieve audit review summaries as an authorized platform compliance actor and verify minimized metadata, target-school attribution, denied-access reason codes, and redaction rules.

**Acceptance Scenarios**:

1. **Given** platform overview, support drill-down, denied access, and validation rejection events exist, **When** an authorized audit reviewer lists support access audit summaries, **Then** the response includes actor, action, outcome, target school when applicable, correlation ID, tenant-safe reason code, and minimized metadata only.
2. **Given** audit metadata references protected student, guardian, teacher, report, file, credential, token, or unauthorized cross-tenant information, **When** the event is stored or returned, **Then** the system redacts or rejects that metadata according to the documented contract.
3. **Given** a support access decision depends on target-school opt-in or internal platform approval, **When** the decision is made, denied, expired, or revoked, **Then** the decision and its outcome are auditable with minimized decision metadata.
4. **Given** an actor without audit review permission requests support audit summaries, **When** the request is processed, **Then** access is denied without disclosing sensitive support activity details.

### Edge Cases

- What happens when a platform user has platform administration permissions but not the new platform-wide reporting or support access permissions?
- How does the system prevent platform summaries from exposing small-count or uniquely identifying school-owned student, guardian, teacher, report, or administration data when counts are below the suppression threshold?
- What happens when school status changes while a support user is performing an approved drill-down workflow?
- How are stale, expired, revoked, or mismatched support access approvals and target-school opt-ins handled before school-owned data is accessed, including decisions older than 24 hours?
- How does the system reject target-school opt-in approval or revocation by a school administrator who lacks explicit support opt-in permission?
- What happens when a support actor requests generated report downloads, emergency access, private file metadata, raw report output, or unrestricted record search?
- How does the system audit denied cross-tenant attempts without storing unauthorized target details?
- How does the system handle concurrent support approval, revocation, and access attempts for the same target school?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Adds platform-level reporting/support authorization behavior, request validation for platform overview and support drill-down workflows, redaction/minimization services, audit recording, response resources, and protected `/api/v1` routes after contract approval.
- **Frontend repository impact**: No frontend implementation in this slice; any future frontend surfaces must consume only approved platform/support contract behavior.
- **Specification or contract repository impact**: Requires this feature specification, feature-level contract definitions, and promotion into `api/openapi.yaml` before backend routes expose platform-wide reporting or support access.
- **Delivery ownership and sequencing**: `schoolmaster-specs` leads specification and OpenAPI work, `schoolmaster-backend` implements only approved contract behavior afterward, and frontend consumption remains a later separate effort.

### API Contract Impact

- **OpenAPI update required**: Yes. Define platform-wide school summary, cross-school reporting overview, support drill-down, support access decision, and support audit summary operations before backend implementation.
- **Versioned endpoints affected**: New `/api/v1/platform/...` or equivalent platform-scoped endpoints for platform school summaries, reporting overview, support access, and support audit review.
- **JSON response impact**: Adds minimized platform/support response envelopes, redacted diagnostic payloads, tenant-safe validation errors, authorization denials, and conflict responses for stale or revoked support access decisions.
- **Authentication/authorization impact**: Adds explicit platform-wide reporting, support drill-down, support decision, support audit review, and same-school support opt-in permissions; platform administrator or school administrator status alone is not sufficient unless paired with documented permission grants.
- **Compatibility impact**: Additive contract change. No existing school-scoped endpoint may change behavior or become accessible to platform/support actors without explicit operation-level contract updates.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-owned records remain scoped by `school_id`; platform views may aggregate or summarize across schools only through documented platform operations.
- **Cross-tenant or platform access impact**: This feature defines the only allowed platform/support exceptions for cross-school operational oversight and support diagnostics in this slice.
- **Soft delete impact**: Platform summaries may include aggregate lifecycle counts where approved, but soft-deleted school-owned record detail remains hidden unless a contract explicitly permits a redacted summary.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST define platform-wide reporting and support access as explicit platform-scoped permissions, separate from school-scoped roles and permissions.
- **FR-002**: System MUST reject platform or support access to school-owned records unless the request uses a documented platform/support operation with the required permission.
- **FR-003**: System MUST provide a platform school summary view that exposes only approved minimized fields for operational oversight.
- **FR-004**: System MUST provide a cross-school reporting overview that exposes only approved aggregate report health, lifecycle, retention, output availability, and failure-summary metrics.
- **FR-005**: System MUST prevent platform overview responses from including raw report outputs, private file paths, credentials, bearer tokens, student detail, guardian detail, teacher detail, school-admin detail, private content, or unauthorized cross-tenant details.
- **FR-006**: System MUST validate platform overview filters, sorting, pagination, and query inputs before accessing platform or school-owned summary data.
- **FR-007**: System MUST define support drill-down access as a target-school-specific workflow with required actor permission, target school, reason code, and audit correlation metadata.
- **FR-008**: System MUST limit support drill-down responses to redacted diagnostics and explicitly contracted detail fields only.
- **FR-009**: System MUST deny support attempts to access student, guardian, teacher, report-run, custom-report, school-admin, or file detail unless that exact detail view is documented in the approved OpenAPI contract.
- **FR-010**: System MUST not allow unrestricted impersonation of school users as part of platform-wide reporting or support access.
- **FR-011**: System MUST not expose platform or support operations that write to school-owned operational records in this slice, including account reactivation, report retry/cancel, school status changes, or other operational mutations.
- **FR-012**: System MUST audit allowed access, denied access, validation rejection, cross-school lookup, platform reporting summary access, support drill-down access, support escalation, support approval, support revocation, and support expiration events.
- **FR-013**: Audit entries MUST include actor, action, outcome, target school when applicable, correlation ID, tenant-safe reason code, and minimized metadata only.
- **FR-014**: Audit entries MUST NOT store credentials, bearer tokens, private file paths, raw report outputs, private content, full student/guardian records, or unauthorized cross-tenant details.
- **FR-015**: System MUST return tenant-safe not-found, authorization, validation, and conflict responses that do not disclose unauthorized school-owned record existence.
- **FR-016**: System MUST deny support drill-down when target-school opt-in or internal platform approval is stale, expired, revoked, mismatched, older than 24 hours, or concurrently changed before access is completed.
- **FR-017**: System MUST define OpenAPI operation IDs, request schemas, response schemas, response envelopes, pagination, filtering, sorting, authorization errors, validation errors, and conflict responses before backend implementation begins.
- **FR-018**: System MUST preserve `School` as the v1 tenant root and `school_id` as the concrete tenant column for school-owned records.
- **FR-019**: System MUST preserve platform versus school authorization separation; platform roles may grant only documented platform capabilities and school roles remain effective only within the resolved school tenant.
- **FR-020**: System MUST keep existing school-scoped student, teacher, guardian, administration, and reporting endpoint behavior unchanged unless a separate approved contract explicitly changes it.
- **FR-021**: System MUST define support drill-down as a read-only, target-school-bound diagnostic workflow with a maximum 24-hour authorization window and no reusable authorization context across schools.
- **FR-022**: System MUST suppress platform-visible summary counts below 5 where summary values could identify protected school-owned student, guardian, teacher, report, or administration data.
- **FR-023**: System MUST reject support access requests that attempt write-oriented intents or mutation endpoints through validation, authorization, or routing before any school-owned operational record is changed.
- **FR-024**: Support drill-down MUST evaluate both target-school opt-in and internal platform approval at request time before school-owned diagnostics are returned, and both gates MUST be no older than 24 hours.
- **FR-025**: Generated report downloads and emergency access MUST remain excluded from v1 platform support access; support users MUST NOT download generated report files, receive raw report outputs, or bypass normal approval gates through emergency access in this slice.
- **FR-026**: Target-school support opt-in approval and revocation MUST require a same-school school administrator with explicit support opt-in permission; school administrator status alone MUST NOT be sufficient.

### Key Entities *(include if feature involves data)*

- **Platform Operational Summary**: Minimized cross-school view containing school identity/status, aggregate counts, reporting health, lifecycle state summaries, and support diagnostics approved for platform oversight.
- **Platform Reporting Overview**: Aggregate cross-school reporting health view covering report lifecycle status, failures, retention/output availability, and operational indicators without raw outputs or school-owned detail records.
- **Support Access Decision**: A target-school-specific authorization decision for support drill-down, including actor, target school, reason code, target-school opt-in state, internal platform approval state, lifecycle state, correlation ID, 24-hour expiration, and revocation outcome.
- **Target-School Support Opt-In**: School-scoped approval record created or revoked only by a same-school administrator with explicit support opt-in permission for a documented support purpose; expires after 24 hours.
- **Internal Platform Approval**: Platform-scoped approval record created or revoked by an authorized platform actor for a specific support access decision, target school, reason code, and support purpose; expires after 24 hours and must be evaluated with the target-school opt-in before diagnostics are returned.
- **Support Diagnostic View**: Redacted support payload for one target school, limited to approved diagnostics and explicitly contracted detail fields.
- **Platform Support Audit Event**: Minimized audit record for allowed, denied, validation, overview, drill-down, approval, revocation, expiration, and conflict events.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of platform-wide reporting and support access operations require explicit platform-scoped permission checks before returning platform or school-owned summary data.
- **SC-002**: 100% of platform overview responses exclude raw report outputs, private file paths, credentials, bearer tokens, full student/guardian records, private content, and unauthorized cross-tenant details.
- **SC-003**: Authorized platform administrators can retrieve approved school operational summaries and cross-school reporting overview data using documented filters and pagination without using any school-scoped endpoint.
- **SC-004**: Authorized support users can retrieve only approved redacted diagnostics for a target school, and denied or mismatched attempts disclose no protected record existence.
- **SC-005**: 100% of allowed, denied, validation, support decision, and conflict outcomes generate tenant-safe audit records with actor, action, outcome, target school when applicable, correlation ID, and reason code.
- **SC-006**: Existing school-scoped student, teacher, guardian, administration, and reporting endpoints remain inaccessible to platform/support actors unless an operation-specific contract grants access.
- **SC-007**: Contract validation passes for all new platform/support OpenAPI operations before backend implementation exposes new routes.
- **SC-008**: Support access validation rejects 100% of generated report download, raw report output, private file metadata, emergency access, and unrestricted record search attempts in this slice.
- **SC-009**: Platform summary responses suppress 100% of protected aggregate counts below 5 and avoid replacing suppressed values with uniquely identifying detail.
- **SC-010**: Support drill-down attempts using platform approvals or target-school opt-ins older than 24 hours, revoked approvals, stale approvals, or mismatched target-school approvals are denied before school-owned diagnostics are returned.
- **SC-011**: 100% of target-school support opt-in approval and revocation attempts by school administrators without explicit support opt-in permission are denied and audited without granting support diagnostics.

## Assumptions

- Feature `012-report-lifecycle-expansion` has been implemented and report lifecycle state, output availability, catalog, and custom definition behavior are available as the baseline for platform reporting summaries.
- Platform-wide operational oversight is a backend API contract feature first; frontend implementation remains out of scope for this slice.
- Existing authentication, role, permission, tenant context, response envelope, pagination, filtering, sorting, and audit foundations will be reused and extended by contract.
- Platform administrators and support users are platform-scoped actors, not school-scoped actors, unless a separate authenticated school context is explicitly established by another approved workflow.
- Platform support access is narrower than school administrator access and must never become unrestricted record search or unrestricted impersonation.
- School-owned records continue to use `school_id`; platform summaries may reference target schools but must not remove school tenant boundaries from underlying records.
