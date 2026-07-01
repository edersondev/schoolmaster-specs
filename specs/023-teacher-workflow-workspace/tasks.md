# Tasks: Teacher Workflow Workspace

**Input**: Design documents from `specs/023-teacher-workflow-workspace/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/teacher-workflow-workspace-ui-contract.md](contracts/teacher-workflow-workspace-ui-contract.md), [quickstart.md](quickstart.md)

**Tests**: Required by FR-019 and quickstart verification. Write focused Vitest and contract-mapper tests before implementation tasks in each story.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after setup and foundation.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish the frontend module, task traceability, and contract gate before shared implementation starts.

- [ ] T001 Confirm OpenAPI contract gaps for roster-aware `createLearningSet` and period/roster filters in `api/openapi.yaml` and record the result in `specs/023-teacher-workflow-workspace/contracts/teacher-workflow-workspace-ui-contract.md`
- [ ] T002 Create the teacher workflow module folders in `src/modules/teacher-workflow/components/`, `src/modules/teacher-workflow/composables/`, `src/modules/teacher-workflow/routes/`, `src/modules/teacher-workflow/services/`, and `src/modules/teacher-workflow/types/`
- [ ] T003 [P] Create focused test folders in `tests/teacher-workflow/components/`, `tests/teacher-workflow/composables/`, `tests/teacher-workflow/routes/`, and `tests/teacher-workflow/services/`
- [ ] T004 [P] Add the teacher workflow i18n namespace scaffold in `src/i18n/modules/teacherWorkflow.js`
- [ ] T005 [P] Add the teacher workflow route export scaffold in `src/modules/teacher-workflow/routes/index.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared scope, service, feedback, permission, and diagnostic infrastructure required by every story.

**Critical**: No user story work starts until this phase is complete.

- [ ] T006 Create the approved operation and capability map in `src/modules/teacher-workflow/services/teacherWorkflowContract.js`
- [ ] T007 Create the service HTTP wrapper that uses the existing Axios service boundary in `src/modules/teacher-workflow/services/teacherWorkflowHttp.js`
- [ ] T008 [P] Create shared feedback-state normalization for validation, denial, conflict, download, import, unsupported, stale, and temporary states in `src/modules/teacher-workflow/services/teacherWorkflowFeedbackMapper.js`
- [ ] T009 [P] Create shared status and lifecycle helpers for active, inactive, deleted, restored, scan, unavailable, and read-only legacy states in `src/modules/teacher-workflow/services/teacherWorkflowStatus.js`
- [ ] T010 Create the workspace scope composable for active school, current academic period, active teacher roster, and route query sync in `src/modules/teacher-workflow/composables/useTeacherWorkspaceScope.js`
- [ ] T011 Create the stale-response guard composable for route, pagination, period, roster, school, target, dialog, and auth changes in `src/modules/teacher-workflow/composables/useTeacherWorkflowStaleGuard.js`
- [ ] T012 [P] Create the safe diagnostics helper that redacts students, teachers, guardians, tokens, permissions, roles, file paths, upload metadata, import payloads, correction notes, and cross-tenant details in `src/modules/teacher-workflow/services/teacherWorkflowDiagnostics.js`
- [ ] T013 [P] Create shared feedback components for empty, denied, conflict, validation, unsupported-contract, and temporary-unavailable states in `src/modules/teacher-workflow/components/TeacherWorkflowFeedbackState.vue`
- [ ] T014 [P] Create shared status and action components for workflow badges, disabled reasons, and confirmation dialogs in `src/modules/teacher-workflow/components/TeacherWorkflowStatusControls.vue`
- [ ] T015 Wire protected teacher and admin-observed route groups into the application router in `src/router/index.js`
- [ ] T016 Add foundation tests for scope query sync, stale guard, feedback mapping, and diagnostics redaction in `tests/teacher-workflow/composables/teacherWorkflowFoundation.spec.js`

**Checkpoint**: Foundation ready. User story implementation can begin.

---

## Phase 3: User Story 1 - Manage teacher materials (Priority: P1) MVP

