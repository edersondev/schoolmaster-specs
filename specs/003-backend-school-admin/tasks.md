# Tasks: Backend School Administration Foundation

**Input**: Design documents from `specs/003-backend-school-admin/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/backend-school-admin.md`, `quickstart.md`  
**Feature ID**: `003-backend-school-admin`

**Tests**: Required. This slice changes critical backend behavior for authorization, tenant isolation, validation, and response envelopes.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared foundations are in place.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Task can run in parallel with other marked tasks in the same phase when prerequisites are met.
- **[Story]**: User-story label for story phases only.
- Every task includes an exact target file path.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Align backend implementation with the approved specification and contract before code changes.

- [ ] T001 Record feature scope and consumed operation IDs in `README.md`
- [ ] T002 [P] Verify the backend specs submodule or mounted specs path references `schoolmaster-specs/specs/003-backend-school-admin/quickstart.md` in `AGENTS.md`
- [ ] T003 [P] Run contract validation and record the result in `docs/implementation-notes/003-backend-school-admin.md`
- [ ] T004 [P] Confirm no undocumented P1 school-admin routes exist and document the route inventory in `docs/implementation-notes/003-backend-school-admin.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared backend persistence, tenant, authorization, and response infrastructure required by every story.

**Critical**: No user story work should begin until this phase is complete.

- [ ] T005 Inspect existing backend migrations for `User`, `Role`, `Permission`, `AcademicYear`, `AcademicPeriod`, `Guardian`, and `StudentProfile` UUID/status/tenant/soft-delete columns in `database/migrations/`
- [ ] T006 Add only missing school-admin persistence changes and document any non-soft-deletable entity rationale in `database/migrations/` and `docs/implementation-notes/003-backend-school-admin.md`
- [ ] T007 [P] Define shared school-scope query helpers or scopes in `app/Models/Concerns/BelongsToSchool.php`
- [ ] T008 [P] Define tenant context access helpers for school-scoped services in `app/Services/TenantContextService.php`
- [ ] T009 Define shared tenant-context middleware behavior for `X-School-Id` failures in `app/Http/Middleware/ResolveSchoolContext.php`
- [ ] T010 [P] Define shared API error code mappings for tenant mismatch, inactive record, forbidden, and validation failures in `app/Exceptions/Handler.php`
- [ ] T011 [P] Define base pagination/resource envelope helpers for list responses in `app/Http/Resources/Concerns/WrapsApiResponses.php`
- [ ] T012 Seed baseline platform and school permission definitions used by this slice in `database/seeders/PermissionSeeder.php`
- [ ] T013 Define the school-admin route group with auth and tenant middleware only, without operation-specific route registration, in `routes/api.php`

**Checkpoint**: Tenant context, permission seeds, shared envelopes, and route boundaries are ready for story work.

---

## Phase 3: User Story 1 - Administer Tenant Users and Roles (Priority: P1) MVP

**Goal**: A school administrator can list users, create users, list permissions, list roles, and create school-scoped roles within one active school tenant.

**Independent Test**: Authenticate as a school administrator, resolve one active school, create a school-scoped role with school permissions, create a user assigned to that role, and confirm user, role, and permission lists remain tenant-scoped.

### Tests for User Story 1

- [ ] T014 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/permissions` in `tests/Feature/Api/V1/PermissionListTest.php`
- [ ] T015 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/roles` and `POST /api/v1/roles` in `tests/Feature/Api/V1/RoleManagementTest.php`
- [ ] T016 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/users` and `POST /api/v1/users` in `tests/Feature/Api/V1/UserManagementTest.php`
- [ ] T017 [P] [US1] Add unit tests for role-permission scope compatibility in `tests/Unit/Services/RolePermissionScopeTest.php`
- [ ] T018 [P] [US1] Add unit tests for same-school user role assignment validation in `tests/Unit/Services/UserRoleAssignmentTest.php`

### Implementation for User Story 1

