# Tasks: Account Lifecycle Workflows UI

**Input**: Design documents from `specs/021-account-lifecycle-ui/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/account-lifecycle-ui-contract.md, quickstart.md

**Tests**: Frontend Vitest coverage is required because this feature changes critical authentication, account lifecycle, permission, tenant, token, and privacy flows. Backend PHPUnit tasks are not included because this is a frontend-only implementation slice. OpenAPI validation is required only if contract review creates a separate contract change.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3, US4)
- Include exact file paths in descriptions

## Path Conventions

- Run implementation tasks from the `schoolmaster-frontend` repository root.
- Specification paths refer to this repository as `schoolmaster-specs/`.
- Frontend paths use `src/` and `tests/` relative to `schoolmaster-frontend`.
- Specification paths use `specs/021-account-lifecycle-ui/` relative to `schoolmaster-specs`.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm repository state, contract gates, and shared frontend structure before story work.

- [ ] T001 Verify the frontend repository has the Feature 017 auth/session and Feature 020 user-detail foundations needed by this plan in `src/pages/auth/` and `src/pages/admin-system/users/`
- [ ] T002 Verify approved account lifecycle operation IDs and blocked token-based admin resend behavior against `schoolmaster-specs/api/openapi.yaml`
- [ ] T003 [P] Create account lifecycle test folder structure in `tests/unit/account-lifecycle/`
- [ ] T004 [P] Create frontend account lifecycle component folders in `src/components/auth/`, `src/components/admin-system/users/`, and `src/components/ui/admin/`
- [ ] T005 [P] Create frontend account lifecycle composable folders in `src/composables/auth/` and `src/composables/admin-system/`
- [ ] T006 [P] Create frontend account lifecycle service and contract placeholders in `src/services/auth/accountLifecycle.js`, `src/services/admin-system/accountLifecycle.js`, `src/contracts/auth/account-lifecycle.js`, and `src/contracts/admin-system/account-lifecycle.js`
- [ ] T007 [P] Create shared account lifecycle locale placeholder in `src/locales/account-lifecycle.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared contract mapping, safe feedback, and route/service boundaries required by every user story.

**CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T008 [P] Add JSDoc typedefs and mappers for `AccountInvitationView`, `InvitationSetupDraft`, `PasswordResetRequestDraft`, `PasswordResetCompletionDraft`, `AccountLifecycleResultView`, and `SafeFeedbackState` in `src/contracts/auth/account-lifecycle.js`
- [ ] T009 [P] Add JSDoc typedefs and mappers for `AccountLockView`, `AccountLifecycleActionDraft`, `AccountLifecycleResultView`, and admin eligibility inputs in `src/contracts/admin-system/account-lifecycle.js`
- [ ] T010 Implement shared account lifecycle operation constants and blocked admin resend constant in `src/contracts/admin-system/account-lifecycle.js`
- [ ] T011 Implement guest account lifecycle service functions for `completeAccountInvitation`, `requestPasswordReset`, and `completePasswordReset` with no UI state mutation in `src/services/auth/accountLifecycle.js`
- [ ] T012 Implement admin account lifecycle service functions for `createAccountInvitation`, `getAccountLock`, `lockAccount`, `unlockAccount`, and `reactivateAccount` with `X-School-Id` support in `src/services/admin-system/accountLifecycle.js`
- [ ] T013 Normalize invalid-token, neutral-confirmation, validation, unauthorized, forbidden, tenant-mismatch, inactive-school, not-found, conflict, and temporary-unavailable outcomes in `src/services/auth/accountLifecycle.js`
- [ ] T014 Normalize admin forbidden, tenant-mismatch, not-found, conflict, validation, and temporary-unavailable outcomes in `src/services/admin-system/accountLifecycle.js`
- [ ] T015 Add account lifecycle translation keys for labels, validation, neutral confirmation, invalid-token, blocked resend, confirmations, conflicts, and recovery actions in `src/locales/account-lifecycle.js`
- [ ] T016 Integrate account lifecycle locale loading with existing i18n registration in `src/locales/index.js`
- [ ] T017 Add shared no-secret diagnostic helper expectations for account lifecycle operations in `src/services/api/errorDiagnostics.js`
- [ ] T018 [P] Add Vitest coverage for guest contract mappers in `tests/unit/account-lifecycle/contracts/authAccountLifecycle.contract.test.js`
- [ ] T019 [P] Add Vitest coverage for admin contract mappers in `tests/unit/account-lifecycle/contracts/adminAccountLifecycle.contract.test.js`
- [ ] T020 [P] Add Vitest coverage for guest service success/error mapping in `tests/unit/account-lifecycle/services/authAccountLifecycle.service.test.js`
- [ ] T021 [P] Add Vitest coverage for admin service success/error mapping and tenant header behavior in `tests/unit/account-lifecycle/services/adminAccountLifecycle.service.test.js`
- [ ] T022 [P] Add Vitest coverage proving token, password, reason, tenant-private, role, and permission values are excluded from diagnostics in `tests/unit/account-lifecycle/services/accountLifecycleDiagnostics.test.js`
- [ ] T023 Wire account lifecycle locale keys into existing auth feedback tests in `tests/unit/auth/AuthFeedbackState.test.js`

