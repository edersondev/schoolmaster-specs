# UI Contract: Reporting Workspace UI

## Purpose

This contract maps Reporting Workspace UI surfaces to approved OpenAPI
operations, frontend state boundaries, and blocked behavior. It is a frontend
consumption contract, not a backend API change.

## Approved Operations

| UI surface | Operation ID | Method/path | Required context |
|------------|--------------|-------------|------------------|
| Report catalog | `getReportCatalog` | `GET /api/v1/report-catalog` | Authenticated session, active school, report/report-definition permission as approved |
| Report history | `listReports` | `GET /api/v1/reports` | Authenticated session, active school, report permission |
| Report request | `requestReport` | `POST /api/v1/reports` | Authenticated session, active school, report permission, valid report type or active custom definition |
| Report retry | `retryReport` | `POST /api/v1/reports/{reportRunId}/retry` | Authenticated session, active school, lifecycle permission, retry-eligible run |
| Report cancel | `cancelReport` | `POST /api/v1/reports/{reportRunId}/cancel` | Authenticated session, active school, lifecycle permission, cancellable run |
| Report delete | `deleteReport` | `DELETE /api/v1/reports/{reportRunId}` | Authenticated session, active school, lifecycle permission |
| Report restore | `restoreReport` | `POST /api/v1/reports/{reportRunId}/restore` | Authenticated session, active school, lifecycle permission, deleted run |
| Report output download | `downloadReport` | `GET /api/v1/reports/{reportRunId}/download` | Authenticated session, active school, report permission, available generated output format |
| Definition list | `listReportDefinitions` | `GET /api/v1/report-definitions` | Authenticated session, active school, report-definition permission |
| Definition create | `createReportDefinition` | `POST /api/v1/report-definitions` | Authenticated session, active school, report-definition permission, catalog-valid definition |
| Definition detail | `getReportDefinition` | `GET /api/v1/report-definitions/{reportDefinitionId}` | Authenticated session, active school, report-definition permission |
| Definition update | `updateReportDefinition` | `PATCH /api/v1/report-definitions/{reportDefinitionId}` | Authenticated session, active school, report-definition permission, edit-eligible definition |
| Definition delete | `deleteReportDefinition` | `DELETE /api/v1/report-definitions/{reportDefinitionId}` | Authenticated session, active school, report-definition permission |
| Definition activate | `activateReportDefinition` | `POST /api/v1/report-definitions/{reportDefinitionId}/activate` | Authenticated session, active school, report-definition permission, valid draft or inactive definition |
| Definition deactivate | `deactivateReportDefinition` | `POST /api/v1/report-definitions/{reportDefinitionId}/deactivate` | Authenticated session, active school, report-definition permission, active definition |
| Definition restore | `restoreReportDefinition` | `POST /api/v1/report-definitions/{reportDefinitionId}/restore` | Authenticated session, active school, report-definition permission, deleted definition |

## Route Surfaces

| Route intent | Behavior |
|--------------|----------|
| Reporting workspace root | Render Report History by default after session, active school, and report permission gates are confirmed |
| Report catalog | Load approved catalog and complexity limits for request and definition forms |
| Report request | Submit built-in or active custom-definition report requests from catalog-approved inputs |
| Report history | Load paginated report runs with documented filters and auto-refresh requested/generating visible runs |
| Report run detail | Show status, source, retry lineage, timestamps, output availability, retention, and approved lifecycle controls |
| Report download | Download only available generated output formats through approved binary operation |
| Custom definitions | List, create, view, edit, activate, deactivate, delete, and restore definitions through approved contracts |
| Direct target route | Show safe not-found, denied, conflict, or tenant-mismatch state without exposing cross-tenant report/definition details |

## Service Boundary

All HTTP access must go through reporting service modules. Components and route
views must not call Axios directly.

Service functions:

- `getReportCatalog({ schoolId })`
- `listReports({ schoolId, page, perPage, reportType, generationStatus, reportSource, includeDeleted })`
- `requestReport({ schoolId, reportType, reportDefinitionId, filters, outputFormats })`
- `retryReport({ schoolId, reportRunId, reasonCode })`
- `cancelReport({ schoolId, reportRunId, reasonCode })`
- `deleteReport({ schoolId, reportRunId })`
- `restoreReport({ schoolId, reportRunId })`
- `downloadReport({ schoolId, reportRunId, format })`
- `listReportDefinitions({ schoolId, page, perPage, lifecycleState, includeDeleted })`
- `getReportDefinition({ schoolId, reportDefinitionId })`
- `createReportDefinition({ schoolId, definition })`
- `updateReportDefinition({ schoolId, reportDefinitionId, definition })`
- `deleteReportDefinition({ schoolId, reportDefinitionId })`
- `activateReportDefinition({ schoolId, reportDefinitionId })`
- `deactivateReportDefinition({ schoolId, reportDefinitionId })`
- `restoreReportDefinition({ schoolId, reportDefinitionId })`

Mapping requirements:

- Submit only documented parameters and request fields.
- Parse paginated envelopes through shared pagination mappers.
- Parse success/accepted envelopes through reporting contract mappers.
- Handle binary downloads without persisting report contents.
- Normalize error envelopes into safe feedback states.
- Drop undocumented response fields.
- Preserve per-format output availability.
- Render timestamps in active school timezone.

## UI State Contract

Reporting surfaces must distinguish:

- loading
- empty
- unauthorized
- forbidden
- tenant-mismatch
- inactive-school
- no-active-school
- unavailable-catalog
- validation
- not-found
- conflict
- output-expired
- unsupported page-size
- temporary-unavailable
- stale-response
- download-unavailable

