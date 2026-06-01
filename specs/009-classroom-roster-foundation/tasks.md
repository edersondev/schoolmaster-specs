# Tasks: Backend Classroom Roster Foundation

**Input**: Design documents from `/specs/009-classroom-roster-foundation/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/backend-classroom-roster-foundation.md, quickstart.md

**Tests**: REQUIRED by FR-019 and quickstart.md. Use OpenAPI linting plus Laravel PHPUnit feature/unit coverage. No frontend tasks are included because this feature is backend-only.

**Organization**: Tasks are grouped by user story to enable independently testable increments after shared foundations are complete.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or does not depend on incomplete tasks
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Every task includes exact target paths

## Path Conventions

- Run implementation from the `schoolmaster-backend` repository root.
- Backend paths are relative to `schoolmaster-backend/`.
- Specification and OpenAPI paths use the backend repository's `specs` symlink to `../schoolmaster-specs`.
- Contract changes must be made in both `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` before matching backend routes are exposed.

---

## Phase 1: Setup (Contract and Repository Alignment)

**Purpose**: Establish the contract-first boundary and backend implementation target before backend routes exist.

- [X] T001 Update aggregate OpenAPI with class-section, roster-membership, and teacher-assignment `/api/v1` operation IDs, schemas, response envelopes, tenant errors, validation errors, conflict errors, pagination, filters, lifecycle states, and authorization notes in `specs/api/openapi.yaml`
- [X] T002 Mirror the same classroom roster foundation OpenAPI operations, schemas, response envelopes, tenant errors, validation errors, conflict errors, pagination, filters, lifecycle states, and authorization notes in `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T003 [P] Add backend implementation notes for the classroom roster module boundary and excluded behavior in `specs/docs/backend-guidelines.md`
- [X] T004 [P] Add or update multi-tenant guidance for `school_id` roster records, `X-School-Id` resolution, and platform-versus-school authorization separation in `specs/docs/multi-tenant.md`
- [X] T005 Run `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` from `specs/` and record the result in `specs/specs/009-classroom-roster-foundation/quickstart.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared backend infrastructure that must exist before any user story route can be implemented.

**CRITICAL**: No user story route work begins until this phase is complete.

- [ ] T006 Create classroom roster route group placeholders protected by existing API auth and school-context middleware in `routes/api.php`
- [ ] T007 [P] Create classroom roster DTO namespace and shared effective-date DTOs in `app/DTOs/ClassroomRoster/EffectiveDateInput.php`
- [ ] T008 [P] Create shared classroom roster service namespace and timezone/effective-date validator in `app/Services/ClassroomRoster/EffectiveDateValidator.php`
- [ ] T009 [P] Create shared tenant-scoped lookup repository for class sections, academic periods, students, teachers, memberships, and assignments in `app/Repositories/ClassroomRoster/ClassroomRosterLookupRepository.php`
- [ ] T010 Create school-context guard helpers that fail before roster, membership, assignment, student, teacher, academic-period, duplicate, lifecycle, persistence, audit, or response lookup in `app/Services/ClassroomRoster/SchoolContextGuard.php`
- [ ] T011 [P] Create audit writer for roster foundation actions, conflicts, blocked tenant attempts, and lifecycle outcomes in `app/Services/ClassroomRoster/RosterAuditLogger.php`
- [ ] T012 [P] Add shared classroom roster authorization policy helpers for school-administrator writes and teacher own-active-assignment reads in `app/Policies/ClassroomRosterPolicy.php`
- [ ] T013 Add shared response-resource helpers for documented success, paginated, validation, forbidden, tenant-mismatch, conflict, unsupported-query, and not-found envelopes in `app/Http/Resources/ClassroomRoster/BaseRosterResource.php`
- [ ] T014 Add foundational PHPUnit unit coverage for school context guard ordering, effective-date timezone fallback, date-in-period validation, and audit-field minimization in `tests/Unit/ClassroomRoster/ClassroomRosterFoundationTest.php`

**Checkpoint**: Foundation ready. User story implementation can proceed.

---

## Phase 3: User Story 1 - Define School Teaching Structures (Priority: P1) MVP

**Goal**: A school administrator can create, list, view, update, and inactivate school-owned ClassSection/Roster records for an active academic period with approved metadata and lifecycle rules.

**Independent Test**: Authenticate as a school administrator with an active school context, create a roster for an active academic period, retrieve it through the documented envelope, confirm it is invisible from another school context, and confirm invalid metadata, duplicate code, inactive creation, reactivation, and inactivation-with-dependencies are rejected.

### Tests for User Story 1

- [ ] T015 [P] [US1] Add OpenAPI contract assertions and documented success/error response-shape checks for `listClassSections`, `createClassSection`, `getClassSection`, `updateClassSection`, and `updateClassSectionStatus` in `tests/Feature/ClassroomRoster/ClassSectionContractTest.php`
- [ ] T016 [P] [US1] Add tenant isolation and school-administrator authorization feature tests for class-section list, create, detail, update, and inactivation in `tests/Feature/ClassroomRoster/ClassSectionAuthorizationTest.php`
- [ ] T017 [P] [US1] Add validation and conflict feature tests for metadata shape, extra fields, duplicate same-school-period code, missing/inactive/closed/cross-tenant/incompatible academic periods, inactive creation, reactivation denial, missing inactivation reason, unsupported filters, unsupported include/sort parameters, page size above 100, and concurrent duplicate-code writes in `tests/Feature/ClassroomRoster/ClassSectionValidationTest.php`

### Implementation for User Story 1

- [ ] T018 [US1] Create class-section migration with UUID public identifiers, `school_id`, `academic_period_id`, required `code`, repeatable `name`, metadata JSON columns, `active/inactive` status, lifecycle reason, actor columns, timestamps, and unique same-school-period code index in `database/migrations/2026_05_30_000001_create_class_sections_table.php`
- [ ] T019 [P] [US1] Create ClassSection model with tenant, academic-period, membership, teacher-assignment, audit, status, UUID, and metadata casts in `app/Models/ClassSection.php`
- [ ] T020 [P] [US1] Create class-section factory covering active records, inactive records, metadata blocks, and same-school-period uniqueness setup in `database/factories/ClassSectionFactory.php`
- [ ] T021 [P] [US1] Create class-section Form Request classes as separate files in `app/Http/Requests/ClassroomRoster/ListClassSectionsRequest.php`, `app/Http/Requests/ClassroomRoster/StoreClassSectionRequest.php`, `app/Http/Requests/ClassroomRoster/UpdateClassSectionRequest.php`, and `app/Http/Requests/ClassroomRoster/UpdateClassSectionStatusRequest.php`
- [ ] T022 [P] [US1] Create class-section API resources for detail and paginated list envelopes in `app/Http/Resources/ClassroomRoster/ClassSectionResource.php`
- [ ] T023 [US1] Implement class-section policy rules for school-administrator management and tenant-scoped reads in `app/Policies/ClassSectionPolicy.php`
- [ ] T024 [US1] Implement class-section service for create, list, detail, update, inactivation, duplicate-code conflict handling, dependency checks, lifecycle reasons, inactive creation rejection, reactivation rejection, audit events, and transaction boundaries in `app/Services/ClassroomRoster/ClassSectionService.php`
- [ ] T025 [US1] Implement class-section controller actions mapped only to approved OpenAPI operation IDs in `app/Http/Controllers/Api/V1/ClassSectionController.php`
- [ ] T026 [US1] Register class-section routes for only OpenAPI-approved `/api/v1/class-sections` operations in `routes/api.php`
- [ ] T027 [US1] Run `docker exec schoolmaster-backend-app-1 php artisan test --filter=ClassSection` and fix failures in `app/Services/ClassroomRoster/ClassSectionService.php`, `app/Http/Controllers/Api/V1/ClassSectionController.php`, or `tests/Feature/ClassroomRoster/`

**Checkpoint**: User Story 1 is independently functional and testable as the MVP.

---

## Phase 4: User Story 2 - Manage Roster Membership (Priority: P2)

**Goal**: A school administrator can add and end eligible student memberships for a roster with all-or-nothing batch behavior, effective-date rules, history preservation, and tenant-safe validation.

**Independent Test**: Add active same-school student profiles with active enrollment covering the effective start date to a roster, list roster members, end one membership with a reason and effective date, and confirm history remains available while invalid or cross-tenant batch requests make no partial changes.

### Tests for User Story 2

- [ ] T028 [P] [US2] Add OpenAPI contract assertions and documented success/error response-shape checks for `listClassSectionMemberships`, `batchAddClassSectionMemberships`, and `batchEndClassSectionMemberships` in `tests/Feature/ClassroomRoster/RosterMembershipContractTest.php`
- [ ] T029 [P] [US2] Add membership authorization and tenant isolation feature tests for school-administrator-only add/end/list behavior in `tests/Feature/ClassroomRoster/RosterMembershipAuthorizationTest.php`
- [ ] T030 [P] [US2] Add membership validation and conflict feature tests for inactive, transferred, deleted, missing, cross-tenant, duplicate, overlapping, no covering enrollment, missing start date, future date, outside-period date, end-before-start date, missing end reason, oversized batch, invalid batch all-or-nothing, and concurrent membership conflict cases in `tests/Feature/ClassroomRoster/RosterMembershipValidationTest.php`

### Implementation for User Story 2

- [ ] T031 [US2] Create roster-membership migration with UUID public identifiers, `school_id`, `class_section_id`, `student_profile_id`, `academic_period_id`, `active/ended` status, effective start/end dates, end reason, actor columns, timestamps, and duplicate/overlap support indexes in `database/migrations/2026_05_30_000002_create_roster_memberships_table.php`
- [ ] T032 [P] [US2] Create RosterMembership model with tenant, class-section, student-profile, academic-period, audit, status, effective-date, and actor relationships in `app/Models/RosterMembership.php`
- [ ] T033 [P] [US2] Create roster-membership factory for active, ended, duplicate-conflict, and historical membership states in `database/factories/RosterMembershipFactory.php`
- [ ] T034 [P] [US2] Create membership batch DTOs for add and end requests with maximum 100 changes in `app/DTOs/ClassroomRoster/RosterMembershipBatchInput.php`
- [ ] T035 [P] [US2] Create roster-membership Form Request classes as separate files in `app/Http/Requests/ClassroomRoster/ListRosterMembershipsRequest.php`, `app/Http/Requests/ClassroomRoster/BatchAddRosterMembershipsRequest.php`, and `app/Http/Requests/ClassroomRoster/BatchEndRosterMembershipsRequest.php`
- [ ] T036 [P] [US2] Create roster-membership API resources for list and batch result envelopes in `app/Http/Resources/ClassroomRoster/RosterMembershipResource.php`
- [ ] T037 [US2] Implement roster-membership service for all-or-nothing add/end batches, enrollment-on-effective-start eligibility, overlap checks, date-order checks, lifecycle reasons, history preservation, audit events, and transactional conflict handling in `app/Services/ClassroomRoster/RosterMembershipService.php`
- [ ] T038 [US2] Implement roster-membership controller actions mapped only to approved OpenAPI operation IDs in `app/Http/Controllers/Api/V1/RosterMembershipController.php`
- [ ] T039 [US2] Register roster-membership routes under approved `/api/v1/class-sections/{classSectionId}/memberships` operations in `routes/api.php`
- [ ] T040 [US2] Run `docker exec schoolmaster-backend-app-1 php artisan test --filter=RosterMembership` and fix failures in `app/Services/ClassroomRoster/RosterMembershipService.php`, `app/Http/Controllers/Api/V1/RosterMembershipController.php`, or `tests/Feature/ClassroomRoster/`

**Checkpoint**: User Story 2 is independently functional with an existing US1 roster.

---

## Phase 5: User Story 3 - Assign Teachers to Teaching Structures (Priority: P3)

**Goal**: A school administrator can assign and deactivate eligible teachers for rosters, while teachers can list and retrieve only their own active assignments.

**Independent Test**: As a school administrator, assign an active same-school teacher with an active teacher-compatible role on the effective start date to an active roster, verify same-school visibility, verify teacher own-active-assignment list/detail access, and verify inactive, cross-tenant, missing-role, duplicate, non-admin, platform-admin-without-school-context, closed-period, and invalid-date cases are rejected.

### Tests for User Story 3

- [ ] T041 [P] [US3] Add OpenAPI contract assertions and documented success/error response-shape checks for `listTeacherAssignments`, `createTeacherAssignment`, `getTeacherAssignment`, and `updateTeacherAssignmentStatus` in `tests/Feature/ClassroomRoster/TeacherAssignmentContractTest.php`
- [ ] T042 [P] [US3] Add teacher-assignment authorization tests for school-administrator-only writes, teacher own-active-assignment reads, other-teacher denial, and platform-admin-without-school-context denial in `tests/Feature/ClassroomRoster/TeacherAssignmentAuthorizationTest.php`
- [ ] T043 [P] [US3] Add teacher-assignment validation and conflict tests for inactive/deleted/cross-tenant teachers, missing active teacher-compatible role from the existing school role/permission authority on effective start date, inactive roster, missing/inactive/closed/cross-tenant/incompatible academic periods, duplicate active assignment, missing start date, future date, outside-period date, deactivation-before-start date, missing deactivation reason, unsupported filters, unsupported include/sort parameters, page size above 100, and concurrent assignment conflict in `tests/Feature/ClassroomRoster/TeacherAssignmentValidationTest.php`

### Implementation for User Story 3

- [ ] T044 [US3] Create teacher-assignment migration with UUID public identifiers, `school_id`, `class_section_id`, `teacher_user_id`, `academic_period_id`, `active/inactive` status, effective start/end dates, deactivation reason, actor columns, timestamps, and duplicate active assignment support indexes in `database/migrations/2026_05_30_000003_create_teacher_assignments_table.php`
- [ ] T045 [P] [US3] Create TeacherAssignment model with tenant, class-section, teacher-user, academic-period, audit, status, effective-date, and actor relationships in `app/Models/TeacherAssignment.php`
- [ ] T046 [P] [US3] Create teacher-assignment factory for active, inactive, duplicate-conflict, and teacher-read-visibility states in `database/factories/TeacherAssignmentFactory.php`
- [ ] T047 [P] [US3] Create teacher-assignment DTO for create and deactivate inputs in `app/DTOs/ClassroomRoster/TeacherAssignmentInput.php`
- [ ] T048 [P] [US3] Create teacher-assignment Form Request classes as separate files in `app/Http/Requests/ClassroomRoster/ListTeacherAssignmentsRequest.php`, `app/Http/Requests/ClassroomRoster/StoreTeacherAssignmentRequest.php`, `app/Http/Requests/ClassroomRoster/ShowTeacherAssignmentRequest.php`, and `app/Http/Requests/ClassroomRoster/UpdateTeacherAssignmentStatusRequest.php`
- [ ] T049 [P] [US3] Create teacher-assignment API resources for detail and paginated list envelopes in `app/Http/Resources/ClassroomRoster/TeacherAssignmentResource.php`
- [ ] T050 [US3] Implement teacher-assignment policy rules for school-administrator management and teacher own-active-assignment detail/list reads in `app/Policies/TeacherAssignmentPolicy.php`
- [ ] T051 [US3] Implement teacher-assignment service for create, list, detail, deactivate, teacher-role-on-effective-start eligibility, duplicate active assignment conflicts, date-order checks, lifecycle reasons, audit events, and transactional conflict handling in `app/Services/ClassroomRoster/TeacherAssignmentService.php`
- [ ] T052 [US3] Implement teacher-assignment controller actions mapped only to approved OpenAPI operation IDs in `app/Http/Controllers/Api/V1/TeacherAssignmentController.php`
- [ ] T053 [US3] Register teacher-assignment routes for only approved `/api/v1/teacher-assignments` operations in `routes/api.php`
- [ ] T054 [US3] Run `docker exec schoolmaster-backend-app-1 php artisan test --filter=TeacherAssignment` and fix failures in `app/Services/ClassroomRoster/TeacherAssignmentService.php`, `app/Http/Controllers/Api/V1/TeacherAssignmentController.php`, or `tests/Feature/ClassroomRoster/`

**Checkpoint**: User Story 3 is independently functional with an existing US1 roster.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Verify cross-story behavior, compatibility, and contract alignment before merge.

- [ ] T055 [P] Add read-only legacy direct learning-set assignment compatibility regression coverage and unchanged grades, attendance, student self-view, and report behavior checks in `tests/Feature/ClassroomRoster/LegacyDirectAssignmentCompatibilityTest.php`
- [ ] T056 [P] Add end-to-end happy-path feature coverage for create roster, add memberships, assign teacher, and prevent cross-school visibility in `tests/Feature/ClassroomRoster/ClassroomRosterEndToEndTest.php`
- [ ] T057 [P] Add audit verification coverage for create, update, roster inactivation, membership add/end, teacher assignment add/deactivate, conflict outcomes, blocked cross-tenant attempts, lifecycle reasons, and tenant-safe metadata in `tests/Feature/ClassroomRoster/ClassroomRosterAuditTest.php`
- [X] T058 Add route-to-OpenAPI operation ID traceability notes for every implemented route in `specs/specs/009-classroom-roster-foundation/quickstart.md`
- [X] T059 Run `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` from `specs/` and fix contract lint failures in `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T060 Run `docker exec schoolmaster-backend-app-1 php artisan test` and fix failures in `app/Services/ClassroomRoster/`, `app/Http/Controllers/Api/V1/`, or `tests/Feature/ClassroomRoster/`
- [ ] T061 Review excluded behavior and confirm no frontend routes, teacher correction workflows, guardian self-service, report lifecycle expansion, billing, messaging, platform support access, separate Course/Classroom/Section/Group resources, permanent purge, or undocumented APIs were added in `routes/api.php`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; must complete before exposing backend routes.
- **Foundational (Phase 2)**: Depends on Setup completion; blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational completion and is the MVP.
- **User Story 2 (Phase 4)**: Depends on Foundational completion and requires a US1 class-section/roster record for independent tests.
- **User Story 3 (Phase 5)**: Depends on Foundational completion and requires a US1 class-section/roster record for independent tests.
- **Polish (Phase 6)**: Depends on all selected user stories being complete.

