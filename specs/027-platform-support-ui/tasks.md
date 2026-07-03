# Tasks: Platform Support Access and Cross-School Oversight UI

**Input**: Design documents from `specs/027-platform-support-ui/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/platform-support-ui-contract.md`, `quickstart.md`

**Tests**: Frontend behavior changes require Vitest service, composable, route/component integration, stale-response, and safe-diagnostics coverage. OpenAPI validation is required because frontend implementation is blocked until platform/support operations are promoted into `api/openapi.yaml` and the platform contract mirror. Backend implementation is out of scope unless a separate backend/API feature approves missing behavior.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared platform-support foundations are complete.

**Repository Scope**: `schoolmaster-specs` owns this task list, OpenAPI readiness, contract mirror checks, and acceptance evidence. `schoolmaster-frontend` is the lead implementation repository for Vue 3 SPA routes, services, composables, components, and Vitest coverage. `schoolmaster-backend` remains out of scope except for separate backend/API work if required platform/support contracts from `013-platform-support-access` are missing or not contract-compliant.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or can be completed without depending on another incomplete task.
- **[Story]**: User story label for story phases only.
- Every task includes an exact file path.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm contract readiness, target frontend structure, and shared implementation locations before story work begins.

- [ ] T001 Review platform support UI scope and blocked actions in `specs/027-platform-support-ui/spec.md`
- [ ] T002 Verify platform/support operation IDs in `api/openapi.yaml` and record coverage or gaps in `specs/027-platform-support-ui/quickstart.md`
- [ ] T003 Verify platform/support operation parity in `specs/001-schoolmaster-platform/contracts/openapi.yaml` and record coverage or gaps in `specs/027-platform-support-ui/quickstart.md`
- [ ] T004 [P] Confirm frontend platform-support target folders in `src/pages/platform-support/`, `src/components/platform-support/`, `src/composables/platform-support/`, `src/services/platform-support/`, and `src/contracts/platform-support/`
- [ ] T005 [P] Confirm frontend test target folders in `tests/platform-support/fixtures/`, `tests/platform-support/services/`, `tests/platform-support/composables/`, and `tests/platform-support/components/`
- [ ] T006 [P] Add platform-support i18n namespace placeholder in `src/i18n/modules/platform-support.js`
- [ ] T007 [P] Confirm platform-support route naming and navigation placement against existing protected-shell routes in `src/router/modules/platform-support.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared contract mapping, access gates, feedback states, stale-response guards, and workspace shell required by all platform support stories.

**Critical**: No user story work can begin until this phase is complete.

### Contract and Readiness Tasks

- [ ] T008 Add or verify platform school summary OpenAPI coverage for `listPlatformSchoolSummaries` in `api/openapi.yaml`
- [ ] T009 Add or verify platform reporting overview OpenAPI coverage for `getPlatformReportingOverview` in `api/openapi.yaml`
- [ ] T010 Add or verify support access decision OpenAPI coverage for `requestSupportAccess` and `getSupportAccessDecision` in `api/openapi.yaml`
- [ ] T011 Add or verify support approval and revocation OpenAPI coverage for `approveSupportAccess` and `revokeSupportAccess` in `api/openapi.yaml`
- [ ] T012 Add or verify support diagnostics OpenAPI coverage for `getSupportSchoolDiagnostics` in `api/openapi.yaml`
- [ ] T013 Add or verify support audit OpenAPI coverage for `listSupportAuditEvents` in `api/openapi.yaml`
- [ ] T014 Mirror verified platform/support operation coverage in `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T015 Run Redocly validation for aggregate and platform contracts and record result in `specs/027-platform-support-ui/quickstart.md`

### Tests for Foundation

