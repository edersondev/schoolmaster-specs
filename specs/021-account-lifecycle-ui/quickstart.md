# Quickstart: Account Lifecycle Workflows UI

## Prerequisites

- Feature 017 Authentication and Session Foundation UI is implemented.
- Feature 018 Administration Foundation UI is implemented.
- Feature 020 Administration Lifecycle UI is implemented.
- Backend account lifecycle operations from `specs/008-account-lifecycle-workflows/`
  are deployed and contract-compliant.
- Implementation confirms approved permission codes or session capability flags
  before admin action visibility is enabled.
- Admin invitation resend remains blocked until OpenAPI provides non-secret
  resend by invitation or user identifier.

## Contract Review

Before frontend implementation:

1. Confirm these operations exist in `api/openapi.yaml`:
   - `createAccountInvitation`
   - `completeAccountInvitation`
   - `requestPasswordReset`
   - `completePasswordReset`
   - `getAccountLock`
   - `lockAccount`
   - `unlockAccount`
   - `reactivateAccount`
2. Confirm `resendAccountInvitation` still requires `invitationToken` and keep
   admin resend blocked unless a non-secret operation is added.
3. Confirm `requestPasswordReset` may be called with email only.
4. Confirm lock requires reason, unlock has no request body, and recovery or
   reactivation accepts optional reason.
5. Confirm invalid-token, validation, forbidden, tenant-mismatch, not-found,
   conflict, and neutral reset envelopes are documented for consumed actions.

## Manual Scenario Review

### Invitation Creation and Blocked Resend

- Sign in as an authorized platform or school account administrator.
- Open an approved user create/detail flow.
- Create an invitation for an eligible target user.
- Verify response shows status, expiry, and delivery request metadata.
- Verify request does not submit `delivery_metadata`.
- Verify no invitation token is visible or stored.
- Verify admin resend is hidden or blocked with clear contract-gap messaging.
- Verify over-limit send or future approved resend shows validation or conflict
  feedback without claiming email delivery.

### Invitation Setup

- Open invitation setup link while signed out.
- Submit too-short, too-long, and common-password values.
- Verify field-level validation and password-manager/paste compatibility.
- Submit a valid password.
- Verify success guides user to sign in, without automatic session creation.
- Reopen the same token and verify invalid-token or conflict feedback.

### Password Reset Request

- Open password reset request while signed out.
- Confirm only email input is visible.
- Submit valid email values for eligible, missing, inactive, locked, deleted,
  unauthorized, and over-limit accounts.
- Verify the same neutral confirmation appears for all outcomes.
- Verify no public school selector is shown and no school context is submitted.

### Password Reset Completion

- Open reset completion link while signed out.
- Submit invalid passwords and verify field-level validation.
- Submit valid replacement password.
- Verify success guides user to sign in.
- Verify expired, reused, superseded, revoked, malformed, missing, mismatched,
  and scope-incompatible tokens all render the safe invalid-token state.

### Account Lock and Recovery

- Open user detail as an authorized administrator.
- Verify account lifecycle actions are hidden until approved permission codes or
  session capability flags are confirmed.
- After confirmation, verify lock state loads only for permitted target users.
- Lock an eligible account with a required reason.
- Unlock an administratively locked account without a reason.
- Recover or reactivate an eligible account with optional reason.
- Verify inactive school, soft-deleted user, role dependency conflict,
  unresolved invitation, unresolved password setup, current-state mismatch,
  self-action denial, and platform-versus-school mismatch show safe conflict or
  denial feedback.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm test
