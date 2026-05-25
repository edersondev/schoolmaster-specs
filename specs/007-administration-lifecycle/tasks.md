# Tasks: Backend Administration Lifecycle Management

**Input**: Design documents from `specs/007-administration-lifecycle/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/backend-administration-lifecycle.md`, `quickstart.md`  
**Feature ID**: `007-administration-lifecycle`

**Tests**: Required. This slice changes critical backend behavior for contract expansion, tenant isolation, administration lifecycle transitions, soft-delete and restore behavior, dependency conflicts, bulk all-or-nothing semantics, platform/school authorization separation, lifecycle history, and response-contract behavior.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared foundations are in place.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Task can run in parallel with other marked tasks in the same phase when prerequisites are met.
- **[Story]**: User-story label for story phases only.
- Every task includes an exact target file path relative to the `schoolmaster-backend` repository root, except shared specification files, which use paths relative to the `schoolmaster-specs` repository root.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Expand the approved contract and align backend implementation with the administration lifecycle specification before backend code changes.

- [ ] T001 Create feature implementation notes with approved operation boundary and blocked scope in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T002 Add administration lifecycle operation paths, operation IDs, parameters, request schemas, response schemas, lifecycle status semantics, school lifecycle reason/effective-date requirements, dependency conflict responses, and bulk result schemas in `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T003 Mirror the approved administration lifecycle contract behavior in `api/openapi.yaml`
- [ ] T004 Run Redocly contract validation for the aggregate and platform contracts and record the result in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T005 Confirm the approved school, user, role, academic year, academic period, and guardian lifecycle operation IDs exist in the mounted OpenAPI contract and record the inventory in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T006 Confirm no undocumented invitation, password recovery, classroom, roster, teacher correction, guardian self-service, report lifecycle, frontend, permanent purge, anonymization, billing, messaging, notification, or support-override routes exist and document the route inventory in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T007 Record blocked contract gaps for account lifecycle, roster models, teacher corrections, guardian self-service, report lifecycle expansion, platform support access, frontend implementation, permanent purge, anonymization, retention management, and additional lifecycle modes in `docs/implementation-notes/007-administration-lifecycle.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared backend persistence, permissions, tenant, lifecycle, validation, authorization, dependency, history, and response infrastructure required by every administration lifecycle story.

**Critical**: No user story work should begin until this phase is complete.

