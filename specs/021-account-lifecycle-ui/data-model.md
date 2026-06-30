# Data Model: Account Lifecycle Workflows UI

## AccountInvitationView

**Purpose**: Frontend-safe invitation status shown from user create/detail
flows after invitation creation or approved future non-secret resend.

**Fields**: `id`, `userId`, `schoolId`, `scope`, `status`, `expiresAt`,
`completedAt`, `deliveryChannel`, `deliveryRequestedAt`.

**Source**: `AccountInvitation` from `createAccountInvitation`. Admin resend
remains blocked while `resendAccountInvitation` requires an invitation token.

**Rules**:

- `scope` is `platform` or `school`.
- `status` is `pending`, `completed`, `expired`, `superseded`, or `revoked`.
- `deliveryChannel` is `email`.
- Must not contain invitation token values.
- School-scoped invitations require authenticated active permitted school
  context.

**States**:

```text
none -> pending -> completed
pending -> expired
pending -> superseded
pending -> revoked
```

Creating a new invitation replaces prior unused invitations for the same user
and scope through backend behavior.

## InvitationSetupDraft

**Purpose**: Route-local guest form state for completing first password setup
from an out-of-band invitation token.

**Fields**: `invitationToken`, `password`, `validationErrors`, `pending`,
`feedbackState`.

**Rules**:

- `password` is required, 12 to 128 characters, and compatible with paste and
  password managers.
- Common-password rejection is shown only from validation feedback.
- `invitationToken` is read from the route/link context and submitted only to
  `completeAccountInvitation`.
- Token values and passwords must not be persisted in Pinia, diagnostics, local
  storage, session storage, or shared feedback.

**States**:

```text
ready -> submitting -> completed
ready -> submitting -> invalid-token
ready -> submitting -> validation
ready -> submitting -> conflict
ready -> submitting -> temporary-unavailable
```

Invalid token covers expired, reused, superseded, revoked, malformed, missing,
mismatched, and scope-incompatible token outcomes.

## PasswordResetRequestDraft

**Purpose**: Guest request state for starting password recovery without account
enumeration.

**Fields**: `email`, `validationErrors`, `pending`, `feedbackState`.

**Rules**:

- `email` is required and must be syntactically valid.
- The frontend submits email only.
- The frontend must not show a public school selector, use cached active school
  context, submit `school_id`, or submit delivery metadata in this feature.
- Successful request always maps to neutral accepted confirmation.

**States**:

```text
ready -> submitting -> neutral-confirmation
ready -> submitting -> validation
ready -> submitting -> temporary-unavailable
```

Neutral confirmation must be identical for eligible, missing, inactive, locked,
deleted, unauthorized, and over-limit identifiers.

## PasswordResetCompletionDraft

**Purpose**: Route-local guest form state for replacing credentials from a
valid reset token.

**Fields**: `resetToken`, `password`, `validationErrors`, `pending`,
`feedbackState`.

**Rules**:

- `password` is required, 12 to 128 characters, and compatible with paste and
  password managers.
- `resetToken` is read from the route/link context and submitted only to
  `completePasswordReset`.
- Missing or locally malformed `resetToken` values may enter `invalid-token`
  before submission; expired, reused, superseded, revoked, mismatched, and
  scope-incompatible token states are only known after `completePasswordReset`
  responds.
- Successful reset revokes active bearer tokens for the affected user on the
  backend; frontend must clear stale assumptions and route to sign in.
- Token values and passwords must not be persisted in Pinia, diagnostics, local
  storage, session storage, or shared feedback.

**States**:

```text
ready -> submitting -> completed
ready -> submitting -> invalid-token
ready -> submitting -> validation
ready -> submitting -> conflict
ready -> submitting -> temporary-unavailable
```

## AccountLockView

**Purpose**: Frontend-safe lock state shown to authorized account
administrators on user detail surfaces.

**Fields**: `id`, `userId`, `schoolId`, `lockType`, `status`, `reason`,
`lockedAt`, `clearedAt`.

**Source**: `AccountLock` from `getAccountLock` and `lockAccount`.

**Rules**:

- `lockType` may be `administrative`, `failed_login`, `recovery_hold`, or null.
- `status` is `active`, `cleared`, or `none`.
- `reason` may be shown only when returned by an approved response for the
  permitted target user.
- Hidden tenant, role, permission, or account details must not be inferred from
  forbidden, tenant-mismatch, or not-found responses.

**States**:

```text
none -> active
active -> cleared
cleared -> active
```

Administrative locks remain active until authorized unlock or recovery clears
them.

## AccountLifecycleActionDraft

**Purpose**: Route-local admin confirmation state for lock, unlock, recovery,
and reactivation actions.

**Fields**: `targetUserId`, `action`, `reason`, `pending`, `validationErrors`,
`feedbackState`, `eligibility`.

**Rules**:

- `action` is `lock`, `unlock`, `recover`, or `reactivate`.
- Lock requires `reason` from 1 to 500 characters.
- Recovery and reactivation may include optional `reason` up to 500 characters.
- Unlock uses no reason unless OpenAPI is expanded later.
- Visibility remains blocked until approved permission codes or session
  capability flags are confirmed.
- Backend authorization and current account state remain authoritative.

**States**:

```text
hidden -> eligible -> confirming -> submitting -> succeeded
eligible -> confirming -> submitting -> validation
eligible -> confirming -> submitting -> conflict
eligible -> confirming -> submitting -> denied
eligible -> confirming -> submitting -> not-found
eligible -> confirming -> submitting -> temporary-unavailable
```

Conflict feedback covers inactive school, soft-deleted user, role dependency
conflict, unresolved invitation, unresolved password setup, current-state
mismatch, self-action denial, and platform-versus-school scope mismatch.

## AccountLifecycleResultView

**Purpose**: Frontend-safe result of setup, reset, unlock, recovery, or
reactivation.

**Fields**: `userId`, `schoolId`, `status`, `action`.

**Rules**:

- `schoolId` is null for platform-scoped results.
- `status` and `action` are display-only values from the approved response.
- Result views must not imply automatic sign-in unless OpenAPI later documents
  that behavior.

## SafeFeedbackState

**Purpose**: Canonical UI state used by account lifecycle pages and admin
panels.

**Values**: `loading`, `validation`, `neutral-confirmation`, `invalid-token`,
`unauthorized`, `forbidden`, `tenant-mismatch`, `inactive-school`, `not-found`,
`conflict`, `temporary-unavailable`, `success`.

**Rules**:

- Feedback must not reveal account existence, hidden tenant details, role
  internals, permission payloads, lifecycle token values, plaintext passwords,
  bearer tokens, or delivery secrets.
- Field-level validation may identify invalid local input fields.
- Diagnostics may include safe operation identifiers and request identifiers
  only.
