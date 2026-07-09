# Tasks: School List Filters

**Input**: Design documents from `specs/030-school-list-filters/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/school-list-filters-contract.md`, `quickstart.md`

**Tests**: Required. This feature changes REST query contracts, backend list behavior, frontend list controls, URL query state, tenant visibility, and critical school administration flows.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared foundations are complete.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish contract, backend, and frontend work locations before story work begins.

- [X] T001 Create OpenAPI school filter parameter directory in `schoolmaster-specs/api/components/parameters/schools/`
- [X] T002 [P] Create backend school filter DTO directory in `schoolmaster-backend/app/DTOs/School/`
- [X] T003 [P] Create backend school filter service directory in `schoolmaster-backend/app/Services/School/`
- [X] T004 [P] Create backend school list test directory in `schoolmaster-backend/tests/Feature/School/`
- [X] T005 [P] Create frontend school list module directories in `schoolmaster-frontend/src/modules/schools/`
- [X] T006 [P] Create frontend school list unit test directory in `schoolmaster-frontend/tests/unit/schools/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Define shared API parameter names, filter model boundaries, and module scaffolding required by all user stories.

**Critical**: No user story work should begin until this phase is complete.

- [X] T007 Update `listSchools` OpenAPI operation to include documented school list filter parameters and standard validation responses in `schoolmaster-specs/api/paths/schools/index.yaml`
- [X] T008 [P] Add status filter parameter with numeric `1`/`0` values in `schoolmaster-specs/api/components/parameters/schools/SchoolStatusFilter.yaml`
- [X] T009 [P] Add INEP code filter parameter with exact normalized digit semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolInepCodeFilter.yaml`
- [X] T010 [P] Add document filter parameter for CNPJ filtering with no `cnpj` alias in `schoolmaster-specs/api/components/parameters/schools/SchoolDocumentFilter.yaml`
- [X] T011 [P] Add name filter parameter with contains, case-insensitive, accent-insensitive semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolNameFilter.yaml`
- [X] T012 [P] Add email filter parameter with contains, case-insensitive, accent-insensitive semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolEmailFilter.yaml`
- [X] T013 [P] Add city filter parameter with contains, case-insensitive, accent-insensitive semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolCityFilter.yaml`
- [X] T014 [P] Add state filter parameter with contains, case-insensitive, accent-insensitive semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolStateFilter.yaml`
- [X] T015 [P] Add administrative type filter parameter with single-value exact lookup semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolAdministrativeTypeFilter.yaml`
- [X] T016 [P] Add legal nature filter parameter with single-value exact lookup semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolLegalNatureFilter.yaml`
- [X] T017 [P] Add management type filter parameter with single-value exact lookup semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolManagementTypeFilter.yaml`
- [X] T018 [P] Add pedagogical approach filter parameter with single-value exact lookup semantics in `schoolmaster-specs/api/components/parameters/schools/SchoolPedagogicalApproachFilter.yaml`
- [X] T019 Add school list filter DTO for normalized query input in `schoolmaster-backend/app/DTOs/School/SchoolListFilters.php`
- [X] T020 Add school list filter service shell for authorized Eloquent query composition in `schoolmaster-backend/app/Services/School/SchoolListFilterService.php`
- [X] T021 Update school list request validation shell for single-value filters, numeric status, lookup IDs, and normalized code fields in `schoolmaster-backend/app/Http/Requests/Api/V1/SchoolListRequest.php`
- [X] T022 Update school list controller to route validated filters through the filter service and existing resource pagination in `schoolmaster-backend/app/Http/Controllers/Api/V1/SchoolController.php`
- [ ] T023 [P] Add frontend school list filter types and query parameter names in `schoolmaster-frontend/src/modules/schools/types/schoolListFilters.ts`
- [ ] T024 [P] Add frontend school list service shell for documented query serialization in `schoolmaster-frontend/src/modules/schools/services/schoolListService.ts`
- [ ] T025 [P] Add frontend route-query filter composable shell in `schoolmaster-frontend/src/modules/schools/composables/useSchoolListFilters.ts`

**Checkpoint**: Foundation ready. User story implementation can now begin.

---

## Phase 3: User Story 1 - Find schools by identity and contact fields (Priority: P1) MVP

**Goal**: Authorized administrators can filter the Schools list by status, INEP code, CNPJ/document, name, email, city, and state, clear filters, and see only matching authorized schools.

**Independent Test**: Open the Schools list, apply each identity/contact/location filter individually and in combination, verify exact versus contains matching, and clear filters without losing authorization boundaries.

### Tests for User Story 1

- [X] T026 [P] [US1] Run Redocly validation for `GET /api/v1/schools` status, INEP, document, name, email, city, and state filter parameters in `schoolmaster-specs/api/openapi.yaml`
- [X] T027 [P] [US1] Add backend feature tests for status exact filtering, INEP exact filtering, document exact filtering, and no `cnpj` alias in `schoolmaster-backend/tests/Feature/School/SchoolListIdentityFiltersTest.php`
- [X] T028 [P] [US1] Add backend feature tests for name, email, city, and state contains matching with case and accent normalization in `schoolmaster-backend/tests/Feature/School/SchoolListTextFiltersTest.php`
- [X] T029 [P] [US1] Add backend feature tests for combined identity/contact/location filters using AND semantics and authorized visibility in `schoolmaster-backend/tests/Feature/School/SchoolListCombinedFiltersTest.php`
- [ ] T030 [P] [US1] Add frontend service tests for status, INEP, document, name, email, city, and state query serialization with no `cnpj` parameter in `schoolmaster-frontend/tests/unit/schools/schoolListService.filters.spec.ts`
- [ ] T031 [P] [US1] Add frontend list filter composable tests for applying, clearing, and restoring identity/contact/location filters from URL query state in `schoolmaster-frontend/tests/unit/schools/useSchoolListFilters.spec.ts`

### Implementation for User Story 1

- [X] T032 [US1] Implement backend normalization for status, INEP code, document/CNPJ digits, name, email, city, and state filters in `schoolmaster-backend/app/DTOs/School/SchoolListFilters.php`
- [X] T033 [US1] Implement backend exact matching for status, INEP code, and document filters in `schoolmaster-backend/app/Services/School/SchoolListFilterService.php`
- [X] T034 [US1] Implement backend case-insensitive and accent-insensitive contains matching for name, email, city, and state filters in `schoolmaster-backend/app/Services/School/SchoolListFilterService.php`
- [X] T035 [US1] Implement backend validation for unsupported status, malformed code filters, and rejected `cnpj` query input in `schoolmaster-backend/app/Http/Requests/Api/V1/SchoolListRequest.php`
- [X] T036 [US1] Wire list controller to preserve existing pagination, sorting, authorization, resource output, and validation envelopes while applying identity/contact/location filters in `schoolmaster-backend/app/Http/Controllers/Api/V1/SchoolController.php`
- [ ] T037 [US1] Implement frontend service serialization for status, INEP, document, name, email, city, and state filters in `schoolmaster-frontend/src/modules/schools/services/schoolListService.ts`
- [ ] T038 [US1] Implement frontend URL query synchronization for identity/contact/location filters, clear-one, clear-all, and page reset in `schoolmaster-frontend/src/modules/schools/composables/useSchoolListFilters.ts`
- [ ] T039 [US1] Implement frontend filter controls for status, INEP code, CNPJ label mapped to `document`, name, email, city, state, and clear actions in `schoolmaster-frontend/src/modules/schools/components/SchoolListFilters.vue`
- [ ] T040 [US1] Integrate identity/contact/location filters into the Schools list route while preserving loading, validation, empty, denied, and pagination states in `schoolmaster-frontend/src/modules/schools/routes/SchoolListPage.vue`

**Checkpoint**: User Story 1 is independently testable as the MVP.

---

## Phase 4: User Story 2 - Narrow schools by institutional classification (Priority: P2)

**Goal**: Authorized administrators can filter the Schools list by administrative type, legal nature, management type, and pedagogical approach using approved single-value lookup options.

**Independent Test**: Apply each institutional filter with known lookup values and verify only schools assigned to selected classification values appear, including invalid or unavailable lookup recovery.

### Tests for User Story 2

- [X] T041 [P] [US2] Run Redocly validation for administrative type, legal nature, management type, and pedagogical approach filter parameters in `schoolmaster-specs/api/openapi.yaml`
- [X] T042 [P] [US2] Add backend feature tests for institutional exact filtering, approved lookup validation, one-value-per-filter rejection, and AND semantics with identity filters in `schoolmaster-backend/tests/Feature/School/SchoolListInstitutionalFiltersTest.php`
- [ ] T043 [P] [US2] Add frontend service tests for institutional query serialization and single-value controls in `schoolmaster-frontend/tests/unit/schools/schoolListService.institutional.spec.ts`
- [ ] T044 [P] [US2] Add frontend component tests for lookup option loading, unavailable lookup state, invalid URL lookup feedback, and clearing institutional filters in `schoolmaster-frontend/tests/unit/schools/SchoolListInstitutionalFilters.spec.ts`

### Implementation for User Story 2

- [X] T045 [US2] Implement backend validation for administrative type, legal nature, management type, and pedagogical approach single-value lookup filters in `schoolmaster-backend/app/Http/Requests/Api/V1/SchoolListRequest.php`
- [X] T046 [US2] Implement backend exact institutional filtering against school reference fields in `schoolmaster-backend/app/Services/School/SchoolListFilterService.php`
- [X] T047 [US2] Ensure institutional filters never bypass existing school list authorization or tenant visibility in `schoolmaster-backend/app/Policies/SchoolPolicy.php`
- [ ] T048 [US2] Implement frontend institutional option loading for school list filters using approved lookup sources in `schoolmaster-frontend/src/modules/schools/services/schoolLookupService.ts`
- [ ] T049 [US2] Implement frontend single-value administrative type, legal nature, management type, and pedagogical approach filter controls in `schoolmaster-frontend/src/modules/schools/components/SchoolListFilters.vue`
- [ ] T050 [US2] Implement frontend unavailable lookup and invalid URL lookup feedback without applying hidden institutional filters in `schoolmaster-frontend/src/modules/schools/composables/useSchoolListFilters.ts`

**Checkpoint**: User Stories 1 and 2 both work independently.

---

## Phase 5: User Story 3 - Preserve list behavior while filters change (Priority: P3)

**Goal**: Pagination, sorting, tenant visibility, response shape, URL persistence, and list actions remain predictable while filters are applied, changed, shared, refreshed, or cleared.

**Independent Test**: Apply filters across paginated and sorted states, refresh or reopen filtered URLs, verify empty states and validation feedback, and confirm unauthorized schools stay hidden.

### Tests for User Story 3

- [X] T051 [P] [US3] Add backend feature tests for filtered pagination shape, filtered page metadata, sorted filtered results, no-result paginated responses, unauthorized visibility, and forbidden responses in `schoolmaster-backend/tests/Feature/School/SchoolListBehaviorPreservationTest.php`
- [ ] T052 [P] [US3] Add frontend route tests for page reset on filter changes, sort preservation, refresh initialization, shared URL initialization, empty state, and clear-all behavior in `schoolmaster-frontend/tests/unit/schools/SchoolListRouteState.spec.ts`
- [ ] T053 [P] [US3] Add frontend contract mapping tests proving list filters submit only documented query names and preserve existing paginated response handling in `schoolmaster-frontend/tests/unit/schools/schoolListContractMapping.spec.ts`

### Implementation for User Story 3

- [X] T054 [US3] Ensure backend list filtering composes after existing authorization, tenant visibility, and lifecycle visibility, then applies sorting, pagination, and page metadata to the filtered result set in `schoolmaster-backend/app/Services/School/SchoolListFilterService.php`
- [X] T055 [US3] Ensure backend list responses keep existing paginated success envelope, School resource shape, validation error envelope, unauthorized response, and forbidden response in `schoolmaster-backend/app/Http/Resources/SchoolResource.php`
- [ ] T056 [US3] Implement frontend page reset on filter changes and sort preservation in `schoolmaster-frontend/src/modules/schools/composables/useSchoolListFilters.ts`
- [ ] T057 [US3] Implement frontend initialization from bookmarked/shared filtered URLs and validation feedback for manipulated URL values in `schoolmaster-frontend/src/modules/schools/routes/SchoolListPage.vue`
- [ ] T058 [US3] Implement frontend empty result state that preserves active URL filters and provides clear filters action in `schoolmaster-frontend/src/modules/schools/components/SchoolListEmptyState.vue`
- [ ] T059 [US3] Update frontend list route actions to preserve existing row actions, pagination controls, loading state, denied state, and not-found behavior while filters are active in `schoolmaster-frontend/src/modules/schools/routes/SchoolListPage.vue`

**Checkpoint**: All user stories are independently functional and contract-aligned.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final verification, documentation, and release readiness across repositories.

- [X] T060 [P] Run OpenAPI lint and record output in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`
- [X] T061 [P] Run backend PHPUnit school list filter tests and record output in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`
- [ ] T062 [P] Run frontend Vitest school list tests and record output in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`
- [ ] T063 [P] Run frontend production build and record output in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`
- [X] T064 Review backend tenant isolation, validation errors, status handling, lookup handling, and cross-tenant existence leakage for list filters in `schoolmaster-backend/app/Services/School/SchoolListFilterService.php`
- [ ] T065 Review frontend responsive layout, keyboard flow, filter clearing, query-string behavior, and text fit for filter controls in `schoolmaster-frontend/src/modules/schools/components/SchoolListFilters.vue`
- [X] T066 Update implementation evidence, manual notes, and release gate status in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`
- [X] T067 Verify no backend, frontend, or OpenAPI path submits or documents a `cnpj` query parameter for school list filtering in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`
- [ ] T068 Run a representative 100-record Schools list filter timing check and record whether any single requested filter reduces to the intended result set in under 10 seconds in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`
- [ ] T069 Run a representative administrator filter-location check and record whether at least 90% can locate and clear active filters without assistance in `schoolmaster-specs/specs/030-school-list-filters/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- Phase 1 Setup has no dependencies.
- Phase 2 Foundational depends on Setup and blocks all user stories.
- Phase 3 User Story 1 depends on Foundational and is the MVP.
- Phase 4 User Story 2 depends on Foundational and can run after shared filter DTO/service/request contracts exist.
- Phase 5 User Story 3 depends on Foundational and validates best after US1 and US2 filter paths exist.
- Phase 6 Polish depends on desired user stories being complete.

### User Story Dependencies

- User Story 1 (P1): No dependency on other stories after Foundation.
- User Story 2 (P2): No dependency on US3; may reuse shared list filter controls from US1.
- User Story 3 (P3): Depends on active filter behavior from US1/US2 for full validation, but its tests can be authored after Foundation.

### Within Each User Story

- Write and run OpenAPI/PHPUnit/Vitest tests first; verify they fail before implementation.
- Complete request/DTO normalization before service query logic.
- Complete service query logic before controller/resource integration.
- Complete frontend service serialization before composables and route pages.
- Validate each story independently before starting lower-priority polish.

---

## Parallel Opportunities

- Setup tasks T002-T006 can run in parallel.
- Foundational parameter files T008-T018 can run in parallel after T001.
- Backend DTO/service/request/controller shell tasks T019-T022 should run sequentially.
- Frontend type/service/composable shell tasks T023-T025 can run in parallel.
- US1 tests T026-T031 can run in parallel.
- US2 tests T041-T044 can run in parallel.
- US3 tests T051-T053 can run in parallel.
- Polish verification tasks T060-T063 can run in parallel.

---

## Parallel Example: User Story 1

```bash
# Contract/backend/frontend tests can be authored together:
Task T026: Redocly validation for identity/contact/location filter parameters
Task T027: Backend exact identifier filter tests
Task T028: Backend normalized text filter tests
Task T030: Frontend service query serialization tests
Task T031: Frontend route-query composable tests