- [ ] T008 Inspect existing backend migrations and models for `School`, `User`, `Role`, `Permission`, `AcademicYear`, `AcademicPeriod`, `Guardian`, guardian-student associations, `StudentProfile`, report references, and teacher workflow references and document the inventory in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T009 Add only missing lifecycle history, soft-delete metadata, status, reason, effective-date, and dependency indexes for affected resources in `database/migrations/2026_05_25_000001_add_administration_lifecycle_management.php`
- [ ] T010 [P] Seed or verify platform-scoped school lifecycle and school-scoped administration lifecycle permission definitions in `database/seeders/PermissionSeeder.php`
- [ ] T011 [P] Define shared lifecycle action and status value constants in `app/Services/AdministrationLifecycle/LifecycleAction.php`
- [ ] T012 [P] Define shared lifecycle transition matrix for school, user, role, academic year, academic period, and guardian resources in `app/Services/AdministrationLifecycle/LifecycleTransitionRules.php`
- [ ] T013 [P] Define shared administration tenant assertion helpers in `app/Services/Concerns/AssertsAdministrationTenantScope.php`
- [ ] T014 [P] Define shared platform-versus-school lifecycle authorization helpers in `app/Services/Concerns/AuthorizesAdministrationLifecycle.php`
- [ ] T015 [P] Define shared dependency conflict checker contract in `app/Services/AdministrationLifecycle/DependencyConflictChecker.php`
- [ ] T016 [P] Implement school lifecycle dependency checks in `app/Services/AdministrationLifecycle/DependencyChecks/SchoolLifecycleDependencyCheck.php`
- [ ] T017 [P] Implement user lifecycle dependency checks in `app/Services/AdministrationLifecycle/DependencyChecks/UserLifecycleDependencyCheck.php`
- [ ] T018 [P] Implement role lifecycle dependency checks in `app/Services/AdministrationLifecycle/DependencyChecks/RoleLifecycleDependencyCheck.php`
- [ ] T019 [P] Implement academic year lifecycle dependency checks in `app/Services/AdministrationLifecycle/DependencyChecks/AcademicYearLifecycleDependencyCheck.php`
- [ ] T020 [P] Implement academic period lifecycle dependency checks in `app/Services/AdministrationLifecycle/DependencyChecks/AcademicPeriodLifecycleDependencyCheck.php`
- [ ] T021 [P] Implement guardian lifecycle dependency checks in `app/Services/AdministrationLifecycle/DependencyChecks/GuardianLifecycleDependencyCheck.php`
- [ ] T022 [P] Implement `LifecycleHistory` model UUIDs, optional `school_id`, resource references, actor relationship, metadata guard, and immutable-history behavior in `app/Models/LifecycleHistory.php`
- [ ] T023 [P] Implement or verify soft-delete, status, UUID, and tenant-root helpers in `app/Models/School.php`
- [ ] T024 [P] Implement or verify soft-delete, status, UUID, and school-scope helpers in `app/Models/User.php`
- [ ] T025 [P] Implement or verify soft-delete, status, UUID, and school-scope helpers in `app/Models/Role.php`
- [ ] T026 [P] Implement or verify soft-delete, status, UUID, and school-scope helpers in `app/Models/AcademicYear.php`
- [ ] T027 [P] Implement or verify soft-delete, status, UUID, and school-scope helpers in `app/Models/AcademicPeriod.php`
- [ ] T028 [P] Implement or verify soft-delete, status, UUID, and school-scope helpers in `app/Models/Guardian.php`
- [ ] T029 [P] Implement lifecycle model factories for tests in `database/factories/AdministrationLifecycleFactory.php`
- [ ] T030 [P] Implement shared lifecycle history resource using the published history shape in `app/Http/Resources/AdministrationLifecycle/LifecycleHistoryResource.php`
- [ ] T031 [P] Implement shared lifecycle outcome resource using the published outcome shape in `app/Http/Resources/AdministrationLifecycle/LifecycleOutcomeResource.php`
- [ ] T032 [P] Implement shared bulk lifecycle outcome resource using the published bulk result shape in `app/Http/Resources/AdministrationLifecycle/BulkLifecycleOutcomeResource.php`
- [ ] T033 Define administration lifecycle route groups with auth and tenant middleware boundaries, without operation-specific route registration, in `routes/api.php`
- [ ] T034 Register or update school, user, role, academic year, academic period, and guardian policies in `app/Providers/AuthServiceProvider.php`

**Checkpoint**: Persistence, permissions, tenant helpers, transition rules, dependency checks, history model, shared resources, factories, and route boundaries are ready for story work.

---

## Phase 3: User Story 1 - Maintain Administrative Records (Priority: P1) MVP

**Goal**: An authorized administrator can view details and update documented mutable fields for existing school and school-administration records without crossing platform or tenant boundaries.

**Independent Test**: Authenticate as an administrator with the required platform or school scope, retrieve a single existing record, update only documented mutable fields, and verify the response reflects the update while records outside the permitted scope remain inaccessible.

### Tests for User Story 1

