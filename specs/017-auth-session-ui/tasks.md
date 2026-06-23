# Tasks: Authentication and Session Foundation UI

**Input**: Design documents from `specs/017-auth-session-ui/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/auth-session-ui-contract.md, quickstart.md

**Tests**: Frontend Vitest coverage is required because this feature changes critical frontend authentication, session, route guard, tenant context, and recovery behavior. Backend PHPUnit tasks are not included because this is a frontend-only implementation slice. OpenAPI validation is required only if a school-selection source contract gap requires OpenAPI changes.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- Run frontend implementation tasks from the `schoolmaster-frontend` repository root.
- Specification and contract paths reference the sibling `schoolmaster-specs` repository.
- **Frontend repository**: `src/pages/`, `src/components/`, `src/composables/`, `src/contracts/`, `src/layouts/`, `src/locales/`, `src/router/`, `src/services/`, `src/stores/`, `tests/unit/`
- **Specification repository**: `../schoolmaster-specs/specs/017-auth-session-ui/`, `../schoolmaster-specs/api/`

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Create the auth/session frontend module structure and shared implementation locations.

- [ ] T001 Create auth module directories in `src/pages/auth/`, `src/components/auth/`, `src/composables/auth/`, `src/contracts/auth/`, `src/layouts/auth/`, `src/locales/`, `src/router/modules/`, `src/services/auth/`, `src/stores/auth/`, and `tests/unit/auth/`
- [ ] T002 [P] Create auth route placeholder module in `src/router/modules/auth.routes.js`
- [ ] T003 [P] Create auth locale namespace placeholder in `src/locales/auth.js`
- [ ] T004 [P] Create auth contract mapping placeholder in `src/contracts/auth/authSession.contract.js`
- [ ] T005 [P] Create auth test fixture placeholder in `tests/unit/auth/auth.fixtures.js`
- [ ] T006 [P] Create auth service placeholder in `src/services/auth/authService.js`
- [ ] T007 [P] Create auth error mapper placeholder in `src/services/auth/authErrorMapper.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core service, state, route, and contract-review boundaries that MUST be complete before ANY user story can be implemented.

**CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T008 Review approved auth operations `login`, `getCurrentUser`, `logout`, and `requestPasswordReset` against `../schoolmaster-specs/api/openapi.yaml` and record any gaps in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`
- [ ] T009 Verify whether an approved user-authorized school-selection source exists in `../schoolmaster-specs/api/openapi.yaml`; if none exists, record the OpenAPI blocking gap in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md` before enabling school-selection UI
- [ ] T010 Implement shared auth JSDoc typedefs and mapping helpers for AuthSession, CurrentUser, PermissionSet, TenantContext, RequestedRoute, and AuthFeedbackState in `src/contracts/auth/authSession.contract.js`
- [ ] T011 Implement OpenAPI success and error envelope normalization helpers for auth flows in `src/services/auth/authErrorMapper.js`
- [ ] T012 Implement auth API service methods for `login`, `getCurrentUser`, `logout`, and `requestPasswordReset` in `src/services/auth/authService.js`
- [ ] T013 Implement Pinia auth session state skeleton and explicit actions in `src/stores/auth/sessionStore.js`
- [ ] T014 Implement protected route guard helpers for auth, permission, tenant context, requested-route preservation, and fallback routing in `src/router/authGuards.js`
- [ ] T015 Register auth routes and guard integration points without enabling story-specific pages in `src/router/index.js`
- [ ] T016 Add shared auth/session translation keys for form labels, validation, denied states, recovery actions, and neutral confirmations in `src/locales/auth.js`
- [ ] T017 Create shared auth feedback component for validation, invalid credentials, lockout, expired-session, unauthorized, forbidden, inactive-user, inactive-school, tenant-mismatch, unavailable, and neutral-confirmation states in `src/components/auth/AuthFeedbackState.vue`
- [ ] T018 Create unauthenticated auth layout composition surface in `src/layouts/auth/AuthLayout.vue`

**Checkpoint**: Foundation ready - user story implementation can now begin.

---

## Phase 3: User Story 1 - Sign in to the correct workspace (Priority: P1) [MVP]

**Goal**: A signed-out user can sign in with valid credentials, reach the correct authenticated workspace, and see safe feedback for invalid or inactive-account outcomes.

