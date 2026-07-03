# Quickstart: Platform Support Access and Cross-School Oversight UI

## Prerequisites

- Feature 015 Frontend Architecture Baseline is implemented.
- Feature 016 System Administrator Shell and Dashboard Foundation is
  implemented where shared protected shell patterns are required.
- Feature 017 Authentication and Session Foundation UI is implemented,
  including current-user hydration, session-expired, unauthorized, forbidden,
  inactive-user, and safe permission loading.
- Feature 026 Reporting Workspace UI is complete and available as a reference
  for async state refresh, stale-response protection, and no-sensitive-data
  diagnostics.
- Backend platform support access from `specs/013-platform-support-access/` is
  implemented and contract-compliant, or implementation is blocked until the
  required platform/support operations are promoted into `specs/api/openapi.yaml` and
  the platform contract mirror.
- Implementation confirms how authenticated session or approved access
  behavior exposes platform-wide operational oversight, platform reporting,
  support access, support drill-down, support approval/revocation, and support
  audit review permissions.

## Contract Review

Promotion status, 2026-07-03: `specs/api/openapi.yaml` and
`specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` include
`listPlatformSchoolSummaries`, `getPlatformReportingOverview`,
`requestSupportAccess`, `getSupportAccessDecision`, `approveSupportAccess`,
`revokeSupportAccess`, `getSupportSchoolDiagnostics`, and
`listSupportAuditEvents`. Redocly validation passed for
`aggregate@v1 schoolmaster-platform@v1`; the platform mirror still reports the
pre-existing unused report component warnings for `ReportRunId`, `ReportRun`,
`ReportRequest`, and `OutputExpired`.

Before frontend implementation:

1. Confirm `specs/api/openapi.yaml` and the platform contract mirror include
   `listPlatformSchoolSummaries`.
2. Confirm platform school summaries expose only approved minimized school
   identity/status fields, aggregate operational indicators, reporting-health
   summaries, lifecycle summaries, diagnostic indicators, and suppression
   markers.
3. Confirm protected counts below 5 use the documented suppressed-count
   representation.
4. Confirm `specs/api/openapi.yaml` and the platform contract mirror include
   `getPlatformReportingOverview`.
5. Confirm reporting overview excludes raw report outputs, generated files,
   private payloads, report-run detail, custom-report detail, and private
   storage metadata.
6. Confirm `specs/api/openapi.yaml` and the platform contract mirror include
   `requestSupportAccess`.
7. Confirm support access request fields are documented for target school,
   reason code, purpose, and correlation metadata.
8. Confirm `specs/api/openapi.yaml` and the platform contract mirror include
   `getSupportAccessDecision`.
9. Confirm support decision states are requested, pending, approved, denied,
   expired, and revoked.
10. Confirm support decision responses include target-school opt-in state,
    internal platform approval state, target-school match, revocation state,
    and 24-hour expiry visibility where needed by UI.
11. Confirm `specs/api/openapi.yaml` and the platform contract mirror include
    `approveSupportAccess` and `revokeSupportAccess`.
12. Confirm approval/revocation permissions and conflict responses are
    documented.
13. Confirm `specs/api/openapi.yaml` and the platform contract mirror include
    `getSupportSchoolDiagnostics`.
14. Confirm diagnostics responses are read-only and redacted and exclude
    generated report downloads, raw outputs, private file metadata, emergency
    access, unrestricted impersonation, unrestricted search, and writes.
15. Confirm `specs/api/openapi.yaml` and the platform contract mirror include
    `listSupportAuditEvents`.
16. Confirm support audit event fields are minimized to actor, action,
    outcome, target school where safe, correlation ID, reason code, timestamp,
    and minimized metadata.
17. Confirm school-admin `createSchoolSupportOptIn` and
    `revokeSchoolSupportOptIn` operations are not consumed by this UI slice.
18. Confirm validation, unauthorized, forbidden, tenant-mismatch, not-found,
    conflict, expired, revoked, denied, stale-response,
    temporary-unavailable, suppressed-count, and redacted-field semantics are
    documented or normalized by existing frontend error mapping.

## Component Boundary Review