- [ ] T019 [P] [US1] Verify or implement `Role` and `Permission` model relationships in `app/Models/Role.php`
- [ ] T020 [P] [US1] Verify or implement permission relationships and active-scope helpers in `app/Models/Permission.php`
- [ ] T021 [P] [US1] Verify or implement user role relationships and school scope helpers in `app/Models/User.php`
- [ ] T022 [US1] Implement role creation input DTO in `app/DTOs/Roles/CreateRoleData.php`
- [ ] T023 [US1] Implement user creation input DTO in `app/DTOs/Users/CreateUserData.php`
- [ ] T024 [P] [US1] Implement role creation validation in `app/Http/Requests/Api/V1/CreateRoleRequest.php`
- [ ] T025 [P] [US1] Implement user creation validation in `app/Http/Requests/Api/V1/CreateUserRequest.php`
- [ ] T026 [P] [US1] Implement role response shape in `app/Http/Resources/Api/V1/RoleResource.php`
- [ ] T027 [P] [US1] Implement permission response shape in `app/Http/Resources/Api/V1/PermissionResource.php`
- [ ] T028 [P] [US1] Implement user response shape in `app/Http/Resources/Api/V1/UserResource.php`
- [ ] T029 [P] [US1] Implement role authorization policy in `app/Policies/RolePolicy.php`
- [ ] T030 [P] [US1] Implement user authorization policy in `app/Policies/UserPolicy.php`
- [ ] T031 [US1] Implement permission listing service in `app/Services/Permissions/PermissionQueryService.php`
- [ ] T032 [US1] Implement role listing and creation service in `app/Services/Roles/RoleService.php`
- [ ] T033 [US1] Implement user listing and creation service in `app/Services/Users/UserService.php`
- [ ] T034 [US1] Implement permission list controller action in `app/Http/Controllers/Api/V1/PermissionController.php`
- [ ] T035 [US1] Implement role list/create controller actions in `app/Http/Controllers/Api/V1/RoleController.php`
- [ ] T036 [US1] Implement user list/create controller actions in `app/Http/Controllers/Api/V1/UserController.php`
- [ ] T037 [US1] Wire `listPermissions`, `listRoles`, `createRole`, `listUsers`, and `createUser` routes in `routes/api.php`

**Checkpoint**: User Story 1 is independently functional and can be validated without academic or guardian workflows.

---

## Phase 4: User Story 2 - Define School Academic Structure (Priority: P2)

**Goal**: A school administrator can create and list academic years and periods inside the active school tenant.

**Independent Test**: Authenticate as a school administrator, create one valid academic year, create ordered periods inside it, list both collections, and verify tenant, inactive-context, date-range, and sequence failures.

### Tests for User Story 2

- [ ] T038 [P] [US2] Add PHPUnit feature tests for `GET /api/v1/academic-years` and `POST /api/v1/academic-years` in `tests/Feature/Api/V1/AcademicYearManagementTest.php`
- [ ] T039 [P] [US2] Add PHPUnit feature tests for `GET /api/v1/academic-periods` and `POST /api/v1/academic-periods` in `tests/Feature/Api/V1/AcademicPeriodManagementTest.php`
- [ ] T040 [P] [US2] Add unit tests for academic period date containment and sequence uniqueness in `tests/Unit/Services/AcademicPeriodValidationTest.php`

### Implementation for User Story 2

- [ ] T041 [P] [US2] Implement or verify `AcademicYear` model fields, relationships, and school scope in `app/Models/AcademicYear.php`
- [ ] T042 [P] [US2] Implement or verify `AcademicPeriod` model fields, relationships, and school scope in `app/Models/AcademicPeriod.php`
- [ ] T043 [US2] Implement academic year creation input DTO in `app/DTOs/AcademicYears/CreateAcademicYearData.php`
- [ ] T044 [US2] Implement academic period creation input DTO in `app/DTOs/AcademicPeriods/CreateAcademicPeriodData.php`
- [ ] T045 [P] [US2] Implement academic year validation in `app/Http/Requests/Api/V1/CreateAcademicYearRequest.php`
- [ ] T046 [P] [US2] Implement academic period validation in `app/Http/Requests/Api/V1/CreateAcademicPeriodRequest.php`
- [ ] T047 [P] [US2] Implement academic year response shape in `app/Http/Resources/Api/V1/AcademicYearResource.php`
- [ ] T048 [P] [US2] Implement academic period response shape in `app/Http/Resources/Api/V1/AcademicPeriodResource.php`
- [ ] T049 [P] [US2] Implement academic year authorization policy in `app/Policies/AcademicYearPolicy.php`
- [ ] T050 [P] [US2] Implement academic period authorization policy in `app/Policies/AcademicPeriodPolicy.php`
- [ ] T051 [US2] Implement academic year listing and creation service in `app/Services/AcademicYears/AcademicYearService.php`
- [ ] T052 [US2] Implement academic period listing and creation service in `app/Services/AcademicPeriods/AcademicPeriodService.php`
- [ ] T053 [US2] Implement academic year list/create controller actions in `app/Http/Controllers/Api/V1/AcademicYearController.php`
- [ ] T054 [US2] Implement academic period list/create controller actions in `app/Http/Controllers/Api/V1/AcademicPeriodController.php`
- [ ] T055 [US2] Wire `listAcademicYears`, `createAcademicYear`, `listAcademicPeriods`, and `createAcademicPeriod` routes in `routes/api.php`

