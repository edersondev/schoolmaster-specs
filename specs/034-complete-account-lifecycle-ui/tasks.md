# Tasks: Complete Admin Account Lifecycle UI

**Input**: Design documents from `/specs/034-complete-account-lifecycle-ui/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/complete-account-lifecycle-ui-contract.md`, `quickstart.md`

**Tests**: Mandatory. Each implementation phase starts with failing contract, service, composable, component, feature, or browser tests. Keep mocked SPA browser evidence distinct from live backend authorization evidence.

**Organization**: Tasks are grouped by user story. Paths prefixed with `schoolmaster-backend/` and `schoolmaster-frontend/` are relative to their sibling repository roots; all other paths are relative to this specifications repository.

## Format: `- [ ] T### [P?] [US?] Description with file path`

- **[P]**: Can run in parallel because it changes different files and has no unfinished dependency
- **[US1]–[US4]**: Maps the task to the user story in `spec.md`
- Every task names the exact file or directory it changes or validates

## Phase 1: Setup and Contract Source of Truth

**Purpose**: Establish synchronized requirements and API contracts before implementation work begins.

- [X] T001 Record the starting branch, worktree state, sibling-repository revisions, and any unrelated changes that must be preserved in `specs/034-complete-account-lifecycle-ui/quickstart.md`
- [X] T002 [P] Update the invitation-ready account creation rules, activation ownership, and non-enumeration requirements in `specs/008-account-lifecycle-workflows/spec.md`, `specs/008-account-lifecycle-workflows/research.md`, `specs/008-account-lifecycle-workflows/data-model.md`, and `specs/008-account-lifecycle-workflows/contracts/backend-account-lifecycle.md`
- [X] T003 [P] Mark the hardcoded lifecycle gate as superseded and document exact scoped authority, master access, hidden denial, and resend exclusion in `specs/021-account-lifecycle-ui/spec.md`, `specs/021-account-lifecycle-ui/research.md`, `specs/021-account-lifecycle-ui/data-model.md`, and `specs/021-account-lifecycle-ui/contracts/account-lifecycle-ui-contract.md`
- [X] T004 [P] Add `account_setup_mode` and the dedicated `active|inactive|invited` user-status schema in `api/components/schemas/users/UserCreateRequest.yaml`, `api/components/schemas/users/UserStatus.yaml`, and `api/components/schemas/users/User.yaml`
- [X] T005 Document invitation-ready user creation, invitation creation eligibility, setup-only invited-to-active completion, platform-only versus exact-school `listUsers`/`getUser` lookup, scoped lifecycle authority, school-header rules, and non-enumerating denial in `api/paths/users/index.yaml`, `api/paths/users/user.yaml`, `api/paths/users/account-lock.yaml`, `api/paths/users/account-reactivation.yaml`, `api/paths/account-lifecycle/invitations.yaml`, and `api/paths/account-lifecycle/invitations-setup.yaml`
- [X] T006 Synchronize the modular contract changes into `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T007 Validate the modular and aggregate contracts with `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` and record the result in `specs/034-complete-account-lifecycle-ui/quickstart.md`

---

## Phase 2: Foundational Permission and Tenant Enforcement

**Purpose**: Deliver and verify the shared backend Policy, tenant-safe lookup, and frontend authority primitives that block all user-story implementation.

**Critical**: No user-story implementation starts until these tests pass and the exact permission scope is available end to end.

### Tests first

- [X] T008 [P] Add failing configured-MySQL migration and seeder tests proving both scoped `account_lifecycle.manage` rows coexist, a second seeder run is idempotent, duplicate `(code, scope)` is rejected, and rollback refuses or reconciles duplicate codes in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecyclePermissionProvisioningTest.php`
- [X] T009 [P] Add failing Policy and authorization tests proving active exact-scope lifecycle permission, System Administrator master access, self-target denial, tenant-before-target ordering, and non-enumerating school denial plus `listUsers` and `getUser` matrices covering school mode with school `users.view`, platform mode with platform `schools.view` or master access, opposite-mode exclusion, missing authority, lifecycle-section separation, identical unknown/opposite-mode outcomes, and no fallback in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleAuthorizationTest.php`, `schoolmaster-backend/tests/Feature/Api/V1/UserManagementTest.php`, and `schoolmaster-backend/tests/Feature/Api/V1/AdministrationLifecycle/UserDetailUpdateTest.php`
- [X] T010 [P] Replace the blocked-gate expectations with failing exact-code, active-state, scope, master-role, target, self, and lifecycle-state cases in `schoolmaster-frontend/tests/unit/account-lifecycle/contracts/adminAccountLifecycle.contract.test.js`
- [X] T011 [P] Add failing session tests that preserve raw permission scope and exact role identity without treating flattened permission codes as lifecycle authority in `schoolmaster-frontend/tests/unit/auth/sessionStore.bootstrap.test.js`

### Implementation

- [X] T012 Change permission uniqueness from code-only to `(code, scope)` in a forward-only migration whose rollback explicitly refuses or safely reconciles duplicate scoped codes before restoring code-only uniqueness in `schoolmaster-backend/database/migrations/2026_08_11_000001_make_permission_codes_scope_unique.php`
- [X] T013 Seed active platform- and school-scoped `account_lifecycle.manage` permissions by their natural composite key in `schoolmaster-backend/database/seeders/PermissionSeeder.php`
- [X] T014 Make user permission evaluation scope-aware and active-state-aware without changing unrelated permission behavior in `schoolmaster-backend/app/Models/User.php`
- [X] T015 Enforce active scoped permission or exact System Administrator master authority plus self-target and resolved-tenant prerequisites in `schoolmaster-backend/app/Policies/AccountLifecyclePolicy.php`
- [X] T016 Invoke `AccountLifecyclePolicy` before lifecycle transition rules or protected reads; make `AdministrationLifecyclePolicy`, `UserService` list queries, and the detail service authorize and query `listUsers`/`getUser` only inside the preselected platform-only or exact-school mode; preserve non-enumerating outcomes and no fallback in `schoolmaster-backend/app/Policies/AdministrationLifecyclePolicy.php`, `schoolmaster-backend/app/Services/Users/UserService.php`, `schoolmaster-backend/app/Services/AdministrationLifecycle/AdministrationDetailService.php`, `schoolmaster-backend/app/Services/AdministrationLifecycle/AdministrationResourceRegistry.php`, `schoolmaster-backend/app/Services/AccountLifecycle/AccountLockService.php`, `schoolmaster-backend/app/Services/AccountLifecycle/AccountRecoveryService.php`, `schoolmaster-backend/app/Services/AdministrationLifecycle/AdministrationUpdateService.php`, and `schoolmaster-backend/app/Repositories/AccountLifecycleRepository.php`
- [X] T017 Replace the hardcoded lifecycle gate with a pure scope-aware eligibility derivation and action-state table in `schoolmaster-frontend/src/contracts/admin-system/account-lifecycle.js`
- [X] T018 Preserve raw scoped permissions and expose exact active System Administrator role state for lifecycle derivation in `schoolmaster-frontend/src/stores/auth/sessionStore.js`
- [X] T019 Run T008 against the configured MySQL test database and run focused backend Policy, self-target, tenant-ordering, and FR-027/SC-012 `listUsers`/`getUser` lookup-mode tests plus frontend contract/session tests; record connection type, exact commands, and results in `specs/034-complete-account-lifecycle-ui/quickstart.md`

**Checkpoint**: MySQL proves both permission scopes and guarded rollback; `AccountLifecyclePolicy`, `AdministrationLifecyclePolicy`, platform-only/exact-school list/detail lookup, and tenant-scoped repositories pass before UI activation; self and unresolved targets remain ineligible; frontend authority uses the same active scope rules.

---

## Phase 3: User Story 1 — Manage an Eligible Account (Priority: P1) 🎯 MVP

**Goal**: Authorized administrators can lock, unlock, recover, and reactivate eligible users, with refreshed safe state and no obsolete actions.

**Independent Test**: With an active same-school administrator and eligible target, perform every state transition and verify the visible action set, target status, and safe lock metadata refresh after each result.

### Tests first

- [X] T020 [P] [US1] Expand backend success, conflict, validation, and safe-response coverage for lock, unlock, recover, and reactivate transitions in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLockRecoveryTest.php` and `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountReactivationTest.php`
- [X] T021 [P] [US1] Add exact method, path, body, scoped header, abort signal, response mapping, and safe-error tests for every lifecycle request in `schoolmaster-frontend/tests/unit/account-lifecycle/services/adminAccountLifecycle.service.test.js`
- [X] T022 [P] [US1] Replace blocked outcomes with failing success-refresh, conflict-refresh, pending-deduplication, stale-response, and safe-error cases in `schoolmaster-frontend/tests/unit/account-lifecycle/composables/useAccountLifecycleActions.outcomes.test.js`
- [X] T023 [P] [US1] Add action visibility and post-refresh transition tests for active/unlocked, locked, and inactive targets in `schoolmaster-frontend/tests/unit/account-lifecycle/components/AccountLifecycleActions.test.js`
- [X] T024 [P] [US1] Add safe lock-state loading, empty, error, hidden, and metadata rendering tests in `schoolmaster-frontend/tests/unit/account-lifecycle/components/AccountLockPanel.test.js`
- [X] T025 [P] [US1] Expand dialog tests for reason length, optionality, labels, pending deduplication, reset, cancel, and accessible error semantics in `schoolmaster-frontend/tests/unit/account-lifecycle/components/AdminAccountLifecycleDialog.test.js`
- [X] T026 [US1] Replace source-text assertions with mounted authorized action, lock-state, and refresh behavior in `schoolmaster-frontend/tests/unit/account-lifecycle/pages/UserDetailAccountLifecycle.test.js`

