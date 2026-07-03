# UI Contract: Platform Support Access and Cross-School Oversight UI

## Purpose

This contract maps Platform Support Access and Cross-School Oversight UI
surfaces to approved OpenAPI operations, frontend state boundaries, and blocked
behavior. It is a frontend consumption contract, not a backend API change.

## Contract Readiness Gate

Frontend implementation must confirm each consumed platform/support operation
is present in `api/openapi.yaml` and the platform contract mirror before any
runtime route calls it. Proposed operation names from
`specs/013-platform-support-access/contracts/backend-platform-support-access.md`
are not sufficient until promoted into the aggregate OpenAPI contract.

## Approved Operations

| UI surface | Operation ID | Method/path | Required context |
|------------|--------------|-------------|------------------|
| Platform school summaries | `listPlatformSchoolSummaries` | `GET /api/v1/platform/schools` | Authenticated active actor, platform-wide operational oversight permission |
| Platform reporting overview | `getPlatformReportingOverview` | `GET /api/v1/platform/reporting/overview` | Authenticated active actor, platform-wide reporting permission |
| Request support access | `requestSupportAccess` | `POST /api/v1/platform/support-access` | Authenticated active support actor, support access permission, target school, reason code, purpose, correlation metadata, target-school opt-in state returned by contract |
| Support access decision detail | `getSupportAccessDecision` | `GET /api/v1/platform/support-access/{supportAccessId}` | Authenticated active actor with support access or approval permission |
| Support approval | `approveSupportAccess` | `POST /api/v1/platform/support-access/{supportAccessId}/approve` | Authenticated active platform approver, support approval permission, matching target-school opt-in |
| Support revocation | `revokeSupportAccess` | `POST /api/v1/platform/support-access/{supportAccessId}/revoke` | Authenticated active platform actor, support revocation permission |
| Support diagnostics | `getSupportSchoolDiagnostics` | `GET /api/v1/platform/support/schools/{schoolId}/diagnostics` | Authenticated active support actor, support drill-down permission, valid target-school opt-in, valid internal platform approval, target-school match |
| Support audit review | `listSupportAuditEvents` | `GET /api/v1/platform/support-audit-events` | Authenticated active audit reviewer, support audit review permission |

## Explicitly Not Consumed

| Operation | Reason |
|-----------|--------|
| `createSchoolSupportOptIn` | School-admin target-school opt-in create screens are outside this platform-only frontend slice |
| `revokeSchoolSupportOptIn` | School-admin target-school opt-in revoke screens are outside this platform-only frontend slice |
| Existing school-scoped student, guardian, teacher, administration, and reporting endpoints | These endpoints are not platform/support bypass paths |

## Route Surfaces

| Route intent | Behavior |
|--------------|----------|
| Platform support workspace root | Render Platform Operational Oversight by default after session and operational oversight gates are confirmed |
| Platform Operational Oversight | Load minimized school summaries and reporting overview indicators only through approved platform operations |
| Support Access Decisions | Request or view support access decisions, approval-gate state, target-school match, display-only opt-in state, and 24-hour expiry |
| Support Diagnostics | Load redacted read-only diagnostics for one target school only after both approval gates are valid |
| Support Audit Review | Load minimized support audit events with approved filters and pagination |
| Direct target route | Show safe unavailable, not-found, denied, conflict, expired, revoked, or tenant-safe feedback without exposing unauthorized target existence |

## Service Boundary

All HTTP access must go through platform support service modules. Components
and route views must not call Axios directly.

Service functions:

- `listPlatformSchoolSummaries({ page, perPage, filters, sort })`
- `getPlatformReportingOverview({ page, perPage, filters, sort })`
- `requestSupportAccess({ targetSchoolId, reasonCode, purpose, correlationId })`
- `getSupportAccessDecision({ supportAccessId })`
- `approveSupportAccess({ supportAccessId, reasonCode })`
- `revokeSupportAccess({ supportAccessId, reasonCode })`
- `getSupportSchoolDiagnostics({ schoolId, supportAccessId })`
- `listSupportAuditEvents({ page, perPage, filters, sort })`

Mapping requirements:

- Submit only documented parameters and request fields.
- Parse paginated envelopes through shared pagination mappers.
- Parse success/accepted envelopes through platform support contract mappers.
- Normalize error envelopes into safe feedback states.
- Drop undocumented response fields.
- Preserve suppressed-count and redacted-field representations.
- Preserve support decision, opt-in, approval, revocation, and expiry state.
- Do not create service functions for school-admin opt-in create/revoke in
  this frontend slice.
- Do not call school-scoped operational endpoints as support shortcuts.

## UI State Contract

Platform support surfaces must distinguish:

- loading
- empty
- unauthorized
- forbidden
- tenant-mismatch
- validation
- not-found
- conflict
- expired
- revoked
- denied
- stale-response
- suppressed
- redacted
- diagnostics-unavailable
- contract-unavailable
- temporary-unavailable
- unsupported-action

True empty states must not be shown for missing permission, denial, validation,
target not-found, contract unavailable, conflict, expired approval, revoked
decision, unavailable diagnostics, or stale response.

