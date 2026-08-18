# Tasks: Duplicate Email Recovery

**Input**: Design documents from `specs/037-duplicate-email-recovery/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: Redocly contract validation plus PHPUnit unit, feature, response-shape,
atomicity, and real MySQL concurrency coverage are required. No frontend tests
are required because feature 037 has no frontend delivery.

**Organization**: Tasks are grouped by user story and ordered contract first,
backend second, verification last. Every duplicate path must preserve global
identity ownership and the privacy boundary defined by the feature contract.

## Phase 1: Setup (Contract and source of truth)

**Purpose**: Publish every changed response and canonical-email rule before
backend behavior changes.

- [ ] T001 [P] Add the exact `409 recoverable_user_conflict` envelope with only `user_id` and `recommended_action=restore` in `api/components/responses/users/RecoverableUserConflict.yaml`
- [ ] T002 [P] Add the reusable `422 validation_failed` unavailable-email example with `error.details.fields.email` array in `api/components/responses/users/UserCreationValidationError.yaml`
- [ ] T003 [P] Preserve existing invitation `conflict` variants and add the recoverable variant in `api/components/responses/account-lifecycle/AccountInvitationCreationConflict.yaml`
- [ ] T004 [P] Document trim/lowercase normalization, global retained ownership, storage behavior, and the 255-character limit in `api/components/schemas/users/UserCreateRequest.yaml`, `api/components/schemas/users/UserUpdateRequest.yaml`, and `api/components/schemas/account-lifecycle/AccountInvitationCreateRequest.yaml`
- [ ] T005 Wire the new response components and provisioning-specific descriptions into `api/paths/users/index.yaml` and `api/paths/account-lifecycle/invitations.yaml`
- [ ] T006 Bundle `api/openapi.yaml`, review the generated operations, and regenerate `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T007 Run Redocly lint for the aggregate and platform APIs and verify the documented status sets and exact safe bodies against `specs/037-duplicate-email-recovery/contracts/duplicate-email-recovery.md`

**Checkpoint**: Contract publishes direct-create 409/422 behavior, preserves all
invitation conflicts, and documents create/update normalization.

---

## Phase 2: Foundational (Shared identity and privacy boundaries)

**Purpose**: Establish shared normalization, lookup, audit, authorization, and
error primitives used by every story.

**⚠️ CRITICAL**: Complete before user-story integration.

- [ ] T008 [P] Write failing unit coverage for canonicalization, global `withTrashed` legacy-aware lookup through `LOWER(TRIM(email))`, current-user exclusion, collision decisions, and at most one ownership query per decision in `schoolmaster-backend/tests/Unit/Services/Users/IdentityEmailServiceTest.php`
- [ ] T009 Implement canonicalization, retained-owner lookup, collision classification, and exact `users_email_unique` identification in `schoolmaster-backend/app/Services/Users/IdentityEmailService.php`
- [ ] T010 [P] Write failing unit coverage for exactly one audit row, actor, resolved school or platform scope, outcome, source IP, workflow, reason code, SHA-256 canonical-email hash, conditional target UUID, strict metadata allowlist, and plaintext exclusion in `schoolmaster-backend/tests/Unit/Services/Users/DuplicateEmailAuditServiceTest.php`
- [ ] T011 Implement exactly-one `user_creation_duplicate_email` audit recording with scope, workflow, outcome, reason, source IP, and conditional affected user in `schoolmaster-backend/app/Services/Users/DuplicateEmailAuditService.php`
- [ ] T012 [P] Write failing tests for exact recovery-disclosure authorization and the minimal recovery error resource in `schoolmaster-backend/tests/Unit/Policies/AdministrationLifecyclePolicyTest.php` and `schoolmaster-backend/tests/Unit/ApiResponseTest.php`
- [ ] T013 Add the typed public-UUID recovery failure and render its exact safe 409 envelope while preserving generic validation formatting in `schoolmaster-backend/app/Exceptions/RecoverableUserConflictException.php`, `schoolmaster-backend/app/Http/Resources/ApiResponse.php`, and `schoolmaster-backend/bootstrap/app.php`
- [ ] T014 Add deny-by-default recovery-disclosure authorization matching effective school `users.lifecycle` and platform `schools.manage` restore authority in `schoolmaster-backend/app/Policies/AdministrationLifecyclePolicy.php`