**Goal**: Teachers manage content and questionnaires with upload, scan/download, lifecycle, empty, denied, conflict, and admin-observed read/detail behavior.

**Independent Test**: Sign in as an authorized teacher with active school context, list content and questionnaires, upload valid and invalid content, create or view a questionnaire, download clean content, attempt unavailable downloads, change approved lifecycle state, and verify empty, validation, authorization, tenant-mismatch, scan-pending, conflict, and stale-response states.

### Tests for User Story 1

- [ ] T017 [P] [US1] Add teacher content service and mapper tests for list, upload, detail, metadata update, lifecycle, delete, restore, clean download, scan states, and download denial in `tests/teacher-workflow/services/teacherContentService.spec.js`
- [ ] T018 [P] [US1] Add questionnaire service and mapper tests for list, create, detail, update, lifecycle, delete, restore, supported question types, and blocked response/grading fields in `tests/teacher-workflow/services/questionnaireService.spec.js`
- [ ] T019 [P] [US1] Add content composable tests for pagination, upload validation, scan-gated availability, transient download metadata, conflict mapping, and stale responses in `tests/teacher-workflow/composables/useTeacherContent.spec.js`
- [ ] T020 [P] [US1] Add questionnaire composable tests for supported authoring, ordering, validation, historical-meaning conflict feedback, and stale responses in `tests/teacher-workflow/composables/useQuestionnaires.spec.js`
- [ ] T021 [P] [US1] Add component tests for content, questionnaire, and admin-observed read/detail states in `tests/teacher-workflow/components/teacherMaterialsViews.spec.js`

### Implementation for User Story 1

- [ ] T022 [P] [US1] Implement teacher content API methods and request mappers in `src/modules/teacher-workflow/services/teacherContentService.js`
- [ ] T023 [P] [US1] Implement questionnaire API methods and request mappers in `src/modules/teacher-workflow/services/questionnaireService.js`
- [ ] T024 [US1] Implement teacher content state, pagination, upload draft, lifecycle actions, download handling, and stale guard integration in `src/modules/teacher-workflow/composables/useTeacherContent.js`
- [ ] T025 [US1] Implement questionnaire state, authoring draft, lifecycle actions, unsupported response/grading omission, and stale guard integration in `src/modules/teacher-workflow/composables/useQuestionnaires.js`
- [ ] T026 [P] [US1] Implement content list and empty/denied/loading states in `src/modules/teacher-workflow/routes/TeacherContentListView.vue`
- [ ] T027 [P] [US1] Implement content detail, metadata edit, lifecycle, scan, delete, restore, and download controls in `src/modules/teacher-workflow/routes/TeacherContentDetailView.vue`
- [ ] T028 [P] [US1] Implement content upload form with approved fields, 25 MB limit feedback, content type choices, and upload rejection feedback in `src/modules/teacher-workflow/components/TeacherContentUploadForm.vue`
- [ ] T029 [P] [US1] Implement questionnaire list and empty/denied/loading states in `src/modules/teacher-workflow/routes/QuestionnaireListView.vue`
- [ ] T030 [P] [US1] Implement questionnaire create/update/detail form with multiple-choice, true-false, short-text, ordering, validation, lifecycle, delete, and restore controls in `src/modules/teacher-workflow/routes/QuestionnaireDetailView.vue`
- [ ] T031 [US1] Implement admin-observed material read/detail views without teacher-owned management takeover in `src/modules/teacher-workflow/routes/AdminTeacherMaterialsView.vue`
- [ ] T032 [US1] Register content, questionnaire, and admin-observed material routes in `src/modules/teacher-workflow/routes/index.js`
- [ ] T033 [US1] Add content and questionnaire display text to `src/i18n/modules/teacherWorkflow.js`

**Checkpoint**: User Story 1 is independently functional and testable.

---

## Phase 4: User Story 2 - Publish roster-aware learning sets (Priority: P2)

**Goal**: Teachers can view learning-set detail and blocked scoped list/create states safely, then use roster-aware learning-set behavior only after OpenAPI supports it.

