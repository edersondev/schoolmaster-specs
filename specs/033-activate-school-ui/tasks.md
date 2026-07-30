# Tasks: School Context Selection UI

**Input**: Design documents from `specs/033-activate-school-ui/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/frontend-school-context-selection-contract.md`, `quickstart.md`

**Tests**: Required by FR-024. Write each story's PHPUnit, Vitest, and
Playwright coverage before its implementation and confirm new assertions fail
for the expected reason.

**Repository Scope**: `schoolmaster-specs` owns OpenAPI and feature evidence;
`schoolmaster-backend` owns current-session conformance; `schoolmaster-frontend`
owns selector, context, routing, lifecycle integration, and browser behavior.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because task changes different files and has no
  dependency on another incomplete task in same phase.
- **[Story]**: User story label appears only in story phases.
- Paths begin with owning repository name and are relative to parent workspace.

## Phase 1: Setup and Contract Baseline

**Purpose**: Establish contract-first change and reusable test data before
cross-repository implementation.

- [X] T001 Require existing nullable `resolved_school` property in `schoolmaster-specs/api/components/schemas/auth/AuthSession.yaml`
- [X] T002 [P] Create active, inactive, duplicate-name, 100-school, numeric-status, and stale-generation fixtures in `schoolmaster-frontend/tests/unit/school-context-selection/fixtures/schoolContextSelection.fixtures.js`

---

## Phase 2: Foundational Contract Conformance

**Purpose**: Make published current-session confirmation real and normalize
School data before any user story consumes it.

**Critical**: No user-story implementation begins before this phase completes.

### Tests for Foundation

- [X] T003 [P] Add failing PHPUnit cases for System Administrator with no header, exact active `X-School-Id`, inactive/unknown header, and existing school-bound mismatch in `schoolmaster-backend/tests/Feature/CurrentUserApiTest.php`
- [X] T004 [P] Add failing numeric/string status and name/INEP/city/state mapping tests in `schoolmaster-frontend/tests/unit/school-context-selection/contracts/schoolSelection.contract.spec.js`
- [X] T005 [P] Add failing `/auth/me` mapping tests using contract-accurate numeric School status and nullable `resolved_school` in `schoolmaster-frontend/tests/unit/auth/authService.currentUser.test.js`

### Implementation for Foundation

- [X] T006 [P] Preserve and return resolved `TenantContext` from current-session service logic in `schoolmaster-backend/app/Services/AuthService.php`
- [X] T007 [P] Accept explicit resolved School context while preserving login serialization in `schoolmaster-backend/app/Http/Resources/AuthSessionResource.php`
- [X] T008 Pass resolved tenant school from service result into auth-session serialization in `schoolmaster-backend/app/Http/Controllers/Api/V1/AuthController.php`
- [X] T009 [P] Normalize School status and expose `inepCode`, city, and state in `schoolmaster-frontend/src/contracts/auth/authSession.contract.js`
- [X] T010 [P] Normalize selector-facing School status and camel-case INEP/address fields in `schoolmaster-frontend/src/contracts/admin-system/schools.js`

**Checkpoint**: `/auth/me` confirms exact resolved school for platform System
Administrator, and frontend mappings match published numeric School schema.

---

## Phase 3: User Story 1 - Select an Active School (Priority: P1) MVP

**Goal**: System Administrator can search paginated active schools, distinguish
them without CNPJ, explicitly select one, and enter school-owned work only after
exact backend confirmation.

**Independent Test**: Sign in as System Administrator without context, request
a school-owned route, search a 100-school dataset by name and INEP, select one
active school, and verify exact confirmed school appears before requested route
loads; repeat with non-System Administrator and failed confirmation.

### Tests for User Story 1

