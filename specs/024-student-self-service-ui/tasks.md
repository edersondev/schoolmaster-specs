# Tasks: Student Self-Service UI

**Input**: Design documents from `specs/024-student-self-service-ui/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/student-self-service-ui-contract.md](contracts/student-self-service-ui-contract.md), [quickstart.md](quickstart.md)

**Tests**: Required by FR-021 and quickstart verification. Write focused Vitest service, composable, route, and component tests before implementation tasks in each story.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after setup and foundation.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish frontend module boundaries, route/i18n scaffolds, and contract traceability before shared implementation starts.

- [ ] T001 Confirm approved student self-service operation IDs and blocked contract gaps in `specs/024-student-self-service-ui/contracts/student-self-service-ui-contract.md`
- [ ] T002 Create student feature folders in `src/pages/student/`, `src/components/student/`, `src/composables/student/`, `src/services/student/`, and `src/contracts/student/`
- [ ] T003 [P] Create focused test folders in `tests/student-self-service/services/`, `tests/student-self-service/composables/`, `tests/student-self-service/components/`, and `tests/student-self-service/routes/`
- [ ] T004 [P] Add the student self-service i18n namespace scaffold in `src/i18n/modules/studentSelfService.js`
- [ ] T005 [P] Add the student route module scaffold in `src/router/modules/student.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared contract, context, feedback, stale-response, and diagnostics infrastructure required by every story.

**Critical**: No user story work starts until this phase is complete.

- [ ] T006 [P] Add foundation tests for unauthenticated access, session-expired handling, active school, linked profile, current period, default route, feedback mapping, stale guard, and diagnostics redaction in `tests/student-self-service/composables/studentSelfServiceFoundation.spec.js`
- [ ] T007 Create the approved operation and blocked-capability map in `src/contracts/student/studentSelfServiceContract.js`
- [ ] T008 Create shared response mappers for paginated envelopes, learning-set entries, content metadata, grade records, attendance records, and safe dropped fields in `src/contracts/student/studentSelfServiceMappers.js`
- [ ] T009 Create the student self-service Axios wrapper and exported service methods in `src/services/student/studentSelfServiceService.js`
- [ ] T010 [P] Create shared feedback-state normalization for unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-student-profile, no-current-period, unavailable-content, validation, not-found, unsupported-page-size, stale-response, and temporary-unavailable states in `src/services/student/studentSelfServiceFeedbackMapper.js`
- [ ] T011 [P] Create safe diagnostics redaction for private file paths, storage keys, tokens, role internals, scan internals, guardian data, other-student data, and cross-tenant details in `src/services/student/studentSelfServiceDiagnostics.js`
- [ ] T012 Create the stale-response guard composable for route, pagination, current period, active school, authentication, and session changes in `src/composables/student/useStudentSelfServiceStaleGuard.js`
- [ ] T013 Create the student workspace context composable for active school, active linked student profile, current active academic period, default landing, and blocking gates in `src/composables/student/useStudentWorkspaceContext.js`
- [ ] T014 [P] Create shared student feedback-state component for loading, empty, denied, unavailable, no-active-school, no-student-profile, no-current-period, not-found, and temporary-unavailable states in `src/components/student/StudentFeedbackState.vue`
- [ ] T015 [P] Create shared student status and pagination components for status badges, content availability, and paginated list controls in `src/components/student/StudentStatusControls.vue`
- [ ] T016 Wire the student route module into the application router in `src/router/index.js`
- [ ] T017 Add shared student workspace and feedback display text to `src/i18n/modules/studentSelfService.js`

**Checkpoint**: Foundation ready. User story implementation can begin.

---

## Phase 3: User Story 1 - Review Assigned Learning Sets (Priority: P1) MVP

**Goal**: Student workspace opens Assigned Learning Sets by default and shows current active-period assigned learning sets with loaded-list-backed detail, read-only questionnaire entries, and safe no-context/empty states.

**Independent Test**: Sign in as a student with active school, active linked profile, and current active academic period; open student workspace root; verify Assigned Learning Sets loads own assigned learning sets, ordered entries, read-only questionnaire entries, empty state, no-current-period state, and no-student-profile gate without other-student data.

