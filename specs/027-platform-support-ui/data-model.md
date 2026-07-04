# Data Model: Platform Support Access and Cross-School Oversight UI

## PlatformSupportWorkspaceContext

**Purpose**: Frontend-safe context required before any platform support
workspace screen loads cross-school summaries, support decisions, diagnostics,
or audit data.

**Fields**: `actorId`, `platformAccessState`, `operationalOversightAccessState`,
`platformReportingAccessState`, `supportAccessState`, `supportApprovalState`,
`supportAuditReviewState`, `selectedSchoolId`, `selectedSupportAccessId`,
`workspaceStatus`, `defaultRoute`, `feedbackState`.

**Source**: Approved authenticated session, current-user or permission context,
and approved platform/support access behavior.

**Rules**:

- Platform support workspace screens require authenticated active actor state.
- Platform Operational Oversight is the default route after authenticated
  access and operational oversight permission are confirmed.
- Platform administrator status alone is not sufficient; explicit
  platform/support permissions are required.
- Missing permission, inactive actor, unavailable contract, validation,
  conflict, not-found, denied, stale response, and temporary-unavailable
  remain distinct states.
- Existing school-scoped endpoints are not used to load platform support data.

## PlatformOperationalSummaryView

**Purpose**: Minimized cross-school operational view for platform oversight.

**Fields**: `items`, `pagination`, `filters`, `sorting`, `loading`,
`emptyState`, `suppressionState`, `feedbackState`, `staleRequestKey`.

**Source**: `listPlatformSchoolSummaries`.

**Rules**:

- Request requires explicit platform-wide operational oversight permission.
- Displayed school fields are limited to documented minimized identity,
  status, aggregate operational indicators, reporting-health summaries,
  lifecycle summaries, support diagnostic indicators, and suppression markers.
- Protected counts below 5 display the documented suppressed-count state.
- Hidden values are not derived from totals, labels, tooltips, filters,
  diagnostics, exports, filenames, or related summaries.
- Empty summaries and no-filter-results are distinct from denied, validation,
  not-found, conflict, unavailable, and stale-response states.

## PlatformReportingOverviewView

**Purpose**: Aggregate cross-school reporting-health presentation for platform
actors.

**Fields**: `summaryGroups`, `filters`, `pagination`, `loading`,
`suppressionState`, `feedbackState`, `staleRequestKey`.

**Source**: `getPlatformReportingOverview`.

**Rules**:

- Request requires explicit platform-wide reporting permission.
- Displayed values are limited to documented aggregate report health,
  lifecycle, retention, output availability, and failure-summary indicators.
- Raw report outputs, generated files, report-run detail, custom report
  detail, private filter payloads, storage paths, and storage keys are not
  represented.
- Suppressed counts and unavailable summary groups remain visibly distinct.

## SupportAccessDecisionListView

**Purpose**: Platform support view of target-school support access decisions.

**Fields**: `items`, `pagination`, `filters`, `loading`, `emptyState`,
`feedbackState`, `staleRequestKey`.

**Source**: `requestSupportAccess` responses, `getSupportAccessDecision`
responses, and any approved decision list behavior if later documented before
implementation.

**Rules**:

- Request and detail surfaces require explicit support access permission.
- If no approved list operation exists, list-like UI must be driven only by
  decision state already returned by approved request/detail/action responses
  or remain unavailable until a contract is added.
- Decision state must not expose hidden school-owned detail or unauthorized
  target existence.

## SupportAccessDecisionView

**Purpose**: Frontend-safe representation of one target-school support access
decision.

**Fields**: `id`, `targetSchoolId`, `targetSchoolLabel`, `supportActorId`,
`reasonCode`, `purpose`, `correlationId`, `decisionState`,
`targetSchoolOptInState`, `internalPlatformApprovalState`, `targetSchoolMatch`,
`approvedAt`, `expiresAt`, `revokedAt`, `revocationReasonCode`,
`expiryLabel`, `diagnosticsAvailable`, `feedbackState`, `refreshState`,
`staleRequestKey`.

**Source**: `requestSupportAccess`, `getSupportAccessDecision`,
`approveSupportAccess`, and `revokeSupportAccess`.

**Rules**:

- Decision state is one of `requested`, `pending`, `approved`, `denied`,
  `expired`, or `revoked`.
- Requested or pending decisions do not load diagnostics.
- Diagnostics require valid target-school opt-in and internal platform
  approval for the same target school and no older than 24 hours.
- Expired, revoked, denied, stale, mismatched, concurrently changed, missing,
  unavailable, or no-longer-authorized decisions remove diagnostics access.
- One decision cannot be reused for another school.
- Returned decision state is authoritative after approval or revocation.

## SupportOptInStateView

**Purpose**: Display-only representation of school-side target-school opt-in
state for support access.