### Implementation

- [X] T027 [US1] Implement abortable lock-state, lock, unlock, recover, and reactivate service calls with exact payload/header rules and normalized secret-safe errors in `schoolmaster-frontend/src/services/admin-system/accountLifecycle.js`
- [X] T028 [US1] Implement pending deduplication, generation-based stale guards, safe feedback, and target-plus-lock refresh after success or conflict in `schoolmaster-frontend/src/composables/admin-system/useAccountLifecycleActions.js`
- [X] T029 [P] [US1] Render only eligible current actions and emit intent/result events without service logic in `schoolmaster-frontend/src/components/admin-system/users/AccountLifecycleActions.vue`
- [X] T030 [P] [US1] Render only approved safe lock metadata with accessible loading, empty, status, and error states in `schoolmaster-frontend/src/components/admin-system/users/AccountLockPanel.vue`
- [X] T031 [US1] Implement accessible lock, unlock, recover, and reactivate dialog validation and pending behavior in `schoolmaster-frontend/src/components/ui/admin/AdminAccountLifecycleDialog.vue`
- [X] T032 [US1] Integrate authorized lifecycle panels and refreshed target state into the user detail page in `schoolmaster-frontend/src/pages/admin-system/users/UserDetailPage.vue`
- [X] T033 [P] [US1] Add lifecycle action, validation, status, and safe-error copy in `schoolmaster-frontend/src/locales/account-lifecycle.js`
- [X] T034 [US1] Run the focused backend lifecycle and frontend US1 unit suites and record results in `specs/034-complete-account-lifecycle-ui/quickstart.md`

