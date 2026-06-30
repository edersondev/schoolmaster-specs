# Tasks: Student Enrollment and Classroom Roster UI

**Input**: Design documents from `specs/022-student-enrollment-roster-ui/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/student-enrollment-roster-ui-contract.md`, `quickstart.md`

**Tests**: Frontend Vitest coverage is required because this feature changes
critical school-owned student, roster, membership, teacher-assignment,
permission, tenant, stale-response, and diagnostics flows. Backend PHPUnit
tasks are not included because this is a frontend-only implementation slice.
OpenAPI validation is required only if contract review creates a separate
contract change.

**Organization**: Tasks are grouped by user story so each story can be
implemented and tested independently. Paths prefixed with
`schoolmaster-frontend/` are implemented from the frontend repository. Paths
under `specs/022-student-enrollment-roster-ui/` are maintained in
`schoolmaster-specs`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because the task changes different files and
  does not depend on another incomplete task.
- **[Story]**: Maps the task to a user story from `spec.md`.
- Setup, Foundational, and Polish phases have no story label.
- No backend or OpenAPI implementation changes are planned unless contract
  verification finds a blocking mismatch before frontend exposure.

## Phase 1: Setup

**Purpose**: Confirm contract readiness and create feature-local frontend test,
localization, and folder scaffolding before shared student/roster work starts.

- [ ] T001 Verify approved student profile, class-section, roster membership, and teacher-assignment operation IDs, request fields, response envelopes, filters, pagination, and no `updateStudentProfile` operation against `api/openapi.yaml` and record blocking mismatches in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T002 Verify Feature 016 through Feature 021 frontend foundations exist in `schoolmaster-frontend/src/router/`, `schoolmaster-frontend/src/pages/admin-system/`, `schoolmaster-frontend/src/components/ui/admin/`, and `schoolmaster-frontend/src/services/api/`
- [ ] T003 [P] Create student enrollment roster test fixture scaffold for sessions, schools, periods, student profiles, rosters, memberships, teacher assignments, envelopes, and errors in `schoolmaster-frontend/tests/unit/student-enrollment-roster/fixtures/studentEnrollmentRoster.fixtures.js`
- [ ] T004 [P] Create test directory markers for contracts, services, composables, components, pages, and routes in `schoolmaster-frontend/tests/unit/student-enrollment-roster/contracts/.gitkeep`, `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/.gitkeep`, `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/.gitkeep`, `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/.gitkeep`, `schoolmaster-frontend/tests/unit/student-enrollment-roster/pages/.gitkeep`, and `schoolmaster-frontend/tests/unit/student-enrollment-roster/routes/.gitkeep`
- [ ] T005 [P] Create student administration component folders in `schoolmaster-frontend/src/components/admin-system/students/` and `schoolmaster-frontend/src/pages/admin-system/students/`
- [ ] T006 [P] Create class-section roster component folders in `schoolmaster-frontend/src/components/admin-system/class-sections/` and `schoolmaster-frontend/src/pages/admin-system/class-sections/`
- [ ] T007 [P] Create frontend contract and service placeholders in `schoolmaster-frontend/src/contracts/admin-system/student-profiles.js`, `schoolmaster-frontend/src/contracts/admin-system/classroom-roster.js`, `schoolmaster-frontend/src/contracts/admin-system/teacher-assignments.js`, `schoolmaster-frontend/src/services/admin-system/studentProfiles.js`, `schoolmaster-frontend/src/services/admin-system/classroomRoster.js`, and `schoolmaster-frontend/src/services/admin-system/teacherAssignments.js`
- [ ] T008 [P] Create centralized student enrollment roster locale scaffold in `schoolmaster-frontend/src/locales/student-enrollment-roster.js`

---

## Phase 2: Foundational

**Purpose**: Shared contracts, service error mapping, academic-period scope,
permissions, route metadata, feedback, and diagnostics required by every user
story.

**CRITICAL**: No user story implementation begins before this phase completes.

### Foundational Tests

