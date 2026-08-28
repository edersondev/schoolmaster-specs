---
description: "Implementation tasks for secure user password delivery"
---

# Tasks: Secure User Password Delivery

**Input**: Design documents from `specs/039-user-password-delivery/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/user-password-delivery.md`, and `quickstart.md`

**Tests**: Critical PHPUnit, OpenAPI, Vitest, browser, and build evidence is
required by FR-012. Complete every backend task and its verification gate before
starting any frontend task.

## Phase 1: Setup and Contract

**Purpose**: Publish the additive API contract and establish the backend-first
delivery order.

- [X] T001 Add `POST /api/v1/users/{userId}/password-delivery` and its path reference, `requestUserPasswordDelivery` operation, tenant header, and safe 201/401/403/404/409/422/429/503 outcomes, keeping missing/inactive/mismatched/unauthorized tenant context under `403 tenant_mismatch` and request validation under `422`, in `schoolmaster-specs/api/openapi.yaml` and `schoolmaster-specs/api/paths/users/password-delivery.yaml`
- [X] T002 [P] Add the safe delivery result schema and any reusable response components with no token, URL, password, email address, or provider diagnostic in `schoolmaster-specs/api/components/schemas/account-lifecycle/PasswordDeliveryRequestResult.yaml` and `schoolmaster-specs/api/components/responses/account-lifecycle/`
- [X] T003 Validate the changed modular contract with `schoolmaster-specs/redocly.yaml` and record the contract decision in `schoolmaster-specs/specs/039-user-password-delivery/contracts/user-password-delivery.md`
- [X] T004 Confirm `039-user-password-delivery` is checked out in `schoolmaster-backend` and `schoolmaster-frontend`, and record that backend contract, implementation, and tests are the gate in `schoolmaster-specs/specs/039-user-password-delivery/quickstart.md`

---

## Phase 2: Foundational Backend Prerequisites

**Purpose**: Create the shared, tenant-safe server foundation that blocks all
user stories.

- [X] T005 Add the authenticated, school-context-scoped password-delivery route and controller dependency wiring in `schoolmaster-backend/routes/api.php` and `schoolmaster-backend/app/Http/Controllers/Api/V1/PasswordDeliveryController.php`
- [X] T006 [P] Add the no-body authorization request and safe response resource in `schoolmaster-backend/app/Http/Requests/AccountLifecycle/RequestPasswordDeliveryRequest.php` and `schoolmaster-backend/app/Http/Resources/PasswordDeliveryRequestResource.php`
- [X] T007 [P] Add the user-scoped `account_lifecycle.manage` authorization and tenant-first lookup rules in `schoolmaster-backend/app/Policies/AccountLifecyclePolicy.php` and `schoolmaster-backend/app/Repositories/AccountLifecycleRepository.php`
- [X] T008 Add delivery-request persistence/audit support, including the 3-per-user/scope/24-hour query and no-secret metadata, in `schoolmaster-backend/app/Models/PasswordResetRequest.php`, `schoolmaster-backend/app/Services/AccountLifecycle/AccountLifecycleAuditService.php`, and `schoolmaster-backend/app/Services/AccountLifecycle/EmailDeliveryRequestMetadataService.php`

**Checkpoint**: Contract and backend foundation are ready; no frontend task may
start until the User Story 1 backend gate passes.

---

## Phase 3: User Story 1 - Send setup or reset delivery after active creation (Priority: P1) 🎯 MVP

**Goal**: An authorized administrator explicitly requests a secure email link
for an eligible active user without handling a secret.

**Independent Test**: An authorized same-school administrator receives only the
safe accepted response; denied, locked, inactive, invited, deleted,
cross-tenant, rate-limited, and mail-failure attempts issue no usable link.

### Tests for User Story 1

- [X] T009 [P] [US1] Add OpenAPI contract coverage for `requestUserPasswordDelivery` and its secret-free response/error shapes in `schoolmaster-specs/api/openapi.yaml` and `schoolmaster-specs/api/paths/users/password-delivery.yaml`
- [X] T010 [P] [US1] Add PHPUnit feature coverage for active creation with no automatic delivery, authority, tenant-first lookup with `403 tenant_mismatch`, eligibility, three-delivery limit, safe response, and no-secret exposure in `schoolmaster-backend/tests/Feature/AccountLifecycle/UserPasswordDeliveryTest.php` and `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleSecretExposureTest.php`
- [X] T011 [US1] Add PHPUnit coverage for accepted mail handoff, failed handoff with retry, single-use token issuance, reset-token suppression blocking administrator delivery before issuance, and supersession only after success in `schoolmaster-backend/tests/Feature/AccountLifecycle/UserPasswordDeliveryTest.php` and `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationDeliveryFailureTest.php`

### Backend implementation for User Story 1

