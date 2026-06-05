# Feature Specification: Report Lifecycle Expansion

**Feature Branch**: `012-report-lifecycle-expansion`  
**Created**: 2026-06-05  
**Status**: Ready for Implementation  
**Input**: User description: "Run speckit-specify for backend roadmap item 7: Report Lifecycle Expansion. Define the next SchoolMaster backend implementation slice after completed guardian self-service. Expand the existing school-scoped reporting foundation beyond launch-scope asynchronous report request, list, download, PDF/CSV output, 90-day output retention, expired-output handling, and explicit regeneration through a new ReportRun.

Specify report retry, cancellation, deletion/restore, manual status mutation if allowed, custom report definitions, report designer backend behavior, additional output formats or filters, and report output lifecycle behavior. Clarify allowed report lifecycle state transitions, retry eligibility, cancellation eligibility, delete versus restore semantics, retention impact, output compatibility, custom report definition ownership, tenant visibility, audit obligations, permissions, validation rules, conflict rules, and response semantics.

Preserve contract-first API-only /api/v1 behavior. Require OpenAPI expansion before backend implementation exposes any new report behavior. Keep School as the v1 tenant root and school_id as the concrete tenant column for school-owned records. Preserve explicit platform versus school authorization separation: platform administrators and support users must not implicitly bypass school-scoped report permissions unless a separate approved specification and contract explicitly grants that access.

Do not include frontend implementation, platform-wide reporting/support access, student self-view changes, guardian self-service changes, teacher workflow corrections, roster changes, billing, messaging, notifications, permanent purge, legal hold, anonymization workflows, or undocumented APIs in this slice."

## Clarifications

### Session 2026-06-05

- Q: How should report lifecycle state be represented for generation progress, deletion, and output expiry? -> A: Separate fields: generation status (`requested`, `generating`, `generated`, `failed`, `canceled`), soft-delete marker, and per-output availability.
- Q: Which data domains may v1 custom report definitions use? -> A: Existing launch-scope domains only: attendance, grades, academic structure, and school activity.
- Q: Which report runs should appear in the default report list? -> A: Default report lists include all non-deleted report runs; deleted runs require an explicit include filter.
- Q: Which lifecycle states should custom report definitions use? -> A: Use `draft`, `active`, `inactive`, and `deleted`; created definitions start `draft`, and restore returns `inactive`.
- Q: Should report outputs have explicit delete or restore actions in this slice? -> A: No explicit output delete or restore; outputs expire after 90 days, and report-run delete only affects list visibility.
- Q: How should concurrent report lifecycle actions and stale worker completions resolve? -> A: First valid transition wins; later conflicting lifecycle actions return conflict, and stale worker completions are ignored and audited.
- Q: How should clients discover approved custom report fields, filters, operators, grouping, sorting, and format compatibility? -> A: Expose a read-only report catalog endpoint for approved launch-scope report configuration.
- Q: What complexity limits apply to v1 custom report definitions? -> A: Maximum 25 selected fields, 10 filters, 2 grouping levels, and 3 sort fields.
- Q: What minimum fields must report lifecycle audit entries include? -> A: Actor, action, outcome, target, school, correlation ID, and tenant-safe reason code.
- Q: Should report output availability include a deleted state? -> A: No; output availability excludes `deleted` because output delete/restore is out of scope.
- Q: How should lifecycle reason inputs be represented? -> A: Use predefined tenant-safe reason codes only; no free-text reason payloads.
- Q: How should custom report definition name uniqueness be enforced? -> A: Names are unique per school among non-deleted definitions; restore conflicts if the name is already in use.
- Q: How should active custom report definitions be editable? -> A: Active definitions allow metadata-only edits; structural fields require deactivation first.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage Report Run Lifecycle (Priority: P1)

A school-scoped reporting administrator can retry failed report runs, cancel report runs that have not completed, soft-delete report runs from operational lists, and restore deleted report runs for audit-preserving history without changing other schools' reports.

**Why this priority**: Existing reporting can request and download reports, but operations teams need controlled lifecycle actions when report runs fail, are requested by mistake, or should be hidden from normal operational lists.

**Independent Test**: Authenticate as a school reporting administrator, create same-school report runs in retryable, cancellable, generated, failed, and deleted states, perform each lifecycle action, and verify state transitions, response envelopes, audit entries, and cross-tenant denials.

**Acceptance Scenarios**:

