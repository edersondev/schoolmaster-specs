# Tasks: Backend Account Lifecycle Workflows

**Input**: Design documents from `/specs/008-account-lifecycle-workflows/`
**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md), [research.md](./research.md), [data-model.md](./data-model.md), [contracts/backend-account-lifecycle.md](./contracts/backend-account-lifecycle.md), [quickstart.md](./quickstart.md)

**Tests**: Critical backend flow tests and OpenAPI contract validation are required by the specification. No frontend tasks are included because this slice explicitly excludes frontend implementation.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared contract and backend foundation work is complete.

## Phase 1: Setup (Contract and Repository Preparation)

**Purpose**: Establish contract-first delivery and repository targets before backend implementation starts.

- [ ] T001 Update account lifecycle scope and operation placeholders in `schoolmaster-specs/api/openapi.yaml`
- [ ] T002 Update account lifecycle scope and operation placeholders in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T003 [P] Add feature-to-repository implementation notes in `schoolmaster-specs/specs/008-account-lifecycle-workflows/quickstart.md`
- [ ] T004 [P] Create backend account lifecycle namespace directories in `schoolmaster-backend/app/Services/AccountLifecycle/.gitkeep`
- [ ] T005 [P] Create backend account lifecycle DTO namespace directory in `schoolmaster-backend/app/DTOs/AccountLifecycle/.gitkeep`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared contract components, persistence, authorization, tenant, token, response, and audit primitives required by every user story.

**Critical**: No user story implementation should begin until this phase is complete.

- [ ] T006 Add reusable OpenAPI account lifecycle schemas, errors, and envelope components in `schoolmaster-specs/api/openapi.yaml`
- [ ] T007 Mirror reusable account lifecycle schemas, errors, and envelope components in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T008 [P] Add `account_invitations` migration with UUIDs, `school_id`, token hash, status, expiry, send counters, failed completion counters, delivery metadata, and audit-safe fields in `schoolmaster-backend/database/migrations/2026_05_27_000001_create_account_invitations_table.php`
- [ ] T009 [P] Add `password_reset_requests` migration with UUIDs, optional `school_id`, identifier hash, token hash, status, expiry, request throttle counters, failure counters, suppression window, and delivery metadata in `schoolmaster-backend/database/migrations/2026_05_27_000002_create_password_reset_requests_table.php`
- [ ] T010 [P] Add `account_locks` migration with UUIDs, optional `school_id`, lock type, status, reason, actor, locked and cleared timestamps in `schoolmaster-backend/database/migrations/2026_05_27_000003_create_account_locks_table.php`
- [ ] T011 [P] Add `account_recoveries` migration with UUIDs, optional `school_id`, recovery type, from/to state, actor, reason, and completion timestamp in `schoolmaster-backend/database/migrations/2026_05_27_000004_create_account_recoveries_table.php`
- [ ] T012 [P] Create `AccountInvitation` Eloquent model with UUID, relationships, casts, and guarded secret fields in `schoolmaster-backend/app/Models/AccountInvitation.php`
- [ ] T013 [P] Create `PasswordResetRequest` Eloquent model with UUID, relationships, casts, and guarded secret fields in `schoolmaster-backend/app/Models/PasswordResetRequest.php`
- [ ] T014 [P] Create `AccountLock` Eloquent model with UUID, relationships, casts, and lock-state helpers in `schoolmaster-backend/app/Models/AccountLock.php`
- [ ] T015 [P] Create `AccountRecovery` Eloquent model with UUID, relationships, casts, and recovery-state helpers in `schoolmaster-backend/app/Models/AccountRecovery.php`
- [ ] T016 Create tenant-aware account lifecycle lookup repository for users, roles, schools, tokens, throttle windows, and lock state in `schoolmaster-backend/app/Repositories/AccountLifecycleRepository.php`
- [ ] T017 Create account lifecycle token service for hashed single-use token creation, verification, expiry, supersession, and secret non-exposure in `schoolmaster-backend/app/Services/AccountLifecycle/LifecycleTokenService.php`
- [ ] T018 Create account lifecycle authorization policy for platform account administrators and same-school user administrators in `schoolmaster-backend/app/Policies/AccountLifecyclePolicy.php`
- [ ] T019 Create bearer-token revocation service for password reset, administrative lock, unlock/recovery, reactivation, and user deactivation transitions in `schoolmaster-backend/app/Services/AccountLifecycle/BearerTokenRevocationService.php`
- [ ] T020 Create account lifecycle audit service for tenant-safe events without plaintext credentials or reusable token values in `schoolmaster-backend/app/Services/AccountLifecycle/AccountLifecycleAuditService.php`
- [ ] T021 Create email delivery request metadata service without provider-specific messaging behavior in `schoolmaster-backend/app/Services/AccountLifecycle/EmailDeliveryRequestMetadataService.php`
- [ ] T022 Create account lifecycle response resource helpers for success, accepted, token-invalid, tenant-mismatch, inactive-record, conflict, validation, and forbidden envelopes in `schoolmaster-backend/app/Http/Resources/AccountLifecycleResource.php`

