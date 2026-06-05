# Tasks: Report Lifecycle Expansion

**Input**: Design documents from `specs/012-report-lifecycle-expansion/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: Critical business flow tests are required. This feature changes REST contracts and backend behavior, so include OpenAPI validation and PHPUnit feature/unit coverage. Frontend implementation is explicitly out of scope, so Vitest tasks are not included in this slice.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Every task includes an exact file path

## Path Conventions

- Specification and contract paths are in `schoolmaster-specs/`.
- Backend implementation paths are in `schoolmaster-backend/`.
- Frontend implementation is out of scope for this feature.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish the contract-first backend slice and shared implementation structure.

- [ ] T001 Register report lifecycle, catalog, and definition routes in `schoolmaster-specs/api/openapi.yaml`
- [ ] T002 Update existing report list/request operations for expanded filters, statuses, custom requests, and output formats in `schoolmaster-specs/api/paths/reports/index.yaml`
- [ ] T003 Update existing report download operation for XLSX, per-format availability, and expired-output semantics in `schoolmaster-specs/api/paths/reports/download.yaml`
- [ ] T004 [P] Add report retry operation contract in `schoolmaster-specs/api/paths/reports/retry.yaml`
- [ ] T005 [P] Add report cancel operation contract in `schoolmaster-specs/api/paths/reports/cancel.yaml`
- [ ] T006 [P] Add report delete/restore operation contracts in `schoolmaster-specs/api/paths/reports/report-run.yaml`
- [ ] T007 [P] Add report catalog operation contract in `schoolmaster-specs/api/paths/report-catalog/index.yaml`
- [ ] T008 [P] Add report definition operation contracts in `schoolmaster-specs/api/paths/report-definitions/index.yaml`
- [ ] T009 [P] Add report definition item, activation, deactivation, delete, and restore operation contracts in `schoolmaster-specs/api/paths/report-definitions/report-definition.yaml`
- [ ] T010 Add report lifecycle expansion schemas in `schoolmaster-specs/api/components/schemas/reports/ReportGenerationStatus.yaml`, `schoolmaster-specs/api/components/schemas/reports/ReportOutputAvailability.yaml`, `schoolmaster-specs/api/components/schemas/reports/ReportOutput.yaml`, `schoolmaster-specs/api/components/schemas/reports/ReportDefinition.yaml`, `schoolmaster-specs/api/components/schemas/reports/ReportDefinitionSnapshot.yaml`, `schoolmaster-specs/api/components/schemas/reports/ReportCatalog.yaml`, and `schoolmaster-specs/api/components/schemas/reports/ReportLifecycleEvent.yaml`
- [ ] T011 Mirror approved report lifecycle expansion in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T012 Run Redocly lint for the aggregate contract before backend implementation and record results in `schoolmaster-specs/specs/012-report-lifecycle-expansion/quickstart.md`
- [ ] T013 [P] Run Redocly lint for the platform contract before backend implementation and record results in `schoolmaster-specs/specs/012-report-lifecycle-expansion/quickstart.md`
- [ ] T014 [P] Create backend report DTO and enum directories in `schoolmaster-backend/app/DTOs/Reports/` and `schoolmaster-backend/app/Enums/Reports/`
- [ ] T015 [P] Create backend report request and resource directories in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/` and `schoolmaster-backend/app/Http/Resources/Reports/`
- [ ] T016 [P] Create backend report service directory in `schoolmaster-backend/app/Services/Reports/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core data, authorization, tenancy, state, audit, and response infrastructure that MUST be complete before any user story can be implemented.

**Critical**: No user story work can begin until this phase is complete.

- [ ] T017 Create report run lifecycle migration in `schoolmaster-backend/database/migrations/2026_06_05_000001_update_report_runs_for_lifecycle_expansion.php`
- [ ] T018 Create report outputs migration with per-format availability in `schoolmaster-backend/database/migrations/2026_06_05_000002_create_report_outputs_table.php`
- [ ] T019 Create report definitions migration with school-scoped non-deleted name uniqueness in `schoolmaster-backend/database/migrations/2026_06_05_000003_create_report_definitions_table.php`
- [ ] T020 Create report definition snapshots migration in `schoolmaster-backend/database/migrations/2026_06_05_000004_create_report_definition_snapshots_table.php`
- [ ] T021 Create report lifecycle events migration in `schoolmaster-backend/database/migrations/2026_06_05_000005_create_report_lifecycle_events_table.php`
- [ ] T022 Update ReportRun model relationships, UUIDs, soft-delete, and lifecycle casts in `schoolmaster-backend/app/Models/ReportRun.php`
- [ ] T023 [P] Create ReportOutput model with school ownership and availability casts in `schoolmaster-backend/app/Models/ReportOutput.php`
- [ ] T024 [P] Create ReportDefinition model with school ownership, lifecycle casts, and active edit boundary helpers in `schoolmaster-backend/app/Models/ReportDefinition.php`
- [ ] T025 [P] Create ReportDefinitionSnapshot model with immutable snapshot relationships in `schoolmaster-backend/app/Models/ReportDefinitionSnapshot.php`
- [ ] T026 [P] Create ReportLifecycleEvent model with tenant-safe audit casts in `schoolmaster-backend/app/Models/ReportLifecycleEvent.php`
- [ ] T027 [P] Create report generation status, output availability, definition state, and lifecycle reason-code enums in `schoolmaster-backend/app/Enums/Reports/`
- [ ] T028 Create report lifecycle policy for report-run view, retry, cancel, delete, restore, and download permissions in `schoolmaster-backend/app/Policies/ReportLifecyclePolicy.php`
- [ ] T029 Create report definition policy for catalog view and definition management permissions in `schoolmaster-backend/app/Policies/ReportDefinitionPolicy.php`
- [ ] T030 Register report policies in `schoolmaster-backend/app/Providers/AuthServiceProvider.php`
- [ ] T031 Create report actor context DTO in `schoolmaster-backend/app/DTOs/Reports/ReportActorContext.php`
- [ ] T032 [P] Create report lifecycle action DTO in `schoolmaster-backend/app/DTOs/Reports/ReportLifecycleActionData.php`
- [ ] T033 [P] Create report request DTO in `schoolmaster-backend/app/DTOs/Reports/ReportRequestData.php`
- [ ] T034 [P] Create report definition DTO in `schoolmaster-backend/app/DTOs/Reports/ReportDefinitionData.php`
- [ ] T035 Create report tenant context service for school-scoped report operations in `schoolmaster-backend/app/Services/Reports/ReportTenantContextService.php`
- [ ] T036 Create report audit service for tenant-safe lifecycle and denial events in `schoolmaster-backend/app/Services/Reports/ReportAuditService.php`
- [ ] T037 Create report response resource base helpers in `schoolmaster-backend/app/Http/Resources/Reports/ReportErrorResource.php`
- [ ] T038 Create report controller shell for run lifecycle and download endpoints in `schoolmaster-backend/app/Http/Controllers/Api/V1/ReportController.php`
- [ ] T039 Create report definition controller shell for catalog and definition endpoints in `schoolmaster-backend/app/Http/Controllers/Api/V1/ReportDefinitionController.php`
- [ ] T040 Register report lifecycle, catalog, and definition routes in `schoolmaster-backend/routes/api.php`
- [ ] T041 [P] Add report run factory states for requested, generating, generated, failed, canceled, deleted, retried, and expired-output runs in `schoolmaster-backend/database/factories/ReportRunFactory.php`
- [ ] T042 [P] Add report output factory states for pending, available, failed, expired, and unsupported availability in `schoolmaster-backend/database/factories/ReportOutputFactory.php`
- [ ] T043 [P] Add report definition and snapshot factories in `schoolmaster-backend/database/factories/ReportDefinitionFactory.php`
- [ ] T044 [P] Add report lifecycle event factory in `schoolmaster-backend/database/factories/ReportLifecycleEventFactory.php`
- [ ] T045 [P] Add PHPUnit feature tests for missing, mismatched, inactive, and unauthorized school context across report lifecycle, definition, catalog, and output endpoints in `schoolmaster-backend/tests/Feature/Reports/ReportTenantContextTest.php`
- [ ] T046 [P] Add PHPUnit feature tests for platform and support users receiving no implicit school-scoped report access in `schoolmaster-backend/tests/Feature/Reports/ReportPlatformAccessBoundaryTest.php`

**Checkpoint**: Foundation ready - user story implementation can now begin.

---

## Phase 3: User Story 1 - Manage Report Run Lifecycle (Priority: P1) MVP

**Goal**: A school-scoped reporting administrator can retry failed report runs, cancel in-progress runs, soft-delete report runs from default lists, and restore deleted runs while preserving audit history and tenant isolation.

**Independent Test**: Authenticate as a school reporting administrator, create same-school report runs in retryable, cancellable, generated, failed, and deleted states, perform each lifecycle action, and verify state transitions, response envelopes, audit entries, list visibility, conflict handling, and cross-tenant denials.

### Tests for User Story 1

- [ ] T047 [P] [US1] Add OpenAPI contract coverage for `retryReport`, `cancelReport`, `deleteReport`, `restoreReport`, and expanded `listReports` in `schoolmaster-specs/api/paths/reports/retry.yaml`, `schoolmaster-specs/api/paths/reports/cancel.yaml`, `schoolmaster-specs/api/paths/reports/report-run.yaml`, and `schoolmaster-specs/api/paths/reports/index.yaml`
- [ ] T048 [P] [US1] Add PHPUnit feature tests for report retry eligibility, retry lineage, and original-run preservation in `schoolmaster-backend/tests/Feature/Reports/ReportRetryTest.php`
- [ ] T049 [P] [US1] Add PHPUnit feature tests for report cancellation, predefined reason-code validation, and stale worker completion rejection in `schoolmaster-backend/tests/Feature/Reports/ReportCancellationTest.php`
- [ ] T050 [P] [US1] Add PHPUnit feature tests for report-run soft delete, restore, and default include-deleted list behavior in `schoolmaster-backend/tests/Feature/Reports/ReportRunSoftDeleteRestoreTest.php`
- [ ] T051 [P] [US1] Add PHPUnit feature tests for lifecycle authorization, cross-tenant denial, and non-enumerating target responses in `schoolmaster-backend/tests/Feature/Reports/ReportLifecycleAuthorizationTest.php`
- [ ] T052 [P] [US1] Add PHPUnit unit tests for first-valid-transition-wins conflict behavior in `schoolmaster-backend/tests/Unit/Reports/ReportLifecycleServiceTest.php`
- [ ] T053 [P] [US1] Add PHPUnit feature tests for tenant-safe lifecycle audit events in `schoolmaster-backend/tests/Feature/Reports/ReportLifecycleAuditTest.php`

### Implementation for User Story 1

- [ ] T054 [P] [US1] Implement ListReportsRequest with generation-status, built-in/custom, and include-deleted filters in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/ListReportsRequest.php`
- [ ] T055 [P] [US1] Implement CancelReportRequest with predefined reason-code validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/CancelReportRequest.php`
- [ ] T056 [P] [US1] Implement ReportRunResource with generation status, soft-delete metadata, retry lineage, and output availability summary in `schoolmaster-backend/app/Http/Resources/Reports/ReportRunResource.php`
- [ ] T057 [P] [US1] Implement ReportLifecycleEventResource for tenant-safe audit summaries in `schoolmaster-backend/app/Http/Resources/Reports/ReportLifecycleEventResource.php`
- [ ] T058 [US1] Implement report-run list query logic with default non-deleted visibility in `schoolmaster-backend/app/Services/Reports/ReportRunQueryService.php`
- [ ] T059 [US1] Implement retry, cancel, delete, and restore transitions in `schoolmaster-backend/app/Services/Reports/ReportLifecycleService.php`
- [ ] T060 [US1] Implement stale worker completion guard and terminal-state conflict handling in `schoolmaster-backend/app/Services/Reports/ReportLifecycleService.php`
- [ ] T061 [US1] Implement lifecycle audit writes for retry, cancel, delete, restore, denial, conflict, and stale completion outcomes in `schoolmaster-backend/app/Services/Reports/ReportAuditService.php`
- [ ] T062 [US1] Wire list, retry, cancel, delete, and restore controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/ReportController.php`
- [ ] T063 [US1] Verify backend report lifecycle routes map to approved OpenAPI operation IDs in `schoolmaster-backend/routes/api.php`