**Checkpoint**: US1 works for an authorized eligible target only after the Phase 2 backend Policy and tenant-security checkpoint passes; production release still requires the complete client and browser denial matrix in US3.

---

## Phase 4: User Story 2 — Create and Explicitly Invite a User (Priority: P1)

**Goal**: Create an invitation-ready persisted user first, then let the administrator explicitly create an invitation without re-entering target details or losing created-user truth on failure.

**Independent Test**: Submit a valid invitation-mode user, verify exactly one `invited` user is persisted with no invitation delivery, then explicitly create the invitation for that returned user and retry safely after an invitation failure.

### Tests first

- [X] T035 [P] [US2] Add failing request and feature cases for default active creation, explicit invitation mode, exactly one invited user, no automatic invitation, duplicate handling, and safe output in `schoolmaster-backend/tests/Feature/Api/V1/UserManagementTest.php`
- [X] T036 [P] [US2] Add failing transition tests proving generic user updates cannot activate invited users and invitation completion is the sole activation owner in `schoolmaster-backend/tests/Feature/AccountLifecycle/InvitedUserActivationTest.php`
- [X] T037 [P] [US2] Expand invitation service tests for persisted-target mapping, exact role/scope fields, school headers, abort signals, omission of request `delivery_metadata`, and safe mapping of documented invitation response fields without token or secret fields while preserving documented identifiers as service data in `schoolmaster-frontend/tests/unit/account-lifecycle/services/createAccountInvitation.test.js`
- [X] T038 [P] [US2] Add failing composable tests for persisted-user identity, eligibility invalidation, pending deduplication, retry, and stale actor/school/permission responses in `schoolmaster-frontend/tests/unit/account-lifecycle/composables/useAccountInvitation.test.js`
- [X] T039 [P] [US2] Replace blocked panel expectations with authorized explicit create, result rendering limited to `status`, `expires_at`, `delivery_channel`, and `delivery_requested_at`, retry, hidden denial, and no-resend cases in `schoolmaster-frontend/tests/unit/account-lifecycle/components/UserInvitationPanel.test.js`
- [X] T040 [US2] Add mounted create-flow tests for draft failure, persisted success, no premature navigation, explicit invite, retry against the same target, duplicate normalization, eligibility invalidation, and navigation/reload recovery that accepts only UUID route intent, performs an authorized tenant-scoped re-fetch, restores the same invited user, and never reuses draft or route-supplied user details in `schoolmaster-frontend/tests/unit/account-lifecycle/pages/CreateUserAccountInvitation.test.js`

