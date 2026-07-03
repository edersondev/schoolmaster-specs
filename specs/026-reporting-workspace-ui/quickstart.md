# Quickstart: Reporting Workspace UI

## Prerequisites

- Feature 015 Frontend Architecture Baseline is implemented.
- Feature 016 System Administrator Shell and Dashboard Foundation is
  implemented where shared protected shell patterns are required.
- Feature 017 Authentication and Session Foundation UI is implemented,
  including current-user hydration, active school context, unauthorized,
  forbidden, inactive-user, inactive-school, no-active-school, session-expired,
  and tenant-mismatch behavior.
- Feature 024 Student Self-Service UI and Feature 025 Guardian Self-Service UI
  are complete and available as references for protected self-service state
  handling, stale-response protection, and no-sensitive-data diagnostics.
- Backend reporting foundation from `specs/005-backend-student-reporting/` and
  report lifecycle expansion from `specs/012-report-lifecycle-expansion/` are
  deployed and contract-compliant for approved reporting operations.
- Implementation confirms how authenticated session or approved access
  behavior exposes active school context, active school timezone, report
  permission, report lifecycle permission, and report-definition permission.

## Contract Review

Before frontend implementation:

1. Confirm `api/openapi.yaml` includes `getReportCatalog`.
2. Confirm `ReportCatalog` exposes approved domains, fields, filters,
   operators, grouping, sorting, output formats, and complexity limits.
3. Confirm catalog access is approved for report request workflows with report
   permission and for custom definition workflows with report-definition
   permission.
4. Confirm `api/openapi.yaml` includes `listReports` with page, per-page,
   report type, generation status, report source, and include-deleted filters.
5. Confirm `ReportRun` includes generation status, soft-delete metadata,
   retry lineage, generated timestamp, expiry timestamp, output availability,
   and per-format outputs.
6. Confirm no standalone `GET /api/v1/reports/{reportRunId}` operation is
   required by this UI slice; report-run detail is hydrated only from approved
   list, request, retry, cancel, delete, or restore response state.
7. Confirm `api/openapi.yaml` includes `requestReport` for built-in and custom
   report requests.
8. Confirm `ReportRequest` supports documented report type or definition ID,
   filters, and output formats only.
9. Confirm `api/openapi.yaml` includes `downloadReport` and the documented
   expired-output response.
10. Confirm `ReportFormat` includes PDF, CSV, and catalog-approved XLSX.
11. Confirm `ReportOutputAvailability` includes pending, available, failed,
   expired, and unsupported.
12. Confirm `api/openapi.yaml` includes retry, cancel, delete, and restore
    operations for report runs.
13. Confirm retry and cancellation reason codes are predefined and tenant-safe.
14. Confirm `api/openapi.yaml` includes report definition list, create,
    detail, update, delete, activate, deactivate, and restore operations.
15. Confirm `ReportDefinition` lifecycle states are draft, active, inactive,
    and deleted.
16. Confirm active definitions allow metadata-only edits and restored
    definitions return to inactive.
17. Confirm validation, unauthorized, forbidden where applicable,
    tenant-mismatch, inactive-school, not-found, conflict, output-expired,
    unsupported page-size, and temporary-unavailable envelopes are documented
    or normalized by existing frontend error mapping.
18. Confirm no platform-wide reporting, manual status mutation, output delete
    or restore, retention override, report sharing, notifications, messaging,
    billing, arbitrary query text, custom code, uploaded templates, or
    unapproved domains are approved for this slice.

## Component Boundary Review

- Route views remain composition surfaces.
- Reporting services own all HTTP access, binary download handling, and
  contract mapping.
- Reporting composables coordinate school context, permission gates, catalog,
  report forms, report history filters, list-backed report detail,
  auto-refresh, manual refresh, selected report, selected definition,
  downloads, lifecycle dialogs, custom definition editing, stale-response
  protection, announcements, and safe feedback.
- Components receive mapped data through props and emit user intent.
- Element Plus component tags remain PascalCase.
- Display text is centralized through Vue I18n.
- Tailwind handles layout and spacing around Element Plus primitives.

## Manual Scenario Review

### Reporting Context Gates

- Sign in as an authenticated reporting user with one active school and report
  permission.
- Open reporting workspace root.
- Verify Report History appears by default after session and school context
  resolve.
- Sign in with no active school and verify no-active-school state blocks
  reporting data requests.
- Sign in without report permission and verify report catalog, request,
  history, and download requests remain blocked or render denied state.
- Sign in with report permission but without lifecycle permission and verify
  retry, cancel, delete, and restore controls are absent or blocked.
- Sign in with report permission but without report-definition permission and
  verify custom definition screens are absent or blocked.
- Sign in with report-definition permission but without report permission and
  verify custom definition catalog access works while report request, history,
  and download surfaces remain absent or blocked.