**Checkpoint**: Shared OpenAPI components and backend foundation are ready for user-story implementation.

---

## Phase 3: User Story 1 - Invite Users and Complete Initial Password Setup (Priority: P1) MVP

**Goal**: Authorized platform or school administrators can invite eligible users and invited users can complete initial password setup with a valid single-use invitation token.

**Independent Test**: Invite a same-scope user, complete password setup with a valid invitation token, verify authentication eligibility only after setup, and verify invalid scope, inactive dependency, over-limit resend, failed completion revocation, and secret non-exposure behavior.

### Tests for User Story 1

- [ ] T023 [P] [US1] Verify invitation create/resend/setup OpenAPI paths, schemas, responses, and no token secret echo in `schoolmaster-specs/api/openapi.yaml` and `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T024 [P] [US1] Add PHPUnit feature tests for platform and same-school invitation creation in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationCreationTest.php`
- [ ] T025 [P] [US1] Add PHPUnit feature tests for invitation completion and first password setup in `schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordSetupTest.php`
- [ ] T026 [P] [US1] Add PHPUnit feature tests for invitation resend limit, token supersession, failed completion revocation, tenant isolation, inactive dependencies, and secret non-exposure in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationSecurityTest.php`

### Implementation for User Story 1

- [ ] T027 [US1] Define invitation create, resend or replace, and setup operations in `schoolmaster-specs/api/openapi.yaml`
- [ ] T028 [US1] Mirror invitation create, resend or replace, and setup operations in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T029 [P] [US1] Create invitation creation DTO with actor scope, target identity, school context, role dependency, and delivery metadata inputs in `schoolmaster-backend/app/DTOs/AccountLifecycle/CreateAccountInvitationData.php`
- [ ] T030 [P] [US1] Create invitation setup DTO with token proof and password policy inputs in `schoolmaster-backend/app/DTOs/AccountLifecycle/CompleteAccountInvitationData.php`
- [ ] T031 [P] [US1] Create invitation create Form Request rejecting undocumented fields and invalid scope payloads in `schoolmaster-backend/app/Http/Requests/AccountLifecycle/CreateAccountInvitationRequest.php`
- [ ] T032 [P] [US1] Create invitation setup Form Request enforcing 12-to-128 character passwords, common-password rejection, and password-manager-compatible input in `schoolmaster-backend/app/Http/Requests/AccountLifecycle/CompleteAccountInvitationRequest.php`
- [ ] T033 [US1] Implement invitation creation, resend/replacement, 7-day expiry, 3-per-user-and-scope-per-24-hours send limit, token supersession, and email delivery metadata behavior in `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationService.php`
- [ ] T034 [US1] Implement invitation completion, single-use token validation, 5-failure revocation, initial password setup, authentication eligibility checks, and audit events in `schoolmaster-backend/app/Services/AccountLifecycle/PasswordSetupService.php`
- [ ] T035 [US1] Create invitation API resource that omits plaintext tokens, credential hashes, and delivery-provider secrets in `schoolmaster-backend/app/Http/Resources/AccountInvitationResource.php`
- [ ] T036 [US1] Wire invitation create, resend or replace, and setup controller actions under `/api/v1/account-invitations` in `schoolmaster-backend/app/Http/Controllers/Api/V1/AccountInvitationController.php`
- [ ] T037 [US1] Register invitation routes with tenant and authorization middleware in `schoolmaster-backend/routes/api.php`

**Checkpoint**: User Story 1 is independently functional and testable as the MVP.

---

## Phase 4: User Story 2 - Reset Passwords Securely (Priority: P2)

**Goal**: Users can request password reset through non-enumerating behavior and complete credential replacement with a valid single-use reset token.

**Independent Test**: Request a reset for an eligible user, complete reset with a valid token, verify old credentials and active bearer tokens stop working, and verify invalid, expired, reused, inactive, throttled, and cross-scope attempts do not reveal account state.

### Tests for User Story 2

- [ ] T038 [P] [US2] Verify password reset request/completion OpenAPI paths, accepted envelope, token-invalid envelope, and throttle semantics in `schoolmaster-specs/api/openapi.yaml` and `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T039 [P] [US2] Add PHPUnit feature tests for non-enumerating reset request behavior across eligible, missing, inactive, locked, deleted, unauthorized, and over-limit identifiers in `schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetRequestTest.php`
- [ ] T040 [P] [US2] Add PHPUnit feature tests for reset completion, 30-minute expiry, token reuse, token supersession, password policy, bearer-token revocation, and invalid-token failures in `schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetCompletionTest.php`
- [ ] T041 [P] [US2] Add PHPUnit feature tests for 3-per-account-or-IP-per-1-hour reset request throttling and 5-failure reset-token suppression in `schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetThrottleTest.php`

