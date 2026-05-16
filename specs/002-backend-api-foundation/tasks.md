# Tasks: Backend API Foundation

**Input**: Design documents from `specs/specs/002-backend-api-foundation/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/backend-readiness.md`, `quickstart.md`

**Tests**: Included because the feature specification explicitly requires feature coverage for API behavior, validation, authorization, tenant isolation, inactive status handling, response shape, token expiry/revocation, login lockout, audit creation, and unit coverage for isolated service behavior.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files and has no dependency on incomplete tasks in the same phase
- **[Story]**: Maps to the user story from `spec.md`
- Every task includes an exact file path

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Prepare the backend repository for API-only work and contract-first validation.

- [ ] T001 Update `README.md` with SchoolMaster backend setup, `/specs` source-of-truth workflow, MySQL setup, `php artisan test`, and Redocly contract validation commands
- [ ] T002 Update `.env.example` to use MySQL defaults for SchoolMaster local development in `.env.example`
- [ ] T003 Update `phpunit.xml` to document the intended test database strategy and keep isolated test environment variables in `phpunit.xml`
- [ ] T004 Create backend architecture placeholder files in `app/DTOs/.gitkeep`, `app/Http/Controllers/Api/V1/.gitkeep`, `app/Http/Requests/.gitkeep`, `app/Http/Resources/.gitkeep`, `app/Policies/.gitkeep`, and `app/Services/.gitkeep`
- [ ] T005 Create product API route entry point in `routes/api.php`
- [ ] T006 Update Laravel routing registration to include API routes in `bootstrap/app.php`
- [ ] T007 Remove the default product-facing welcome route from `routes/web.php`
- [ ] T008 Remove the default product-facing Blade view from `resources/views/welcome.blade.php`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Resolve contract-visible security rules and shared backend primitives before any user story implementation proceeds.

**CRITICAL**: No authentication or school management implementation should begin until this phase is complete.

