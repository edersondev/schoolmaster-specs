# Tasks: Frontend Architecture Baseline

**Input**: Design documents from `specs/015-frontend-architecture-baseline/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: This feature is a specification and documentation baseline only. No backend, frontend runtime, or OpenAPI implementation tests are required in this slice. Validation tasks below use documentation review, contract consistency checks, and quickstart checks.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- Run tasks from the `schoolmaster-specs` repository root.
- Feature artifacts live under `specs/015-frontend-architecture-baseline/`.
- Shared architecture guidance lives under `docs/`.
- This feature does not modify `schoolmaster-backend`, `schoolmaster-frontend`, or `api/openapi.yaml`.

---

## Phase 1: Setup (Shared Documentation Baseline)

**Purpose**: Confirm the feature artifact set and source documentation before story work starts.

- [ ] T001 Confirm the active feature inputs exist in `specs/015-frontend-architecture-baseline/spec.md`, `specs/015-frontend-architecture-baseline/plan.md`, `specs/015-frontend-architecture-baseline/research.md`, `specs/015-frontend-architecture-baseline/data-model.md`, `specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md`, and `specs/015-frontend-architecture-baseline/quickstart.md`
- [ ] T002 [P] Inventory existing frontend architecture guidance in `docs/frontend-architecture.md`, `docs/frontend-guidelines.md`, `docs/naming-conventions.md`, and `docs/frontend-admin-system-architecture.md`
- [ ] T003 [P] Review frontend roadmap sequencing for item 1 in `docs/frontend-feature-roadmap.md`
- [ ] T004 Verify the feature remains documentation-only with no required changes to `api/openapi.yaml`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish shared documentation decisions that all user stories depend on.

**CRITICAL**: No user story work should begin until this phase is complete.

- [ ] T005 [P] Align the approved package baseline in `docs/frontend-architecture.md` with JavaScript, Vue 3, Vue Router, Pinia, Axios, Element Plus, `@element-plus/icons-vue`, Vue I18n, and Tailwind CSS
- [ ] T006 [P] Align the approved package baseline in `docs/frontend-admin-system-architecture.md` with JavaScript, Element Plus Icons, Vue I18n, and Tailwind CSS
- [ ] T007 [P] Document JavaScript file naming and JSDoc contract-shape conventions in `docs/naming-conventions.md`
- [ ] T008 [P] Capture research decisions for JavaScript, Vue Router, Pinia, Axios, Element Plus, Element Plus Icons, Vue I18n, Tailwind CSS, JSDoc contracts, accessibility, and observability in `specs/015-frontend-architecture-baseline/research.md`
- [ ] T009 [P] Update architecture concept entities and relationships in `specs/015-frontend-architecture-baseline/data-model.md`
- [ ] T010 Update repository sequencing and active plan context in `AGENTS.md`

**Checkpoint**: Shared stack, repository scope, and architecture concept decisions are ready.

---

## Phase 3: User Story 1 - Establish Frontend Architecture Baseline (Priority: P1) MVP

**Goal**: A frontend implementer can identify the approved SPA stack, application structure, folder responsibilities, and service/store/component boundaries before frontend implementation starts.

**Independent Test**: Review the architecture documents and confirm they define the approved frontend stack, folder responsibilities, JavaScript conventions, Element Plus usage rules, Tailwind responsibilities, and cross-repository sequencing without approving concrete business module behavior.

### Validation for User Story 1

- [ ] T011 [P] [US1] Validate approved stack coverage against FR-001, FR-002, FR-011, FR-012, FR-013, FR-014, FR-022, FR-024, and FR-025 in `specs/015-frontend-architecture-baseline/spec.md`
- [ ] T012 [P] [US1] Validate no TypeScript, backend implementation, concrete business workflow, or undocumented API behavior is approved in `specs/015-frontend-architecture-baseline/spec.md`

### Implementation for User Story 1

- [ ] T013 [US1] Define the durable SPA architecture baseline in `docs/frontend-architecture.md`
- [ ] T014 [US1] Document frontend working principles, state/module conventions, UI composition standards, layout baseline, and error/loading patterns in `docs/frontend-guidelines.md`
- [ ] T015 [US1] Document frontend naming conventions for Vue components, Element Plus PascalCase tags, composables, Pinia stores, router modules, services, and JSDoc contract shapes in `docs/naming-conventions.md`
- [ ] T016 [US1] Define the language, file, folder responsibility, UI primitive, icon, internationalization, accessibility, state, and observability contract in `specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md`
- [ ] T017 [US1] Update the quickstart stack, folder boundary, UI convention, JSDoc contract, and review-command checklist in `specs/015-frontend-architecture-baseline/quickstart.md`
- [ ] T018 [US1] Cross-check US1 success criteria SC-001, SC-002, SC-005, SC-008, SC-010, SC-012, and SC-013 against `docs/frontend-architecture.md`, `docs/frontend-guidelines.md`, `docs/naming-conventions.md`, and `specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md`

**Checkpoint**: User Story 1 is independently reviewable as the MVP baseline.

---

## Phase 4: User Story 2 - Standardize Reusable Admin Workflow Patterns (Priority: P2)

**Goal**: A frontend implementer can reuse shared CRUD-oriented admin patterns for list pages, filters, tables, forms, dialogs, pagination, loading states, empty states, validation states, and error states.

**Independent Test**: Review the baseline and verify reusable admin workflow patterns are documented without defining module-specific behavior for schools, users, guardians, classes, reports, or other business modules.

### Validation for User Story 2

- [ ] T019 [P] [US2] Validate reusable CRUD requirements FR-005, FR-008, FR-019, FR-021, and SC-006 in `specs/015-frontend-architecture-baseline/spec.md`
- [ ] T020 [P] [US2] Validate reusable CRUD coverage in `specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md`

### Implementation for User Story 2

- [ ] T021 [US2] Document reusable admin layout, dashboard, component, CRUD, store, router, service, and contract blueprint boundaries in `docs/frontend-admin-system-architecture.md`
- [ ] T022 [US2] Document reusable CRUD-heavy admin workflow expectations in `docs/frontend-guidelines.md`
- [ ] T023 [US2] Ensure `ReusableCrudPattern` covers list, filter, table, form, dialog, pagination, loading, empty, error, and validation behavior in `specs/015-frontend-architecture-baseline/data-model.md`
- [ ] T024 [US2] Ensure quickstart CRUD reuse checks distinguish reusable patterns from resource-specific business rules in `specs/015-frontend-architecture-baseline/quickstart.md`
- [ ] T025 [US2] Cross-check that `docs/frontend-admin-system-architecture.md` does not approve concrete schools, users, guardians, classes, reports, authentication, or dashboard data behavior beyond reusable blueprint boundaries

**Checkpoint**: User Story 2 is independently reviewable as reusable CRUD guidance.

---

## Phase 5: User Story 3 - Preserve API-First Frontend Delivery (Priority: P3)

**Goal**: A frontend implementer can verify that future screens consume only approved OpenAPI-backed `/api/v1` contract semantics through services and do not depend on undocumented backend behavior.

**Independent Test**: Review the baseline and verify it requires documented `/api/v1` behavior, service-isolated HTTP access, blocked undocumented fields/routes/status meanings, and clear repository sequencing.

### Validation for User Story 3

- [ ] T026 [P] [US3] Validate API-first requirements FR-006, FR-009, FR-010, FR-015, FR-016, FR-017, and FR-020 in `specs/015-frontend-architecture-baseline/spec.md`
- [ ] T027 [P] [US3] Validate API consumption rules, state rules, and blocked behavior in `specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md`

### Implementation for User Story 3

- [ ] T028 [US3] Document service isolation, API access responsibilities, and prohibited direct Axios usage in `docs/frontend-architecture.md`
- [ ] T029 [US3] Document contract alignment, tenant context sourcing, backend authorization authority, and service/store boundaries in `docs/frontend-guidelines.md`
- [ ] T030 [US3] Ensure `ServiceBoundary`, `StateBoundary`, and `FrontendContractDefinition` explicitly block undocumented API fields, status meanings, tenant assumptions, and TypeScript-only contracts in `specs/015-frontend-architecture-baseline/data-model.md`
- [ ] T031 [US3] Ensure quickstart API-first readiness checks require documented `/api/v1` endpoints, fields, filters, pagination, status meanings, error envelopes, auth semantics, and tenant semantics in `specs/015-frontend-architecture-baseline/quickstart.md`
- [ ] T032 [US3] Cross-check that no task or document in `specs/015-frontend-architecture-baseline/` requires backend code, OpenAPI changes, or undocumented frontend consumption

**Checkpoint**: User Story 3 is independently reviewable as API-first frontend delivery guidance.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final consistency and readiness checks across all stories.

- [ ] T033 [P] Run unresolved template-marker scan in `specs/015-frontend-architecture-baseline/` and `docs/`
- [ ] T034 [P] Run markdown whitespace validation with `git diff --check` in `.`
- [ ] T035 [P] Verify roadmap item 1 status and sequencing in `docs/frontend-feature-roadmap.md`
- [ ] T036 Verify all task checklist lines in `specs/015-frontend-architecture-baseline/tasks.md` follow `- [ ] T### [P?] [US?] Description with file path`
- [ ] T037 Review `specs/015-frontend-architecture-baseline/spec.md`, `specs/015-frontend-architecture-baseline/plan.md`, `specs/015-frontend-architecture-baseline/research.md`, `specs/015-frontend-architecture-baseline/data-model.md`, `specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md`, and `specs/015-frontend-architecture-baseline/quickstart.md` for consistency before handoff

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; can start immediately.
- **Foundational (Phase 2)**: Depends on Setup completion; blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational completion; MVP scope.
- **User Story 2 (Phase 4)**: Depends on Foundational completion; can run after or alongside US1 once shared terminology is stable.
- **User Story 3 (Phase 5)**: Depends on Foundational completion; can run after or alongside US1 once contract terminology is stable.
- **Polish (Phase 6)**: Depends on all desired user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational; no dependency on US2 or US3.
- **User Story 2 (P2)**: Can start after Foundational; should reuse baseline terms from US1 when available.
- **User Story 3 (P3)**: Can start after Foundational; should reuse baseline contract terms from US1 when available.