- [ ] T009 [P] Add Vitest coverage for student profile JSDoc contracts, create payload projection, status/transfer drafts, blocked general edit metadata, and feedback states in `schoolmaster-frontend/tests/unit/student-enrollment-roster/contracts/studentProfiles.contract.spec.js`
- [ ] T010 [P] Add Vitest coverage for class-section, membership batch, academic-period scope, and oversized-batch contract mappers in `schoolmaster-frontend/tests/unit/student-enrollment-roster/contracts/classroomRoster.contract.spec.js`
- [ ] T011 [P] Add Vitest coverage for teacher-assignment contract mappers, admin-only scope, create/deactivate payload projection, and deferred teacher route metadata in `schoolmaster-frontend/tests/unit/student-enrollment-roster/contracts/teacherAssignments.contract.spec.js`
- [ ] T012 [P] Add Vitest coverage for normalized validation, unauthorized, forbidden, tenant-mismatch, inactive-school, inactive-record, not-found, conflict, unsupported filter, unsupported sort, unsupported page-size, oversized-batch, and temporary-unavailable mapping in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentEnrollmentRosterErrorMapper.spec.js`
- [ ] T013 [P] Add Vitest coverage for no-sensitive-data diagnostics redacting student, guardian, token, permission, role, reason, full payload, private academic, and cross-tenant details in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentEnrollmentRosterDiagnostics.spec.js`
- [ ] T014 [P] Add Vitest coverage for current active academic period default, no-current-period blocked state, route/query selected-period restoration, and active-school changes in `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useAcademicPeriodScope.spec.js`
- [ ] T015 [P] Add Vitest coverage for shared safe feedback rendering, status labels, conflict states, and responsive behavior at 390px, 768px, and 1440px in `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/AdminSafeFeedbackState.spec.js`
- [ ] T016 [P] Add Vitest coverage for route metadata auth requirements, active-school gating, admin-only teacher assignment routes, blocked teacher own-assignment routes, and no direct requests before school resolution in `schoolmaster-frontend/tests/unit/student-enrollment-roster/routes/studentEnrollmentRoster.routes.spec.js`

### Foundational Implementation

- [ ] T017 Define student profile typedefs, mappers, status/transfer draft factories, blocked edit constants, and feedback constants in `schoolmaster-frontend/src/contracts/admin-system/student-profiles.js`
- [ ] T018 Define class-section, roster membership, batch draft, academic-period scope, oversized-batch, and feedback mappers in `schoolmaster-frontend/src/contracts/admin-system/classroom-roster.js`
- [ ] T019 Define teacher-assignment typedefs, admin-only constants, create/deactivate draft factories, and feedback mappers in `schoolmaster-frontend/src/contracts/admin-system/teacher-assignments.js`
- [ ] T020 Implement shared student enrollment roster error mapper without raw Axios exposure or sensitive payload leakage in `schoolmaster-frontend/src/services/admin-system/studentEnrollmentRosterErrorMapper.js`
- [ ] T021 Implement student profile service functions for `listStudentProfiles`, `createStudentProfile`, `getStudentProfile`, `updateStudentProfileStatus`, and `transferStudentProfile` in `schoolmaster-frontend/src/services/admin-system/studentProfiles.js`
- [ ] T022 Implement class-section and roster membership service functions for `listClassSections`, `createClassSection`, `getClassSection`, `updateClassSection`, `updateClassSectionStatus`, `listClassSectionMemberships`, `batchAddClassSectionMemberships`, and `batchEndClassSectionMemberships` in `schoolmaster-frontend/src/services/admin-system/classroomRoster.js`
- [ ] T023 Implement teacher-assignment service functions for `listTeacherAssignments`, `createTeacherAssignment`, `getTeacherAssignment`, and `updateTeacherAssignmentStatus` in `schoolmaster-frontend/src/services/admin-system/teacherAssignments.js`
- [ ] T024 Implement academic-period scope coordination with current-period default, route/query persistence, no-current-period blocked state, and stale-response reset in `schoolmaster-frontend/src/composables/admin-system/useAcademicPeriodScope.js`
- [ ] T025 Implement shared student enrollment roster permission and capability gate helpers for student, roster, membership, transfer, and teacher-assignment surfaces in `schoolmaster-frontend/src/composables/admin-system/useStudentEnrollmentRosterPermissions.js`
- [ ] T026 [P] Implement safe feedback component for loading, empty, validation, forbidden, tenant, inactive, not-found, conflict, unsupported, oversized-batch, and temporary states in `schoolmaster-frontend/src/components/admin-system/shared/AdminSafeFeedbackState.vue`
- [ ] T027 [P] Implement reusable academic-period selector with selected count-safe labels, current-period default, route/query emits, and no transport logic in `schoolmaster-frontend/src/components/admin-system/shared/AcademicPeriodScopeSelector.vue`
- [ ] T028 [P] Implement reusable lifecycle status tag for active, inactive, transferred, deleted, unavailable, ended, historical-only, and conflict states in `schoolmaster-frontend/src/components/admin-system/shared/AdminLifecycleStatusTag.vue`
- [ ] T029 Register student enrollment roster locale messages through existing i18n assembly in `schoolmaster-frontend/src/locales/index.js`
- [ ] T030 Add admin student and class-section route records, metadata, breadcrumbs, permission gates, and lazy page imports in `schoolmaster-frontend/src/router/modules/access-administration.routes.js`