## Capability Gates

- No platform support data request before authenticated active actor state is
  confirmed.
- No platform school summaries without operational oversight permission.
- No platform reporting overview without platform reporting permission.
- No support access decision request or detail without support access
  permission.
- No support diagnostics without support drill-down permission, target-school
  opt-in, internal platform approval, target-school match, and both gates no
  older than 24 hours.
- No support approval or revocation controls without platform support approval
  or revocation permission.
- No school-admin opt-in create/revoke controls in this feature.
- No support audit review without support audit review permission.
- No automatic refresh outside visible support decision or diagnostics state
  that depends on active or pending approval gates.
- No generated report download, raw output viewing, private file metadata,
  emergency access, unrestricted impersonation, unrestricted search, support
  writes, account reactivation, report retry/cancel, school status change, or
  other school-owned operational mutation.

## Platform Summary Contract

Allowed:

- Display documented school identity/status fields.
- Display approved aggregate operational indicators.
- Display approved reporting-health, lifecycle, output availability, and
  support diagnostic indicators.
- Display documented suppressed-count markers.
- Display unavailable summary groups where returned.

Blocked:

- Student, guardian, teacher, report-run, custom-report, school-admin, or
  file detail records.
- Raw report outputs, generated files, private file paths, storage keys,
  private content, full payloads, hidden fields, credentials, token values,
  and unauthorized cross-tenant details.
- Deriving suppressed values from totals, labels, filters, tooltips,
  diagnostics, filenames, or related fields.

## Support Decision and Refresh Contract

Decision states:

- `requested`
- `pending`
- `approved`
- `denied`
- `expired`
- `revoked`

Approval-gate rules:

- Support diagnostics require active matching target-school opt-in.
- Support diagnostics require active matching internal platform approval.
- Both approval gates must be no older than 24 hours.
- Expired, revoked, denied, stale, mismatched, missing, or concurrently changed
  gates remove diagnostics access.
- One decision applies to exactly one target school.

Refresh rules:

- Automatic refresh runs while visible support decision or diagnostics state
  depends on active or pending approval gates.
- Manual refresh remains available on visible decision and diagnostics
  surfaces.
- Refresh responses are ignored after route, selected school, filters,
  support decision, target school, authentication, permission, approval,
  revocation, or opt-in state changes.
- Returned decision/diagnostics state is authoritative.
- Meaningful access changes render visible updates without moving keyboard
  focus.

## Diagnostics Contract

Allowed:

- Target school identity/status where approved.
- Redacted operational diagnostic indicators.
- Approved report health and lifecycle summaries.
- Approved troubleshooting metadata.
- Suppressed-count and redacted-field indicators.
- Active support decision context and expiry.

Blocked:

- Generated report downloads.
- Raw report outputs.
- Private file metadata.
- Student, guardian, teacher, report-run, custom-report, school-admin, or file
  details.
- Emergency access.
- Unrestricted impersonation.
- Unrestricted raw record search.
- Account reactivation, report retry/cancel, school status changes, or any
  school-owned operational writes.

## Audit Review Contract

Allowed audit fields:

- actor
- action
- outcome
- target school where safe
- correlation ID
- tenant-safe reason code
- timestamp
- minimized metadata returned by contract

Blocked audit fields:

- credentials
- bearer tokens
- private paths
- raw report outputs
- private content
- full student/guardian/teacher/school-admin records
- full request or response payloads
- unauthorized cross-tenant target details
- raw denial internals

## Sensitive Data Contract

UI state, diagnostics, visible errors, filenames, route labels, audit review,
and test output must not include:

- hidden counts
- raw report outputs
- generated files
- private file metadata
- private content
- credentials
- token values
- role internals
- full records
- full payloads
- private paths
- storage keys
- raw backend denial reasons
- unauthorized school-owned record existence

Allowed diagnostics:

- operation ID
- generic state kind
- field label for validation
- decision state
- approval-gate kind
- diagnostic group label
- current route name
- safe correlation/request ID if provided by shared error normalization

## Verification Contract

Implementation must include Vitest coverage for:

- approved operation mapping
- no undocumented parameter submission
- contract-unavailable state when platform/support operations are absent
- authenticated active actor gate
- platform operational oversight permission gate
- platform reporting overview permission gate
- support access permission gate
- support approval/revocation permission gates
- support drill-down permission and approval-gate checks
- support audit review permission gate
- Platform Operational Oversight as root default
- platform school summary loading, filters, pagination, empty states, and
  suppressed-count rendering
- reporting overview loading and suppressed-count rendering
- support access request and decision states
- display-only target-school opt-in state
- internal platform approval state
- 24-hour expiry visibility
- approval and revocation state transitions
- automatic refresh and manual refresh
- diagnostics access removal after expiry, revocation, denial, stale state,
  mismatch, unavailable state, or no-longer-authorized response
- redacted diagnostics rendering
- support audit review filters, pagination, empty states, and safe metadata
- unsupported action hiding/blocking
- stale-response protection
- safe diagnostics redaction
