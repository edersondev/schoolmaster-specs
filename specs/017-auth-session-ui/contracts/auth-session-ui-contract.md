# Authentication and Session UI Contract

This contract defines what `schoolmaster-frontend` may implement for the
Authentication and Session Foundation UI.

## Scope Contract

This feature approves:

- Login page and sign-in form.
- Forgot-password request entry page with neutral confirmation behavior.
- Authenticated session bootstrap.
- Current-user, role, permission, and tenant-context hydration.
- Active school restoration when the last approved school remains authorized.
- School selection gate when an authorized active school cannot be restored.
- School selection choices only from an approved OpenAPI operation that returns
  schools authorized for the current user.
- Protected-route preservation for signed-out direct access and expired
  sessions.
- Layout selection between unauthenticated auth pages and authenticated
  protected surfaces.
- UI states for validation, invalid credentials, lockout, expired session,
  unauthorized, forbidden, inactive-user, inactive-school, tenant-mismatch, and
  temporary-unavailable outcomes.

This feature does not approve:

- Password reset completion.
- Password setup.
- Invitations.
- Reactivation workflows.
- Account lock administration.
- New backend endpoints.
- School selection backed by undocumented or ambiguously scoped school lists.
- Direct frontend consumption of undocumented routes or payloads.
- Client-side tenant inference.
- TypeScript requirements.

## OpenAPI Consumption Contract

Frontend services may consume only these approved operations for this slice:

| Operation ID | Method and Path | Purpose |
|--------------|-----------------|---------|
| `login` | `POST /api/v1/auth/login` | Authenticate a user and return `AuthSession`. |
| `getCurrentUser` | `GET /api/v1/auth/me` | Hydrate current user, resolved school, roles, and permissions. |
| `logout` | `POST /api/v1/auth/logout` | Revoke the current bearer token. |
| `requestPasswordReset` | `POST /api/v1/auth/password-reset-requests` | Start password recovery with non-enumerating accepted response. |

Frontend implementation must not add route aliases or consume backend-local
payload fields that are not documented in OpenAPI.

School selection may be implemented only after the team confirms an approved
OpenAPI operation that returns only schools authorized for the current user. If
the existing `listSchools` operation is used, the plan/tasks must verify that
its authorization scope is valid for school selection across the intended user
types. Otherwise, OpenAPI must be updated before frontend implementation.

## Request Contract

### Login

`login` submissions may send:

- `email`
- `password`
- `school_id` when an explicit school context is needed

The frontend must validate required email and password input before submission.
Field names and validation behavior must follow the published
`LoginRequest` schema.

### Current User

`getCurrentUser` may send the documented `X-School-Id` tenant context header
when the session is not already bound to exactly one active school and the
frontend is attempting to restore or select an active school.

The header may request context only. It does not itself authorize tenant-owned
content.

### Password Reset Request

`requestPasswordReset` submissions may send:

- `email`
- `school_id` when recovery is school-context-specific
- `delivery_metadata` only if a later implementation has approved metadata
  semantics

The frontend must not use password reset request responses to infer account
existence or account state.

## Session Response Contract

Successful `login` and `getCurrentUser` responses return `AuthSession` data
that may be mapped into frontend session state:

- `token`
- `token_expires_at`
- `user`
- `resolved_school`
- `roles`
- `permissions`

Frontend session state must not treat cached user, role, permission, or school
data as authoritative after token rejection, tenant mismatch, inactive-user, or
inactive-school outcomes.

## Tenant Context Contract

- Restore the last approved active school only when `getCurrentUser` confirms
  the requested school remains authorized.
- Require school selection before tenant-owned content renders when no
  authorized active school can be restored for a multi-school user.
- Do not automatically select the first returned or visible school unless a
  future spec explicitly approves that behavior.
- Do not render tenant-owned content while tenant context is missing, inactive,
  mismatched, or still being confirmed.
- Populate school selection choices only from an approved operation that returns
  user-authorized schools.
- Block school-selection UI implementation when no approved source operation is
  confirmed.
- Clear stale tenant-owned state when a tenant mismatch or inactive-school
  outcome is received.

## Protected Route Contract

- Signed-out users who open a protected route must be sent to sign in while the
  originally requested route is preserved.
- Users whose session expires while opening protected content must be sent to
  sign in while the originally requested route is preserved.
- After sign-in or re-authentication, the originally requested route may be
  restored only when the current session remains authorized for the route and
  tenant context.
- If the originally requested route is not authorized after sign-in, route the
  user to an allowed authenticated workspace instead of the denied route.
- Route guards are client-side navigation controls only; backend authorization
  remains authoritative.

## Error and Feedback State Contract

Frontend services and stores must normalize documented success and error
envelopes into these UI states where applicable:

- `validation`
- `invalid-credentials`
- `lockout`
- `expired-session`
- `unauthorized`
- `forbidden`
- `inactive-user`
- `inactive-school`
- `tenant-mismatch`
- `temporary-unavailable`
- `neutral-confirmation`

Error codes currently documented in the shared `ErrorEnvelope` examples include:

- `validation_failed`
- `forbidden`
- `tenant_mismatch`
- `auth_locked`
- `token_expired`
- `token_revoked`
- `inactive_user`
- `inactive_school`

UI messages must use centralized reusable text and must not reveal account
existence, unauthorized tenant details, or permission internals.

## Forgot-Password Entry Contract

- Valid password reset request submissions must show the same neutral accepted
  confirmation regardless of account existence, inactive state, locked state,
  deleted state, unauthorized state, or rate-limit state.
- Invalid email format may show field-level validation.
- Reset-token entry, new password submission, invitation setup, and account
  recovery completion are out of scope for this feature.

## State Boundary Contract

Frontend state may own:

- authenticated session bootstrap status
- token expiration timestamp needed for user-facing session behavior
- current user display data
- role and permission arrays from the approved contract
- confirmed active school context
- last approved active school identifier for restoration attempts
- originally requested protected route during sign-in or re-authentication
- auth and session feedback state

Frontend state must not:

- authorize tenant access without backend confirmation
- persist sensitive auth data outside approved authentication behavior and
  security guidance
- bypass service mapping for API calls
- expose raw backend error payloads directly in presentation components

## Accessibility, Localization, and Observability Contract

- Auth forms, feedback states, school selection, and recovery actions must
  target WCAG 2.1 AA.
- Reusable auth/session text must be centralized for Vue I18n.
- Auth/session failures should be traceable through frontend diagnostics
  without logging passwords, bearer tokens, tenant-private details, or recovery
  identifiers.
