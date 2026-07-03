# Tasks: Reporting Workspace UI

**Input**: Design documents from `specs/026-reporting-workspace-ui/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/reporting-workspace-ui-contract.md`, `quickstart.md`

**Tests**: Frontend behavior changes require Vitest service, composable, route/component integration, and accessibility-oriented announcement coverage. Backend and OpenAPI work is out of scope unless an approved reporting contract gap is found during implementation.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared reporting foundations are complete.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or can be completed without depending on another incomplete task.
- **[Story]**: User story label for story phases only.
- Every task includes an exact file path.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm contract scope, target frontend structure, and shared implementation locations before story work begins.

- [ ] T001 Review reporting UX scope and blocked actions in `specs/026-reporting-workspace-ui/spec.md`
- [ ] T002 Verify approved reporting operation IDs in `api/openapi.yaml` and record no-change status in `specs/026-reporting-workspace-ui/quickstart.md`
- [ ] T003 [P] Confirm frontend reporting target folders in `schoolmaster-frontend/src/pages/reporting/`, `schoolmaster-frontend/src/components/reporting/`, `schoolmaster-frontend/src/composables/reporting/`, `schoolmaster-frontend/src/services/reporting/`, and `schoolmaster-frontend/src/contracts/reporting/`
- [ ] T004 [P] Confirm frontend test target folders in `schoolmaster-frontend/tests/reporting-workspace/fixtures/`, `schoolmaster-frontend/tests/reporting-workspace/services/`, `schoolmaster-frontend/tests/reporting-workspace/composables/`, and `schoolmaster-frontend/tests/reporting-workspace/components/`
- [ ] T005 [P] Add reporting i18n namespace placeholder in `schoolmaster-frontend/src/i18n/reporting.js`
- [ ] T006 [P] Confirm reporting route naming and navigation placement against existing protected-shell routes in `schoolmaster-frontend/src/router/index.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared contract mapping, access gates, feedback states, stale-response guards, and workspace shell required by all reporting stories.

**Critical**: No user story work can begin until this phase is complete.

### Tests for Foundation

- [ ] T007 [P] Add reporting fixture builders for catalog, report runs, outputs, definitions, and error envelopes in `schoolmaster-frontend/tests/reporting-workspace/fixtures/reportingFixtures.js`
- [ ] T008 [P] Add reporting service contract mapper tests for approved operations and undocumented-field dropping in `schoolmaster-frontend/tests/reporting-workspace/services/reportingService.test.js`
- [ ] T009 [P] Add reporting access-gate composable tests for active school, inactive school, no active school, report, lifecycle, definition, and definition-only catalog permissions in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportingAccess.test.js`
- [ ] T010 [P] Add stale-response and safe-diagnostics tests in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportingRequestGuards.test.js`
- [ ] T011 [P] Add shared reporting feedback and status component tests in `schoolmaster-frontend/tests/reporting-workspace/components/ReportingSharedStates.test.js`

### Implementation for Foundation

- [ ] T012 Create reporting contract mappers and constants in `schoolmaster-frontend/src/contracts/reporting/reportingContract.js`
- [ ] T013 Implement approved reporting HTTP service functions in `schoolmaster-frontend/src/services/reporting/reportingService.js`
- [ ] T014 Implement reporting error-to-feedback normalization in `schoolmaster-frontend/src/services/reporting/reportingErrorMapper.js`
- [ ] T015 Implement reporting access gate composable in `schoolmaster-frontend/src/composables/reporting/useReportingAccess.js`
- [ ] T016 Implement reporting stale-response and safe-diagnostics guards in `schoolmaster-frontend/src/composables/reporting/useReportingRequestGuards.js`
- [ ] T017 Implement protected reporting workspace route shell in `schoolmaster-frontend/src/pages/reporting/ReportingWorkspacePage.vue`
- [ ] T018 Wire reporting root, history, catalog, run-detail, and custom-definition child routes in `schoolmaster-frontend/src/router/index.js`
- [ ] T019 [P] Implement shared reporting feedback, empty, denied, unavailable, conflict, expired, and stale states in `schoolmaster-frontend/src/components/reporting/ReportingFeedbackState.vue`
- [ ] T020 [P] Implement shared report status, output availability, retention, and definition lifecycle badges in `schoolmaster-frontend/src/components/reporting/ReportingStatusBadges.vue`
- [ ] T021 [P] Implement polite announcement region for reporting state changes in `schoolmaster-frontend/src/components/reporting/ReportingAnnouncementRegion.vue`
- [ ] T022 Add shared reporting display text for foundation states in `schoolmaster-frontend/src/i18n/reporting.js`

**Checkpoint**: Foundation ready. User stories can now proceed in priority order or in parallel with separate owners.

---

## Phase 3: User Story 1 - Browse Report Catalog and Request Reports (Priority: P1) MVP

**Goal**: Authorized reporting users can browse approved catalog options and submit built-in or active custom-definition report requests through documented contracts only.

**Independent Test**: Sign in with active same-school report permission, load the reporting workspace and catalog, submit one built-in report and one active custom-definition report with valid filters and formats, and verify accepted asynchronous runs appear without unsupported catalog entries.

### Tests for User Story 1

- [ ] T023 [P] [US1] Add catalog service and request service mapper tests in `schoolmaster-frontend/tests/reporting-workspace/services/reportingCatalogRequestService.test.js`
- [ ] T024 [P] [US1] Add catalog composable tests for approved domains, fields, filters, formats, complexity limits, report-permission catalog access, report-definition-permission catalog access, and unavailable catalog state in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportCatalog.test.js`
- [ ] T025 [P] [US1] Add report request form composable tests for built-in requests, active custom-definition requests, validation, and unsupported-option blocking in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportRequestForm.test.js`
- [ ] T026 [P] [US1] Add catalog browser and request form component tests in `schoolmaster-frontend/tests/reporting-workspace/components/ReportCatalogRequestFlow.test.js`

### Implementation for User Story 1

- [ ] T027 [P] [US1] Implement catalog loading and catalog-valid option mapping in `schoolmaster-frontend/src/composables/reporting/useReportCatalog.js`
- [ ] T028 [P] [US1] Implement built-in and custom-definition request form state in `schoolmaster-frontend/src/composables/reporting/useReportRequestForm.js`
- [ ] T029 [P] [US1] Build approved catalog browser in `schoolmaster-frontend/src/components/reporting/ReportCatalogBrowser.vue`
- [ ] T030 [P] [US1] Build catalog-driven report request form in `schoolmaster-frontend/src/components/reporting/ReportRequestForm.vue`
- [ ] T031 [US1] Wire catalog browser and request form into `schoolmaster-frontend/src/pages/reporting/ReportingWorkspacePage.vue`
- [ ] T032 [US1] Add accepted asynchronous run feedback and history handoff behavior in `schoolmaster-frontend/src/composables/reporting/useReportRequestForm.js`
- [ ] T033 [US1] Add US1 catalog, request, validation, and unsupported-option text in `schoolmaster-frontend/src/i18n/reporting.js`

**Checkpoint**: User Story 1 is independently functional and testable as the MVP.

---

## Phase 4: User Story 2 - Review Report Run History and States (Priority: P2)

**Goal**: Authorized reporting users can open Report History by default, review report states, use documented filters, understand output and retention status, and receive visible plus polite state-change updates.

**Independent Test**: Open report history with requested, generating, generated, failed, canceled, expired-output, soft-deleted, built-in, custom, retried, empty, and filtered states; verify filters, pagination, retention, active school timezone timestamps, auto-refresh, manual refresh, announcements, and tenant-safe denial behavior.

### Tests for User Story 2

- [ ] T034 [P] [US2] Add report history service mapper tests for pagination, filters, include-deleted, retry lineage, soft-delete metadata, and output availability in `schoolmaster-frontend/tests/reporting-workspace/services/reportHistoryService.test.js`
- [ ] T035 [P] [US2] Add report history composable tests for default root, empty states, filters, pagination, manual refresh, and stale-response protection in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportHistory.test.js`
- [ ] T036 [P] [US2] Add auto-refresh and polite announcement tests for requested and generating runs in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportAutoRefresh.test.js`
- [ ] T037 [P] [US2] Add report history and detail component tests for state rendering and active school timezone timestamps in `schoolmaster-frontend/tests/reporting-workspace/components/ReportHistoryStates.test.js`

### Implementation for User Story 2

- [ ] T038 [P] [US2] Implement report history list state, filters, pagination, and empty-state mapping in `schoolmaster-frontend/src/composables/reporting/useReportHistory.js`
- [ ] T039 [P] [US2] Implement list-backed report run detail hydration and tenant-safe unavailable or not-found handling in `schoolmaster-frontend/src/composables/reporting/useReportRunDetail.js`
- [ ] T040 [P] [US2] Implement auto-refresh, manual refresh, and meaningful-change detection in `schoolmaster-frontend/src/composables/reporting/useReportAutoRefresh.js`
- [ ] T041 [P] [US2] Implement active school timezone timestamp formatting in `schoolmaster-frontend/src/composables/reporting/useReportingTimeFormatter.js`
- [ ] T042 [P] [US2] Build report history list, filters, pagination, and empty states in `schoolmaster-frontend/src/components/reporting/ReportHistoryList.vue`
- [ ] T043 [P] [US2] Build report run detail summary with status, lineage, soft-delete, retention, and output availability in `schoolmaster-frontend/src/components/reporting/ReportRunDetail.vue`
- [ ] T044 [US2] Set Report History as reporting workspace root default in `schoolmaster-frontend/src/router/index.js`
- [ ] T045 [US2] Connect auto-refresh announcements to visible history and detail state in `schoolmaster-frontend/src/pages/reporting/ReportingWorkspacePage.vue`
- [ ] T046 [US2] Add US2 history, state, empty, refresh, timezone, and announcement text in `schoolmaster-frontend/src/i18n/reporting.js`

**Checkpoint**: User Stories 1 and 2 work independently, and the reporting root opens Report History by default.

---

## Phase 5: User Story 3 - Download Available Report Outputs (Priority: P3)

**Goal**: Authorized users can download only generated, unexpired, same-school outputs in supported formats and see safe messaging for every unavailable output state.

**Independent Test**: Open generated report runs with PDF, CSV, and approved XLSX outputs in available, pending, failed, expired, unsupported, canceled, deleted, unauthorized, and cross-tenant states; download one available format and verify blocked messaging for all others.

### Tests for User Story 3

- [ ] T047 [P] [US3] Add download service tests for binary responses, format parameter mapping, expired-output, conflict, validation, not-found, unauthorized, and tenant-mismatch responses in `schoolmaster-frontend/tests/reporting-workspace/services/reportDownloadService.test.js`
- [ ] T048 [P] [US3] Add output availability composable tests for pending, available, failed, expired, unsupported, and download-unavailable gates in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportDownloads.test.js`
- [ ] T049 [P] [US3] Add download surface component tests for enabled controls, retention messaging, expired-output feedback, and no storage-path disclosure in `schoolmaster-frontend/tests/reporting-workspace/components/ReportDownloadSurface.test.js`