**Checkpoint**: Shared contracts, services, composables, feedback, period scope, routes, permission gates, stale-response handling, and diagnostics are testable before story pages begin.

---

## Phase 3: User Story 1 - Manage student profiles and enrollment state (Priority: P1) MVP

**Goal**: Authorized school administrators can list same-school students, create approved student profiles, open student detail, update approved status, and see safe empty, validation, denied, inactive, not-found, and conflict states.

**Independent Test**: Sign in as an authorized school administrator, open student administration, list same-school students, create or view a student profile, apply filters and pagination, submit valid and invalid creation/status changes, and confirm cross-school, unauthorized, inactive, not-found, validation, and conflict states render correctly.

### Tests for User Story 1

- [ ] T031 [P] [US1] Add Vitest coverage for student profile list service parameters, pagination, status/search/sort allowlist, forbidden, tenant-mismatch, unsupported filter, unsupported sort, and stale cancellation in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentProfilesList.service.spec.js`
- [ ] T032 [P] [US1] Add Vitest coverage for student profile create service payload projection, required fields, guardian association duplicates, validation, conflict, and no undocumented fields in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentProfileCreate.service.spec.js`
- [ ] T033 [P] [US1] Add Vitest coverage for student profile detail and status service mapping, no general update call, safe denied/not-found states, and lifecycle result mapping in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentProfileDetailStatus.service.spec.js`
- [ ] T034 [P] [US1] Add Vitest coverage for student list query state, pagination reset, latest-request protection, active-school reset, and empty state classification in `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useStudentProfiles.spec.js`
- [ ] T035 [P] [US1] Add Vitest coverage for student status action draft, required reason/effective date, validation mapping, conflict mapping, success reconciliation, and stale-response protection in `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useStudentProfileLifecycle.spec.js`
- [ ] T036 [P] [US1] Add component tests for student profile form required fields, guardian association input, validation summary, pending state, and submit emits in `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/StudentProfileForm.spec.js`
- [ ] T037 [P] [US1] Add page-flow tests for student list/create/detail/status routes, query restoration, hidden general edit controls, safe denial states, and list return in `schoolmaster-frontend/tests/unit/student-enrollment-roster/pages/StudentProfilesPages.spec.js`

### Implementation for User Story 1

- [ ] T038 [US1] Implement student list orchestration with approved query parsing, pagination, latest-request protection, active-school reset, safe feedback, and retry in `schoolmaster-frontend/src/composables/admin-system/useStudentProfiles.js`
- [ ] T039 [US1] Implement student profile lifecycle orchestration with status draft state, required reason/effective date validation, pending state, conflict handling, success reconciliation, and stale-response protection in `schoolmaster-frontend/src/composables/admin-system/useStudentProfileLifecycle.js`
- [ ] T040 [P] [US1] Implement student profile form with approved create fields, guardian association input, validation summary, pending state, props down, and submit emits in `schoolmaster-frontend/src/components/admin-system/students/StudentProfileForm.vue`
- [ ] T041 [P] [US1] Implement student profile summary panel with identity, status, guardian-compatible summary, lifecycle history cues, hidden general edit controls, and no transport logic in `schoolmaster-frontend/src/components/admin-system/students/StudentProfileSummaryPanel.vue`
- [ ] T042 [P] [US1] Implement student enrollment status panel with active/inactive status actions, effective date, reason prompt, safe conflict feedback, and props/emits in `schoolmaster-frontend/src/components/admin-system/students/StudentEnrollmentStatusPanel.vue`
- [ ] T043 [US1] Compose student list page with filters, pagination, loading, empty, denied states, create action, and detail navigation in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfilesPage.vue`
- [ ] T044 [US1] Compose student create page with form draft, validation feedback, dirty leave guard, success return, and no undocumented payload fields in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfileCreatePage.vue`
- [ ] T045 [US1] Compose student detail page with summary, enrollment status panel, safe denied/not-found states, hidden general edit controls, and list-query return in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfileDetailPage.vue`
- [ ] T046 [US1] Add student list, create, detail, status, validation, empty, and denial text to `schoolmaster-frontend/src/locales/student-enrollment-roster.js`

