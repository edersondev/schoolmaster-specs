# Tasks: Administration Foundation UI

**Input**: Design documents from `specs/018-administration-foundation-ui/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/administration-foundation-ui-contract.md`, `quickstart.md`

**Tests**: Frontend behavior changes require Vitest coverage for services,
contracts, composables, routes, shared components, tenant handling, forms, and
critical user flows.

**Organization**: Tasks are grouped by user story. Implementation paths are
relative to `schoolmaster-frontend`; specification paths are relative to
`schoolmaster-specs`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because task changes different files and does not
  depend on another incomplete task.
- **[Story]**: Maps task to a user story from `spec.md`.
- Run specification tasks from `schoolmaster-specs`.
- Run frontend tasks from `schoolmaster-frontend`.
- No backend or OpenAPI implementation changes are planned.

## Phase 1: Setup

**Purpose**: Confirm contract readiness and create feature-local frontend test
and localization scaffolding.

- [X] T001 Verify all 14 approved operation IDs, paths, parameters, request schemas, envelopes, backend readiness, the exact frontend permission matrix, and feature 017's unresolved-school selection gate against `api/openapi.yaml`, `specs/017-auth-session-ui/`, and `../schoolmaster-backend/database/seeders/PermissionSeeder.php`, then record any blocking mismatch in `specs/018-administration-foundation-ui/quickstart.md`
- [X] T002 [P] Create administration test fixture scaffold for sessions, tenants, envelopes, errors, and resource records in `tests/unit/admin-system/administration/administration.fixtures.js`
- [X] T003 [P] Create centralized administration locale scaffold for navigation, lists, forms, feedback, validation summaries, and discard confirmation in `src/locales/administration.js`
- [X] T004 Register administration locale messages through the existing i18n assembly used by `src/main.js` and `src/locales/administration.js`

---

## Phase 2: Foundational

**Purpose**: Shared contract, query, service-error, list, form, route, and
unsaved-change infrastructure required by every user story.

**CRITICAL**: No user story implementation begins before this phase completes.

### Foundational Tests

- [X] T005 [P] Add Vitest coverage for administration JSDoc envelope mapping, list metadata, feedback states, and form/list model factories in `tests/unit/admin-system/administration/contracts/administration.contract.spec.js`
- [X] T006 [P] Add Vitest coverage for approved query allowlists, school sort exclusion, parsing, serialization, malformed-value normalization, filter page reset, and unknown-key removal in `tests/unit/admin-system/administration/composables/useAdminListQuery.spec.js`
- [X] T007 [P] Add Vitest coverage for validation, unauthorized, forbidden, tenant-mismatch, inactive-context, not-found, unavailable, unknown, and safe diagnostic mapping in `tests/unit/admin-system/administration/services/administration-error-mapper.spec.js`
- [X] T008 [P] Add Vitest coverage for latest-request protection, empty final-page recovery, retry, tenant-generation reset, paginated role/permission/academic-year lookup traversal, approved lookup parameters, and selected-option retention in `tests/unit/admin-system/administration/composables/useAdminList.spec.js` and `tests/unit/admin-system/administration/composables/useAdminLookup.spec.js`
- [X] T009 [P] Add Vitest coverage for dirty-state derivation, duplicate-submit prevention, field-error preservation, success reset, and return-query behavior in `tests/unit/admin-system/administration/composables/useAdminCreateForm.spec.js`
- [X] T010 [P] Add Vitest coverage for browser and in-app route exit confirmation, successful-submit bypass, unchanged-form bypass, and school-switch confirmation in `tests/unit/admin-system/administration/composables/useUnsavedChangesGuard.spec.js`
- [X] T011 [P] Add Vitest coverage for shared list frame, authorized create slot, loading, base empty, filtered empty, retry, and denial rendering in `tests/unit/admin-system/administration/components/AdminListPage.spec.js`
- [X] T012 [P] Add Vitest coverage for approved filter emits, reset behavior, operation-gated remote sort emits, pagination emits, responsive rendering at 390px, 768px, and 1440px, and keyboard access in `tests/unit/admin-system/administration/components/AdminListControls.spec.js`
- [X] T013 [P] Add Vitest coverage for shared form frame, accessible validation summary, pending submit state, cancel behavior, success feedback, and focus management in `tests/unit/admin-system/administration/components/AdminFormPage.spec.js`
- [X] T014 [P] Add Vitest coverage for administration route metadata schema, lazy loading, auth requirement, school-context requirement, unresolved-school blocking without administration requests, the exact single- and multi-permission matrix, breadcrumb, and sidebar placement in `tests/unit/admin-system/administration/routes/administration.routes.spec.js`

