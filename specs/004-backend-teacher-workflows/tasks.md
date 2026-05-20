# Tasks: Backend Teacher Workflow Foundation

**Input**: Design documents from `specs/004-backend-teacher-workflows/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/backend-teacher-workflows.md`, `quickstart.md`  
**Feature ID**: `004-backend-teacher-workflows`

**Tests**: Required. This slice changes critical backend behavior for tenant isolation, authorization, file upload safety, malware-scan gating, active academic-period enforcement, selected-student validation, and response envelopes.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared foundations are in place.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Task can run in parallel with other marked tasks in the same phase when prerequisites are met.
- **[Story]**: User-story label for story phases only.
- Every task includes an exact target file path relative to the `schoolmaster-backend` repository root.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Align backend implementation with the approved teacher workflow specification and contract before code changes.

- [X] T001 Create feature implementation notes with approved operation IDs in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T002 [P] Verify the backend specs mount references `specs/specs/004-backend-teacher-workflows/quickstart.md` in `AGENTS.md`
- [X] T003 Run contract validation and record the result in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T004 Confirm no undocumented teacher workflow routes exist and document the route inventory in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T005 Record blocked contract gaps for folder CRUD, downloads, updates, corrections, student self-service, reports, and classroom/course workflows in `docs/implementation-notes/004-backend-teacher-workflows.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared backend persistence, permissions, file handling, scan-status, tenant, and response infrastructure required by every teacher workflow story.

**Critical**: No user story work should begin until this phase is complete.

- [X] T006 Inspect existing backend migrations for `TeacherContentFolder`, `TeacherContentItem`, `Questionnaire`, `QuestionnaireQuestion`, `LearningSet`, `LearningSetEntry`, `LearningSetAssignment`, `GradeRecord`, and `AttendanceRecord` UUID/status/tenant/soft-delete columns and document the inventory in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T007 Add only missing teacher workflow persistence changes with tenant indexes and status fields in `database/migrations/2026_05_18_000001_create_teacher_workflow_tables.php`
- [X] T008 [P] Seed baseline teacher workflow permission definitions for content, questionnaires, learning sets, grades, and attendance in `database/seeders/PermissionSeeder.php`
- [X] T009 [P] Configure private tenant-scoped teacher content storage without public URLs in `config/filesystems.php`
- [X] T010 [P] Define teacher workflow authorization helper methods in `app/Services/Concerns/AuthorizesTeacherWorkflows.php`
- [X] T011 [P] Define shared teacher workflow list-query validation rules in `app/Services/TeacherWorkflows/TeacherWorkflowListQuery.php`
- [X] T012 [P] Define upload type, size, detected-content, filename, metadata, and storage-path sanitization helper in `app/Services/TeacherContent/TeacherContentUploadValidator.php`
- [X] T013 [P] Define malware scan status transition service in `app/Services/TeacherContent/TeacherContentScanService.php`
- [X] T014 [P] Define asynchronous scan-status job boundary in `app/Jobs/ProcessTeacherContentScan.php`
- [X] T015 [P] Add teacher workflow model factories for tests in `database/factories/TeacherWorkflowFactory.php`
- [X] T016 Define the teacher workflow route group with auth and tenant middleware only, without operation-specific route registration, in `routes/api.php`

**Checkpoint**: Persistence, permissions, private storage, scan-status, authorization helpers, and route boundaries are ready for story work.

---

## Phase 3: User Story 1 - Manage Teacher Content and Questionnaires (Priority: P1) MVP

**Goal**: A teacher can upload instructional content, keep it unavailable until malware scanning marks it clean, create questionnaires with v1-supported question types, and list content and questionnaires inside one active school tenant.

**Independent Test**: Authenticate as a teacher with an active resolved school context, upload one allowed instructional file, verify it is unavailable until the malware scan marks it clean, create one questionnaire using only v1 question types, and confirm both lists remain tenant-scoped.

### Tests for User Story 1

- [X] T017 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/teacher-content` and `POST /api/v1/teacher-content` in `tests/Feature/Api/V1/TeacherContentManagementTest.php`
- [X] T018 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/questionnaires` and `POST /api/v1/questionnaires` in `tests/Feature/Api/V1/QuestionnaireManagementTest.php`
- [X] T019 [P] [US1] Add unit tests for upload type, size, executable/archive, declared/detected content-type, filename, metadata, and storage-path sanitization in `tests/Unit/Services/TeacherContentUploadValidationTest.php`
- [X] T020 [P] [US1] Add unit tests for malware scan status transitions and availability gating in `tests/Unit/Services/TeacherContentScanStatusTest.php`
- [X] T021 [P] [US1] Add unit tests for questionnaire question type and sequence validation in `tests/Unit/Services/QuestionnaireValidationTest.php`

### Implementation for User Story 1

- [X] T022 [P] [US1] Implement or verify internal/pre-existing `TeacherContentFolder` model relationships and same-school `folder_id` validation support in `app/Models/TeacherContentFolder.php`
- [X] T023 [P] [US1] Implement `TeacherContentItem` model relationships, school scope, and scan-status helpers in `app/Models/TeacherContentItem.php`
- [X] T024 [P] [US1] Implement `Questionnaire` model relationships and school scope in `app/Models/Questionnaire.php`
- [X] T025 [P] [US1] Implement `QuestionnaireQuestion` model relationships and sequence helpers in `app/Models/QuestionnaireQuestion.php`
- [X] T026 [US1] Implement teacher content creation input DTO in `app/DTOs/TeacherContent/CreateTeacherContentData.php`
- [X] T027 [US1] Implement questionnaire creation input DTO in `app/DTOs/Questionnaires/CreateQuestionnaireData.php`
- [X] T028 [P] [US1] Implement teacher content upload validation request in `app/Http/Requests/TeacherContent/CreateTeacherContentRequest.php`
- [X] T029 [P] [US1] Implement questionnaire validation request in `app/Http/Requests/Questionnaires/CreateQuestionnaireRequest.php`
- [X] T030 [P] [US1] Implement teacher content response shape in `app/Http/Resources/TeacherContentResource.php`
- [X] T031 [P] [US1] Implement questionnaire response shape in `app/Http/Resources/QuestionnaireResource.php`
- [X] T032 [P] [US1] Implement questionnaire question response shape in `app/Http/Resources/QuestionnaireQuestionResource.php`
- [X] T033 [P] [US1] Implement teacher content authorization policy in `app/Policies/TeacherContentPolicy.php`
- [X] T034 [P] [US1] Implement questionnaire authorization policy in `app/Policies/QuestionnairePolicy.php`
- [X] T035 [US1] Implement teacher content listing, upload, private storage, and scan-status service in `app/Services/TeacherContent/TeacherContentService.php`
- [X] T036 [US1] Implement questionnaire listing and creation service in `app/Services/Questionnaires/QuestionnaireService.php`
- [X] T037 [US1] Implement teacher content list/create controller actions in `app/Http/Controllers/Api/V1/TeacherContentController.php`
- [X] T038 [US1] Implement questionnaire list/create controller actions in `app/Http/Controllers/Api/V1/QuestionnaireController.php`
- [X] T039 [US1] Wire `listTeacherContent`, `createTeacherContent`, `listQuestionnaires`, and `createQuestionnaire` routes in `routes/api.php`

**Checkpoint**: User Story 1 is independently functional and can be validated without learning set, grade, attendance, student self-service, or reporting workflows.

---

## Phase 4: User Story 2 - Publish Learning Sets for Selected Students (Priority: P2)

**Goal**: A teacher can create learning sets for an active academic period using ordered clean content/questionnaire entries and direct assignment to selected active same-school student profiles.

**Independent Test**: Create clean content and an active questionnaire in one school, create a learning set for an active academic period with ordered entries, assign it to selected active `StudentProfile` records from the same school, and verify cross-tenant, inactive, unclean, or wrong-period references reject the request atomically.

### Tests for User Story 2

- [X] T040 [P] [US2] Add PHPUnit feature tests for `GET /api/v1/learning-sets` and `POST /api/v1/learning-sets` in `tests/Feature/Api/V1/LearningSetManagementTest.php`
- [X] T041 [P] [US2] Add unit tests for clean-content and active-questionnaire entry validation in `tests/Unit/Services/LearningSetEntryValidationTest.php`
- [X] T042 [P] [US2] Add unit tests for direct selected-student assignment validation in `tests/Unit/Services/LearningSetAssignmentValidationTest.php`
- [X] T043 [P] [US2] Add unit tests for atomic learning set creation rollback behavior in `tests/Unit/Services/LearningSetAtomicCreationTest.php`

### Implementation for User Story 2

- [X] T044 [P] [US2] Implement `LearningSet` model relationships and school scope in `app/Models/LearningSet.php`
- [X] T045 [P] [US2] Implement `LearningSetEntry` model relationships and sequence helpers in `app/Models/LearningSetEntry.php`
- [X] T046 [P] [US2] Implement `LearningSetAssignment` model relationships and school scope in `app/Models/LearningSetAssignment.php`
- [X] T047 [US2] Implement learning set creation input DTO in `app/DTOs/LearningSets/CreateLearningSetData.php`
- [X] T048 [P] [US2] Implement learning set validation request in `app/Http/Requests/LearningSets/CreateLearningSetRequest.php`
- [X] T049 [P] [US2] Implement learning set response shape in `app/Http/Resources/LearningSetResource.php`
- [X] T050 [P] [US2] Implement learning set entry response shape in `app/Http/Resources/LearningSetEntryResource.php`
- [X] T051 [P] [US2] Implement learning set assignment response shape in `app/Http/Resources/LearningSetAssignmentResource.php`
- [X] T052 [P] [US2] Implement learning set authorization policy in `app/Policies/LearningSetPolicy.php`
- [X] T053 [US2] Implement learning set entry validation service for clean content and active questionnaires in `app/Services/LearningSets/LearningSetEntryValidator.php`
- [X] T054 [US2] Implement selected-student assignment validation service in `app/Services/LearningSets/LearningSetAssignmentValidator.php`
- [X] T055 [US2] Implement learning set listing and atomic creation service in `app/Services/LearningSets/LearningSetService.php`
- [X] T056 [US2] Implement learning set list/create controller actions in `app/Http/Controllers/Api/V1/LearningSetController.php`
- [X] T057 [US2] Wire `listLearningSets` and `createLearningSet` routes in `routes/api.php`

**Checkpoint**: User Story 2 is independently functional after shared foundations and US1 content/questionnaire prerequisites.

---

## Phase 5: User Story 3 - Record Grades and Attendance (Priority: P3)

**Goal**: A teacher can record and list grades and attendance for active same-school student profiles during an active academic period.

**Independent Test**: Authenticate as a teacher, resolve an active school and academic period, record one grade and one attendance entry for an active same-school student profile, list the records, and verify inactive, cross-tenant, closed-period, or invalid-value attempts are rejected.

### Tests for User Story 3

- [X] T058 [P] [US3] Add PHPUnit feature tests for `GET /api/v1/grades` and `POST /api/v1/grades` in `tests/Feature/Api/V1/GradeManagementTest.php`
- [X] T059 [P] [US3] Add PHPUnit feature tests for `GET /api/v1/attendance` and `POST /api/v1/attendance` in `tests/Feature/Api/V1/AttendanceManagementTest.php`
- [X] T060 [P] [US3] Add unit tests for grade value, student profile, teacher recorder, and active-period validation in `tests/Unit/Services/GradeRecordValidationTest.php`
- [X] T061 [P] [US3] Add unit tests for attendance status, student profile, teacher recorder, and active-period validation in `tests/Unit/Services/AttendanceRecordValidationTest.php`

### Implementation for User Story 3

- [X] T062 [P] [US3] Implement `GradeRecord` model relationships and school scope in `app/Models/GradeRecord.php`
- [X] T063 [P] [US3] Implement `AttendanceRecord` model relationships and school scope in `app/Models/AttendanceRecord.php`
- [X] T064 [US3] Implement grade creation input DTO in `app/DTOs/Grades/CreateGradeData.php`
- [X] T065 [US3] Implement attendance creation input DTO in `app/DTOs/Attendance/CreateAttendanceData.php`
- [X] T066 [P] [US3] Implement grade validation request in `app/Http/Requests/Grades/CreateGradeRequest.php`
- [X] T067 [P] [US3] Implement attendance validation request in `app/Http/Requests/Attendance/CreateAttendanceRequest.php`
- [X] T068 [P] [US3] Implement grade response shape in `app/Http/Resources/GradeRecordResource.php`
- [X] T069 [P] [US3] Implement attendance response shape in `app/Http/Resources/AttendanceRecordResource.php`
- [X] T070 [P] [US3] Implement grade authorization policy in `app/Policies/GradeRecordPolicy.php`
- [X] T071 [P] [US3] Implement attendance authorization policy in `app/Policies/AttendanceRecordPolicy.php`
- [X] T072 [US3] Implement shared active-period and same-school student validation service in `app/Services/TeacherWorkflows/AcademicRecordTargetValidator.php`
- [X] T073 [US3] Implement grade listing and creation service in `app/Services/Grades/GradeRecordService.php`
- [X] T074 [US3] Implement attendance listing and creation service in `app/Services/Attendance/AttendanceRecordService.php`
- [X] T075 [US3] Implement grade list/create controller actions in `app/Http/Controllers/Api/V1/GradeController.php`
- [X] T076 [US3] Implement attendance list/create controller actions in `app/Http/Controllers/Api/V1/AttendanceController.php`
- [X] T077 [US3] Wire `listGrades`, `createGrade`, `listAttendance`, and `createAttendance` routes in `routes/api.php`

**Checkpoint**: User Story 3 is independently functional after shared foundations and existing school-admin student profile and academic period data.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final contract compliance, verification, documentation, and cleanup across all selected stories.

- [X] T078 Run backend PHP syntax checks and record result in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T079 Run backend style checks and record result in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T080 Run backend PHPUnit suite and record result in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T081 Run Redocly validation and record result in `docs/implementation-notes/004-backend-teacher-workflows.md`
- [X] T082 [P] Add response-shape regression coverage for all ten teacher workflow operation IDs, including success, validation for create operations, unauthorized, and tenant-mismatch outcomes exactly as declared by OpenAPI for each operation, in `tests/Feature/Api/V1/TeacherWorkflowResponseShapeTest.php`
- [X] T083 [P] Add validation-contract regression coverage for undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, upload errors, and invalid references across all ten operation IDs in `tests/Feature/Api/V1/TeacherWorkflowValidationContractTest.php`
- [X] T084 [P] Add tenant-isolation regression coverage across teacher content, questionnaires, learning sets, grades, and attendance in `tests/Feature/Api/V1/TeacherWorkflowTenantIsolationTest.php`
- [X] T085 [P] Add blocked-operation regression coverage for folder CRUD, downloads, updates, corrections, student self-service, reports, and classroom/course routes in `tests/Feature/Api/V1/TeacherWorkflowBlockedOperationsTest.php`
- [X] T086 [P] Add end-to-end teacher workflow happy-path coverage from clean content and questionnaire through learning set assignment, grade creation, and attendance creation in `tests/Feature/Api/V1/TeacherWorkflowHappyPathTest.php`
- [X] T087 Review implemented backend routes against the blocked-operation list in `routes/api.php`
- [X] T088 Update implementation notes with final operation IDs, test commands, storage behavior, scan-status behavior, and blocked follow-up contract gaps in `docs/implementation-notes/004-backend-teacher-workflows.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks every user story.
- **Phase 3 US1**: Depends on Phase 2; recommended MVP.
- **Phase 4 US2**: Depends on Phase 2 and US1 content/questionnaire availability.
- **Phase 5 US3**: Depends on Phase 2 and existing `StudentProfile`/`AcademicPeriod` data from school-admin foundations.
- **Phase 6 Polish**: Depends on all selected user stories for the implementation increment.