- [X] T011 [P] [US1] Add failing active-only query, separate name/INEP, explicit `per_page=25`, pagination, and safe-error service tests in `schoolmaster-frontend/tests/unit/school-context-selection/services/schoolSelectionService.spec.js`
- [X] T012 [P] [US1] Add failing search cancellation, stale-result, refresh, pagination, loading, empty, and retry tests in `schoolmaster-frontend/tests/unit/school-context-selection/composables/useSchoolSelection.spec.js`
- [X] T013 [P] [US1] Add failing labeled filter, keyboard submission, and clear-filter component tests in `schoolmaster-frontend/tests/unit/school-context-selection/components/SchoolSelectionSearch.spec.js`
- [X] T014 [P] [US1] Add failing duplicate-name identity, active-only action, no-auto-select, no-CNPJ, focus, and accessible-name tests in `schoolmaster-frontend/tests/unit/school-context-selection/components/SchoolSelectionList.spec.js`
- [X] T015 [P] [US1] Add failing loading, filtered no-results, unfiltered no-active-schools, validation, unauthorized, forbidden, inactive, tenant-mismatch, expired-session, conflict, temporary-unavailable, pagination, duplicate-submit, and exact-role-gate tests in `schoolmaster-frontend/tests/unit/school-context-selection/components/SchoolContextSelector.spec.js`
- [X] T016 [P] [US1] Add failing dedicated-page selection and post-confirmation navigation tests in `schoolmaster-frontend/tests/unit/school-context-selection/pages/SchoolSelectionPage.spec.js`
- [X] T017 [P] [US1] Extend missing-context capture, registered and existing release/feature-gate-enabled requested-route resume, authorization recheck, and dashboard fallback tests in `schoolmaster-frontend/tests/unit/auth/authGuards.requestedRoute.test.js`
- [X] T018 [P] [US1] Add failing exact confirmation, numeric active status, duplicate submission, mismatch, inactive, recoverable error, and 401 identity-loss tests in `schoolmaster-frontend/tests/unit/system-admin-master/schoolContext/systemAdminSchoolContext.spec.js`
- [X] T019 [P] [US1] Add failing exact current-school presentation and read-only US1/non-System Administrator behavior tests with no switch event in `schoolmaster-frontend/tests/unit/admin-system/AdminShellHeader.spec.js`
- [X] T020 [P] [US1] Add failing 100-school search, pagination, duplicate-name, exact-selection, no-CNPJ, and initial responsive keyboard flow in `schoolmaster-frontend/e2e/school-context-selection.spec.js`

### Implementation for User Story 1

- [X] T021 [US1] Create constrained System Administrator active-school discovery wrapper over existing school service in `schoolmaster-frontend/src/services/auth/schoolSelectionService.js`
- [X] T022 [US1] Implement transient query, pagination, AbortController, request-generation, empty, error, retry, and refresh state in `schoolmaster-frontend/src/composables/auth/useSchoolSelection.js`
- [X] T023 [P] [US1] Implement labeled name/INEP controls with explicit search and clear events in `schoolmaster-frontend/src/components/auth/SchoolSelectionSearch.vue`
- [X] T024 [P] [US1] Replace CNPJ cards with active School choices showing name, INEP, city, and state in `schoolmaster-frontend/src/components/auth/SchoolSelectionList.vue`
- [X] T025 [US1] Compose search, result list, pagination, loading, empty, feedback, and explicit selection in `schoolmaster-frontend/src/components/auth/SchoolContextSelector.vue`
- [X] T026 [US1] Keep route page thin while enforcing exact actor gate and safe success navigation in `schoolmaster-frontend/src/pages/auth/SchoolSelectionPage.vue` and `schoolmaster-frontend/src/router/modules/auth.routes.js`
- [X] T027 [P] [US1] Remove blocked generic selection placeholder while retaining auth-only current-session transport in `schoolmaster-frontend/src/services/auth/authService.js`
- [X] T028 [US1] Commit school only after latest exact active confirmation, preserve identity on recoverable selection failure, and block duplicate selection in `schoolmaster-frontend/src/stores/auth/sessionStore.js`
- [X] T029 [US1] Capture missing-context intent and resume only a still-registered route enabled by the existing release/feature-gate mechanism and authorized after confirmation, otherwise use dashboard, in `schoolmaster-frontend/src/router/authGuards.js`
- [X] T030 [P] [US1] Create presentational persistent read-only current-school indicator without a switch event in `schoolmaster-frontend/src/components/admin-system/shell/SchoolContextControl.vue`
- [X] T031 [US1] Integrate read-only current-school presentation into authenticated shell without changing platform scope or exposing shell switching in `schoolmaster-frontend/src/components/admin-system/shell/AdminShellHeader.vue` and `schoolmaster-frontend/src/layouts/admin-system/AdminSystemLayout.vue`
- [X] T032 [P] [US1] Add distinct loading, filtered no-results, unfiltered no-active-schools, validation, unauthorized, forbidden, inactive, tenant-mismatch, expired-session, conflict, temporary-unavailable, pagination, identity, progress, and read-only shell strings in `schoolmaster-frontend/src/locales/auth.js` and `schoolmaster-frontend/src/locales/admin-system.js`

