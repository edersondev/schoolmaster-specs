# Tasks: Backend Teacher Workflow Lifecycle and Corrections

**Input**: Design documents from `specs/specs/010-teacher-workflow-lifecycle/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/backend-teacher-workflow-lifecycle.md, quickstart.md

**Tests**: REQUIRED by FR-019 and quickstart.md. Use OpenAPI linting plus Laravel PHPUnit feature/unit coverage. No frontend tasks are included because this feature is backend-only.

**Organization**: Tasks are grouped by user story to enable independently testable increments after shared foundations are complete.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or does not depend on incomplete tasks
- **[Story]**: Which user story this task belongs to (US1, US2, US3, US4)
- Every task includes exact target paths

## Path Conventions

- Run implementation from the `schoolmaster-backend` repository root.
- Backend paths are relative to `schoolmaster-backend/`.
- Specification and OpenAPI paths use the backend repository's `specs` symlink to `../schoolmaster-specs`.
- Contract changes must be made in both `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` before matching backend routes are exposed.
- Frontend paths are intentionally absent because frontend implementation is outside this slice.

---

## Phase 1: Setup (Contract and Repository Alignment)

**Purpose**: Establish the OpenAPI-approved API surface and implementation boundary before backend routes exist.

- [X] T001 Update aggregate OpenAPI with teacher content, questionnaire, learning-set, grade, attendance, grade/attendance correction, import, lifecycle, restore, delete, and download operation IDs, request schemas, response schemas, tenant errors, validation errors, conflict errors, editable/immutable field matrices, lifecycle states, correction reason rules, import limits, student visibility notes, and authorization notes in `specs/api/openapi.yaml`
- [X] T002 Mirror the same teacher workflow lifecycle, grade/attendance correction, and import OpenAPI operations, schemas, response envelopes, errors, editable/immutable field matrices, lifecycle states, correction rules, import rules, student visibility notes, and authorization notes in `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T003 [P] Add backend implementation notes for teacher workflow lifecycle, correction, import, download, audit, and excluded behavior in `specs/docs/backend-guidelines.md`
- [X] T004 [P] Add or update multi-tenant guidance for `school_id` teacher workflow records, `X-School-Id` ordering, owner authority, school-administrator override, and platform-access denial in `specs/docs/multi-tenant.md`
- [X] T005 [P] Add or update security guidance for private teacher content downloads, scan gating, denied-download audits, correction reason privacy, and import error-summary minimization in `specs/docs/security.md`
- [X] T006 Run `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` from `specs/` and record the result in `specs/specs/010-teacher-workflow-lifecycle/quickstart.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared backend infrastructure that must exist before any user story route can be implemented.

**CRITICAL**: No user story route work begins until this phase is complete.

- [X] T007 Create teacher workflow route group placeholders protected by existing API auth and school-context middleware in `routes/api.php`
- [X] T008 [P] Create shared lifecycle DTOs for status transitions, delete, and restore inputs in `app/DTOs/TeacherWorkflow/LifecycleInput.php`
- [X] T009 [P] Create shared teacher workflow lookup repository for content, questionnaires, learning sets, grades, attendance, rosters, memberships, teacher assignments, students, teachers, academic periods, and school-owned dependency checks in `app/Repositories/TeacherWorkflow/TeacherWorkflowLookupRepository.php`
- [X] T010 Create school-context guard that fails before teacher workflow lookup, private file access, roster lookup, student lookup, teacher lookup, academic-period lookup, authorization, validation, persistence, import processing, audit, or response shaping in `app/Services/TeacherWorkflow/SchoolContextGuard.php`
- [X] T011 [P] Create shared teacher workflow authorization helper for creator/owner authority, school-administrator same-school override, same-school non-owner teacher denial, student/guardian denial, and platform-without-school-context denial in `app/Policies/TeacherWorkflowPolicy.php`
- [X] T012 [P] Create tenant-safe audit writer for lifecycle, update, delete, restore, correction, import, download, conflict, validation, and blocked cross-tenant outcomes in `app/Services/TeacherWorkflow/TeacherWorkflowAuditLogger.php`
- [X] T013 [P] Create shared lifecycle transition service for `active`, `inactive`, and `deleted` states, restore-to-`inactive`, activation dependency validation hooks, invalid transition conflicts, and transaction boundaries in `app/Services/TeacherWorkflow/LifecycleTransitionService.php`
- [X] T014 [P] Create shared student visibility projector that returns active current records only and hides inactive/deleted records except explicitly documented historical labels in `app/Services/TeacherWorkflow/StudentVisibilityProjector.php`
- [X] T015 [P] Create shared response-resource base helpers for documented success, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, import-validation, correction-history, download-denial, and restore/status envelopes in `app/Http/Resources/TeacherWorkflow/BaseTeacherWorkflowResource.php`
- [X] T016 [P] Create shared request validation helpers for rejecting undocumented fields, unsupported lifecycle states, unsupported filters, unsupported include expansion, unsupported sorts, UUID mismatches, and cross-tenant references in `app/Http/Requests/TeacherWorkflow/Concerns/ValidatesTeacherWorkflowRequests.php`
- [X] T017 [P] Add or update AuditEvent model support for teacher workflow actions, reason category, target identifiers, tenant-safe metadata, and no private payload storage in `app/Models/AuditEvent.php`
- [X] T018 [P] Add foundational unit coverage for school-context guard ordering and fail-before-lookup behavior in `tests/Unit/TeacherWorkflow/SchoolContextGuardTest.php`
- [X] T019 [P] Add foundational unit coverage for lifecycle transition rules, restore-to-inactive behavior, invalid transition conflicts, and activation hook invocation in `tests/Unit/TeacherWorkflow/LifecycleTransitionServiceTest.php`
- [X] T020 [P] Add foundational unit coverage for audit metadata minimization, blocked cross-tenant summaries, successful download events, and denied download events in `tests/Unit/TeacherWorkflow/TeacherWorkflowAuditLoggerTest.php`
- [X] T021 Register teacher workflow policies and shared services with the existing Laravel provider conventions in `app/Providers/AppServiceProvider.php`

**Checkpoint**: Foundation ready. User story implementation can proceed.

---

## Phase 3: User Story 1 - Maintain Teacher Materials (Priority: P1) MVP

**Goal**: Authorized creating/owning teachers and school administrators can view, update, activate/deactivate, soft-delete, restore, and download clean teacher content and questionnaires within an active school context.

**Independent Test**: Authenticate with an active permitted school context, create clean content and a questionnaire, retrieve details, update editable fields, deactivate active records, reactivate inactive records, soft-delete eligible records, restore deleted records to `inactive`, download only clean authorized content, and verify cross-tenant or unclean content remains inaccessible.

### Tests for User Story 1

- [X] T022 [P] [US1] Add OpenAPI contract assertions and documented success/error response-shape checks for `getTeacherContent`, `updateTeacherContent`, `updateTeacherContentStatus`, `deleteTeacherContent`, `restoreTeacherContent`, `downloadTeacherContent`, `getQuestionnaire`, `updateQuestionnaire`, `updateQuestionnaireStatus`, `deleteQuestionnaire`, and `restoreQuestionnaire` in `tests/Feature/TeacherWorkflow/TeacherMaterialsContractTest.php`
- [X] T023 [P] [US1] Add tenant isolation and authorization feature tests for teacher owner access, school-administrator same-school access, same-school non-owner teacher denial, student/guardian denial, platform-without-school-context denial, and cross-tenant not-found behavior in `tests/Feature/TeacherWorkflow/TeacherMaterialsAuthorizationTest.php`
- [X] T024 [P] [US1] Add lifecycle, restore, immutable-field, unsupported-field, unsupported-status, inactive-dependency, deleted-dependency, and validation feature tests for teacher content and questionnaires in `tests/Feature/TeacherWorkflow/TeacherMaterialsLifecycleValidationTest.php`
- [X] T025 [P] [US1] Add teacher content download tests for clean scan success, pending scan denial, failed scan denial, inactive denial, deleted denial, cross-tenant denial, unauthorized denial, tenant-safe delivery, and private path suppression in `tests/Feature/TeacherWorkflow/TeacherContentDownloadTest.php`
- [X] T026 [P] [US1] Add content and questionnaire historical-meaning edit rejection tests after use by a published or assigned learning set in `tests/Feature/TeacherWorkflow/TeacherMaterialsHistoricalMeaningTest.php`
- [X] T027 [P] [US1] Add successful and denied teacher content download audit tests verifying actor, outcome, target when resolved, school when resolved, safe denial category, and absence of private file contents, storage paths, credentials, full payloads, and unauthorized cross-tenant details in `tests/Feature/TeacherWorkflow/TeacherContentDownloadAuditTest.php`

### Implementation for User Story 1

- [X] T028 [US1] Add or update teacher content and questionnaire persistence for lifecycle status, soft-delete metadata, restore metadata, scan-status eligibility, owner identity, and indexes required by documented operations in `database/migrations/2026_06_01_000001_update_teacher_materials_lifecycle_columns.php`
- [X] T029 [P] [US1] Update teacher content model with lifecycle casts, owner relationship, school relationship, learning-set-entry relationships, audit relationships, soft-delete metadata, and UUID route binding in `app/Models/TeacherContentItem.php`
- [X] T030 [P] [US1] Update questionnaire and questionnaire-question models with lifecycle casts, owner relationships, school relationship, ordered question relationships, learning-set-entry relationships, audit relationships, and UUID route binding in `app/Models/Questionnaire.php` and `app/Models/QuestionnaireQuestion.php`
- [X] T031 [P] [US1] Update teacher content and questionnaire factories for clean, pending-scan, failed-scan, active, inactive, deleted, used, owner, and cross-tenant states in `database/factories/TeacherContentItemFactory.php` and `database/factories/QuestionnaireFactory.php`
- [X] T032 [P] [US1] Create teacher material Form Requests for detail, update, status, delete, restore, and download actions in `app/Http/Requests/TeacherWorkflow/TeacherMaterialsRequest.php`
- [X] T033 [P] [US1] Create teacher content and questionnaire API Resources for detail, status, restore, delete, audit-history-safe, and download metadata envelopes in `app/Http/Resources/TeacherWorkflow/TeacherMaterialsResource.php`
- [X] T034 [US1] Implement teacher content policy rules for owner management, school-administrator same-school management, same-school non-owner denial, student/guardian denial, platform denial, scan-gated download authority, and deleted/inactive handling in `app/Policies/TeacherContentItemPolicy.php`
- [X] T035 [US1] Implement questionnaire policy rules for owner management, school-administrator same-school management, same-school non-owner denial, student/guardian denial, platform denial, and deleted/inactive handling in `app/Policies/QuestionnairePolicy.php`
- [X] T036 [US1] Implement historical-meaning guard for used content and questionnaires, published or assigned learning-set dependencies, immutable student-facing fields, and v1 no-versioning rejection in `app/Services/TeacherWorkflow/HistoricalMeaningGuard.php`
- [X] T037 [US1] Implement teacher content service for detail, editable-field update, lifecycle status, delete, restore-to-inactive, clean-scan download authorization, private delivery, dependency conflicts, audit events, and transaction boundaries in `app/Services/TeacherWorkflow/TeacherContentService.php`
- [X] T038 [US1] Implement questionnaire service for detail, editable-field update, supported v1 question types, ordered question preservation, lifecycle status, delete, restore-to-inactive, historical-meaning rejection, dependency conflicts, audit events, and transaction boundaries in `app/Services/TeacherWorkflow/QuestionnaireService.php`
- [X] T039 [US1] Implement teacher content and questionnaire controllers mapped only to approved OpenAPI operation IDs in `app/Http/Controllers/Api/V1/TeacherMaterialsController.php`
- [X] T040 [US1] Register only OpenAPI-approved teacher content and questionnaire detail, update, status, delete, restore, and download routes in `routes/api.php`
- [X] T041 [US1] Run `docker exec schoolmaster-backend-app-1 php artisan test --filter='TeacherMaterials|TeacherContentDownload|Questionnaire'` and fix failures in `app/Services/TeacherWorkflow/`, `app/Http/Controllers/Api/V1/TeacherMaterialsController.php`, or `tests/Feature/TeacherWorkflow/`

**Checkpoint**: User Story 1 is independently functional and testable as the MVP.

---

## Phase 4: User Story 2 - Maintain Roster-Aware Learning Sets (Priority: P2)

**Goal**: Authorized creating/owning teachers and school administrators can retrieve, update, activate/deactivate, soft-delete, and restore roster-aware learning sets without creating new direct selected-student assignments.

**Independent Test**: Create a class section with active roster memberships and an active teacher assignment, create or maintain a learning set for that roster, update editable learning-set fields, deactivate an active learning set, restore a deleted learning set to `inactive`, and confirm existing direct student assignments remain readable only as legacy records.

### Tests for User Story 2

- [X] T042 [P] [US2] Add OpenAPI contract assertions and documented success/error response-shape checks for `getLearningSet`, `updateLearningSet`, `updateLearningSetStatus`, `deleteLearningSet`, and `restoreLearningSet` in `tests/Feature/TeacherWorkflow/LearningSetContractTest.php`
- [X] T043 [P] [US2] Add tenant isolation and authorization tests for teacher owner access, school-administrator same-school access, same-school non-owner teacher denial, inactive teacher-assignment denial, student/guardian denial, platform-without-school-context denial, and cross-tenant not-found behavior in `tests/Feature/TeacherWorkflow/LearningSetAuthorizationTest.php`
- [X] T044 [P] [US2] Add roster-aware assignment validation tests for active same-school roster membership writes, inactive membership rejection, cross-tenant roster rejection, deleted roster rejection, inactive academic-period rejection, unclean content rejection, deleted content rejection, unsupported direct selected-student assignment write rejection, and no partial state changes in `tests/Feature/TeacherWorkflow/LearningSetRosterAssignmentTest.php`
- [X] T045 [P] [US2] Add legacy direct selected-student assignment read-compatibility tests that confirm existing records remain readable without permitting new direct-assignment writes in `tests/Feature/TeacherWorkflow/LearningSetLegacyAssignmentCompatibilityTest.php`
- [X] T046 [P] [US2] Add learning-set lifecycle, restore-to-inactive, immutable-field, published-student-visible update, dependency conflict, and audit-history preservation tests in `tests/Feature/TeacherWorkflow/LearningSetLifecycleValidationTest.php`

### Implementation for User Story 2

- [X] T047 [US2] Add or update learning-set and learning-set-assignment persistence for lifecycle status, soft-delete metadata, roster-aware assignment references, legacy direct-assignment read markers, restore metadata, and indexes required by documented operations in `database/migrations/2026_06_01_000002_update_learning_sets_lifecycle_and_roster_assignments.php`
- [X] T048 [P] [US2] Update LearningSet, LearningSetEntry, and LearningSetAssignment models with lifecycle casts, owner relationships, school relationship, academic-period relationship, roster assignment relationships, legacy assignment read relationships, audit relationships, and UUID route binding in `app/Models/LearningSet.php`, `app/Models/LearningSetEntry.php`, and `app/Models/LearningSetAssignment.php`
- [X] T049 [P] [US2] Update learning-set factories for active, inactive, deleted, roster-assigned, legacy-direct-assigned, published, dependency-conflict, owner, and cross-tenant states in `database/factories/LearningSetFactory.php`, `database/factories/LearningSetEntryFactory.php`, and `database/factories/LearningSetAssignmentFactory.php`
- [X] T050 [P] [US2] Create learning-set Form Requests for detail, update, status, delete, restore, and roster-aware assignment payloads in `app/Http/Requests/TeacherWorkflow/LearningSetRequest.php`
- [X] T051 [P] [US2] Create learning-set API Resources for detail, lifecycle, restore, delete, roster-aware assignment, and legacy direct-assignment read envelopes in `app/Http/Resources/TeacherWorkflow/LearningSetResource.php`
- [X] T052 [US2] Implement learning-set policy rules for owner management, active teacher-assignment eligibility, school-administrator same-school management, same-school non-owner denial, student/guardian denial, platform denial, and deleted/inactive handling in `app/Policies/LearningSetPolicy.php`
- [X] T053 [US2] Implement roster-aware learning-set service for detail, editable-field update, ordered entries, roster membership assignment writes, legacy direct-assignment read compatibility, lifecycle status, delete, restore-to-inactive, activation dependency checks, audit events, and transaction boundaries in `app/Services/TeacherWorkflow/LearningSetService.php`
- [X] T054 [US2] Implement learning-set controller actions mapped only to approved OpenAPI operation IDs in `app/Http/Controllers/Api/V1/LearningSetController.php`
- [X] T055 [US2] Register only OpenAPI-approved learning-set detail, update, status, delete, and restore routes in `routes/api.php`
- [X] T056 [US2] Run `docker exec schoolmaster-backend-app-1 php artisan test --filter=LearningSet` and fix failures in `app/Services/TeacherWorkflow/LearningSetService.php`, `app/Http/Controllers/Api/V1/LearningSetController.php`, or `tests/Feature/TeacherWorkflow/`

**Checkpoint**: User Story 2 is independently functional with existing classroom roster foundation records.

---

## Phase 5: User Story 3 - Correct Grades and Attendance (Priority: P3)

**Goal**: Authorized creating/owning teachers and school administrators can retrieve grade and attendance details, apply reasoned corrections, manage lifecycle state, preserve original values, and protect student-visible data.

**Independent Test**: Record a grade and attendance entry, retrieve details, correct each record with a 10-500 character free-text reason, verify the current value changes according to visibility rules while original values remain in history, and verify unauthorized, cross-tenant, closed-period, blank-reason, too-short-reason, too-long-reason, or missing-reason attempts are rejected.

### Tests for User Story 3

- [X] T057 [P] [US3] Add OpenAPI contract assertions and documented success/error response-shape checks for `getGrade`, `correctGrade`, `updateGradeStatus`, `deleteGrade`, `restoreGrade`, `getAttendance`, `correctAttendance`, `updateAttendanceStatus`, `deleteAttendance`, and `restoreAttendance` in `tests/Feature/TeacherWorkflow/AcademicRecordContractTest.php`
- [X] T058 [P] [US3] Add tenant isolation and authorization tests for open-period teacher owner correction, school-administrator same-school correction, same-school non-owner teacher denial, student/guardian denial, platform-without-school-context denial, closed-period teacher denial, and cross-tenant not-found behavior in `tests/Feature/TeacherWorkflow/AcademicRecordAuthorizationTest.php`
- [X] T059 [P] [US3] Add grade and attendance correction validation tests for missing reason, blank reason, reason shorter than 10 characters, reason longer than 500 characters, invalid value, immutable original recorder, immutable original student, immutable academic period, prior correction retention, and unsupported fields in `tests/Feature/TeacherWorkflow/AcademicRecordCorrectionValidationTest.php`
- [X] T060 [P] [US3] Add closed-period school-administrator correction tests for accepted grade corrections, accepted attendance corrections, current value changes, original value preservation, actor preservation, timestamp preservation, and audit history preservation in `tests/Feature/TeacherWorkflow/ClosedPeriodCorrectionTest.php`
- [X] T061 [P] [US3] Add grade and attendance lifecycle, restore-to-inactive, activation dependency, inactive/deleted student visibility, and private correction note suppression tests in `tests/Feature/TeacherWorkflow/AcademicRecordLifecycleVisibilityTest.php`
- [X] T062 [P] [US3] Add correction audit tests for accepted corrections, rejected corrections, conflict outcomes, blocked cross-tenant attempts, tenant-safe summaries, and no private correction notes in `tests/Feature/TeacherWorkflow/AcademicRecordAuditTest.php`

### Implementation for User Story 3

- [X] T063 [US3] Create correction-record migration with UUID public identifiers, `school_id`, polymorphic target record, original value reference, new value, 10-500 character reason, actor, academic period, student profile, student visibility marker, timestamps, and tenant indexes in `database/migrations/2026_06_01_000003_create_correction_records_table.php`
- [X] T064 [US3] Add or update grade and attendance persistence for lifecycle status, soft-delete metadata, current value fields, original recorder immutability, restore metadata, correction-history relationships, and indexes required by documented operations in `database/migrations/2026_06_01_000004_update_grades_attendance_lifecycle_and_corrections.php`
- [X] T065 [P] [US3] Update GradeRecord, AttendanceRecord, and CorrectionRecord models with lifecycle casts, original recorder relationships, current value casts, correction relationships, school relationship, student profile relationship, academic-period relationship, audit relationships, and UUID route binding in `app/Models/GradeRecord.php`, `app/Models/AttendanceRecord.php`, and `app/Models/CorrectionRecord.php`
- [X] T066 [P] [US3] Update grade, attendance, and correction factories for active, inactive, deleted, open-period, closed-period, corrected, owner, non-owner, and cross-tenant states in `database/factories/GradeRecordFactory.php`, `database/factories/AttendanceRecordFactory.php`, and `database/factories/CorrectionRecordFactory.php`
- [X] T067 [P] [US3] Create correction DTOs for grade and attendance correction values, 10-500 character reason, actor context, period state, and student visibility impact in `app/DTOs/TeacherWorkflow/CorrectionInput.php`
- [X] T068 [P] [US3] Create grade and attendance Form Requests for detail, correction, status, delete, restore, and validation of rejected undocumented fields in `app/Http/Requests/TeacherWorkflow/AcademicRecordRequest.php`
- [X] T069 [P] [US3] Create grade, attendance, and correction-history API Resources that expose current approved values and permitted history labels without private correction notes or unauthorized actor metadata in `app/Http/Resources/TeacherWorkflow/AcademicRecordResource.php`
- [X] T070 [US3] Implement grade and attendance policies for owner/recorder open-period correction, school-administrator same-school correction, school-administrator-only closed-period correction, same-school non-owner denial, student/guardian denial, platform denial, and deleted/inactive handling in `app/Policies/AcademicRecordPolicy.php`
- [X] T071 [US3] Implement academic record correction service for grade corrections, attendance corrections, reason bounds, original value preservation, current value updates, closed-period authority, prior correction retention, audit events, student visibility projection, and transaction boundaries in `app/Services/TeacherWorkflow/AcademicRecordCorrectionService.php`
- [X] T072 [US3] Implement academic record lifecycle service for grade and attendance detail, status, delete, restore-to-inactive, activation dependency checks, immutable-field conflicts, audit events, and transaction boundaries in `app/Services/TeacherWorkflow/AcademicRecordLifecycleService.php`
- [X] T073 [US3] Implement grade and attendance controllers mapped only to approved OpenAPI operation IDs in `app/Http/Controllers/Api/V1/AcademicRecordController.php`
- [X] T074 [US3] Register only OpenAPI-approved grade and attendance detail, correction, status, delete, and restore routes in `routes/api.php`
- [X] T075 [US3] Run `docker exec schoolmaster-backend-app-1 php artisan test --filter='AcademicRecord|ClosedPeriodCorrection'` and fix failures in `app/Services/TeacherWorkflow/`, `app/Http/Controllers/Api/V1/AcademicRecordController.php`, or `tests/Feature/TeacherWorkflow/`

**Checkpoint**: User Story 3 is independently functional for grade and attendance corrections.

---

## Phase 6: User Story 4 - Import Teacher Workflow Records (Priority: P4)

**Goal**: Authorized school administrators can submit create-only JSON imports for grades and attendance with 500-row limits, all-or-nothing validation, tenant-safe error summaries, and no partial state changes.

**Independent Test**: Submit a valid same-school JSON import payload for the approved import scope, verify records are created atomically, then submit an import with one invalid, duplicate existing, cross-tenant, unauthorized, or closed-period row and verify no row changes state.

### Tests for User Story 4

- [X] T076 [P] [US4] Add OpenAPI contract assertions and documented success/error response-shape checks for `importGrades` and `importAttendance` in `tests/Feature/TeacherWorkflow/AcademicRecordImportContractTest.php`
- [X] T077 [P] [US4] Add import authorization tests for school-administrator success, teacher denial, student denial, guardian denial, same-school non-administrator denial, platform-without-school-context denial, and cross-tenant target denial in `tests/Feature/TeacherWorkflow/AcademicRecordImportAuthorizationTest.php`
- [X] T078 [P] [US4] Add import validation tests for JSON-only payloads, more than 500 rows, malformed rows, duplicate existing records, existing-record update attempts, correction attempts, cross-tenant references, inactive students, invalid academic periods, unsupported values, missing required fields, and tenant-safe error summaries in `tests/Feature/TeacherWorkflow/AcademicRecordImportValidationTest.php`
- [X] T079 [P] [US4] Add all-or-nothing import tests proving valid rows are not committed when any row is invalid, unauthorized, duplicate, dependency-conflicting, malformed, or closed-period-restricted in `tests/Feature/TeacherWorkflow/AcademicRecordImportAtomicityTest.php`
- [X] T080 [P] [US4] Add import audit tests for accepted imports, rejected imports, actor, target import run, safe row counts, safe error summaries, and no full request payload storage in `tests/Feature/TeacherWorkflow/AcademicRecordImportAuditTest.php`

### Implementation for User Story 4

- [X] T081 [US4] Create import-run migration with UUID public identifiers, `school_id`, actor, import type, row count, accepted row count, rejected row count, `accepted/rejected` status, tenant-safe error summary, timestamps, and tenant indexes in `database/migrations/2026_06_01_000005_create_import_runs_table.php`
- [X] T082 [P] [US4] Create ImportRun model with school, actor, accepted grade, accepted attendance, audit, status, row-count, and error-summary relationships or casts in `app/Models/ImportRun.php`
- [X] T083 [P] [US4] Create import factories for accepted grade imports, rejected grade imports, accepted attendance imports, rejected attendance imports, oversized imports, duplicate-row imports, and cross-tenant-row imports in `database/factories/ImportRunFactory.php`
- [X] T084 [P] [US4] Create import DTOs for JSON grade rows, JSON attendance rows, row validation outcomes, safe row errors, and all-or-nothing import inputs in `app/DTOs/TeacherWorkflow/AcademicRecordImportInput.php`
- [X] T085 [P] [US4] Create grade and attendance import Form Requests for JSON-only payloads, 500-row maximum, create-only rows, rejected undocumented fields, and school-administrator-only access in `app/Http/Requests/TeacherWorkflow/AcademicRecordImportRequest.php`
- [X] T086 [P] [US4] Create import API Resources for accepted import results, rejected import results, row counts, tenant-safe error summaries, and no full payload echoing in `app/Http/Resources/TeacherWorkflow/AcademicRecordImportResource.php`
- [X] T087 [US4] Implement grade and attendance import policy rules for school-administrator-only authority, teacher denial, student/guardian denial, non-administrator denial, platform denial, and cross-tenant target denial in `app/Policies/AcademicRecordImportPolicy.php`
- [X] T088 [US4] Implement academic record import service for JSON-only validation, create-only grade rows, create-only attendance rows, 500-row cap, duplicate detection, dependency validation, closed-period restrictions, all-or-nothing transactions, import-run persistence, audit events, and tenant-safe error summaries in `app/Services/TeacherWorkflow/AcademicRecordImportService.php`
- [X] T089 [US4] Implement grade and attendance import controller actions mapped only to approved OpenAPI operation IDs in `app/Http/Controllers/Api/V1/AcademicRecordImportController.php`
- [X] T090 [US4] Register only OpenAPI-approved grade and attendance import routes in `routes/api.php`
- [X] T091 [US4] Run `docker exec schoolmaster-backend-app-1 php artisan test --filter=AcademicRecordImport` and fix failures in `app/Services/TeacherWorkflow/AcademicRecordImportService.php`, `app/Http/Controllers/Api/V1/AcademicRecordImportController.php`, or `tests/Feature/TeacherWorkflow/`

**Checkpoint**: User Story 4 is independently functional for administrator grade and attendance imports.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Verify cross-story behavior, compatibility, security boundaries, and contract alignment before merge.

- [X] T092 [P] Add end-to-end feature coverage for teacher content, questionnaire, roster-aware learning set, grade correction, attendance correction, and import happy paths within one school context in `tests/Feature/TeacherWorkflow/TeacherWorkflowLifecycleEndToEndTest.php`
- [X] T093 [P] Add cross-story tenant isolation coverage for missing, mismatched, inactive, unauthorized, and cross-tenant `X-School-Id` across detail, update, lifecycle, restore, download, correction, and import operations in `tests/Feature/TeacherWorkflow/TeacherWorkflowTenantIsolationTest.php`
- [X] T094 [P] Add cross-story compatibility regression coverage for existing list/create teacher workflow operations, current student self-view, reporting behavior, and legacy direct learning-set assignment reads in `tests/Feature/TeacherWorkflow/TeacherWorkflowCompatibilityTest.php`
- [X] T095 [P] Add concurrent conflict coverage for simultaneous update, lifecycle, delete, restore, correction, download authorization, and import writes without partial state in `tests/Feature/TeacherWorkflow/TeacherWorkflowConcurrencyTest.php`
- [X] T096 [P] Add cross-story audit verification for updates, lifecycle transitions, delete/restore, successful downloads, denied downloads, imports, import rejections, corrections, correction rejections, conflict outcomes, and blocked cross-tenant attempts in `tests/Feature/TeacherWorkflow/TeacherWorkflowAuditCoverageTest.php`
- [X] T097 Add route-to-OpenAPI operation ID traceability notes for every implemented route in `specs/specs/010-teacher-workflow-lifecycle/quickstart.md`
- [X] T098 Run `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` from `specs/` and fix contract lint failures in `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T099 Run `docker exec schoolmaster-backend-app-1 php artisan test` and fix failures in `app/Services/TeacherWorkflow/`, `app/Http/Controllers/Api/V1/`, `app/Policies/`, `app/Http/Requests/TeacherWorkflow/`, `app/Http/Resources/TeacherWorkflow/`, or `tests/Feature/TeacherWorkflow/`
- [X] T100 Review excluded behavior and confirm no frontend routes, guardian self-service, report lifecycle expansion, platform support access, advanced assessment types, content/questionnaire versioning, permanent purge, legal hold, anonymization, messaging, billing, imports for content/questionnaires/learning sets, approval queues, new direct selected-student assignment writes, or undocumented APIs were added in `routes/api.php`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; must complete before exposing backend routes.
- **Foundational (Phase 2)**: Depends on Setup completion; blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational completion and is the MVP.
- **User Story 2 (Phase 4)**: Depends on Foundational completion and existing classroom roster foundation records; it does not depend on US1 except for optional teacher material references in integrated tests.
- **User Story 3 (Phase 5)**: Depends on Foundational completion and existing academic-period/student/teacher records.
- **User Story 4 (Phase 6)**: Depends on Foundational completion and can be implemented after or alongside US3 if grade and attendance persistence exists.
- **Polish (Phase 7)**: Depends on all selected user stories being complete.