### User Story Dependencies

- **US1 (P1)**: Start after Phase 2; no dependency on US2 or US3.
- **US2 (P2)**: Start after Phase 2, but its independent test uses an existing class-section/roster from US1.
- **US3 (P3)**: Start after Phase 2, but its independent test uses an existing class-section/roster from US1.

### Within Each User Story

- Contract and PHPUnit tests must be written first and fail before implementation.
- Persistence and models come before services.
- Services come before controllers and route registration.
- Route registration must expose only operations already documented in OpenAPI.
- Each story reaches its checkpoint before moving to the next priority unless the team deliberately works stories in parallel.

### Parallel Opportunities

- Setup documentation tasks T003 and T004 can run in parallel after T001/T002 are understood.
- Foundational tasks T007, T008, T009, T011, and T012 can run in parallel after T006.
- US1 tests T015, T016, and T017 can run in parallel; implementation tasks T019, T020, T021, and T022 can run in parallel after T018.
- US2 tests T028, T029, and T030 can run in parallel; implementation tasks T032, T033, T034, T035, and T036 can run in parallel after T031.
- US3 tests T041, T042, and T043 can run in parallel; implementation tasks T045, T046, T047, T048, and T049 can run in parallel after T044.
- Polish tests T055, T056, and T057 can run in parallel after US1, US2, and US3 are implemented.

