# Tasks: System Administrator Master Access

**Input**: Design documents from `specs/031-system-admin-master/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/system-admin-master-contract.md`, `quickstart.md`

**Tests**: Required. This feature changes global authorization behavior,
frontend route guards, REST authorization notes, tenant/subject context gates,
and audit expectations.

**Organization**: Tasks are grouped by user story so each story can be
implemented and tested independently after shared master-access foundations are
complete.

**Repository Scope**: `schoolmaster-specs` owns security documentation,
multi-tenant documentation, OpenAPI authorization notes, and feature evidence.
`schoolmaster-backend` owns Laravel authorization, policy, tenant/subject
context, audit, and PHPUnit coverage. `schoolmaster-frontend` owns route guard,
navigation/action visibility, context states, and Vitest coverage.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because the task changes different files or can
  be completed without depending on another incomplete task.
- **[Story]**: User story label for story phases only.
- Every task includes an exact file path.

## Phase 1: Setup

**Purpose**: Confirm affected contracts, implementation targets, and evidence
locations before shared authorization work starts.

- [ ] T001 Review current System Administrator, permission, tenant, support, and self-service authorization language and record contradictions in `specs/031-system-admin-master/quickstart.md`
- [ ] T002 [P] Inventory protected OpenAPI operations requiring master-access notes in `api/openapi.yaml` and record the operation groups in `specs/031-system-admin-master/quickstart.md`
- [ ] T003 [P] Inventory protected frontend route modules and route metadata requiring master-access treatment and record route groups in `specs/031-system-admin-master/quickstart.md`
- [ ] T004 [P] Inventory backend authorization policy and permission-check entry points requiring centralized master-access treatment and record entry points in `specs/031-system-admin-master/quickstart.md`
- [ ] T005 [P] Record backend fixture requirements for System Administrator, limited role, active school, inactive school, selected subject, and audit evidence in `specs/031-system-admin-master/quickstart.md`
- [ ] T006 [P] Create frontend test fixtures for System Administrator session, limited session, route metadata, selected school context, selected subject context, and denial states in `schoolmaster-frontend/tests/unit/system-admin-master/fixtures/masterAccess.fixtures.js`
- [ ] T007 [P] Confirm Redocly, PHPUnit, and Vitest commands and evidence capture locations in `specs/031-system-admin-master/quickstart.md`

---

## Phase 2: Foundational

**Purpose**: Shared specs, contract wording, authorization helpers, and
frontend guard foundations required before any user story can be implemented.

**Critical**: No user story implementation begins before this phase completes.

### Foundational Contract and Documentation Tasks

- [ ] T008 Update System Administrator master-access rule and remove contradictory no-bypass wording in `docs/security.md`
- [ ] T009 Update tenant selection, any-active-school access, selected-school scoping, and platform-wide exception rules in `docs/multi-tenant.md`
- [ ] T010 [P] Update frontend route guard and navigation guidance for System Administrator master access in `docs/frontend-guidelines.md`
- [ ] T011 [P] Update backend policy and authorization guidance for System Administrator master access in `docs/backend-guidelines.md`
- [ ] T012 Add reusable OpenAPI authorization description language for System Administrator master access in `api/openapi.yaml`
- [ ] T013 Update common forbidden response examples to distinguish permission denial from tenant-context denial in `api/components/responses/common/AdministrationForbidden.yaml`
- [ ] T014 Update classroom or module-specific forbidden response examples to preserve non-permission prerequisite denials in `api/components/responses/classroom/ClassroomRosterForbidden.yaml`
- [ ] T015 Update administration foundation permission matrix notes for System Administrator master access in `specs/018-administration-foundation-ui/contracts/administration-foundation-ui-contract.md`
- [ ] T016 [P] Update administration lifecycle UI authorization notes for System Administrator master access in `specs/020-administration-lifecycle-ui/contracts/administration-lifecycle-ui-contract.md`
- [ ] T017 [P] Update platform support UI permission-gate notes for System Administrator master access without bypassing support approval gates in `specs/027-platform-support-ui/contracts/platform-support-ui-contract.md`
- [ ] T018 [P] Update student self-service UI context notes for selected subject context under System Administrator master access in `specs/024-student-self-service-ui/contracts/student-self-service-ui-contract.md`
- [ ] T019 [P] Update guardian self-service UI context notes for selected subject context under System Administrator master access in `specs/025-guardian-self-service-ui/contracts/guardian-self-service-ui-contract.md`

### Foundational Backend Tasks