**Independent Test**: Start signed out, submit valid credentials, invalid credentials, and inactive-user responses; verify authenticated state, workspace routing, safe denial messaging, and no protected-content exposure before confirmation.

### Tests for User Story 1

- [ ] T019 [P] [US1] Add Vitest coverage for login service success, validation failure, invalid credentials, lockout, inactive-user mapping, and the 30-second sign-in acceptance target in `tests/unit/auth/authService.login.test.js`
- [ ] T020 [P] [US1] Add Vitest coverage for login store transitions from signed-out to authenticated and denied states in `tests/unit/auth/sessionStore.login.test.js`
- [ ] T021 [P] [US1] Add Vitest coverage for login page form validation and safe denial messaging in `tests/unit/auth/LoginPage.test.js`

### Implementation for User Story 1

- [ ] T022 [P] [US1] Implement login request and AuthSession response mapping in `src/contracts/auth/authSession.contract.js`
- [ ] T023 [US1] Implement `login` service integration with normalized success, validation, unauthorized, lockout, inactive-user, and inactive-school outcomes in `src/services/auth/authService.js`
- [ ] T024 [US1] Implement session store login action, authenticated state hydration, denied-state handling, and stale state clearing in `src/stores/auth/sessionStore.js`
- [ ] T025 [P] [US1] Implement focused login form component with props-down/events-up behavior in `src/components/auth/LoginForm.vue`
- [ ] T026 [US1] Implement login route page as a thin composition surface in `src/pages/auth/LoginPage.vue`
- [ ] T027 [US1] Wire login route metadata and unauthenticated layout selection in `src/router/modules/auth.routes.js`
- [ ] T028 [US1] Integrate post-login routing to the allowed authenticated workspace using the route guard helper in `src/router/authGuards.js`
- [ ] T029 [US1] Add login page i18n keys for labels, actions, validation, invalid credentials, lockout, and inactive-user messages in `src/locales/auth.js`

**Checkpoint**: User Story 1 is independently functional and testable as the MVP.

---

## Phase 4: User Story 2 - Resume an existing session (Priority: P2)

**Goal**: A returning authenticated user can bootstrap current user, permissions, and tenant context before protected content renders, with active school restoration or selection gating when needed.

**Independent Test**: Start with an existing valid session and open protected routes under valid, missing, inactive, mismatched, and multi-school tenant contexts; verify bootstrap blocks content until context is confirmed and school selection is blocked without an approved source contract.

### Tests for User Story 2

- [ ] T030 [P] [US2] Add Vitest coverage for `getCurrentUser` service mapping of user, roles, permissions, resolved school, token rejection, tenant mismatch, and the 30-second valid-session bootstrap acceptance target in `tests/unit/auth/authService.currentUser.test.js`
- [ ] T031 [P] [US2] Add Vitest coverage for session bootstrap state transitions and stale tenant clearing in `tests/unit/auth/sessionStore.bootstrap.test.js`
- [ ] T032 [P] [US2] Add Vitest coverage for protected route guards hiding content until bootstrap completes in `tests/unit/auth/authGuards.bootstrap.test.js`
- [ ] T033 [P] [US2] Add Vitest coverage for active school restoration, school selection gating, and blocked missing school-selection source behavior in `tests/unit/auth/tenantContext.test.js`

### Implementation for User Story 2

- [ ] T034 [P] [US2] Extend AuthSession and TenantContext mapping for `resolved_school`, roles, permissions, and tenant status in `src/contracts/auth/authSession.contract.js`
- [ ] T035 [US2] Implement `getCurrentUser` service support for optional `X-School-Id` tenant context and normalized token rejection or tenant mismatch in `src/services/auth/authService.js`
- [ ] T036 [US2] Implement bootstrap, current-user hydration, permission hydration, active school restoration, and stale tenant clearing actions in `src/stores/auth/sessionStore.js`
- [ ] T037 [US2] Implement derived permission and tenant readiness selectors in `src/stores/auth/sessionStore.js`
- [ ] T038 [US2] Implement route guard behavior for requires-auth, requires-school-context, permission checks, and protected-content blocking in `src/router/authGuards.js`
- [ ] T039 [US2] Wire protected route guard registration for authenticated routes in `src/router/index.js`
- [ ] T040 [P] [US2] Implement school selection page as a guarded composition surface that renders a blocked-source state unless T009 confirms an approved source operation in `src/pages/auth/SchoolSelectionPage.vue`
- [ ] T041 [P] [US2] Implement school selection list component that renders only when T009 confirms an approved source operation and accepts only authorized schools from store state in `src/components/auth/SchoolSelectionList.vue`
- [ ] T042 [US2] Block school selection list population and service consumption unless the approved source operation from `../schoolmaster-specs/specs/017-auth-session-ui/contracts/auth-session-ui-contract.md` is confirmed in `src/services/auth/authService.js`
- [ ] T043 [US2] Add school selection, bootstrap, permission, and tenant mismatch translation keys in `src/locales/auth.js`
- [ ] T044 [US2] Integrate authenticated layout selection with the completed admin shell target in `src/router/modules/auth.routes.js`

