# Tasks: Academic Year List Filters

**Input**: Design documents from `specs/035-academic-year-filters/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md`

**Tests**: Contract, PHPUnit, and Vitest coverage are required by FR-012 and the
project constitution. Test tasks precede matching implementation tasks.

**Organization**: Tasks are grouped by user story. User Story 1 delivers each
filter independently; User Story 2 adds combined criteria, URL persistence,
pagination retention, and reset.

## Phase 1: Setup (Contract First)

**Purpose**: Publish additive list parameters before runtime work.

- [X] T001 [P] Add academic-year name and date-range parameter components in `api/components/parameters/academic-years/AcademicYearNameFilter.yaml`, `api/components/parameters/academic-years/AcademicYearDateFromFilter.yaml`, and `api/components/parameters/academic-years/AcademicYearDateToFilter.yaml`
- [X] T002 [P] Add full lifecycle status parameter component in `api/components/parameters/academic-years/AcademicYearStatusFilter.yaml`
- [X] T003 Update `listAcademicYears` parameter references and validation response in `api/paths/academic-years/index.yaml`
- [X] T004 Promote the approved parameters and semantics into `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T005 Validate both published contracts with Redocly using `redocly.yaml`

**Checkpoint**: OpenAPI contract is valid and implementation may begin.

---

## Phase 2: Foundational (Shared Query Contract)

**Purpose**: Establish validated backend input and frontend route-query names
used by both stories.

- [X] T006 Add failing PHPUnit validation coverage for documented/unknown query fields, full status domain, paired date boundaries, malformed dates, and reversed ranges in `schoolmaster-backend/tests/Feature/Api/V1/AcademicYearManagementTest.php`
- [X] T007 Add failing Vitest parse/serialize/update coverage for `name`, `date_from`, `date_to`, and academic-year statuses in `schoolmaster-frontend/tests/unit/admin-system/administration/composables/useAdminListQuery.academic-year-filters.spec.js`
- [X] T008 Implement academic-year list query normalization and validation in `schoolmaster-backend/app/Http/Requests/Api/V1/AcademicYearListRequest.php` and wire it through `schoolmaster-backend/app/Http/Controllers/Api/V1/AcademicYearController.php`
- [X] T009 Implement academic-year route-query parsing and serialization in `schoolmaster-frontend/src/composables/admin-system/useAdminListQuery.js`

**Checkpoint**: Both repositories accept the same documented filter shape.

---

## Phase 3: User Story 1 — Find an academic year (Priority: P1) 🎯 MVP

**Goal**: Filter the resolved school's academic years independently by partial
name, inclusive overlapping date range, or any approved lifecycle status.

**Independent Test**: Apply each criterion alone against distinct same-school
records and confirm only matching records appear; matching cross-school records
must never appear.

### Tests for User Story 1

- [X] T010 [P] [US1] Add failing PHPUnit matching, inclusive-boundary, empty-result, pagination, and tenant-isolation cases in `schoolmaster-backend/tests/Feature/Api/V1/AcademicYearManagementTest.php`
- [X] T011 [P] [US1] Add failing service serialization coverage for documented academic-year filter parameter names in `schoolmaster-frontend/tests/unit/admin-system/administration/services/academic-years.spec.js`
- [X] T012 [P] [US1] Add failing component coverage for dedicated fields, four statuses, explicit submit, and absence of school-only fields in `schoolmaster-frontend/tests/unit/admin-system/administration/components/AcademicYearFilters.spec.js`

### Implementation for User Story 1

- [X] T013 [US1] Apply tenant-scoped partial-name, date-overlap, and exact-status predicates while preserving order and pagination in `schoolmaster-backend/app/Services/AcademicYears/AcademicYearService.php`
- [X] T014 [US1] Replace school-filter reuse with a focused submitted form and paired date picker in `schoolmaster-frontend/src/components/admin-system/academic-years/AcademicYearFilters.vue`
- [X] T015 [P] [US1] Add academic-year filter and lifecycle status labels in `schoolmaster-frontend/src/locales/administration.js`
- [X] T016 [US1] Wire complete filter values into `schoolmaster-frontend/src/pages/admin-system/academic-years/AcademicYearsListPage.vue` and verify `schoolmaster-frontend/src/services/admin-system/academic-years.js` forwards the documented query unchanged

**Checkpoint**: Name, date-range, and status filters work independently end to
end without exposing school-specific fields.

---

## Phase 4: User Story 2 — Combine and reset filters (Priority: P2)

**Goal**: Combine criteria with AND semantics, preserve them across reload and
pagination, and reset them together.

**Independent Test**: Submit all criteria, reload and paginate the filtered URL,
change page size, then reset; verify criteria and results at every step.

### Tests for User Story 2

- [X] T017 [P] [US2] Add failing PHPUnit combined-filter AND-semantics coverage in `schoolmaster-backend/tests/Feature/Api/V1/AcademicYearManagementTest.php`
- [X] T018 [P] [US2] Add failing page integration coverage for combined submit, first-page reset, reload state, pagination retention, and reset in `schoolmaster-frontend/tests/unit/admin-system/administration/pages/AcademicYearsListPage.filters.spec.js`

### Implementation for User Story 2

- [X] T019 [US2] Complete combined submission, query persistence, pagination retention, and reset behavior across `schoolmaster-frontend/src/pages/admin-system/academic-years/AcademicYearsListPage.vue` and `schoolmaster-frontend/src/composables/admin-system/useAdminListQuery.js`

**Checkpoint**: Both user stories pass independently and together.

---

## Phase 5: Polish & Cross-Cutting Verification

**Purpose**: Format changed code and run all release gates.

- [X] T020 [P] Run focused PHPUnit tests and Pint for `schoolmaster-backend/tests/Feature/Api/V1/AcademicYearManagementTest.php` and changed PHP files
- [X] T021 [P] Run focused Vitest suites and production build in `schoolmaster-frontend`
- [X] T022 Re-run Redocly contract validation and record automated/manual evidence in `specs/035-academic-year-filters/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- Phase 1 has no dependency and must complete first.
- Phase 2 depends on validated OpenAPI and blocks both user stories.
- User Story 1 depends on Phase 2 and provides the MVP.
- User Story 2 depends on Phase 2 and integrates with User Story 1's filter
  controls while remaining independently verifiable through query behavior.
- Phase 5 depends on both selected user stories.

### Parallel Opportunities

- T001 and T002 touch different parameter files.
- T010, T011, and T012 cover different repositories/files.
- T015 can proceed beside backend T013 after component labels are identified.
- T017 and T018 cover different repositories.
- T020 and T021 validate separate repositories.

## Parallel Example: User Story 1

```text
Task T010: PHPUnit academic-year matching and tenant cases
Task T011: Vitest service parameter serialization
Task T012: Vitest dedicated filter component behavior
```

## Implementation Strategy

1. Publish and lint OpenAPI.
2. Establish shared validated query names with failing tests.
3. Deliver User Story 1 as end-to-end MVP.
4. Add User Story 2 persistence and reset behavior.
5. Run contract, backend, frontend, formatting, and build gates.

## Format Validation

All 22 tasks use checkbox, sequential ID, optional `[P]`, required story labels
inside story phases, and concrete file paths.