**Checkpoint**: User Story 1 is fully functional and independently testable.

---

## Phase 4: User Story 2 - Define Custom School Reports (Priority: P2)

**Goal**: A school reporting administrator can discover approved report catalog options, create and maintain school-owned custom report definitions, activate/deactivate/delete/restore them, and request reports from active definitions without unrestricted data access.

**Independent Test**: Authenticate as a school reporting administrator, retrieve the catalog, create a valid draft definition, activate it, request a report from it, update allowed metadata, deactivate for structural edits, and verify unsupported fields, duplicate names, restore conflicts, cross-tenant references, and deleted definitions are rejected.

### Tests for User Story 2

- [ ] T064 [P] [US2] Add OpenAPI contract coverage for `getReportCatalog` and report definition operations in `schoolmaster-specs/api/paths/report-catalog/index.yaml`, `schoolmaster-specs/api/paths/report-definitions/index.yaml`, and `schoolmaster-specs/api/paths/report-definitions/report-definition.yaml`
- [ ] T065 [P] [US2] Add PHPUnit feature tests for report catalog visibility and hidden-field exclusion in `schoolmaster-backend/tests/Feature/Reports/ReportCatalogTest.php`
- [ ] T066 [P] [US2] Add PHPUnit feature tests for custom definition create, update, activate, deactivate, delete, and restore lifecycle in `schoolmaster-backend/tests/Feature/Reports/ReportDefinitionLifecycleTest.php`
- [ ] T067 [P] [US2] Add PHPUnit feature tests for duplicate non-deleted definition names and restore name conflicts in `schoolmaster-backend/tests/Feature/Reports/ReportDefinitionNameUniquenessTest.php`
- [ ] T068 [P] [US2] Add PHPUnit feature tests for active-definition metadata-only updates and structural-edit rejection in `schoolmaster-backend/tests/Feature/Reports/ReportDefinitionActiveEditTest.php`
- [ ] T069 [P] [US2] Add PHPUnit feature tests for custom report request snapshot preservation after definition update, delete, and restore in `schoolmaster-backend/tests/Feature/Reports/CustomReportRequestSnapshotTest.php`
- [ ] T070 [P] [US2] Add PHPUnit unit tests for catalog validation, complexity limits, unsupported entries, and cross-tenant reference rejection in `schoolmaster-backend/tests/Unit/Reports/ReportDefinitionValidationServiceTest.php`