- Verify platform/support roles do not bypass same-school reporting permission.

### Report Catalog and Request

- Open report catalog.
- Verify approved domains, fields, filters, operators, grouping, sorting,
  formats, and complexity limits render.
- Verify unsupported catalog entries do not appear.
- Submit a built-in report request with valid filters and formats.
- Verify accepted asynchronous report run appears in history as requested or
  generating.
- Submit an active custom-definition report request.
- Verify draft, inactive, deleted, invalid, inaccessible, or stale definitions
  cannot be requested.
- Verify invalid filters, references, output formats, and empty output format
  selections show safe validation feedback.

### Report History and State Rendering

- Open report history with requested, generating, generated, failed, canceled,
  expired-output, soft-deleted, retried, built-in, custom, and empty filtered
  states.
- Verify documented filters and pagination work.
- Verify soft-deleted runs are omitted by default and appear only when
  include-deleted is explicitly selected.
- Verify requested and generating visible runs auto-refresh and manual refresh
  is available.
- Verify generated time, expiry time, lifecycle timestamps, and custom
  definition timestamps render in active school timezone.
- Verify direct or bookmarked report-run detail identifiers that are not
  present in approved list/request/lifecycle response state show safe
  unavailable or not-found feedback without a standalone detail API call.
- Verify meaningful state changes show visible updates and polite
  assistive-technology announcements without moving keyboard focus.
- Verify stale refresh responses do not overwrite current visible state.

### Report Output Download

- Open a generated report with PDF, CSV, and approved XLSX output states.
- Verify download is enabled only for available formats.
- Download one available format.
- Verify pending, failed, expired, unsupported, canceled, deleted,
  unauthorized, and cross-tenant outputs do not download.
- Verify expired-output response explains a new report request or approved
  retry is required and does not promise automatic regeneration.
- Verify storage paths, storage keys, generated report contents, and
  cross-tenant file existence do not appear in UI or diagnostics.

### Lifecycle Actions

- Open failed and expired generated runs that are retry eligible.
- Retry using approved reason code and verify a new asynchronous run appears
  while original history remains.
- Open requested or generating runs that are cancellable.
- Cancel using approved reason code and verify canceled state appears.
- Delete eligible report runs and verify they leave default lists but remain
  available through include-deleted where approved.
- Restore deleted report runs and verify generation status, output
  availability, and retention messaging remain consistent.
- Trigger or mock conflicts for stale lifecycle actions and verify pending
  action state clears without contradicting returned run state.

### Custom Report Definitions

- Open custom definitions list.
- Verify pagination, lifecycle-state filter, include-deleted filter, empty
  state, and denied state.
- Create a draft definition from approved catalog options.
- Verify complexity counters for 25 fields, 10 filters, 2 grouping levels, and
  3 sort fields.
- Activate a valid draft or inactive definition.
- Request a report from an active definition.
- Edit active definition metadata and verify structural controls are blocked.
- Deactivate definition and verify structural edits become available.
- Delete and restore definition and verify restored state is inactive.
- Verify duplicate-name, unsupported-field, unsupported-format,
  complexity-limit, active-edit-boundary, restore-conflict, validation,
  conflict, denied, not-found, and tenant-mismatch feedback.

### Stale Response and Diagnostics

- Start catalog, request, history, detail, download, lifecycle, or definition
  requests, then change route, active school, authentication, filters,
  selected report, selected definition, catalog, or dialog state before
  response applies.
- Verify stale response does not overwrite current visible state.
- Verify visible errors, diagnostics, filenames, and test output omit private
  filter payloads, report contents, storage paths, storage keys,
  cross-school identifiers, hidden fields, credentials, role internals, token
  values, raw denial reasons, and unauthorized report or definition existence.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm run test:unit