**Checkpoint**: User Stories 1 and 2 work independently; protected content remains hidden until session and tenant context are confirmed.

---

## Phase 5: User Story 3 - Recover from session and authorization failures (Priority: P3)

**Goal**: Users recover predictably from expired-session, unauthorized, forbidden, inactive-user, inactive-school, and tenant-mismatch states, preserving originally requested protected routes only when still authorized.

**Independent Test**: Force expired session, signed-out direct access, forbidden, inactive-user, inactive-school, and tenant-mismatch outcomes; verify recovery messages, requested-route preservation, authorized return, and fallback to an allowed workspace.

### Tests for User Story 3

- [ ] T045 [P] [US3] Add Vitest coverage for expired-session and token-rejected service error mapping in `tests/unit/auth/authErrorMapper.session.test.js`
- [ ] T046 [P] [US3] Add Vitest coverage for requested-route preservation and authorized restoration in `tests/unit/auth/authGuards.requestedRoute.test.js`
- [ ] T047 [P] [US3] Add Vitest coverage for forbidden, unauthorized, inactive-user, inactive-school, and tenant-mismatch feedback state rendering in `tests/unit/auth/AuthFeedbackState.test.js`
- [ ] T048 [P] [US3] Add Vitest coverage for fallback routing to an allowed workspace when the originally requested route is no longer authorized in `tests/unit/auth/sessionRecovery.test.js`

### Implementation for User Story 3

- [ ] T049 [US3] Implement requested-route capture, validation, restoration, and clearing logic in `src/router/authGuards.js`
- [ ] T050 [US3] Implement expired-session, unauthorized, forbidden, inactive-user, inactive-school, tenant-mismatch, and unavailable state transitions in `src/stores/auth/sessionStore.js`
- [ ] T051 [US3] Extend error envelope mapping for token_expired, token_revoked, forbidden, tenant_mismatch, inactive_user, inactive_school, and unavailable outcomes in `src/services/auth/authErrorMapper.js`
- [ ] T052 [P] [US3] Implement session recovery action UI in `src/components/auth/AuthRecoveryActions.vue`
- [ ] T053 [US3] Integrate recovery actions into shared auth feedback rendering in `src/components/auth/AuthFeedbackState.vue`
- [ ] T054 [US3] Implement direct protected-route access handling from signed-out state in `src/router/authGuards.js`
- [ ] T055 [US3] Implement fallback routing to an allowed authenticated workspace when requested route restoration fails in `src/router/authGuards.js`
- [ ] T056 [US3] Add recovery and denied-state translation keys in `src/locales/auth.js`

**Checkpoint**: User Stories 1, 2, and 3 work independently and preserve protected-route intent only after authorization is confirmed.

---

## Phase 6: User Story 4 - Start password recovery (Priority: P4)

**Goal**: A signed-out user can start the approved forgot-password entry flow and receive neutral confirmation without account enumeration.

**Independent Test**: Submit syntactically valid and invalid email addresses; verify field-level validation for invalid input and identical neutral confirmation for accepted submissions regardless of account state.

### Tests for User Story 4

- [ ] T057 [P] [US4] Add Vitest coverage for `requestPasswordReset` service success, validation, neutral accepted mapping, and the 15-second neutral confirmation acceptance target in `tests/unit/auth/authService.passwordResetRequest.test.js`
- [ ] T058 [P] [US4] Add Vitest coverage for forgot-password page validation and neutral confirmation behavior in `tests/unit/auth/ForgotPasswordPage.test.js`
- [ ] T059 [P] [US4] Add Vitest coverage that password reset request messaging does not reveal account existence or account state in `tests/unit/auth/passwordRecoveryPrivacy.test.js`