**Checkpoint**: User Story 1 is independently functional as MVP; no
school-owned content appears before exact active confirmation, and persistent
shell context remains read-only until User Story 2 protections ship.

---

## Phase 4: User Story 2 - Restore or Switch School Context (Priority: P2)

**Goal**: Same valid identity can restore or deliberately switch school without
stale data, unsafe route identifiers, preference leakage, or polling.

**Independent Test**: Confirm School A, load tenant data, switch to School B,
and verify A data clears before B loads. Repeat across reload, safe list route,
unsafe detail route, platform route, cancelled dirty form, stale responses, and
authoritative invalidation.

### Tests for User Story 2

- [X] T033 [P] [US2] Extend bootstrap tests for valid restore, invalid persisted-school cleanup, one headerless retry, two-call limit, and no retry for forbidden/token/5xx in `schoolmaster-frontend/tests/unit/auth/sessionStore.bootstrap.test.js`
- [X] T034 [P] [US2] Add identity-bound preference clearing tests for logout, expiry, token rejection, lifecycle cleanup, and identity replacement in `schoolmaster-frontend/tests/unit/auth/sessionRecovery.test.js`
- [X] T035 [P] [US2] Add idempotent invalidation tests preserving identity while clearing context/data/preference and incrementing generation once in `schoolmaster-frontend/tests/unit/school-context-selection/stores/sessionStore.invalidation.spec.js`
- [X] T036 [P] [US2] Add `normalizeSchoolContextErrorCode` tests for canonical nested, documented legacy top-level, unsupported, lowercase, and uppercase inputs plus classifier header/generation matching, passthrough, exclusion, and duplicate responses in `schoolmaster-frontend/tests/unit/school-context-selection/services/schoolContextInvalidation.spec.js`
- [X] T037 [P] [US2] Extend reset ordering and stale School A response rejection tests in `schoolmaster-frontend/tests/unit/system-admin-master/schoolContext/schoolContextSwitchClearsData.spec.js`
- [X] T038 [P] [US2] Add safe platform/list retention and unsafe detail/edit/action/subject fallback tests in `schoolmaster-frontend/tests/unit/school-context-selection/router/schoolContextRoutes.spec.js`
- [X] T039 [P] [US2] Extend requested-intent capture, one-time consumption, registered-route and existing release/feature-gate mechanism recheck, permission recheck, and invalidation redirect tests in `schoolmaster-frontend/tests/unit/auth/authGuards.requestedRoute.test.js`
- [X] T040 [P] [US2] Extend dirty-form and open-lifecycle-dialog cancel/approve ordering tests in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/useUnsavedChangesGuard.spec.js`
- [X] T041 [P] [US2] Extend platform-route no-filter/unresolved-indicator tests and prove shell switch navigation is enabled only with guarded US2 orchestration in `schoolmaster-frontend/tests/unit/system-admin-master/schoolContext/platformWidePresentationScope.spec.js` and `schoolmaster-frontend/tests/unit/admin-system/AdminShellHeader.spec.js`
- [X] T042 [P] [US2] Extend browser coverage for restoration, safe/unsafe route switching, dirty cancellation, stale generation, matching authoritative invalidation, and platform retention in `schoolmaster-frontend/e2e/school-context-selection.spec.js`

### Implementation for User Story 2

- [X] T043 [US2] Implement identity-bound persistence cleanup, bounded bootstrap fallback, idempotent `invalidateSchoolContext`, and authenticated unresolved-context state in `schoolmaster-frontend/src/stores/auth/sessionStore.js`
- [X] T044 [US2] Centralize resetter execution, generation changes, selection pending state, and stale completion rejection in `schoolmaster-frontend/src/composables/auth/useSchoolContextSwitch.js`
- [X] T045 [P] [US2] Implement pure `normalizeSchoolContextErrorCode` for only canonical nested/documented legacy top-level inputs, authoritative normalized-code classification, request context stamping, response matching, passthrough, and observer teardown in `schoolmaster-frontend/src/services/api/schoolContextInvalidation.js`
- [X] T046 [US2] Install one invalidation observer with explicit Pinia/router dependencies at application composition root in `schoolmaster-frontend/src/main.js`
- [X] T047 [P] [US2] Implement default-deny named-route compatibility and context-neutral query handling in `schoolmaster-frontend/src/router/schoolContextRoutes.js`
- [X] T048 [US2] Apply route compatibility, requested-intent consumption, school-owned redirect, and platform-route retention in `schoolmaster-frontend/src/router/authGuards.js`
- [X] T049 [US2] Mark only safe generic workspace/list routes retainable in `schoolmaster-frontend/src/router/modules/administration-route.js`, `schoolmaster-frontend/src/modules/teacher-workflow/routes/index.js`, `schoolmaster-frontend/src/router/modules/reporting.js`, and `schoolmaster-frontend/src/router/modules/assessments.js`
- [X] T050 [P] [US2] Enable the shell choose/switch event only through guarded selector navigation and connect lifecycle-dialog open state to route-leave protection in `schoolmaster-frontend/src/components/admin-system/shell/SchoolContextControl.vue`, `schoolmaster-frontend/src/components/admin-system/shell/AdminShellHeader.vue`, `schoolmaster-frontend/src/composables/admin-system/useAdminLifecycleAction.js`, and `schoolmaster-frontend/src/composables/admin-system/useUnsavedChangesGuard.js`
- [X] T051 [P] [US2] Add safe context-invalidated, restoration-failed, retry, and choose-school feedback strings in `schoolmaster-frontend/src/locales/auth.js` and `schoolmaster-frontend/src/locales/admin-system.js`

**Checkpoint**: User Story 2 restores and switches context safely; generic
401/403, platform calls, stale headers/generations, and transient errors never
invalidate a valid current school.

---

## Phase 5: User Story 3 - Activate Then Select a School (Priority: P3)

**Goal**: Existing lifecycle UI activates an inactive school, selector exposes
it only after deliberate refresh, and current-school deactivation/deletion
invalidates context without auto-selection.

**Independent Test**: Activate an inactive school through existing lifecycle
confirmation, refresh selector, select it, then deactivate/delete current
school and verify context/data clear. Repeat failed/conflicting lifecycle and
non-current target cases.

### Tests for User Story 3

- [X] T052 [P] [US3] Extend list-page lifecycle tests for activate refresh, current-school deactivate/delete invalidation, and non-current/failed/conflict no-op in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/SingleLifecycleOutcomes.spec.js`
- [X] T053 [P] [US3] Extend detail/edit lifecycle tests for current-school target matching and failed/activation/restore no-invalidation behavior in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/SchoolDetailEditPage.spec.js`
- [X] T054 [P] [US3] Extend selector tests for deliberate refresh, newly active eligibility, no auto-selection, and authorized School administration path in `schoolmaster-frontend/tests/unit/school-context-selection/components/SchoolContextSelector.spec.js`
- [X] T055 [P] [US3] Extend browser flow for activate-then-refresh-select, conflict failure, and current-school invalidation in `schoolmaster-frontend/e2e/school-context-selection.spec.js`

### Implementation for User Story 3

- [X] T056 [P] [US3] Invoke shared context invalidation after successful current-school deactivate/delete and refresh eligibility after activation in `schoolmaster-frontend/src/pages/admin-system/schools/SchoolsListPage.vue`
- [X] T057 [P] [US3] Invoke shared context invalidation only for successful matching lifecycle outcomes in `schoolmaster-frontend/src/pages/admin-system/schools/SchoolDetailPage.vue`
- [X] T058 [P] [US3] Invoke shared context invalidation only for successful matching edit-page lifecycle outcomes in `schoolmaster-frontend/src/modules/schools/routes/SchoolEditPage.vue`
- [X] T059 [US3] Add deliberate selector refresh and authorized School administration navigation without auto-selecting lifecycle results in `schoolmaster-frontend/src/components/auth/SchoolContextSelector.vue` and `schoolmaster-frontend/src/pages/auth/SchoolSelectionPage.vue`
- [X] T060 [P] [US3] Keep select-school and activate-school labels, feedback, and recovery actions distinct in `schoolmaster-frontend/src/locales/auth.js` and `schoolmaster-frontend/src/locales/admin-system.js`

**Checkpoint**: All three stories work with lifecycle and context selection as
separate authoritative actions.

---

## Phase 6: Polish and Cross-Cutting Verification

**Purpose**: Complete contract, regression, accessibility, performance, and
delivery evidence across all repositories.

- [X] T061 [P] Lint aggregate OpenAPI and resolve only feature-caused failures in `schoolmaster-specs/api/components/schemas/auth/AuthSession.yaml`
- [X] T062 [P] Run focused `CurrentUserApiTest` then full PHPUnit suite and resolve feature regressions in `schoolmaster-backend/tests/Feature/CurrentUserApiTest.php`
- [X] T063 [P] Run focused school-context/auth/System Administrator Vitest suites then full Vitest suite and resolve regressions in `schoolmaster-frontend/tests/unit/school-context-selection/`, `schoolmaster-frontend/tests/unit/auth/`, and `schoolmaster-frontend/tests/unit/system-admin-master/`
- [X] T064 [P] Run production build and non-mutating targeted lint checks for changed frontend files using `schoolmaster-frontend/package.json`
- [X] T065 [P] Run Chromium then configured cross-browser selector Playwright suite at 390px, 768px, and 1440px, plus 100 selection and 100 restoration timing attempts with 100 active fixtures and deterministic 300 ms responses, and resolve failures in `schoolmaster-frontend/e2e/school-context-selection.spec.js`
- [X] T066 [P] Audit direct Axios/API use, CNPJ/document rendering, poll/timer additions, and duplicate observer installation across `schoolmaster-frontend/src/pages/`, `schoolmaster-frontend/src/components/`, `schoolmaster-frontend/src/composables/`, `schoolmaster-frontend/src/router/`, and `schoolmaster-frontend/src/services/`
- [ ] T067 Record Redocly, PHPUnit, Vitest, build, browser, source-audit, responsive, keyboard, NVDA/VoiceOver, separate 95-of-100 selection/restoration timing, and 9-of-10 selection/activate-then-select usability evidence in `schoolmaster-specs/specs/033-activate-school-ui/quickstart.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies.
- **Foundational (Phase 2)**: Depends on Setup; blocks all user stories.
- **US1 (Phase 3)**: Depends on Foundational and delivers MVP selector.
- **US2 (Phase 4)**: State/error/router work can begin after Foundational, but
  complete switch-flow validation depends on US1 confirmation and selector.