**Fields**: `optInState`, `targetSchoolId`, `approvedAt`, `expiresAt`,
`revokedAt`, `reasonCode`, `expiryLabel`, `feedbackState`.

**Source**: Approved platform/support decision responses that include opt-in
state. Dedicated school-admin opt-in create/revoke operations are outside this
frontend slice.

**Rules**:

- Opt-in state is display-only.
- School-admin target-school opt-in create/revoke controls are not
  represented in this feature.
- Revoked, denied, expired, older-than-24-hour, missing, or mismatched opt-in
  blocks diagnostics.

## InternalPlatformApprovalView

**Purpose**: UI representation of platform-side approval state for support
drill-down.

**Fields**: `approvalState`, `approverId`, `supportActorId`, `targetSchoolId`,
`approvedAt`, `expiresAt`, `revokedAt`, `reasonCode`, `expiryLabel`,
`feedbackState`.

**Source**: `getSupportAccessDecision`, `approveSupportAccess`, and
`revokeSupportAccess`.

**Rules**:

- Approval controls require explicit platform support approval or revocation
  permission.
- Approval must match the support actor, target school, and decision context.
- Revoked, denied, expired, stale, older-than-24-hour, missing, or mismatched
  approval blocks diagnostics.
- Approval/revocation actions do not write to school-owned operational
  records.

## SupportDiagnosticView

**Purpose**: Redacted read-only diagnostic presentation for one approved
target school.

**Fields**: `targetSchoolId`, `targetSchoolStatus`, `decisionId`,
`decisionExpiresAt`, `diagnosticGroups`, `reportingHealthSummary`,
`lifecycleSummary`, `suppressionState`, `redactionState`, `loading`,
`feedbackState`, `refreshState`, `staleRequestKey`.

**Source**: `getSupportSchoolDiagnostics`.

**Rules**:

- Request requires explicit support drill-down permission and active valid
  approval gates.
- Data is read-only and redacted.
- Suppressed counts below 5 and redacted fields remain visible as
  suppression/redaction indicators.
- Student detail, guardian detail, teacher detail, report-run detail,
  custom-report detail, school-admin detail, generated report downloads, raw
  report outputs, private file metadata, emergency access, unrestricted
  search, impersonation, and write actions are not represented.
- Automatic refresh removes diagnostics access when returned decision or
  diagnostics state becomes expired, revoked, denied, stale, mismatched,
  unavailable, or no longer authorized.

## SupportAuditReviewView

**Purpose**: Minimized audit event list for support oversight and governance.

**Fields**: `items`, `pagination`, `filters`, `loading`, `emptyState`,
`feedbackState`, `staleRequestKey`.

**Source**: `listSupportAuditEvents`.

**Rules**:

- Request requires explicit support audit review permission.
- Displayed audit fields are limited to actor, action, outcome, target school
  where safe, correlation ID, reason code, timestamp, and minimized metadata.
- Denied cross-tenant attempts must not reveal unauthorized protected target
  details.
- Audit review never reconstructs credentials, bearer tokens, private paths,
  raw outputs, private content, full records, full request/response payloads,
  or unauthorized cross-tenant detail.

## PlatformSupportFeedbackState

**Purpose**: Canonical platform support UI state used across summaries,
decisions, diagnostics, approvals, and audit surfaces.

**Fields**: `kind`, `messageKey`, `recoverable`, `safeDetails`.

**States**:

```text
idle
loading
empty
unauthorized
forbidden
tenant_mismatch
validation
not_found
conflict
expired
revoked
denied
stale_response
suppressed
redacted
temporary_unavailable
contract_unavailable
diagnostics_unavailable
unsupported_action
```

**Rules**:

- `safeDetails` may include operation ID, generic state kind, field label,
  route name, decision state, approval-gate kind, diagnostic group label, or
  safe correlation/request ID only.
- Hidden counts, raw report outputs, generated files, private file metadata,
  private content, credentials, token values, role internals, full records,
  full payloads, raw denial reasons, and unauthorized school-owned record
  existence are excluded.

## PlatformSupportRefreshState

**Purpose**: Automatic and manual refresh state for visible support decisions
and diagnostics.

**Fields**: `enabled`, `lastRefreshedAt`, `pendingRequestKey`,
`currentDecisionId`, `currentTargetSchoolId`, `refreshReason`,
`feedbackState`.

**Source**: Visible decision/diagnostics route state and approved
decision/diagnostics responses.

**Rules**:

- Automatic refresh runs only while visible state depends on active or pending
  approval gates.
- Manual refresh is always available where decision or diagnostics state is
  visible and the operation is approved.
- Refresh responses are ignored when route, selected school, filters,
  decision, target school, authentication, permission, approval, revocation,
  or opt-in state changes before response application.
- Meaningful changes render visible status updates without moving keyboard
  focus.
