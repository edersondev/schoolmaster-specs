# Account Lifecycle UI Contract

This contract defines what `schoolmaster-frontend` may implement for Account
Lifecycle Workflows UI.

## Scope Contract

This feature approves:

- Invitation creation from existing user create/detail flows.
- Blocked admin invitation resend state until a non-secret resend operation is
  approved.
- Invitation setup with first password creation through token-proven guest
  flow.
- Password reset request with email only and neutral confirmation.
- Password reset completion through token-proven guest flow.
- Account lock state review for authorized account administrators.
- Administrative account lock with required reason.
- Account unlock with no reason.
- Account recovery and reactivation with optional reason.
- Safe invalid-token, validation, neutral, forbidden, tenant, not-found,
  conflict, and temporary-unavailable feedback states.
- Route-local sensitive state and service-isolated API consumption.

This feature does not approve:

- Creating users directly from account lifecycle UI outside existing user
  administration workflows.
- Admin invitation resend while the only approved operation requires an
  invitation token.
- Public school selection for password reset request.
- Submitting `school_id` or delivery metadata in password reset request.
- Automatic sign-in after invitation setup or password reset completion.
- Provider-specific email delivery, SMS, notification preferences, inboxes,
  campaigns, or templates.
- Guardian user-link management, classroom or roster workflows, platform
  support override, permanent purge, legal hold, retention management, or
  undocumented account lifecycle actions.

## OpenAPI Consumption Contract

Frontend services may consume only these approved operations for this slice:

| Operation ID | Method and Path | UI use |
|--------------|-----------------|--------|
| `createAccountInvitation` | `POST /api/v1/account-invitations` | Create or replace invitation for an existing user create/detail flow. |
| `completeAccountInvitation` | `POST /api/v1/account-invitations/{invitationToken}/setup` | Complete first password setup from invitation token. |
| `requestPasswordReset` | `POST /api/v1/auth/password-reset-requests` | Request password reset with email-only, non-enumerating confirmation. |
| `completePasswordReset` | `POST /api/v1/auth/password-resets` | Complete password reset from reset token. |
| `getAccountLock` | `GET /api/v1/users/{userId}/account-lock` | Review lock state for permitted target user. |
| `lockAccount` | `POST /api/v1/users/{userId}/account-lock` | Administratively lock account with required reason. |
| `unlockAccount` | `DELETE /api/v1/users/{userId}/account-lock` | Clear administrative account lock without request body. |
| `reactivateAccount` | `POST /api/v1/users/{userId}/account-reactivation` | Recover or reactivate account with action and optional reason. |

`resendAccountInvitation` is present in OpenAPI at
`POST /api/v1/account-invitations/{invitationToken}/resend`, but it is not
approved for admin UI while it requires an invitation token. Admin resend may
be exposed only after OpenAPI provides a non-secret resend operation by
invitation or user identifier.

Frontend implementation must not add route aliases, request fields, response
fields, token modes, action values, status values, or permission names that are
not documented in OpenAPI or the approved session contract.

## Request Contract

### Invitation Creation

`createAccountInvitation` may send only:

- `scope`
- `school_id` when approved for school scope and active school context
  requires it
- `full_name`
- `email`
- `role_ids`

The UI must start from existing user create/detail state and must not create a
user independently of the approved user administration workflow. The UI must
not submit `delivery_metadata` in this feature.

### Invitation Setup

`completeAccountInvitation` sends:

- `invitationToken` path parameter from the out-of-band link
- `password` request field

The password must be 12 to 128 characters. Paste and password managers must be
allowed. Common-password rejection comes from validation feedback.

### Password Reset Request

`requestPasswordReset` sends:

- `email`

The UI must not send `school_id`, `delivery_metadata`, cached active school
context, or any public school selector value in this feature.

### Password Reset Completion

`completePasswordReset` sends:

- `token`
- `password`

The password must be 12 to 128 characters. Paste and password managers must be
allowed. Common-password rejection comes from validation feedback.

### Account Lock

`lockAccount` sends:

- `reason` from 1 to 500 characters

### Account Unlock

`unlockAccount` sends no request body and no reason.

### Account Recovery or Reactivation

`reactivateAccount` sends:

- `action`: `unlock` or `reactivate`
- `reason` optional, max 500 characters

The UI labels may present `unlock`, `recover`, or `reactivate` actions, but the
service mapper must send only approved OpenAPI action values.

## Response and Feedback Contract

Frontend services and composables must normalize documented success and error
envelopes into these UI states where applicable:

- `success`
- `neutral-confirmation`
- `validation`
- `invalid-token`
- `unauthorized`
- `forbidden`
- `tenant-mismatch`
- `inactive-school`
- `not-found`
- `conflict`
- `temporary-unavailable`

Invalid-token state covers lifecycle token expired, reused, superseded,
revoked, malformed, missing, mismatched, or scope-incompatible outcomes.

Password reset request success must always show the same neutral confirmation
for eligible, missing, inactive, locked, deleted, unauthorized, and over-limit
identifiers.

Conflict feedback must cover inactive school, soft-deleted user, role
dependency conflict, unresolved invitation, unresolved password setup,
current-state mismatch, self-action denial, and platform-versus-school scope
mismatch without exposing hidden tenant or role details.

## Tenant and Authorization Contract

- Administrative account lifecycle operations require authenticated authorized
  actors.
- School-scoped admin operations must use authenticated active permitted school
  context.
- Platform account administration and school user administration remain
  separate.
- Frontend visibility for admin actions remains blocked until implementation
  confirms approved permission codes or session capability flags.
- Backend authorization remains authoritative when client state is stale.
- Token-proven invitation setup and password reset completion must not require
  an authenticated session and must not mix current signed-in permissions with
  the token target.

## State Boundary Contract

Frontend route-local state may own:

- pending flags
- current validation errors
- current token-flow feedback state
- current lock/action dialog state
- in-memory token and password values needed for the current submission

Frontend state must not persist:

- invitation tokens
- reset tokens
- plaintext passwords
- bearer tokens
- account lifecycle reasons
- delivery secrets
- tenant-private details
- role internals
- permission payloads

Stale responses after route, session, active school, target user, token, or
permission changes must be ignored.

## Accessibility, Localization, and Observability Contract

- Account lifecycle forms, dialogs, token states, and recovery actions must
  target WCAG 2.1 AA at 390px, 768px, and 1440px.
- Reusable account lifecycle text must be centralized through Vue I18n.
- Element Plus component tags must use PascalCase.
- Frontend diagnostics may include safe operation identifiers and request
  identifiers, but must not log passwords, lifecycle tokens, bearer tokens,
  tenant-private data, role internals, permission payloads, or reason text.
