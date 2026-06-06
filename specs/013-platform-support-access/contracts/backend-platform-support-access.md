# Contract Boundary: Platform-Wide Reporting and Support Access

## Purpose

This feature adds a public API surface for platform-wide operational summaries, cross-school reporting health summaries, read-only support drill-down, support access decisions, and support access audit review. Backend implementation must wait until OpenAPI documents exact operations, parameters, request schemas, response schemas, errors, tenant behavior, authorization behavior, approval states, redaction rules, conflict semantics, audit expectations, and operation IDs.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend roadmap: `docs/backend-feature-roadmap.md`
- Architecture guidance: `docs/architecture.md`
- Security guidance: `docs/security.md`

## Current Platform and Reporting Baseline

Existing behavior intentionally keeps platform roles from implicitly bypassing school-scoped student, teacher, guardian, administration, and report permissions. Existing report lifecycle expansion remains school-scoped and does not grant platform/support users cross-school report output access.

## Approved Operation Boundary

OpenAPI must define operation IDs and routes for this backend slice before implementation. Proposed operation mapping:

| Method | Route | Proposed Operation ID | Boundary |
|--------|-------|-----------------------|----------|
| `GET` | `/api/v1/platform/schools` | `listPlatformSchoolSummaries` | List minimized school operational summaries with documented filters, sorting, pagination, and small-count suppression |
| `GET` | `/api/v1/platform/reporting/overview` | `getPlatformReportingOverview` | Return cross-school aggregate report health and lifecycle summaries without raw outputs or school-owned detail records |
| `POST` | `/api/v1/platform/support-access` | `requestSupportAccess` | Create or evaluate a target-school support access decision using reason code, target school, opt-in, and internal approval requirements |
| `GET` | `/api/v1/platform/support-access/{supportAccessId}` | `getSupportAccessDecision` | Retrieve minimized support access decision state for authorized platform/support actors |
| `POST` | `/api/v1/platform/support-access/{supportAccessId}/approve` | `approveSupportAccess` | Record internal platform approval where target-school opt-in also exists |
| `POST` | `/api/v1/platform/support-access/{supportAccessId}/revoke` | `revokeSupportAccess` | Revoke an active support access decision before its 24-hour expiration |
| `POST` | `/api/v1/schools/{schoolId}/support-opt-ins` | `createSchoolSupportOptIn` | Record target-school opt-in for a documented support purpose without granting operational write access; opt-in expires after 24 hours |
| `POST` | `/api/v1/schools/{schoolId}/support-opt-ins/{supportOptInId}/revoke` | `revokeSchoolSupportOptIn` | Revoke a target-school opt-in before support drill-down continues |
| `GET` | `/api/v1/platform/support/schools/{schoolId}/diagnostics` | `getSupportSchoolDiagnostics` | Return approved redacted read-only support diagnostics for one target school when both approval gates are valid |
| `GET` | `/api/v1/platform/support-audit-events` | `listSupportAuditEvents` | List minimized platform/support audit summaries for authorized audit reviewers |

Target-school opt-in is a school-scoped approval record and must be represented through the dedicated school support opt-in operations above. These operations approve support visibility only; they do not authorize support writes to school-owned operational records.

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths for every approved operation
- platform-scoped permissions for operational overview, reporting overview, support drill-down, support access decision, support approval, support revocation, and support audit review
- school-scoped permission for creating and revoking target-school support opt-ins
- request schemas for target school, reason code, correlation ID, filters, sorting, pagination, support access decision IDs, approval, and revocation
- response schemas for minimized school summaries, reporting overview summaries, support access decisions, support diagnostics, and support audit summaries
- small-count suppression behavior for protected counts below 5
- 24-hour support access expiration behavior
- 24-hour target-school support opt-in expiration behavior
- target-school opt-in and internal platform approval requirements
- stale, expired, revoked, mismatched, missing, unauthorized, validation, not-found, and conflict response envelopes
- tenant-safe denial behavior that does not disclose unauthorized school-owned record existence
- audit expectations with actor, action, outcome, target school where applicable, correlation ID, tenant-safe reason code, and minimized metadata
- explicit exclusions for generated report downloads, raw report outputs, private file metadata, emergency access, support writes, unrestricted impersonation, and unrestricted record search

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelopes for platform school summaries, platform reporting overview, support access decisions, support diagnostics, and audit summaries
- paginated envelopes for platform school summaries and support audit event lists
- validation error envelope
- unauthorized response for unauthenticated or inactive actor access
- forbidden response for authenticated actors without required platform/support permission
- not-found response for missing, unauthorized, mismatched, or cross-tenant targets according to contract
- conflict response for stale, expired, revoked, mismatched, or concurrently changed support approvals
- suppressed-count representation for protected counts below 5

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented include expansion, undocumented sort behavior, undocumented support diagnostic, undocumented support decision state, undocumented audit payload, or authorization exception is approved in this slice.

