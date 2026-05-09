---

description: "Task list for SchoolMaster Platform Foundation"
---

# Tasks: SchoolMaster Platform Foundation

**Input**: Design documents from `/specs/001-schoolmaster-platform/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, quickstart.md, `contracts/openapi.yaml`

**Tests**: Critical business flow tests are REQUIRED for this feature. Include OpenAPI contract verification, backend PHPUnit coverage, and frontend Vitest coverage for each user story before implementation.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (`US1`, `US2`, `US3`)
- Include exact file paths in descriptions

## Path Conventions

- **Specification repository**: `schoolmaster-specs/specs/001-schoolmaster-platform/`
- **Backend repository**: `schoolmaster-backend/app/`, `schoolmaster-backend/routes/api.php`, `schoolmaster-backend/tests/Feature/`, `schoolmaster-backend/tests/Unit/`
- **Frontend repository**: `schoolmaster-frontend/src/modules/`, `schoolmaster-frontend/src/router/`, `schoolmaster-frontend/src/services/`, `schoolmaster-frontend/src/stores/`, `schoolmaster-frontend/tests/`

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish contract, module boundaries, and repository scaffolding for the platform foundation.

- [ ] T001 Update `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` with the v1 module operations, shared schemas, security requirements, and `/api/v1` response envelopes needed by the planned stories
- [ ] T001a Expand `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` with security schemes, auth responses, pagination and filtering parameters, sorting semantics, and tenant-scope rules for P1 endpoints
- [ ] T001b Define concrete request and response schemas in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` for schools, users, roles, permissions, academic years, academic periods, and guardians
- [ ] T002 Create cross-repository delivery notes in `schoolmaster-specs/specs/001-schoolmaster-platform/plan.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md` that map each module to `schoolmaster-backend` and `schoolmaster-frontend` implementation targets
- [ ] T002a Add cross-repository traceability conventions for feature id `001-schoolmaster-platform` in `schoolmaster-specs/specs/001-schoolmaster-platform/plan.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`
- [ ] T003 [P] Create baseline feature directories and placeholder README files in `schoolmaster-backend/app/Services/`, `schoolmaster-backend/app/DTOs/`, `schoolmaster-frontend/src/modules/`, and `schoolmaster-frontend/tests/` for the v1 feature slices

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core tenant-aware infrastructure that blocks all user stories until complete.

**⚠️ CRITICAL**: No user story work should begin before this phase is complete.