- [ ] T035 [P] [US1] Add PHPUnit feature tests for school detail and update platform-scoped success, forbidden school-owned access, immutable field rejection, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/SchoolDetailUpdateTest.php`
- [ ] T036 [P] [US1] Add PHPUnit feature tests for user detail and update same-school success, tenant isolation, immutable field rejection, role-scope rejection, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/UserDetailUpdateTest.php`
- [ ] T037 [P] [US1] Add PHPUnit feature tests for role detail and update scope integrity, permission compatibility, tenant isolation, immutable scope rejection, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/RoleDetailUpdateTest.php`
- [ ] T038 [P] [US1] Add PHPUnit feature tests for academic year detail and update date validation, tenant isolation, dependency conflict, immutable ownership rejection, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/AcademicYearDetailUpdateTest.php`
- [ ] T039 [P] [US1] Add PHPUnit feature tests for academic period detail and update parent-year containment, sequence uniqueness, tenant isolation, immutable ownership rejection, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/AcademicPeriodDetailUpdateTest.php`
- [ ] T040 [P] [US1] Add PHPUnit feature tests for guardian detail and update same-school associations, tenant isolation, cross-tenant reference rejection, immutable ownership rejection, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/GuardianDetailUpdateTest.php`
- [ ] T041 [P] [US1] Add OpenAPI response-shape regression coverage for all approved detail and update operation IDs in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationDetailUpdateContractTest.php`
- [ ] T042 [P] [US1] Add unit tests for update mutability, immutable ownership, tenant, and dependency validation rules in `tests/Unit/Services/AdministrationLifecycle/AdministrationUpdateRulesTest.php`

### Implementation for User Story 1

- [ ] T043 [P] [US1] Implement generic administration update input DTO in `app/DTOs/AdministrationLifecycle/UpdateAdministrationResourceData.php`
- [ ] T044 [P] [US1] Implement school update request validation in `app/Http/Requests/AdministrationLifecycle/UpdateSchoolLifecycleRequest.php`
- [ ] T045 [P] [US1] Implement user update request validation in `app/Http/Requests/AdministrationLifecycle/UpdateUserLifecycleRequest.php`
- [ ] T046 [P] [US1] Implement role update request validation in `app/Http/Requests/AdministrationLifecycle/UpdateRoleLifecycleRequest.php`
- [ ] T047 [P] [US1] Implement academic year update request validation in `app/Http/Requests/AdministrationLifecycle/UpdateAcademicYearLifecycleRequest.php`
- [ ] T048 [P] [US1] Implement academic period update request validation in `app/Http/Requests/AdministrationLifecycle/UpdateAcademicPeriodLifecycleRequest.php`
- [ ] T049 [P] [US1] Implement guardian update request validation in `app/Http/Requests/AdministrationLifecycle/UpdateGuardianLifecycleRequest.php`
- [ ] T050 [P] [US1] Implement school detail and update resources using the published lifecycle shape in `app/Http/Resources/AdministrationLifecycle/SchoolLifecycleResource.php`
- [ ] T051 [P] [US1] Implement user detail and update resources using the published lifecycle shape in `app/Http/Resources/AdministrationLifecycle/UserLifecycleResource.php`
- [ ] T052 [P] [US1] Implement role detail and update resources using the published lifecycle shape in `app/Http/Resources/AdministrationLifecycle/RoleLifecycleResource.php`
- [ ] T053 [P] [US1] Implement academic year detail and update resources using the published lifecycle shape in `app/Http/Resources/AdministrationLifecycle/AcademicYearLifecycleResource.php`
- [ ] T054 [P] [US1] Implement academic period detail and update resources using the published lifecycle shape in `app/Http/Resources/AdministrationLifecycle/AcademicPeriodLifecycleResource.php`
- [ ] T055 [P] [US1] Implement guardian detail and update resources using the published lifecycle shape in `app/Http/Resources/AdministrationLifecycle/GuardianLifecycleResource.php`
- [ ] T056 [P] [US1] Implement administration detail policy methods for school, user, role, academic year, academic period, and guardian records in `app/Policies/AdministrationLifecyclePolicy.php`
- [ ] T057 [P] [US1] Implement administration update policy methods for platform and school scopes in `app/Policies/AdministrationLifecyclePolicy.php`
- [ ] T058 [US1] Implement administration detail service with platform/school scope, tenant, inactive, soft-deleted visibility, and not-found rules in `app/Services/AdministrationLifecycle/AdministrationDetailService.php`
- [ ] T059 [US1] Implement administration update service with mutable-field, immutable-field, tenant, dependency, lifecycle history, and atomic rollback rules in `app/Services/AdministrationLifecycle/AdministrationUpdateService.php`
- [ ] T060 [US1] Implement school lifecycle detail and update controller actions in `app/Http/Controllers/Api/V1/SchoolLifecycleController.php`
- [ ] T061 [US1] Implement school-owned administration detail and update controller actions in `app/Http/Controllers/Api/V1/AdministrationLifecycleController.php`
- [ ] T062 [US1] Wire approved school, user, role, academic year, academic period, and guardian detail/update routes in `routes/api.php`
- [ ] T063 [US1] Update implementation notes with final detail/update operation IDs, mutable-field inventory, immutable-field inventory, route inventory, and test commands in `docs/implementation-notes/007-administration-lifecycle.md`

**Checkpoint**: User Story 1 is independently functional and can be validated without lifecycle transition or bulk behavior.

---

## Phase 4: User Story 2 - Control Activation and Recoverable Removal (Priority: P2)

**Goal**: An authorized administrator can activate, deactivate, soft-delete, and restore eligible records while preserving history and blocking invalid dependency states.

**Independent Test**: Authenticate with the required scope, deactivate a same-school guardian or academic period, verify new operational use is blocked where documented, restore the record, and confirm historical references remain intact.

### Tests for User Story 2

- [ ] T064 [P] [US2] Add PHPUnit feature tests for school activation, deactivation, soft deletion, restoration, inactive-school rejection, platform authorization, lifecycle history, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/SchoolLifecycleTransitionTest.php`
- [ ] T065 [P] [US2] Add PHPUnit feature tests for user activation, deactivation, soft deletion, restoration, active-session or profile dependency conflicts, lifecycle history, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/UserLifecycleTransitionTest.php`
- [ ] T066 [P] [US2] Add PHPUnit feature tests for role activation, deactivation, soft deletion, restoration, active-user assignment conflicts, permission compatibility, lifecycle history, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/RoleLifecycleTransitionTest.php`
- [ ] T067 [P] [US2] Add PHPUnit feature tests for academic year activation, deactivation, soft deletion, restoration, child-period and academic-record conflicts, lifecycle history, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/AcademicYearLifecycleTransitionTest.php`
- [ ] T068 [P] [US2] Add PHPUnit feature tests for academic period activation, deactivation, soft deletion, restoration, grade/attendance/learning-set/report conflicts, lifecycle history, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/AcademicPeriodLifecycleTransitionTest.php`
- [ ] T069 [P] [US2] Add PHPUnit feature tests for guardian activation, deactivation, soft deletion, restoration, student association preservation, lifecycle history, and response shape in `tests/Feature/Api/V1/AdministrationLifecycle/GuardianLifecycleTransitionTest.php`
- [ ] T070 [P] [US2] Add PHPUnit feature tests for unsupported transitions, already-active, already-inactive, already-deleted, missing reason, invalid effective date, cross-tenant records, inactive tenant context, and undocumented fields in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleValidationTest.php`
- [ ] T071 [P] [US2] Add unit tests for transition matrix, dependency conflict behavior, restore eligibility, lifecycle history generation, and rollback on lifecycle failure in `tests/Unit/Services/AdministrationLifecycle/AdministrationLifecycleRulesTest.php`
- [ ] T072 [P] [US2] Add OpenAPI response-shape regression coverage for all approved activate, deactivate, soft-delete, and restore operation IDs in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleContractTest.php`