- Route views remain composition surfaces.
- Platform support services own all HTTP access and contract mapping.
- Platform support composables coordinate platform permission gates, summary
  filters, reporting overview, support decision state, approval/revocation,
  display-only opt-in state, diagnostics, auto-refresh, manual refresh, audit
  filters, stale-response protection, and safe feedback.
- Components receive mapped data through props and emit user intent.
- Element Plus component tags remain PascalCase.
- Display text is centralized through Vue I18n.
- Tailwind handles layout and spacing around Element Plus primitives.

## Manual Scenario Review

### Platform Context Gates

- Sign in as an authenticated platform administrator with platform-wide
  operational oversight permission.
- Open the platform support workspace root.
- Verify Platform Operational Oversight appears by default after session and
  permission context resolve.
- Sign in with only school-scoped roles and verify platform support data
  requests remain blocked or render denied state.
- Sign in with platform administrator role but without explicit platform
  support permissions and verify denied state does not expose school counts,
  reporting health, support activity, or diagnostics.
- Verify school-scoped administration, reporting, teacher, student, and
  guardian endpoints are not called as support shortcuts.

### Platform Operational Oversight

- Load platform school summaries.
- Verify only minimized school identity/status, operational indicators,
  reporting-health summaries, lifecycle summaries, support diagnostic
  indicators, and suppression markers appear.
- Verify protected counts below 5 show suppression state and no hidden values
  can be derived from labels, tooltips, filters, diagnostics, filenames, or
  related summaries.
- Load cross-school reporting overview.
- Verify aggregate report health, lifecycle, retention, output availability,
  and failure-summary indicators appear without raw outputs, private payloads,
  generated files, report-run detail, or custom-report detail.
- Verify empty summaries, no-filter-results, denied, validation, conflict,
  unavailable, not-found, and stale-response states are distinct.

### Support Access Decisions

- Sign in as a support user with support access permission.
- Request support access with documented target school, reason code, purpose,
  and correlation metadata.
- Verify requested or pending state appears when internal platform approval is
  not yet active.
- Verify display-only target-school opt-in state, revocation state, target
  match, and 24-hour expiry appear where returned.
- Verify no school-admin target-school opt-in create/revoke controls appear.
- Sign in as an actor with support approval/revocation permission.
- Approve and revoke a support access decision where the contract allows it.
- Verify returned decision state is authoritative and no school-owned
  operational writes are available.

### Redacted Diagnostics

- Open diagnostics for a target school with active target-school opt-in and
  internal platform approval no older than 24 hours.
- Verify only approved redacted diagnostic fields render.
- Verify active decision context and expiry are visible.
- Verify suppressed counts and redacted fields remain indicators, not hidden
  values.
- Expire, revoke, deny, stale, mismatch, or invalidate the decision or opt-in
  while diagnostics is open.
- Verify automatic refresh removes diagnostics access and shows safe feedback.
- Verify manual refresh is available.
- Verify generated report downloads, raw report output viewing, private file
  metadata access, emergency access, unrestricted search, impersonation,
  account reactivation, report retry/cancel, school status changes, and
  school-owned writes are absent or blocked.

### Support Audit Review

- Sign in as an actor with support audit review permission.
- Open support audit review.
- Verify audit event filters and pagination use documented behavior.
- Verify event summaries include only actor, action, outcome, safe target
  school, correlation ID, reason code, timestamp, and minimized metadata.
- Verify denied cross-tenant attempts do not reveal unauthorized protected
  target details.
- Sign in without audit review permission and verify denied state does not
  expose support activity details.

### Stale Response and Diagnostics

- Start summary, reporting overview, decision, approval, revocation,
  diagnostics, or audit requests, then change route, selected school, filters,
  support decision, target school, authentication, permission, approval,
  revocation, or opt-in state before response applies.
- Verify stale response does not overwrite current visible state.
- Verify visible errors, diagnostics, route labels, filenames, audit review,
  and test output omit hidden counts, raw report outputs, generated files,
  private file metadata, private content, credentials, token values, role
  internals, full records, full payloads, raw denial reasons, and unauthorized
  school-owned record existence.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm run test:unit