- [ ] T016 [P] Add platform support fixture builders for summaries, reporting overview, decisions, approvals, diagnostics, audits, and error envelopes in `tests/platform-support/fixtures/platformSupportFixtures.js`
- [ ] T017 [P] Add platform support service mapper tests for approved operations, contract-unavailable behavior, undocumented-field dropping, and no school-admin opt-in service functions in `tests/platform-support/services/platformSupportService.test.js`
- [ ] T018 [P] Add platform support access-gate composable tests for authenticated active actor, contract-derived permission identifiers, platform oversight, platform reporting, support access, support approval/revocation, support drill-down, and audit permissions in `tests/platform-support/composables/usePlatformSupportAccess.test.js`
- [ ] T019 [P] Add stale-response and safe-diagnostics tests in `tests/platform-support/composables/usePlatformSupportRequestGuards.test.js`
- [ ] T020 [P] Add shared platform support feedback and status component tests in `tests/platform-support/components/PlatformSupportSharedStates.test.js`

### Implementation for Foundation

- [ ] T021 Create platform support contract mappers and constants in `src/contracts/platform-support/platformSupportContract.js`
- [ ] T022 Implement approved platform support HTTP service functions in `src/services/platform-support/platformSupportService.js`
- [ ] T023 Implement platform support error-to-feedback normalization in `src/services/platform-support/platformSupportErrorMapper.js`
- [ ] T024 Implement platform support access gate composable with centralized contract-derived permission identifiers in `src/composables/platform-support/usePlatformSupportAccess.js`
- [ ] T025 Implement platform support stale-response and safe-diagnostics guards in `src/composables/platform-support/usePlatformSupportRequestGuards.js`
- [ ] T026 Implement protected platform support workspace route shell in `src/pages/platform-support/PlatformSupportWorkspacePage.vue`
- [ ] T027 Wire platform support root, oversight, decisions, diagnostics, and audit child routes in `src/router/modules/platform-support.js`
- [ ] T028 [P] Implement shared platform support feedback, empty, denied, unavailable, conflict, expired, revoked, and stale states in `src/components/platform-support/PlatformSupportFeedbackState.vue`
- [ ] T029 [P] Implement shared decision, approval, opt-in, suppression, and redaction status badges in `src/components/platform-support/PlatformSupportStatusBadges.vue`
- [ ] T030 [P] Implement shared refresh control and safe status update region in `src/components/platform-support/PlatformSupportRefreshStatus.vue`
- [ ] T031 Add shared platform support display text for foundation states in `src/i18n/modules/platform-support.js`

**Checkpoint**: Foundation ready. User stories can now proceed in priority order or in parallel with separate owners.

---

## Phase 3: User Story 1 - Review Platform Operational Oversight (Priority: P1) MVP

**Goal**: Authorized platform administrators can open Platform Operational Oversight by default, review minimized school summaries and reporting-health indicators, and see suppression, empty, unavailable, and denied states without school-owned detail records.

**Independent Test**: Sign in with platform-wide operational oversight and reporting overview permissions, open the platform support workspace root, load platform school summaries and reporting overview data, and verify minimized fields, suppressed-count indicators, loading states, empty states, denied states, and tenant-safe errors.

### Tests for User Story 1

- [ ] T032 [P] [US1] Add platform school summary service mapper tests for pagination, filters, sorting, minimized fields, suppression markers, denied states, and no detail records in `tests/platform-support/services/platformSchoolSummaryService.test.js`
- [ ] T033 [P] [US1] Add platform reporting overview service mapper tests for aggregate reporting health, lifecycle, retention, output availability, failure-summary indicators, and raw-output exclusion in `tests/platform-support/services/platformReportingOverviewService.test.js`
- [ ] T034 [P] [US1] Add platform oversight composable tests for default route, permission gates, empty states, suppressed-count rendering, unavailable groups, and stale responses in `tests/platform-support/composables/usePlatformOperationalOversight.test.js`
- [ ] T035 [P] [US1] Add Platform Operational Oversight component tests for summary list, reporting overview, suppression/redaction badges, no-detail disclosure, and denied states in `tests/platform-support/components/PlatformOperationalOversight.test.js`

### Implementation for User Story 1