### Implementation for User Story 2

- [ ] T073 [P] [US2] Implement lifecycle transition input DTO in `app/DTOs/AdministrationLifecycle/ApplyLifecycleTransitionData.php`
- [ ] T074 [P] [US2] Implement activate request validation in `app/Http/Requests/AdministrationLifecycle/ActivateAdministrationResourceRequest.php`
- [ ] T075 [P] [US2] Implement deactivate request validation in `app/Http/Requests/AdministrationLifecycle/DeactivateAdministrationResourceRequest.php`
- [ ] T076 [P] [US2] Implement soft-delete request validation in `app/Http/Requests/AdministrationLifecycle/DeleteAdministrationResourceRequest.php`
- [ ] T077 [P] [US2] Implement restore request validation in `app/Http/Requests/AdministrationLifecycle/RestoreAdministrationResourceRequest.php`
- [ ] T078 [P] [US2] Extend administration lifecycle policy with activate, deactivate, delete, and restore methods for platform and school scopes in `app/Policies/AdministrationLifecyclePolicy.php`
- [ ] T079 [US2] Implement lifecycle transition service with status rules, reason/effective-date validation, dependency checks, soft-delete behavior, restore eligibility, history writes, and atomic rollback in `app/Services/AdministrationLifecycle/AdministrationLifecycleService.php`
- [ ] T080 [US2] Implement school activate, deactivate, soft-delete, and restore controller actions in `app/Http/Controllers/Api/V1/SchoolLifecycleController.php`
- [ ] T081 [US2] Implement school-owned resource activate, deactivate, soft-delete, and restore controller actions in `app/Http/Controllers/Api/V1/AdministrationLifecycleController.php`
- [ ] T082 [US2] Wire approved school, user, role, academic year, academic period, and guardian lifecycle transition routes in `routes/api.php`
- [ ] T083 [US2] Update implementation notes with final lifecycle operation IDs, transition matrix, dependency conflict behavior, soft-delete behavior, restore behavior, inactive-school impact, and test commands in `docs/implementation-notes/007-administration-lifecycle.md`