1. **Given** a same-school report run has failed before producing downloadable output, **When** an authorized reporting administrator retries it, **Then** a new asynchronous report run is created from the original type, filters, and requested output formats while the original failed run remains available for history.
2. **Given** a same-school report run is still waiting or actively generating, **When** an authorized reporting administrator cancels it, **Then** the run becomes canceled, no new output is published, and any later worker completion for the canceled run is rejected as stale.
3. **Given** a same-school report run is generated, failed, canceled, or expired, **When** an authorized reporting administrator deletes it, **Then** it is removed from default operational lists but remains retained for audit and can be restored by an authorized actor.
4. **Given** same-school report runs exist in requested, generating, generated, failed, canceled, and deleted states, **When** an authorized reporting administrator lists reports without delete-specific filters, **Then** all non-deleted runs are returned and deleted runs are omitted.
5. **Given** a report run belongs to another school or the actor lacks school-scoped report lifecycle permission, **When** the actor attempts retry, cancel, delete, or restore, **Then** the request is denied without exposing cross-tenant report details.

---

### User Story 2 - Define Custom School Reports (Priority: P2)

A school reporting administrator can create and maintain custom report definitions using approved fields, filters, grouping, and output options from existing launch-scope report domains so repeated school-specific reports can be requested without granting unrestricted data access.

**Why this priority**: Custom definitions are the boundary between the fixed launch-scope reports and a future report designer experience. The backend must govern what data can be selected before frontend surfaces consume it.

**Independent Test**: Authenticate as a school reporting administrator, create a custom report definition from approved fields and filters, update non-historical metadata, activate/deactivate it, request a report run from it, and verify unsupported fields, cross-tenant references, and deleted definitions are rejected.

**Acceptance Scenarios**:

1. **Given** the actor has school-scoped custom report permission, **When** they retrieve the report catalog, **Then** the response lists only approved launch-scope domains, fields, filters, operators, grouping, sorting, and output-format compatibility for the resolved school.
2. **Given** the actor has school-scoped custom report permission, **When** they create a definition using only approved attendance, grades, academic structure, or school activity fields, filters, grouping, and output formats, **Then** the definition is saved in the resolved school with `draft` lifecycle state until explicitly activated.
3. **Given** a custom report definition is active in the resolved school, **When** an authorized actor requests a report from it with valid runtime filters, **Then** an asynchronous report run is created using the definition snapshot and same-school filter values.
4. **Given** a custom report definition has already been used by one or more report runs, **When** an authorized actor updates allowed metadata or updates structural configuration after deactivation, **Then** existing report runs retain their original definition snapshot and future runs use the updated active definition snapshot.
5. **Given** a custom definition references unsupported fields, hidden guardian/student/private content fields, arbitrary query text, cross-tenant data, unsupported filter operators, or exceeds v1 complexity limits, **When** the definition is submitted, **Then** validation rejects it with the documented error response.

---

### User Story 3 - Govern Report Outputs and Filters (Priority: P3)

A school reporting administrator can request supported report outputs with documented filters and formats, including spreadsheet output where approved, while retention and expired-output rules remain predictable.

**Why this priority**: Output compatibility and filter expansion affect client behavior, storage, retention, and user expectations. These rules must be explicit before backend behavior changes.

**Independent Test**: Request built-in and custom reports with supported formats and filters, verify PDF, CSV, and spreadsheet output availability before expiry, verify unsupported format/filter combinations fail validation, and verify expired output behavior remains contract-compliant.

**Acceptance Scenarios**:

