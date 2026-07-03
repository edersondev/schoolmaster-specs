# Feature Specification: Reporting Workspace UI

**Feature Branch**: `026-reporting-workspace-ui`  
**Created**: 2026-07-03  
**Status**: Ready for Planning  
**Input**: User description: "Specify the Reporting Workspace UI for authorized reporting users. Define report catalog browsing, report request flows, report run history, retry or cancellation where approved, custom report definitions, and download surfaces using only approved reporting contracts. Include report state rendering, retention visibility, custom-definition ownership, output availability messaging, authorization behavior, empty states, unavailable reports, failed or expired runs, tenant-safe access, and blocked unsupported reporting actions. This feature follows the completed Guardian Self-Service UI and must establish reporting UX, routing, permissions, async-operation states, and contract expectations before frontend implementation."

## Clarifications

### Session 2026-07-03

- Q: How should the UI keep asynchronous report states current while report runs are still pending? → A: Auto-refresh visible report history/detail while runs are `requested` or `generating`, with manual refresh.
- Q: Which timezone should the UI use for report timestamps and output expiry? → A: Active school timezone.
- Q: How should the UI announce meaningful report state changes during automatic refresh? → A: Visible status changes plus polite screen-reader announcements for meaningful state changes.
- Q: Which surface should the reporting workspace root open by default? → A: Report History.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Browse Report Catalog and Request Reports (Priority: P1)

An authorized school reporting user opens the reporting workspace, reviews approved built-in and custom-report catalog options for the active school, chooses valid filters and output formats, and submits a report request that becomes an asynchronous report run.

**Why this priority**: Report request is the core workspace outcome. The UI must guide users toward approved report domains, filters, formats, and school-scoped references before any history, download, or lifecycle action has value.

**Independent Test**: Can be fully tested by signing in as a reporting user with active same-school report permission, opening the reporting workspace, loading the catalog, submitting one built-in report and one active custom-definition report with valid filters and formats, and confirming each accepted request appears as a requested report run without exposing unsupported catalog entries.

**Acceptance Scenarios**:

1. **Given** an authenticated reporting user has an active permitted school context and school-scoped report permission, **When** they open the reporting workspace, **Then** the UI loads the approved report catalog and shows only documented report domains, fields, filters, grouping, sorting, output formats, and complexity limits for that school.
2. **Given** a reporting user selects a built-in report or active custom report definition, **When** they configure valid same-school filters and supported output formats, **Then** the UI submits the request through the approved report request contract and shows the accepted asynchronous run state.
3. **Given** the catalog excludes a field, filter, operator, grouping option, sort option, format, or report domain, **When** the user configures a report, **Then** the UI does not offer the unsupported option and blocks direct form submission that would rely on it.
4. **Given** no report catalog is available because access is denied, school context is missing, or the approved operation is unavailable, **When** the user opens request surfaces, **Then** the UI shows a safe unavailable or denied state and does not send report request payloads.

---

### User Story 2 - Review Report Run History and States (Priority: P2)

An authorized reporting user reviews school-scoped report run history, understands generation status, deletion visibility, retry lineage, output availability, and retention timing, and can distinguish empty history from denied, filtered, failed, canceled, expired, or unavailable states.

**Why this priority**: Reporting is asynchronous. Users need trustworthy run history and state rendering to know whether a report is still being generated, failed, canceled, available for download, expired, deleted, or superseded by a retry.

**Independent Test**: Can be fully tested by opening report history with report runs in requested, generating, generated, failed, canceled, expired-output, soft-deleted, built-in, custom, retried, and empty filtered states, then verifying visible state, filters, retention messages, pagination, and tenant-safe denial behavior.

**Acceptance Scenarios**:

