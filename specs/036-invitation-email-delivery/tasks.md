# Tasks: Invitation Email Delivery

**Input**: Design documents from `specs/036-invitation-email-delivery/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: Contract, PHPUnit, Vitest, Playwright, and build coverage are required
for this critical cross-repository onboarding flow.

**Organization**: Tasks are grouped by user story and ordered contract first,
backend second, frontend third.

## Phase 1: Setup (Contract and source of truth)

**Purpose**: Approve delivery semantics before implementation.

- [X] T001 Add reusable `503 temporary_unavailable` response in `api/components/responses/common/TemporaryUnavailable.yaml` and register it in `api/openapi.yaml`
- [X] T002 Update invitation email acceptance, failure, and token-secrecy behavior in `api/paths/account-lifecycle/invitations.yaml`, `specs/008-account-lifecycle-workflows/spec.md`, and `specs/008-account-lifecycle-workflows/contracts/backend-account-lifecycle.md`
- [X] T003 [P] Reconcile provider-delivery exclusions and UI default wording in `specs/008-account-lifecycle-workflows/research.md`, `specs/034-complete-account-lifecycle-ui/spec.md`, and `specs/034-complete-account-lifecycle-ui/plan.md`
- [X] T004 Bundle and validate updated aggregate contract in `specs/001-schoolmaster-platform/contracts/openapi.yaml`

---

## Phase 2: Foundational (Backend delivery boundary)

**Purpose**: Establish trusted configuration and standard failure handling.

**⚠️ CRITICAL**: Complete before invitation mail stories.

- [X] T005 Add trusted frontend origin configuration to `schoolmaster-backend/config/app.php` and `schoolmaster-backend/.env.example`
- [X] T006 [P] Add delivery-domain exception in `schoolmaster-backend/app/Exceptions/InvitationDeliveryException.php`
- [X] T007 Map delivery failure to standard secret-safe 503 response in `schoolmaster-backend/bootstrap/app.php` and `schoolmaster-backend/app/Http/Resources/ApiResponse.php`

**Checkpoint**: Backend can represent delivery configuration and retryable failure without provider detail.

---

## Phase 3: User Story 1 - Receive and complete account invitation (Priority: P1) 🎯 MVP

**Goal**: Send one secure setup email and complete setup/login through existing route.

**Independent Test**: Create invitation, inspect one target email and trusted link,
complete setup, reject reuse, then log in.

### Tests for User Story 1

- [X] T008 [P] [US1] Add rendered content, recipient, trusted URL, expiry, and escaping tests in `schoolmaster-backend/tests/Unit/Mail/AccountInvitationMailTest.php`
- [X] T009 [P] [US1] Add send, stored-hash, response secrecy, setup completion, reuse rejection, and login coverage in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationCreationTest.php`
- [X] T010 [P] [US1] Extend browser email-link journey fixture in `schoolmaster-frontend/e2e/account-lifecycle.spec.js`

### Implementation for User Story 1

- [X] T011 [P] [US1] Create transactional mailable in `schoolmaster-backend/app/Mail/AccountInvitationMail.php`
- [X] T012 [P] [US1] Create escaped setup message in `schoolmaster-backend/resources/views/mail/account-invitation.blade.php`
- [X] T013 [US1] Build trusted setup URL and synchronous mail submission in `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationDeliveryService.php`
- [X] T014 [US1] Retain plaintext token only through delivery, defer accepted metadata, and preserve lifecycle rules in `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationService.php`

**Checkpoint**: User Story 1 works without frontend default change.

---

## Phase 4: User Story 2 - Default new users to invitation setup (Priority: P1)

**Goal**: Fresh administrator user forms explicitly choose invitation while API omission stays active.

**Independent Test**: Open fresh form, verify no setup choice is rendered,
submit identity and role details, and verify explicit invitation payload and
separate invitation action.

### Tests for User Story 2

- [X] T015 [P] [US2] Update form-default and request-mapping tests in `schoolmaster-frontend/tests/unit/admin-system/administration/contracts/access.contract.spec.js` and `schoolmaster-frontend/tests/unit/admin-system/administration/services/users.spec.js`
- [X] T016 [P] [US2] Update default create-then-invite browser expectations in `schoolmaster-frontend/e2e/account-lifecycle.spec.js`