- [ ] T004 Create shared tenant-aware domain models with UUIDs and status support in `schoolmaster-backend/app/Models/School.php`, `schoolmaster-backend/app/Models/User.php`, `schoolmaster-backend/app/Models/Role.php`, and `schoolmaster-backend/app/Models/Permission.php`
- [ ] T004a Create foundational Laravel migrations for `schools`, `users`, `roles`, `permissions`, and supporting pivot tables in `schoolmaster-backend/database/migrations/` with UUID keys, tenant-aware indexes, status fields, and soft deletes where applicable
- [ ] T004b Implement shared tenant query scope or equivalent tenant-resolution infrastructure in `schoolmaster-backend/app/Models/` and `schoolmaster-backend/app/Repositories/`
- [ ] T005 [P] Implement shared backend DTO and repository scaffolding for tenant-scoped access in `schoolmaster-backend/app/DTOs/` and `schoolmaster-backend/app/Repositories/`
- [ ] T005a [P] Implement authorization mapping for platform-scope versus school-scope permissions in `schoolmaster-backend/app/Services/Auth/` and `schoolmaster-backend/app/Policies/`
- [ ] T006 [P] Implement shared JSON envelope, exception, and API resource foundations in `schoolmaster-backend/app/Http/Resources/` and `schoolmaster-backend/app/Exceptions/`
- [ ] T006a [P] Define machine-readable error codes and validation error formats aligned with `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` in `schoolmaster-backend/app/Exceptions/` and `schoolmaster-backend/app/Http/Resources/`
- [ ] T007 Implement authentication, tenant resolution, and role-based authorization foundations in `schoolmaster-backend/app/Services/Auth/`, `schoolmaster-backend/app/Policies/`, and `schoolmaster-backend/routes/api.php`
- [ ] T007a Implement and document inactive-school, inactive-user, and tenant-mismatch rejection rules in `schoolmaster-backend/app/Services/Auth/`, `schoolmaster-backend/app/Policies/`, and `schoolmaster-backend/tests/Feature/`
- [ ] T008 [P] Create shared frontend auth, tenant context, and API client foundations in `schoolmaster-frontend/src/services/http/`, `schoolmaster-frontend/src/stores/auth/`, and `schoolmaster-frontend/src/router/index.ts`
- [ ] T008a [P] Implement frontend session bootstrap and tenant-context restoration based only on API responses in `schoolmaster-frontend/src/services/http/`, `schoolmaster-frontend/src/stores/auth/`, and `schoolmaster-frontend/src/router/index.ts`
- [ ] T009 [P] Add contract verification and repository traceability workflow notes in `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T009a [P] Add executable OpenAPI validation and response-shape verification workflow guidance in `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`

**Checkpoint**: Shared contract and tenant foundations are ready. User stories can now proceed.

---

## Phase 3: User Story 1 - Onboard a school and its staff (Priority: P1) 🎯 MVP

**Goal**: Enable platform administrators to provision a school tenant and school administrators to configure academic structure, users, roles, and guardians within their own school.

**Independent Test**: Create and activate a school, define one academic year and periods, create staff and student accounts with roles, associate guardians, and verify all access remains tenant-scoped.

### Tests for User Story 1

- [ ] T010 [P] [US1] Add contract coverage for onboarding, schools, users, roles, academic years, academic periods, and guardians in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T011 [P] [US1] Add backend feature tests for school provisioning, tenant isolation, academic structure, and role assignment in `schoolmaster-backend/tests/Feature/Admin/SchoolOnboardingTest.php`
- [ ] T012 [P] [US1] Add frontend Vitest coverage for auth, school administration, and academic setup flows in `schoolmaster-frontend/tests/modules/admin/school-onboarding.spec.ts`

### Implementation for User Story 1

- [ ] T013 [P] [US1] Create school administration persistence models and relationships in `schoolmaster-backend/app/Models/AcademicYear.php`, `schoolmaster-backend/app/Models/AcademicPeriod.php`, `schoolmaster-backend/app/Models/StudentProfile.php`, and `schoolmaster-backend/app/Models/Guardian.php`
- [ ] T013a [P] [US1] Create Laravel migrations for `academic_years`, `academic_periods`, `student_profiles`, `guardians`, and the student-guardian pivot tables in `schoolmaster-backend/database/migrations/` with tenant-safe constraints and soft deletes where applicable
- [ ] T014 [P] [US1] Implement onboarding and administration services in `schoolmaster-backend/app/Services/Schools/SchoolProvisioningService.php`, `schoolmaster-backend/app/Services/Users/UserAdministrationService.php`, and `schoolmaster-backend/app/Services/Academics/AcademicStructureService.php`
- [ ] T014a [P] [US1] Implement scoped role management and permission assignment rules in `schoolmaster-backend/app/Services/Users/` and `schoolmaster-backend/app/Policies/`
- [ ] T015 [US1] Implement form requests, policies, API resources, and controllers for `/api/v1/schools`, `/api/v1/users`, `/api/v1/roles`, `/api/v1/permissions`, `/api/v1/academic-years`, `/api/v1/academic-periods`, and `/api/v1/guardians` in `schoolmaster-backend/app/Http/Controllers/Api/V1/`, `schoolmaster-backend/app/Http/Requests/`, and `schoolmaster-backend/app/Http/Resources/`
- [ ] T015a [US1] Implement contract-backed permission exposure through `/api/v1/roles` responses and `/api/v1/permissions` in `schoolmaster-backend/app/Http/Controllers/Api/V1/`, `schoolmaster-backend/app/Http/Resources/`, and `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T016 [P] [US1] Implement admin-facing API service modules in `schoolmaster-frontend/src/services/schools.ts`, `schoolmaster-frontend/src/services/users.ts`, `schoolmaster-frontend/src/services/roles.ts`, `schoolmaster-frontend/src/services/permissions.ts`, `schoolmaster-frontend/src/services/academic-years.ts`, `schoolmaster-frontend/src/services/academic-periods.ts`, and `schoolmaster-frontend/src/services/guardians.ts`
- [ ] T017 [P] [US1] Implement Pinia stores for school setup and user administration in `schoolmaster-frontend/src/stores/schools.ts`, `schoolmaster-frontend/src/stores/users.ts`, and `schoolmaster-frontend/src/stores/academics.ts`
- [ ] T018 [US1] Build the onboarding and school administration views in `schoolmaster-frontend/src/modules/admin/` and register routes in `schoolmaster-frontend/src/router/index.ts`
- [ ] T019 [US1] Document tenant-boundary, inactive-status, and onboarding validation rules in `schoolmaster-specs/specs/001-schoolmaster-platform/spec.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`
- [ ] T019a [US1] Document role inheritance, permission assignment, and platform-versus-school administration boundaries in `schoolmaster-specs/specs/001-schoolmaster-platform/spec.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`