1. **Given** report runs exist in the active school, **When** the user opens report history, **Then** the UI lists only same-school report runs using approved pagination, report type, generation status, report source, and include-deleted filters.
2. **Given** an authenticated reporting user opens the reporting workspace root with an active permitted school context and report permission, **When** the workspace resolves its default route, **Then** the UI opens Report History by default.
3. **Given** a report run is requested, generating, generated, failed, canceled, soft-deleted, superseded, or has expired outputs, **When** it appears in history, **Then** the UI renders the documented generation status, soft-delete visibility, retry lineage, output availability, generated time, and output-expiry information without inventing hidden states.
4. **Given** visible report history or report detail contains report runs still in `requested` or `generating` state, **When** the screen remains open, **Then** the UI automatically refreshes visible status from approved contracts, shows visible state changes, politely announces meaningful state changes to assistive technology, and also provides manual refresh without applying stale responses.
5. **Given** report history or detail shows generated time, output expiry time, cancellation time, deletion time, restore time, or definition timestamps, **When** the UI renders those values, **Then** it uses the active school timezone consistently.
6. **Given** no report runs match the current filters, **When** history loads successfully, **Then** the UI shows a true empty history or no-filter-results state distinct from denied, validation, tenant-mismatch, not-found, unavailable, and stale-response states.
7. **Given** a direct route, stale selection, or bookmarked report identifier targets a missing, deleted-without-include, unauthorized, or cross-tenant run, **When** the request resolves, **Then** the UI shows the approved not-found, denied, or tenant-mismatch state without revealing protected report details.

---

### User Story 3 - Download Available Report Outputs (Priority: P3)

An authorized reporting user downloads only generated, unexpired, same-school report outputs in supported formats and sees clear messaging when outputs are pending, unavailable, failed, unsupported, expired, canceled, or not authorized.

**Why this priority**: Download is the high-value endpoint of reporting. The UI must make output availability and 90-day retention visible while preventing unsupported regeneration or unsafe file access assumptions.

**Independent Test**: Can be fully tested by opening generated report runs with PDF, CSV, and approved XLSX outputs in available, pending, failed, expired, and unsupported states, downloading one available format, and verifying blocked messaging for every unavailable format.

**Acceptance Scenarios**:

1. **Given** a same-school report output has availability `available` and is inside the 90-day retention window, **When** the user chooses a supported format, **Then** the UI downloads only that authorized generated file through the approved download contract.
2. **Given** a report output is pending, failed, expired, unsupported, canceled, missing, deleted from default visibility, or belongs to a different school, **When** the user attempts to download it, **Then** the UI blocks the action or renders the documented response without exposing storage paths, private file identifiers, or cross-tenant existence.
3. **Given** a generated output has an expiry timestamp, **When** it appears in history or detail, **Then** the UI shows retention visibility and expired-output messaging that explains a new report run or approved retry is required rather than promising automatic regeneration.
4. **Given** a download response indicates expired output, conflict, validation failure, not found, unauthorized, or tenant mismatch, **When** the UI handles the response, **Then** it clears any pending download state and shows safe feedback tied to the documented response category.

---

### User Story 4 - Manage Approved Report Lifecycle Actions (Priority: P4)

An authorized reporting user performs only approved lifecycle actions for retry, cancellation, soft-delete, and restore where the current run state and permissions allow them, while unsupported or conflicting actions remain hidden or blocked.

**Why this priority**: Lifecycle controls reduce operational friction, but they can change report history and audit meaning. The UI must reflect backend-approved eligibility rather than encouraging invalid transitions.

**Independent Test**: Can be fully tested by opening retryable, cancellable, generated, failed, canceled, deleted, superseded, and cross-tenant report runs, verifying visible actions, submitting allowed retry and cancel actions with approved reason codes, and confirming unsupported transitions show safe validation or conflict states.

**Acceptance Scenarios**:

1. **Given** a same-school report run is failed or generated with expired output and is eligible for retry, **When** the user selects retry with an approved reason code where required, **Then** the UI creates a new asynchronous report run and preserves the original run in history.
2. **Given** a same-school report run is requested or generating and cancellation is still allowed, **When** the user cancels it with an approved reason code, **Then** the UI renders the canceled state and prevents any assumption that outputs will later become downloadable.
3. **Given** a same-school report run is eligible for soft-delete or restore, **When** the user performs the lifecycle action, **Then** the UI updates default list visibility while preserving history semantics, generation state, output availability, and retention messaging.
4. **Given** a lifecycle action is not approved for the current state, actor, school, deleted status, output availability, or retry lineage, **When** the user views or attempts that action, **Then** the UI hides or blocks it and renders documented validation, conflict, denied, not-found, or tenant-mismatch feedback.

