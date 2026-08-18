# Implementation Plan: Duplicate Email Recovery

**Branch**: `037-duplicate-email-recovery` | **Date**: 2026-08-18 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/037-duplicate-email-recovery/spec.md`

## Summary

Preserve one globally unique user identity per email across active and
soft-deleted records while replacing database-level duplicate failures with
documented, privacy-safe outcomes. OpenAPI leads with a scoped
`409 recoverable_user_conflict` and a generic `422 validation_failed` variant.
Laravel then centralizes trim/lowercase normalization, legacy-aware ownership
lookup, restore-authority classification, post-rollback unique-index handling,
and allowlisted audit recording across direct user creation and platform user
provisioning through account invitations. Existing restore and update workflows
remain explicit; no migration, bulk email rewrite, new package, endpoint, or
frontend delivery is required.

## Technical Context

**Language/Version**: PHP 8.3.31 with Laravel 13.15.0  
**Primary Dependencies**: Existing Eloquent `SoftDeletes`, Form Requests, DTOs, administration policy and tenant services, database transactions, `UniqueConstraintViolationException`, API response helpers, and audit services; no new package  
**Storage**: Existing MySQL `users` table with case-insensitive global `users_email_unique` index plus existing `audit_events`; no migration and no bulk normalization  
**Testing**: Redocly OpenAPI bundle/lint; PHPUnit 12.5 feature, unit, response-shape, transaction, and MySQL concurrency tests; Laravel Pint  
**Target Platform**: Linux-hosted Laravel JSON API exposing versioned `/api/v1` operations  
**Project Type**: Multi-repository web service: shared specification/OpenAPI repository and Laravel API repository; Vue SPA unchanged  
**Performance Goals**: Preserve indexed uniqueness arbitration; add at most one ownership lookup to each creation decision and one audit write only for rejected duplicate attempts  
**Constraints**: Contract before backend; authorization and tenant mode before disclosure; target UUID only for an authorized same-scope soft-deleted user eligible for recovery disclosure; generic responses byte/shape-equivalent across hidden states; no plaintext email in duplicate audits; persistence races become generic 422 after rollback; existing email rows remain unchanged until updated  
**Scale/Scope**: Two user-provisioning paths, one user-email update path, one shared identity-email service, one focused audit service, one typed conflict exception, three reusable response components, and focused regression/concurrency coverage

## Constitution Check

*GATE: PASS before Phase 0; re-checked and PASS after Phase 1 design.*

- PASS: Modular and aggregate OpenAPI changes define the new 409 variant,
  exact generic 422 example, normalization semantics, and affected operations
  before backend behavior changes.
- PASS: `schoolmaster-specs` leads contract/design work; `schoolmaster-backend`
  implements next under feature identifier 037. `schoolmaster-frontend` has no
  required change and remains independently releasable.
- PASS: Existing controllers remain orchestration-only. Form Requests normalize
  transport input, DTOs carry validated data, focused services own identity and
  audit rules, `AdministrationLifecyclePolicy` owns recovery-disclosure
  authorization, API response helpers normalize output, and domain exceptions
  carry failures. No new repository is justified for one indexed ownership
  lookup; the obsolete account-lifecycle-specific lookup is retired.
- PASS: Public recovery references remain user UUIDs. Internal numeric actor and
  school identifiers remain inside persistence/audit boundaries.
- PASS: Frontend architecture requirements are not applicable because this
  feature adds no frontend behavior, route, store, service, or component.
- PASS: MySQL remains authoritative. Global email ownership includes retained
  rows, school/platform mode is preselected before disclosure, and opposite
  scope or cross-tenant collisions remain generic.
- PASS: Compatibility is additive: direct user creation gains a documented 409;
  account invitation preserves existing conflict cases while adding one code;
  generic duplicates remain 422 and successful envelopes remain unchanged.
- PASS: Redocly, PHPUnit response/tenant/privacy/atomicity/concurrency coverage,
  full backend regression, and Pint cover all changed critical flows. Vitest is
  not required because the frontend repository is unchanged.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/037-duplicate-email-recovery/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── duplicate-email-recovery.md
└── tasks.md                         # Created later by /speckit-tasks
```

### Source Code (target repositories)

```text
schoolmaster-specs/
├── api/
│   ├── components/
│   │   ├── responses/
│   │   │   ├── account-lifecycle/AccountInvitationCreationConflict.yaml
│   │   │   └── users/
│   │   │       ├── RecoverableUserConflict.yaml
│   │   │       └── UserCreationValidationError.yaml
│   │   └── schemas/
│   │       ├── account-lifecycle/AccountInvitationCreateRequest.yaml
│   │       └── users/
│   │           ├── UserCreateRequest.yaml
│   │           └── UserUpdateRequest.yaml
│   └── paths/
│       ├── account-lifecycle/invitations.yaml
│       └── users/index.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
└── specs/037-duplicate-email-recovery/

schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   ├── AccountLifecycle/CreateAccountInvitationData.php
│   │   └── Users/CreateUserData.php
│   ├── Exceptions/RecoverableUserConflictException.php
│   ├── Http/
│   │   ├── Controllers/Api/V1/UserController.php
│   │   ├── Requests/
│   │   │   ├── AccountLifecycle/CreateAccountInvitationRequest.php
│   │   │   ├── AdministrationLifecycle/UpdateUserLifecycleRequest.php
│   │   │   └── Api/V1/CreateUserRequest.php
│   │   └── Resources/ApiResponse.php
│   ├── Policies/AdministrationLifecyclePolicy.php
│   ├── Repositories/AccountLifecycleRepository.php
│   └── Services/
│       ├── AccountLifecycle/AccountInvitationService.php
│       ├── AdministrationLifecycle/AdministrationUpdateService.php
│       └── Users/
│           ├── DuplicateEmailAuditService.php
│           ├── IdentityEmailService.php
│           └── UserService.php
├── bootstrap/app.php
└── tests/
    ├── Feature/
    │   ├── AccountLifecycle/AccountInvitationDuplicateEmailTest.php
    │   └── Api/V1/
    │       ├── AdministrationLifecycle/UserDetailUpdateTest.php
    │       ├── UserDuplicateEmailConcurrencyTest.php
    │       └── UserDuplicateEmailRecoveryTest.php
    └── Unit/
        ├── ApiResponseTest.php
        ├── Policies/AdministrationLifecyclePolicyTest.php
        └── Services/Users/
            ├── DuplicateEmailAuditServiceTest.php
            └── IdentityEmailServiceTest.php
```