- [ ] T020 [P] Add backend authorization fixture builders for System Administrator, limited users, active schools, inactive schools, selected subjects, permissions, and audit outcomes in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SystemAdminMasterAccessFixtures.php`
- [ ] T021 [P] Add unit coverage for centralized master-role detection and permission override boundaries in `schoolmaster-backend/tests/Unit/Auth/SystemAdministratorAccessTest.php`
- [ ] T022 Define centralized System Administrator master-access service or helper with permission-only override semantics in `schoolmaster-backend/app/Services/Auth/SystemAdministratorAccess.php`
- [ ] T023 Integrate centralized System Administrator master-access checks into existing policy authorization boundary in `schoolmaster-backend/app/Policies/Concerns/AuthorizesSystemAdministrator.php`
- [ ] T024 Preserve tenant-context, subject-context, account-state, session-state, school-state, release-state, approval, and safety-gate checks around the new permission override in `schoolmaster-backend/app/Services/Auth/SystemAdministratorAccess.php`

### Foundational Frontend Tasks

- [ ] T025 [P] Add frontend route guard unit coverage for master-role permission override and non-permission prerequisite preservation in `schoolmaster-frontend/tests/unit/system-admin-master/router/masterAccessGuard.spec.js`
- [ ] T026 [P] Add frontend visibility helper unit coverage for System Administrator protected navigation/action visibility and limited-role denial behavior in `schoolmaster-frontend/tests/unit/system-admin-master/composables/useMasterAccessVisibility.spec.js`
- [ ] T027 Update centralized route guard permission evaluation for System Administrator master access in `schoolmaster-frontend/src/router/guards/authorization.js`
- [ ] T028 Update shared permission or session composable to expose System Administrator master-access status in `schoolmaster-frontend/src/composables/auth/useAuthorization.js`
- [ ] T029 Update navigation and action visibility helper to treat System Administrator as satisfying route permission metadata in `schoolmaster-frontend/src/composables/admin-system/useAdminNavigation.js`

**Checkpoint**: Shared documentation, OpenAPI language, backend override
boundary, and frontend route guard foundation are ready for user story work.

---

## Phase 3: User Story 1 - Access Every Authorized Workspace (Priority: P1) MVP

**Goal**: System Administrator can open every released protected route and
perform protected actions without explicit feature-specific permissions, while
non-permission prerequisites still block access.

**Independent Test**: Sign in as System Administrator with no extra
feature-specific permissions and verify protected navigation, direct route
access, and protected actions are allowed unless a tenant target, subject
target, account/session state, release state, or safety prerequisite is missing.

### Tests for User Story 1

- [ ] T030 [P] [US1] Add OpenAPI lint expectation for master-access authorization wording across protected operation groups in `specs/031-system-admin-master/quickstart.md`
- [ ] T031 [P] [US1] Add backend feature tests proving System Administrator can call every released protected operation group listed in `api/openapi.yaml` without feature-specific permissions in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/ProtectedOperationMasterAccessTest.php`
- [ ] T032 [P] [US1] Add backend feature tests proving limited non-System Administrator users without required permissions still receive documented permission denial for every released protected operation group in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/LimitedRolePermissionDenialTest.php`
- [ ] T033 [P] [US1] Add frontend route tests proving System Administrator can open every released protected route group inventoried in T003 after session resolution in `schoolmaster-frontend/tests/unit/system-admin-master/router/systemAdminRouteAccess.spec.js`
- [ ] T034 [P] [US1] Add frontend visibility tests proving every released protected navigation destination and route action is visible to System Administrator and still hidden for limited users in `schoolmaster-frontend/tests/unit/system-admin-master/navigation/systemAdminNavigationVisibility.spec.js`

### Implementation for User Story 1

- [ ] T035 [US1] Update protected operation authorization notes for System Administrator master access in `api/openapi.yaml`
- [ ] T036 [US1] Apply master-access authorization wording to users, roles, permissions, schools, academics, guardians, reports, platform support, classroom, teacher, student, and assessment path descriptions listed in `api/openapi.yaml`
- [ ] T037 [US1] Wire System Administrator permission override into every backend policy entry point inventoried in T004 while preserving non-permission gates in `schoolmaster-backend/app/Policies/Concerns/AuthorizesSystemAdministrator.php`
- [ ] T038 [US1] Wire System Administrator permission override into shared backend service authorization checks used outside policies in `schoolmaster-backend/app/Services/Auth/SystemAdministratorAccess.php`
- [ ] T039 [US1] Update backend current-user or session resource output to expose enough role context for frontend master-access evaluation in `schoolmaster-backend/app/Http/Resources/Auth/CurrentUserResource.php`
- [ ] T040 [US1] Update frontend session store mapping for System Administrator master-access status in `schoolmaster-frontend/src/stores/auth.store.js`
- [ ] T041 [US1] Update frontend route guard direct navigation behavior for System Administrator protected routes in `schoolmaster-frontend/src/router/guards/authorization.js`
- [ ] T042 [US1] Update frontend admin, platform, reporting, teacher, student, guardian, and assessment navigation visibility to honor master-access status in `schoolmaster-frontend/src/router/index.js`
- [ ] T043 [US1] Document protected route and operation allow evidence for System Administrator and limited-role denial evidence in `specs/031-system-admin-master/quickstart.md`

**Checkpoint**: User Story 1 is independently testable as the MVP.

---

## Phase 4: User Story 2 - Operate Inside School Contexts (Priority: P2)

**Goal**: System Administrator can select any active school for school-scoped
work, and all school-owned responses remain limited to the selected school
unless an operation is explicitly platform-wide.

**Independent Test**: Sign in as System Administrator, select any active school,
open school-owned resources, confirm only selected-school data appears, switch
to another active school, and confirm stale data clears before the new school's
data loads.

### Tests for User Story 2

- [ ] T044 [P] [US2] Add backend feature tests proving System Administrator may select any active school context and is blocked from inactive or missing school context in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolContextSelectionTest.php`
- [ ] T045 [P] [US2] Add backend tenant isolation tests proving school-scoped System Administrator operations return only selected-school data and never unscoped cross-school data in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolScopedTenantIsolationTest.php`
- [ ] T046 [P] [US2] Add backend platform-wide operation tests proving any cross-school output is allowed only for documented platform-wide operations in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/PlatformWideOperationScopeTest.php`
- [ ] T047 [P] [US2] Add frontend school-context tests proving System Administrator can select any active school, inactive schools are unavailable, and missing-school state blocks school-owned loading in `schoolmaster-frontend/tests/unit/system-admin-master/schoolContext/systemAdminSchoolContext.spec.js`
- [ ] T048 [P] [US2] Add frontend stale-data tests proving school-owned data clears on school context switch before loading the new selected school's data in `schoolmaster-frontend/tests/unit/system-admin-master/schoolContext/schoolContextSwitchClearsData.spec.js`