1. **Given** a built-in or custom report supports PDF, CSV, and XLSX outputs, **When** an authorized actor requests those formats, **Then** the accepted report run records the requested formats and exposes availability only for generated, unexpired outputs.
2. **Given** a report type or custom definition does not support a requested format or filter, **When** an actor submits the report request, **Then** validation rejects the unsupported combination before creating a report run.
3. **Given** a generated report output is still inside the 90-day retention window, **When** an authorized actor downloads a supported format, **Then** the backend returns only the requested authorized file without exposing storage paths.
4. **Given** a generated report output is expired, unsupported for the requested format, associated with a canceled or failed run, or belongs to another school, **When** an actor attempts download, **Then** the backend returns the documented response without regenerating output during download.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized school context before report lifecycle, definition, filter, output, or audit data is accessed?
- What happens when a retry is requested for a generated, canceled, deleted, already retried, or still-running report run?
- What happens when cancellation races with successful generation or when a worker finishes after a run was canceled?
- What happens when retry, cancel, delete, restore, worker completion, or output expiry attempts race against each other for the same report run?
- How does delete/restore behave for report runs with generated outputs that are unexpired, expired, failed, canceled, or already deleted?
- How does the backend prevent report definition updates from changing the historical meaning of previously generated report runs?
- What happens when a custom report definition attempts to include private file paths, guardian-restricted fields, questionnaire answer keys, correction history, school-only notes, credentials, or report outputs from another school?
- What happens when a custom report definition exceeds allowed field, filter, grouping, or sorting limits?
- What happens when an actor attempts to change an active custom definition's domain, selected fields, filters, grouping, sorting, or output formats?
- How are unsupported format and filter combinations rejected for built-in versus custom report definitions?
- How does the backend preserve 90-day output retention while allowing soft-deleted report runs to be restored?
- How does the backend keep report-run soft delete from deleting, restoring, or extending generated output files?
- How does the backend ensure platform administrators and support users do not receive implicit access to school-scoped report lifecycle or custom definition workflows?
- How does the backend audit report lifecycle, report definition, report catalog, output, validation, conflict, and denied-access outcomes without storing private report contents or filter payloads?
- How does the backend avoid implementing frontend report designer behavior, platform-wide reporting, legal hold, permanent purge, anonymization, billing, messaging, notifications, or undocumented APIs in this slice?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement backend-only report lifecycle and custom report definition behavior for approved `/api/v1` operations, including validation, authorization, tenant context enforcement, response resources, report-run services, output lifecycle handling, audit events, and regression coverage.
- **Frontend repository impact**: No frontend implementation is included in this feature. Report designer and reporting screens may consume this behavior only after the OpenAPI contract and backend implementation are approved.
- **Specification or contract repository impact**: OpenAPI expansion is required before backend implementation exposes report retry, cancellation, delete/restore, custom report definitions, report catalog discovery, custom report run requests, additional XLSX output, expanded filters, or new report lifecycle states.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines the product and contract boundary first, `schoolmaster-backend` implements only approved operations next, and `schoolmaster-frontend` remains blocked from using the new behavior until the backend remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: Yes. The aggregate OpenAPI contract must add the report lifecycle, report definition, report catalog, expanded request, expanded response, audit-relevant state, conflict, and output-format semantics before backend exposure.
- **Versioned endpoints affected**: Existing `/api/v1/reports` and `/api/v1/reports/{reportRunId}/download` behavior must remain compatible; new or expanded `/api/v1/reports/{reportRunId}/retry`, `/api/v1/reports/{reportRunId}/cancel`, `/api/v1/reports/{reportRunId}`, `/api/v1/reports/{reportRunId}/restore`, `/api/v1/report-catalog`, `/api/v1/report-definitions`, and `/api/v1/report-definitions/{reportDefinitionId}` operations must be documented before implementation.
- **JSON response impact**: Report JSON responses must use documented success, accepted, paginated, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, gone/expired-output, generation-status, soft-delete, and per-output-availability semantics only. Binary output downloads remain documented per format.
- **Authentication/authorization impact**: All operations require authenticated access, active permitted school context, active user status, and explicit school-scoped report or report-definition permission. Platform administration and support access do not imply school report access.
- **Compatibility impact**: The feature is additive against existing reporting behavior. Existing report request, list, download, PDF/CSV output, 90-day retention, and expired-output behavior must remain compatible unless OpenAPI explicitly documents a versioned change.

### Data & Tenancy Impact