**Checkpoint**: User Story 1 delivers a testable tenant onboarding and school setup MVP.

---

## Phase 4: User Story 2 - Run day-to-day teaching workflows (Priority: P2)

**Goal**: Enable teachers to manage instructional content, build learning sets, and record attendance and grades within an active academic period.

**Independent Test**: As a teacher, create folders and content, publish a learning set containing content and questionnaires, then record attendance and grades for students in an active academic period.

### Tests for User Story 2

- [ ] T020 [P] [US2] Add contract coverage for teacher content, questionnaires, learning sets, grades, and attendance in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T021 [P] [US2] Add backend feature tests for teacher workflow, period validation, and tenant-safe record creation in `schoolmaster-backend/tests/Feature/Teacher/TeachingWorkflowTest.php`
- [ ] T022 [P] [US2] Add frontend Vitest coverage for teacher workflow services and stores in `schoolmaster-frontend/tests/modules/teacher/teaching-workflow.spec.ts`

### Implementation for User Story 2

- [ ] T023 [P] [US2] Create teacher workflow persistence models in `schoolmaster-backend/app/Models/TeacherContentFolder.php`, `schoolmaster-backend/app/Models/TeacherContentItem.php`, `schoolmaster-backend/app/Models/Questionnaire.php`, `schoolmaster-backend/app/Models/LearningSet.php`, `schoolmaster-backend/app/Models/LearningSetEntry.php`, `schoolmaster-backend/app/Models/GradeRecord.php`, and `schoolmaster-backend/app/Models/AttendanceRecord.php`
- [ ] T023a [P] [US2] Create Laravel migrations for teacher content, questionnaires, learning sets, learning set entries, grade records, and attendance records in `schoolmaster-backend/database/migrations/` with tenant indexes and soft deletes where applicable
- [ ] T024 [P] [US2] Implement teacher content, questionnaire, learning set, grade, and attendance services in `schoolmaster-backend/app/Services/TeacherContent/`, `schoolmaster-backend/app/Services/Questionnaires/`, `schoolmaster-backend/app/Services/LearningSets/`, `schoolmaster-backend/app/Services/Grades/`, and `schoolmaster-backend/app/Services/Attendance/`
- [ ] T024a [P] [US2] Implement tenant-scoped file storage, upload sanitization, and private asset access workflow in `schoolmaster-backend/app/Services/TeacherContent/` and `schoolmaster-backend/app/Http/Requests/`
- [ ] T025 [US2] Implement requests, policies, resources, and controllers for `/api/v1/teacher-content`, `/api/v1/questionnaires`, `/api/v1/learning-sets`, `/api/v1/grades`, and `/api/v1/attendance` in `schoolmaster-backend/app/Http/Controllers/Api/V1/`, `schoolmaster-backend/app/Http/Requests/`, and `schoolmaster-backend/app/Http/Resources/`
- [ ] T025a [US2] Define contract-backed upload, download authorization, MIME and size validation, and inactive-asset handling in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`, `schoolmaster-backend/app/Http/Requests/`, and `schoolmaster-backend/app/Http/Controllers/Api/V1/`
- [ ] T026 [P] [US2] Implement teacher workflow API clients in `schoolmaster-frontend/src/services/teacher-content.ts`, `schoolmaster-frontend/src/services/questionnaires.ts`, `schoolmaster-frontend/src/services/learning-sets.ts`, `schoolmaster-frontend/src/services/grades.ts`, and `schoolmaster-frontend/src/services/attendance.ts`
- [ ] T027 [P] [US2] Implement Pinia stores for teacher operations in `schoolmaster-frontend/src/stores/teacher-content.ts`, `schoolmaster-frontend/src/stores/questionnaires.ts`, `schoolmaster-frontend/src/stores/learning-sets.ts`, and `schoolmaster-frontend/src/stores/class-records.ts`
- [ ] T028 [US2] Build the teacher workflow views for content management, learning sets, attendance, and grades in `schoolmaster-frontend/src/modules/teacher/` and update routes in `schoolmaster-frontend/src/router/index.ts`
- [ ] T029 [US2] Document upload constraints, inactive-content behavior, and academic-period validation rules in `schoolmaster-specs/specs/001-schoolmaster-platform/spec.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`
- [ ] T029a [US2] Document allowed file types, size limits, storage visibility, and validation or sanitization outcomes in `schoolmaster-specs/specs/001-schoolmaster-platform/spec.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`

**Checkpoint**: User Story 2 delivers an independently testable teacher operations slice.

---

## Phase 5: User Story 3 - View student progress and official outputs (Priority: P3)

**Goal**: Enable students to view assigned learning materials and academic records, and enable school administrators to request tenant-safe reports.

**Independent Test**: As a student, view assigned learning sets, grades, and attendance for the active period; as a school administrator, generate reports for the same school and period without cross-tenant leakage.

### Tests for User Story 3

- [ ] T030 [P] [US3] Add contract coverage for student learning views and report requests in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T031 [P] [US3] Add backend feature tests for student self-view and school reporting scope in `schoolmaster-backend/tests/Feature/Student/StudentProgressAndReportsTest.php`
- [ ] T032 [P] [US3] Add frontend Vitest coverage for student progress and reporting flows in `schoolmaster-frontend/tests/modules/student/student-progress-and-reports.spec.ts`

### Implementation for User Story 3

- [ ] T033 [P] [US3] Create reporting persistence and projection support in `schoolmaster-backend/app/Models/ReportRun.php` and `schoolmaster-backend/app/Repositories/Reports/`
- [ ] T033a [P] [US3] Create Laravel migrations and projection support for report requests and persisted outputs in `schoolmaster-backend/database/migrations/`, `schoolmaster-backend/app/Models/ReportRun.php`, and `schoolmaster-backend/app/Repositories/Reports/`
- [ ] T034 [P] [US3] Implement student progress and report generation services in `schoolmaster-backend/app/Services/Students/StudentProgressService.php` and `schoolmaster-backend/app/Services/Reports/ReportGenerationService.php`
- [ ] T034a [P] [US3] Define launch-scope report types, filters, and tenant-bound report generation rules in `schoolmaster-backend/app/Services/Reports/` and `schoolmaster-specs/specs/001-schoolmaster-platform/spec.md`
- [ ] T035 [US3] Implement controllers, requests, policies, and resources for student views and `/api/v1/reports` in `schoolmaster-backend/app/Http/Controllers/Api/V1/`, `schoolmaster-backend/app/Http/Requests/`, and `schoolmaster-backend/app/Http/Resources/`
- [ ] T035a [US3] Add explicit student self-view contract coverage for learning timeline, grades, and attendance endpoints in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` and `schoolmaster-backend/app/Http/Controllers/Api/V1/`
- [ ] T036 [P] [US3] Implement student and report API clients in `schoolmaster-frontend/src/services/student-progress.ts` and `schoolmaster-frontend/src/services/reports.ts`
- [ ] T037 [P] [US3] Implement Pinia stores for student timeline and school reporting in `schoolmaster-frontend/src/stores/student-progress.ts` and `schoolmaster-frontend/src/stores/reports.ts`
- [ ] T038 [US3] Build the student progress and reporting screens in `schoolmaster-frontend/src/modules/student/`, `schoolmaster-frontend/src/modules/reports/`, and `schoolmaster-frontend/src/router/index.ts`
- [ ] T039 [US3] Document report filters, student visibility rules, and cross-tenant rejection scenarios in `schoolmaster-specs/specs/001-schoolmaster-platform/spec.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`
- [ ] T039a [US3] Document the report-type matrix, filter constraints, and platform-versus-school reporting boundaries in `schoolmaster-specs/specs/001-schoolmaster-platform/spec.md` and `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`