- [ ] T036 [P] [US1] Implement platform school summary loading, filter state, pagination, and suppression mapping in `src/composables/platform-support/usePlatformSchoolSummaries.js`
- [ ] T037 [P] [US1] Implement platform reporting overview loading and aggregate summary mapping in `src/composables/platform-support/usePlatformReportingOverview.js`
- [ ] T038 [P] [US1] Build minimized platform school summary list in `src/components/platform-support/PlatformSchoolSummaryList.vue`
- [ ] T039 [P] [US1] Build cross-school reporting overview panel in `src/components/platform-support/PlatformReportingOverviewPanel.vue`
- [ ] T040 [US1] Build Platform Operational Oversight page composition in `src/pages/platform-support/PlatformOperationalOversightPage.vue`
- [ ] T041 [US1] Set Platform Operational Oversight as platform support workspace root default in `src/router/modules/platform-support.js`
- [ ] T042 [US1] Add US1 oversight, suppression, reporting overview, empty, unavailable, and denied text in `src/i18n/modules/platform-support.js`

**Checkpoint**: User Story 1 is independently functional and testable as the MVP.

---

## Phase 4: User Story 2 - Manage Support Access Decisions and Approval States (Priority: P2)

**Goal**: Platform support users can request or view target-school support access decisions, understand display-only opt-in and internal approval state, see 24-hour expiry, and use platform approval/revocation controls only where permitted.

**Independent Test**: Sign in as a support user with support access permission, create or view support access decisions across requested, pending, approved, denied, expired, revoked, stale, and mismatched states, and verify diagnostics remain unavailable until active target-school opt-in and internal platform approval are both valid.

### Tests for User Story 2

- [ ] T043 [P] [US2] Add support access decision service tests for request, detail, requested, pending, approved, denied, expired, revoked, stale, mismatched, validation, conflict, and denied responses in `tests/platform-support/services/supportAccessDecisionService.test.js`
- [ ] T044 [P] [US2] Add support approval service tests for approve, revoke, permission denial, stale state, expired decision, mismatched target, and returned-state authority in `tests/platform-support/services/supportApprovalService.test.js`
- [ ] T045 [P] [US2] Add support decision composable tests for target school, reason code, purpose, correlation metadata, display-only opt-in state, internal approval state, expiry, and diagnostics gating in `tests/platform-support/composables/useSupportAccessDecision.test.js`
- [ ] T046 [P] [US2] Add approval action composable tests for permission gates, confirmation state, conflict handling, stale responses, and no school-owned writes in `tests/platform-support/composables/useSupportApprovalActions.test.js`
- [ ] T047 [P] [US2] Add support decision component tests for request form, decision detail, approval controls, opt-in state panel, expiry callouts, and no school-admin opt-in controls in `tests/platform-support/components/SupportAccessDecisionWorkspace.test.js`

### Implementation for User Story 2

- [ ] T048 [P] [US2] Implement support access request and decision state composable in `src/composables/platform-support/useSupportAccessDecision.js`
- [ ] T049 [P] [US2] Implement support approval and revocation action composable in `src/composables/platform-support/useSupportApprovalActions.js`
- [ ] T050 [P] [US2] Build support access request form with target school, reason code, purpose, and correlation metadata controls in `src/components/platform-support/SupportAccessRequestForm.vue`
- [ ] T051 [P] [US2] Build support decision detail and approval-gate summary in `src/components/platform-support/SupportAccessDecisionDetail.vue`
- [ ] T052 [P] [US2] Build display-only target-school opt-in state panel in `src/components/platform-support/SupportOptInStatePanel.vue`
- [ ] T053 [P] [US2] Build platform approval and revocation controls in `src/components/platform-support/SupportApprovalControls.vue`
- [ ] T054 [US2] Build Support Access Decisions page composition in `src/pages/platform-support/SupportAccessDecisionsPage.vue`
- [ ] T055 [US2] Add US2 decision, approval, opt-in, expiry, revocation, conflict, and denied text in `src/i18n/modules/platform-support.js`

**Checkpoint**: User Story 2 is independently testable from support decision request/detail surfaces.