**Checkpoint**: User Story 1 is independently functional and testable as MVP.

---

## Phase 4: User Story 2 - Record student transfers safely (Priority: P2)

**Goal**: Authorized school administrators can record approved student transfer behavior from student detail while preserving source-school history and avoiding cross-tenant data exposure.

**Independent Test**: Open an active same-school student profile, submit valid and invalid transfer actions with destination, effective date, and reason inputs, and verify success, validation, unauthorized destination, conflict, and history-preservation states.

### Tests for User Story 2

- [ ] T047 [P] [US2] Add Vitest coverage for transfer request mapping with required `effective_at`, required `reason`, optional destination fields, and no source-school academic or guardian payload fields in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentTransfer.service.spec.js`
- [ ] T048 [P] [US2] Add Vitest coverage for transfer draft state, required date/reason validation, unauthorized destination feedback, conflict feedback, success reconciliation, and stale-response protection in `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useStudentTransfer.spec.js`
- [ ] T049 [P] [US2] Add component tests for transfer dialog destination inputs, required effective date and reason, pending state, validation summary, conflict feedback, and no private source data rendering in `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/StudentTransferDialog.spec.js`
- [ ] T050 [P] [US2] Add page-flow tests for transfer from student detail, success transferred state, historical-only cues, unauthorized destination denial, invalid date/reason validation, and no cross-tenant detail exposure in `schoolmaster-frontend/tests/unit/student-enrollment-roster/pages/StudentTransferFlow.spec.js`

### Implementation for User Story 2

- [ ] T051 [US2] Implement transfer orchestration with draft state, required effective date/reason validation, optional destination fields, pending state, success reconciliation, conflict mapping, and stale-response protection in `schoolmaster-frontend/src/composables/admin-system/useStudentTransfer.js`
- [ ] T052 [P] [US2] Implement transfer confirmation dialog with approved destination metadata inputs, required effective date and reason, safe validation/conflict feedback, props down, and submit/cancel emits in `schoolmaster-frontend/src/components/admin-system/students/StudentTransferDialog.vue`
- [ ] T053 [US2] Integrate student transfer dialog and transferred/historical-only cues into student detail composition without direct service calls in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfileDetailPage.vue`
- [ ] T054 [US2] Add transfer success, validation, unauthorized destination, conflict, historical-only, and source-history-preserved text to `schoolmaster-frontend/src/locales/student-enrollment-roster.js`

**Checkpoint**: User Stories 1 and 2 work independently.

---

## Phase 5: User Story 3 - Maintain class sections and roster membership (Priority: P3)

**Goal**: Authorized school administrators can manage academic-period-scoped class sections, add eligible students to rosters in all-or-nothing batches capped at 100, end memberships in all-or-nothing batches capped at 100, and see dependency conflicts before partial changes are shown.