### Foundational Implementation

- [X] T015 Define shared JSDoc contracts and mapping helpers for list queries, paginated results, feedback, form state, and safe diagnostics in `src/contracts/admin-system/administration.js`
- [X] T016 Implement per-resource query allowlists including school sort exclusion, parse/serialize helpers, approved defaults, and page-reset rules in `src/composables/admin-system/useAdminListQuery.js`
- [X] T017 Implement normalized administration error mapping without sensitive payload logging in `src/services/admin-system/administration-error-mapper.js`
- [X] T018 Implement latest-request-wins list orchestration, scoped loading, retry, tenant clearing, and invalidated-page recovery in `src/composables/admin-system/useAdminList.js`, plus reusable paginated lookup orchestration with approved parameter allowlists and selected-option retention in `src/composables/admin-system/useAdminLookup.js`
- [X] T019 Implement create-form orchestration for dirty state, pending submit, validation mapping, success reset, and validated list return query in `src/composables/admin-system/useAdminCreateForm.js`
- [X] T020 Implement reusable browser and in-app leave protection with centralized discard confirmation in `src/composables/admin-system/useUnsavedChangesGuard.js`
- [X] T021 [P] Implement shared administration feedback rendering for loading, empty, filtered-empty, forbidden, tenant-mismatch, inactive, not-found, unavailable, and retry states in `src/components/ui/admin/AdminFeedbackState.vue`
- [X] T022 [P] Implement shared page header, authorized create action, filter/table/pagination slots, and feedback composition in `src/components/ui/admin/AdminListPage.vue`
- [X] T023 [P] Implement approved filter controls with explicit props/emits and no transport logic in `src/components/ui/admin/AdminFilterBar.vue`
- [X] T024 [P] Implement remote-sort data table wrapper with loading, empty slots, responsive primary-field presentation, and accessible labels in `src/components/ui/admin/AdminDataTable.vue`
- [X] T025 [P] Implement paginated-envelope controls with page and page-size emits in `src/components/ui/admin/AdminPagination.vue`
- [X] T026 [P] Implement dedicated create-page frame with validation summary, pending submit, cancel, feedback, and explicit slots in `src/components/ui/admin/AdminFormPage.vue`
- [X] T027 Create lazy route assembly and shared route-meta helpers in `src/router/modules/administration.routes.js`, with empty resource module exports in `src/router/modules/schools.routes.js`, `src/router/modules/access-administration.routes.js`, `src/router/modules/academics.routes.js`, and `src/router/modules/guardians.routes.js`
- [X] T028 Register administration routes through the existing router module assembly in `src/router/index.js`
- [X] T029 Integrate administration navigation metadata and hidden permission behavior with existing shell definitions in `src/router/modules/admin-system.routes.js`

**Checkpoint**: Shared contracts, composables, components, routes, errors, tenant
reset, and unsaved guards are testable before resource modules begin.

---

## Phase 3: User Story 1 - Manage Schools from the Platform Workspace (Priority: P1) MVP

**Goal**: System administrator lists schools with approved status and
pagination controls and creates a school through a dedicated route.

**Independent Test**: Authenticate with platform school permissions, exercise
list query controls and empty states, create valid and invalid schools, and
verify direct-route denial without tenant-owned dependencies.

### Tests for User Story 1