- **Tenant scoping impact**: Report runs, report outputs, custom report definitions, report definition snapshots, filter values, output format selections, lifecycle reasons, and audit events are school-owned records governed by `school_id`.
- **Cross-tenant or platform access impact**: Cross-school report runs, definitions, filters, academic periods, users, student profiles, guardian data, outputs, and audit entries must be rejected before data exposure. Platform-wide reporting and support-user visibility remain outside this slice.
- **Soft delete impact**: Report runs and custom report definitions use soft delete semantics separate from generation status. Report-run delete hides records from default operational lists while preserving audit history and historical report-run meaning; report-run restore clears the soft-delete marker while preserving generation status and output availability. Restored custom report definitions return to `inactive` and require separate activation before use.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend implementation slice MUST expose only report lifecycle, report definition, report output, and report request behavior documented by the expanded OpenAPI contract.
- **FR-002**: Every operation in this slice MUST resolve an active permitted school context before validation that depends on school-owned report runs, report definitions, filter references, output files, audit events, persistence, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any report run, report output, report definition, academic-period, student-profile, user, guardian, audit, or file data is accessed.
- **FR-004**: Report lifecycle operations MUST require explicit school-scoped report lifecycle permission in the resolved school.
- **FR-005**: Custom report definition operations MUST require explicit school-scoped report definition permission in the resolved school.
- **FR-006**: Report retry MUST create a new `ReportRun` linked to the original run and MUST preserve the original run, filters, output formats, status, failure reason, and audit history.
- **FR-007**: Report retry MUST be allowed only for failed report runs and expired generated report runs where the original filters and definition snapshot remain valid in the resolved school.
- **FR-008**: Report retry MUST reject requested, generating, generated-unexpired, canceled, deleted, cross-tenant, unauthorized, or already-superseded report runs with the documented validation, conflict, not-found, or authorization response.
- **FR-009**: Report cancellation MUST be allowed only while a report run is not terminal and before output publication.
- **FR-010**: Report cancellation MUST require a predefined tenant-safe reason code and MUST prevent later publication of outputs for the canceled run, including stale worker completion attempts.
- **FR-011**: Report cancellation MUST NOT delete historical metadata, filter summaries, definition snapshots, or audit events.
- **FR-012**: Report delete MUST soft-delete report runs from default operational lists while preserving metadata, audit history, definition snapshots, and retention records.
- **FR-013**: Report restore MUST clear the report run soft-delete marker while preserving generation status, output availability, definition snapshots, and audit history; restore MUST NOT make expired outputs downloadable, restart generation, or mutate original generation timestamps.
- **FR-014**: Report run lifecycle MUST represent generation progress with `requested`, `generating`, `generated`, `failed`, and `canceled` generation statuses; soft deletion MUST be tracked separately from generation status, and output availability MUST be tracked per generated output format.
- **FR-015**: Manual report status mutation MUST NOT be exposed as a free-form administrator operation in this slice; all visible state changes MUST occur through documented request, worker completion, retry, cancel, delete, restore, or output-expiry workflows.
- **FR-016**: Report listing MUST include requested, generating, generated, failed, canceled, expired-output, retried, custom-definition, and built-in report runs by default when they are not soft-deleted.
- **FR-017**: Report listing MUST omit soft-deleted report runs by default and MUST expose them only through an explicit documented include-deleted filter.
- **FR-018**: Concurrent report lifecycle actions MUST use first-valid-transition-wins semantics; later conflicting retry, cancel, delete, restore, worker completion, or output-expiry attempts MUST return or record the documented conflict outcome without partial state changes.
- **FR-019**: Stale worker completions for canceled, superseded, or otherwise terminal generation states MUST be ignored and audited without publishing outputs or mutating generation status. Soft delete or restore alone MUST NOT block a later valid worker completion for a run that is still `requested` or `generating`.
- **FR-020**: Custom report definitions MUST belong to exactly one school and MUST NOT be shared across schools unless a future platform-wide reporting specification approves that behavior.
- **FR-021**: Custom report definitions MUST be limited to the existing launch-scope report domains: attendance, grades, academic structure, and school activity.
- **FR-022**: The backend MUST expose a read-only report catalog operation documenting the approved launch-scope report domains, fields, filter operators, grouping options, sorting options, output formats, and format compatibility available in the resolved school.
- **FR-023**: Custom report definitions MUST be built only from the approved catalog documented in OpenAPI and returned by the report catalog operation.
- **FR-024**: Custom report definitions MUST reject arbitrary query text, executable expressions, unsupported joins, hidden fields, private file paths, report output references, credentials, school-only notes, full correction history, guardian-restricted fields, questionnaire answer keys, non-launch-scope domains, cross-tenant references, and any catalog entry not approved for the resolved school.
- **FR-025**: Custom report definitions MUST allow no more than 25 selected fields, 10 filters, 2 grouping levels, and 3 sort fields.
- **FR-026**: Custom report definitions MUST reject submissions that exceed any v1 complexity limit before creating or updating the definition.
- **FR-027**: Custom report definitions MUST use `draft`, `active`, `inactive`, and `deleted` lifecycle states.
- **FR-028**: New custom report definitions MUST start in `draft`; only `active` custom report definitions may be used to request report runs.
- **FR-029**: Restoring a deleted custom report definition MUST return it to `inactive`; activation MUST be a separate documented transition with validation.
- **FR-030**: Updating a custom report definition that has prior report runs MUST NOT change historical report-run snapshots or downloaded historical outputs.
- **FR-031**: Requesting a custom report MUST snapshot the active report definition, runtime filters, selected fields, grouping, sorting, and requested output formats onto the created `ReportRun`.
- **FR-032**: Built-in and custom report requests MUST validate that every filter reference, including academic periods, users, student profiles, statuses, and date ranges, belongs to the resolved school and is allowed for the selected report type or definition.
- **FR-033**: XLSX MUST be the only additional output format approved by this specification beyond the existing PDF and CSV formats, and only for report types or custom definitions whose OpenAPI schema and report catalog response declare XLSX support.
- **FR-034**: Report output availability MUST be tracked per output format so one generated format can be available, pending, expired, failed, or unsupported independently from another documented format.
- **FR-035**: Generated report output files MUST keep the existing 90-day availability window unless a future retention specification changes it.
- **FR-036**: Expired report output downloads MUST return the documented expired-output response and MUST NOT regenerate files, enqueue retry, mutate the expired run, or change retention timestamps during download.
- **FR-037**: Report outputs MUST NOT expose explicit delete or restore actions in this slice; output lifecycle is limited to requested format support, generated availability, per-format generation failure, and 90-day expiry.
- **FR-038**: Report-run soft delete and restore MUST NOT delete, restore, extend, regenerate, or otherwise mutate generated output files or output retention timestamps.
- **FR-039**: Report output lifecycle responses MUST NOT expose private storage paths, storage keys, generation worker details, or cross-tenant file existence.
- **FR-040**: Report lifecycle, definition, catalog access, request, retry, cancellation, delete, restore, output expiry, denied access, validation failure, and conflict outcomes MUST create tenant-safe audit events without storing private payloads, free-text lifecycle reasons, credentials, file paths, generated report contents, full filter payloads, or unauthorized cross-tenant details.
- **FR-041**: Report audit events MUST include actor, action, outcome, target, school, correlation ID, and tenant-safe reason code.
- **FR-042**: Backend responses MUST match the expanded OpenAPI success, accepted, binary, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, and expired-output semantics for every approved operation.
- **FR-043**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, lifecycle conflicts, stale generation completion, retry eligibility, cancellation eligibility, delete/restore behavior, report catalog discovery, custom definition snapshots, unsupported fields and filters, complexity-limit rejection, XLSX support boundaries, output retention, expired-output handling, audit events, and response shapes.
- **FR-044**: Backend implementation MUST NOT expose frontend report designer behavior, platform-wide reporting, support-user overrides, student self-view changes, guardian self-service changes, teacher workflow corrections, roster changes, billing, messaging, notifications, permanent purge, legal hold, anonymization workflows, or undocumented APIs in this slice.
- **FR-045**: Custom report definition names MUST be unique per school among non-deleted definitions; restoring a deleted definition MUST return the documented conflict response if another non-deleted definition in the same school already uses that name.
- **FR-046**: Active custom report definitions MUST allow metadata-only edits, limited to name and description with normal validation; changes to domain, selected fields, filters, grouping, sorting, or output formats MUST require deactivation before update.

