# Data Model: Reporting Workspace UI

## ReportingWorkspaceContext

**Purpose**: Frontend-safe context required before any reporting workspace
screen loads tenant-owned report, output, catalog, or definition data.

**Fields**: `schoolId`, `schoolTimezone`, `reportAccessState`,
`reportLifecycleAccessState`, `reportDefinitionAccessState`, `selectedReportRunId`,
`selectedReportDefinitionId`, `workspaceStatus`, `defaultRoute`,
`feedbackState`.

**Source**: Approved authenticated session, active school context, current-user
or permission context, and approved reporting access behavior.

**Rules**:

- Reporting workspace screens require authenticated access.
- Active permitted school context is required before reporting requests.
- Active school timezone is required before timestamped reporting data is
  displayed.
- Report catalog, report requests, report history, and downloads require
  report permission.
- Lifecycle actions require report lifecycle permission.
- Custom definition screens require report-definition permission.
- Missing school, inactive school, denied access, unavailable catalog,
  conflict, expired output, and stale response remain distinct states.

## ReportCatalogView

**Purpose**: Read-only catalog of approved reporting options for the active
school.

**Fields**: `domains`, `fields`, `filters`, `operators`, `grouping`,
`sorting`, `outputFormats`, `complexityLimits`, `loading`, `feedbackState`,
`staleRequestKey`.

**Source**: `getReportCatalog`.

**Rules**:

- Request uses active school tenant context only.
- Catalog drives built-in report requests and custom definition forms.
- Unsupported domains, fields, filters, operators, grouping, sorting, and
  formats are not represented as selectable options.
- Complexity limits are visible before definition submission.
- Stale catalog results are ignored when active school, authentication,
  permission, route, or selected definition changes.

## ReportRequestFlow

**Purpose**: User-facing configuration and submission path for built-in reports
and active custom report definitions.

**Fields**: `reportType`, `reportDefinitionId`, `filters`, `outputFormats`,
`catalogValidationState`, `submissionState`, `acceptedReportRun`,
`feedbackState`.

**Source**: `requestReport`, using values constrained by `ReportCatalogView`
and loaded custom definitions.

**Rules**:

- Request is blocked until active school, report permission, catalog, selected
  report, filters, and formats are valid.
- Built-in requests use documented report type and filters.
- Custom requests use active custom definition identifiers only.
- Unsupported filters, references, formats, and empty output format selections
  remain blocked or mapped to validation state.
- Accepted response creates or updates visible report history/detail state
  without inventing generated output availability.

## ReportRunHistoryView

**Purpose**: Paginated school-scoped list and detail source for report runs.

**Fields**: `items`, `pagination`, `filters`, `includeDeleted`, `loading`,
`emptyState`, `refreshState`, `lastRefreshedAt`, `feedbackState`,
`staleRequestKey`.

**Source**: `listReports`.

**Rules**:

- Request uses tenant context and documented page, per-page, report type,
  generation status, report source, and include-deleted filters only.
- Default list omits soft-deleted runs unless include-deleted is explicitly
  selected.
- Empty history and no-filter-results are distinct from denied, validation,
  not-found, conflict, unavailable, and stale-response states.
- Visible requested or generating runs trigger automatic refresh and always
  allow manual refresh.
- Refresh responses are ignored after route, active school, authentication,
  filters, selected report, selected definition, or dialog state changes.

## ReportRunView

**Purpose**: Frontend-safe representation of one report run in list/detail,
download, and lifecycle surfaces.

**Fields**: `id`, `schoolId`, `requestedByUserId`, `reportType`,
`reportSource`, `filterSummary`, `outputFormats`, `generationStatus`,
`sourceReportRunId`, `supersededByReportRunId`, `reportDefinitionId`,
`reportDefinitionSnapshotId`, `deletedAt`, `cancellationReasonCode`,
`failureReasonCode`, `generatedAt`, `outputExpiresAt`, `outputsAvailable`,
`outputs`, `timestampLabels`, `feedbackState`.

**Source**: `ReportRun` from report list, request, retry, cancel, delete, or
restore responses.

**Rules**:

- Generation status is one of `requested`, `generating`, `generated`,
  `failed`, or `canceled`.
- Soft-delete visibility is separate from generation status.
- Source and superseding run references are shown only where returned and must
  not expose inaccessible report details.
- Timestamps render in active school timezone.
- Returned run state is authoritative after lifecycle actions; optimistic
  state cannot contradict returned data.

## ReportOutputAvailabilityView

**Purpose**: Per-format output availability and download gating for a report
run.

**Fields**: `format`, `availability`, `downloadEnabled`, `expiryLabel`,
`downloadState`, `feedbackState`.

**Source**: `ReportOutput` objects and `downloadReport`.

**Rules**:

- Format is limited to documented `pdf`, `csv`, and catalog-approved `xlsx`.
- Availability is one of `pending`, `available`, `failed`, `expired`, or
  `unsupported`.