```

Focused Vitest coverage should include:

- account lifecycle contract mappers
- guest account lifecycle services
- admin account lifecycle services
- invitation setup composable and page
- password reset request email-only behavior
- password reset completion composable and page
- invalid-token feedback mapping
- neutral reset confirmation privacy
- blocked admin resend state
- permission/capability-gated action visibility
- lock required reason and unlock no-reason behavior
- optional recovery/reactivation reason behavior
- stale response protection
- no-secret diagnostics

Run build checks if available:

```bash
npm run build
```

Run OpenAPI validation only if account lifecycle contracts change:

```bash
npx @redocly/cli lint api/openapi.yaml
```

## Acceptance Evidence

Record in implementation PR:

- Operation ID to UI surface mapping.
- Permission code or session capability source used for admin action visibility.
- Evidence that admin resend remains blocked or that a non-secret resend
  contract has been approved.
- Evidence that password reset request submits email only.
- Evidence that token values, plaintext passwords, reasons, tenant-private
  details, role internals, and permission payloads are absent from diagnostics.
- Responsive and keyboard review for 390px, 768px, and 1440px.

## Implementation Evidence - 2026-06-29

- Operation mapping implemented in `src/services/auth/accountLifecycle.js`, `src/services/admin-system/accountLifecycle.js`, `src/contracts/auth/account-lifecycle.js`, and `src/contracts/admin-system/account-lifecycle.js`.
- Permission/capability source: no approved exact account lifecycle permission code or session capability flag was found in the current OpenAPI/session contract; admin invitation/action visibility remains blocked by `ACCOUNT_LIFECYCLE_PERMISSION_SOURCE_CONFIRMED = false`.
- Admin resend remains blocked because `resendAccountInvitation` still requires `{invitationToken}`.
- Password reset request mapper submits email only; `school_id` and `delivery_metadata` are omitted.
- No-secret diagnostics implemented in `src/services/api/errorDiagnostics.js`; token, password, reason, tenant, role, and permission keys are redacted.
- Focused account lifecycle suite: `npm run test:unit -- tests/unit/account-lifecycle tests/unit/auth/AuthFeedbackState.test.js` passed, 28 files / 57 tests.
- Full frontend unit suite: `npm run test:unit` passed, 151 files / 276 tests. `package.json` has no `npm test` script; `test:unit` is the project unit-test command.
- Build check: `npm run build` passed. Build emitted existing third-party Rolldown pure-annotation warnings from `node_modules/@vueuse/core` and a large chunk warning.
- Responsive/static review: forms, token states, panels, and dialogs use constrained card widths, grid/flex layouts, full-width mobile buttons, and Element Plus dialog width `min(92vw, 520px)`/`min(92vw, 560px)` suitable for 390px, 768px, and 1440px review.
- Keyboard/accessibility review: forms use labels, native submit handling, password-manager-compatible inputs, status regions with `aria-live`, and Element Plus dialog focus behavior.
- Representative administrator usability review (SC-009) remains pending because no human administrator review was available in this execution.

## Feature 034 Completion Evidence - 2026-08-12

- Feature 034 supersedes the hardcoded blocked gate. Lifecycle authority now
  requires an active raw `account_lifecycle.manage` permission in the exact
  platform or school scope, or the exact active platform role name
  `System Administrator`. Flattened permission codes do not grant authority.
- Backend Policy enforcement occurs before tenant-scoped target lookup;
  self-targets are denied and unknown, opposite-mode, cross-school, and
  soft-deleted targets remain non-enumerating.
- User list/detail select exactly one lookup mode: platform-only with
  `schools.view` or exact-school with school `users.view`; there is no fallback.
- Invitation-mode user creation persists one `invited` user without automatic
  invitation/delivery. Explicit invitation uses that persisted user; generic
  updates/recovery cannot activate `invited` users.
- Administrator resend remains absent because the published resend operation is
  still token-path-dependent. Invitation UI renders only `status`,
  `expires_at`, `delivery_channel`, and `delivery_requested_at`.
- Live MySQL full backend suite: `php artisan test --compact` passed, 502 tests
  and 2454 assertions. Focused tenant/invitation matrix passed, 24 tests and 75
  assertions.
- Full frontend suite: `npm run test:unit -- --run` passed, 341 files and 685
  tests. `npm run build` passed with existing third-party pure-annotation and
  large-chunk warnings.
- Mocked-SPA Playwright evidence: focused account lifecycle matrix passed on
  Chromium, Firefox, and WebKit (9 tests), including 390/768/1440 responsive
  checks, hidden/zero-request denial, UUID-only create recovery, and no resend
  or secret persistence. This is SPA orchestration evidence, not live backend
  authorization evidence.
- Five-administrator moderated UAT and representative human accessibility
  review remain genuinely pending.