### Implementation for User Story 2

- [ ] T071 [P] [US2] Implement ReportCatalogResource in `schoolmaster-backend/app/Http/Resources/Reports/ReportCatalogResource.php`
- [ ] T072 [P] [US2] Implement ReportDefinitionResource in `schoolmaster-backend/app/Http/Resources/Reports/ReportDefinitionResource.php`
- [ ] T073 [P] [US2] Implement CreateReportDefinitionRequest with catalog, domain, complexity, and name validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/CreateReportDefinitionRequest.php`
- [ ] T074 [P] [US2] Implement UpdateReportDefinitionRequest with active metadata-only and inactive structural-edit validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/UpdateReportDefinitionRequest.php`
- [ ] T075 [P] [US2] Implement RequestCustomReportRequest with active-definition and runtime-filter validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/RequestCustomReportRequest.php`
- [ ] T076 [US2] Implement read-only launch-scope catalog resolution in `schoolmaster-backend/app/Services/Reports/ReportCatalogService.php`
- [ ] T077 [US2] Implement custom definition create, update, activate, deactivate, delete, and restore behavior in `schoolmaster-backend/app/Services/Reports/ReportDefinitionService.php`
- [ ] T078 [US2] Implement definition-name uniqueness and restore conflict checks in `schoolmaster-backend/app/Services/Reports/ReportDefinitionService.php`
- [ ] T079 [US2] Implement active metadata-only edit boundary and inactive structural-edit validation in `schoolmaster-backend/app/Services/Reports/ReportDefinitionValidationService.php`
- [ ] T080 [US2] Implement definition snapshot creation for custom report requests in `schoolmaster-backend/app/Services/Reports/ReportDefinitionSnapshotService.php`
- [ ] T081 [US2] Wire catalog and definition controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/ReportDefinitionController.php`
- [ ] T082 [US2] Wire custom definition request path through report request service in `schoolmaster-backend/app/Services/Reports/ReportRequestService.php`
- [ ] T083 [US2] Add tenant-safe audit writes for catalog reads, definition lifecycle actions, definition validation failures, and custom report requests in `schoolmaster-backend/app/Services/Reports/ReportAuditService.php`