- Download is enabled only for `available` outputs for authorized same-school
  report runs.
- Expired downloads show expired-output feedback and do not promise automatic
  regeneration.
- Private storage paths, storage keys, generated report contents, and
  cross-tenant file existence are not represented.

## ReportLifecycleActionView

**Purpose**: Permission-gated retry, cancellation, soft-delete, and restore
action state for report runs.

**Fields**: `reportRunId`, `availableActions`, `reasonCodeOptions`,
`confirmationState`, `submissionState`, `conflictState`, `feedbackState`.

**Source**: Report run state, reporting permissions, and approved lifecycle
operation responses.

**Rules**:

- Retry appears only for approved failed or expired generated runs.
- Cancellation appears only for requested or generating runs where cancellation
  remains allowed.
- Delete and restore appear only where current soft-delete state and
  permission allow them.
- Reason code choices are predefined and tenant-safe; free text is not
  represented.
- Conflict, validation, denied, not-found, and tenant-mismatch responses clear
  pending action state and preserve returned/loaded report state.

## ReportDefinitionListView

**Purpose**: Paginated school-scoped list of custom report definitions.

**Fields**: `items`, `pagination`, `lifecycleStateFilter`, `includeDeleted`,
`loading`, `emptyState`, `feedbackState`, `staleRequestKey`.

**Source**: `listReportDefinitions`.

**Rules**:

- Request uses tenant context and documented page, per-page, lifecycle state,
  and include-deleted filters only.
- Default list omits deleted definitions unless include-deleted is explicitly
  selected.
- Empty definitions and no-filter-results are distinct from denied,
  validation, unavailable, and stale-response states.

## ReportDefinitionView

**Purpose**: Frontend-safe representation of one school-owned custom report
definition.

**Fields**: `id`, `schoolId`, `name`, `description`, `domain`, `fields`,
`filters`, `grouping`, `sorting`, `outputFormats`, `lifecycleState`, `version`,
`createdByUserId`, `updatedByUserId`, `deletedAt`, `timestampLabels`,
`editBoundary`, `feedbackState`.

**Source**: Report definition list, detail, create, update, activate,
deactivate, delete, or restore responses.

**Rules**:

- Lifecycle state is one of `draft`, `active`, `inactive`, or `deleted`.
- New definitions start as draft.
- Active definitions allow metadata-only edits.
- Structural edits require inactive or draft state.
- Restored definitions return to inactive and require separate activation
  before report request.
- Definition names must surface duplicate-name conflict feedback without
  exposing cross-tenant details.
- Timestamps render in active school timezone.

## ReportDefinitionEditorView

**Purpose**: Catalog-bound form state for creating and updating custom report
definitions.

**Fields**: `name`, `description`, `domain`, `selectedFields`, `filters`,
`grouping`, `sorting`, `outputFormats`, `complexityUsage`,
`catalogValidationState`, `submissionState`, `feedbackState`.

**Source**: `ReportCatalogView`, `createReportDefinition`, and
`updateReportDefinition`.

**Rules**:

- Domain, field, filter, operator, grouping, sorting, and format controls are
  limited to catalog-approved options.
- Maximum selected values are 25 fields, 10 filters, 2 grouping levels, and 3
  sort fields.
- Active definitions restrict edits to name and description.
- No arbitrary query text, script, uploaded template, hidden field, private
  file path, output reference, credential, unapproved domain, or custom code is
  represented.

## ReportingAnnouncementState

**Purpose**: Polite assistive-technology announcements for meaningful report
state changes.

**Fields**: `messageKey`, `reportRunId`, `stateKind`, `timestamp`,
`announcedRequestKey`.

**Source**: Automatic refresh, lifecycle action responses, and download
availability changes.

**Rules**:

- Announcements are polite, not assertive.
- Meaningful changes include status transition, output availability change,
  expired output, lifecycle action completion, and conflict requiring user
  attention.
- Routine refresh with no changed state does not announce.
- Keyboard focus remains on the user's current control or dialog.

## ReportingFeedbackState

**Purpose**: Canonical reporting UI state used across catalog, request,
history, detail, download, lifecycle, and custom definition surfaces.

**Fields**: `kind`, `messageKey`, `recoverable`, `safeDetails`.

**States**:

```text
idle
loading
empty
unauthorized
forbidden
tenant_mismatch
inactive_school
no_active_school
unavailable_catalog
validation
not_found
conflict
output_expired
unsupported_page_size
temporary_unavailable
stale_response
download_unavailable
```

**Rules**:

- `safeDetails` may include operation ID, generic state kind, field label,
  route name, output format, lifecycle action kind, or safe correlation/request
  ID only.
- Private filter payloads, report contents, storage paths, storage keys,
  cross-school identifiers, hidden fields, credentials, role internals, token
  values, raw denial reasons, and unauthorized report existence are excluded.