### Tests for User Story 1

- [ ] T018 [P] [US1] Add service and mapper tests for `listStudentLearningSets`, required `academic_period_id`, pagination, dropped fields, read-only questionnaire entries, and no undocumented parameters in `tests/student-self-service/services/assignedLearningSetsService.spec.js`
- [ ] T019 [P] [US1] Add composable tests for default Assigned Learning Sets landing, current-period gate, pagination, empty state, loaded-list-backed detail, no-current-period, no-student-profile, and stale-response handling in `tests/student-self-service/composables/useAssignedLearningSets.spec.js`
- [ ] T020 [P] [US1] Add route/component tests for workspace root, assigned learning-set list, ordered entries, read-only questionnaire entries, no-active-school, no-current-period, no-student-profile, and not-found states in `tests/student-self-service/components/assignedLearningSetViews.spec.js`

### Implementation for User Story 1

- [ ] T021 [US1] Implement assigned learning-set service methods and request/response mappers in `src/services/student/studentSelfServiceService.js`
- [ ] T022 [US1] Implement assigned learning-set state, pagination, current-period gate, loaded-list detail lookup, empty state, stale guard, and feedback mapping in `src/composables/student/useAssignedLearningSets.js`
- [ ] T023 [P] [US1] Implement assigned learning-set list component with status, pagination, empty, loading, and denied states in `src/components/student/AssignedLearningSetList.vue`
- [ ] T024 [P] [US1] Implement learning-set entry list component with content entries, read-only questionnaire entries, sequence display, and safe availability labels in `src/components/student/StudentLearningSetEntryList.vue`
- [ ] T025 [P] [US1] Implement assigned learning-set detail route that uses loaded timeline data only and never calls standalone detail endpoints in `src/pages/student/StudentLearningSetDetailView.vue`
- [ ] T026 [US1] Implement Assigned Learning Sets route view with default landing behavior and context gates in `src/pages/student/AssignedLearningSetsView.vue`
- [ ] T027 [US1] Register workspace root, assigned learning-set list, and loaded-detail routes in `src/router/modules/student.js`
- [ ] T028 [US1] Add Assigned Learning Sets, questionnaire read-only, empty, no-current-period, no-student-profile, and not-found text in `src/i18n/modules/studentSelfService.js`

**Checkpoint**: User Story 1 is independently functional and testable as MVP.

---

## Phase 4: User Story 2 - Access Authorized Content (Priority: P2)

**Goal**: Students can download only assigned content that returned metadata marks as available, with safe unavailable-content and denial handling.

**Independent Test**: Open an assigned learning set with clean downloadable content, download it, then attempt unavailable, pending, failed, inactive, unassigned, cross-school, missing, and denied content states and verify safe feedback with no private file details.

### Tests for User Story 2

- [ ] T029 [P] [US2] Add content download service tests for `downloadStudentTeacherContent`, `download_available` gate, binary response handling, unavailable-content mapping, not-found mapping, and no private path persistence in `tests/student-self-service/services/studentContentDownloadService.spec.js`
- [ ] T030 [P] [US2] Add content download composable tests for pending state, success state, unavailable-content state, denied responses, stale route changes, and diagnostics redaction in `tests/student-self-service/composables/useStudentContentDownload.spec.js`
- [ ] T031 [P] [US2] Add content action component tests for enabled download, disabled unavailable content, pending/failed scan labels, not-found feedback, and safe visible errors in `tests/student-self-service/components/studentContentDownloadViews.spec.js`

### Implementation for User Story 2

- [ ] T032 [US2] Implement content download service method and safe binary handling in `src/services/student/studentSelfServiceService.js`
- [ ] T033 [US2] Implement content download state, `download_available` gate, stale guard, feedback mapping, and diagnostics redaction in `src/composables/student/useStudentContentDownload.js`
- [ ] T034 [P] [US2] Implement student content availability and download action component in `src/components/student/StudentContentDownloadAction.vue`
- [ ] T035 [US2] Integrate content download actions into learning-set entry rendering in `src/components/student/StudentLearningSetEntryList.vue`
- [ ] T036 [US2] Add content download, unavailable-content, pending scan, failed scan, denied, and not-found display text in `src/i18n/modules/studentSelfService.js`