**Checkpoint**: User Story 2 is independently functional after shared foundations and User Story 1 visibility/update behavior.

---

## Phase 5: User Story 3 - Apply Selected Bulk Lifecycle Actions (Priority: P3)

**Goal**: An authorized administrator can apply one documented lifecycle action to a bounded set of same-scope records with all-or-nothing behavior.

**Independent Test**: Submit a documented bulk deactivate request for a small set of same-school eligible records, verify every requested record is processed atomically according to the contract, and verify any invalid record causes the documented failure behavior without cross-tenant changes.

### Tests for User Story 3

- [ ] T084 [P] [US3] Add PHPUnit feature tests for successful bulk user lifecycle action, affected record result envelope, lifecycle history per record, and all-or-nothing behavior in `tests/Feature/Api/V1/AdministrationLifecycle/BulkUserLifecycleTest.php`
- [ ] T085 [P] [US3] Add PHPUnit feature tests for successful bulk role lifecycle action, assignment conflict handling, result envelope, and all-or-nothing behavior in `tests/Feature/Api/V1/AdministrationLifecycle/BulkRoleLifecycleTest.php`
- [ ] T086 [P] [US3] Add PHPUnit feature tests for successful bulk academic year lifecycle action, child-period conflict handling, result envelope, and all-or-nothing behavior in `tests/Feature/Api/V1/AdministrationLifecycle/BulkAcademicYearLifecycleTest.php`
- [ ] T087 [P] [US3] Add PHPUnit feature tests for successful bulk academic period lifecycle action, academic dependency conflict handling, result envelope, and all-or-nothing behavior in `tests/Feature/Api/V1/AdministrationLifecycle/BulkAcademicPeriodLifecycleTest.php`
- [ ] T088 [P] [US3] Add PHPUnit feature tests for successful bulk guardian lifecycle action, association preservation, result envelope, and all-or-nothing behavior in `tests/Feature/Api/V1/AdministrationLifecycle/BulkGuardianLifecycleTest.php`
- [ ] T089 [P] [US3] Add PHPUnit feature tests for duplicate identifiers, mixed resource types, mixed tenant scopes, unsupported actions, over-limit requests, missing records, unauthorized records, dependency-blocked records, and cross-tenant records in `tests/Feature/Api/V1/AdministrationLifecycle/BulkLifecycleValidationTest.php`
- [ ] T090 [P] [US3] Add unit tests for bulk request validation, all-or-nothing transaction behavior, per-record authorization reuse, dependency check reuse, and history creation in `tests/Unit/Services/AdministrationLifecycle/BulkLifecycleServiceTest.php`
- [ ] T091 [P] [US3] Add OpenAPI response-shape regression coverage for all approved selected bulk lifecycle operation IDs in `tests/Feature/Api/V1/AdministrationLifecycle/BulkLifecycleContractTest.php`

### Implementation for User Story 3

- [ ] T092 [P] [US3] Implement bulk lifecycle input DTO in `app/DTOs/AdministrationLifecycle/ApplyBulkLifecycleActionData.php`
- [ ] T093 [P] [US3] Implement bulk lifecycle request validation in `app/Http/Requests/AdministrationLifecycle/BulkLifecycleActionRequest.php`
- [ ] T094 [P] [US3] Extend administration lifecycle policy with bulk lifecycle authorization for supported resource families in `app/Policies/AdministrationLifecyclePolicy.php`
- [ ] T095 [US3] Implement bulk lifecycle service with one-resource, one-action, one-scope, maximum-count, duplicate rejection, dependency checks, per-record authorization, all-or-nothing transaction, and history rules in `app/Services/AdministrationLifecycle/BulkAdministrationLifecycleService.php`
- [ ] T096 [US3] Implement bulk lifecycle controller actions for approved school-owned resources in `app/Http/Controllers/Api/V1/BulkAdministrationLifecycleController.php`
- [ ] T097 [US3] Wire approved selected bulk lifecycle routes in `routes/api.php`
- [ ] T098 [US3] Update implementation notes with final bulk operation IDs, maximum record count, all-or-nothing behavior, unsupported bulk modes, tenant isolation evidence, and test commands in `docs/implementation-notes/007-administration-lifecycle.md`

**Checkpoint**: User Story 3 is independently functional after shared foundations and single-record lifecycle rules.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final contract compliance, verification, documentation, and cleanup across all selected stories.