### User Story Dependencies

- **US1 (P1)**: Start after Phase 2; no dependency on US2, US3, or US4.
- **US2 (P2)**: Start after Phase 2; requires roster foundation data and may reference US1 content/questionnaires in integrated tests.
- **US3 (P3)**: Start after Phase 2; no dependency on US1 or US2 for grade/attendance correction flows.
- **US4 (P4)**: Start after Phase 2; depends on grade and attendance create persistence being available and can share US3 academic record support.

### Within Each User Story

- Contract and PHPUnit tests must be written first and fail before implementation.
- Persistence and models come before services.
- Services come before controllers and route registration.
- Route registration must expose only operations already documented in OpenAPI.
- Each story reaches its checkpoint before moving to the next priority unless the team deliberately works stories in parallel.

### Parallel Opportunities

- Setup documentation tasks T003, T004, and T005 can run in parallel after T001 and T002 are understood.
- Foundational tasks T008, T009, T011, T012, T013, T014, T015, T016, T017, T018, T019, and T020 can run in parallel where file ownership does not overlap.
- US1 tests T022 through T027 can run in parallel; implementation tasks T029 through T033 can run in parallel after T028.
- US2 tests T042 through T046 can run in parallel; implementation tasks T048 through T051 can run in parallel after T047.
- US3 tests T057 through T062 can run in parallel; implementation tasks T065 through T069 can run in parallel after T063 and T064.
- US4 tests T076 through T080 can run in parallel; implementation tasks T082 through T086 can run in parallel after T081.
- Polish tests T092 through T096 can run in parallel after selected story phases are complete.