**Checkpoint**: User Story 2 is independently functional and download-gated.

---

## Phase 5: User Story 3 - Review Own Grades and Attendance (Priority: P3)

**Goal**: Students view their own current-period grade and attendance lists with read-only loaded-record details, pagination, empty states, denial states, and no correction/import/authoring controls.

**Independent Test**: Sign in as a student with grade and attendance records, open both lists, open read-only details from loaded records, verify no other-student or cross-school records appear, and verify stale direct detail routes show not-found without undocumented detail requests.

### Tests for User Story 3

- [ ] T037 [P] [US3] Add grade service and mapper tests for `listStudentGrades`, current active academic-period parameter, pagination, student-visible fields, dropped private fields, and no undocumented detail request in `tests/student-self-service/services/studentGradesService.spec.js`
- [ ] T038 [P] [US3] Add attendance service and mapper tests for `listStudentAttendance`, current active academic-period parameter, pagination, student-visible fields, dropped private fields, and no undocumented detail request in `tests/student-self-service/services/studentAttendanceService.spec.js`
- [ ] T039 [P] [US3] Add grade and attendance composable tests for pagination, current-period scope, empty states, loaded-record detail lookup, stale direct route not-found, and stale-response handling in `tests/student-self-service/composables/useStudentAcademicRecords.spec.js`
- [ ] T040 [P] [US3] Add grade and attendance route/component tests for own-record lists, read-only details, no correction/import/restore controls, denied states, empty states, and not-found states in `tests/student-self-service/components/studentAcademicRecordViews.spec.js`

### Implementation for User Story 3

- [ ] T041 [P] [US3] Implement student grade service method and response mapper in `src/services/student/studentSelfServiceService.js`
- [ ] T042 [P] [US3] Implement student attendance service method and response mapper in `src/services/student/studentSelfServiceService.js`
- [ ] T043 [US3] Implement shared student academic records composable for current-period scope, pagination, loaded-record detail lookup, empty state, stale guard, and safe feedback in `src/composables/student/useStudentAcademicRecords.js`
- [ ] T044 [P] [US3] Implement student grades list and read-only detail route states in `src/pages/student/StudentGradesView.vue`
- [ ] T045 [P] [US3] Implement student attendance list and read-only detail route states in `src/pages/student/StudentAttendanceView.vue`
- [ ] T046 [P] [US3] Implement reusable read-only academic record detail component with no correction/import/restore actions in `src/components/student/StudentAcademicRecordDetail.vue`
- [ ] T047 [US3] Register grades and attendance routes in `src/router/modules/student.js`
- [ ] T048 [US3] Add grade, attendance, read-only detail, empty, denied, and not-found text in `src/i18n/modules/studentSelfService.js`

**Checkpoint**: User Story 3 is independently functional and own-record scoped.

---

## Phase 6: User Story 4 - View Approved Academic Summaries (Priority: P4)

**Goal**: Students view a limited academic overview with counts and statuses only, while report, transcript, GPA, attendance-rate, ranking, and trend behavior stays absent.

**Independent Test**: Open academic overview with approved learning-set, grade, and attendance data, verify counts/statuses match loaded responses, and confirm no report-run, report-download, transcript, calculated GPA, calculated attendance rate, ranking, or trend controls appear.

### Tests for User Story 4

- [ ] T049 [P] [US4] Add academic overview composable tests for learning-set counts, content-availability counts, grade status counts, attendance status counts, no GPA, no attendance-rate, and stale-response handling in `tests/student-self-service/composables/useStudentAcademicOverview.spec.js`
- [ ] T050 [P] [US4] Add academic overview route/component tests for counts/statuses only, no report controls, no transcript controls, no rankings/trends, and context gates in `tests/student-self-service/components/studentAcademicOverviewView.spec.js`

### Implementation for User Story 4