**Checkpoint**: User Story 2 is fully functional and independently testable.

---

## Phase 5: User Story 3 - Govern Report Outputs and Filters (Priority: P3)

**Goal**: A school reporting administrator can request supported built-in and custom report outputs, including XLSX where catalog-approved, while filters, output availability, retention, and expired-output behavior remain predictable.

**Independent Test**: Request built-in and custom reports with supported formats and filters, verify PDF, CSV, and XLSX availability before expiry, verify unsupported format/filter combinations fail validation, and verify expired output downloads return the documented response without regeneration.

### Tests for User Story 3

- [ ] T084 [P] [US3] Add OpenAPI contract coverage for expanded `requestReport`, `downloadReport`, `ReportFormat`, `ReportOutput`, and output availability schemas in `schoolmaster-specs/api/paths/reports/index.yaml`, `schoolmaster-specs/api/paths/reports/download.yaml`, `schoolmaster-specs/api/components/schemas/reports/ReportFormat.yaml`, `schoolmaster-specs/api/components/schemas/reports/ReportOutput.yaml`, and `schoolmaster-specs/api/components/schemas/reports/ReportOutputAvailability.yaml`
- [ ] T085 [P] [US3] Add PHPUnit feature tests for built-in and custom report requests with PDF, CSV, and approved XLSX outputs in `schoolmaster-backend/tests/Feature/Reports/ReportOutputFormatRequestTest.php`
- [ ] T086 [P] [US3] Add PHPUnit feature tests for unsupported output format, unsupported filter, unsupported domain, and cross-tenant filter rejection in `schoolmaster-backend/tests/Feature/Reports/ReportRequestValidationTest.php`
- [ ] T087 [P] [US3] Add PHPUnit feature tests for per-format output availability and absence of deleted output availability in `schoolmaster-backend/tests/Feature/Reports/ReportOutputAvailabilityTest.php`
- [ ] T088 [P] [US3] Add PHPUnit feature tests for expired output downloads returning the documented response without regeneration or timestamp mutation in `schoolmaster-backend/tests/Feature/Reports/ReportOutputExpirationTest.php`
- [ ] T089 [P] [US3] Add PHPUnit unit tests for report output compatibility and retention rules in `schoolmaster-backend/tests/Unit/Reports/ReportOutputServiceTest.php`