---

## Phase 5: User Story 3 - Perform Redacted Support Drill-Down (Priority: P3)

**Goal**: Platform support users with valid approval gates can open redacted read-only diagnostics for one target school, while diagnostics disappear after expiry, revocation, denial, stale, mismatched, unavailable, or no-longer-authorized state.

**Independent Test**: Open diagnostics for an approved target school and verify only approved redacted diagnostics, suppression indicators, decision context, and expiry appear; then expire, revoke, deny, mismatch, or invalidate access and verify automatic/manual refresh removes diagnostics safely.

### Tests for User Story 3

- [ ] T056 [P] [US3] Add support diagnostics service tests for redacted payloads, suppression markers, private metadata exclusion, generated-report download denial, raw-output exclusion, and unavailable diagnostics in `tests/platform-support/services/supportDiagnosticsService.test.js`
- [ ] T057 [P] [US3] Add diagnostics composable tests for support drill-down gates, target-school match, active decision context, expiry, redaction, unsupported action blocking, and safe feedback in `tests/platform-support/composables/useSupportDiagnostics.test.js`
- [ ] T058 [P] [US3] Add support refresh composable tests for automatic refresh, manual refresh, response cleanup, diagnostics removal, stale responses, and focus-preserving visible updates in `tests/platform-support/composables/useSupportDecisionRefresh.test.js`
- [ ] T059 [P] [US3] Add diagnostics component tests for redacted field rendering, suppressed counts, active decision context, blocked unsupported actions, and no sensitive output in `tests/platform-support/components/SupportDiagnosticsView.test.js`

### Implementation for User Story 3

- [ ] T060 [P] [US3] Implement support diagnostics loading and redaction-state mapping in `src/composables/platform-support/useSupportDiagnostics.js`
- [ ] T061 [P] [US3] Implement automatic and manual refresh for visible decisions and diagnostics in `src/composables/platform-support/useSupportDecisionRefresh.js`
- [ ] T062 [P] [US3] Build redacted diagnostic groups component in `src/components/platform-support/SupportDiagnosticGroups.vue`
- [ ] T063 [P] [US3] Build diagnostics access gate and active decision context panel in `src/components/platform-support/SupportDiagnosticsGate.vue`
- [ ] T064 [P] [US3] Build unsupported support action blocker in `src/components/platform-support/SupportUnsupportedActionsGuard.vue`
- [ ] T065 [US3] Build Support Diagnostics page composition in `src/pages/platform-support/SupportDiagnosticsPage.vue`
- [ ] T066 [US3] Connect refresh-driven diagnostics access removal to Support Diagnostics page in `src/pages/platform-support/SupportDiagnosticsPage.vue`
- [ ] T067 [US3] Add US3 diagnostics, redaction, suppression, refresh, expiry, revocation, unsupported-action, and safe feedback text in `src/i18n/modules/platform-support.js`

**Checkpoint**: User Story 3 is independently testable from an approved support diagnostics route using fixture-backed decision and diagnostics responses.

---

## Phase 6: User Story 4 - Review Support Audit Activity (Priority: P4)

**Goal**: Authorized support audit reviewers can list minimized support audit events with documented filters and pagination without exposing private payloads, unauthorized target details, or support activity to actors without audit permission.

**Independent Test**: Sign in as an actor with support audit review permission, list audit events with filters and pagination, and verify each event shows only actor, action, outcome, safe target school, correlation ID, reason code, timestamp, and minimized metadata.

### Tests for User Story 4

- [ ] T068 [P] [US4] Add support audit service tests for pagination, filters, minimized event fields, denied cross-tenant target omission, forbidden responses, and private metadata exclusion in `tests/platform-support/services/supportAuditService.test.js`
- [ ] T069 [P] [US4] Add support audit composable tests for filters, pagination, empty/no-filter-results states, denied state, stale responses, and safe metadata mapping in `tests/platform-support/composables/useSupportAuditReview.test.js`
- [ ] T070 [P] [US4] Add support audit component tests for event list, filters, minimized metadata, denied state, empty state, and no private payload disclosure in `tests/platform-support/components/SupportAuditReview.test.js`