---

### User Story 5 - Maintain Custom Report Definitions (Priority: P5)

An authorized school reporting user creates and maintains school-owned custom report definitions from the approved catalog, manages draft, active, inactive, and deleted states, and requests reports only from active definitions.

**Why this priority**: Custom definitions power repeated school-specific reporting. The UI must keep definition ownership, catalog limits, activation rules, active metadata-only edits, restore behavior, and unsupported custom-report actions explicit before implementation.

**Independent Test**: Can be fully tested by listing definitions, creating a draft from approved catalog entries, activating it, requesting a report from it, editing allowed metadata while active, deactivating for structural edits, deleting and restoring it to inactive, and verifying duplicate-name and unsupported-field errors.

**Acceptance Scenarios**:

1. **Given** the user has school-scoped report-definition permission, **When** they open custom definitions, **Then** the UI lists only same-school definitions with approved lifecycle-state, include-deleted, and pagination behavior.
2. **Given** the user creates or updates a definition, **When** they select fields, filters, grouping, sorting, and formats, **Then** the UI enforces catalog-approved options and visible complexity limits before submitting.
3. **Given** a custom definition is active, **When** the user edits it, **Then** the UI allows only approved metadata edits and blocks structural changes until the definition is deactivated.
4. **Given** a custom definition is deleted and restored, **When** restore succeeds, **Then** the UI shows it as inactive and does not allow report requests until activation succeeds.
5. **Given** a definition is inactive, draft, deleted, invalid, duplicate-named, cross-tenant, or unsupported by the catalog, **When** a report request or lifecycle action is attempted, **Then** the UI shows safe validation, conflict, not-found, denied, or tenant-mismatch feedback.

### Edge Cases

- Active school context is missing, inactive, changed during a request, or no longer authorized while reporting screens are open.
- Authenticated user lacks school-scoped report permission, report lifecycle permission, or report-definition permission for the active school.
- Platform administrators and support users have platform roles but no explicit school-scoped reporting permission.
- Report catalog is empty, temporarily unavailable, unavailable because the approved backend contract is not implemented, or returns only domains that do not support the user's intended format or filters.
- Report history has no runs, no filtered results, only soft-deleted runs, only expired outputs, or only runs in requested, generating, failed, canceled, superseded, or unsupported-output states.
- Reporting workspace root is opened before report catalog, report history, or permissions finish loading.
- Visible report history or detail contains requested or generating runs that later become generated, failed, canceled, superseded, soft-deleted, unavailable, or denied during automatic refresh.
- Automatic refresh detects a meaningful report state change while keyboard focus remains on a report action, filter control, dialog, download control, or custom definition form.
- Direct routes target missing, unauthorized, deleted, restored, stale, superseded, already-canceled, already-retried, already-generated, expired-output, or cross-tenant report runs.
- Direct routes target missing, unauthorized, inactive, deleted, restored, duplicate-name, structurally invalid, or cross-tenant custom report definitions.
- Report responses arrive after the user changes route, filters, active school, authentication state, selected report, selected definition, or lifecycle dialog state.
- Download attempts return expired-output, validation, conflict, not-found, unauthorized, tenant-mismatch, temporary-unavailable, or binary response behavior.
- Lifecycle actions race with generation completion, retry, cancellation, delete, restore, output expiry, or another user action.
- Active school timezone is missing or changes while timestamped report history, report detail, output expiry, lifecycle action, or custom definition screens are open.
- Custom definition catalog entries, complexity limits, active edit boundaries, lifecycle state, output formats, or runtime filter requirements change while the user is editing.
- UI receives validation, unauthorized, forbidden, tenant-mismatch, no-active-school, inactive-school, not-found, conflict, output-expired, unsupported pagination, stale-response, and temporary-unavailable behavior.
- Unsupported reporting actions are attempted by route or stale controls, including platform-wide reporting, manual free-form status mutation, output delete or restore, retention override, permanent purge, legal hold, anonymization, arbitrary query text, custom code, uploaded templates, unapproved domains, report sharing, messaging, notifications, billing, or undocumented APIs.
- Visible errors, diagnostics, downloaded filenames, and automated test output must not expose private filter payloads, report contents, storage paths, storage keys, cross-school identifiers, hidden fields, credentials, role internals, token values, or unauthorized report existence.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is included in this frontend feature. Reporting catalog, report request, report history, download, lifecycle, and custom definition behavior must already be approved, implemented, and contract-compliant before frontend runtime exposure.
- **Frontend repository impact**: Adds protected reporting workspace routes, navigation, catalog browsing, report request forms, report history, report run detail, output download surfaces, lifecycle action controls, custom definition list/detail/edit flows, and safe status rendering for authorized reporting users.
- **Specification or contract repository impact**: This specification defines the frontend consumption boundary for approved reporting contracts. OpenAPI changes are not required for the listed reporting routes, but any unsupported reporting action, new state, new output format, new domain, new field, new filter, new lifecycle reason, platform-wide reporting, or undocumented route requires a separate specification and OpenAPI update before implementation.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines this UI boundary first, `schoolmaster-frontend` implements after plan and tasks, and `schoolmaster-backend` changes are allowed only through separate backend or contract work if a missing approved operation is identified.

