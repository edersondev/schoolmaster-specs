# Tasks: System Administrator Frontend Access

**Input**: Design documents from `specs/032-system-admin-frontend/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/frontend-system-admin-access-contract.md`, `quickstart.md`

**Tests**: Required. This feature changes global frontend authorization,
protected navigation and action visibility, school-context state, and
prerequisite-specific feedback.

**Repository Scope**: `schoolmaster-frontend` is the only implementation
repository. `schoolmaster-specs` owns the plan, contract, and implementation
evidence. The completed backend feature `031-system-admin-master` and existing
OpenAPI contract are consumed unchanged.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because the task changes different files or has
  no dependency on another incomplete task.
- **[Story]**: User story label for story phases only.
- Run code tasks from the `schoolmaster-frontend` repository root. Paths that
  begin with `specs/` refer to its mounted specification repository.

## Phase 1: Setup

**Purpose**: Confirm the released frontend surfaces and existing session/context
contracts before changing shared authorization behavior.

- [ ] T001 Inventory every released non-identity-owned protected route, navigation destination, and action with its permission and context metadata in `specs/specs/032-system-admin-frontend/quickstart.md`
- [ ] T002 [P] Confirm the existing session role collection supplies active role name and platform scope without a response change in `src/contracts/auth/authSession.contract.js`
- [ ] T003 [P] Record frontend test fixtures for System Administrator, limited role, unresolved session, active school, missing school, changed school, and identity-owned self-service in `tests/unit/system-admin-master/fixtures/masterAccess.fixtures.js`
- [ ] T004 [P] Confirm existing route, navigation, school-context, subject-context, and error boundaries against the frontend access contract in `specs/specs/032-system-admin-frontend/quickstart.md`

---

## Phase 2: Foundational

**Purpose**: Establish one shared master-access decision and preserve the
non-permission boundaries that every user story depends on.

**Critical**: No user-story implementation begins before this phase completes.

- [ ] T005 [P] Add unit coverage for exact active platform `System Administrator` recognition, limited-role behavior, and unresolved-session denial in `tests/unit/system-admin-master/auth/useAuthorization.spec.js`
- [ ] T006 [P] Add unit coverage proving identity-owned student and guardian self-service routes do not use master access in `tests/unit/system-admin-master/router/identityOwnedRouteBoundary.spec.js`
- [ ] T007 Define the exact active platform `System Administrator` predicate and permission-satisfaction behavior using the existing session role collection in `src/contracts/auth/authSession.contract.js`
- [ ] T008 Preserve existing session-resolution, active-account, release, school-context, subject-context, approval, and safety prerequisite evaluation around the shared predicate in `src/router/authGuards.js`
- [ ] T009 Document the frontend role-recognition, context-switch, and feedback contract in `specs/specs/032-system-admin-frontend/contracts/frontend-system-admin-access-contract.md`

**Checkpoint**: Shared role detection and prerequisite boundaries are ready for
route, navigation, and action adoption.

---

## Phase 3: User Story 1 - Access Protected Frontend Workspaces (Priority: P1) MVP

**Goal**: A resolved System Administrator can access released
non-identity-owned protected routes and see their navigation destinations and
actions without feature-specific permissions; limited roles retain denial.

**Independent Test**: Use a zero-permission active System Administrator session
to open every inventoried route group directly and through navigation, then
repeat with a limited role and verify current denial and hidden-action behavior.

### Tests for User Story 1

- [ ] T010 [P] [US1] Add route-guard tests for direct System Administrator access to every inventoried released non-identity-owned protected route group after session resolution in `tests/unit/system-admin-master/router/systemAdminRouteAccess.spec.js`
- [ ] T011 [P] [US1] Add route-guard tests proving limited roles still receive the existing permission-denied outcome for the same route metadata in `tests/unit/system-admin-master/router/limitedRolePermissionDenial.spec.js`
- [ ] T012 [P] [US1] Add navigation tests proving System Administrator sees every inventoried released non-identity-owned protected destination while limited roles do not in `tests/unit/system-admin-master/navigation/systemAdminNavigationVisibility.spec.js`
- [ ] T013 [P] [US1] Add action-visibility tests proving System Administrator sees applicable released protected actions without changing hidden or unavailable states for limited roles in `tests/unit/system-admin-master/navigation/systemAdminActionVisibility.spec.js`

### Implementation for User Story 1