**Checkpoint**: Foundation ready. User story implementation can now begin in parallel.

---

## Phase 3: User Story 1 - Invite users and complete first password setup (Priority: P1) MVP

**Goal**: An authorized administrator can create an invitation from existing user create/detail flows, admin resend is blocked until a safe contract exists, and invited users can complete password setup from a token link.

**Independent Test**: Sign in as an authorized administrator, create an invitation for an eligible existing user, confirm no `delivery_metadata` or token exposure, confirm admin resend is blocked, open invitation setup link, submit valid and invalid passwords, and verify success returns to sign in.

### Tests for User Story 1

- [ ] T024 [P] [US1] Add Vitest coverage for invitation creation request mapping without `delivery_metadata` in `tests/unit/account-lifecycle/services/createAccountInvitation.test.js`
- [ ] T025 [P] [US1] Add Vitest coverage for blocked admin resend rendering in `tests/unit/account-lifecycle/components/UserInvitationPanel.test.js`
- [ ] T026 [P] [US1] Add Vitest coverage for invitation setup composable success, validation, conflict, invalid-token, stale response, and no-secret state in `tests/unit/account-lifecycle/composables/useInvitationSetup.test.js`
- [ ] T027 [P] [US1] Add Vitest coverage for password setup form validation, paste/password-manager compatibility, and submit events in `tests/unit/account-lifecycle/components/PasswordSetupForm.test.js`
- [ ] T028 [P] [US1] Add Vitest coverage for invitation setup page route token handling and sign-in recovery in `tests/unit/account-lifecycle/pages/InvitationSetupPage.test.js`
- [ ] T029 [P] [US1] Add Vitest coverage for auth route metadata for invitation setup guest flow in `tests/unit/account-lifecycle/routes/invitationSetup.routes.test.js`

### Implementation for User Story 1

- [ ] T030 [US1] Implement `createAccountInvitation` request mapper that omits `delivery_metadata` in `src/contracts/admin-system/account-lifecycle.js`
- [ ] T031 [US1] Implement invitation creation and blocked resend service consumption in `src/services/admin-system/accountLifecycle.js`
- [ ] T032 [P] [US1] Implement `useInvitationSetup` route-local state, pending handling, validation mapping, invalid-token mapping, and stale-response protection in `src/composables/auth/useInvitationSetup.js`
- [ ] T033 [P] [US1] Implement `PasswordSetupForm.vue` with password validation, paste/password-manager compatibility, props down, and submit emits in `src/components/auth/PasswordSetupForm.vue`
- [ ] T034 [P] [US1] Implement `AccountLifecycleTokenState.vue` for invalid-token, conflict, and temporary failure feedback in `src/components/auth/AccountLifecycleTokenState.vue`
- [ ] T035 [P] [US1] Implement `AccountLifecycleSuccessState.vue` with return-to-sign-in guidance and no automatic session claim in `src/components/auth/AccountLifecycleSuccessState.vue`
- [ ] T036 [US1] Implement `InvitationSetupPage.vue` as a thin composition surface using `useInvitationSetup` in `src/pages/auth/InvitationSetupPage.vue`
- [ ] T037 [US1] Add invitation setup guest route and metadata in `src/router/modules/auth.routes.js`
- [ ] T038 [US1] Implement `UserInvitationPanel.vue` for invitation creation status and blocked admin resend messaging in `src/components/admin-system/users/UserInvitationPanel.vue`
- [ ] T039 [US1] Integrate `UserInvitationPanel.vue` into existing user create and detail surfaces without direct HTTP calls in `src/pages/admin-system/users/UserCreatePage.vue` and `src/pages/admin-system/users/UserDetailPage.vue`
- [ ] T040 [US1] Add US1 labels, errors, blocked resend, and success messages to `src/locales/account-lifecycle.js`