### Implementation

- [X] T041 [US2] Validate and normalize `account_setup_mode` while keeping active creation as the backward-compatible default in `schoolmaster-backend/app/Http/Requests/Api/V1/CreateUserRequest.php` and `schoolmaster-backend/app/DTOs/Users/CreateUserData.php`
- [X] T042 [US2] Persist invitation-mode users exactly once with `invited` status and without invitation or delivery side effects in `schoolmaster-backend/app/Services/Users/UserService.php`
- [X] T043 [US2] Prevent generic update paths from activating invited users while preserving invitation-completion activation in `schoolmaster-backend/app/Services/AdministrationLifecycle/AdministrationUpdateService.php`
- [X] T044 [US2] Map invitation-mode user creation and retain the returned persisted user in `schoolmaster-frontend/src/contracts/admin-system/users.js` and `schoolmaster-frontend/src/services/admin-system/users.js`
- [X] T045 [US2] Implement explicit invitation orchestration, stale-context invalidation, retry, and pending deduplication in `schoolmaster-frontend/src/composables/admin-system/useAccountInvitation.js`
- [X] T046 [US2] Render the invitation section only when authorized and emit explicit creation intent plus safe result events in `schoolmaster-frontend/src/components/admin-system/users/UserInvitationPanel.vue`
- [X] T047 [US2] Keep the create page in a persisted-user success phase until the administrator invites or finishes; persist only the created-user UUID as route intent and, on navigation or reload, restore the phase only after an authorized exact-tenant re-fetch confirms the same invited user, without resubmitting or rebuilding from the draft, in `schoolmaster-frontend/src/pages/admin-system/users/CreateUserPage.vue` and `schoolmaster-frontend/src/router/modules/access-administration.routes.js`
- [X] T048 [US2] Run the focused backend user-management and frontend invitation/create-flow suites and record results in `specs/034-complete-account-lifecycle-ui/quickstart.md`

**Checkpoint**: User persistence and invitation creation are two explicit, retry-safe operations; failed invitation delivery never erases or duplicates the created user.

---

## Phase 5: User Story 3 — Preserve Authorization and Tenant Isolation (Priority: P2)

**Goal**: Complete and prove the school/platform authorization matrix, hidden/no-request denial, non-enumeration, and stale-context protection.

**Independent Test**: Exercise school, platform, System Administrator, missing/inactive/mismatched school, unrelated permission, self, soft-deleted, and cross-tenant cases; allow only exact eligible cases and observe zero lifecycle requests for every client-side denial.

### Tests first