### Implementation for User Story 4

- [ ] T060 [P] [US4] Implement password reset request contract mapping for email, optional school_id, accepted result, and validation errors in `src/contracts/auth/authSession.contract.js`
- [ ] T061 [US4] Implement `requestPasswordReset` service integration with neutral accepted confirmation and validation handling in `src/services/auth/authService.js`
- [ ] T062 [US4] Implement password recovery request action state in `src/stores/auth/sessionStore.js`
- [ ] T063 [P] [US4] Implement focused forgot-password form component with field validation and submit events in `src/components/auth/ForgotPasswordForm.vue`
- [ ] T064 [US4] Implement forgot-password route page as a thin composition surface in `src/pages/auth/ForgotPasswordPage.vue`
- [ ] T065 [US4] Wire forgot-password route metadata and unauthenticated layout selection in `src/router/modules/auth.routes.js`
- [ ] T066 [US4] Add forgot-password labels, validation, submitted-state, and neutral confirmation translation keys in `src/locales/auth.js`

**Checkpoint**: All user stories are independently functional and testable.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Verification, documentation, and cross-story hardening.

- [ ] T067 [P] Run auth/session quickstart review commands, including timing acceptance checks for 30-second sign-in/bootstrap, 15-second password reset confirmation, and SC-006 session-recovery UAT targets, and record results in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`
- [ ] T068 [P] Verify no direct Axios usage exists in auth pages, components, layouts, or router guards and record findings in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`
- [ ] T069 [P] Verify Element Plus component tags use PascalCase in auth Vue files and record findings in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`
- [ ] T070 [P] Review local or session storage usage for token, password, Authorization, and tenant metadata handling and record findings in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`
- [ ] T071 [P] Verify all auth/session reusable text uses Vue I18n keys instead of hardcoded shared messages in `src/locales/auth.js`
- [ ] T072 [P] Run auth/session Vitest tests, verify timing acceptance coverage is included, and record command output in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`
- [ ] T073 Update frontend roadmap implementation status and any discovered school-selection contract follow-up in `../schoolmaster-specs/docs/frontend-feature-roadmap.md`
- [ ] T074 If OpenAPI changed for school selection, run Redocly validation and record result in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`
- [ ] T075 Review feature against `../schoolmaster-specs/specs/017-auth-session-ui/contracts/auth-session-ui-contract.md` and record deviations or confirmations in `../schoolmaster-specs/specs/017-auth-session-ui/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately.
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories.
- **User Stories (Phase 3+)**: All depend on Foundational phase completion.
- **Polish (Phase 7)**: Depends on all selected user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational - MVP and no dependency on later stories.
- **User Story 2 (P2)**: Can start after Foundational but integrates naturally after US1 because it reuses authenticated session state.
- **User Story 3 (P3)**: Can start after Foundational but depends on route guard and session store foundations used by US1 and US2.
- **User Story 4 (P4)**: Can start after Foundational and can be implemented in parallel with US2/US3 because it uses a separate unauthenticated recovery entry flow.

### Within Each User Story

- Vitest tasks should be written first and fail before implementation.
- Contract mapping comes before service integration.
- Services come before stores and route/page integration.
- Route guards and stores come before protected page behavior.
- Components use props-down/events-up and keep route pages as composition surfaces.
- Story complete before moving to the next priority unless staffing supports parallel delivery.

### Parallel Opportunities

- Setup placeholders T002-T007 can run in parallel.
- User Story test tasks marked [P] can run in parallel before implementation.
- Story-specific focused components marked [P] can run in parallel with contract mapping after shared foundations exist.
- US4 can run in parallel with US2/US3 after Foundational because forgot-password entry is unauthenticated and independent.
- Polish review tasks T067-T072 can run in parallel after implementation.

---

## Parallel Example: User Story 1

```bash
Task: "T019 [P] [US1] Add Vitest coverage for login service success, validation failure, invalid credentials, lockout, inactive-user mapping, and the 30-second sign-in acceptance target in tests/unit/auth/authService.login.test.js"
Task: "T020 [P] [US1] Add Vitest coverage for login store transitions from signed-out to authenticated and denied states in tests/unit/auth/sessionStore.login.test.js"
Task: "T021 [P] [US1] Add Vitest coverage for login page form validation and safe denial messaging in tests/unit/auth/LoginPage.test.js"
Task: "T025 [P] [US1] Implement focused login form component with props-down/events-up behavior in src/components/auth/LoginForm.vue"
```

## Parallel Example: User Story 2

```bash
Task: "T030 [P] [US2] Add Vitest coverage for getCurrentUser service mapping of user, roles, permissions, resolved school, token rejection, tenant mismatch, and the 30-second valid-session bootstrap acceptance target in tests/unit/auth/authService.currentUser.test.js"
Task: "T031 [P] [US2] Add Vitest coverage for session bootstrap state transitions and stale tenant clearing in tests/unit/auth/sessionStore.bootstrap.test.js"
Task: "T032 [P] [US2] Add Vitest coverage for protected route guards hiding content until bootstrap completes in tests/unit/auth/authGuards.bootstrap.test.js"
Task: "T033 [P] [US2] Add Vitest coverage for active school restoration, school selection gating, and blocked missing school-selection source behavior in tests/unit/auth/tenantContext.test.js"
```

## Parallel Example: User Story 3

```bash
Task: "T045 [P] [US3] Add Vitest coverage for expired-session and token-rejected service error mapping in tests/unit/auth/authErrorMapper.session.test.js"
Task: "T046 [P] [US3] Add Vitest coverage for requested-route preservation and authorized restoration in tests/unit/auth/authGuards.requestedRoute.test.js"
Task: "T047 [P] [US3] Add Vitest coverage for forbidden, unauthorized, inactive-user, inactive-school, and tenant-mismatch feedback state rendering in tests/unit/auth/AuthFeedbackState.test.js"
Task: "T052 [P] [US3] Implement session recovery action UI in src/components/auth/AuthRecoveryActions.vue"
```

## Parallel Example: User Story 4

```bash
Task: "T057 [P] [US4] Add Vitest coverage for requestPasswordReset service success, validation, neutral accepted mapping, and the 15-second neutral confirmation acceptance target in tests/unit/auth/authService.passwordResetRequest.test.js"
Task: "T058 [P] [US4] Add Vitest coverage for forgot-password page validation and neutral confirmation behavior in tests/unit/auth/ForgotPasswordPage.test.js"
Task: "T059 [P] [US4] Add Vitest coverage that password reset request messaging does not reveal account existence or account state in tests/unit/auth/passwordRecoveryPrivacy.test.js"
Task: "T063 [P] [US4] Implement focused forgot-password form component with field validation and submit events in src/components/auth/ForgotPasswordForm.vue"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. Stop and validate login independently with service, store, and page tests.
5. Demo signed-out, valid login, invalid login, lockout, and inactive-user outcomes.