### Key Entities *(include if feature involves data)*

- **ReportRun**: Existing school-owned asynchronous report request; this feature expands lifecycle visibility with retry, cancellation, source run linkage, terminal generation statuses, separate soft-delete metadata, and definition snapshots.
- **ReportOutput**: Private school-owned generated file for one report run and one output format; availability is tracked per format, has no explicit delete or restore action in this slice, and remains governed by the 90-day retention window.
- **ReportDefinition**: School-owned custom report definition containing one approved launch-scope report domain, a name unique among the school's non-deleted definitions, selected fields, allowed filters, grouping, sorting, output formats, lifecycle state (`draft`, `active`, `inactive`, `deleted`), ownership, version metadata, v1 complexity limits, and an edit boundary where active definitions allow metadata-only updates while structural updates require deactivation.
- **ReportDefinitionSnapshot**: Immutable copy of the report definition and runtime filter shape captured when a report run is requested, preserving historical meaning after definition updates.
- **ReportCatalog**: Read-only school-scoped catalog of approved fields, operators, references, grouping options, sorting options, and format compatibility rules available to built-in and custom reports for attendance, grades, academic structure, and school activity.
- **ReportLifecycleEvent**: Tenant-safe audit and history record for retry, cancellation, delete, restore, generation completion, output expiry, validation failure, conflict, denied access, and report catalog access; entries include actor, action, outcome, target, school, correlation ID, and tenant-safe reason code.
- **AcademicPeriod**: Existing school-owned period used by built-in and custom report filters; every reference must resolve to the same permitted school.
- **User**: Existing authenticated actor identity; report lifecycle and definition actions require active status and explicit same-school permissions.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of new report lifecycle, definition, custom request, output, and format behavior to expanded OpenAPI operation IDs, schemas, and responses before implementation starts.
- **SC-002**: Tenant-isolation tests reject 100% of cross-school, missing-school, inactive-school, and unauthorized-school attempts for report runs, report outputs, report definitions, filters, and lifecycle actions.
- **SC-003**: Lifecycle tests verify 100% of approved retry, cancellation, delete, restore, expired-output, and stale worker completion transitions produce the documented state and reject unsupported transitions.
- **SC-004**: Custom definition tests reject 100% of unsupported fields, unsupported operators, arbitrary query text, hidden/private fields, cross-tenant references, and unsupported output formats covered by this specification.
- **SC-005**: Historical integrity tests confirm existing report runs and downloads retain their original definition snapshot and filter meaning after custom report definitions are updated, deleted, or restored.
- **SC-006**: Output tests confirm PDF, CSV, and approved XLSX outputs are generated, listed, downloaded, expired, and rejected independently according to documented per-format availability rules.
- **SC-007**: Retention tests confirm expired downloads return the documented expired-output response after 90 days without regenerating files, enqueueing retries, or mutating original report-run timestamps.
- **SC-008**: Authorization tests confirm platform administrators and support users receive no implicit school report lifecycle, output, or custom definition access unless a future approved contract grants it.
- **SC-009**: Audit verification confirms successful and denied lifecycle, definition, catalog, output, retry, cancellation, delete, restore, validation, conflict, and cross-tenant attempts create tenant-safe audit records containing actor, action, outcome, target, school, correlation ID, and tenant-safe reason code without private payload leakage.
- **SC-010**: Report listing tests confirm default lists include all non-deleted report runs and exclude soft-deleted runs unless the documented include-deleted filter is used.
- **SC-011**: Output lifecycle tests confirm report-run delete and restore do not delete, restore, extend, regenerate, or mutate generated output files or their retention timestamps.
- **SC-012**: Concurrency tests confirm conflicting lifecycle actions and stale worker completions never produce partial state, never publish outputs after cancellation, and return or record documented conflict outcomes.
- **SC-013**: Report catalog tests confirm the catalog exposes only approved launch-scope report domains and that custom definitions using unsupported or absent catalog entries are rejected.
- **SC-014**: Complexity-limit tests confirm custom definitions with more than 25 fields, 10 filters, 2 grouping levels, or 3 sort fields are rejected before persistence.
- **SC-015**: Definition identity tests confirm duplicate non-deleted custom report definition names are rejected per school and restoring a deleted definition with a reused name returns the documented conflict response.
- **SC-016**: Definition edit tests confirm active custom definitions allow only name and description updates, while structural changes to domain, fields, filters, grouping, sorting, or formats are rejected until the definition is inactive.