- [X] T012 [US1] Implement authorized delivery orchestration, active/unlocked eligibility, tenant-first scope with established `tenant_mismatch` failures, 24-hour limit, reset-token suppression precheck, audit records, and safe exceptions in `schoolmaster-backend/app/Services/AccountLifecycle/PasswordDeliveryService.php`
- [X] T013 [US1] Integrate single-use reset-token issue only after suppression passes, accepted-mail handoff, rollback on mail failure, and older-token supersession into `schoolmaster-backend/app/Services/AccountLifecycle/PasswordDeliveryService.php`, `schoolmaster-backend/app/Services/AccountLifecycle/LifecycleTokenService.php`, and `schoolmaster-backend/app/Mail/PasswordDeliveryMail.php`
- [X] T014 [US1] Wire the controller to policy, request, service, resource, and documented HTTP outcomes in `schoolmaster-backend/app/Http/Controllers/Api/V1/PasswordDeliveryController.php`, `schoolmaster-backend/app/Policies/AccountLifecyclePolicy.php`, and `schoolmaster-backend/app/Exceptions/`
- [X] T015 [US1] Run the User Story 1 PHP feature suite and Redocly validation, then record the backend verification result before frontend work in `schoolmaster-specs/specs/039-user-password-delivery/quickstart.md`

### Frontend implementation for User Story 1 — only after T015

- [x] T016 [P] [US1] Add the secret-free delivery result contract, operation ID, and eligibility mapping in `schoolmaster-frontend/src/contracts/admin-system/account-lifecycle.js` and `schoolmaster-frontend/tests/unit/account-lifecycle/contracts/adminAccountLifecycle.contract.test.js`
- [x] T017 [P] [US1] Add the authorized `POST` service method, tenant header, abort signal, and safe error mapping in `schoolmaster-frontend/src/services/admin-system/accountLifecycle.js` and `schoolmaster-frontend/tests/unit/account-lifecycle/services/adminAccountLifecycle.service.test.js`
- [x] T018 [US1] Extend the lifecycle composable with delivery single-flight, retry, and target/actor/tenant/route invalidation in `schoolmaster-frontend/src/composables/admin-system/useAccountLifecycleActions.js` and `schoolmaster-frontend/tests/unit/account-lifecycle/composables/useAccountLifecycleActions.outcomes.test.js`
- [x] T019 [US1] Add authorized post-create and user-detail delivery controls with pending, success, denial, and unavailable states that expose no secret in `schoolmaster-frontend/src/components/admin-system/users/AccountLifecycleActions.vue`, `schoolmaster-frontend/src/pages/admin-system/users/CreateUserPage.vue`, and `schoolmaster-frontend/src/pages/admin-system/users/UserDetailPage.vue`
- [x] T020 [US1] Add component and page coverage for delivery controls, stale requests, focus/keyboard handling, and no token/password/private diagnostic leakage in `schoolmaster-frontend/tests/unit/account-lifecycle/components/AccountLifecycleActions.test.js`, `schoolmaster-frontend/tests/unit/account-lifecycle/pages/CreateUserAccountInvitation.test.js`, and `schoolmaster-frontend/tests/unit/account-lifecycle/pages/UserDetailAccountLifecycle.test.js`

**Checkpoint**: Administrators can safely request eligible-user delivery, and
the implementation is independently testable end to end.

---

## Phase 4: User Story 2 - Complete delivered credential setup or reset (Priority: P2)

**Goal**: A recipient uses the existing guest completion flow once, receives
neutral invalid-link feedback, and is sent to sign in after success.

**Independent Test**: A valid delivered link completes once and revokes active
sessions; invalid, expired, reused, superseded, and malformed links remain
neutral and a validation retry keeps no password beyond the form.

### Tests and backend verification for User Story 2

- [X] T021 [P] [US2] Extend completion PHPUnit coverage for delivery-issued valid, expired, reused, superseded, locked, malformed, and completion-throttled reset links plus session revocation, including proof that completion suppression blocks new administrator delivery during the 15-minute suppression window, in `schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetCompletionTest.php` and `schoolmaster-backend/tests/Feature/AccountLifecycle/UserPasswordDeliveryTest.php`
- [X] T022 [US2] Verify `completePasswordReset` preserves neutral token rejection, password validation, and atomic session revocation for delivery-issued tokens in `schoolmaster-backend/app/Services/AccountLifecycle/PasswordResetService.php` and `schoolmaster-backend/app/Http/Controllers/Api/V1/PasswordResetController.php`
- [X] T023 [US2] Run and record the completion backend suite before guest-frontend work in `schoolmaster-specs/specs/039-user-password-delivery/quickstart.md`

### Frontend implementation for User Story 2 — only after T023