**Independent Test**: Create or open a roster for an active academic period, add eligible students in a capped batch, attempt duplicate or invalid memberships, end memberships in a capped batch, change roster status where approved, and verify all-or-nothing batch behavior, pagination, empty states, and conflicts.

### Tests for User Story 3

- [ ] T055 [P] [US3] Add Vitest coverage for class-section list/create/detail/update/status service mapping, academic-period and status filters only, metadata field allowlist, duplicate-code conflict, and stale cancellation in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/classSections.service.spec.js`
- [ ] T056 [P] [US3] Add Vitest coverage for roster membership list and batch add/end service mapping, 1 to 100 limits, unique IDs, required dates/reasons, oversized-batch mapping, and all-or-nothing conflict handling in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/rosterMemberships.service.spec.js`
- [ ] T057 [P] [US3] Add Vitest coverage for class-section list/detail composable current-period default, route/query restoration, filters, pagination, inactive period, no-current-period blocked state, and stale-response protection in `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useClassSections.spec.js`
- [ ] T058 [P] [US3] Add Vitest coverage for roster membership batch selection, selected count cap, duplicate selection prevention, batch add validation, batch end validation, all-or-nothing failure retention, and success reconciliation in `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useRosterMemberships.spec.js`
- [ ] T059 [P] [US3] Add component tests for class-section form approved code/name/metadata fields, validation summary, pending state, and no status controls in metadata update mode in `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/ClassSectionForm.spec.js`
- [ ] T060 [P] [US3] Add component tests for roster membership table active/ended status, history rows, pagination emits, empty states, and safe denied/not-found states in `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/RosterMembershipTable.spec.js`
- [ ] T061 [P] [US3] Add component tests for roster membership batch panel add/end selections, selected count accessibility, 100-request cap, required reason/date, conflict feedback, and no partial-success messaging in `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/RosterMembershipBatchPanel.spec.js`
- [ ] T062 [P] [US3] Add page-flow tests for class-section list/create/detail/update/status, current-period route/query restoration, no-current-period blocked state, roster inactivation dependency conflict, batch add/end success, oversized batch, and all-or-nothing failure in `schoolmaster-frontend/tests/unit/student-enrollment-roster/pages/ClassSectionsRosterPages.spec.js`

### Implementation for User Story 3

- [ ] T063 [US3] Implement class-section orchestration with current-period default, route/query selected period, list/detail/create/update/status state, metadata projection, conflict handling, and stale-response protection in `schoolmaster-frontend/src/composables/admin-system/useClassSections.js`
- [ ] T064 [US3] Implement roster membership orchestration with list loading, batch add/end draft state, unique selections, 100-request cap, required date/reason validation, all-or-nothing feedback, and success reconciliation in `schoolmaster-frontend/src/composables/admin-system/useRosterMemberships.js`
- [ ] T065 [P] [US3] Implement class-section form with approved code, name, course, classroom, section, and group metadata fields, props down, and submit emits in `schoolmaster-frontend/src/components/admin-system/class-sections/ClassSectionForm.vue`
- [ ] T066 [P] [US3] Implement class-section summary panel with status, academic period, metadata, lifecycle constraints, dependency conflict feedback, and no transport logic in `schoolmaster-frontend/src/components/admin-system/class-sections/ClassSectionSummaryPanel.vue`
- [ ] T067 [P] [US3] Implement roster membership table with active/ended rows, status tags, pagination, empty states, selected membership emits, and safe feedback slots in `schoolmaster-frontend/src/components/admin-system/class-sections/RosterMembershipTable.vue`
- [ ] T068 [P] [US3] Implement roster membership batch panel with add/end modes, selected count cap, effective date fields, reason field, oversized-batch blocking, all-or-nothing messaging, and submit emits in `schoolmaster-frontend/src/components/admin-system/class-sections/RosterMembershipBatchPanel.vue`
- [ ] T069 [P] [US3] Implement shared batch action dialog for selected count, max count, confirmation copy, validation summary, pending state, and cancel/submit emits in `schoolmaster-frontend/src/components/ui/admin/AdminBatchActionDialog.vue`
- [ ] T070 [P] [US3] Implement shared lifecycle confirm dialog for class-section inactivation date/reason, pending state, validation summary, and conflict feedback in `schoolmaster-frontend/src/components/ui/admin/AdminLifecycleConfirmDialog.vue`
- [ ] T071 [US3] Compose class-section list page with academic-period selector, status filter, pagination, loading, empty, denied states, create action, and route/query restoration in `schoolmaster-frontend/src/pages/admin-system/class-sections/ClassSectionsPage.vue`
- [ ] T072 [US3] Compose class-section create page with approved metadata form, validation, dirty leave guard, success return, and no inactive creation in `schoolmaster-frontend/src/pages/admin-system/class-sections/ClassSectionCreatePage.vue`
- [ ] T073 [US3] Compose class-section detail page with summary, metadata update, inactivation, membership table, membership batch add/end, dependency conflict feedback, and no partial-success display in `schoolmaster-frontend/src/pages/admin-system/class-sections/ClassSectionDetailPage.vue`
- [ ] T074 [US3] Add class-section, roster membership, batch add/end, current-period, oversized-batch, dependency conflict, and no-current-period text to `schoolmaster-frontend/src/locales/student-enrollment-roster.js`