- [X] T030 [P] [US1] Add contract-mapping tests for School records, school list queries, and SchoolCreateForm payloads in `tests/unit/admin-system/administration/contracts/schools.contract.spec.js`
- [X] T031 [P] [US1] Add service tests for `listSchools` and `createSchool`, approved parameters, payload mapping, envelopes, validation, forbidden, and stale cancellation in `tests/unit/admin-system/administration/services/schools.spec.js`
- [X] T032 [P] [US1] Add component tests for school status/pagination controls, absence of sort UI, responsive rows, and create fields in `tests/unit/admin-system/administration/components/SchoolsModule.spec.js`
- [X] T033 [P] [US1] Add page-flow tests for URL query restoration, base/filtered empty states, create success return, validation preservation, duplicate-submit prevention, and forbidden direct access in `tests/unit/admin-system/administration/pages/SchoolsPages.spec.js`

### Implementation for User Story 1

- [X] T034 [P] [US1] Define School, SchoolCreateForm, list-query, and mapper contracts in `src/contracts/admin-system/schools.js`
- [X] T035 [US1] Implement explicit `listSchools` and `createSchool` service functions in `src/services/admin-system/schools.js`
- [X] T036 [P] [US1] Implement approved school status controls without sort UI in `src/components/admin-system/schools/SchoolFilters.vue`
- [X] T037 [P] [US1] Implement school columns and responsive primary-field rendering in `src/components/admin-system/schools/SchoolTable.vue`
- [X] T038 [P] [US1] Implement contract-shaped school create fields and client guidance in `src/components/admin-system/schools/SchoolForm.vue`
- [X] T039 [US1] Compose school list behavior, permission visibility, query state, feedback, and pagination in `src/pages/admin-system/schools/SchoolsListPage.vue`
- [X] T040 [US1] Compose dedicated school create behavior, validation, leave guard, and list return in `src/pages/admin-system/schools/CreateSchoolPage.vue`
- [X] T041 [US1] Add school list/create routes, localized metadata, sidebar entry, `schools.view` list permission, and `schools.view` plus `schools.manage` create permissions in `src/router/modules/schools.routes.js`

**Checkpoint**: Schools list/create works independently as MVP.

---

## Phase 4: User Story 2 - Manage Tenant Users and Access Definitions (Priority: P2)

**Goal**: School administrator lists users, roles, and permissions, creates
school-scoped roles, and creates users through role assignment only.

**Independent Test**: Resolve one active school, list all three resources,
create a compatible role, create a user with that role, and verify missing,
mismatched, inactive, forbidden, validation, and school-switch states.

### Tests for User Story 2

- [X] T042 [P] [US2] Add contract tests for User, UserCreateForm, Role, school-only RoleCreateForm, fixed `scope=school` payload mapping, Permission, and access mapper behavior in `tests/unit/admin-system/administration/contracts/access.contract.spec.js`
- [X] T043 [P] [US2] Add user service tests for approved list query, tenant header, create payload, role IDs, errors, and stale response protection in `tests/unit/admin-system/administration/services/users.spec.js`
- [X] T044 [P] [US2] Add role service tests for approved list query, tenant header, fixed `scope=school` create payload, school permission IDs, errors, and stale response protection in `tests/unit/admin-system/administration/services/roles.spec.js`
- [X] T045 [P] [US2] Add permission service tests for read-only pagination, tenant header, envelope mapping, denial, and absence of mutation functions in `tests/unit/admin-system/administration/services/permissions.spec.js`
- [X] T046 [P] [US2] Add component tests for user/role/permission filters, tables, paginated role choices, paginated school-scope permission choices, selected-option retention, absence of role scope control, and no direct user permission assignment in `tests/unit/admin-system/administration/components/AccessAdministrationModules.spec.js`
- [X] T047 [P] [US2] Add page-flow tests for user, role, and permission lists, URL query state, paginated lookup traversal beyond the first page, create success, field validation, direct denial, unresolved-school blocking, tenant clearing, and dirty school-switch confirmation in `tests/unit/admin-system/administration/pages/AccessAdministrationPages.spec.js`

### Implementation for User Story 2