## Tenant Behavior

- School-owned records continue to use `school_id`.
- Platform operations may summarize across schools only through documented platform routes.
- Existing school-scoped endpoints remain inaccessible to platform/support actors unless an operation-specific future contract explicitly grants access.
- Support drill-down targets exactly one school per active decision.
- Missing, mismatched, stale, revoked, expired, unauthorized, or older-than-24-hour support decisions or target-school opt-ins fail before school-owned diagnostics are returned.

## Authorization Behavior

- All operations require authenticated access and active actor status.
- Platform school summaries require explicit platform-wide operational oversight permission.
- Cross-school reporting overview requires explicit platform-wide reporting permission.
- Support drill-down requires explicit support drill-down permission, target-school opt-in, internal platform approval, target-school match, reason code, correlation ID, and both opt-in and approval age of 24 hours or less.
- Support approval/revocation requires explicit platform support approval permission.
- Target-school opt-in create/revoke requires explicit same-school support opt-in permission.
- Support audit review requires explicit support audit review permission.
- Platform administrator status alone is not sufficient unless paired with the documented permission.

## Validation Behavior

- Platform summary filters, sorting, pagination, and include fields must be documented in OpenAPI.
- Protected counts below 5 must be suppressed.
- Support access decisions must reject missing target school, missing reason code, missing correlation ID, missing target-school opt-in, missing internal platform approval, mismatched actor, mismatched school, stale approval or opt-in, revoked approval or opt-in, expired approval or opt-in, approval or opt-in older than 24 hours, or concurrent approval changes.
- Target-school support opt-ins must reject missing target school, unsupported reason code, inactive school context, unauthorized school actor, mismatched school, expired opt-in reuse, revoked support access decision, and duplicate active opt-in for the same support purpose where OpenAPI declares uniqueness.
- Support diagnostics must reject generated report downloads, raw report outputs, private file metadata, emergency access, unrestricted record search, unrestricted impersonation, and all write actions.
- Audit metadata must reject or redact credentials, bearer tokens, private file paths, raw report outputs, private content, full student/guardian records, full request/response payloads, and unauthorized cross-tenant details.

## Blocked Until Future Specification

These behaviors are outside this implementation boundary until a future spec and OpenAPI update approve them:

- frontend platform/support UI implementation
- support write actions against school-owned records
- account reactivation, report retry/cancel, school status changes, or other operational mutations through support access
- generated report downloads for support users
- raw report output or private file metadata visibility
- emergency or break-glass access
- unrestricted impersonation of school users
- unrestricted raw database or record search
- advanced assessment and content types
- billing, payroll, accounting, messaging, notifications, live classroom, or video conferencing behavior
- permanent purge, legal hold, or anonymization workflows

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- explicit platform permission checks for overview, reporting overview, support drill-down, approval, revocation, and audit review
- platform administrator without explicit permission denied
- platform/support users unable to use existing school-scoped endpoints as implicit bypasses
- small-count suppression below 5
- target-school opt-in required before support diagnostics
- school-scoped support opt-in create and revoke behavior
- internal platform approval required before support diagnostics
- 24-hour approval expiration and older-than-24-hour denial
- 24-hour target-school opt-in expiration and older-than-24-hour opt-in denial
- stale, revoked, expired, mismatched, and concurrently changed support approval denials
- support diagnostics remaining read-only and redacted
- generated report downloads, raw report outputs, private file metadata, emergency access, unrestricted record search, and support writes rejected
- tenant-safe audit events for allowed, denied, validation, lookup, approval, revocation, expiration, and conflict outcomes
- audit metadata redaction for credentials, tokens, private paths, raw outputs, private content, full records, and unauthorized cross-tenant details