```

Focused Vitest coverage should include:

- reporting service mappers for `getReportCatalog`
- reporting service mappers for `listReports`
- reporting service mappers for `requestReport`
- reporting service mappers for `downloadReport`
- reporting service mappers for `retryReport`, `cancelReport`, `deleteReport`,
  and `restoreReport`
- reporting service mappers for report definition operations
- no undocumented request parameters or fields
- active school gate before reporting data requests
- report, lifecycle, and definition permission gates
- definition-only catalog permission for custom definition workflows
- Report History as the workspace root default
- catalog-driven request fields and filters
- built-in and active custom-definition report requests
- report history pagination, filters, empty states, and include-deleted state
- list-backed report-run detail hydration without undocumented detail lookup
- automatic refresh and manual refresh
- stale-response protection
- active school timezone formatting
- polite announcements for meaningful state changes
- output availability states and available download
- expired-output response mapping
- retry, cancel, delete, restore eligibility and conflicts
- custom definition create, update, activate, deactivate, delete, and restore
- active metadata-only edit boundary
- complexity-limit validation
- unauthorized, forbidden, tenant-mismatch, inactive-school, validation,
  not-found, conflict, unsupported page-size, stale-response, and
  temporary-unavailable mapping
- safe diagnostics redaction

Run build checks if available:

```bash
npm run build
```

Run OpenAPI validation only if contracts change:

```bash
npx @redocly/cli lint api/openapi.yaml
```

## Acceptance Evidence

Record in implementation PR:

- Operation ID to UI surface mapping.
- Evidence that reporting workspace gates on active school and same-school
  reporting permissions.
- Evidence that reporting workspace root opens Report History by default.
- Evidence that report catalog drives request and custom definition controls.
- Evidence that no unsupported catalog entries, output formats, lifecycle
  reasons, or undocumented fields appear.
- Evidence that requested and generating runs auto-refresh and manual refresh
  works.
- Evidence that meaningful state changes have visible updates and polite
  announcements without disrupting keyboard focus.
- Evidence that timestamps render in active school timezone.
- Evidence that downloads are available only for available generated outputs.
- Evidence that expired-output behavior does not imply automatic regeneration.
- Evidence that lifecycle controls are permission-gated and state-gated.
- Evidence that custom definitions follow draft, active, inactive, deleted,
  active metadata-only edit, restore-to-inactive, and complexity-limit rules.
- Evidence that timed usability checks meet the SC-002 built-in request,
  SC-003 download, SC-005 custom definition timing targets, and SC-004 report
  state identification accuracy target.
- Evidence that frontend performance checks meet the plan targets for mocked
  rendering, protected route transitions, report request feedback, download
  action start, auto-refresh, and stale-response handling, or record an
  approved deviation.
- Evidence that no private filter payloads, report contents, storage paths,
  storage keys, cross-school identifiers, hidden fields, credentials, role
  internals, token values, raw denial reasons, or unauthorized report
  existence appear in diagnostics or test output.

## Implementation Evidence

- 2026-07-03: Reporting workspace implemented in `src/pages/reporting/`,
  `src/components/reporting/`, `src/composables/reporting/`,
  `src/services/reporting/`, `src/contracts/reporting/`,
  `src/router/modules/reporting.js`, and `src/i18n/modules/reporting.js`.
- 2026-07-03: Operation IDs verified in `specs/api/openapi.yaml` and referenced
  path files for catalog, reports, report lifecycle, downloads, and report
  definitions. No OpenAPI file changes were required.
- 2026-07-03: Reporting root redirects to Report History by default.
- 2026-07-03: Services isolate all HTTP access; reporting pages, components,
  and composables contain no direct Axios calls.
- 2026-07-03: Catalog, request, history, list-backed detail, auto-refresh,
  manual refresh, active-school timezone formatting, downloads, lifecycle
  actions, custom definitions, stale-response guards, and safe diagnostics
  are covered by focused Vitest tests.
- 2026-07-03: `npm run test:unit -- tests/reporting-workspace` passed:
  23 test files, 29 tests.
- 2026-07-03: `npm run build` passed. Vite/Rolldown reported existing
  third-party pure-annotation and chunk-size warnings only.
- 2026-07-03: OpenAPI lint was not run because `specs/api/openapi.yaml` was not
  changed by this frontend implementation.
- 2026-07-03: Code audit found unsupported reporting actions absent from
  controls; blocked capabilities are represented only as contract constants.
- 2026-07-03: Code audit found Element Plus components use PascalCase tags.
- 2026-07-03: Automated tests verify safe redaction for reporting diagnostics
  and fixtures avoid private report contents, storage paths, token values, raw
  denial reasons, and unauthorized report existence.
- 2026-07-03: Playwright timed usability and performance verification passed
  with mocked approved reporting APIs:
  `env CI=1 npm run test:e2e -- reporting-workspace --project=chromium --retries=0`.
  The Chromium run passed in 3.3s and covered reporting root-to-history
  default routing, report state identification, available/expired output
  states, available output download feedback, built-in report request timing,
  active custom-definition request handoff, and mocked render/route/download/
  request/definition timing thresholds.

## Out of Scope Verification

Confirm none of these appear in the reporting workspace UI slice:

- platform-wide reporting
- manual free-form status mutation
- output delete or restore controls
- retention override
- permanent purge
- legal hold
- anonymization
- arbitrary query text
- custom code
- uploaded report templates
- unapproved report domains
- unapproved output formats
- report sharing
- messaging
- notification-center behavior
- billing, payment, payroll, or accounting
- student self-service report access
- guardian self-service report access
- teacher workflow correction behavior
- school administration writes outside approved report definitions
- platform support overrides
- undocumented APIs