**Checkpoint**: User Story 3 delivers independently testable student visibility and reporting outcomes.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final hardening, consistency, and validation across all stories.

- [ ] T040 [P] Reconcile final contract examples, shared schemas, and error cases in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T040a [P] Reconcile success, validation failure, forbidden, not found, inactive-record, and tenant-isolation examples in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T041 Run the end-to-end validation walkthrough and update expected outcomes in `schoolmaster-specs/specs/001-schoolmaster-platform/quickstart.md`
- [ ] T042 [P] Add cross-story backend regression coverage for tenant isolation and inactive-status handling in `schoolmaster-backend/tests/Feature/Security/TenantIsolationRegressionTest.php`
- [ ] T042a [P] Add backend regression coverage for platform-admin override rules and upload access control in `schoolmaster-backend/tests/Feature/Security/`
- [ ] T043 [P] Add cross-story frontend regression coverage for route guards and role-based navigation in `schoolmaster-frontend/tests/router/role-navigation.spec.ts`
- [ ] T043a [P] Add frontend regression coverage for tenant-loss and session-mismatch behavior in `schoolmaster-frontend/tests/router/` and `schoolmaster-frontend/tests/modules/`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1: Setup**: No dependencies; starts immediately.
- **Phase 2: Foundational**: Depends on Phase 1 and blocks all user stories.
- **Phase 3: User Story 1**: Depends on Phase 2; establishes the MVP slice.
- **Phase 4: User Story 2**: Depends on Phase 2 and the shared academic structures from US1.
- **Phase 5: User Story 3**: Depends on Phase 2 and consumes academic and instructional data produced by US1 and US2.
- **Phase 6: Polish**: Depends on all implemented stories.