**Checkpoint**: User Story 2 is independently functional after the shared tenant and authorization foundation.

---

## Phase 5: User Story 3 - Maintain Guardian Contact Foundation (Priority: P3)

**Goal**: A school administrator can create and list guardian records, with optional same-school student profile associations.

**Independent Test**: Authenticate as a school administrator, create a guardian with required relationship information and valid same-school student profile references, list guardians for the school, and verify invalid references reject the whole request.

### Tests for User Story 3

- [ ] T056 [P] [US3] Add PHPUnit feature tests for `GET /api/v1/guardians` and `POST /api/v1/guardians` in `tests/Feature/Api/V1/GuardianManagementTest.php`
- [ ] T057 [P] [US3] Add unit tests for atomic guardian-student association validation in `tests/Unit/Services/GuardianAssociationTest.php`

### Implementation for User Story 3

- [ ] T058 [P] [US3] Implement or verify `Guardian` model fields, relationships, and school scope in `app/Models/Guardian.php`
- [ ] T059 [P] [US3] Implement or verify `StudentProfile` relationship helpers needed for guardian association validation in `app/Models/StudentProfile.php`
- [ ] T060 [US3] Implement guardian creation input DTO in `app/DTOs/Guardians/CreateGuardianData.php`
- [ ] T061 [P] [US3] Implement guardian validation in `app/Http/Requests/Api/V1/CreateGuardianRequest.php`
- [ ] T062 [P] [US3] Implement guardian response shape in `app/Http/Resources/Api/V1/GuardianResource.php`
- [ ] T063 [P] [US3] Implement guardian authorization policy in `app/Policies/GuardianPolicy.php`
- [ ] T064 [US3] Implement guardian listing, creation, and atomic student association service in `app/Services/Guardians/GuardianService.php`
- [ ] T065 [US3] Implement guardian list/create controller actions in `app/Http/Controllers/Api/V1/GuardianController.php`
- [ ] T066 [US3] Wire `listGuardians` and `createGuardian` routes in `routes/api.php`

**Checkpoint**: User Story 3 is independently functional after shared foundations and optional student-profile references are validated safely.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final contract compliance, verification, documentation, and cleanup across all selected stories.

