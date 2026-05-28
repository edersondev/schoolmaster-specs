# Quickstart: Backend Account Lifecycle Workflows

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-account-lifecycle.md](./contracts/backend-account-lifecycle.md) for the proposed operation boundary and contract expansion requirements.
- Expand `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` before backend implementation exposes account lifecycle behavior.

## Delivery Boundary

## Feature-to-Repository Implementation Notes

- Run `/speckit-implement` from the `schoolmaster-backend` repository root.
- Backend implementation paths in `tasks.md` are relative to `schoolmaster-backend`.
- Specification and OpenAPI paths use the backend repository's `specs` symlink, which points to `../schoolmaster-specs`.
- Contract changes must be made in both `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` before the matching backend routes are exposed.

Implement only OpenAPI-approved operations for:

- account invitation creation for platform users by platform account administrators
- account invitation creation for same-school users by school user administrators
- invitation resend or replacement where OpenAPI approves it
- invitation completion and first password setup
- password reset request with non-enumerating response behavior
- password reset completion
- account lock review or mutation where OpenAPI approves it
- account unlock, recovery, or reactivation where OpenAPI approves it
- bearer-token revocation side effects required by the contract

Do not implement classroom/course/section/roster workflows, teacher correction workflows, guardian self-service, report lifecycle expansion, platform support-user school-owned access, frontend behavior, provider-specific email delivery, SMS delivery, messaging, notification preferences, delivery templates, parent portal communication, permanent purge, anonymization, billing, or undocumented APIs until the specification and OpenAPI contracts are expanded.

## Validation Walkthrough

### 1. Contract readiness

- Confirm OpenAPI contains approved operation IDs for every account lifecycle operation implemented.
- Confirm every implemented route uses the documented `/api/v1` path, request schema, parameters, response status, and content type.
- Confirm invitation tokens document 7-day expiry, single-use completion, token hashing expectations, and supersession of prior unused invitation tokens for the same user and scope.
- Confirm invitation sends/resends document the 3-per-user-and-scope-per-24-hours limit.
- Confirm invitation completion documents revocation after 5 failed invitation-token completion attempts per invitation, account, or IP within 15 minutes.
- Confirm password reset tokens document 30-minute expiry, single-use completion, token hashing expectations, and supersession of prior unused reset tokens for the same user and scope.
- Confirm password setup and reset completion document 12-to-128 character length, common-password rejection, and paste/password-manager compatibility.
- Confirm password reset request behavior documents non-enumerating responses.
- Confirm password reset request throttling documents 3 requests per account or IP within 1 hour, with over-limit requests returning the same non-enumerating accepted envelope while creating no token or email delivery request.
- Confirm reset-token failure suppression documents 5 failed reset-token completions per account or IP within 15 minutes and 15-minute new-token suppression.
- Confirm bearer-token revocation is documented after password reset completion, administrative lock, unlock/recovery, reactivation, and user deactivation.
- Confirm no backend route exposes fields, filters, sorts, status codes, error responses, status values, token modes, lifecycle actions, or authorization exceptions absent from OpenAPI.

### 2. Tenant and authorization validation

- Confirm school-scoped account lifecycle requests resolve an active `school_id` context before target user lookup, token lookup, authorization, recovery, lock, or persistence logic.
- Confirm missing, inactive, mismatched, and unauthorized tenant contexts fail before service logic runs.
- Confirm platform account administrators can manage platform users only.
- Confirm school user administrators can manage same-school users only.
- Confirm platform access does not imply school-owned account lifecycle access.
- Confirm teachers, students, and guardians cannot use account lifecycle administration operations unless a future contract grants that behavior.

### 3. Invitation and password setup validation

- Confirm invitation creation accepts only documented target identity, account scope, role/scope inputs, and delivery metadata.
- Confirm inactive schools, inactive roles, cross-tenant roles, inactive or deleted users, unsupported actors, duplicate active invitations where prohibited, and undocumented fields are rejected.
- Confirm creating or resending an invitation supersedes prior unused invitations of the same type for that user and scope.
- Confirm invitation sends/resends beyond 3 attempts per user and scope within 24 hours are rejected without creating a new token.
- Confirm invitation completion rejects expired, reused, superseded, malformed, missing, mismatched, revoked, or scope-incompatible tokens.
- Confirm invitation completion revokes the invitation after 5 failed completion attempts per invitation, account, or IP within 15 minutes.
- Confirm invitation completion validates that the submitted password is 12 to 128 characters, not a common password, and compatible with paste/password-manager input.
- Confirm invitation completion does not expose plaintext passwords, reusable tokens, credential hashes, or delivery-provider secrets.

### 4. Password reset validation