### User Story Dependencies

- **US1**: No story dependency after foundational work.
- **US2**: Depends on US1 data structures for schools, users, students, and academic periods.
- **US3**: Depends on US1 for tenant, user, and academic setup and on US2 for learning sets, grades, and attendance data.

### Within Each User Story

- Contract, backend, and frontend tests should be created before implementation and should fail before code is added.
- Persistence models and repository support should land before services.
- Services should land before controllers, requests, resources, and frontend integration.
- Frontend services and stores should land before route-level screens.
- Documentation and rule updates should finalize the story.

## Parallel Opportunities

- T003 can run in parallel with T001-T002 after repository targeting is clear.
- T005, T006, T008, and T009 can run in parallel after T004 starts the shared foundation.
- For each story, the contract task, backend test task, and frontend test task can run in parallel.
- For each story, model creation and service-layer implementation can be split across backend contributors while frontend service/store work proceeds separately after contract alignment.
- T040, T042, and T043 can run in parallel during polish.

---

## Parallel Example: User Story 1

```bash
# Launch the US1 verification work together
Task: "T010 [US1] Add contract coverage for onboarding flows"
Task: "T011 [US1] Add backend feature tests for school onboarding"
Task: "T012 [US1] Add frontend Vitest coverage for admin setup flows"

# Launch the US1 implementation streams together after tests exist
Task: "T013 [US1] Create school administration persistence models"
Task: "T014 [US1] Implement onboarding and administration services"
Task: "T016 [US1] Implement admin-facing API service modules"
Task: "T017 [US1] Implement Pinia stores for school setup"
```

## Parallel Example: User Story 2

```bash
# Parallel teacher workflow implementation after contracts are aligned
Task: "T023 [US2] Create teacher workflow persistence models"
Task: "T024 [US2] Implement teacher workflow backend services"
Task: "T026 [US2] Implement teacher workflow API clients"
Task: "T027 [US2] Implement teacher workflow Pinia stores"
```

## Parallel Example: User Story 3

```bash
# Parallel reporting and student visibility work
Task: "T033 [US3] Create reporting persistence and projection support"
Task: "T034 [US3] Implement student progress and report generation services"
Task: "T036 [US3] Implement student and report API clients"
Task: "T037 [US3] Implement student timeline and reporting stores"
```

---

## Implementation Strategy

### MVP First

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. Validate tenant onboarding, academic setup, and role-based access before expanding scope.

### Incremental Delivery

1. Deliver US1 as the tenant onboarding MVP.
2. Add US2 for teacher operations after the academic structure is stable.
3. Add US3 for student self-view and reporting after operational data exists.
4. Finish with polish tasks to harden cross-story behavior.

### Parallel Team Strategy

1. One stream updates contracts and specification notes in `schoolmaster-specs/specs/001-schoolmaster-platform/`.
2. One stream builds backend feature slices in `schoolmaster-backend/app/` and `schoolmaster-backend/tests/`.
3. One stream builds frontend modules in `schoolmaster-frontend/src/` and `schoolmaster-frontend/tests/`.

---

## Notes

- All tasks follow the required checklist format with task ID, optional `[P]`, optional story label, and explicit file paths.
- MVP scope is User Story 1 only.
- The task list assumes backend and frontend repositories will mirror the structure defined in `plan.md`.