---

## Parallel Example: User Story 1

```bash
# Contract and backend tests can be authored together:
Task: "T015 Add OpenAPI contract assertions for class-section operations"
Task: "T016 Add tenant isolation and school-administrator authorization feature tests"
Task: "T017 Add validation and conflict feature tests"

# After the migration is defined, separate implementation files can proceed together:
Task: "T019 Create ClassSection model"
Task: "T020 Create class-section factory"
Task: "T021 Create class-section request classes"
Task: "T022 Create class-section API resources"
```

## Parallel Example: User Story 2

```bash
# Membership tests can be authored together:
Task: "T028 Add OpenAPI contract assertions for membership operations"
Task: "T029 Add membership authorization and tenant isolation feature tests"
Task: "T030 Add membership validation and conflict feature tests"

# After the migration is defined, separate membership support files can proceed together:
Task: "T032 Create RosterMembership model"
Task: "T033 Create roster-membership factory"
Task: "T034 Create membership batch DTOs"
Task: "T035 Create membership request classes"
Task: "T036 Create roster-membership API resources"
```

## Parallel Example: User Story 3

```bash
# Teacher-assignment tests can be authored together:
Task: "T041 Add OpenAPI contract assertions for teacher-assignment operations"
Task: "T042 Add teacher-assignment authorization tests"
Task: "T043 Add teacher-assignment validation and conflict tests"

# After the migration is defined, separate assignment support files can proceed together:
Task: "T045 Create TeacherAssignment model"
Task: "T046 Create teacher-assignment factory"
Task: "T047 Create teacher-assignment DTO"
Task: "T048 Create teacher-assignment request classes"
Task: "T049 Create teacher-assignment API resources"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 contract setup.
2. Complete Phase 2 shared backend foundations.
3. Complete Phase 3 class-section/roster implementation.
4. Validate with `docker exec schoolmaster-backend-app-1 php artisan test --filter=ClassSection`.
5. Stop and review route-to-OpenAPI operation alignment before adding memberships or teacher assignments.

### Incremental Delivery

1. Complete Setup and Foundational phases.
2. Deliver US1 as the MVP: ClassSection/Roster management.
3. Deliver US2: roster membership add/end/list.
4. Deliver US3: teacher assignment management plus teacher own-active-assignment reads.
5. Complete cross-story audit, legacy compatibility, OpenAPI lint, and full backend tests.

### Parallel Team Strategy

1. One engineer owns OpenAPI contract updates in `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`.
2. One engineer owns shared backend foundations in `app/Services/ClassroomRoster/`, `app/Repositories/ClassroomRoster/`, `app/Policies/`, and `app/Http/Resources/ClassroomRoster/`.
3. After Phase 2, separate engineers can work US1, US2, and US3, with US2 and US3 using US1 fixtures for independent tests.

---

## Notes

- [P] tasks are parallelizable only when their prerequisites are already complete.
- Contract updates precede backend route exposure.
- Backend tests are required because the feature changes tenant, authorization, lifecycle, and REST behavior.
- No frontend implementation is authorized by this feature.
- Stop if an implementation task requires behavior not documented in OpenAPI or the feature spec.