- [x] T024 [P] [US2] Extend guest reset-completion service/composable coverage for delivery-issued token success, neutral invalid links, and validation retry safety in `schoolmaster-frontend/src/services/auth/accountLifecycle.js`, `schoolmaster-frontend/src/composables/auth/usePasswordResetCompletion.js`, and `schoolmaster-frontend/tests/unit/account-lifecycle/composables/usePasswordResetCompletion.test.js`
- [x] T025 [US2] Update the reset-completion page and form to preserve password-manager support, neutral token state, and sign-in guidance without storing secrets in `schoolmaster-frontend/src/pages/auth/PasswordResetCompletionPage.vue` and `schoolmaster-frontend/src/components/auth/PasswordResetCompletionForm.vue`
- [x] T026 [US2] Add guest route/component coverage for valid, expired, reused, superseded, malformed, and validation-retry completion states in `schoolmaster-frontend/tests/unit/account-lifecycle/components/PasswordResetCompletionForm.test.js` and `schoolmaster-frontend/tests/unit/account-lifecycle/routes/passwordReset.routes.test.js`

**Checkpoint**: Delivered links complete safely through the existing guest path
without changing invitation setup behavior.

---

## Phase 5: User Story 3 - Preserve privacy and delivery controls (Priority: P3)

**Goal**: All public and administrative feedback remains safe while preserving
the public reset request's non-enumerating behavior.

**Independent Test**: Authorized, unauthorized, unavailable, and limited
delivery paths reveal only documented safe feedback, and public reset requests
remain indistinguishable for eligible and ineligible accounts.

- [X] T027 [P] [US3] Add backend privacy and tenant-isolation coverage for unauthorized, cross-tenant, missing/inactive/mismatched tenant context as `403 tenant_mismatch`, inactive, invited, soft-deleted, locked, limited, suppressed, and mail-unavailable delivery attempts in `schoolmaster-backend/tests/Feature/AccountLifecycle/UserPasswordDeliveryTest.php` and `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleTenantIsolationTest.php`
- [X] T028 [US3] Verify public reset request behavior remains non-enumerating across delivery-related eligibility and rate-limit states in `schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetRequestTest.php` and `schoolmaster-backend/app/Services/AccountLifecycle/PasswordResetService.php`
- [x] T029 [US3] Add frontend service/composable diagnostics coverage that permits only status, channel, and requested time and clears state on permission, tenant, target, and route changes in `schoolmaster-frontend/tests/unit/account-lifecycle/services/accountLifecycleDiagnostics.test.js` and `schoolmaster-frontend/tests/unit/account-lifecycle/composables/useAccountLifecycleActions.permissions.test.js`
- [x] T030 [US3] Localize safe delivery acceptance, conflict, limit, and unavailable feedback without account or mail-provider detail in `schoolmaster-frontend/src/locales/account-lifecycle.js` and `schoolmaster-frontend/src/components/admin-system/users/AccountLifecycleActions.vue`

**Checkpoint**: Delivery and public reset feedback meet the privacy and
non-enumeration requirements.

---

## Phase 6: Polish and Cross-Cutting Validation

**Purpose**: Validate the released behavior across both implementation repos.

- [X] T031 [P] Run the documented backend contract and feature commands and record exact results in `schoolmaster-specs/specs/039-user-password-delivery/quickstart.md`
- [x] T032 [P] Add and run Playwright delivery coverage for authorized request, safe feedback, stale-state invalidation, keyboard/focus behavior, and forbidden-data absence; then run focused Vitest and the production build in `schoolmaster-frontend/e2e/account-lifecycle.spec.js`, `schoolmaster-frontend/tests/unit/account-lifecycle/`, and `schoolmaster-frontend/package.json`
- [x] T033 Review the final contract, UI, storage, browser URL, and diagnostics for forbidden token, URL, password, email, and provider-detail exposure in `schoolmaster-specs/specs/039-user-password-delivery/quickstart.md`

---

## Dependencies and Execution Order

1. Complete T001–T004, then T005–T008.
2. Complete the P1 backend tests and implementation (T009–T015). **T015 is the
   mandatory backend gate** before T016–T020.
3. Complete P2 backend verification (T021–T023) before its guest frontend work
   (T024–T026).
4. Complete P3 privacy work (T027–T030), then cross-cutting validation
   (T031–T033).

## Parallel Opportunities

- T002 can run with T001; T006 and T007 can run after the contract is fixed.
- T009 and T010 can run in parallel; complete T011 after T010 because both
  modify `UserPasswordDeliveryTest.php`. T016 and T017 may run in parallel only
  after T015.
- T021 and T024 are each parallelizable within their respective backend-first
  gates; T027 and T029 may run in parallel after the P1/P2 paths exist.
- T031 and T032 can run in parallel once implementation is complete.

## Implementation Strategy

Deliver the MVP through T020: contract, backend delivery, verified backend
gate, then authorized controls. Add the recipient completion proof (T021–T026)
next, preserve privacy/non-enumeration (T027–T030), and finish with the release
evidence (T031–T033).