- [X] T048 [P] [US2] Define User and UserCreateForm contracts and mappers in `src/contracts/admin-system/users.js`
- [X] T049 [P] [US2] Define Role, school-only RoleCreateForm, Permission, fixed `scope=school` payload mapping, and school-scope compatibility in `src/contracts/admin-system/access.js`
- [X] T050 [P] [US2] Implement explicit `listUsers` and `createUser` service functions in `src/services/admin-system/users.js`
- [X] T051 [P] [US2] Implement explicit `listRoles` and `createRole` service functions in `src/services/admin-system/roles.js`
- [X] T052 [P] [US2] Implement read-only `listPermissions` service function in `src/services/admin-system/permissions.js`
- [X] T053 [P] [US2] Implement user filters, table, and role-only create form in `src/components/admin-system/users/UserFilters.vue`, `src/components/admin-system/users/UserTable.vue`, and `src/components/admin-system/users/UserForm.vue`
- [X] T054 [P] [US2] Implement role filters, table, and school-permission create form with no editable scope control in `src/components/admin-system/roles/RoleFilters.vue`, `src/components/admin-system/roles/RoleTable.vue`, and `src/components/admin-system/roles/RoleForm.vue`
- [X] T055 [P] [US2] Implement read-only permission table without create or lifecycle actions in `src/components/admin-system/permissions/PermissionTable.vue`
- [X] T056 [US2] Compose user list and dedicated create routes with paginated role lookup, selected-option retention, tenant gating, errors, and unsaved guard in `src/pages/admin-system/users/UsersListPage.vue` and `src/pages/admin-system/users/CreateUserPage.vue`
- [X] T057 [US2] Compose role list and school-only dedicated create routes with paginated permission lookup, selected-option retention, fixed school scope, tenant gating, errors, and unsaved guard in `src/pages/admin-system/roles/RolesListPage.vue` and `src/pages/admin-system/roles/CreateRolePage.vue`
- [X] T058 [US2] Compose read-only permission list with pagination, tenant gating, denial, and empty states in `src/pages/admin-system/permissions/PermissionsListPage.vue`
- [X] T059 [US2] Add user, role, and permission route records with exact requirements: `users.view`; `users.view` plus `users.manage` plus `roles.view`; `roles.view`; `roles.view` plus `roles.manage` plus `permissions.view`; and `permissions.view`, including school-context metadata in `src/router/modules/access-administration.routes.js`

**Checkpoint**: Access administration works independently inside one active
school and exposes no direct permission assignment or lifecycle action.

---

## Phase 5: User Story 3 - Configure the Academic Calendar (Priority: P3)

**Goal**: School administrator lists and creates academic years and periods,
including approved status and academic-year filtering.

**Independent Test**: In one active school create a year, create a period within
it, filter periods by year, and verify date, sequence, missing-parent,
not-found, tenant, pagination, and unsaved-route behavior.

### Tests for User Story 3

- [X] T060 [P] [US3] Add contract tests for AcademicYear, AcademicYearCreateForm, AcademicPeriod, AcademicPeriodCreateForm, date mapping, and sequence mapping in `tests/unit/admin-system/administration/contracts/academics.contract.spec.js`
- [X] T061 [P] [US3] Add academic-year service tests for approved status/pagination query, tenant header, create payload, validation, denial, and stale response protection in `tests/unit/admin-system/administration/services/academic-years.spec.js`
- [X] T062 [P] [US3] Add academic-period service tests for approved status/year/pagination query, tenant header, create payload, validation, not-found, and stale response protection in `tests/unit/admin-system/administration/services/academic-periods.spec.js`
- [X] T063 [P] [US3] Add component tests for year/period filters, tables, date fields, sequence field, paginated parent-year choices, selected-option retention, and responsive rendering in `tests/unit/admin-system/administration/components/AcademicAdministrationModules.spec.js`
- [X] T064 [P] [US3] Add page-flow tests for year and period lists, year filter query, parent-year lookup traversal beyond the first page, create success, date/sequence validation, unavailable parent, unresolved-school blocking, tenant clearing, and unsaved guards in `tests/unit/admin-system/administration/pages/AcademicAdministrationPages.spec.js`