**Checkpoint**: User Story 1 is independently functional and testable.

---

## Phase 4: User Story 2 - Complete password reset safely (Priority: P2)

**Goal**: Signed-out users can request password reset with email only and complete reset from a valid token with safe neutral and invalid-token feedback.

**Independent Test**: Submit password reset requests for eligible, missing, inactive, locked, deleted, unauthorized, and over-limit identifiers; verify identical neutral confirmation; complete reset with valid and invalid tokens; verify success returns to sign in.

### Tests for User Story 2

- [ ] T041 [P] [US2] Add Vitest coverage for password reset request service mapping that submits email only and never sends `school_id` or `delivery_metadata` in `tests/unit/account-lifecycle/services/passwordResetRequest.service.test.js`
- [ ] T042 [P] [US2] Add Vitest coverage for neutral reset confirmation privacy across eligible, missing, inactive, locked, deleted, unauthorized, and over-limit outcomes in `tests/unit/account-lifecycle/pages/PasswordResetRequestPage.test.js`
- [ ] T043 [P] [US2] Add Vitest coverage for password reset completion service success, validation, conflict, invalid-token, and no-secret mapping in `tests/unit/account-lifecycle/services/passwordResetCompletion.service.test.js`
- [ ] T044 [P] [US2] Add Vitest coverage for password reset completion composable stale-response and sign-in recovery behavior in `tests/unit/account-lifecycle/composables/usePasswordResetCompletion.test.js`
- [ ] T045 [P] [US2] Add Vitest coverage for password reset completion form validation, paste/password-manager compatibility, and submit events in `tests/unit/account-lifecycle/components/PasswordResetCompletionForm.test.js`
- [ ] T046 [P] [US2] Add Vitest coverage for auth route metadata for password reset request and completion guest flows in `tests/unit/account-lifecycle/routes/passwordReset.routes.test.js`

### Implementation for User Story 2

- [ ] T047 [US2] Implement `usePasswordResetRequest` email-only request state and neutral confirmation handling in `src/composables/auth/usePasswordResetRequest.js`
- [ ] T048 [US2] Implement `PasswordResetRequestPage.vue` with only email input, neutral confirmation, and no public school selector in `src/pages/auth/PasswordResetRequestPage.vue`
- [ ] T049 [US2] Implement `usePasswordResetCompletion` token-local state, validation, invalid-token mapping, success, and stale-response protection in `src/composables/auth/usePasswordResetCompletion.js`
- [ ] T050 [P] [US2] Implement `PasswordResetCompletionForm.vue` with password validation, paste/password-manager compatibility, props down, and submit emits in `src/components/auth/PasswordResetCompletionForm.vue`
- [ ] T051 [US2] Implement `PasswordResetCompletionPage.vue` as a thin composition surface with token extraction and return-to-sign-in guidance in `src/pages/auth/PasswordResetCompletionPage.vue`
- [ ] T052 [US2] Replace or extend the existing forgot-password route to use `PasswordResetRequestPage.vue` while preserving Feature 017 behavior in `src/router/modules/auth.routes.js`
- [ ] T053 [US2] Add password reset request and completion labels, neutral confirmation, validation, invalid-token, and success messages to `src/locales/account-lifecycle.js`
- [ ] T054 [US2] Ensure successful reset completion clears stale session assumptions without automatic sign-in in `src/stores/auth/sessionStore.js`

**Checkpoint**: User Stories 1 and 2 both work independently.

---

## Phase 5: User Story 3 - Recover, lock, unlock, and reactivate accounts (Priority: P3)

**Goal**: Authorized account administrators can review lock state and perform approved lock, unlock, recovery, and reactivation actions without bypassing tenant, role, lifecycle, or permission boundaries.

**Independent Test**: Sign in with and without account lifecycle authority, open user detail, verify action visibility stays blocked until approved permission/capability source exists, then review lock state and exercise lock, unlock, recovery, reactivation, conflict, forbidden, tenant, and not-found outcomes.

### Tests for User Story 3