True empty states must not be shown for missing school, denial, validation,
target not-found, catalog unavailable, conflict, expired output, unavailable
download, or stale response.

## Capability Gates

- No reporting data request before active school is confirmed.
- No report catalog, history, request, or download without report permission.
- No retry, cancel, delete, or restore controls without lifecycle permission.
- No custom definition screens without report-definition permission.
- No report request before catalog-valid filters and output formats are
  present.
- No custom report request unless the definition is active.
- No structural custom definition edit while definition is active.
- No output download unless the selected format availability is `available`.
- No automatic refresh outside visible requested/generating report runs.
- No lifecycle free-text reason input.
- No platform-wide reporting, manual status mutation, output delete/restore,
  retention override, permanent purge, legal hold, anonymization, arbitrary
  query text, custom code, uploaded templates, unapproved domains, report
  sharing, messaging, notifications, billing, or undocumented controls.

## Report Catalog Contract

Allowed:

- Display approved domains and labels.
- Display approved fields and visibility metadata.
- Display approved filter IDs, operators, and reference types.
- Display approved grouping, sorting, and output formats.
- Display complexity limits: 25 fields, 10 filters, 2 grouping levels, 3 sort
  fields.

Blocked:

- Hardcoded fields not returned by catalog.
- Unsupported operators, joins, domains, references, or formats.
- Arbitrary query text, custom code, uploaded templates, credentials, hidden
  fields, private paths, report output references, and non-catalog entries.

## Report History and Refresh Contract

Allowed:

- Page and per-page.
- Report type filter.
- Generation status filter.
- Report source filter.
- Include-deleted filter.
- Manual refresh.
- Automatic refresh while visible runs remain `requested` or `generating`.

Refresh rules:

- Automatic refresh must stop or ignore responses after route, active school,
  authentication, filters, selected report, selected definition, catalog, or
  dialog state changes.
- Meaningful status, output availability, lifecycle, conflict, and expired
  changes must render visible updates and polite assistive-technology
  announcements.
- Routine no-change refresh results must not announce.

## Output Download Contract

Allowed formats:

- `pdf`
- `csv`
- `xlsx` where catalog and report output state approve it

Availability states:

- `pending`
- `available`
- `failed`
- `expired`
- `unsupported`

Download rules:

- Enable download only for `available` output formats.
- Expired output response maps to output-expired state.
- Expired output messaging directs users to new report request or approved
  retry; it must not promise download-time regeneration.
- Binary download handling must not log or persist report contents.

## Lifecycle Action Contract

Retry:

- Allowed only where approved state and permission allow failed or expired
  generated runs.
- Uses approved reason codes only.
- Accepted response creates or links a new asynchronous report run while the
  original remains visible according to history filters.

Cancel:

- Allowed only for requested or generating runs where cancellation remains
  valid.
- Requires approved reason code.
- Canceled state must prevent any assumption that outputs later become
  downloadable.

Delete/restore:

- Soft-delete changes default list visibility only.
- Restore clears deleted visibility but does not make expired outputs
  downloadable, restart generation, mutate timestamps, or activate definitions.

All lifecycle responses:

- Returned report run state is authoritative.
- Conflict, validation, denied, not-found, and tenant-mismatch clear pending
  action state.

## Custom Definition Contract

Allowed:

- List definitions with lifecycle-state and include-deleted filters.
- Create draft definitions from catalog-approved inputs.
- Edit draft and inactive definitions structurally.
- Edit active definitions by metadata only.
- Activate valid draft or inactive definitions.
- Deactivate active definitions.
- Delete definitions.
- Restore deleted definitions to inactive.
- Request reports from active definitions only.

Blocked:

- Structural edits while active.
- Requests from draft, inactive, deleted, invalid, inaccessible, or stale
  definitions.
- Automatic activation after restore.
- Duplicate non-deleted names.
- More than 25 fields, 10 filters, 2 grouping levels, or 3 sort fields.
- Arbitrary query text, custom code, uploaded templates, unsupported domains,
  hidden fields, report output references, credentials, and private paths.

## Sensitive Data Contract

UI state, diagnostics, visible errors, filenames, and test output must not
include:

- private filter payloads
- generated report contents
- report output storage paths
- storage keys
- cross-school identifiers
- hidden fields
- credentials
- role internals
- token values
- raw backend denial reasons
- unauthorized report or definition existence

Allowed diagnostics:

- operation ID
- generic state kind
- field label for validation
- output format
- lifecycle action kind
- current route name
- safe correlation/request ID if provided by shared error normalization

## Verification Contract

Implementation must include Vitest coverage for:

- approved operation mapping
- no undocumented parameter submission
- no data requests before active school gate
- report permission gates
- lifecycle permission gates
- report-definition permission gates
- catalog-driven request fields and filters
- built-in report request
- active custom-definition report request
- report history pagination and filters
- empty history and no-filter-results states
- automatic refresh and manual refresh
- stale-response protection
- active school timezone formatting
- polite announcements for meaningful state changes
- output availability states
- available output download
- expired-output response mapping
- retry/cancel/delete/restore eligibility and conflicts
- custom definition create/update/activate/deactivate/delete/restore behavior
- active metadata-only edit boundary
- complexity-limit validation
- unauthorized, forbidden, tenant-mismatch, inactive-school, validation,
  not-found, conflict, unsupported page-size, stale-response, and
  temporary-unavailable mapping
- safe diagnostics redaction