### Implementation for User Story 3

- [ ] T090 [P] [US3] Implement ReportRequestResource with requested formats and per-format availability in `schoolmaster-backend/app/Http/Resources/Reports/ReportRequestResource.php`
- [ ] T091 [P] [US3] Implement ReportOutputResource without storage paths or deleted availability in `schoolmaster-backend/app/Http/Resources/Reports/ReportOutputResource.php`
- [ ] T092 [P] [US3] Implement RequestReportRequest for built-in/custom filters, format compatibility, and tenant reference validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/RequestReportRequest.php`
- [ ] T093 [P] [US3] Implement DownloadReportRequest for requested format, same-school output access, and expired-output validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Reports/DownloadReportRequest.php`
- [ ] T094 [US3] Implement built-in report request validation and queue handoff in `schoolmaster-backend/app/Services/Reports/ReportRequestService.php`
- [ ] T095 [US3] Implement per-format output creation, availability transitions, and retention timestamp handling in `schoolmaster-backend/app/Services/Reports/ReportOutputService.php`
- [ ] T096 [US3] Implement XLSX output support where catalog-approved in `schoolmaster-backend/app/Services/Reports/ReportOutputGenerationService.php`
- [ ] T097 [US3] Implement download response handling without exposing storage paths in `schoolmaster-backend/app/Services/Reports/ReportDownloadService.php`
- [ ] T098 [US3] Wire request and download controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/ReportController.php`
- [ ] T099 [US3] Add tenant-safe audit writes for report requests, output generation failures, output expiry, downloads, and denied output access in `schoolmaster-backend/app/Services/Reports/ReportAuditService.php`

**Checkpoint**: User Story 3 is fully functional and independently testable.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Contract validation, regression hardening, documentation, and release readiness across all stories.

- [ ] T100 Run Redocly lint for aggregate OpenAPI and record results in `schoolmaster-specs/specs/012-report-lifecycle-expansion/quickstart.md`
- [ ] T101 [P] Run Redocly lint for platform contract and record results in `schoolmaster-specs/specs/012-report-lifecycle-expansion/quickstart.md`
- [ ] T102 Run backend PHPUnit suite and record results in `schoolmaster-specs/specs/012-report-lifecycle-expansion/quickstart.md`
- [ ] T103 [P] Review OpenAPI route-to-operation traceability in `schoolmaster-specs/specs/012-report-lifecycle-expansion/contracts/backend-report-lifecycle-expansion.md`
- [ ] T104 [P] Review backend route-to-operation traceability in `schoolmaster-backend/routes/api.php`
- [ ] T105 [P] Review report audit payload redaction and reason-code usage in `schoolmaster-backend/app/Services/Reports/ReportAuditService.php`
- [ ] T106 [P] Review report output retention and no-output-delete behavior in `schoolmaster-backend/app/Services/Reports/ReportOutputService.php`
- [ ] T107 [P] Review report definition active-edit and name uniqueness behavior in `schoolmaster-backend/app/Services/Reports/ReportDefinitionService.php`
- [ ] T108 Update backend implementation notes for report lifecycle expansion in `schoolmaster-backend/README.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks all user stories.
- **Phase 3 US1**: Depends on Phase 2. This is the MVP.
- **Phase 4 US2**: Depends on Phase 2 and may integrate with US1 list/lifecycle response conventions.
- **Phase 5 US3**: Depends on Phase 2 and may integrate with US1 report-run lifecycle and US2 custom definition snapshots.
- **Phase 6 Polish**: Depends on all desired user stories being complete.