- [ ] T055 [P] [US3] Add Vitest coverage for account lifecycle permission/capability gate and blocked visibility in `tests/unit/account-lifecycle/composables/useAccountLifecycleActions.permissions.test.js`
- [ ] T056 [P] [US3] Add Vitest coverage for account lock review service and mapper behavior in `tests/unit/account-lifecycle/services/getAccountLock.service.test.js`
- [ ] T057 [P] [US3] Add Vitest coverage for lock required reason, unlock no-reason, and optional recovery/reactivation reason request mapping in `tests/unit/account-lifecycle/services/accountLifecycleActions.service.test.js`
- [ ] T058 [P] [US3] Add Vitest coverage for account lifecycle conflict, forbidden, tenant-mismatch, not-found, validation, and temporary-unavailable feedback in `tests/unit/account-lifecycle/composables/useAccountLifecycleActions.outcomes.test.js`
- [ ] T059 [P] [US3] Add Vitest coverage for account lifecycle confirmation dialog reason rules and submit emits in `tests/unit/account-lifecycle/components/AdminAccountLifecycleDialog.test.js`
- [ ] T060 [P] [US3] Add Vitest coverage for user detail account lock and action panels in `tests/unit/account-lifecycle/pages/UserDetailAccountLifecycle.test.js`

### Implementation for User Story 3

- [ ] T061 [US3] Verify and document the approved permission codes or session capability flags for platform and school account lifecycle administration in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T062 [US3] Implement account lifecycle eligibility derivation using confirmed permission codes or capability flags in `src/composables/admin-system/useAccountLifecycleActions.js`
- [ ] T063 [US3] Implement lock state loading, action pending state, stale-response protection, and outcome normalization in `src/composables/admin-system/useAccountLifecycleActions.js`
- [ ] T064 [P] [US3] Implement `AccountLockPanel.vue` with safe lock state display and denied/not-found hiding in `src/components/admin-system/users/AccountLockPanel.vue`
- [ ] T065 [P] [US3] Implement `AccountLifecycleActions.vue` for lock, unlock, recovery, and reactivation visibility and action emits in `src/components/admin-system/users/AccountLifecycleActions.vue`
- [ ] T066 [P] [US3] Implement `AdminAccountLifecycleDialog.vue` with required lock reason, optional recovery/reactivation reason, no-reason unlock, validation summary, and pending state in `src/components/ui/admin/AdminAccountLifecycleDialog.vue`
- [ ] T067 [US3] Integrate account lock and action panels into the existing user detail page without direct HTTP calls in `src/pages/admin-system/users/UserDetailPage.vue`
- [ ] T068 [US3] Add account lock, unlock, recovery, reactivation, conflict, denial, and permission-gate text to `src/locales/account-lifecycle.js`

**Checkpoint**: User Stories 1, 2, and 3 work independently.

---

## Phase 6: User Story 4 - Preserve safe login and session recovery behavior (Priority: P4)

**Goal**: Account lifecycle flows preserve safe auth/session behavior across signed-in guest-link access, inactive-user, locked-account, expired-session, invalid-token, forbidden, tenant-mismatch, and temporary-unavailable states.

**Independent Test**: Force inactive-user, locked-account, invalid-token, expired-session, forbidden, tenant-mismatch, and temporary-unavailable states across invitation, reset, and admin lifecycle flows and verify safe messages and recovery actions.

### Tests for User Story 4

- [ ] T069 [P] [US4] Add Vitest coverage for signed-in users opening guest invitation setup and password reset completion links in `tests/unit/account-lifecycle/routes/guestLifecycleRouteGuards.test.js`
- [ ] T070 [P] [US4] Add Vitest coverage for preserving Feature 017 inactive-user, lockout, expired-session, unauthorized, forbidden, inactive-school, tenant-mismatch, and temporary-unavailable mappings in `tests/unit/account-lifecycle/services/accountLifecycleAuthIntegration.test.js`
- [ ] T071 [P] [US4] Add Vitest coverage for shared token and success feedback accessibility and recovery actions in `tests/unit/account-lifecycle/components/AccountLifecycleFeedbackState.test.js`
- [ ] T072 [P] [US4] Add Vitest coverage that lifecycle flows never claim automatic sign-in after setup/reset completion in `tests/unit/account-lifecycle/pages/accountLifecycleNoAutoSignin.test.js`

### Implementation for User Story 4