- [ ] T051 [US4] Implement academic overview aggregation from approved loaded learning-set, grade, attendance, and content-availability responses in `src/composables/student/useStudentAcademicOverview.js`
- [ ] T052 [P] [US4] Implement academic overview summary cards for counts/statuses only in `src/components/student/StudentAcademicOverviewCards.vue`
- [ ] T053 [US4] Implement academic overview route with no-active-school, no-student-profile, no-current-period, empty, denied, and stale states in `src/pages/student/StudentAcademicOverviewView.vue`
- [ ] T054 [US4] Register academic overview route in `src/router/modules/student.js`
- [ ] T055 [US4] Add academic overview, counts/statuses, blocked report, and blocked metric display text in `src/i18n/modules/studentSelfService.js`

**Checkpoint**: User Story 4 is independently functional and summary-limited.

---

## Phase 7: Polish and Cross-Cutting Concerns

**Purpose**: Verify feature-wide contract compliance, accessibility, diagnostics, documentation, and quality gates.

- [ ] T056 [P] Verify operation ID to UI surface traceability against `specs/024-student-self-service-ui/contracts/student-self-service-ui-contract.md`
- [ ] T057 [P] Verify WCAG 2.1 AA responsive behavior, keyboard navigation, focus visibility, form/control labels, landmarks/headings, and contrast at 390px, 768px, and 1440px for all student routes in `src/pages/student/`
- [ ] T058 Verify no direct Axios calls exist outside student services in `src/pages/student/`, `src/components/student/`, and `src/composables/student/`
- [ ] T059 Verify no manual period switch, questionnaire response submit/review, standalone detail API calls, report UI, transcript UI, GPA, attendance-rate, ranking, trend, correction, import, restore, guardian, teacher, administrator, platform, billing, messaging, or undocumented behavior exists in `src/pages/student/`
- [ ] T060 [P] Verify diagnostics and test output redaction for private file paths, storage keys, tokens, role internals, scan internals, guardian data, other-student data, and cross-tenant details in `tests/student-self-service/`
- [ ] T061 Run focused student self-service tests through `npm run test:unit -- tests/student-self-service` and record results in the implementation PR notes
- [ ] T062 Run full frontend unit tests through the `npm run test:unit` script in `package.json` and record results in the implementation PR notes
- [ ] T063 Run frontend build verification through the `npm run build` script in `package.json` and record results in the implementation PR notes
- [ ] T064 Run OpenAPI validation with `npx @redocly/cli lint api/openapi.yaml` only if `api/openapi.yaml` changed, and record results in the implementation PR notes
- [ ] T065 Record timed usability evidence for SC-002, SC-003, and SC-004 against `specs/024-student-self-service-ui/spec.md` using assigned learning-set detail under 2 minutes, content download under 2 minutes, and grades/attendance review under 3 minutes
- [ ] T066 Record representative student UAT evidence for SC-008 against `specs/024-student-self-service-ui/spec.md` confirming at least 90% can distinguish active assignment, empty assignment, downloadable content, unavailable content, no-current-period, and not-found states
- [ ] T067 Verify frontend timing goals from `specs/024-student-self-service-ui/plan.md` and record evidence in the implementation PR notes for mocked service render within 1.5s, protected route transition within 2s, and download action start within 2s after service resolution

---

## Dependencies and Execution Order

### Phase Dependencies

- Setup (Phase 1): no dependencies.
- Foundational (Phase 2): depends on Setup completion and blocks all user stories.
- User Story 1 (Phase 3): depends on Foundation; recommended MVP.
- User Story 2 (Phase 4): depends on Foundation and integrates with US1 learning-set entry rendering.
- User Story 3 (Phase 5): depends on Foundation and can run parallel to US2 after shared context and feedback exist.
- User Story 4 (Phase 6): depends on Foundation and can run after US1 and US3 data flows exist, or use mocked approved responses for independent testing.
- Polish (Phase 7): depends on selected user stories being complete.

### User Story Dependencies

- US1: can start after Foundation; no other story dependency.
- US2: can start after Foundation; uses US1 entry component integration but download behavior remains independently testable.
- US3: can start after Foundation; independent from US1 and US2 except shared context and feedback.
- US4: depends on approved data shapes from US1 and US3; may be built against mocked approved responses but final integration should follow US1 and US3.

### Within Each User Story

- Write tests first and confirm they fail.
- Implement service/mappers before composables.
- Implement composables before route views.
- Add route registration and i18n after primary components exist.
- Verify each story independently before starting polish.

## Parallel Opportunities