- Confirm reset requests return non-enumerating responses for missing, inactive, locked, deleted, unauthorized, and eligible accounts.
- Confirm password reset requests beyond 3 per account or IP within 1 hour return the same non-enumerating accepted envelope and create no reset token or email delivery request.
- Confirm reset token creation supersedes prior unused reset tokens for the same user and scope.
- Confirm reset completion rejects expired, reused, superseded, malformed, missing, mismatched, revoked, or scope-incompatible tokens.
- Confirm reset completion validates that the submitted replacement password is 12 to 128 characters, not a common password, and compatible with paste/password-manager input.
- Confirm reset completion changes credentials and revokes all active bearer tokens atomically.
- Confirm invalid reset-token completion attempts leave credentials unchanged.
- Confirm 5 failed reset-token completions per account or IP within 15 minutes block new reset token creation for 15 minutes without locking the whole account.

### 5. Lock, recovery, and reactivation validation

- Confirm failed-login lockout rules from the backend foundation remain preserved.
- Confirm administrative lock, unlock, recovery, and reactivation accept only documented transitions.
- Confirm administrative locks remain active until authorized unlock or recovery clears them.
- Confirm administrative lock, unlock/recovery, reactivation, and user deactivation revoke all active bearer tokens.
- Confirm recovery rejects inactive schools, soft-deleted users, missing or inactive role dependencies, unresolved invitations, unresolved password setup, cross-tenant targets, and platform/school authorization mismatches.
- Confirm reactivation allows authentication to resume only when user lifecycle state, school lifecycle state, role assignments, password setup state, lock state, and tenant or platform scope are valid.

### 6. Audit and secret handling validation

- Confirm account lifecycle events are audited with actor where available, target user where known, school where applicable, event type, outcome, timestamp, IP or request metadata where allowed, and tenant-safe metadata.
- Confirm audit events cover invitation creation, resend or replacement, over-limit invitation send/resend attempts, invitation completion, failed invitation-token completion, invitation revocation, password reset request acceptance, over-limit password reset request suppression, password reset completion, failed token completion, reset-token suppression, administrative lock, account unlock, recovery, reactivation, blocked inactive-account attempts, and bearer-token revocation outcomes.
- Confirm responses, logs, audit events, and delivery metadata do not store or expose plaintext passwords, reusable invitation tokens, reset tokens, credential hashes, delivery-provider secrets, private content, or unauthorized cross-tenant details.

### 7. Response-shape validation

- Confirm successful JSON responses use the documented success envelopes.
- Confirm non-enumerating reset request responses use the documented accepted envelope.
- Confirm validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, token-invalid, and not-found cases use only the error envelopes or codes documented on each affected OpenAPI operation.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Implementation PRs should record the contract validation result, test result, feature id `008-account-lifecycle-workflows`, and the operation IDs implemented.

## Implementation Verification Results

Recorded on 2026-05-27 for feature `008-account-lifecycle-workflows`:

- Contract validation: `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` passed for `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml`.
- Backend validation: `docker exec schoolmaster-backend-app-1 php artisan test` passed with 178 tests and 810 assertions.
- Route mapping: implemented account lifecycle routes map to `createAccountInvitation`, `resendAccountInvitation`, `completeAccountInvitation`, `requestPasswordReset`, `completePasswordReset`, `getAccountLock`, `lockAccount`, `unlockAccount`, and `reactivateAccount`.
- Cross-cutting verification: feature tests cover secret non-exposure, response envelope consistency, account lifecycle authorization separation, administrative lock durability, reactivation eligibility, and bearer-token revocation after user deactivation.

## Verified Out-of-Scope Behavior

Implementation remains limited to account lifecycle workflows approved by OpenAPI. No frontend behavior, provider-specific delivery integration, SMS or messaging behavior, classroom/course/roster workflows, report lifecycle expansion, platform support override for school-owned accounts, purge/anonymization/legal-hold behavior, billing behavior, or undocumented API routes were added.

## Exit Criteria for Planning

- The implementation boundary maps to approved OpenAPI operation IDs.
- Tenant and authorization rules are explicit for platform account administrators and school user administrators.
- Invitation, password setup, password reset, lock, recovery, reactivation, password policy, token expiry, token supersession, bearer-token revocation, invitation send/resend limits, invitation failed-completion revocation, password reset request throttling, reset-token suppression, non-enumerating responses, email delivery request metadata, and secret-handling behavior are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, tenant isolation, authorization, validation, inactive statuses, password policy, token expiry, token reuse, token supersession, invitation send/resend limits, invitation failed-completion revocation, password reset request throttling, reset-token suppression, administrative lock durability, bearer-token revocation, audit events, and credential-secret non-exposure.