- [ ] T073 [US4] Update route guards to allow token-proven guest lifecycle routes without mixing current signed-in permissions in `src/router/modules/auth.routes.js`
- [ ] T074 [US4] Integrate lifecycle invalid-token and success states with existing auth feedback patterns in `src/components/auth/AuthFeedbackState.vue`
- [ ] T075 [US4] Add auth/session recovery actions for lifecycle invalid-token, success, and temporary-unavailable states in `src/components/auth/AuthRecoveryActions.vue`
- [ ] T076 [US4] Ensure account lifecycle services preserve existing auth/session error mapping for inactive-user, lockout, expired-session, unauthorized, forbidden, inactive-school, tenant-mismatch, and temporary-unavailable outcomes in `src/services/auth/authErrorMapper.js`
- [ ] T077 [US4] Add remaining auth/session integration labels and recovery action text to `src/locales/account-lifecycle.js`

**Checkpoint**: All user stories are independently functional.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Verification, documentation, accessibility, and release readiness across all stories.

- [ ] T078 [P] Run focused account lifecycle Vitest suite and record results in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T079 [P] Run full frontend unit suite and record `npm test` result in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T080 [P] Run frontend build check and record `npm run build` result in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T081 [P] Review account lifecycle forms and dialogs at 390px, 768px, and 1440px and record responsive findings in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T082 [P] Review keyboard navigation, focus order, dialog focus trap, form labels, and feedback semantics for account lifecycle UI in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T083 [P] Review diagnostics and client storage for lifecycle token, password, reason, tenant-private, role, and permission leaks in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T084 Update implementation evidence for operation mapping, permission/capability source, blocked resend state, email-only reset request, no-secret diagnostics, and timed SC-003/SC-005/SC-006 scenario results in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T085 If OpenAPI changes were required for non-secret resend or permission/capability exposure, run Redocly validation and record the result in `specs/021-account-lifecycle-ui/quickstart.md`
- [ ] T086 Run representative administrator usability review for invite, blocked resend, lock, unlock, recover, and reactivate decisions and record the SC-009 result in `specs/021-account-lifecycle-ui/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; can start immediately.
- **Foundational (Phase 2)**: Depends on Setup completion; blocks all user stories.
- **User Stories (Phase 3+)**: Depend on Foundational completion.
- **Polish (Phase 7)**: Depends on all desired user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational; MVP.
- **User Story 2 (P2)**: Can start after Foundational; independent guest reset flow that reuses foundational contract/services.
- **User Story 3 (P3)**: Can start after Foundational; independent admin workflow, but final visibility depends on confirmed permission/capability source.
- **User Story 4 (P4)**: Can start after Foundational and is most useful after US1/US2 routes exist; it must preserve all earlier story behavior.

### Within Each User Story

- Write Vitest coverage first and confirm it fails before implementation.
- Contract mapper and service tests precede composable/component/page tasks.
- Services and mappers precede composables.
- Composables precede route pages.
- Components receive props and emit events; they do not call services directly.
- Route pages compose components and composables; they do not call Axios directly.

### Parallel Opportunities

- Setup folder/file scaffolding tasks T003-T007 can run in parallel.
- Foundational mapper, service, locale, and test tasks T008-T023 can run in parallel by file after T006.
- US1 tests T024-T029 can run in parallel.
- US1 component tasks T032-T035 can run in parallel after T030-T031.
- US2 tests T041-T046 can run in parallel.
- US2 implementation tasks T047-T050 can run in parallel after foundational services.
- US3 tests T055-T060 can run in parallel.
- US3 component tasks T064-T066 can run in parallel after T062-T063.
- US4 tests T069-T072 can run in parallel.
- Polish tasks T078-T083 and T086 can run in parallel after implementation.

---

## Parallel Example: User Story 1

```bash
Task: "T024 [P] [US1] Add Vitest coverage for invitation creation request mapping without delivery_metadata in tests/unit/account-lifecycle/services/createAccountInvitation.test.js"
Task: "T025 [P] [US1] Add Vitest coverage for blocked admin resend rendering in tests/unit/account-lifecycle/components/UserInvitationPanel.test.js"
Task: "T026 [P] [US1] Add Vitest coverage for invitation setup composable success, validation, conflict, invalid-token, stale response, and no-secret state in tests/unit/account-lifecycle/composables/useInvitationSetup.test.js"
Task: "T027 [P] [US1] Add Vitest coverage for password setup form validation, paste/password-manager compatibility, and submit events in tests/unit/account-lifecycle/components/PasswordSetupForm.test.js"
Task: "T028 [P] [US1] Add Vitest coverage for invitation setup page route token handling and sign-in recovery in tests/unit/account-lifecycle/pages/InvitationSetupPage.test.js"
Task: "T029 [P] [US1] Add Vitest coverage for auth route metadata for invitation setup guest flow in tests/unit/account-lifecycle/routes/invitationSetup.routes.test.js"
```

## Parallel Example: User Story 2

```bash
Task: "T041 [P] [US2] Add Vitest coverage for password reset request service mapping that submits email only and never sends school_id or delivery_metadata in tests/unit/account-lifecycle/services/passwordResetRequest.service.test.js"
Task: "T042 [P] [US2] Add Vitest coverage for neutral reset confirmation privacy across eligible, missing, inactive, locked, deleted, unauthorized, and over-limit outcomes in tests/unit/account-lifecycle/pages/PasswordResetRequestPage.test.js"
Task: "T043 [P] [US2] Add Vitest coverage for password reset completion service success, validation, conflict, invalid-token, and no-secret mapping in tests/unit/account-lifecycle/services/passwordResetCompletion.service.test.js"
Task: "T044 [P] [US2] Add Vitest coverage for password reset completion composable stale-response and sign-in recovery behavior in tests/unit/account-lifecycle/composables/usePasswordResetCompletion.test.js"
```

## Parallel Example: User Story 3

```bash
Task: "T055 [P] [US3] Add Vitest coverage for account lifecycle permission/capability gate and blocked visibility in tests/unit/account-lifecycle/composables/useAccountLifecycleActions.permissions.test.js"
Task: "T056 [P] [US3] Add Vitest coverage for account lock review service and mapper behavior in tests/unit/account-lifecycle/services/getAccountLock.service.test.js"
Task: "T057 [P] [US3] Add Vitest coverage for lock required reason, unlock no-reason, and optional recovery/reactivation reason request mapping in tests/unit/account-lifecycle/services/accountLifecycleActions.service.test.js"
Task: "T058 [P] [US3] Add Vitest coverage for account lifecycle conflict, forbidden, tenant-mismatch, not-found, validation, and temporary-unavailable feedback in tests/unit/account-lifecycle/composables/useAccountLifecycleActions.outcomes.test.js"
```

## Parallel Example: User Story 4

```bash
Task: "T069 [P] [US4] Add Vitest coverage for signed-in users opening guest invitation setup and password reset completion links in tests/unit/account-lifecycle/routes/guestLifecycleRouteGuards.test.js"
Task: "T070 [P] [US4] Add Vitest coverage for preserving Feature 017 inactive-user, lockout, expired-session, unauthorized, forbidden, inactive-school, tenant-mismatch, and temporary-unavailable mappings in tests/unit/account-lifecycle/services/accountLifecycleAuthIntegration.test.js"
Task: "T071 [P] [US4] Add Vitest coverage for shared token and success feedback accessibility and recovery actions in tests/unit/account-lifecycle/components/AccountLifecycleFeedbackState.test.js"
Task: "T072 [P] [US4] Add Vitest coverage that lifecycle flows never claim automatic sign-in after setup/reset completion in tests/unit/account-lifecycle/pages/accountLifecycleNoAutoSignin.test.js"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. Stop and validate US1 independently with invitation creation, blocked resend, token setup, invalid-token, validation, and no-secret checks.
5. Demo or hand off US1 before adding reset or admin recovery flows.