### Implementation for User Story 2

- [ ] T049 [US2] Update active school selection rules for System Administrator in `schoolmaster-backend/app/Services/School/SchoolContextService.php`
- [ ] T050 [US2] Preserve active-school validation and inactive-school denial for System Administrator in `schoolmaster-backend/app/Http/Middleware/ResolveSchoolContext.php`
- [ ] T051 [US2] Enforce selected-school scoping for System Administrator school-owned queries in `schoolmaster-backend/app/Services/Concerns/EnforcesSchoolContext.php`
- [ ] T052 [US2] Document platform-wide operation exceptions and selected-school scoping in `api/openapi.yaml`
- [ ] T053 [US2] Update frontend active-school selection store so System Administrator can select any active school in `schoolmaster-frontend/src/stores/school-context.store.js`
- [ ] T054 [US2] Update frontend school-context guard to block school-owned routes until active selected school context exists in `schoolmaster-frontend/src/router/guards/schoolContext.js`
- [ ] T055 [US2] Update frontend stale school-owned data cleanup on context change in `schoolmaster-frontend/src/composables/school-context/useSchoolContextSwitch.js`
- [ ] T056 [US2] Document any-active-school selection, inactive-school denial, selected-school isolation, and context-switch evidence in `specs/031-system-admin-master/quickstart.md`

**Checkpoint**: User Story 2 is independently testable for school selection,
tenant isolation, and platform-wide exception boundaries.

---

## Phase 5: User Story 3 - Audit Master Access Decisions (Priority: P3)

**Goal**: System Administrator writes and lifecycle actions record
master-access audit evidence, while read-only navigation keeps existing access
evidence unless an operation already requires audit records.

**Independent Test**: Review security, route, contract, and test evidence to
confirm System Administrator writes/lifecycle actions mark master access,
school context remains required, and non-System Administrator denial behavior
is unchanged.

### Tests for User Story 3