- [ ] T014 [US1] Expose resolved System Administrator status through the existing session store without changing the session response contract in `src/stores/auth/sessionStore.js`
- [ ] T015 [US1] Apply the centralized master-access predicate to protected direct-navigation permission evaluation after session resolution in `src/router/authGuards.js`
- [ ] T016 [US1] Apply the centralized master-access predicate to shared navigation and quick-action visibility evaluation in `src/composables/admin-system/useAdminShellPermissions.js` and `src/composables/admin-system/useAdminQuickActions.js`
- [ ] T017 [US1] Update shared route fallback and protected-route permission consumers to use the centralized authorization decision instead of local feature-permission checks in `src/router/authGuards.js` and `src/router/adminFallbackRoute.js`
- [ ] T018 [US1] Record protected route, navigation, action allow evidence and limited-role denial evidence in `specs/specs/032-system-admin-frontend/quickstart.md`

**Checkpoint**: User Story 1 is independently testable as the MVP.

---

## Phase 4: User Story 2 - Keep School and Subject Boundaries (Priority: P2)

**Goal**: System Administrator can work within any active selected school
without stale cross-school UI state, while identity-owned self-service retains
existing subject and ownership boundaries.

**Independent Test**: Select an active school, load school-owned views, change
schools, and verify prior school state is cleared before new data appears. Test
missing school and identity-owned self-service routes separately.

### Tests for User Story 2

- [ ] T019 [P] [US2] Add school-context tests proving a System Administrator can select any active school while missing or inactive context blocks school-owned loading in `tests/unit/system-admin-master/schoolContext/systemAdminSchoolContext.spec.js`
- [ ] T020 [P] [US2] Add stale-data tests proving prior school-owned state is cleared and prior-context responses cannot repopulate the new context in `tests/unit/system-admin-master/schoolContext/schoolContextSwitchClearsData.spec.js`
- [ ] T021 [P] [US2] Add subject-context tests proving master access does not create selected subject context or bypass identity-owned student and guardian boundaries in `tests/unit/system-admin-master/router/subjectContextBoundary.spec.js`
- [ ] T022 [P] [US2] Add platform-wide presentation tests proving an active selected school neither filters nor leaks into already documented platform-wide views in `tests/unit/system-admin-master/schoolContext/platformWidePresentationScope.spec.js`

### Implementation for User Story 2

- [ ] T023 [US2] Allow System Administrator to use any active school through the existing selected-school session state without weakening active-school validation in `src/stores/auth/sessionStore.js`
- [ ] T024 [US2] Preserve the school-context guard so school-owned routes wait for a valid active selected school before module loading in `src/router/authGuards.js`
- [ ] T025 [US2] Create school-context switch orchestration that clears and invalidates school-owned state before loading against a changed selected school and rejects stale prior-context results in `src/composables/auth/useSchoolContextSwitch.js`
- [ ] T026 [US2] Preserve existing identity-owned student and guardian route boundaries outside the master-access override in `src/composables/student/useStudentWorkspaceContext.js` and `src/composables/guardian/useGuardianWorkspaceContext.js`
- [ ] T027 [US2] Record active-school selection, missing-context denial, selected-school isolation, platform-wide presentation, stale-data cleanup, and identity-owned boundary evidence in `specs/specs/032-system-admin-frontend/quickstart.md`

**Checkpoint**: User Story 2 is independently testable for tenant-safe frontend
context behavior.

---

## Phase 5: User Story 3 - Preserve Non-Permission States (Priority: P3)

**Goal**: System Administrator receives the existing session, context, approval,
and safety blocked states instead of a misleading feature-permission denial.

**Independent Test**: Trigger representative missing-context, session, approval,
confirmation, support, file-safety, and closed-period conditions and verify the
frontend preserves each established state without creating an audit request.

### Tests for User Story 3

- [ ] T028 [P] [US3] Add error-state tests proving missing school, missing subject, expired session, approval, confirmation, support, file-safety, and closed-period outcomes remain distinct from missing permission for System Administrator in `tests/unit/system-admin-master/states/masterAccessNonPermissionStates.spec.js`
- [ ] T029 [P] [US3] Add client-side audit-boundary tests proving a System Administrator state-changing action does not create a new frontend audit request or event in `tests/unit/system-admin-master/states/masterAccessAuditBoundary.spec.js`
- [ ] T030 [P] [US3] Add session-restoration tests proving unresolved session state does not temporarily grant or deny System Administrator access in `tests/unit/system-admin-master/router/masterAccessSessionResolution.spec.js`

### Implementation for User Story 3

- [ ] T031 [US3] Preserve distinct session, tenant, student self-service, guardian self-service, and administration prerequisite feedback without reclassifying it as missing permission in `src/services/auth/authErrorMapper.js`, `src/services/admin-system/administration-error-mapper.js`, `src/services/student/studentSelfServiceFeedbackMapper.js`, and `src/services/guardian/guardianSelfServiceFeedbackMapper.js`
- [ ] T032 [US3] Preserve route feedback behavior for unresolved, expired, or invalid session states around master-access evaluation in `src/router/authGuards.js`
- [ ] T033 [US3] Record non-permission feedback, session-resolution, and client-side audit-boundary evidence in `specs/specs/032-system-admin-frontend/quickstart.md`