**Independent Test**: Open an assigned roster or teacher workspace, verify create and scoped list controls are blocked while roster-aware create or scoped list contracts are missing, enable controls only after approved OpenAPI support exists, create or update a learning set with clean content and active questionnaires, verify roster-aware audience and read-only legacy direct assignments, and confirm dependency conflicts show no partial success.

### Tests for User Story 2

- [ ] T034 [P] [US2] Add learning-set service tests for blocked create gate, blocked scoped list gate, detail, update, lifecycle, delete, restore, read-only legacy direct assignments, and no undocumented filters in `tests/teacher-workflow/services/learningSetService.spec.js`
- [ ] T035 [P] [US2] Add learning-set composable tests for selected period/roster scope, unsupported-contract states, dependency conflicts, stale responses, and no broad client-side filtering in `tests/teacher-workflow/composables/useLearningSets.spec.js`
- [ ] T036 [P] [US2] Add learning-set route/component tests for blocked create, blocked scoped list, roster-aware audience display, legacy read-only display, lifecycle states, and dependency conflict feedback in `tests/teacher-workflow/components/learningSetViews.spec.js`

### Implementation for User Story 2

- [ ] T037 [US2] Implement learning-set service methods, request mappers, response mappers, contract-gate checks, and no-undocumented-filter enforcement in `src/modules/teacher-workflow/services/learningSetService.js`
- [ ] T038 [US2] Implement learning-set state, selected period/roster scope, blocked scoped-list handling, blocked create handling, dependency conflict mapping, and stale guard integration in `src/modules/teacher-workflow/composables/useLearningSets.js`
- [ ] T039 [P] [US2] Implement learning-set list with unsupported-contract, empty, loading, status, read-only legacy, and denial states in `src/modules/teacher-workflow/routes/LearningSetListView.vue`
- [ ] T040 [P] [US2] Implement learning-set detail with entries, roster-aware audience, lifecycle, dependency conflict, delete, restore, and read-only legacy direct assignments in `src/modules/teacher-workflow/routes/LearningSetDetailView.vue`
- [ ] T041 [P] [US2] Implement learning-set form shell that remains disabled until OpenAPI roster-aware create support is confirmed in `src/modules/teacher-workflow/components/LearningSetForm.vue`
- [ ] T042 [US2] Register learning-set routes and route query handling in `src/modules/teacher-workflow/routes/index.js`
- [ ] T043 [US2] Add learning-set display text, unsupported-contract copy, and legacy read-only labels to `src/i18n/modules/teacherWorkflow.js`

**Checkpoint**: User Story 2 is independently functional and contract-gated.

---

## Phase 5: User Story 3 - Record and correct grades and attendance (Priority: P3)

**Goal**: Teachers record grades and attendance for approved roster context, review current values and tenant-safe correction history, and submit approved corrections while admin closed-period authority remains explicit.

**Independent Test**: Verify scoped grade and attendance lists are blocked while period/roster filters are missing, open scoped lists only after approved filters exist, create valid grade and attendance records, submit invalid values, open details, submit corrections with valid and invalid reasons, review redacted correction history, and confirm teacher closed-period denial plus administrator authority where approved.

### Tests for User Story 3

- [ ] T044 [P] [US3] Add grade service tests for blocked scoped list gate, create, detail, correction, lifecycle, delete, restore, validation, closed-period denial, and tenant-safe correction history mapping in `tests/teacher-workflow/services/gradeService.spec.js`
- [ ] T045 [P] [US3] Add attendance service tests for blocked scoped list gate, create, detail, correction, lifecycle, delete, restore, supported statuses, closed-period denial, and tenant-safe correction history mapping in `tests/teacher-workflow/services/attendanceService.spec.js`
- [ ] T046 [P] [US3] Add grade and attendance composable tests for selected period/roster scope, correction reason length, current value updates, private-note redaction, conflict feedback, and stale responses in `tests/teacher-workflow/composables/useAcademicRecords.spec.js`
- [ ] T047 [P] [US3] Add grade, attendance, correction dialog, and admin closed-period component tests in `tests/teacher-workflow/components/academicRecordViews.spec.js`