- [X] T049 [P] [US3] Add the complete backend permission-scope, tenant-context, platform-target, System Administrator, self-target, soft-delete, and non-enumeration matrix in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleTenantIsolationTest.php`
- [X] T050 [P] [US3] Rewrite lifecycle permission tests for same-school, platform, master, missing/inactive/mismatched school, unrelated permission, self, and cross-school targets with zero-call assertions in `schoolmaster-frontend/tests/unit/account-lifecycle/composables/useAccountLifecycleActions.permissions.test.js`
- [X] T051 [P] [US3] Expand stale-response tests across actor identity, raw permission scope, target, and active-school changes, including suppression of old-context follow-up refreshes, in `schoolmaster-frontend/tests/unit/account-lifecycle/composables/useAccountLifecycleActions.outcomes.test.js`
- [X] T052 [P] [US3] Add mounted hidden-section and zero-request cases for unauthorized, missing-context, cross-tenant, master, and self targets plus explicit school/platform lookup mode and no-fallback behavior in `schoolmaster-frontend/tests/unit/account-lifecycle/pages/UserDetailAccountLifecycle.test.js`
- [X] T053 [US3] Add a stateful mocked API authorization ledger covering `listUsers`/`getUser` plus lifecycle requests in explicit school/platform lookup modes, master access, denial, no cross-mode fallback, errors, stale responses, exact-header scenarios, and create-flow reload recovery from UUID-only route intent with exact-tenant re-fetch and zero duplicate user creation in `schoolmaster-frontend/e2e/account-lifecycle.spec.js`

### Implementation

- [X] T054 [US3] Close regression failures while preserving Phase 2 Policy-before-service ordering and normalize forbidden, tenant-mismatch, self, and missing-target outcomes without existence disclosure in `schoolmaster-backend/app/Policies/AccountLifecyclePolicy.php`, `schoolmaster-backend/app/Services/AccountLifecycle/AccountLockService.php`, `schoolmaster-backend/app/Services/AccountLifecycle/AccountRecoveryService.php`, and `schoolmaster-backend/app/Services/AdministrationLifecycle/AdministrationUpdateService.php`
- [X] T055 [US3] Select one user-route lookup mode from validated route/list intent, otherwise active school, otherwise platform authority, and fail closed when none resolves in `schoolmaster-frontend/src/router/modules/access-administration.routes.js`
- [X] T056 [P] [US3] Resolve detail targets in exactly one mode—exact active-school header or authorized platform-only request with no header—and never retry in the other mode in `schoolmaster-frontend/src/composables/admin-system/useAdminDetail.js`
- [X] T057 [P] [US3] Resolve the user list in platform-only or exact-school mode and carry that validated mode into user-detail navigation without broadening other administration resources in `schoolmaster-frontend/src/composables/admin-system/useAdministrationResourceList.js`
- [X] T058 [US3] Hide lifecycle action, lock, and invitation sections by unmounting them before their composables can issue requests in `schoolmaster-frontend/src/pages/admin-system/users/UserDetailPage.vue` and `schoolmaster-frontend/src/pages/admin-system/users/CreateUserPage.vue`
- [X] T059 [US3] Invalidate all in-flight lock, action, refresh, and invitation results when actor, permission scope, target, or active school changes in `schoolmaster-frontend/src/composables/admin-system/useAccountLifecycleActions.js` and `schoolmaster-frontend/src/composables/admin-system/useAccountInvitation.js`
- [X] T060 [US3] Omit `X-School-Id` for platform targets and require the exact target school header for school-owned lifecycle and invitation requests in `schoolmaster-frontend/src/services/admin-system/accountLifecycle.js`
- [X] T061 [US3] Normalize 403, 404, 409, 422, tenant-context, inactive-school, and temporary errors into approved feedback without raw backend diagnostics in `schoolmaster-frontend/src/contracts/admin-system/account-lifecycle.js` and `schoolmaster-frontend/src/services/admin-system/administration-error-mapper.js`
- [X] T062 [US3] Run the full backend authorization and FR-027/SC-012 list/detail matrix, focused frontend permission/page suites, and focused Chromium browser journey, then record separate backend and mocked-SPA evidence in `specs/034-complete-account-lifecycle-ui/quickstart.md`

**Checkpoint**: Authority is exact and scope-aware on both sides; every denied UI case is hidden and produces zero lifecycle calls; server denials do not disclose target existence.

---

## Phase 6: User Story 4 — Keep Secret-Dependent Resend Unavailable (Priority: P3)

**Goal**: Preserve invitation visibility and explicit creation while exposing no resend action until a non-secret backend contract exists.

**Independent Test**: Inspect eligible invitation UI, invoke every available control, and verify no resend control, token-bearing path, secret value, request `delivery_metadata`, or undocumented delivery diagnostic appears in requests, DOM, storage, or feedback while the four approved invitation-result fields remain usable.

### Tests first

- [X] T063 [P] [US4] Add contract and component assertions that resend and request `delivery_metadata` remain absent while invitation creation and administrator-visible `status`, `expires_at`, `delivery_channel`, and `delivery_requested_at` remain available in `schoolmaster-frontend/tests/unit/account-lifecycle/contracts/adminAccountLifecycle.contract.test.js` and `schoolmaster-frontend/tests/unit/account-lifecycle/components/UserInvitationPanel.test.js`
- [X] T064 [US4] Add browser assertions for no resend control/request and no token, submitted plaintext reason, private permission, tenant-private data, request `delivery_metadata`, or undocumented or secret-derived delivery diagnostics in post-submit feedback, diagnostics, or browser storage while allowing the active reason input, documented outgoing action body, and administrator-visible `status`, `expires_at`, `delivery_channel`, and `delivery_requested_at` in `schoolmaster-frontend/e2e/account-lifecycle.spec.js`

### Implementation

- [X] T065 [US4] Preserve the documented resend capability as unavailable and keep secret-bearing paths out of exported lifecycle actions in `schoolmaster-frontend/src/contracts/admin-system/account-lifecycle.js`
- [X] T066 [US4] Remove or suppress all admin resend controls and undocumented or secret-derived delivery diagnostics while retaining only `status`, `expires_at`, `delivery_channel`, and `delivery_requested_at` in `schoolmaster-frontend/src/components/admin-system/users/UserInvitationPanel.vue`
- [X] T067 [US4] Audit account-lifecycle source and tests for token-path resend calls or secret persistence and record findings in `specs/034-complete-account-lifecycle-ui/quickstart.md`

**Checkpoint**: Admin invitation creation is usable, but resend remains absent and no secret-dependent API behavior is implied or exercised.

---

## Phase 7: Documentation, Regression, and Delivery Evidence

**Purpose**: Reconcile superseded Feature 021 evidence, run proportional cross-repository validation, and leave honest reproducible delivery records.

- [X] T068 [P] Update the Feature 021 permission/capability table, closing notes, obsolete blocked tasks, and correct frontend test commands in `specs/021-account-lifecycle-ui/plan.md` and `specs/021-account-lifecycle-ui/tasks.md`
- [X] T069 [P] Add Feature 034 completion links and dated supersession notes without claiming pending human UAT in `specs/021-account-lifecycle-ui/quickstart.md`
- [X] T070 Run `vendor/bin/pint --dirty --format agent`, the configured-MySQL permission migration/seeder test, focused account-lifecycle and user-detail lookup tests, `UserManagementTest`, and the full backend suite; record database connection type, exact results, and revision in `specs/034-complete-account-lifecycle-ui/quickstart.md`
- [X] T071 Run `npm run test:unit -- --run tests/unit/account-lifecycle tests/unit/auth/AuthFeedbackState.test.js`, the full unit suite, and `npm run build`; record exact results and revision in `specs/034-complete-account-lifecycle-ui/quickstart.md`
- [X] T072 Run focused mocked Playwright evidence on Chromium, Firefox, and WebKit, then the full browser regression; label it SPA orchestration rather than live backend policy evidence in `specs/034-complete-account-lifecycle-ui/quickstart.md`
- [X] T073 Verify FR-026 and SC-011 for create, detail, invitation result/error, lock panel, and every dialog at 390, 768, and 1440 pixels across Chromium, Firefox, and WebKit, covering zero overflow, named controls, live feedback, keyboard reachability, focus containment/return, Escape/cancel, and pending controls in `schoolmaster-frontend/e2e/account-lifecycle.spec.js`
- [X] T074 Audit post-submit feedback, diagnostics, local/session storage, fixtures, and logs for tokens, submitted plaintext lock/recovery reasons, request `delivery_metadata`, undocumented delivery diagnostics, tenant-private values, and raw errors while excluding the active reason input, documented outgoing action body, and four approved invitation-result fields; record the outcome in `specs/034-complete-account-lifecycle-ui/quickstart.md`
- [X] T075 Re-run Redocly lint and compare Feature 008, Feature 021, Feature 034, modular OpenAPI, aggregate OpenAPI, backend behavior, and frontend request shapes for drift in `specs/034-complete-account-lifecycle-ui/contracts/complete-account-lifecycle-ui-contract.md`
- [X] T076 Record final commands, test counts, browser matrix, commit hashes, known limitations, separate mocked/live evidence, and genuinely pending five-administrator UAT in `specs/034-complete-account-lifecycle-ui/quickstart.md`
- [X] T077 Perform a final cross-repository diff review for unrelated changes, unsafe diagnostics, undocumented APIs, and incomplete task evidence, then capture the completion decision in `specs/034-complete-account-lifecycle-ui/quickstart.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Phase 1 — Contract Source of Truth**: Starts immediately and must finish before backend or frontend behavior changes.
- **Phase 2 — Foundation**: Depends on Phase 1; backend Policy, self-target, tenant-before-target, and lookup-mode tests must pass and block all user stories.
- **US1 — Manage Account**: Depends on the complete Phase 2 security checkpoint; provides the first independently demonstrable action flow.
- **US2 — Create and Invite**: Depends on the complete Phase 2 security checkpoint and may proceed alongside US1 when shared frontend contract edits are coordinated.
- **US3 — Authorization and Isolation**: Depends on Phase 2's already-enforced backend gates; its frontend integration and expanded regression tasks depend on the US1/US2 composables they harden. Production release requires its full denial matrix even if the US1 MVP is demonstrated earlier.
- **US4 — Resend Exclusion**: Depends on the US2 invitation surface and contract, then independently proves the exclusion.
- **Phase 7 — Delivery Evidence**: Depends on all selected user stories and their validation checkpoints.