### User Story Dependencies

- **US1 Manage teacher content and questionnaires**: First recommended story because learning sets depend on clean content and valid questionnaires.
- **US2 Publish learning sets for selected students**: Depends on US1 content/questionnaire records and active academic periods/student profiles from earlier backend slices.
- **US3 Record grades and attendance**: Independent of US1 and US2 after Phase 2, but depends on active same-school academic periods and student profiles from school-admin foundations.

### Within Each User Story

- Tests should be written first and fail before implementation.
- Model and persistence verification precedes DTOs, requests, services, and controllers.
- Services own business rules before controllers wire routes.
- Policies and resources must be in place before marking a route complete.
- Routes must not expose blocked operations absent from OpenAPI.

## Parallel Opportunities

- T002 can run in parallel with the implementation-notes setup sequence.
- T008 through T015 can run in parallel after T006 and T007 establish the persistence baseline.
- Test files within each user story can be written in parallel.
- Request, resource, and policy files within a story can be implemented in parallel after model relationships are verified.
- US3 can run alongside US2 after Phase 2 if separate implementers avoid editing `routes/api.php` at the same time.
- T082, T083, T084, T085, and T086 can run in parallel during polish.

## Parallel Example: User Story 1

```text
Task: "T017 Add PHPUnit feature tests for GET/POST /api/v1/teacher-content in tests/Feature/Api/V1/TeacherContentManagementTest.php"
Task: "T018 Add PHPUnit feature tests for GET/POST /api/v1/questionnaires in tests/Feature/Api/V1/QuestionnaireManagementTest.php"
Task: "T019 Add unit tests for upload validation in tests/Unit/Services/TeacherContentUploadValidationTest.php"
Task: "T020 Add unit tests for scan status in tests/Unit/Services/TeacherContentScanStatusTest.php"
Task: "T021 Add unit tests for questionnaire validation in tests/Unit/Services/QuestionnaireValidationTest.php"
```