---

## Parallel Example: User Story 1

```bash
# Contract, authorization, lifecycle, download, historical-meaning, and audit tests can be authored together:
Task: "T022 Add OpenAPI contract assertions for teacher content and questionnaire operations"
Task: "T023 Add tenant isolation and authorization feature tests"
Task: "T024 Add lifecycle and validation feature tests"
Task: "T025 Add teacher content download tests"
Task: "T026 Add historical-meaning edit rejection tests"
Task: "T027 Add successful and denied download audit tests"

# After persistence changes are defined, separate implementation files can proceed together:
Task: "T029 Update teacher content model"
Task: "T030 Update questionnaire models"
Task: "T031 Update teacher content and questionnaire factories"
Task: "T032 Create teacher material Form Requests"
Task: "T033 Create teacher content and questionnaire API Resources"
```

## Parallel Example: User Story 2

```bash
# Learning-set tests can be authored together:
Task: "T042 Add OpenAPI contract assertions for learning-set operations"
Task: "T043 Add authorization and tenant isolation tests"
Task: "T044 Add roster-aware assignment validation tests"
Task: "T045 Add legacy direct-assignment read compatibility tests"
Task: "T046 Add lifecycle and dependency conflict tests"

# After persistence changes are defined, separate learning-set support files can proceed together:
Task: "T048 Update LearningSet models"
Task: "T049 Update learning-set factories"
Task: "T050 Create learning-set Form Requests"
Task: "T051 Create learning-set API Resources"
```