- [ ] T057 [P] [US3] Add backend audit tests proving System Administrator create, update, delete, restore, activate, deactivate, import, retry, cancel, approve, and revoke actions record master-access audit evidence in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessAuditEvidenceTest.php`
- [ ] T058 [P] [US3] Add backend tests proving read-only System Administrator list/detail operations do not create new master-access audit evidence unless the existing operation already audits reads in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessReadAuditBoundaryTest.php`
- [ ] T059 [P] [US3] Add backend tests proving selected subject-context gates for identity-owned self-service plus approval workflows, support opt-ins, explicit confirmations, file safety gates, closed-period checks, and other safety gates still block System Administrator when prerequisites are unmet in `schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessSafetyGateTest.php`
- [ ] T060 [P] [US3] Add frontend tests proving approval, confirmation, support opt-in, file safety, closed-period, missing-subject, and missing-school states remain distinct from permission denial in `schoolmaster-frontend/tests/unit/system-admin-master/states/masterAccessNonPermissionStates.spec.js`
- [ ] T061 [P] [US3] Add contract tests or review checks proving audit-marker expectations and non-permission prerequisite language are present in `specs/031-system-admin-master/contracts/system-admin-master-contract.md`

### Implementation for User Story 3

- [ ] T062 [US3] Add master-access marker support to existing audit metadata builder in `schoolmaster-backend/app/Services/Audit/AuditMetadataBuilder.php`
- [ ] T063 [US3] Record master-access audit evidence for System Administrator writes and lifecycle actions in `schoolmaster-backend/app/Services/Audit/AuditLogger.php`
- [ ] T064 [US3] Propagate selected school context and selected subject context into master-access audit metadata in `schoolmaster-backend/app/Services/Audit/AuditContext.php`
- [ ] T065 [US3] Preserve selected subject-context enforcement for identity-owned self-service plus approval workflow, support opt-in, explicit confirmation, file safety, closed-period, and other business gate checks around master access in `schoolmaster-backend/app/Services/Auth/SystemAdministratorAccess.php`
- [ ] T066 [US3] Update frontend feedback mapping so non-permission prerequisite denials remain distinct for System Administrator in `schoolmaster-frontend/src/services/error-mapper.js`
- [ ] T067 [US3] Update frontend subject-context guard for identity-owned self-service routes under System Administrator access in `schoolmaster-frontend/src/router/guards/subjectContext.js`
- [ ] T068 [US3] Document audit evidence, safety-gate denial, read-audit boundary, and selected-subject context validation in `specs/031-system-admin-master/quickstart.md`

**Checkpoint**: User Story 3 is independently testable for audit evidence and
non-permission gate preservation.

---

## Phase 6: Polish and Cross-Cutting Concerns

**Purpose**: Full-flow verification, documentation cleanup, contract linting,
and delivery evidence across all stories.