## Assumptions

- `005-backend-student-reporting` has already implemented school-scoped asynchronous report request, list, download, PDF/CSV output, 90-day output retention, expired-output handling, and explicit regeneration through a new `ReportRun`.
- `011-guardian-self-service` has been completed and does not grant report request, report output, report lifecycle, or custom report definition authority to guardians.
- Manual status mutation is intentionally constrained to named lifecycle operations; a free-form status override endpoint is out of scope for this slice.
- XLSX is the only additional v1 output format approved by this specification because it extends operational tabular reporting without introducing document-designer or archive-delivery scope.
- Custom report definitions are constrained to the existing launch-scope domains and do not allow arbitrary SQL, scripting, uploaded templates, custom code, cross-tenant joins, non-launch-scope domains, or unrestricted data exploration.
- Custom report definition activation is separate from creation and restore so draft or restored definitions cannot be requested before current validation passes.
- Active custom report definitions permit name and description updates only; structural report shape changes require deactivation to avoid request-time races.
- Custom report clients discover approved custom-report inputs from the read-only report catalog instead of relying on hardcoded frontend assumptions.
- Custom report complexity limits intentionally bound v1 validation, query cost, and output shape until later specifications approve broader report-designer behavior.
- Report output deletion, restoration, retention override, and permanent purge are out of scope for this slice.
- Report retention remains 90 days for generated output files, while durable report-run metadata and audit events remain available according to existing backend retention expectations.
- Platform-wide reporting, support-user cross-school visibility, legal hold, permanent purge, anonymization, and retention override workflows require separate specifications and OpenAPI expansion.