- [ ] T099 [P] Add response-shape regression coverage for all administration lifecycle operation IDs, including success, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, bulk-result, and not-found outcomes exactly as declared by OpenAPI, in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleResponseShapeTest.php`
- [ ] T100 [P] Add validation-contract regression coverage for undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, immutable fields, invalid lifecycle transitions, invalid effective dates, duplicate identifiers, mixed-scope bulk requests, inactive references, dependency conflicts, and invalid cross-tenant references in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleValidationContractTest.php`
- [ ] T101 [P] Add tenant-isolation regression coverage across school-owned detail, update, lifecycle, dependency, bulk, and history lookup paths in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleTenantIsolationTest.php`
- [ ] T102 [P] Add authorization matrix regression coverage for platform administrators, school administrators, teachers, students, guardians, inactive users, unauthorized school context, and cross-tenant school context across all administration lifecycle operation IDs in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleAuthorizationTest.php`
- [ ] T103 [P] Add blocked-operation regression coverage for invitations, password setup/reset, account recovery, classroom/course/section/roster, teacher corrections, guardian self-service, report lifecycle expansion, report output lifecycle, platform support access, permanent purge, anonymization, frontend-only behavior, billing, messaging, and notifications in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleBlockedOperationsTest.php`
- [ ] T104 [P] Add end-to-end administration lifecycle happy-path coverage from detail/update through deactivation, restore, soft delete, and selected bulk action for eligible same-school resources in `tests/Feature/Api/V1/AdministrationLifecycle/AdministrationLifecycleHappyPathTest.php`
- [ ] T105 Review implemented backend routes against the blocked-operation list in `routes/api.php`
- [ ] T106 Run backend PHP syntax checks and record result in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T107 Run backend style checks and record result in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T108 Run backend PHPUnit suite with `docker exec schoolmaster-backend-app-1 php artisan test` and record result in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T109 Run Redocly validation and record result in `docs/implementation-notes/007-administration-lifecycle.md`
- [ ] T110 Update implementation notes with final operation IDs, test commands, tenant rules, authorization matrix, lifecycle transition behavior, dependency conflicts, soft-delete behavior, restore behavior, bulk behavior, history preservation behavior, and blocked follow-up contract gaps in `docs/implementation-notes/007-administration-lifecycle.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks every user story.
- **Phase 3 US1**: Depends on Phase 2; recommended MVP.
- **Phase 4 US2**: Depends on Phase 2 and should reuse US1 detail/update visibility and resource behavior.
- **Phase 5 US3**: Depends on Phase 2 and should reuse US2 single-record lifecycle and dependency behavior.
- **Phase 6 Polish**: Depends on all selected user stories for the implementation increment.

### User Story Dependencies

- **US1 Maintain administrative records**: First recommended story because it establishes detail, update, mutable-field validation, immutable-field rejection, resources, and route visibility.
- **US2 Control activation and recoverable removal**: Can start after Phase 2, but should reuse US1 detail/update visibility and lifecycle resources for consistent response shapes.
- **US3 Apply selected bulk lifecycle actions**: Can start after Phase 2, but bulk behavior should reuse US2 transition, dependency, history, and rollback logic.

### Within Each User Story

- Contract and backend tests should be written first and fail before implementation.
- Persistence and model verification precede DTOs, requests, services, and controllers.
- Services own business rules before controllers wire routes.
- Policies and resources must be in place before marking a route complete.
- Routes must not expose blocked operations absent from OpenAPI.

## Parallel Opportunities

- T002 and T003 can run in parallel if maintainers coordinate schema naming, operation IDs, and response components.
- T010 through T032 can run in parallel after T008 and T009 establish the persistence baseline.
- T016 through T021 can run in parallel because dependency checks are split by resource family.
- Test files within each user story can be written in parallel.
- Request, resource, and policy work within US1 and US2 can be implemented in parallel after shared models and rules are verified.
- US2 and US3 can run alongside each other after Phase 2 if implementers coordinate edits to `AdministrationLifecyclePolicy.php`, `AdministrationLifecycleController.php`, and `routes/api.php`.
- T099, T100, T101, T102, T103, and T104 can run in parallel during polish.

## Parallel Example: User Story 1