**Checkpoint**: Shared primitives can distinguish disclosure-eligible recovery
from a generic unavailable email without disclosing hidden identity state.

---

## Phase 3: User Story 1 - Restore Retained Identity (Priority: P1) 🎯 MVP

**Goal**: Direct and platform-provisioning creation reject an authorized,
same-scope soft-deleted owner with safe recovery guidance; the original user is
then restored and updated only through existing workflows.

**Independent Test**: Soft-delete an in-scope user, create with the same email,
assert exact 409/no partial state/one safe audit, then restore the returned UUID
and update permitted status, roles, and profile while retaining identity and
history.

### Tests for User Story 1

> Write these tests first and confirm they fail before implementation.

- [ ] T015 [P] [US1] Add direct-create recovery, exact minimal 409 body, no duplicate or role state, exact target-bearing audit fields, explicit restore, permitted follow-up update, and post-409 dependency, uniqueness, or lifecycle restore rejection coverage in `schoolmaster-backend/tests/Feature/Api/V1/UserDuplicateEmailRecoveryTest.php`
- [ ] T016 [P] [US1] Add platform-invitation recoverable conflict, exact platform audit fields, and no user, role, invitation, credential, delivery request, or email submission coverage in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationDuplicateEmailTest.php`

### Implementation for User Story 1

- [ ] T017 [US1] Resolve school scope and create authorization before lookup, pass source IP, emit/audit authorized recovery, and keep user-plus-role persistence atomic in `schoolmaster-backend/app/Services/Users/UserService.php` and `schoolmaster-backend/app/Http/Controllers/Api/V1/UserController.php`
- [ ] T018 [US1] Replace the invitation-specific owner lookup with the shared identity decision and emit/audit recovery only in the platform provisioning branch in `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationService.php` and `schoolmaster-backend/app/Repositories/AccountLifecycleRepository.php`
- [ ] T019 [US1] Run the direct and invitation recovery tests, including the existing restore/update follow-up, and record the P1 checkpoint in `specs/037-duplicate-email-recovery/quickstart.md`

**Checkpoint**: The retained user remains the sole identity; recovery is
explicit, authorized, and independently testable.

---

## Phase 4: User Story 2 - Reject Duplicates Without Disclosure (Priority: P2)

**Goal**: Every collision that is hidden or not eligible for recovery disclosure
returns the identical generic 422 email validation result, persists nothing,
and records no target identity.

**Independent Test**: Exercise active, inactive, invited, cross-school,
opposite-scope, inaccessible, inactive-parent-scope, and unauthorized deleted
owners that are not eligible for recovery disclosure;
assert byte/shape-equivalent 422 responses, no identifiers or lifecycle facts,
no partial writes, and one target-free audit per attempt.

### Tests for User Story 2

> Write these tests first and confirm they fail before implementation.

- [ ] T020 [P] [US2] Add the complete direct-create lifecycle, cross-tenant, opposite-scope, inactive-parent-scope, and missing-restore-authority privacy matrix with exact 422 and exact target-free audit field assertions in `schoolmaster-backend/tests/Feature/Api/V1/UserDuplicateEmailRecoveryTest.php`
- [ ] T021 [P] [US2] Add invitation-provisioning generic collision, preserved existing 409 cases, response equivalence, exact target-free audit fields, audit secrecy, and full rollback assertions in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationDuplicateEmailTest.php`

### Implementation for User Story 2

- [ ] T022 [US2] Convert every direct retained-owner decision not eligible for recovery disclosure under FR-006 to `ValidationException` with only `email => [The email is unavailable.]` and a target-free audit in `schoolmaster-backend/app/Services/Users/UserService.php`
- [ ] T023 [US2] Apply the same generic decision and atomic rejection to platform invitation provisioning while preserving school invitation eligibility and unrelated conflicts in `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationService.php`
- [ ] T024 [US2] Run the direct and invitation privacy matrices and record response-equivalence, audit-secrecy, and atomicity evidence in `specs/037-duplicate-email-recovery/quickstart.md`