### Incremental Delivery

1. Deliver Setup + Foundational shared mappers, services, feedback, and locale.
2. Deliver US1 invitation creation and setup.
3. Deliver US2 password reset request and completion.
4. Deliver US3 admin lock/recovery/reactivation after permission/capability source is confirmed.
5. Deliver US4 auth/session integration hardening.
6. Run Polish verification and quickstart evidence.

### Parallel Team Strategy

1. Team completes Setup + Foundational together.
2. Developer A implements US1 invitation setup and admin invitation panel.
3. Developer B implements US2 password reset request/completion.
4. Developer C implements US3 account lock/recovery actions after permission/capability source is confirmed.
5. Developer D implements US4 route guard/auth feedback integration once guest route shells exist.

---

## Notes

- `[P]` tasks are different files with no dependency on incomplete same-file work.
- `[US1]`, `[US2]`, `[US3]`, and `[US4]` labels map to prioritized user stories in `spec.md`.
- Backend and OpenAPI behavior are not implemented in this task list.
- Admin resend stays blocked until OpenAPI provides non-secret resend by invitation or user identifier.
- Account lifecycle action visibility stays blocked until approved permission codes or session capability flags are confirmed.
- Token values, plaintext passwords, reasons, tenant-private data, role internals, and permission payloads must not be persisted or logged.