**Checkpoint**: All user stories are independently testable with master access
limited to feature-permission decisions.

---

## Phase 6: Polish and Cross-Cutting Concerns

**Purpose**: Complete exhaustive coverage, quality validation, and delivery
evidence across the frontend change.

- [ ] T034 [P] Review all frontend route and navigation modules for local feature-permission checks that bypass the shared master-access predicate and record findings in `specs/specs/032-system-admin-frontend/quickstart.md`
- [ ] T035 [P] Review school-owned stores, composables, and request orchestration for stale cross-school state and record findings in `specs/specs/032-system-admin-frontend/quickstart.md`
- [ ] T036 [P] Run the focused System Administrator Vitest suite and record results in `specs/specs/032-system-admin-frontend/quickstart.md`
- [ ] T037 [P] Run the frontend production build and record results in `specs/specs/032-system-admin-frontend/quickstart.md`
- [ ] T038 Verify no component or route guard performs direct Axios access and record the scan result in `specs/specs/032-system-admin-frontend/quickstart.md`
- [ ] T039 Update final implementation evidence, route inventory coverage, and remaining limitations in `specs/specs/032-system-admin-frontend/quickstart.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies.
- **Foundational (Phase 2)**: Depends on Setup and blocks every user story.
- **US1 (Phase 3)**: Depends on Foundational completion and delivers the MVP.
- **US2 (Phase 4)**: Depends on Foundational completion; it integrates with
  US1's shared predicate but can be verified with its own context fixtures.
- **US3 (Phase 5)**: Depends on Foundational completion and can proceed in
  parallel with US2 once shared route evaluation is available.
- **Polish (Phase 6)**: Depends on all selected user stories being implemented.

### User Story Dependencies

- **US1 (P1)**: Starts after Foundational. It is the MVP and establishes global
  protected route, navigation, and action behavior.
- **US2 (P2)**: Starts after Foundational. It relies on the same shared
  authorization decision but preserves independent school and subject context
  boundaries.
- **US3 (P3)**: Starts after Foundational. It preserves feedback and session
  behavior around the shared decision.

### Within Each User Story

- Write the story's Vitest coverage before implementation.
- Use shared authorization and context boundaries rather than local page logic.
- Update quickstart evidence before moving to polish.

### Parallel Opportunities

- Setup tasks T002-T004 can run in parallel after T001 begins.
- Foundational tests T005-T006 can run in parallel.
- US1 tests T010-T013, US2 tests T019-T022, and US3 tests T028-T030 can each
  run in parallel.
- Polish review and verification tasks T034-T037 can run in parallel once all
  user stories are implemented.

---

## Parallel Example: User Story 1

```bash
Task: "T010 [US1] Add direct route-access tests"
Task: "T011 [US1] Add limited-role denial tests"
Task: "T012 [US1] Add navigation visibility tests"
Task: "T013 [US1] Add action visibility tests"
```

## Parallel Example: User Story 2

```bash
Task: "T019 [US2] Add active-school and missing-context tests"
Task: "T020 [US2] Add stale-data context-switch tests"
Task: "T021 [US2] Add subject-context boundary tests"
Task: "T022 [US2] Add platform-wide presentation tests"
```

## Parallel Example: User Story 3

```bash
Task: "T028 [US3] Add non-permission feedback tests"
Task: "T029 [US3] Add client-side audit-boundary tests"
Task: "T030 [US3] Add session-resolution tests"
```

## Implementation Strategy

### MVP First

1. Complete Setup and Foundational phases.
2. Complete US1 route, navigation, and action master access.
3. Validate System Administrator allow behavior and limited-role denial across
   the protected route inventory.
4. Stop for review before expanding school-context and feedback coverage.

### Incremental Delivery

1. US1: shared master-access predicate and released non-identity-owned route,
   navigation, and action visibility.
2. US2: active-school selection, selected-school isolation, stale-data
   cleanup, and retained identity-owned boundaries.
3. US3: non-permission feedback, session-resolution behavior, and no frontend
   audit transport.
4. Polish: exhaustive scans, focused suite, build, and final evidence.

### Parallel Team Strategy

1. One owner inventories routes and updates feature evidence.
2. One owner implements shared auth, route guard, and navigation behavior.
3. One owner implements school and subject context behavior.
4. One owner implements feedback behavior and focused Vitest coverage.

## Notes

- `[P]` tasks touch distinct files or can be completed independently.
- Master access satisfies feature-permission checks only.
- Do not expose identity-owned student or guardian self-service routes outside
  their established actor-owned or guardian-link context.
- Do not add a frontend audit request, endpoint, response field, or OpenAPI
  change for this feature.