- [ ] T067 [P] Run backend test suite and record result in `docs/implementation-notes/003-backend-school-admin.md`
- [ ] T068 [P] Run Redocly validation and record result in `docs/implementation-notes/003-backend-school-admin.md`
- [ ] T069 [P] Add response-shape regression coverage for all eleven operation IDs in `tests/Feature/Api/V1/SchoolAdminResponseShapeTest.php`
- [ ] T070 [P] Add validation-contract regression coverage for undocumented request fields, unsupported filters, unsupported sort values, and invalid payload shapes across all eleven operation IDs in `tests/Feature/Api/V1/SchoolAdminValidationContractTest.php`
- [ ] T071 [P] Add tenant-isolation regression coverage across users, roles, academic years, academic periods, and guardians in `tests/Feature/Api/V1/SchoolAdminTenantIsolationTest.php`
- [ ] T072 Review implemented backend routes against the blocked-operation list in `routes/api.php`
- [ ] T073 Update implementation notes with final operation IDs, test commands, and blocked follow-up contract gaps in `docs/implementation-notes/003-backend-school-admin.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks every user story.
- **Phase 3 US1**: Depends on Phase 2; recommended MVP.
- **Phase 4 US2**: Depends on Phase 2 and can run after or alongside US1 once shared user/permission seeds are stable.
- **Phase 5 US3**: Depends on Phase 2; may use existing student profile data but must not create undocumented student-profile APIs.
- **Phase 6 Polish**: Depends on all selected user stories for the implementation increment.

### User Story Dependencies

- **US1 Administer tenant users and roles**: First recommended story because roles and permissions support later school workflows.
- **US2 Define school academic structure**: Independent after foundation; can proceed without guardian implementation.
- **US3 Maintain guardian contact foundation**: Independent after foundation; validates existing student profile references only.

### Within Each User Story

- Tests should be written first and fail before implementation.
- Model and persistence verification precedes DTOs, requests, services, and controllers.
- Services own business rules before controllers wire routes.
- Policies and resources must be in place before marking a route complete.

## Parallel Opportunities

- T002, T003, and T004 can run in parallel during setup.
- T007, T008, T010, T011, and T012 can run in parallel after T005 and T006 establish the persistence baseline.
- Test files within each user story can be written in parallel.
- Request, resource, and policy files within a story can be implemented in parallel after model relationships are verified.
- US2 and US3 can run in parallel after Phase 2 if separate implementers avoid editing `routes/api.php` at the same time.

## Parallel Example: User Story 1

```text
Task: "T014 Add PHPUnit feature tests for GET /api/v1/permissions in tests/Feature/Api/V1/PermissionListTest.php"
Task: "T015 Add PHPUnit feature tests for GET /api/v1/roles and POST /api/v1/roles in tests/Feature/Api/V1/RoleManagementTest.php"
Task: "T016 Add PHPUnit feature tests for GET /api/v1/users and POST /api/v1/users in tests/Feature/Api/V1/UserManagementTest.php"
Task: "T017 Add unit tests for role-permission scope compatibility in tests/Unit/Services/RolePermissionScopeTest.php"
Task: "T018 Add unit tests for same-school user role assignment validation in tests/Unit/Services/UserRoleAssignmentTest.php"
```

## Parallel Example: User Story 2

```text
Task: "T045 Implement academic year validation in app/Http/Requests/Api/V1/CreateAcademicYearRequest.php"
Task: "T046 Implement academic period validation in app/Http/Requests/Api/V1/CreateAcademicPeriodRequest.php"
Task: "T047 Implement academic year response shape in app/Http/Resources/Api/V1/AcademicYearResource.php"
Task: "T048 Implement academic period response shape in app/Http/Resources/Api/V1/AcademicPeriodResource.php"
Task: "T049 Implement academic year authorization policy in app/Policies/AcademicYearPolicy.php"
Task: "T050 Implement academic period authorization policy in app/Policies/AcademicPeriodPolicy.php"
```

## Parallel Example: User Story 3

```text
Task: "T058 Implement or verify Guardian model fields, relationships, and school scope in app/Models/Guardian.php"
Task: "T059 Implement or verify StudentProfile relationship helpers needed for guardian association validation in app/Models/StudentProfile.php"
Task: "T061 Implement guardian validation in app/Http/Requests/Api/V1/CreateGuardianRequest.php"
Task: "T062 Implement guardian response shape in app/Http/Resources/Api/V1/GuardianResource.php"
Task: "T063 Implement guardian authorization policy in app/Policies/GuardianPolicy.php"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 only.
3. Validate US1 independently with permission, role, and user tests.
4. Stop before US2 or US3 if the backend needs an early review checkpoint.

### Incremental Delivery

1. Deliver US1 for tenant users, roles, and permissions.
2. Deliver US2 for academic years and periods.
3. Deliver US3 for guardians.
4. Run Phase 6 after the selected story set is complete.

### Scope Guardrails

- Do not add update, detail, activate, deactivate, delete, restore, invitation, password-management, student self-service, teacher workflow, or reporting routes.
- Do not add request fields, response fields, filters, sort options, or error envelopes absent from OpenAPI.
- Do not treat platform administrator access as an implicit school-scoped bypass.
- Do not assign permissions directly to users.