**Checkpoint**: User Stories 1, 2, and 3 work independently.

---

## Phase 6: User Story 4 - Assign teachers to class sections or rosters (Priority: P4)

**Goal**: Authorized school administrators can assign eligible same-school teachers to class sections or rosters, review assignment status, and deactivate assignments while teacher-facing own-assignment routes remain deferred.

**Independent Test**: Assign an eligible teacher to a roster, attempt inactive or cross-school teachers, duplicate assignments, incompatible periods, missing reasons, or unauthorized actors, and verify admin-visible states plus deferred teacher route absence.

### Tests for User Story 4

- [ ] T075 [P] [US4] Add Vitest coverage for teacher-assignment list/create/detail/deactivate service mapping, academic-period and status filters only, tenant headers, duplicate conflict, validation, and stale cancellation in `schoolmaster-frontend/tests/unit/student-enrollment-roster/services/teacherAssignments.service.spec.js`
- [ ] T076 [P] [US4] Add Vitest coverage for teacher-assignment composable admin-only visibility, eligible teacher selection, create validation, deactivation validation, conflict feedback, success reconciliation, and stale-response protection in `schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useTeacherAssignments.spec.js`
- [ ] T077 [P] [US4] Add component tests for teacher assignment panel list, create controls, deactivation controls, forbidden state, duplicate conflict, invalid teacher feedback, and props/emits contract in `schoolmaster-frontend/tests/unit/student-enrollment-roster/components/TeacherAssignmentPanel.spec.js`
- [ ] T078 [P] [US4] Add route tests proving teacher-facing own-assignment routes are absent from Feature 022 route modules while admin assignment routes remain protected in `schoolmaster-frontend/tests/unit/student-enrollment-roster/routes/teacherAssignmentRoutes.spec.js`
- [ ] T079 [P] [US4] Add page-flow tests for teacher assignment from class-section detail, active assignment display, duplicate conflict, invalid effective date, deactivation, forbidden actor, and safe denial feedback in `schoolmaster-frontend/tests/unit/student-enrollment-roster/pages/TeacherAssignmentFlow.spec.js`

### Implementation for User Story 4

- [ ] T080 [US4] Implement teacher-assignment orchestration with admin-only eligibility, selected period, list/detail/create/deactivate state, conflict handling, validation mapping, success reconciliation, and stale-response protection in `schoolmaster-frontend/src/composables/admin-system/useTeacherAssignments.js`
- [ ] T081 [P] [US4] Implement teacher assignment panel with assignment list, teacher selector boundary, create/deactivate controls, effective dates, reason input where required, safe feedback, and no transport logic in `schoolmaster-frontend/src/components/admin-system/class-sections/TeacherAssignmentPanel.vue`
- [ ] T082 [US4] Integrate teacher assignment panel into class-section detail page using `useTeacherAssignments` and no direct service calls in `schoolmaster-frontend/src/pages/admin-system/class-sections/ClassSectionDetailPage.vue`
- [ ] T083 [US4] Ensure Feature 022 route registration does not add teacher-facing own-assignment routes and keeps all teacher-assignment management under admin route metadata in `schoolmaster-frontend/src/router/modules/access-administration.routes.js`
- [ ] T084 [US4] Add teacher assignment labels, eligibility messages, duplicate conflict, deactivation copy, forbidden feedback, and deferred teacher-route text to `schoolmaster-frontend/src/locales/student-enrollment-roster.js`