### Implementation for User Story 3

- [ ] T050 [P] [US3] Implement output availability and download state composable in `schoolmaster-frontend/src/composables/reporting/useReportDownloads.js`
- [ ] T051 [P] [US3] Implement binary download handling without content persistence in `schoolmaster-frontend/src/services/reporting/reportingDownloadAdapter.js`
- [ ] T052 [P] [US3] Build per-format output availability and download controls in `schoolmaster-frontend/src/components/reporting/ReportDownloadSurface.vue`
- [ ] T053 [US3] Integrate download surface into report detail in `schoolmaster-frontend/src/components/reporting/ReportRunDetail.vue`
- [ ] T054 [US3] Add US3 download, output availability, retention, expired-output, and blocked-download text in `schoolmaster-frontend/src/i18n/reporting.js`

**Checkpoint**: User Story 3 is independently testable from a generated report detail surface.

---

## Phase 6: User Story 4 - Manage Approved Report Lifecycle Actions (Priority: P4)

**Goal**: Authorized users can retry, cancel, soft-delete, and restore report runs only where permissions, state, contract eligibility, and approved reason codes allow.

**Independent Test**: Open retryable, cancellable, generated, failed, canceled, deleted, superseded, and cross-tenant report runs; verify visible actions, submit retry and cancel with approved reason codes, soft-delete and restore eligible runs, and confirm unsupported transitions show safe validation or conflict states.