```text
Task: "T035 Add PHPUnit feature tests for school detail and update platform-scoped success, forbidden school-owned access, immutable field rejection, and response shape in tests/Feature/Api/V1/AdministrationLifecycle/SchoolDetailUpdateTest.php"
Task: "T036 Add PHPUnit feature tests for user detail and update same-school success, tenant isolation, immutable field rejection, role-scope rejection, and response shape in tests/Feature/Api/V1/AdministrationLifecycle/UserDetailUpdateTest.php"
Task: "T037 Add PHPUnit feature tests for role detail and update scope integrity, permission compatibility, tenant isolation, immutable scope rejection, and response shape in tests/Feature/Api/V1/AdministrationLifecycle/RoleDetailUpdateTest.php"
Task: "T043 Implement generic administration update input DTO in app/DTOs/AdministrationLifecycle/UpdateAdministrationResourceData.php"
Task: "T044 Implement school update request validation in app/Http/Requests/AdministrationLifecycle/UpdateSchoolLifecycleRequest.php"
Task: "T050 Implement school detail and update resources using the published lifecycle shape in app/Http/Resources/AdministrationLifecycle/SchoolLifecycleResource.php"
```

## Parallel Example: User Story 2

```text
Task: "T064 Add PHPUnit feature tests for school activation, deactivation, soft deletion, restoration, inactive-school rejection, platform authorization, lifecycle history, and response shape in tests/Feature/Api/V1/AdministrationLifecycle/SchoolLifecycleTransitionTest.php"
Task: "T065 Add PHPUnit feature tests for user activation, deactivation, soft deletion, restoration, active-session or profile dependency conflicts, lifecycle history, and response shape in tests/Feature/Api/V1/AdministrationLifecycle/UserLifecycleTransitionTest.php"
Task: "T073 Implement lifecycle transition input DTO in app/DTOs/AdministrationLifecycle/ApplyLifecycleTransitionData.php"
Task: "T074 Implement activate request validation in app/Http/Requests/AdministrationLifecycle/ActivateAdministrationResourceRequest.php"
Task: "T078 Extend administration lifecycle policy with activate, deactivate, delete, and restore methods for platform and school scopes in app/Policies/AdministrationLifecyclePolicy.php"
```

## Parallel Example: User Story 3

```text
Task: "T084 Add PHPUnit feature tests for successful bulk user lifecycle action, affected record result envelope, lifecycle history per record, and all-or-nothing behavior in tests/Feature/Api/V1/AdministrationLifecycle/BulkUserLifecycleTest.php"
Task: "T085 Add PHPUnit feature tests for successful bulk role lifecycle action, assignment conflict handling, result envelope, and all-or-nothing behavior in tests/Feature/Api/V1/AdministrationLifecycle/BulkRoleLifecycleTest.php"
Task: "T092 Implement bulk lifecycle input DTO in app/DTOs/AdministrationLifecycle/ApplyBulkLifecycleActionData.php"
Task: "T093 Implement bulk lifecycle request validation in app/Http/Requests/AdministrationLifecycle/BulkLifecycleActionRequest.php"
Task: "T094 Extend administration lifecycle policy with bulk lifecycle authorization for supported resource families in app/Policies/AdministrationLifecyclePolicy.php"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 only.
3. Validate US1 independently with detail, update, tenant, immutable-field, dependency, authorization, and response-shape tests.
4. Stop before US2 or US3 if the backend needs an early review checkpoint.

### Incremental Delivery

1. Deliver US1 for detail and update behavior.
2. Deliver US2 for activation, deactivation, soft delete, restore, dependency conflicts, and history preservation.
3. Deliver US3 for selected bulk lifecycle actions and all-or-nothing guarantees.
4. Run Phase 6 after the selected story set is complete.

### Scope Guardrails

- Do not add account invitation, password setup/reset, account recovery, lock recovery, token refresh, direct per-user permission assignment, classroom/course/section/group/roster workflows, teacher correction workflows, guardian self-service, student academic correction workflows, report lifecycle expansion, platform support-user access, frontend implementation, permanent purge, anonymization, legal hold, retention management, billing, messaging, notifications, parent portal behavior, or undocumented APIs.
- Do not add request fields, response fields, filters, sort options, status values, lifecycle actions, bulk modes, status codes, or error envelopes absent from OpenAPI.
- Do not treat platform administrator access as an implicit school-owned lifecycle permission bypass.
- Do not hard-delete recoverable business records in this slice.