```

Focused Vitest coverage should include:

- platform support service mappers for `listPlatformSchoolSummaries`
- platform support service mappers for `getPlatformReportingOverview`
- platform support service mappers for `requestSupportAccess`
- platform support service mappers for `getSupportAccessDecision`
- platform support service mappers for `approveSupportAccess` and
  `revokeSupportAccess`
- platform support service mappers for `getSupportSchoolDiagnostics`
- platform support service mappers for `listSupportAuditEvents`
- no undocumented request parameters or fields
- contract-unavailable state for missing platform/support operations
- authenticated active actor gate
- platform operational oversight permission gate
- platform reporting permission gate
- support access permission gate
- support approval/revocation permission gates
- support drill-down permission gate
- support audit review permission gate
- Platform Operational Oversight as workspace root default
- platform summaries pagination, filters, empty states, and suppression state
- reporting overview aggregate and suppression state
- support access request and decision states
- display-only target-school opt-in state
- internal platform approval state
- 24-hour expiry visibility
- approval and revocation action handling
- automatic refresh and manual refresh
- diagnostics access removal after expiry, revocation, denial, stale state,
  mismatch, unavailable state, or no-longer-authorized response
- redacted diagnostics rendering
- unsupported action hiding/blocking
- support audit review filters, pagination, empty states, denied states, and
  safe metadata
- unauthorized, forbidden, tenant-mismatch, validation, not-found, conflict,
  expired, revoked, denied, stale-response, temporary-unavailable,
  contract-unavailable, suppressed-count, and redacted-field mapping
- stale-response protection
- safe diagnostics redaction

Run build checks if available:

```bash
npm run build
```

Run OpenAPI validation if platform/support contracts are promoted or changed:

```bash
npx @redocly/cli lint specs/api/openapi.yaml
```

## Acceptance Evidence

Record in implementation PR:

- Operation ID to UI surface mapping.
- Evidence that platform support workspace gates on authenticated active actor
  state and explicit platform/support permissions.
- Evidence that platform support workspace root opens Platform Operational
  Oversight by default.
- Evidence that platform summaries and reporting overview preserve
  minimization, redaction, and suppressed-count behavior.
- Evidence that no school-scoped endpoint is used as a support shortcut.
- Evidence that support access decisions show target-school opt-in state,
  internal platform approval state, revocation, target match, and 24-hour
  expiry.
- Evidence that school-admin target-school opt-in create/revoke controls are
  absent.
- Evidence that support approval/revocation controls are permission-gated and
  state-gated.
- Evidence that diagnostics remain read-only, target-school-bound, redacted,
  and unavailable until both approval gates are valid.
- Evidence that visible decision/diagnostics state auto-refreshes and manual
  refresh works.
- Evidence that diagnostics access is removed after returned expiry,
  revocation, denial, stale, mismatched, unavailable, or no-longer-authorized
  state.
- Evidence that support audit review uses minimized metadata only.
- Evidence that timed usability checks meet the SC-002, SC-003, SC-004, and
  SC-010 targets and SC-013 default-route target.
- Evidence that frontend performance checks meet the plan targets for mocked
  rendering, route transitions, decision submission, approval/revocation, and
  refresh behavior, or record an approved deviation.
- Evidence that hidden counts, raw report outputs, generated files, private
  file metadata, private content, credentials, token values, role internals,
  full records, full payloads, raw denial reasons, and unauthorized
  school-owned record existence do not appear in diagnostics or test output.

## Out of Scope Verification

Confirm none of these appear in the platform support UI slice:

- school-admin target-school opt-in create/revoke screens
- generated report downloads
- raw report output viewing
- private file metadata
- emergency or break-glass access
- unrestricted impersonation
- unrestricted raw database or record search
- support writes
- account reactivation
- report retry/cancel
- school status changes
- school-owned operational mutations
- school-scoped endpoint bypasses
- student, guardian, teacher, report-run, custom-report, school-admin, or file
  detail records unless separately approved in a future contract
- billing, payroll, accounting, messaging, notifications, live classroom,
  video conferencing
- permanent purge, legal hold, anonymization
- advanced assessment behavior
- undocumented APIs