### Tests for User Story 4

- [ ] T055 [P] [US4] Add lifecycle service tests for retry, cancel, delete, restore, approved reason codes, conflicts, validation, denied, not-found, and tenant-mismatch responses in `schoolmaster-frontend/tests/reporting-workspace/services/reportLifecycleService.test.js`
- [ ] T056 [P] [US4] Add lifecycle action composable tests for permission gates, state eligibility, stale dialog responses, and returned-state authority in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportLifecycleActions.test.js`
- [ ] T057 [P] [US4] Add lifecycle dialog component tests for retry, cancel, soft-delete, restore, no free-text reasons, and blocked unsupported actions in `schoolmaster-frontend/tests/reporting-workspace/components/ReportLifecycleActions.test.js`

### Implementation for User Story 4

- [ ] T058 [P] [US4] Implement lifecycle eligibility, reason-code options, pending state, and conflict mapping in `schoolmaster-frontend/src/composables/reporting/useReportLifecycleActions.js`
- [ ] T059 [P] [US4] Build retry and cancel dialogs with approved reason-code controls in `schoolmaster-frontend/src/components/reporting/ReportRetryCancelDialogs.vue`
- [ ] T060 [P] [US4] Build report-run soft-delete and restore controls in `schoolmaster-frontend/src/components/reporting/ReportRunLifecycleControls.vue`
- [ ] T061 [US4] Integrate lifecycle controls into report run detail in `schoolmaster-frontend/src/components/reporting/ReportRunDetail.vue`
- [ ] T062 [US4] Update report history state after returned retry, cancel, delete, and restore responses in `schoolmaster-frontend/src/composables/reporting/useReportHistory.js`
- [ ] T063 [US4] Add US4 lifecycle, reason-code, conflict, delete, restore, and unsupported-action text in `schoolmaster-frontend/src/i18n/reporting.js`

**Checkpoint**: User Story 4 is independently testable from report run detail and history surfaces.

---

## Phase 7: User Story 5 - Maintain Custom Report Definitions (Priority: P5)

**Goal**: Authorized report-definition users can list, create, update, activate, deactivate, delete, and restore school-owned custom report definitions while enforcing catalog limits, ownership, lifecycle states, and active metadata-only edit rules.

**Independent Test**: List definitions, create a draft from approved catalog entries, activate it, request a report from it, edit active metadata only, deactivate for structural edits, delete and restore to inactive, and verify duplicate-name and unsupported-field errors.

### Tests for User Story 5

- [ ] T064 [P] [US5] Add definition service tests for list, detail, create, update, delete, activate, deactivate, restore, duplicate-name, conflict, validation, not-found, denied, and tenant-mismatch responses in `schoolmaster-frontend/tests/reporting-workspace/services/reportDefinitionService.test.js`
- [ ] T065 [P] [US5] Add definition list composable tests for lifecycle filters, include-deleted, pagination, empty states, and stale-response protection in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportDefinitions.test.js`
- [ ] T066 [P] [US5] Add definition editor composable tests for catalog-approved controls, complexity limits, active metadata-only edits, restore-to-inactive, and blocked requests from non-active definitions in `schoolmaster-frontend/tests/reporting-workspace/composables/useReportDefinitionEditor.test.js`
- [ ] T067 [P] [US5] Add custom definition component tests for list, detail, editor, lifecycle actions, request-from-active flow, and safe feedback states in `schoolmaster-frontend/tests/reporting-workspace/components/ReportDefinitionsWorkspace.test.js`