### Implementation for User Story 2

- [ ] T042 [US2] Define password reset request and completion operations in `schoolmaster-specs/api/openapi.yaml`
- [ ] T043 [US2] Mirror password reset request and completion operations in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T044 [P] [US2] Create password reset request DTO with account identifier, optional scope context, IP metadata, and delivery metadata in `schoolmaster-backend/app/DTOs/AccountLifecycle/RequestPasswordResetData.php`
- [ ] T045 [P] [US2] Create password reset completion DTO with token proof and replacement password inputs in `schoolmaster-backend/app/DTOs/AccountLifecycle/CompletePasswordResetData.php`
- [ ] T046 [P] [US2] Create password reset request Form Request preserving non-enumerating validation behavior in `schoolmaster-backend/app/Http/Requests/AccountLifecycle/RequestPasswordResetRequest.php`
- [ ] T047 [P] [US2] Create password reset completion Form Request enforcing token input and password policy in `schoolmaster-backend/app/Http/Requests/AccountLifecycle/CompletePasswordResetRequest.php`
- [ ] T048 [US2] Implement non-enumerating reset request acceptance, 3-per-account-or-IP-per-1-hour throttle, no token/email creation over limit, 30-minute token creation, token supersession, and audit events in `schoolmaster-backend/app/Services/AccountLifecycle/PasswordResetService.php`
- [ ] T049 [US2] Implement reset completion, token validation, password replacement, bearer-token revocation, 5-failure suppression, and invalid-token audit behavior in `schoolmaster-backend/app/Services/AccountLifecycle/PasswordResetService.php`
- [ ] T050 [US2] Create password reset API resource for accepted and completion responses without token or credential secret exposure in `schoolmaster-backend/app/Http/Resources/PasswordResetResource.php`
- [ ] T051 [US2] Wire password reset request and completion controller actions under `/api/v1/auth/password-reset-requests` and `/api/v1/auth/password-resets` in `schoolmaster-backend/app/Http/Controllers/Api/V1/PasswordResetController.php`
- [ ] T052 [US2] Register password reset routes with token-proof and tenant-aware middleware where required in `schoolmaster-backend/routes/api.php`

**Checkpoint**: User Stories 1 and 2 are independently functional and testable.

---

## Phase 5: User Story 3 - Recover Locked or Inactive Accounts (Priority: P3)

**Goal**: Authorized administrators can review, lock, unlock, recover, or reactivate eligible same-scope accounts without bypassing school, user, role, lock, or authorization rules.

**Independent Test**: Lock or recover an eligible account with an authorized same-scope actor, verify bearer-token revocation and audit events, and verify inactive-school, soft-deleted-user, unresolved setup, cross-tenant, and unsupported actor attempts fail.

### Tests for User Story 3

- [ ] T053 [P] [US3] Verify account lock, unlock, recovery, and reactivation OpenAPI paths, transition responses, and conflict envelopes in `schoolmaster-specs/api/openapi.yaml` and `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T054 [P] [US3] Add PHPUnit feature tests for administrative lock durability, unlock, recovery, bearer-token revocation, and audit events in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLockRecoveryTest.php`
- [ ] T055 [P] [US3] Add PHPUnit feature tests for account reactivation eligibility, inactive school rejection, soft-deleted user rejection, unresolved invitation/setup conflicts, and role dependency conflicts in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountReactivationTest.php`
- [ ] T056 [P] [US3] Add PHPUnit feature tests for platform account administrator versus same-school user administrator authorization separation in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleAuthorizationTest.php`