## Parallel Example: User Story 3

```bash
# Academic correction tests can be authored together:
Task: "T057 Add OpenAPI contract assertions for grade and attendance operations"
Task: "T058 Add authorization and tenant isolation tests"
Task: "T059 Add correction validation tests"
Task: "T060 Add closed-period correction tests"
Task: "T061 Add lifecycle and student visibility tests"
Task: "T062 Add correction audit tests"

# After persistence changes are defined, separate academic record support files can proceed together:
Task: "T065 Update GradeRecord, AttendanceRecord, and CorrectionRecord models"
Task: "T066 Update grade, attendance, and correction factories"
Task: "T067 Create correction DTOs"
Task: "T068 Create grade and attendance Form Requests"
Task: "T069 Create grade, attendance, and correction-history API Resources"
```

## Parallel Example: User Story 4

```bash
# Import tests can be authored together:
Task: "T076 Add OpenAPI contract assertions for grade and attendance import operations"
Task: "T077 Add import authorization tests"
Task: "T078 Add import validation tests"
Task: "T079 Add all-or-nothing import tests"
Task: "T080 Add import audit tests"

# After import-run persistence is defined, separate import support files can proceed together:
Task: "T082 Create ImportRun model"
Task: "T083 Create import factories"
Task: "T084 Create import DTOs"
Task: "T085 Create import Form Requests"
Task: "T086 Create import API Resources"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 contract setup.
2. Complete Phase 2 shared backend foundations.
3. Complete Phase 3 teacher materials implementation.
4. Validate with `docker exec schoolmaster-backend-app-1 php artisan test --filter='TeacherMaterials|TeacherContentDownload|Questionnaire'`.
5. Stop and review route-to-OpenAPI operation alignment before adding learning-set, correction, or import behavior.

### Incremental Delivery

1. Complete Setup and Foundational phases.
2. Deliver US1 as the MVP: teacher content and questionnaire lifecycle, restore, and download.
3. Deliver US2: roster-aware learning-set lifecycle and legacy direct-assignment read compatibility.
4. Deliver US3: grade and attendance corrections plus lifecycle and visibility rules.
5. Deliver US4: administrator-only create-only JSON imports for grades and attendance.
6. Complete cross-story audit, tenant isolation, compatibility, OpenAPI lint, and full backend tests.

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup and Foundational phases together.
2. Developer A implements US1 teacher materials.
3. Developer B implements US2 learning sets after roster foundation fixtures are ready.
4. Developer C implements US3 academic corrections.
5. Developer D implements US4 imports once grade and attendance persistence support is available.
6. Team completes Polish tasks together and verifies no excluded behavior leaked into routes or contracts.

---

## Notes

- [P] tasks use different files or can proceed without depending on incomplete implementation tasks.
- [US1], [US2], [US3], and [US4] labels map directly to the user stories in `spec.md`.
- All backend routes must map to operation IDs in OpenAPI before exposure.
- No frontend implementation, guardian self-service, report lifecycle expansion, platform support access, permanent purge, messaging, billing, approval queue, or undocumented API task is approved in this slice.
- Stop at each checkpoint to validate the story independently before proceeding to the next priority.