- **US3 (Phase 5)**: Lifecycle test scaffolding can begin after Foundational;
  integration depends on US1 selector and US2 shared invalidation action.
- **Polish (Phase 6)**: Depends on all selected stories.

### User Story Dependency Graph

```text
Setup -> Foundation -> US1 (MVP) -> US2 -> US3 -> Polish
                         \-----------> US3 test preparation
```

### Within Each User Story

- Write listed tests first and confirm they fail for missing behavior.
- Implement contract/service/store state before orchestration and UI consumers.
- Keep components free of direct HTTP access; use props down and events up.
- Re-run story-focused tests at checkpoint before starting next dependent story.

### Parallel Opportunities

- T001 and T002 can run in parallel.
- Foundation tests T003-T005 can run in parallel; backend service/resource work
  T006-T007 and mapping work T009-T010 can run in parallel afterward.
- US1 tests T011-T020 can run in parallel; presentational components T023-T024,
  auth cleanup T027, shell control T030, and locales T032 use distinct files.
- US2 tests T033-T042 can run in parallel; classifier T045, route resolver T047,
  guard integration T050, and locales T051 use distinct files.
- US3 tests T052-T055 and lifecycle page changes T056-T058 can run in parallel.
- Verification T061-T066 can run in parallel after all implementation phases.