### Implementation for User Story 3

- [ ] T057 [US3] Define account lock, unlock, recovery, and reactivation operations in `schoolmaster-specs/api/openapi.yaml`
- [ ] T058 [US3] Mirror account lock, unlock, recovery, and reactivation operations in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T059 [P] [US3] Create account lock DTO with actor scope, target user, lock type, reason, and school context inputs in `schoolmaster-backend/app/DTOs/AccountLifecycle/AccountLockData.php`
- [ ] T060 [P] [US3] Create account recovery DTO with actor scope, target user, recovery type, reason, and school context inputs in `schoolmaster-backend/app/DTOs/AccountLifecycle/AccountRecoveryData.php`
- [ ] T061 [P] [US3] Create account lock Form Request rejecting unsupported transitions and undocumented fields in `schoolmaster-backend/app/Http/Requests/AccountLifecycle/AccountLockRequest.php`
- [ ] T062 [P] [US3] Create account recovery/reactivation Form Request rejecting unsupported transitions and invalid tenant context in `schoolmaster-backend/app/Http/Requests/AccountLifecycle/AccountRecoveryRequest.php`
- [ ] T063 [US3] Implement administrative lock, durable lock state, unlock, bearer-token revocation, and audit behavior in `schoolmaster-backend/app/Services/AccountLifecycle/AccountLockService.php`
- [ ] T064 [US3] Implement account recovery and reactivation eligibility checks for user state, school state, roles, password setup, lock state, tenant scope, and soft deletes in `schoolmaster-backend/app/Services/AccountLifecycle/AccountRecoveryService.php`
- [ ] T065 [US3] Create account lock and recovery API resources for transition outcomes without cross-tenant state disclosure in `schoolmaster-backend/app/Http/Resources/AccountRecoveryResource.php`
- [ ] T066 [US3] Wire account lock, unlock, recovery, and reactivation controller actions under `/api/v1/users/{userId}/account-lock` and `/api/v1/users/{userId}/account-reactivation` in `schoolmaster-backend/app/Http/Controllers/Api/V1/AccountRecoveryController.php`
- [ ] T067 [US3] Register account lock, recovery, and reactivation routes with authenticated actor, tenant, and policy middleware in `schoolmaster-backend/routes/api.php`

**Checkpoint**: All user stories are independently functional and testable.

---

## Phase 6: Polish and Cross-Cutting Verification

**Purpose**: Validate contract compliance, response consistency, security, and documentation across all completed stories.