### User Story Dependencies

- **US1 Manage Report Run Lifecycle**: No dependency on other stories after foundation.
- **US2 Define Custom School Reports**: Can start after foundation; custom report request integration is clearer after US1 report-run responses exist.
- **US3 Govern Report Outputs and Filters**: Can start after foundation; full custom report coverage benefits from US2 snapshots but built-in output behavior remains independently testable.

### Within Each User Story

- Contract and PHPUnit tests first.
- Request validation and resources before controller wiring.
- Services before controller actions.
- Audit writes integrated with each story's service path.
- Route-to-operation verification before story checkpoint.

---

## Parallel Opportunities

- Setup path and schema tasks T004-T010 can run in parallel after T001.
- Early contract validation tasks T012-T013 can run in parallel after T001-T011.
- Backend directory setup tasks T014-T016 can run in parallel.
- Foundational model tasks T023-T027 can run in parallel after migrations T017-T021.
- Foundational factory and tenant-boundary tests T041-T046 can run in parallel after model contracts are drafted.
- US1 tests T047-T053 can run in parallel.
- US1 request/resource tasks T054-T057 can run in parallel.
- US2 tests T064-T070 can run in parallel.
- US2 request/resource tasks T071-T075 can run in parallel.
- US3 tests T084-T089 can run in parallel.
- US3 request/resource tasks T090-T093 can run in parallel.
- Polish review tasks T101 and T103-T107 can run in parallel.