### Incremental Delivery

1. Deliver Setup and Foundational service/store/router boundaries.
2. Deliver US1 login as the MVP.
3. Deliver US2 session bootstrap, permission hydration, and tenant context restoration or selection gating.
4. Deliver US3 recovery from expired, unauthorized, forbidden, inactive-user, inactive-school, and tenant-mismatch states.
5. Deliver US4 forgot-password request entry with neutral confirmation.
6. Run Polish verification and update implementation notes.

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup and Foundational tasks together.
2. Developer A implements US1 login.
3. Developer B implements US2 bootstrap and tenant context.
4. Developer C implements US4 forgot-password entry.
5. Developer A or B implements US3 route recovery after guard foundations stabilize.

---

## Notes

- [P] tasks = different files, no dependencies on incomplete tasks.
- [Story] label maps task to a specific user story for traceability.
- Each story has independent Vitest criteria and can be validated separately.
- Keep route-level pages thin; move form UI into focused components and stateful logic into stores/composables.
- Use Composition API with `<script setup>` and JavaScript per the project plan.
- Do not add direct Axios calls in pages, components, layouts, or route guards.
- Do not implement school selection from undocumented or ambiguously scoped school lists.
- Do not add backend implementation tasks unless OpenAPI review creates a separate backend feature.