- [ ] T068 [P] Run Redocly validation and record results in `schoolmaster-specs/specs/008-account-lifecycle-workflows/quickstart.md`
- [ ] T069 Run full backend PHPUnit suite and record results in `schoolmaster-specs/specs/008-account-lifecycle-workflows/quickstart.md`
- [ ] T070 [P] Verify every implemented backend route maps to an approved OpenAPI operation ID in `schoolmaster-specs/specs/008-account-lifecycle-workflows/contracts/backend-account-lifecycle.md`
- [ ] T071 [P] Verify no account lifecycle response, log, audit event, resource, or delivery metadata exposes plaintext passwords, reusable tokens, credential hashes, or delivery-provider secrets in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleSecretExposureTest.php`
- [ ] T072 [P] Verify user deactivation revokes all active bearer tokens for the affected user in `schoolmaster-backend/tests/Feature/AccountLifecycle/UserDeactivationBearerTokenRevocationTest.php`
- [ ] T073 [P] Verify response envelope consistency for success, accepted, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, token-invalid, and not-found outcomes in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleResponseShapeTest.php`
- [ ] T074 Update implementation notes and blocked-out-of-scope behavior after verification in `schoolmaster-specs/specs/008-account-lifecycle-workflows/quickstart.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Phase 1: Setup** has no dependencies and starts immediately.
- **Phase 2: Foundational** depends on Phase 1 and blocks all user stories.
- **Phase 3: User Story 1** depends on Phase 2 and is the MVP.
- **Phase 4: User Story 2** depends on Phase 2 and can run in parallel with User Story 1 after shared token, policy, audit, response, and repository services exist.
- **Phase 5: User Story 3** depends on Phase 2 and can run in parallel with User Stories 1 and 2 after shared account state, tenant, policy, audit, and bearer-token revocation services exist.
- **Phase 6: Polish** depends on all desired user stories being complete.

### User Story Dependencies

- **US1 (P1)**: No dependency on other user stories after Phase 2.
- **US2 (P2)**: No dependency on US1 after Phase 2, but shares token, password policy, audit, and bearer-token revocation foundation.
- **US3 (P3)**: No dependency on US1 or US2 after Phase 2, but shares authorization, tenant, audit, lock, and bearer-token revocation foundation.

### Within Each User Story

- Contract tasks must be completed before backend route exposure.
- Tests must be written and fail before implementation.
- DTOs and Form Requests precede Services.
- Services precede Resources, Controllers, and route registration.
- Story-specific verification must pass before moving to the next priority story.

---

## Parallel Opportunities

- **Setup**: T003, T004, and T005 can run in parallel.
- **Foundational**: migrations T008-T011 and models T012-T015 can run in parallel after T006-T007 are underway.
- **US1**: tests T023-T026 can run in parallel; DTO/Form Request work T029-T032 can run in parallel.
- **US2**: tests T038-T041 can run in parallel; DTO/Form Request work T044-T047 can run in parallel.
- **US3**: tests T053-T056 can run in parallel; DTO/Form Request work T059-T062 can run in parallel.
- **Polish**: T068, T070, T071, T072, and T073 can run in parallel after all selected user stories are complete.

---

## Parallel Example: User Story 1

```bash
Task: "T023 [US1] Verify invitation create/resend/setup OpenAPI coverage in schoolmaster-specs/api/openapi.yaml and schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml"
Task: "T024 [US1] Add PHPUnit feature tests for platform and same-school invitation creation in schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationCreationTest.php"
Task: "T025 [US1] Add PHPUnit feature tests for invitation completion and first password setup in schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordSetupTest.php"
Task: "T026 [US1] Add PHPUnit feature tests for invitation security cases in schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationSecurityTest.php"
```

---

## Parallel Example: User Story 2

```bash
Task: "T038 [US2] Verify password reset OpenAPI coverage in schoolmaster-specs/api/openapi.yaml and schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml"
Task: "T039 [US2] Add PHPUnit feature tests for non-enumerating reset request behavior in schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetRequestTest.php"
Task: "T040 [US2] Add PHPUnit feature tests for reset completion behavior in schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetCompletionTest.php"
Task: "T041 [US2] Add PHPUnit feature tests for reset throttling in schoolmaster-backend/tests/Feature/AccountLifecycle/PasswordResetThrottleTest.php"
```

---

## Parallel Example: User Story 3

```bash
Task: "T053 [US3] Verify account lock/recovery/reactivation OpenAPI coverage in schoolmaster-specs/api/openapi.yaml and schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml"
Task: "T054 [US3] Add PHPUnit feature tests for lock and recovery behavior in schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLockRecoveryTest.php"
Task: "T055 [US3] Add PHPUnit feature tests for reactivation eligibility in schoolmaster-backend/tests/Feature/AccountLifecycle/AccountReactivationTest.php"
Task: "T056 [US3] Add PHPUnit feature tests for account lifecycle authorization separation in schoolmaster-backend/tests/Feature/AccountLifecycle/AccountLifecycleAuthorizationTest.php"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup.
2. Complete Phase 2 foundation.
3. Complete Phase 3 invitation and password setup.
4. Validate US1 independently with OpenAPI checks and PHPUnit feature tests.
5. Stop before password reset or recovery behavior unless MVP acceptance is confirmed.

### Incremental Delivery

1. Deliver setup and shared foundation.
2. Deliver US1 invitation and password setup as MVP.
3. Deliver US2 password reset without changing US1 behavior.
4. Deliver US3 lock, recovery, and reactivation without changing US1 or US2 behavior.
5. Run Phase 6 cross-cutting verification before implementation merge.

### Contract-First Gate

No backend route from this task list may be exposed until its matching operation, schema, response envelope, authorization rule, tenant behavior, and error semantics are present in `schoolmaster-specs/api/openapi.yaml` and `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`.