### Implementation for User Story 5

- [ ] T068 [P] [US5] Implement custom definition list state, filters, pagination, and empty-state mapping in `schoolmaster-frontend/src/composables/reporting/useReportDefinitions.js`
- [ ] T069 [P] [US5] Implement catalog-bound definition editor state and complexity counters in `schoolmaster-frontend/src/composables/reporting/useReportDefinitionEditor.js`
- [ ] T070 [P] [US5] Implement definition lifecycle actions and restore-to-inactive mapping in `schoolmaster-frontend/src/composables/reporting/useReportDefinitionLifecycle.js`
- [ ] T071 [P] [US5] Build custom definition list and filters in `schoolmaster-frontend/src/components/reporting/ReportDefinitionsList.vue`
- [ ] T072 [P] [US5] Build custom definition detail and lifecycle controls in `schoolmaster-frontend/src/components/reporting/ReportDefinitionDetail.vue`
- [ ] T073 [P] [US5] Build catalog-bound custom definition editor in `schoolmaster-frontend/src/components/reporting/ReportDefinitionEditor.vue`
- [ ] T074 [US5] Wire custom definition routes and definition request handoff in `schoolmaster-frontend/src/pages/reporting/ReportingWorkspacePage.vue`
- [ ] T075 [US5] Add US5 custom definition, lifecycle, complexity-limit, active-edit-boundary, duplicate-name, and restore-to-inactive text in `schoolmaster-frontend/src/i18n/reporting.js`