---

## Parallel Example: User Story 1

```bash
# Launch US1 tests together:
Task: "T047 OpenAPI contract coverage for retryReport, cancelReport, deleteReport, restoreReport, and expanded listReports"
Task: "T048 PHPUnit feature tests for report retry eligibility and lineage"
Task: "T049 PHPUnit feature tests for report cancellation and stale worker completion"
Task: "T050 PHPUnit feature tests for soft delete, restore, and list behavior"
Task: "T051 PHPUnit feature tests for lifecycle authorization and tenant isolation"
Task: "T052 PHPUnit unit tests for lifecycle conflict behavior"
Task: "T053 PHPUnit feature tests for lifecycle audit events"

# Launch US1 HTTP shaping work together:
Task: "T054 ListReportsRequest"
Task: "T055 CancelReportRequest"
Task: "T056 ReportRunResource"
Task: "T057 ReportLifecycleEventResource"
```

## Parallel Example: User Story 2

```bash
# Launch US2 tests together:
Task: "T064 OpenAPI contract coverage for catalog and definition operations"
Task: "T065 PHPUnit feature tests for catalog visibility"
Task: "T066 PHPUnit feature tests for definition lifecycle"
Task: "T067 PHPUnit feature tests for duplicate names and restore conflicts"
Task: "T068 PHPUnit feature tests for active-definition edit boundary"
Task: "T069 PHPUnit feature tests for custom request snapshots"
Task: "T070 PHPUnit unit tests for catalog and definition validation"

# Launch US2 request/resource work together:
Task: "T071 ReportCatalogResource"
Task: "T072 ReportDefinitionResource"
Task: "T073 CreateReportDefinitionRequest"
Task: "T074 UpdateReportDefinitionRequest"
Task: "T075 RequestCustomReportRequest"
```

## Parallel Example: User Story 3

```bash
# Launch US3 tests together:
Task: "T084 OpenAPI contract coverage for report request/download output behavior"
Task: "T085 PHPUnit feature tests for PDF, CSV, and XLSX output requests"
Task: "T086 PHPUnit feature tests for unsupported filters and formats"
Task: "T087 PHPUnit feature tests for per-format output availability"
Task: "T088 PHPUnit feature tests for expired output downloads"
Task: "T089 PHPUnit unit tests for output compatibility and retention"

# Launch US3 request/resource work together:
Task: "T090 ReportRequestResource"
Task: "T091 ReportOutputResource"
Task: "T092 RequestReportRequest"
Task: "T093 DownloadReportRequest"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup and OpenAPI path/schema registration.
2. Complete Phase 2 foundational data, tenancy, policy, audit, route, and factory work.
3. Complete Phase 3 User Story 1 lifecycle management.
4. Stop and validate retry, cancel, delete, restore, list visibility, tenant isolation, conflicts, audit, and response shapes.

### Incremental Delivery

1. Deliver US1 report lifecycle management as the MVP.
2. Add US2 custom report catalog and definition lifecycle.
3. Add US3 output and filter governance.
4. Run cross-story contract lint, backend PHPUnit, traceability review, audit review, and retention review.

### Parallel Team Strategy

1. Complete Setup and Foundational phases together.
2. After foundation, split by story if needed:
   - Developer A: US1 report lifecycle.
   - Developer B: US2 custom definitions.
   - Developer C: US3 output and filter governance.
3. Reconcile through shared route, OpenAPI, audit, and report service contracts before polish.

---

## Notes

- [P] tasks use different files or can start from stable contracts without waiting on another task in the same phase.
- Frontend implementation is out of scope; generated clients or UI work require a later frontend feature.
- OpenAPI changes must land before backend route behavior is exposed.