### Implementation for User Story 3

- [ ] T048 [P] [US3] Implement grade API methods, request mappers, response mappers, correction history redaction, and blocked scoped-list enforcement in `src/modules/teacher-workflow/services/gradeService.js`
- [ ] T049 [P] [US3] Implement attendance API methods, request mappers, response mappers, correction history redaction, and blocked scoped-list enforcement in `src/modules/teacher-workflow/services/attendanceService.js`
- [ ] T050 [US3] Implement grade state, selected period/roster scope, create draft, lifecycle actions, correction draft, blocked scoped-list handling, and stale guard integration in `src/modules/teacher-workflow/composables/useGrades.js`
- [ ] T051 [US3] Implement attendance state, selected period/roster scope, create draft, lifecycle actions, correction draft, blocked scoped-list handling, and stale guard integration in `src/modules/teacher-workflow/composables/useAttendance.js`
- [ ] T052 [P] [US3] Implement reusable correction reason dialog with 10-500 character validation, current/original value display, denial feedback, and private-note redaction in `src/modules/teacher-workflow/components/CorrectionReasonDialog.vue`
- [ ] T053 [P] [US3] Implement grade list/detail/create/correction route states in `src/modules/teacher-workflow/routes/GradeRecordsView.vue`
- [ ] T054 [P] [US3] Implement attendance list/detail/create/correction route states in `src/modules/teacher-workflow/routes/AttendanceRecordsView.vue`
- [ ] T055 [US3] Implement admin-observed grade and attendance detail with closed-period correction controls only where approved in `src/modules/teacher-workflow/routes/AdminAcademicRecordsView.vue`
- [ ] T056 [US3] Register grade, attendance, correction, and admin-observed academic record routes in `src/modules/teacher-workflow/routes/index.js`
- [ ] T057 [US3] Add grade, attendance, correction, closed-period, and history display text to `src/i18n/modules/teacherWorkflow.js`

**Checkpoint**: User Story 3 is independently functional and contract-gated.

---

## Phase 6: User Story 4 - Run approved imports and review operational feedback (Priority: P4)

**Goal**: School administrators run approved grade and attendance structured JSON imports with 500-row cap, all-or-nothing feedback, and safe denial for unauthorized actors.

**Independent Test**: Open approved grade or attendance import screens as a school administrator, submit valid and invalid structured JSON payloads, verify accepted summaries or all-or-nothing rejection details, attempt teacher or unauthorized import access, and confirm no CSV, spreadsheet, archive, file-upload, or partial success UI appears.

### Tests for User Story 4

- [ ] T058 [P] [US4] Add import service tests for grade import, attendance import, JSON-only payloads, 1-500 row cap, all-or-nothing rejection, no partial success, and safe validation mapping in `tests/teacher-workflow/services/importService.spec.js`
- [ ] T059 [P] [US4] Add import composable tests for admin-only access, denied actors, draft cleanup, payload redaction, stale responses, and rejected import summaries in `tests/teacher-workflow/composables/useTeacherWorkflowImports.spec.js`
- [ ] T060 [P] [US4] Add import route/component tests for JSON editor, accepted summary, rejected summary, absent CSV/spreadsheet/file-upload controls, and unauthorized route feedback in `tests/teacher-workflow/components/importViews.spec.js`

### Implementation for User Story 4

- [ ] T061 [US4] Implement grade and attendance import API methods, request mappers, response mappers, row-limit validation, and all-or-nothing feedback mapping in `src/modules/teacher-workflow/services/importService.js`
- [ ] T062 [US4] Implement import draft state, admin-only permission gate, JSON parsing, row cap checks, payload cleanup, diagnostics redaction, and stale guard integration in `src/modules/teacher-workflow/composables/useTeacherWorkflowImports.js`
- [ ] T063 [P] [US4] Implement structured JSON import editor with grade/attendance mode, row count, validation summary, accepted summary, rejected summary, and no file-upload controls in `src/modules/teacher-workflow/components/TeacherWorkflowImportEditor.vue`
- [ ] T064 [US4] Implement admin import route with denied states for teachers, students, guardians, platform-without-school authority, and unauthorized administrators in `src/modules/teacher-workflow/routes/AdminTeacherWorkflowImportsView.vue`
- [ ] T065 [US4] Register admin import routes in `src/modules/teacher-workflow/routes/index.js`
- [ ] T066 [US4] Add import, JSON payload, row limit, all-or-nothing, and authorization denial display text to `src/i18n/modules/teacherWorkflow.js`