### API Contract Impact

- **OpenAPI update required**: No for the currently approved reporting workspace routes in scope. Yes before exposing platform-wide reporting, manual status mutation, output delete or restore, retention override, permanent purge, legal hold, anonymization, report sharing, unapproved formats, unapproved domains, arbitrary query behavior, uploaded report templates, notifications, messaging, billing, or undocumented lifecycle actions.
- **Versioned endpoints affected**: Frontend may consume only `/api/v1/report-catalog`, `/api/v1/reports`, `/api/v1/reports/{reportRunId}`, `/api/v1/reports/{reportRunId}/retry`, `/api/v1/reports/{reportRunId}/cancel`, `DELETE /api/v1/reports/{reportRunId}`, `/api/v1/reports/{reportRunId}/download`, `/api/v1/reports/{reportRunId}/restore`, `/api/v1/report-definitions`, `/api/v1/report-definitions/{reportDefinitionId}`, `DELETE /api/v1/report-definitions/{reportDefinitionId}`, `/api/v1/report-definitions/{reportDefinitionId}/activate`, `/api/v1/report-definitions/{reportDefinitionId}/deactivate`, and `/api/v1/report-definitions/{reportDefinitionId}/restore` for this slice, plus already approved authentication, current-user, permission, active-school, and session context behavior from the authentication foundation.
- **JSON response impact**: UI behavior depends on documented success, accepted, paginated, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, output-expired, generation-status, soft-delete, lifecycle-state, catalog, definition, and per-output-availability semantics only. No UI may depend on undocumented fields, status codes, denial reasons, storage paths, backend internals, worker details, or hidden actor metadata.
- **Authentication/authorization impact**: All reporting workspace screens require authenticated access and active permitted school context. Report browsing and downloads require school-scoped report permission. Lifecycle controls require explicit report lifecycle permission. Custom definition screens require explicit report-definition permission. Navigation and controls are visibility aids only; backend authorization remains authoritative.
- **Compatibility impact**: Frontend delivery is additive. It must not change existing authentication, administration, teacher workflow, student self-service, guardian self-service, platform support, backend reporting, or report contract behavior unless a separate approved specification changes those areas.

### Data & Tenancy Impact