### Implementation for User Story 2

- [X] T017 [US2] Remove the account-setup selector and form state, then enforce invitation mapping in `schoolmaster-frontend/src/components/admin-system/users/UserForm.vue`, `schoolmaster-frontend/src/pages/admin-system/users/CreateUserPage.vue`, and `schoolmaster-frontend/src/contracts/admin-system/users.js`

**Checkpoint**: UI creation is invitation-only; API compatibility remains.

---

## Phase 5: User Story 3 - Recover safely from delivery failure (Priority: P2)

**Goal**: Report transport failure accurately and permit secret-safe replacement.

**Independent Test**: Force mail failure, receive 503 with unset delivery
metadata, retry successfully, and verify old pending invitation is superseded.

### Tests for User Story 3

- [X] T018 [P] [US3] Add transport/configuration failure, safe envelope, audit secrecy, and successful retry coverage in `schoolmaster-backend/tests/Feature/AccountLifecycle/AccountInvitationDeliveryFailureTest.php`
- [X] T019 [P] [US3] Add frontend normalized 503 feedback coverage in `schoolmaster-frontend/tests/unit/account-lifecycle/services/adminAccountLifecycle.service.test.js`

### Implementation for User Story 3

- [X] T020 [US3] Convert configuration and mail transport failures into `InvitationDeliveryException` and record secret-free failure audit in `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationDeliveryService.php` and `schoolmaster-backend/app/Services/AccountLifecycle/AccountInvitationService.php`
- [X] T021 [US3] Preserve normalized temporary failure display without token or provider detail in `schoolmaster-frontend/src/services/admin-system/administration-error-mapper.js` and `schoolmaster-frontend/src/locales/account-lifecycle.js`

**Checkpoint**: All stories independently functional.

---

## Phase 6: Polish & Cross-Cutting Verification

**Purpose**: Synchronize evidence and run release gates.

- [X] T022 Update implementation evidence and deployment checks in `specs/036-invitation-email-delivery/quickstart.md`, `docs/backend-feature-roadmap.md`, and `docs/frontend-feature-roadmap.md`
- [X] T023 Run Redocly lint/bundle for `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T024 Run focused/full PHPUnit and Pint in `schoolmaster-backend/`
- [X] T025 Run focused/full Vitest, Playwright `e2e/account-lifecycle.spec.js`, and production build in `schoolmaster-frontend/`
- [X] T026 Validate token absence from repository-visible response, log, audit, metadata, and queue assertions; record results in `specs/036-invitation-email-delivery/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- Phase 1 contract work blocks backend behavior.
- Phase 2 blocks US1 and US3 backend work.
- US1 and US2 can proceed independently after their foundations; US3 extends US1 delivery.
- Phase 6 depends on all desired stories.

### User Story Dependencies

- **US1**: Depends on Phases 1-2; no US2 dependency.
- **US2**: Depends on Phase 1 compatibility decision; no backend delivery dependency.
- **US3**: Depends on US1 delivery service but is independently failure-testable.

### Parallel Opportunities

- T003 can run alongside T001-T002.
- T006 can run alongside T005.
- T008-T010 can be authored in parallel before T011-T014.
- T011 and T012 can run in parallel.
- T015 and T016 can run in parallel.
- T018 and T019 can run in parallel.

## Parallel Example: User Story 1

```text
Task: T008 mailable content tests
Task: T009 backend invitation journey tests
Task: T010 browser journey fixture
```

## Implementation Strategy

### MVP First

1. Complete contract and foundational phases.
2. Deliver US1 email/setup/login journey.
3. Validate secret handling and mail failure boundary.
4. Add US2 default and US3 retry behavior.

### Repository Sequence

1. `schoolmaster-specs`: contract and rules.
2. `schoolmaster-backend`: mail, exception, service, PHPUnit.
3. `schoolmaster-frontend`: default and feedback tests.
4. All repositories: full gates and evidence.

## Notes

- `[P]` means different files or no dependency on unfinished task.
- Tests precede matching implementation.
- No new endpoint, package, database migration, queue payload, or token response.
- Mark each task `[X]` only after implementation and verification evidence exists.