**Checkpoint**: User Story 4 is independently functional and admin-gated.

---

## Phase 7: Polish and Cross-Cutting Concerns

**Purpose**: Verify feature-wide contract compliance, accessibility, diagnostics, documentation, and quality gates.

- [ ] T067 [P] Verify route, permission, operation ID, and UI-surface traceability against `specs/023-teacher-workflow-workspace/contracts/teacher-workflow-workspace-ui-contract.md`
- [ ] T068 [P] Verify responsive and keyboard behavior at 390px, 768px, and 1440px for all teacher workflow routes in `src/modules/teacher-workflow/routes/`
- [ ] T069 Verify no direct Axios calls exist outside services in `src/modules/teacher-workflow/`
- [ ] T070 Verify no student response review, teacher response grading, student response file downloads, CSV import, spreadsheet import, archive import, file-upload import, platform support, reporting, or undocumented API behavior exists in `src/modules/teacher-workflow/`
- [ ] T071 [P] Verify diagnostics and test output redaction for student, teacher, guardian, token, permission, role, private file path, upload metadata, import payload, correction private note, and cross-tenant details in `tests/teacher-workflow/`
- [ ] T072 Run focused teacher workflow unit tests through `npm run test:unit -- tests/teacher-workflow` and record results in the implementation PR notes
- [ ] T073 Run full frontend unit tests through the `npm run test:unit` script in `package.json` and record results in the implementation PR notes
- [ ] T074 Run frontend build verification through the `npm run build` script in `package.json` and record results in the implementation PR notes
- [ ] T075 Run OpenAPI validation with `npx @redocly/cli lint api/openapi.yaml` only if `api/openapi.yaml` changed, and record results in the implementation PR notes
- [ ] T076 Record timed usability evidence for SC-002, SC-003, and SC-004 in the implementation PR notes using teacher learning-set detail under 2 minutes, content/questionnaire/learning-set workflow under 6 minutes, and grade or attendance validation workflow under 4 minutes
- [ ] T077 Record representative teacher and administrator UAT evidence for SC-009 in the implementation PR notes confirming at least 90% can distinguish active, inactive, deleted, restored, scan-pending, scan-failed, unavailable, read-only legacy, and conflict states
- [ ] T078 Verify frontend timing goals from `specs/023-teacher-workflow-workspace/plan.md` and record evidence in the implementation PR notes for mocked service render within 1.5s, route transition within 2s, and submission settlement within 2s after service resolution

---

## Dependencies and Execution Order

### Phase Dependencies

- Setup (Phase 1): no dependencies.
- Foundational (Phase 2): depends on Setup completion and blocks all user stories.
- User Story 1 (Phase 3): depends on Foundation; recommended MVP.
- User Story 2 (Phase 4): depends on Foundation and may reuse US1 content/questionnaire services, but must remain contract-gated.
- User Story 3 (Phase 5): depends on Foundation and may run parallel to US2 after scope infrastructure exists.
- User Story 4 (Phase 6): depends on Foundation and may run parallel to US3 after admin route and feedback patterns exist.
- Polish (Phase 7): depends on selected user stories being complete.

### User Story Dependencies

- US1: can start after Foundation; no other story dependency.
- US2: can start after Foundation; integrates with US1 material references but blocked states are independently testable.
- US3: can start after Foundation; independent from US1 and US2 except shared scope and feedback.
- US4: can start after Foundation; independent from US1 and US2 and shares US3 academic record service concepts.

### Within Each User Story

- Write tests first and confirm they fail.
- Implement services before composables.
- Implement composables before route views.
- Add route registration and i18n after primary components exist.
- Verify each story independently before starting polish.

## Parallel Opportunities