- [ ] T069 [P] Run Redocly validation for `api/openapi.yaml` and record result in `specs/031-system-admin-master/quickstart.md`
- [ ] T070 [P] Run backend PHPUnit suite for System Administrator master access and record result in `specs/031-system-admin-master/quickstart.md`
- [ ] T071 [P] Run frontend Vitest suite for System Administrator route, navigation, school-context, subject-context, and non-permission state tests and record result in `specs/031-system-admin-master/quickstart.md`
- [ ] T072 [P] Review `docs/security.md`, `docs/multi-tenant.md`, affected feature specs, and affected contracts for contradictory no-bypass language and record result in `specs/031-system-admin-master/quickstart.md`
- [ ] T073 [P] Review OpenAPI protected operations for consistent permission-denial, tenant-context, subject-context, and platform-wide language in `api/openapi.yaml`
- [ ] T074 [P] Review backend implementation for direct permission checks that bypass centralized System Administrator master-access behavior and record findings in `specs/031-system-admin-master/quickstart.md`
- [ ] T075 [P] Review frontend implementation for direct permission checks outside shared guards/composables and record findings in `specs/031-system-admin-master/quickstart.md`
- [ ] T076 Verify quickstart manual scenarios for platform route access, any-active-school selection, selected-school scoping, self-service subject context, safety gates, and audit markers in `specs/031-system-admin-master/quickstart.md`
- [ ] T077 Update final implementation evidence and remaining known limitations in `specs/031-system-admin-master/quickstart.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies. Complete before shared foundation.
- **Foundational (Phase 2)**: Depends on Setup. Blocks all user stories.
- **User Stories (Phases 3-5)**: Depend on Foundational completion.
- **Polish (Phase 6)**: Depends on all implemented user stories.

### User Story Dependencies

- **US1 (P1)**: Starts after Foundational. MVP scope.
- **US2 (P2)**: Starts after Foundational. It can be tested with active school
  fixtures independently, but integrates naturally with US1 route access.
- **US3 (P3)**: Starts after Foundational. It can be tested with write,
  lifecycle, and safety-gate fixtures independently, but depends on the same
  master-access decision boundary from US1.

### Within Each User Story

- Write contract, backend, and frontend tests before implementation.
- Contract and documentation updates before backend/frontend behavior changes
  that depend on the changed rule.
- Backend authorization helper and policies before protected operation wiring.
- Frontend session/route guard changes before navigation/action visibility.
- Story evidence recorded in quickstart before moving to polish.

### Parallel Opportunities

- Setup inventory tasks T002-T007 can run in parallel after T001 starts.
- Foundational documentation tasks T010-T019 can run in parallel after T008 and
  T009 define canonical wording.
- Foundational backend tasks T020-T024 can run in parallel with frontend tasks
  T025-T029 after T012 defines shared contract language.
- Story test tasks marked [P] can run in parallel within each story.
- US2 and US3 can begin after Foundational if separate implementers use mocked
  or fixture-backed master-access behavior.
- Polish validation tasks T069-T075 can run in parallel after implemented
  stories are ready.

---

## Parallel Example: User Story 1

```bash
Task: "T031 [P] [US1] Add backend feature tests for every released protected operation group in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/ProtectedOperationMasterAccessTest.php"
Task: "T032 [P] [US1] Add backend feature tests for limited-role permission denial in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/LimitedRolePermissionDenialTest.php"
Task: "T033 [P] [US1] Add frontend route tests in schoolmaster-frontend/tests/unit/system-admin-master/router/systemAdminRouteAccess.spec.js"
Task: "T034 [P] [US1] Add frontend navigation visibility tests in schoolmaster-frontend/tests/unit/system-admin-master/navigation/systemAdminNavigationVisibility.spec.js"
```

## Parallel Example: User Story 2

```bash
Task: "T044 [P] [US2] Add backend active-school selection tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolContextSelectionTest.php"
Task: "T045 [P] [US2] Add backend selected-school isolation tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/SchoolScopedTenantIsolationTest.php"
Task: "T047 [P] [US2] Add frontend school-context selection tests in schoolmaster-frontend/tests/unit/system-admin-master/schoolContext/systemAdminSchoolContext.spec.js"
Task: "T048 [P] [US2] Add frontend context-switch stale-data tests in schoolmaster-frontend/tests/unit/system-admin-master/schoolContext/schoolContextSwitchClearsData.spec.js"
```

## Parallel Example: User Story 3

```bash
Task: "T057 [P] [US3] Add backend audit evidence tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessAuditEvidenceTest.php"
Task: "T058 [P] [US3] Add backend read-audit boundary tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessReadAuditBoundaryTest.php"
Task: "T059 [P] [US3] Add backend safety gate tests in schoolmaster-backend/tests/Feature/SystemAdminMasterAccess/MasterAccessSafetyGateTest.php"
Task: "T060 [P] [US3] Add frontend non-permission state tests in schoolmaster-frontend/tests/unit/system-admin-master/states/masterAccessNonPermissionStates.spec.js"
```

---

## Implementation Strategy

### MVP First

1. Complete Phase 1 setup.
2. Complete Phase 2 foundation.
3. Complete Phase 3 / US1.
4. Validate that System Administrator can access protected route and operation
   groups without explicit feature permissions, while limited roles remain
   denied.
5. Stop for review before expanding tenant-context and audit behavior.

### Incremental Delivery

1. US1: master permission-check override and protected route/operation access.
2. US2: any-active-school selection and selected-school tenant isolation.
3. US3: master-access audit markers and business safety gate preservation.
4. Polish: full contract/backend/frontend verification and manual evidence.

### Parallel Team Strategy

With multiple implementers:

1. One owner updates `schoolmaster-specs` documentation and OpenAPI language.
2. One backend owner implements centralized master-access authorization and
   PHPUnit coverage.
3. One frontend owner implements route guard/navigation/context behavior and
   Vitest coverage.
4. Owners reconcile quickstart evidence before final review.

## Notes

- [P] tasks touch different files or can be completed independently after
  their phase prerequisites.
- Keep tenant and subject context denial separate from permission denial.
- Do not add unscoped school-owned output unless the operation is documented
  as platform-wide.
- Do not add new success response fields for this feature.