- [x] T009 Update aggregate OpenAPI auth operations and responses for 8-hour token expiry, logout revocation, inactive user/school token rejection, and lockout response in `specs/api/openapi.yaml`
- [x] T010 Update active feature OpenAPI auth operations and responses for 8-hour token expiry, logout revocation, inactive user/school token rejection, and lockout response in `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [x] T011 Update security guidance with token expiry, lockout, token rejection, and audit requirements in `specs/docs/security.md`
- [ ] T012 [P] Create API response envelope helper in `app/Http/Resources/ApiResponse.php`
- [ ] T013 [P] Create API exception response mapping for validation, unauthorized, forbidden, tenant mismatch, lockout, token rejection, and not found outcomes in `bootstrap/app.php`
- [ ] T014 [P] Create tenant context value object in `app/DTOs/TenantContext.php`
- [ ] T015 [P] Create tenant context resolver service in `app/Services/TenantContextResolver.php`
- [ ] T016 [P] Create audit event service interface and base service in `app/Services/AuditEventService.php`
- [ ] T017 [P] Create base authorization policy scaffolding for platform and school scopes in `app/Policies/ScopePolicy.php`
- [x] T018 Run aggregate and active feature OpenAPI validation and record results in `specs/specs/002-backend-api-foundation/quickstart.md`

**Checkpoint**: Contract-visible security behavior is documented; shared API, tenant, authorization, and audit primitives are ready.

---

## Phase 3: User Story 1 - Establish Backend Readiness (Priority: P1) MVP

**Goal**: A maintainer can verify the backend repository is API-only, aligned with `/specs`, and ready for product implementation.

**Independent Test**: Review setup documentation and run readiness checks confirming `/specs` availability, `/api/v1` API routing, no product Blade route, and contract validation commands.

### Tests for User Story 1

- [ ] T019 [P] [US1] Add feature test for API route registration and no product welcome route in `tests/Feature/BackendReadinessTest.php`
- [ ] T020 [P] [US1] Add feature test for `/specs` source files availability in `tests/Feature/SpecsSubmoduleReadinessTest.php`
- [ ] T021 [P] [US1] Add unit test for route prefix expectations in `tests/Unit/ApiRoutePrefixTest.php`

### Implementation for User Story 1

- [ ] T022 [US1] Wire API route group placeholder under `/api/v1` in `routes/api.php`
- [ ] T023 [US1] Add non-product health route handling policy note to `README.md`
- [ ] T024 [US1] Add source-of-truth and mandatory read-order backend setup section to `AGENTS.md`
- [ ] T025 [US1] Verify `php artisan route:list` output contains no product route outside `/api/v1` and update `specs/specs/002-backend-api-foundation/quickstart.md`
- [ ] T026 [US1] Run `php artisan test` and update readiness result notes in `specs/specs/002-backend-api-foundation/quickstart.md`

**Checkpoint**: User Story 1 is independently testable as backend repository readiness without implementing product behavior.

---

## Phase 4: User Story 2 - Define API and Security Foundations (Priority: P2)

**Goal**: Shared authentication, authorization, validation, response, tenant context, lockout, token rejection, and audit foundations are ready for consistent product endpoint implementation.

**Independent Test**: Validate that shared foundation tests pass without creating undocumented endpoints and that response/tenant/auth security behavior maps to OpenAPI.

### Tests for User Story 2

- [ ] T027 [P] [US2] Add unit tests for API success and error envelope shapes in `tests/Unit/ApiResponseTest.php`
- [ ] T028 [P] [US2] Add unit tests for tenant context resolution states in `tests/Unit/TenantContextResolverTest.php`
- [ ] T029 [P] [US2] Add unit tests for login attempt lockout threshold in `tests/Unit/LoginAttemptControlTest.php`
- [ ] T030 [P] [US2] Add unit tests for audit event payload sanitization in `tests/Unit/AuditEventServiceTest.php`
- [ ] T031 [P] [US2] Add feature tests for token rejection response envelopes in `tests/Feature/AuthTokenRejectionTest.php`

### Implementation for User Story 2

- [ ] T032 [P] [US2] Create login attempt control service for email/IP counting and 15-minute lockout windows in `app/Services/LoginAttemptControlService.php`
- [ ] T033 [P] [US2] Create authentication token lifecycle service for 8-hour expiry and logout revocation in `app/Services/AuthTokenLifecycleService.php`
- [ ] T034 [P] [US2] Create authentication audit event types in `app/DTOs/AuditEventData.php`
- [ ] T035 [US2] Implement sanitized audit recording for login success, login failure, logout, token rejection, and school lifecycle events in `app/Services/AuditEventService.php`
- [ ] T036 [US2] Implement tenant context resolution for session-bound and `X-School-Id` contexts in `app/Services/TenantContextResolver.php`
- [ ] T037 [US2] Implement API response envelope methods for success, paginated, validation, unauthorized, forbidden, tenant mismatch, lockout, token rejection, and not found responses in `app/Http/Resources/ApiResponse.php`
- [ ] T038 [US2] Register shared exception rendering for validation, authorization, tenant mismatch, lockout, token rejection, and not found outcomes in `bootstrap/app.php`
- [ ] T039 [US2] Add Form Request base behavior for rejecting undocumented fields in `app/Http/Requests/ApiFormRequest.php`

**Checkpoint**: User Story 2 is independently testable as shared API/security foundation without requiring school management endpoints.

---

## Phase 5: User Story 3 - Bound First Backend Product Slice (Priority: P3)

**Goal**: The first product slice for authentication and school management can be implemented within the approved OpenAPI boundary.

**Independent Test**: Exercise only approved operations after OpenAPI documents them: `login`, `getCurrentUser`, `logout`, `listSchools`, `createSchool`, `getSchool`, and `updateSchool`, including success, validation, authorization, inactive status, lockout, token rejection, audit, and response-shape behavior.

### Tests for User Story 3

- [ ] T040 [P] [US3] Add authentication feature tests for login success, failed login, lockout, logout revocation, 8-hour token expiry, inactive user, and inactive school in `tests/Feature/AuthApiTest.php`
- [ ] T041 [P] [US3] Add current-user feature tests for resolved school, roles, permissions, unauthorized token, and tenant mismatch cases in `tests/Feature/CurrentUserApiTest.php`
- [ ] T042 [P] [US3] Add school management feature tests for list, create, get, update, activate, deactivate, validation, forbidden, and not found cases in `tests/Feature/SchoolManagementApiTest.php`
- [ ] T043 [P] [US3] Add audit feature tests for auth and school lifecycle events in `tests/Feature/AuditEventApiBehaviorTest.php`

### Implementation for User Story 3

- [ ] T044 [P] [US3] Create School model with UUID, status, and factory support in `app/Models/School.php`
- [ ] T045 [P] [US3] Update User model for UUID, status, school relationship, and role relationship expectations in `app/Models/User.php`
- [ ] T046 [P] [US3] Create Role model and Permission model with scope/status relationships in `app/Models/Role.php` and `app/Models/Permission.php`
- [ ] T047 [P] [US3] Create authentication and school foundation migration in `database/migrations/2026_05_14_000001_create_schoolmaster_foundation_tables.php`
- [ ] T048 [P] [US3] Create AuditEvent model and migration for append-only auth/school audit records in `app/Models/AuditEvent.php` and `database/migrations/2026_05_14_000002_create_audit_events_table.php`
- [ ] T049 [US3] Create login request validation for email, password, optional school_id, and undocumented-field rejection in `app/Http/Requests/Auth/LoginRequest.php`
- [ ] T050 [US3] Create school create and update request validation in `app/Http/Requests/Schools/StoreSchoolRequest.php` and `app/Http/Requests/Schools/UpdateSchoolRequest.php`
- [ ] T051 [US3] Create AuthSession, School, User, Role, Permission, and AuditEvent API resources in `app/Http/Resources/AuthSessionResource.php`, `app/Http/Resources/SchoolResource.php`, `app/Http/Resources/UserResource.php`, `app/Http/Resources/RoleResource.php`, `app/Http/Resources/PermissionResource.php`, and `app/Http/Resources/AuditEventResource.php`
- [ ] T052 [US3] Implement authentication service for login, current user, logout revocation, token expiry checks, inactive checks, lockout, and audit events in `app/Services/AuthService.php`
- [ ] T053 [US3] Implement school service for list, create, get, update, activate/deactivate, platform authorization checks, and audit events in `app/Services/SchoolService.php`
- [ ] T054 [US3] Create AuthController for `login`, `me`, and logout-adjacent revocation behavior if documented by OpenAPI in `app/Http/Controllers/Api/V1/AuthController.php`
- [ ] T055 [US3] Create SchoolController for list, create, get, and update operations in `app/Http/Controllers/Api/V1/SchoolController.php`
- [ ] T056 [US3] Register approved `/api/v1/auth/login`, `/api/v1/auth/me`, `/api/v1/auth/logout`, `/api/v1/schools`, and `/api/v1/schools/{schoolId}` routes only after OpenAPI documents them in `routes/api.php`
- [ ] T057 [US3] Create school and platform authorization policies in `app/Policies/SchoolPolicy.php`
- [ ] T058 [US3] Seed baseline platform/school permissions and roles needed for first-slice tests in `database/seeders/DatabaseSeeder.php`
- [ ] T059 [US3] Run OpenAPI validation and backend tests, then record first-slice verification results in `specs/specs/002-backend-api-foundation/quickstart.md`

**Checkpoint**: User Story 3 delivers the bounded first product slice and remains limited to approved authentication and school management operations.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final quality checks and documentation consistency across the feature.

- [ ] T060 [P] Review all changed backend files for PSR-12 compliance and run formatter if configured in `composer.json`
- [ ] T061 [P] Review OpenAPI operation IDs implemented by backend and document traceability in `README.md`
- [ ] T062 [P] Review tenant terminology and replace implementation-facing `tenant_id` references with `school_id` where v1 school-owned records are meant in `app/` and `database/`
- [ ] T063 [P] Run `php artisan route:list` and verify no undocumented product route exists outside `routes/api.php`
- [ ] T064 Run full validation from `specs/specs/002-backend-api-foundation/quickstart.md`
- [ ] T065 Update `specs/specs/002-backend-api-foundation/tasks.md` with completion notes only after validation commands pass

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; can start immediately.
- **Foundational (Phase 2)**: Depends on Setup completion; blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational; MVP readiness slice.
- **User Story 2 (Phase 4)**: Depends on Foundational; can run after or alongside US1 if setup is stable.
- **User Story 3 (Phase 5)**: Depends on Foundational and should start after US2 shared services are available.
- **Polish (Phase 6)**: Depends on selected story phases being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational; does not depend on US2 or US3.
- **User Story 2 (P2)**: Can start after Foundational; shared services support US3.
- **User Story 3 (P3)**: Depends on US2 shared API/security foundations for clean implementation.

### Within Each User Story

- Tests must be written first and fail before implementation.
- Shared DTOs/resources/services before controllers.
- Models and migrations before services that persist data.
- Services before controllers.
- Routes after controller/request/resource classes exist.
- Validation and OpenAPI checks before marking a story complete.

## Parallel Opportunities

- T012 through T017 can run in parallel after T009 through T011 are complete.
- T019 through T021 can run in parallel.
- T027 through T031 can run in parallel.
- T032 through T034 can run in parallel before T035 through T039.
- T040 through T043 can run in parallel.
- T044 through T048 can run in parallel before T049 through T058.
- T060 through T063 can run in parallel during polish.

## Parallel Example: User Story 1

```bash
Task: "Add feature test for API route registration and no product welcome route in tests/Feature/BackendReadinessTest.php"
Task: "Add feature test for /specs source files availability in tests/Feature/SpecsSubmoduleReadinessTest.php"
Task: "Add unit test for route prefix expectations in tests/Unit/ApiRoutePrefixTest.php"
```

## Parallel Example: User Story 2

```bash
Task: "Add unit tests for API success and error envelope shapes in tests/Unit/ApiResponseTest.php"
Task: "Add unit tests for tenant context resolution states in tests/Unit/TenantContextResolverTest.php"
Task: "Add unit tests for login attempt lockout threshold in tests/Unit/LoginAttemptControlTest.php"
Task: "Add unit tests for audit event payload sanitization in tests/Unit/AuditEventServiceTest.php"
```

## Parallel Example: User Story 3

```bash
Task: "Create School model with UUID, status, and factory support in app/Models/School.php"
Task: "Create Role model and Permission model with scope/status relationships in app/Models/Role.php and app/Models/Permission.php"
Task: "Create AuditEvent model and migration for append-only auth/school audit records in app/Models/AuditEvent.php and database/migrations/"
```

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational contract and shared primitives.
3. Complete Phase 3: User Story 1.
4. Stop and validate backend readiness independently.

### Incremental Delivery

1. Setup + Foundational: repository and contract gates are ready.
2. User Story 1: backend readiness is demonstrable without product behavior.
3. User Story 2: shared API/security foundations are tested.
4. User Story 3: first bounded auth/school product slice is implemented.
5. Polish: route, contract, tenant, audit, and documentation checks pass.

### Contract-First Guardrail

Authentication implementation tasks in US3 must not start until T009, T010, and T011 document token expiry, logout revocation, token rejection, lockout, and audit behavior in `/specs` and OpenAPI.

## Notes

- `[P]` tasks touch different files and can run in parallel after their dependencies are satisfied.
- `[US1]`, `[US2]`, and `[US3]` map directly to `spec.md` user stories.
- Exact routes, request fields, response fields, filters, status codes, and error envelopes must come from OpenAPI.
- Do not add product endpoints beyond the first-slice operations documented by OpenAPI after T009 and T010.