**Checkpoint**: Hidden identity states and scopes are externally
indistinguishable and no rejected workflow leaves partial state.

---

## Phase 5: User Story 3 - Preserve Uniqueness Under Equivalent and Concurrent Input (Priority: P3)

**Goal**: Equivalent email forms share one owner, future writes are canonical,
and MySQL race losers receive the documented generic 422 after full rollback.

**Independent Test**: Submit exact, case-varied, and whitespace-varied email
values through create, provisioning, and update; then coordinate two MySQL
connections for the same unowned email and prove at most one commit, generic
loser responses, no partial state, and correct post-rollback audits.

### Tests for User Story 3

> Write these tests first and confirm they fail before implementation.

- [ ] T025 [P] [US3] Extend normalization tests for exact, case, whitespace, complete-value lowercase behavior, unchanged omitted legacy values, and normalized legacy ownership lookup in `schoolmaster-backend/tests/Unit/Services/Users/IdentityEmailServiceTest.php`
- [ ] T026 [P] [US3] Add canonical storage, equivalent-owner, and 255-character validation coverage across direct create, platform invitation provisioning, and future email update in `schoolmaster-backend/tests/Feature/Api/V1/UserDuplicateEmailRecoveryTest.php`, `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationDuplicateEmailTest.php`, and `schoolmaster-backend/tests/Feature/Api/V1/AdministrationLifecycle/UserDetailUpdateTest.php`
- [ ] T027 [US3] Add deterministic exact-index translation, unrelated-index rethrow, retained-owner concurrent rejection, and barrier-controlled two-connection MySQL race coverage in `schoolmaster-backend/tests/Feature/Api/V1/UserDuplicateEmailConcurrencyTest.php`

### Implementation for User Story 3

- [ ] T028 [US3] Normalize before validation, enforce the 255-character limit, and normalize defensively before persistence for both creation workflows in `schoolmaster-backend/app/Http/Requests/Api/V1/CreateUserRequest.php`, `schoolmaster-backend/app/DTOs/Users/CreateUserData.php`, `schoolmaster-backend/app/Http/Requests/AccountLifecycle/CreateAccountInvitationRequest.php`, and `schoolmaster-backend/app/DTOs/AccountLifecycle/CreateAccountInvitationData.php`
- [ ] T029 [US3] Normalize future email updates, enforce the 255-character limit, exclude the current target from global retained ownership, leave omitted legacy values untouched, and translate email uniqueness collisions to generic 422 in `schoolmaster-backend/app/Http/Requests/AdministrationLifecycle/UpdateUserLifecycleRequest.php` and `schoolmaster-backend/app/Services/AdministrationLifecycle/AdministrationUpdateService.php`
- [ ] T030 [US3] Catch only `UniqueConstraintViolationException` for `users_email_unique` outside the complete user/role transaction, audit after rollback with `persistence_conflict`, and return generic 422 in `schoolmaster-backend/app/Services/Users/UserService.php`
- [ ] T031 [US3] Catch only `UniqueConstraintViolationException` for `users_email_unique` outside the complete provisioned-user/role/invitation transaction, audit after rollback, and preserve unrelated exceptions in `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationService.php`
- [ ] T032 [US3] Run normalization, exact-index, rollback, and repeated two-connection MySQL race tests and record the P3 checkpoint in `specs/037-duplicate-email-recovery/quickstart.md`

**Checkpoint**: New and changed emails are canonical, legacy owners remain
reserved, and concurrent creation cannot produce duplicate identities or 500s.

---

## Phase 6: Polish & Cross-Cutting Verification

**Purpose**: Prove contract synchronization, regression safety, privacy, and
release readiness across specification and backend repositories.