- T003, T004, and T005 can run in parallel after T002.
- T008, T009, T012, T013, and T014 can run in parallel after T006 and T007 are drafted.
- US1 service tests T017 and T018 can run in parallel with component test drafting T021.
- US1 services T022 and T023 can run in parallel before composables T024 and T025.
- US2 tests T034, T035, and T036 can run in parallel.
- US3 grade work T044, T048, T050, and T053 can run in parallel with attendance work T045, T049, T051, and T054 after shared correction contracts are agreed.
- US4 tests T058, T059, and T060 can run in parallel.
- Polish checks T067, T068, and T071 can run in parallel after selected stories are implemented.

## Parallel Example: User Story 1

```bash
Task: "T017 [P] [US1] Add teacher content service and mapper tests in tests/teacher-workflow/services/teacherContentService.spec.js"
Task: "T018 [P] [US1] Add questionnaire service and mapper tests in tests/teacher-workflow/services/questionnaireService.spec.js"
Task: "T021 [P] [US1] Add component tests in tests/teacher-workflow/components/teacherMaterialsViews.spec.js"
Task: "T022 [P] [US1] Implement teacher content service in src/modules/teacher-workflow/services/teacherContentService.js"
Task: "T023 [P] [US1] Implement questionnaire service in src/modules/teacher-workflow/services/questionnaireService.js"
```

## Parallel Example: User Story 2

```bash
Task: "T034 [P] [US2] Add learning-set service tests in tests/teacher-workflow/services/learningSetService.spec.js"
Task: "T035 [P] [US2] Add learning-set composable tests in tests/teacher-workflow/composables/useLearningSets.spec.js"
Task: "T036 [P] [US2] Add learning-set component tests in tests/teacher-workflow/components/learningSetViews.spec.js"
Task: "T039 [P] [US2] Implement learning-set list view in src/modules/teacher-workflow/routes/LearningSetListView.vue"
Task: "T040 [P] [US2] Implement learning-set detail view in src/modules/teacher-workflow/routes/LearningSetDetailView.vue"
```

## Parallel Example: User Story 3

```bash
Task: "T044 [P] [US3] Add grade service tests in tests/teacher-workflow/services/gradeService.spec.js"
Task: "T045 [P] [US3] Add attendance service tests in tests/teacher-workflow/services/attendanceService.spec.js"
Task: "T048 [P] [US3] Implement grade service in src/modules/teacher-workflow/services/gradeService.js"
Task: "T049 [P] [US3] Implement attendance service in src/modules/teacher-workflow/services/attendanceService.js"
Task: "T052 [P] [US3] Implement correction dialog in src/modules/teacher-workflow/components/CorrectionReasonDialog.vue"
```

## Parallel Example: User Story 4

```bash
Task: "T058 [P] [US4] Add import service tests in tests/teacher-workflow/services/importService.spec.js"
Task: "T059 [P] [US4] Add import composable tests in tests/teacher-workflow/composables/useTeacherWorkflowImports.spec.js"
Task: "T060 [P] [US4] Add import component tests in tests/teacher-workflow/components/importViews.spec.js"
Task: "T063 [P] [US4] Implement import editor in src/modules/teacher-workflow/components/TeacherWorkflowImportEditor.vue"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 for User Story 1.
3. Run focused US1 tests and manual checks for content/questionnaire upload, lifecycle, download, denial, conflict, and admin observation.
4. Ship only US1 if later stories are not ready.

### Incremental Delivery

1. Add US2 as a contract-gated learning-set increment.
2. Add US3 for grade, attendance, corrections, and correction history.
3. Add US4 for admin-only structured JSON imports.
4. Finish Phase 7 quality, accessibility, diagnostics, and build checks.

### Contract Gate Strategy

- Keep learning-set create disabled until `api/openapi.yaml` documents roster-aware create behavior.
- Keep scoped learning-set, grade, and attendance lists disabled until `api/openapi.yaml` documents period or roster filters.
- Never send undocumented filters or load broad lists for client-side filtering.
- Keep legacy direct selected-student learning-set assignments read-only when returned.