- T003, T004, and T005 can run in parallel after T002 is agreed.
- T006, T010, T011, T014, and T015 can run in parallel after T007 and T008 are drafted.
- US1 tests T018, T019, and T020 can run in parallel.
- US1 components T023, T024, and T025 can run in parallel after T022 exists.
- US2 tests T029, T030, and T031 can run in parallel.
- US3 grade work T037, T041, and T044 can run in parallel with attendance work T038, T042, and T045 after shared mappers are established.
- US4 tests T049 and T050 can run in parallel.
- Polish checks T056, T057, and T060 can run in parallel after selected stories are implemented.

## Parallel Example: User Story 1

```bash
Task: "T018 [P] [US1] Add assigned learning-set service tests in tests/student-self-service/services/assignedLearningSetsService.spec.js"
Task: "T019 [P] [US1] Add assigned learning-set composable tests in tests/student-self-service/composables/useAssignedLearningSets.spec.js"
Task: "T020 [P] [US1] Add assigned learning-set component tests in tests/student-self-service/components/assignedLearningSetViews.spec.js"
Task: "T023 [P] [US1] Implement assigned learning-set list component in src/components/student/AssignedLearningSetList.vue"
Task: "T024 [P] [US1] Implement learning-set entry list component in src/components/student/StudentLearningSetEntryList.vue"
```

## Parallel Example: User Story 2

```bash
Task: "T029 [P] [US2] Add content download service tests in tests/student-self-service/services/studentContentDownloadService.spec.js"
Task: "T030 [P] [US2] Add content download composable tests in tests/student-self-service/composables/useStudentContentDownload.spec.js"
Task: "T031 [P] [US2] Add content action component tests in tests/student-self-service/components/studentContentDownloadViews.spec.js"
Task: "T034 [P] [US2] Implement download action component in src/components/student/StudentContentDownloadAction.vue"
```

## Parallel Example: User Story 3

```bash
Task: "T037 [P] [US3] Add grade service tests in tests/student-self-service/services/studentGradesService.spec.js"
Task: "T038 [P] [US3] Add attendance service tests in tests/student-self-service/services/studentAttendanceService.spec.js"
Task: "T041 [P] [US3] Implement grade service method in src/services/student/studentSelfServiceService.js"
Task: "T042 [P] [US3] Implement attendance service method in src/services/student/studentSelfServiceService.js"
Task: "T046 [P] [US3] Implement academic record detail component in src/components/student/StudentAcademicRecordDetail.vue"
```

## Parallel Example: User Story 4

```bash
Task: "T049 [P] [US4] Add academic overview composable tests in tests/student-self-service/composables/useStudentAcademicOverview.spec.js"
Task: "T050 [P] [US4] Add academic overview component tests in tests/student-self-service/components/studentAcademicOverviewView.spec.js"
Task: "T052 [P] [US4] Implement overview summary cards in src/components/student/StudentAcademicOverviewCards.vue"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 for User Story 1.
3. Run focused US1 tests and manual checks for Assigned Learning Sets, default landing, read-only questionnaire entries, empty/no-current-period/no-student-profile states, and loaded-list-backed detail.
4. Stop and demo MVP before adding content download, academic records, or overview.

### Incremental Delivery

1. Complete Setup and Foundation.
2. Add US1 Assigned Learning Sets as MVP.
3. Add US2 authorized content download.
4. Add US3 own grades and attendance.
5. Add US4 academic overview.
6. Run polish checks and quickstart validation.

### Parallel Team Strategy

1. Team completes Setup and Foundation together.
2. Developer A implements US1 default landing and learning sets.
3. Developer B implements US2 download after US1 entry component contract is stable.
4. Developer C implements US3 grades/attendance.
5. Developer D implements US4 overview after data-shape mocks are stable.
6. Team finishes polish checks together.

## Notes

- [P] tasks use different files or test scopes and can run in parallel after dependencies are met.
- [US1], [US2], [US3], and [US4] labels map to spec user stories.
- This is a frontend-only implementation task list unless a future contract gap creates separate backend/OpenAPI work.
- No task may introduce undocumented API behavior or direct Axios calls outside services.
- Verify required critical-flow tests fail before implementing each story.