- **Tenant scoping impact**: Report catalog visibility, report runs, report outputs, custom report definitions, definition snapshots, runtime filters, selected formats, lifecycle reason codes, and visible reporting history are school-owned and scoped to the active permitted school.
- **Cross-tenant or platform access impact**: Cross-school reports, report outputs, definitions, catalog entries, academic periods, users, student profiles, guardian data, private filter details, hidden fields, and audit-sensitive lifecycle details remain inaccessible. Platform roles do not grant reporting workspace access without explicit same-school reporting permission.
- **Soft delete impact**: Report runs and custom report definitions may be soft-deleted and restored only through approved contracts. UI list visibility must respect default non-deleted views and explicit include-deleted filters. Restore does not make expired outputs downloadable, restart generation, mutate timestamps, or activate restored custom definitions.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The UI MUST expose only approved reporting workspace behavior documented before implementation begins.
- **FR-002**: The UI MUST require an authenticated user and active permitted school context before loading reporting data or enabling protected reporting actions.
- **FR-003**: The UI MUST keep report browsing, lifecycle actions, downloads, and custom definition management permission-gated by the approved school-scoped permissions for the active school.
- **FR-004**: The UI MUST retrieve report catalog options only through the approved catalog operation and MUST show only documented domains, fields, filters, operators, grouping options, sorting options, output formats, and complexity limits.
- **FR-005**: The UI MUST block report request controls when the catalog is unavailable, denied, empty, stale, or missing options required by the selected report.
- **FR-006**: The UI MUST support built-in report requests and active custom-definition report requests using only documented report types, definition identifiers, filters, and output formats.
- **FR-007**: The UI MUST validate visible report filters, reference selections, date ranges, output formats, and runtime custom-definition inputs against approved catalog and request contract behavior before submission.
- **FR-008**: The UI MUST show accepted report requests as asynchronous runs and MUST render requested, generating, generated, failed, and canceled generation statuses exactly as documented.
- **FR-009**: The UI MUST list report runs using only documented pagination, report type, generation status, report source, and include-deleted filters.
- **FR-010**: The reporting workspace root MUST open Report History by default after authenticated access, active permitted school context, and report permission are confirmed.
- **FR-011**: The UI MUST automatically refresh visible report history and report detail while visible report runs remain in `requested` or `generating` state, and MUST also provide a manual refresh action.
- **FR-012**: The UI MUST distinguish true empty report history, no filtered results, no included deleted runs, denied access, tenant mismatch, validation failure, not found, conflict, temporary unavailable, and stale response states.
- **FR-013**: The UI MUST render report-run soft-delete visibility separately from generation status and MUST not treat deletion as output deletion or generation failure.
- **FR-014**: The UI MUST render report output availability per format using only `pending`, `available`, `failed`, `expired`, and `unsupported` states.
- **FR-015**: The UI MUST show report and custom definition timestamps, generated time, expiry timing, lifecycle timestamps, and retention visibility in the active school timezone where available.
- **FR-016**: The UI MUST enable report downloads only for documented formats whose output availability is available and whose report run is authorized in the active school.
- **FR-017**: The UI MUST handle expired-output download responses by showing that output is expired and that a new report request or approved retry is required; it MUST NOT promise automatic regeneration during download.
- **FR-018**: The UI MUST NOT expose private storage paths, storage keys, worker internals, private report contents, full hidden filter payloads, or cross-tenant file existence in download, error, history, or diagnostics surfaces.
- **FR-019**: The UI MUST expose retry only where the approved contract and current state allow retry for failed runs or generated runs with expired outputs.
- **FR-020**: The UI MUST expose cancellation only for requested or generating runs where the approved contract and permissions allow cancellation.
- **FR-021**: The UI MUST use only approved lifecycle reason codes for retry and cancellation inputs and MUST not provide free-text lifecycle reasons.
- **FR-022**: The UI MUST expose report-run soft-delete and restore controls only where approved by contract, permissions, active school context, and current run state.
- **FR-023**: The UI MUST render lifecycle action success, validation, conflict, denied, not-found, tenant-mismatch, and stale-response outcomes without partial optimistic state that contradicts the returned report run.
- **FR-024**: The UI MUST maintain report retry lineage visibility where returned by the approved contract, including source and superseding run references without exposing inaccessible reports.
- **FR-025**: The UI MUST list custom report definitions using only documented pagination, lifecycle-state, and include-deleted filters.
- **FR-026**: The UI MUST create and update custom report definitions using only approved school-owned catalog entries, domains, fields, filters, grouping, sorting, output formats, lifecycle behavior, and complexity limits.
- **FR-027**: The UI MUST enforce visible v1 complexity limits of 25 selected fields, 10 filters, 2 grouping levels, and 3 sort fields before custom-definition submission.
- **FR-028**: The UI MUST render custom definition lifecycle states `draft`, `active`, `inactive`, and `deleted` without inventing additional states.
- **FR-029**: The UI MUST allow report requests from active custom definitions only and MUST block requests from draft, inactive, deleted, invalid, inaccessible, or stale definitions.
- **FR-030**: The UI MUST allow metadata-only edits for active custom definitions and MUST block structural edits until the definition is deactivated.
- **FR-031**: The UI MUST show restored custom report definitions as inactive and MUST require a separate activation action before they can be requested.
- **FR-032**: The UI MUST show duplicate-name, unsupported-field, unsupported-format, complexity-limit, active-edit-boundary, restore-conflict, and catalog-validation failures as safe validation or conflict feedback.
- **FR-033**: The UI MUST hide or block unsupported reporting actions, including platform-wide reporting, manual free-form status mutation, output delete or restore, retention override, permanent purge, legal hold, anonymization, arbitrary query text, custom code, uploaded templates, unapproved domains, report sharing, messaging, notifications, billing, and undocumented APIs.
- **FR-034**: The UI MUST ignore or cancel stale reporting results when the user changes route, active school, authentication state, list filters, selected report run, selected definition, catalog version, or open lifecycle dialog before the response is applied.
- **FR-035**: The UI MUST show visible status changes and use polite assistive-technology announcements for meaningful report state changes caused by automatic refresh, lifecycle actions, and download availability changes.
- **FR-036**: The UI MUST reuse existing protected shell, navigation, list, detail, status, pagination, loading, empty-state, denial, unavailable, conflict, stale-response, download, and not-found behavior from completed frontend features.
- **FR-037**: The UI MUST keep reporting navigation, route names, titles, empty states, and denied states distinct from administration, teacher workflow, student self-service, guardian self-service, and platform support surfaces.
- **FR-038**: The UI MUST include test coverage or equivalent verification for successful request, catalog, history, default Report History route, automatic refresh, manual refresh, visible state changes, polite state-change announcements, download, retry, cancel, delete, restore, custom-definition, activation, deactivation, restore, denied, tenant mismatch, not-found, conflict, expired-output, empty-state, stale-response, unsupported-action, and no-sensitive-data diagnostics behavior.