---

## Parallel Example: User Story 1

```bash
Task: "T011 [US1] Add selection-service contract tests"
Task: "T012 [US1] Add stale-safe selection composable tests"
Task: "T014 [US1] Add choice identity and no-CNPJ component tests"
Task: "T017 [US1] Add requested-route guard tests"
```

## Parallel Example: User Story 2

```bash
Task: "T033 [US2] Add bounded bootstrap fallback tests"
Task: "T036 [US2] Add authoritative invalidation observer tests"
Task: "T038 [US2] Add route compatibility tests"
Task: "T040 [US2] Add pending-work guard tests"
```

## Parallel Example: User Story 3

```bash
Task: "T052 [US3] Add list lifecycle integration tests"
Task: "T053 [US3] Add detail/edit lifecycle integration tests"
Task: "T054 [US3] Add deliberate selector refresh tests"
Task: "T055 [US3] Add activate-then-select browser flow"
```

---

## Implementation Strategy

### MVP First

1. Complete Setup and Foundational phases.
2. Complete US1 active-school discovery, selector, confirmation, read-only
   shell context presentation, and requested-route resume.
3. Run US1 focused PHPUnit/Vitest/Playwright checks.
4. Stop for review before enabling shell switching or adding restoration and
   lifecycle invalidation.

### Incremental Delivery

1. Foundation: contract and `/auth/me` conformance plus School normalization.
2. US1: active-school selection page and read-only shell context indicator.
3. US2: guarded shell entry, restoration, switch cleanup, route safety,
   persistence, and authoritative invalidation.
4. US3: lifecycle activate-refresh-select and current-school invalidation.
5. Polish: full regression, browser, accessibility, usability, and evidence.

### Parallel Team Strategy

1. Contract/backend owner completes T001 and T003-T008.
2. Frontend contract owner completes T002, T004-T005, and T009-T010.
3. After Foundation, selector owner implements US1 while context/router owner
   prepares US2 tests.
4. Lifecycle owner prepares US3 tests, then integrates after US2 invalidation
   action exists.

## Notes

- `[P]` tasks touch distinct files or can execute independently.
- No migration, new endpoint, package, polling, or client-side authorization.
- `GET /api/v1/schools` is a selector source only for exact System
  Administrator.
- Generic 401/403 and semantically similar errors never imply context
  invalidity; match exact code, school header, and generation.
- Platform-wide routes remain platform-wide when school context changes.
- Update task checkboxes as implementation progresses.