### Implementation for User Story 3

- [X] T065 [US3] Define academic year/period records, forms, list queries, date helpers, and mappers in `src/contracts/admin-system/academics.js`
- [X] T066 [P] [US3] Implement explicit `listAcademicYears` and `createAcademicYear` service functions in `src/services/admin-system/academic-years.js`
- [X] T067 [P] [US3] Implement explicit `listAcademicPeriods` and `createAcademicPeriod` service functions in `src/services/admin-system/academic-periods.js`
- [X] T068 [P] [US3] Implement academic-year filters, table, and create form in `src/components/admin-system/academic-years/AcademicYearFilters.vue`, `src/components/admin-system/academic-years/AcademicYearTable.vue`, and `src/components/admin-system/academic-years/AcademicYearForm.vue`
- [X] T069 [P] [US3] Implement academic-period filters, table, and create form with parent-year and sequence controls in `src/components/admin-system/academic-periods/AcademicPeriodFilters.vue`, `src/components/admin-system/academic-periods/AcademicPeriodTable.vue`, and `src/components/admin-system/academic-periods/AcademicPeriodForm.vue`
- [X] T070 [US3] Compose academic-year list and create pages with tenant gating, validation, query state, and leave guard in `src/pages/admin-system/academic-years/AcademicYearsListPage.vue` and `src/pages/admin-system/academic-years/CreateAcademicYearPage.vue`
- [X] T071 [US3] Compose academic-period list and create pages with paginated year lookup and selected-option retention, year filtering, tenant gating, safe not-found state, query state, and leave guard in `src/pages/admin-system/academic-periods/AcademicPeriodsListPage.vue` and `src/pages/admin-system/academic-periods/CreateAcademicPeriodPage.vue`
- [X] T072 [US3] Add academic route records with exact requirements: `academic_years.view`; `academic_years.view` plus `academic_years.manage`; `academic_periods.view`; and `academic_periods.view` plus `academic_periods.manage` plus `academic_years.view`, including school-context metadata in `src/router/modules/academics.routes.js`

**Checkpoint**: Academic calendar foundation works independently for one active
school.

---

## Phase 6: User Story 4 - Create and List Guardians (Priority: P4)

**Goal**: School administrator lists guardians and creates one with optional
same-school student associations selected through remote lookup.

**Independent Test**: In one active school list guardians, search/page student
lookup, create a guardian with valid associations, and verify invalid,
inactive, stale-school, cross-tenant-safe, empty, denial, and no-partial-success
states.

### Tests for User Story 4

- [X] T073 [P] [US4] Add contract tests for Guardian, GuardianCreateForm, StudentProfileLookupOption, UUID selection, and mappers in `tests/unit/admin-system/administration/contracts/guardians.contract.spec.js`
- [X] T074 [P] [US4] Add guardian service tests for approved list query, tenant header, create payload, student profile IDs, validation, denial, and no-partial-success mapping in `tests/unit/admin-system/administration/services/guardians.spec.js`
- [X] T075 [P] [US4] Add student-profile lookup service tests for mandatory `status=active`, approved search/sort/pagination parameters, tenant header, envelope mapping, and stale cancellation in `tests/unit/admin-system/administration/services/student-profiles.spec.js`
- [X] T076 [P] [US4] Add lookup composable tests for remote search, pagination, selected option retention, old-search rejection, school reset, and unavailable states in `tests/unit/admin-system/administration/composables/useStudentProfileLookup.spec.js`
- [X] T077 [P] [US4] Add guardian component tests for list filters/table, contact fields, relationship field, hidden student lookup without `student_profiles.view`, remote multi-select with permission, loading, empty lookup, keyboard use, and UUID emits in `tests/unit/admin-system/administration/components/GuardiansModule.spec.js`
- [X] T078 [P] [US4] Add guardian page-flow tests for list/create, query state, lookup association, validation preservation, safe missing/cross-tenant response, denial, school switch, and unsaved guard in `tests/unit/admin-system/administration/pages/GuardiansPages.spec.js`