### Key Entities *(include if feature involves data)*

- **ReportingWorkspace**: Protected school-scoped UI surface for authorized reporting users to browse catalog options, request reports, review history, download outputs, perform approved lifecycle actions, and maintain custom definitions.
- **ReportingAccessContext**: Authenticated user, active permitted school, and explicit school-scoped reporting permissions that determine visible report, lifecycle, download, and definition surfaces.
- **ReportCatalogView**: Read-only school-scoped presentation of approved report domains, fields, filters, operators, grouping options, sorting options, output formats, and complexity limits.
- **ReportRequestFlow**: User-facing configuration and submission path for built-in reports and active custom report definitions using documented filters and output formats.
- **ReportRunHistory**: Paginated school-scoped list and detail surface for report runs, generation status, soft-delete visibility, source, output availability, retry lineage, and retention metadata.
- **ReportOutputDownloadSurface**: UI state and action surface for downloading available generated outputs and safely rendering pending, failed, expired, unsupported, denied, conflict, and validation states.
- **ReportLifecycleActionSurface**: Permission-gated controls for approved retry, cancellation, soft-delete, and restore actions with approved reason codes and conflict handling.
- **CustomReportDefinitionWorkspace**: School-owned definition list, detail, creation, edit, activation, deactivation, delete, and restore surface governed by catalog options, lifecycle states, complexity limits, ownership, and active-edit rules.
- **ReportingSafeState**: UI state category for denied, unavailable, not-found, empty, expired, conflict, unsupported, stale, or missing-context behavior that avoids report and tenant enumeration.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of reporting catalog, request, history, download, lifecycle, and custom-definition UI actions can be traced to approved reporting contract behavior before frontend implementation begins.
- **SC-002**: In usability checks, an authorized reporting user can open the workspace, choose a built-in report from the catalog, submit valid filters and formats, and identify the accepted asynchronous run in under 3 minutes without assistance.
- **SC-003**: In usability checks, an authorized reporting user can find a generated report run, identify available and expired output formats, and download one available format in under 2 minutes without assistance.
- **SC-004**: In usability checks, an authorized reporting user can identify requested, generating, generated, failed, canceled, expired-output, soft-deleted, retried, built-in, and custom report states with at least 90% accuracy.
- **SC-005**: In usability checks, an authorized report-definition user can create a draft custom definition from approved catalog options, activate it, and request a report from it in under 5 minutes without assistance.
- **SC-006**: 100% of tested unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, validation, not-found, conflict, output-expired, unsupported-pagination, stale-response, unavailable-catalog, and temporary-unavailable responses show safe feedback without exposing protected report, output, definition, catalog, storage, or cross-school details.
- **SC-007**: 100% of tested unavailable, unsupported, pending, failed, expired, canceled, deleted, cross-tenant, and unauthorized output cases prevent unsafe downloads and show documented availability or denial messaging.
- **SC-008**: 100% of tested unsupported lifecycle and custom-definition actions are hidden or blocked before submission when the UI has enough approved state to decide, and otherwise resolve to documented validation, conflict, denied, not-found, or tenant-mismatch feedback.
- **SC-009**: 100% of tested stale responses caused by route, active school, authentication, filters, selected report run, selected definition, catalog, or lifecycle dialog changes do not overwrite the current visible screen state.
- **SC-010**: 100% of tested visible requested or generating report runs refresh to their latest documented state without page reload and without stale updates overwriting the current screen.
- **SC-011**: 100% of tested reporting workspace root visits land on Report History after authenticated access, active school, and report permission are confirmed.
- **SC-012**: 100% of tested report timestamps, output expiry values, lifecycle timestamps, and custom definition timestamps render in the active school timezone.
- **SC-013**: 100% of tested meaningful report state changes from automatic refresh, lifecycle actions, and download availability changes have visible updates and polite assistive-technology announcements without disrupting keyboard focus.
- **SC-014**: Review confirms no UI surface exposes platform-wide reporting, manual status mutation, output delete or restore, retention override, permanent purge, legal hold, anonymization, arbitrary query text, custom code, uploaded templates, unapproved domains, report sharing, messaging, notifications, billing, or undocumented reporting APIs.
- **SC-015**: Diagnostics verification confirms private filter payloads, report contents, storage paths, storage keys, cross-school identifiers, hidden fields, credentials, role internals, token values, and unauthorized report existence do not appear in visible errors, filenames, client-side diagnostics, or automated test output.