### User Story Dependency Graph

```text
Phase 1 Contracts
        |
Phase 2 Foundation
       / \
     US1  US2
      |   / \
      +--US3  US4
          \   /
        Phase 7 Evidence
```

### Test-First Order Within Each Story

1. Add or rewrite contract/feature/service tests and confirm the new expectation fails for the intended reason.
2. Add composable/component/page tests and confirm no unexpected network calls occur.
3. Implement and pass backend Policy, tenant lookup, and transition rules before exposing dependent frontend controls.
4. Implement frontend services, orchestration, and presentation in that order.
5. Run the story checkpoint before starting dependent integration work.

### Parallel Opportunities

- T002, T003, and T004 change independent contract-source files.
- T008–T011 establish independent backend and frontend failing tests.
- US1 service, action-list, lock-panel, and dialog tests T021–T025 can run in parallel.
- US2 backend, service, composable, and component tests T035–T039 can run in parallel.
- US1 and US2 can proceed concurrently after Phase 2 if ownership of shared account-lifecycle contract and locale files is explicit.
- US3 backend matrix T049 and frontend matrix/stale/page tests T050–T052 can run in parallel.
- Final documentation tasks T068–T069 can run in parallel; mutation-heavy validation tasks T070–T076 should be serialized when they share evidence files.