### Implementation for User Story 4

- [X] T079 [P] [US4] Define Guardian, GuardianCreateForm, StudentProfileLookupOption, and mapper contracts in `src/contracts/admin-system/guardians.js`
- [X] T080 [P] [US4] Implement explicit `listGuardians` and `createGuardian` service functions in `src/services/admin-system/guardians.js`
- [X] T081 [P] [US4] Implement lookup-only `listStudentProfiles` service function that always sends `status=active` with approved remote parameters in `src/services/admin-system/student-profiles.js`
- [X] T082 [US4] Implement remote student lookup orchestration with cancellation, pagination, selected option retention, and tenant reset in `src/composables/admin-system/useStudentProfileLookup.js`
- [X] T083 [P] [US4] Implement guardian status filters and responsive table in `src/components/admin-system/guardians/GuardianFilters.vue` and `src/components/admin-system/guardians/GuardianTable.vue`
- [X] T084 [US4] Implement guardian create fields and conditionally visible remote same-school student multi-select gated by `student_profiles.view` in `src/components/admin-system/guardians/GuardianForm.vue`
- [X] T085 [US4] Compose guardian list and dedicated create pages with lookup, validation, no-partial-success feedback, tenant gating, and leave guard in `src/pages/admin-system/guardians/GuardiansListPage.vue` and `src/pages/admin-system/guardians/CreateGuardianPage.vue`
- [X] T086 [US4] Add guardian list/create route records with `guardians.view` and `guardians.view` plus `guardians.manage`, keep `student_profiles.view` as lookup-control permission, and add localized metadata, sidebar entry, and school-context requirements in `src/router/modules/guardians.routes.js`

**Checkpoint**: Guardian foundation works independently and student profiles are
used only as same-school lookup data.

---

## Phase 7: Polish and Cross-Cutting Concerns

**Purpose**: Full-feature integration, accessibility, security, performance,
scope, and documentation verification.