- [ ] T033 Run all focused feature 037 PHPUnit commands from `specs/037-duplicate-email-recovery/quickstart.md` against MySQL in `schoolmaster-backend/`
- [ ] T034 Run the full PHPUnit suite and Laravel Pint check in `schoolmaster-backend/`
- [ ] T035 Rebundle `api/openapi.yaml`, regenerate `specs/001-schoolmaster-platform/contracts/openapi.yaml`, and run Redocly lint for both named APIs
- [ ] T036 Audit responses, logs, exceptions, and serialized audit metadata for plaintext email or unauthorized target leakage and document findings in `specs/037-duplicate-email-recovery/quickstart.md`
- [ ] T037 Confirm no migration, backfill, frontend change, automatic restore, or undocumented API was introduced by reviewing `specs/037-duplicate-email-recovery/spec.md`, `specs/037-duplicate-email-recovery/plan.md`, and implementation diffs
- [ ] T038 Record focused/full test, Pint, Redocly, concurrency repetition, compatibility, and deployment evidence in `specs/037-duplicate-email-recovery/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 (Contract)**: No dependencies; blocks backend behavior changes.
- **Phase 2 (Foundation)**: Depends on Phase 1; blocks all user-story integration.
- **Phase 3 (US1)**: Depends on Phase 2 and is the MVP.
- **Phase 4 (US2)**: Depends on Phase 2. It is independently testable, but the
  recommended sequence follows US1 because both stories extend the same
  creation services.
- **Phase 5 (US3)**: Depends on Phase 2. Run after US1/US2 in the recommended
  sequence because it hardens the same persistence boundaries.
- **Phase 6 (Polish)**: Depends on all stories selected for delivery.

### User Story Dependencies

- **US1 (P1)**: No dependency on another story after the shared foundation.
- **US2 (P2)**: No behavioral dependency on US1; its generic outcome can be
  tested using only owners that are not eligible for recovery disclosure.
- **US3 (P3)**: No behavioral dependency on US1 or US2; its independent proof is
  canonical storage plus MySQL arbitration, though shared service files make
  sequential integration safer.

### Within Each User Story

- Author tests and confirm meaningful failures before matching implementation.
- Resolve authentication, tenant mode, and scope before global owner disclosure.
- Keep audits outside transactions that reject or roll back.
- Preserve the unique index as final race authority; never catch unrelated
  database failures as email validation.
- Complete the story checkpoint before moving to the next priority.

### Parallel Opportunities

- T001-T004 can run in parallel because they create or update separate contract
  components and schemas.
- T008, T010, and T012 can run in parallel before T009, T011, and T013.
- T015 and T016 can be authored in parallel.
- T020 and T021 can be authored in parallel.
- T025 and T026 can be authored in parallel before T028-T031.
- After Phase 2, story test authoring can proceed in parallel, but implementation
  tasks touching `UserService.php` or `AccountInvitationService.php` must be
  serialized or coordinated to avoid conflicting edits.

## Parallel Example: User Story 1

```text
Task: T015 direct recovery and restore-follow-up tests
Task: T016 platform invitation recovery and atomicity tests
```

## Parallel Example: User Story 2

```text
Task: T020 direct privacy matrix tests
Task: T021 invitation privacy and rollback tests
```

## Parallel Example: User Story 3

```text
Task: T025 identity-email normalization unit tests
Task: T026 workflow canonical-storage feature tests
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 contract publication.
2. Complete Phase 2 shared backend primitives.
3. Deliver Phase 3 US1 recovery guidance and explicit restore/update proof.
4. Stop and validate P1 independently before adding broader duplicate states.

### Incremental Delivery

1. Contract + foundation: publish safe response and identity boundaries.
2. US1: authorized retained identity recovery path.
3. US2: non-disclosing generic outcomes for every other collision.
4. US3: canonical input and database-race hardening.
5. Polish: full gates, privacy review, and release evidence.

### Repository Sequence

1. `schoolmaster-specs`: modular and bundled OpenAPI contract.
2. `schoolmaster-backend`: shared services, adapters, tests, and quality gates.
3. `schoolmaster-specs`: final verification evidence.

## Notes

- `[P]` means different files or no dependency on unfinished work.
- `[US1]`, `[US2]`, and `[US3]` map tasks to specification stories.
- No frontend, database migration, bulk email rewrite, new endpoint, package,
  permanent deletion, or automatic restoration belongs in feature 037.
- Mark a task `[X]` only after its implementation and verification evidence
  exist.