---

## Parallel Example: User Story 1

```text
Task T021: Frontend lifecycle service contract tests
Task T023: Action-state component tests
Task T024: Lock-panel safe-rendering tests
Task T025: Dialog validation and accessibility tests
```

## Parallel Example: User Story 2

```text
Task T035: Backend invitation-mode user creation tests
Task T037: Frontend invitation service tests
Task T038: Invitation orchestration composable tests
Task T039: Invitation panel tests
```

## Parallel Example: User Story 3

```text
Task T049: Backend tenant/authorization matrix
Task T050: Frontend permission/no-request matrix
Task T051: Stale-context response matrix
Task T052: Mounted page visibility matrix
```

## Parallel Example: User Story 4

```text
Task T063: Unit contract/component resend-exclusion assertions
Task T064: Browser resend/secret-leakage assertions after the shared E2E fixture exists
```

---

## Implementation Strategy

### MVP First

1. Complete Phase 1 contracts.
2. Complete Phase 2 backend Policy, tenant-safe lookup, and frontend authority foundation.
3. Complete US1 and demonstrate same-scope action transitions.
4. Do not release the MVP to production until US3's complete negative authorization matrix also passes.

### Incremental Delivery

1. Contracts and foundation establish exact shared authority semantics.
2. US1 activates lifecycle actions and refreshed state.
3. US2 repairs creation and explicit invitation as separate persisted operations.
4. US3 proves tenant isolation, platform behavior, non-enumeration, and stale-context safety.
5. US4 preserves the explicit resend exclusion.
6. Phase 7 produces reproducible evidence without overstating mocked E2E or pending human UAT.

### Task Completion Rules

- Mark a task complete only after its named files and acceptance behavior are implemented and validated.
- Do not expose a frontend action before its backend authorization and tenant rules pass.
- Do not treat `permissionCodes` or UI visibility as server authorization.
- Do not add a resend endpoint, token-path request, package, or global store under this feature.
- Preserve unrelated worktree changes and document any unavoidable overlap before editing.