### Parallel Opportunities

- T002 and T003 can run in parallel during Setup.
- T005 through T009 can run in parallel during Foundational because they target different files or artifact areas.
- US1 validation tasks T011 and T012 can run in parallel.
- US2 validation tasks T019 and T020 can run in parallel.
- US3 validation tasks T026 and T027 can run in parallel.
- Polish checks T033 through T035 can run in parallel.

---

## Parallel Example: User Story 1

```bash
Task: "Validate approved stack coverage against FR-001, FR-002, FR-011, FR-012, FR-013, FR-014, FR-022, FR-024, and FR-025 in specs/015-frontend-architecture-baseline/spec.md"
Task: "Validate no TypeScript, backend implementation, concrete business workflow, or undocumented API behavior is approved in specs/015-frontend-architecture-baseline/spec.md"
```

## Parallel Example: User Story 2

```bash
Task: "Validate reusable CRUD requirements FR-005, FR-008, FR-019, FR-021, and SC-006 in specs/015-frontend-architecture-baseline/spec.md"
Task: "Validate reusable CRUD coverage in specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md"
```

## Parallel Example: User Story 3

```bash
Task: "Validate API-first requirements FR-006, FR-009, FR-010, FR-015, FR-016, FR-017, and FR-020 in specs/015-frontend-architecture-baseline/spec.md"
Task: "Validate API consumption rules, state rules, and blocked behavior in specs/015-frontend-architecture-baseline/contracts/frontend-architecture-contract.md"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. Stop and validate the architecture baseline independently against US1 acceptance scenarios.

### Incremental Delivery

1. Complete Setup and Foundational documentation alignment.
2. Deliver US1 as the minimum approved frontend architecture baseline.
3. Add US2 reusable admin CRUD guidance.
4. Add US3 API-first frontend delivery guardrails.
5. Complete Polish checks before moving to the next frontend roadmap item.

### Parallel Team Strategy

With multiple contributors:

1. One contributor aligns package, naming, and data-model decisions.
2. One contributor drafts the architecture and guideline updates for US1.
3. One contributor reviews CRUD and API-first contracts for US2 and US3 after Foundational completion.

---

## Notes

- [P] tasks target different files or independent review checks.
- This feature does not approve runtime frontend code, backend code, OpenAPI changes, or business module behavior.
- Future `schoolmaster-frontend` implementation tasks should be generated from later feature specs that consume this baseline.