### Implementation for User Story 4

- [ ] T071 [P] [US4] Implement support audit review list state, filters, pagination, and safe metadata mapping in `src/composables/platform-support/useSupportAuditReview.js`
- [ ] T072 [P] [US4] Build support audit filters and pagination controls in `src/components/platform-support/SupportAuditFilters.vue`
- [ ] T073 [P] [US4] Build minimized support audit event list in `src/components/platform-support/SupportAuditEventList.vue`
- [ ] T074 [US4] Build Support Audit Review page composition in `src/pages/platform-support/SupportAuditReviewPage.vue`
- [ ] T075 [US4] Add US4 audit filters, minimized metadata, empty states, denied states, and safe target text in `src/i18n/modules/platform-support.js`

**Checkpoint**: User Story 4 is independently testable from support audit review route and fixtures.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Full-flow verification, accessibility, security, performance, documentation evidence, and contract guardrails.

- [ ] T076 [P] Audit responsive layout and WCAG 2.1 AA behavior at 390px, 768px, and 1440px in `src/pages/platform-support/PlatformSupportWorkspacePage.vue`
- [ ] T077 [P] Audit keyboard focus retention during refresh, approval, revocation, diagnostics, and audit interactions in `src/components/platform-support/PlatformSupportRefreshStatus.vue`
- [ ] T078 [P] Audit no-sensitive-data diagnostics, visible errors, route labels, filenames, and automated test fixtures in `tests/platform-support/`
- [ ] T079 [P] Audit blocked unsupported platform/support actions are absent from platform support controls in `src/components/platform-support/`
- [ ] T080 [P] Audit Element Plus PascalCase usage and no direct Axios calls outside services in `src/pages/platform-support/`, `src/components/platform-support/`, and `src/composables/platform-support/`
- [ ] T081 Verify timed usability checks for SC-002, SC-003, SC-004, SC-010, and SC-013, then record result in `specs/027-platform-support-ui/quickstart.md`
- [ ] T082 Verify frontend performance goals for mocked render, protected route transition, decision submission, approval/revocation, refresh behavior, and stale-response handling, then record result in `specs/027-platform-support-ui/quickstart.md`
- [ ] T083 Run focused frontend unit tests and record result in `specs/027-platform-support-ui/quickstart.md`
- [ ] T084 Run frontend build check and record result in `specs/027-platform-support-ui/quickstart.md`
- [ ] T085 Run OpenAPI lint for platform/support contract coverage and record result in `specs/027-platform-support-ui/quickstart.md`
- [ ] T086 Update implementation acceptance evidence in `specs/027-platform-support-ui/quickstart.md`
- [ ] T087 Verify all platform support tasks remain traceable to approved contracts in `specs/027-platform-support-ui/contracts/platform-support-ui-contract.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies. Complete before shared foundation.
- **Foundational (Phase 2)**: Depends on Setup. Blocks all user stories.
- **User Stories (Phases 3-6)**: Depend on Foundational completion.
- **Polish (Phase 7)**: Depends on all implemented user stories.

### User Story Dependencies

- **US1 (P1)**: Can start after Foundational. MVP scope.
- **US2 (P2)**: Can start after Foundational and can be tested with fixture-backed decisions independently of US1.
- **US3 (P3)**: Can start after Foundational and can be tested with fixture-backed decisions and diagnostics; integrates naturally with US2 decision state.
- **US4 (P4)**: Can start after Foundational and can be tested with audit fixtures independently of US2 and US3.

### Within Each User Story

- Write service, composable, and component tests before implementation.
- Implement service and contract mappers before composables.
- Implement composables before route/page integration.
- Keep route views thin and keep Axios calls inside platform support services only.
- Preserve platform permission gates, tenant-safe denial, stale-response, redaction, suppression, and safe-diagnostics behavior in every story.

## Parallel Opportunities

- Setup tasks T004, T005, T006, and T007 can run in parallel after T001 through T003 context review.
- Foundation contract tasks T008 through T013 can run in parallel before T014 and T015.
- Foundation tests T016 through T020 can run in parallel.
- Foundation implementation tasks T028, T029, and T030 can run in parallel after T021 through T027 define shared contracts and shell.
- Tests inside each user story can run in parallel because they target separate service, composable, and component files.
- US2, US3, and US4 can be implemented by separate owners after Foundation if fixture-backed integration points are stable.

## Parallel Example: User Story 1

```bash
# Launch US1 test tasks together:
Task: "T032 [US1] Add platform school summary service mapper tests in tests/platform-support/services/platformSchoolSummaryService.test.js"
Task: "T033 [US1] Add platform reporting overview service mapper tests in tests/platform-support/services/platformReportingOverviewService.test.js"
Task: "T034 [US1] Add platform oversight composable tests in tests/platform-support/composables/usePlatformOperationalOversight.test.js"
Task: "T035 [US1] Add Platform Operational Oversight component tests in tests/platform-support/components/PlatformOperationalOversight.test.js"
```

## Parallel Example: User Story 2

```bash
# Launch US2 independent implementation tasks after tests:
Task: "T048 [US2] Implement support access request and decision state composable in src/composables/platform-support/useSupportAccessDecision.js"
Task: "T049 [US2] Implement support approval and revocation action composable in src/composables/platform-support/useSupportApprovalActions.js"
Task: "T050 [US2] Build support access request form in src/components/platform-support/SupportAccessRequestForm.vue"
Task: "T051 [US2] Build support decision detail in src/components/platform-support/SupportAccessDecisionDetail.vue"
Task: "T052 [US2] Build display-only target-school opt-in state panel in src/components/platform-support/SupportOptInStatePanel.vue"
Task: "T053 [US2] Build platform approval and revocation controls in src/components/platform-support/SupportApprovalControls.vue"
```

## Parallel Example: User Story 3

```bash
# Launch US3 refresh and diagnostics implementation tasks after tests:
Task: "T060 [US3] Implement support diagnostics loading in src/composables/platform-support/useSupportDiagnostics.js"
Task: "T061 [US3] Implement automatic and manual refresh in src/composables/platform-support/useSupportDecisionRefresh.js"
Task: "T062 [US3] Build redacted diagnostic groups in src/components/platform-support/SupportDiagnosticGroups.vue"
Task: "T063 [US3] Build diagnostics access gate in src/components/platform-support/SupportDiagnosticsGate.vue"
Task: "T064 [US3] Build unsupported action blocker in src/components/platform-support/SupportUnsupportedActionsGuard.vue"
```

## Parallel Example: User Story 4

```bash
# Launch US4 test tasks together:
Task: "T068 [US4] Add support audit service tests in tests/platform-support/services/supportAuditService.test.js"
Task: "T069 [US4] Add support audit composable tests in tests/platform-support/composables/useSupportAuditReview.test.js"
Task: "T070 [US4] Add support audit component tests in tests/platform-support/components/SupportAuditReview.test.js"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Deliver Phase 3 User Story 1 as the MVP.
3. Validate that platform summary and reporting overview surfaces use only approved contracts and preserve minimization, redaction, suppression, and denied states.

### Incremental Delivery

1. Add User Story 2 for support decision, approval state, display-only opt-in state, and platform approval/revocation controls.
2. Add User Story 3 for redacted diagnostics and automatic/manual refresh.
3. Add User Story 4 for support audit review.
4. Complete Phase 7 polish, evidence, and validation.

### Team Strategy

1. One owner completes OpenAPI readiness and service contract mapping in Foundation.
2. One owner builds platform summaries and reporting overview as MVP.
3. One owner builds support decision and approval state after Foundation.
4. One owner builds diagnostics refresh after decision fixtures stabilize.
5. QA or another frontend owner runs cross-story accessibility, diagnostics, and blocked-action audits in Phase 7.