**Checkpoint**: User Story 5 is independently testable from custom definition list/detail/editor surfaces.

---

## Phase 8: Polish & Cross-Cutting Concerns

**Purpose**: Full-flow verification, accessibility, security, performance, documentation evidence, and contract guardrails.

- [ ] T076 [P] Audit responsive layout and WCAG 2.1 AA behavior at 390px, 768px, and 1440px in `schoolmaster-frontend/src/pages/reporting/ReportingWorkspacePage.vue`
- [ ] T077 [P] Audit keyboard focus retention and polite announcement behavior across reporting dialogs and refresh updates in `schoolmaster-frontend/src/components/reporting/ReportingAnnouncementRegion.vue`
- [ ] T078 [P] Audit no-sensitive-data diagnostics, visible errors, filenames, and automated test fixtures in `schoolmaster-frontend/tests/reporting-workspace/`
- [ ] T079 [P] Audit blocked unsupported reporting actions are absent from reporting controls in `schoolmaster-frontend/src/components/reporting/`
- [ ] T080 [P] Audit Element Plus PascalCase usage and no direct Axios calls outside services in `schoolmaster-frontend/src/pages/reporting/`, `schoolmaster-frontend/src/components/reporting/`, and `schoolmaster-frontend/src/composables/reporting/`
- [ ] T081 Verify timed usability checks for SC-002, SC-003, SC-005 and SC-004 report-state identification accuracy, then record result in `specs/026-reporting-workspace-ui/quickstart.md`
- [ ] T082 Verify frontend performance goals for mocked render, protected route transition, report request feedback, download action, auto-refresh, and stale-response handling, then record result in `specs/026-reporting-workspace-ui/quickstart.md`
- [ ] T083 Run focused frontend unit tests and record result in `specs/026-reporting-workspace-ui/quickstart.md`
- [ ] T084 Run frontend build check and record result in `specs/026-reporting-workspace-ui/quickstart.md`
- [ ] T085 Run OpenAPI lint only if `api/openapi.yaml` changed and record result or not-run reason in `specs/026-reporting-workspace-ui/quickstart.md`
- [ ] T086 Update implementation acceptance evidence in `specs/026-reporting-workspace-ui/quickstart.md`
- [ ] T087 Verify all reporting tasks remain traceable to approved contracts in `specs/026-reporting-workspace-ui/contracts/reporting-workspace-ui-contract.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies. Complete before shared foundation.
- **Foundational (Phase 2)**: Depends on Setup. Blocks all user stories.
- **User Stories (Phases 3-7)**: Depend on Foundational completion.
- **Polish (Phase 8)**: Depends on all implemented user stories.

### User Story Dependencies

- **US1 (P1)**: Can start after Foundational. MVP scope.
- **US2 (P2)**: Can start after Foundational and may integrate accepted runs from US1, but history can be tested with fixtures independently.
- **US3 (P3)**: Can start after Foundational and can be tested from fixture-backed generated report details.
- **US4 (P4)**: Can start after Foundational and can be tested from fixture-backed report detail states.
- **US5 (P5)**: Can start after Foundational and can be tested with catalog and definition fixtures independently.

### Within Each User Story

- Write service, composable, and component tests before implementation.
- Implement service and contract mappers before composables.
- Implement composables before route/page integration.
- Keep route views thin and keep Axios calls inside reporting services only.
- Preserve active school, permission, tenant-safe denial, stale-response, and safe-diagnostics behavior in every story.

## Parallel Opportunities

- Setup tasks T003, T004, T005, and T006 can run in parallel after T001 and T002 context review.
- Foundation tests T007 through T011 can run in parallel.
- Foundation implementation tasks T019, T020, and T021 can run in parallel after T012 through T018 define shared contracts and shell.
- Tests inside each user story can run in parallel because they target separate service, composable, and component files.
- US2, US3, US4, and US5 can be implemented by separate owners after Foundation if fixture-backed integration points are stable.

## Parallel Example: User Story 1

```bash
# Launch US1 test tasks together:
Task: "T023 [US1] Add catalog service and request service mapper tests in schoolmaster-frontend/tests/reporting-workspace/services/reportingCatalogRequestService.test.js"
Task: "T024 [US1] Add catalog composable tests in schoolmaster-frontend/tests/reporting-workspace/composables/useReportCatalog.test.js"
Task: "T025 [US1] Add report request form composable tests in schoolmaster-frontend/tests/reporting-workspace/composables/useReportRequestForm.test.js"
Task: "T026 [US1] Add catalog browser and request form component tests in schoolmaster-frontend/tests/reporting-workspace/components/ReportCatalogRequestFlow.test.js"
```

## Parallel Example: User Story 2

```bash
# Launch US2 independent implementation tasks after tests:
Task: "T038 [US2] Implement report history list state in schoolmaster-frontend/src/composables/reporting/useReportHistory.js"
Task: "T039 [US2] Implement report run detail loading in schoolmaster-frontend/src/composables/reporting/useReportRunDetail.js"
Task: "T040 [US2] Implement auto-refresh in schoolmaster-frontend/src/composables/reporting/useReportAutoRefresh.js"
Task: "T041 [US2] Implement timezone formatting in schoolmaster-frontend/src/composables/reporting/useReportingTimeFormatter.js"
```

## Parallel Example: User Story 5

```bash
# Launch US5 implementation tasks after service/composable contracts are stable:
Task: "T068 [US5] Implement custom definition list state in schoolmaster-frontend/src/composables/reporting/useReportDefinitions.js"
Task: "T069 [US5] Implement catalog-bound definition editor state in schoolmaster-frontend/src/composables/reporting/useReportDefinitionEditor.js"
Task: "T070 [US5] Implement definition lifecycle actions in schoolmaster-frontend/src/composables/reporting/useReportDefinitionLifecycle.js"
Task: "T071 [US5] Build custom definition list in schoolmaster-frontend/src/components/reporting/ReportDefinitionsList.vue"
Task: "T072 [US5] Build custom definition detail in schoolmaster-frontend/src/components/reporting/ReportDefinitionDetail.vue"
Task: "T073 [US5] Build custom definition editor in schoolmaster-frontend/src/components/reporting/ReportDefinitionEditor.vue"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Deliver Phase 3 User Story 1 as the MVP.
3. Validate that catalog browsing and report request submission use only approved contracts and blocked unsupported actions do not appear.

### Incremental Delivery

1. Add User Story 2 for default Report History, state rendering, timezone timestamps, refresh, and announcements.
2. Add User Story 3 for output availability and downloads.
3. Add User Story 4 for approved report-run lifecycle actions.
4. Add User Story 5 for custom report definitions.
5. Complete Phase 8 polish, evidence, and validation.

### Team Strategy

1. One owner completes service contracts and feedback mapping in Foundation.
2. One owner builds history/detail and refresh behavior after Foundation.
3. One owner builds custom definitions after catalog and definition fixtures stabilize.
4. QA or another frontend owner runs cross-story accessibility, diagnostics, and blocked-action audits in Phase 8.