## Parallel Example: User Story 2

```text
Task: "T044 Implement LearningSet model in app/Models/LearningSet.php"
Task: "T045 Implement LearningSetEntry model in app/Models/LearningSetEntry.php"
Task: "T046 Implement LearningSetAssignment model in app/Models/LearningSetAssignment.php"
Task: "T048 Implement learning set validation request in app/Http/Requests/LearningSets/CreateLearningSetRequest.php"
Task: "T052 Implement learning set authorization policy in app/Policies/LearningSetPolicy.php"
```

## Parallel Example: User Story 3

```text
Task: "T058 Add PHPUnit feature tests for GET/POST /api/v1/grades in tests/Feature/Api/V1/GradeManagementTest.php"
Task: "T059 Add PHPUnit feature tests for GET/POST /api/v1/attendance in tests/Feature/Api/V1/AttendanceManagementTest.php"
Task: "T062 Implement GradeRecord model in app/Models/GradeRecord.php"
Task: "T063 Implement AttendanceRecord model in app/Models/AttendanceRecord.php"
Task: "T072 Implement shared academic record target validator in app/Services/TeacherWorkflows/AcademicRecordTargetValidator.php"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 only.
3. Validate US1 independently with teacher content, questionnaire, upload, scan-status, tenant, and response-shape tests.
4. Stop before US2 or US3 if the backend needs an early review checkpoint.

### Incremental Delivery

1. Deliver US1 for teacher content uploads and questionnaires.
2. Deliver US2 for learning sets and selected-student assignments.
3. Deliver US3 for grades and attendance.
4. Run Phase 6 after the selected story set is complete.

### Scope Guardrails

- Do not add public content-folder CRUD, teacher downloads, student downloads, detail, update, deactivate, delete, restore, bulk import, or correction routes.
- Do not add student self-service, reporting, classroom, course, section, group, roster, or teacher assignment workflows.
- Do not add request fields, response fields, filters, sort options, or error envelopes absent from OpenAPI.
- Do not treat platform administrator access as an implicit school-scoped teacher workflow bypass.
- Do not expose uploaded files from public storage or before malware scan status is `clean`.