- [X] T087 [P] Add cross-module Vitest coverage for shell navigation reachability within two interactions, exact permission-matrix visibility, multi-permission create routes, hidden unauthorized administration destinations, and blocked unresolved school selection without tenant-owned requests in `tests/unit/admin-system/administration/routes/administration.navigation.spec.js`
- [X] T088 [P] Add cross-module Vitest coverage that tenant switches clear all tenant-owned lists, lookups, requests, and drafts without affecting platform school state in `tests/unit/admin-system/administration/composables/administration.tenant-isolation.spec.js`
- [X] T089 [P] Add cross-module Vitest coverage with the app bootstrapped and mocked service latency of 1.5 seconds, asserting stable list data, empty state, or recoverable error within 2 seconds from route navigation or committed query change, plus scoped loading, stale response rejection, and retry behavior in `tests/unit/admin-system/administration/pages/administration.performance.spec.js`
- [X] T090 [P] Add accessibility coverage at 390px, 768px, and 1440px for labels, focus, validation summaries, announced feedback, table alternatives, pagination, forms, and discard confirmation in `tests/unit/admin-system/administration/components/administration.accessibility.spec.js`
- [X] T091 Audit administration services and diagnostics for operation IDs, safe error codes, request IDs, and absence of form values, emails, tokens, tenant data, or permission payload logging in `src/services/admin-system/`
- [X] T092 Audit administration UI for PascalCase Element Plus tags, centralized locale text, responsive desktop/tablet/mobile behavior, and no direct Axios or endpoint strings in `src/pages/admin-system/`, `src/components/admin-system/`, `src/components/ui/admin/`, `src/composables/admin-system/`, and `src/router/`
- [X] T093 Verify no detail, update, activate, deactivate, delete, restore, bulk, account lifecycle, guardian user-link, or student management actions exist in `src/router/modules/administration.routes.js`, `src/router/modules/schools.routes.js`, `src/router/modules/access-administration.routes.js`, `src/router/modules/academics.routes.js`, `src/router/modules/guardians.routes.js`, `src/pages/admin-system/`, and `src/components/admin-system/`
- [X] T094 Run administration unit tests with `npm run test:unit -- --run tests/unit/admin-system/administration`
- [X] T095 Run full frontend unit suite with `npm run test:unit -- --run`
- [X] T096 Run production build with `npm run build`
- [ ] T097 Execute manual review at 390px, 768px, and 1440px plus keyboard, denial, tenant-switch, list-query, create-return, deterministic 1.5-second service-latency checks, and moderated UAT with five administrators and 30 total create attempts from `specs/018-administration-foundation-ui/quickstart.md`, then record results and confirm at least 27 unguided first-attempt completions in `specs/018-administration-foundation-ui/quickstart.md`
- [ ] T098 Update roadmap item 4 from specified to complete and record frontend artifact links after verified frontend completion in `docs/frontend-feature-roadmap.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Phase 1 Setup**: Starts immediately.
- **Phase 2 Foundational**: Depends on Setup; blocks all stories.
- **Phase 3 US1**: Depends on Foundational; recommended MVP.
- **Phase 4 US2**: Depends on Foundational; may run parallel with US1, but user
  create uses role data implemented inside US2.
- **Phase 5 US3**: Depends on Foundational; independent of US1 and US2.
- **Phase 6 US4**: Depends on Foundational; independent of prior stories.
- **Phase 7 Polish**: Depends on all desired story phases.

### User Story Dependencies

- **US1 Schools**: No story dependency.
- **US2 Access Definitions**: No story dependency; role service/form must finish
  before user create integration inside US2.
- **US3 Academic Calendar**: No story dependency; academic-year service/form
  must finish before period create integration inside US3.
- **US4 Guardians**: No story dependency; student lookup service/composable must
  finish before guardian create integration inside US4.

### Within Each Story

1. Write contract, service, component, and page tests first.
2. Implement JSDoc contracts and mappers.
3. Implement explicit service functions.
4. Implement resource components.
5. Compose route pages and dependent lookups.
6. Register route metadata/navigation.
7. Run story tests at checkpoint.

### Parallel Opportunities

- Setup fixtures and locale scaffold can run in parallel.
- Foundational tests T005-T014 can run in parallel.
- Shared components T021-T026 can run in parallel after their tests.
- After Foundational, US1-US4 may run in parallel with separate owners because
  each story owns a separate resource route module.
- Within US2, user, role, and permission services/components can run in
  parallel before page integration.
- Within US3, year and period service/component work can run in parallel before
  page integration.
- Within US4, guardian and student-profile services can run in parallel before
  lookup/form integration.

---

## Parallel Examples

### User Story 1

```text
T030 School contract tests
T031 School service tests
T032 School component tests
T033 School page-flow tests
```

### User Story 2

```text
T043 User service tests
T044 Role service tests
T045 Permission service tests
T046 Access component tests
```

### User Story 3

```text
T061 Academic-year service tests
T062 Academic-period service tests
T063 Academic component tests
```

### User Story 4

```text
T074 Guardian service tests
T075 Student-profile service tests
T076 Lookup composable tests
T077 Guardian component tests
```

---

## Implementation Strategy

### MVP First

1. Complete Setup.
2. Complete Foundational.
3. Complete US1 Schools.
4. Run US1 tests and manual checkpoint.
5. Demo or release platform school list/create foundation.

### Incremental Delivery

1. Shared foundation.
2. US1 schools.
3. US2 users, roles, permissions.
4. US3 academic years and periods.
5. US4 guardians and student lookup.
6. Cross-cutting verification and roadmap update.

### Parallel Team Strategy

1. Team completes Setup and Foundational together.
2. After foundation:
   - Developer A: US1 Schools
   - Developer B: US2 Access administration
   - Developer C: US3 Academic calendar
   - Developer D: US4 Guardians
3. Team integrates and completes Phase 7.

## Notes

- Every task uses exact repository-relative paths.
- `[P]` means files do not conflict at task start.
- Story labels appear only in story phases.
- Tests must fail before corresponding implementation.
- No package installation, backend implementation, or OpenAPI change is
  approved.