**Structure Decision**: Extend existing user, account-lifecycle,
administration-lifecycle, exception-rendering, and audit boundaries. A shared
identity-email service owns canonicalization and collision classification so
direct creation, invitation provisioning, and updates cannot drift. A separate
allowlisted audit service prevents plaintext or hidden identity metadata from
reaching generic audit paths. No frontend tree is included because no client
implementation is part of feature 037.

## Component Map

| Component | Responsibility | Inputs / outputs |
|---|---|---|
| `IdentityEmailService` | Normalize identity emails, perform global legacy-aware retained-owner lookup, classify creation collisions, and identify the email unique index | Raw email, actor, resolved context, workflow in; normalized email or domain failure out |
| `AdministrationLifecyclePolicy` | Decide whether actor may receive same-scope deleted-user recovery disclosure using exact restore permissions | Actor and retained user in; boolean out |
| `DuplicateEmailAuditService` | Create exactly one allowlisted duplicate-attempt audit outside rolled-back transactions | Actor, scope, workflow, outcome, normalized email, optional authorized target, source IP in; audit event out |
| `RecoverableUserConflictException` | Carry public target UUID and constant recommended action to global renderer | Authorized target UUID in; documented 409 out |
| `UserService` | Orchestrate school user creation, roles, transaction, collision handling, and final unique-index translation | Validated DTO/context/actor/IP in; user or documented failure out |
| `AccountInvitationService` | Preserve invitation lifecycle while applying the same rules only when platform invitation provisioning would create a user | Validated DTO/context/actor/IP in; invitation or documented failure out |
| `AdministrationUpdateService` | Normalize future email changes and preserve global retained ownership without bulk-rewriting legacy rows | Validated update and target user in; updated user or generic validation failure out |

## Implementation Approach

### Phase 0: Contract and source-of-truth alignment

- Add specialized reusable responses for direct recoverable conflict, user
  creation validation with exact unavailable-email example, and invitation
  creation conflicts that preserve existing lifecycle cases.
- Update direct user creation and account invitation descriptions, responses,
  and email request-schema normalization rules. Document update normalization
  without changing restore/update operation shapes.
- Regenerate the platform OpenAPI mirror from the modular aggregate, then lint
  both named APIs before backend changes.

### Phase 1: Backend identity decision and privacy boundaries

- Normalize submitted email with trim plus lowercase before validation and
  defensively at the shared service boundary. Enforce the existing 255-character
  storage limit in every affected request. New and updated values are stored
  canonical; existing rows are not rewritten.
- Query retained ownership globally with a legacy-aware normalized comparison.
  Tenant mode and creation authorization resolve first. A deleted target is
  eligible for recovery disclosure only in the exact preselected active scope
  when the actor has the effective restore permission (`users.lifecycle` for
  school users or `schools.manage` for platform users); all other states remain
  generic. The 409 does not pre-approve restoration: the explicit restore action
  re-evaluates dependency, uniqueness, lifecycle, reason, and effective-date
  constraints against current state.
- Add typed recovery exception rendering through `ApiResponse` with only
  `user_id` and `recommended_action=restore` in details. Generic failures use
  Laravel validation errors with `fields.email` array.
- Keep the existing MySQL unique index as final concurrency authority. Catch
  only `UniqueConstraintViolationException` for `users_email_unique` outside
  the full persistence transaction; after rollback, audit and return generic
  422 without reclassifying current target state.
- Record one `user_creation_duplicate_email` audit per rejected creation using
  actor, resolved school/platform scope, workflow, outcome, source IP, and a
  SHA-256 hash of the canonical email. Record affected user UUID only for an
  authorized recoverable result. Construct metadata from an allowlist rather
  than relying on shallow global sanitization.
- Preserve explicit restore, activation, role assignment, and profile update
  operations. Duplicate creation never auto-restores and invitation rejection
  never persists partial user, role, invitation, setup credential, or mail
  delivery state.

### Phase 2: Verification and rollout

- Cover lifecycle states, case/whitespace variants, the 255-character request
  limit, legacy mixed-case rows, and the one-ownership-query performance bound,
  same-scope restore authority, missing restore authority, cross-tenant and
  opposite-mode privacy, platform invitation provisioning, update
  normalization, every required audit field, conditional audit target UUID, and
  plaintext exclusion.
- Add deterministic unique-index translation tests and a barrier-controlled
  two-connection MySQL integration test proving at most one concurrent creation
  succeeds and race losers receive the canonical 422 after rollback.
- Run focused and full PHPUnit, Pint, modular/aggregate OpenAPI bundle and lint,
  and response-shape review. Deploy contract first, backend second; no data
  migration or coordinated frontend release is needed.

## Complexity Tracking

No constitution violations.