**Checkpoint**: All user stories are independently functional.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Verification, documentation, accessibility, responsive review, and release readiness across all stories.

- [ ] T085 [P] Run focused student enrollment roster Vitest suite and record results in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T086 [P] Run full frontend unit suite and record `npm run test:unit` result in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T087 [P] Run frontend build check and record `npm run build` result in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T088 [P] Run Redocly validation only if student enrollment or roster OpenAPI contracts changed and record result in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T089 [P] Review student, transfer, class-section, membership batch, and teacher-assignment screens at 390px, 768px, and 1440px and record responsive findings in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T090 [P] Review keyboard navigation, focus order, dialogs, batch selection controls, form labels, status tags, and feedback semantics and record findings in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T091 [P] Review diagnostics and client storage for student, guardian, token, permission, role, reason, full payload, private academic, and cross-tenant leaks and record findings in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T092 Update implementation evidence for operation mapping, permission/capability sources, blocked student profile edit, deferred teacher own-assignment routes, current-period default, route/query period restoration, batch 100-request cap, and stale-response behavior in `specs/022-student-enrollment-roster-ui/quickstart.md`
- [ ] T093 Run representative administrator usability review for student lookup, student status review, roster creation/update, batch membership add/end, and state distinction success criteria and record SC-002, SC-003, and SC-007 results in `specs/022-student-enrollment-roster-ui/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; can start immediately.
- **Foundational (Phase 2)**: Depends on Setup completion; blocks all user stories.
- **User Stories (Phase 3+)**: Depend on Foundational completion.
- **Polish (Phase 7)**: Depends on all desired user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational; MVP.
- **User Story 2 (P2)**: Can start after Foundational and is easiest to integrate into US1 detail page after student detail exists; transfer remains independently testable with mocked detail state.
- **User Story 3 (P3)**: Can start after Foundational; depends on student status semantics from US1 for full eligibility display but can be tested independently with mocked student records.
- **User Story 4 (P4)**: Can start after Foundational and integrates into US3 class-section detail page; admin-only teacher assignment behavior remains independently testable.

### Within Each User Story

- Write Vitest coverage first and confirm it fails before implementation.
- Contract mapper and service tests precede composable/component/page tasks.
- Services and mappers precede composables.
- Composables precede route pages.
- Components receive props and emit events; they do not call services directly.
- Route pages compose components and composables; they do not call Axios directly.
- Update localized text before final page-flow verification.

### Parallel Opportunities

- Setup scaffold tasks T003-T008 can run in parallel.
- Foundational tests T009-T016 can run in parallel by file.
- Foundational implementation tasks T017-T019, T026-T028 can run in parallel after T007-T008.
- US1 tests T031-T037 can run in parallel.
- US1 component tasks T040-T042 can run in parallel after T017 and T039.
- US2 tests T047-T050 can run in parallel.
- US3 tests T055-T062 can run in parallel.
- US3 component/dialog tasks T065-T070 can run in parallel after T018 and T064.
- US4 tests T075-T079 can run in parallel.
- Polish tasks T085-T091 and T093 can run in parallel after implementation.

---

## Parallel Example: User Story 1

```bash
Task: "T031 [P] [US1] Add Vitest coverage for student profile list service parameters in schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentProfilesList.service.spec.js"
Task: "T032 [P] [US1] Add Vitest coverage for student profile create service payload projection in schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentProfileCreate.service.spec.js"
Task: "T033 [P] [US1] Add Vitest coverage for student profile detail and status service mapping in schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentProfileDetailStatus.service.spec.js"
Task: "T034 [P] [US1] Add Vitest coverage for student list query state in schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useStudentProfiles.spec.js"
Task: "T035 [P] [US1] Add Vitest coverage for student status action draft in schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useStudentProfileLifecycle.spec.js"
```

## Parallel Example: User Story 2

```bash
Task: "T047 [P] [US2] Add Vitest coverage for transfer request mapping in schoolmaster-frontend/tests/unit/student-enrollment-roster/services/studentTransfer.service.spec.js"
Task: "T048 [P] [US2] Add Vitest coverage for transfer draft state in schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useStudentTransfer.spec.js"
Task: "T049 [P] [US2] Add component tests for transfer dialog in schoolmaster-frontend/tests/unit/student-enrollment-roster/components/StudentTransferDialog.spec.js"
Task: "T050 [P] [US2] Add page-flow tests for transfer from student detail in schoolmaster-frontend/tests/unit/student-enrollment-roster/pages/StudentTransferFlow.spec.js"
```

## Parallel Example: User Story 3

```bash
Task: "T055 [P] [US3] Add Vitest coverage for class-section services in schoolmaster-frontend/tests/unit/student-enrollment-roster/services/classSections.service.spec.js"
Task: "T056 [P] [US3] Add Vitest coverage for roster membership services in schoolmaster-frontend/tests/unit/student-enrollment-roster/services/rosterMemberships.service.spec.js"
Task: "T057 [P] [US3] Add Vitest coverage for class-section composable in schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useClassSections.spec.js"
Task: "T058 [P] [US3] Add Vitest coverage for roster membership composable in schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useRosterMemberships.spec.js"
Task: "T061 [P] [US3] Add component tests for roster membership batch panel in schoolmaster-frontend/tests/unit/student-enrollment-roster/components/RosterMembershipBatchPanel.spec.js"
```

## Parallel Example: User Story 4

```bash
Task: "T075 [P] [US4] Add Vitest coverage for teacher-assignment services in schoolmaster-frontend/tests/unit/student-enrollment-roster/services/teacherAssignments.service.spec.js"
Task: "T076 [P] [US4] Add Vitest coverage for teacher-assignment composable in schoolmaster-frontend/tests/unit/student-enrollment-roster/composables/useTeacherAssignments.spec.js"
Task: "T077 [P] [US4] Add component tests for teacher assignment panel in schoolmaster-frontend/tests/unit/student-enrollment-roster/components/TeacherAssignmentPanel.spec.js"
Task: "T078 [P] [US4] Add route tests proving teacher-facing own-assignment routes are absent in schoolmaster-frontend/tests/unit/student-enrollment-roster/routes/teacherAssignmentRoutes.spec.js"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup.
2. Complete Phase 2 foundation.
3. Complete Phase 3 User Story 1.
4. Stop and validate student list/create/detail/status independently.
5. Demo MVP with no transfer, roster, membership, or teacher-assignment UI enabled yet.

### Incremental Delivery

1. Setup + Foundation: shared contracts, services, routes, feedback, period scope, diagnostics.
2. US1: student profile list/create/detail/status.
3. US2: student transfer from student detail.
4. US3: class sections and roster membership batches.
5. US4: admin teacher assignment controls.
6. Polish: full verification and evidence capture.

### Parallel Team Strategy

With multiple implementers:

1. Team completes Setup and Foundational phases together.
2. After Foundation, one implementer can own US1, one can start US3 service/composable work with fixtures, and one can build US4 tests/components against mocked assignment data.
3. US2 integrates into US1 detail when available but can keep service/composable tests independent.
4. Merge in priority order to preserve MVP path.

---

## Notes

- [P] tasks change different files and can run in parallel after their prerequisites.
- Every user-story task includes a concrete file path.
- Backend and OpenAPI tasks are intentionally limited to verification/evidence because this slice is frontend-only.
- General student profile edit and teacher-facing own-assignment routes remain blocked by contract and roadmap scope.