# Frontend implementation can proceed after query serialization types are stable:
Task T037: schoolListService.ts
Task T038: useSchoolListFilters.ts
Task T039: SchoolListFilters.vue
```

## Parallel Example: User Story 2

```bash
# Institutional tests can be authored together:
Task T041: Redocly validation for institutional filter parameters
Task T042: Backend institutional filtering tests
Task T043: Frontend institutional serialization tests
Task T044: Frontend institutional filter component tests

# Backend and frontend implementation can proceed after lookup names are stable:
Task T045: SchoolListRequest.php
Task T046: SchoolListFilterService.php
Task T048: schoolLookupService.ts
Task T049: SchoolListFilters.vue
```

## Parallel Example: User Story 3

```bash
# Behavior preservation checks can run together:
Task T051: Backend pagination/sorting/tenant visibility tests
Task T052: Frontend route state tests
Task T053: Frontend contract mapping tests

# UI behavior can be implemented while backend response shape remains unchanged:
Task T056: useSchoolListFilters.ts
Task T057: SchoolListPage.vue
Task T058: SchoolListEmptyState.vue
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 Setup.
2. Complete Phase 2 Foundational contract and module prerequisites.
3. Complete Phase 3 User Story 1.
4. Validate identity/contact/location filters through OpenAPI, PHPUnit, Vitest, and manual list checks.
5. Stop for review before institutional filters.

### Incremental Delivery

1. Foundation: OpenAPI parameters, backend request/service boundaries, frontend service/composable shells.
2. US1: identity, contact, and location filters.
3. US2: institutional classification filters.
4. US3: pagination, sorting, URL persistence, empty state, and tenant behavior preservation.
5. Polish: full regression, quickstart evidence, security and accessibility review.

### Team Parallel Strategy

1. One engineer owns OpenAPI parameters and specs updates.
2. One backend engineer owns request validation, filter service, authorization safety, and PHPUnit.
3. One frontend engineer owns services, composables, filter controls, routes, and Vitest.
4. Contract owner reconciles query parameter names and behavior before merge.

## Notes

- `[P]` tasks use separate files or can be done after shared interfaces are stable.
- `[US1]`, `[US2]`, and `[US3]` labels map directly to spec user stories.
- Tests are required because this feature changes critical REST, backend, and frontend list behavior.
- Do not add a `cnpj` query alias; CNPJ UI label maps to `document`.
- Do not add runtime management of institutional lookup values.