## Assumptions

- Backend reporting behavior from `005-backend-student-reporting` and `012-report-lifecycle-expansion` is the source of approved domain behavior for report requests, report history, downloads, retry, cancellation, delete, restore, catalog discovery, custom report definitions, output availability, and retention.
- Existing authentication and session UI behavior from `017-auth-session-ui` provides protected-route handling, current-user hydration, active school context, no-active-school handling, session-expired handling, unauthorized handling, forbidden handling, inactive-user handling, inactive-school handling, and tenant-mismatch handling.
- Existing frontend features provide reusable protected-shell, list, detail, status, pagination, loading, empty-state, denial, unavailable, conflict, stale-response, download, and not-found patterns that this feature must reuse rather than redefine.
- Reporting workspace users are school-scoped staff with explicit reporting permissions; guardians and students do not receive reporting workspace access from this feature.
- Report output files remain downloadable for 90 days after generation where the approved contract reports availability; output expiry requires a new report request or approved retry and is not regenerated during download.
- Active school timezone is available from approved school or session context before timestamped reporting data is displayed.
- PDF, CSV, and catalog-approved XLSX are the only output formats visible in this slice.
- Custom report definitions are constrained to approved catalog domains and complexity limits. Arbitrary query text, scripting, uploaded templates, custom code, platform-wide reporting, unrestricted data exploration, output retention controls, and undocumented APIs remain outside scope.
