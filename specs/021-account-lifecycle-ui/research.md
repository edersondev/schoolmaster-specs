# Research: Account Lifecycle Workflows UI

## Decision: Split guest token flows from admin account lifecycle controls

**Rationale**: Invitation setup and password reset completion are token-proven
guest workflows. Invitation creation, lock review, lock, unlock, recovery, and
reactivation are authenticated administrator workflows. Keeping these surfaces
separate prevents the current signed-in session from leaking into token-proven
actions and keeps route guards simple.

**Alternatives considered**:

- One account lifecycle module for all routes: rejected because guest flows and
  admin flows have different authentication, tenant, and error-state rules.
- Store all account lifecycle state in Pinia: rejected because most state is
  route-local and sensitive.

## Decision: Admin invitation resend remains blocked

**Rationale**: Current `resendAccountInvitation` requires
`{invitationToken}` in the path. Admin UI must not expose, persist, or depend on
reusable invitation token secrets. Exposing admin resend requires a future
OpenAPI operation that resends by non-secret invitation or user identifier.

**Alternatives considered**:

- Use the token-based resend operation in admin UI: rejected because it
  conflicts with token non-exposure.
- Remove resend from the feature entirely: rejected because the roadmap and
  backend contracts include resend semantics, but the UI must block the unsafe
  admin surface until contract expansion.

## Decision: Password reset request submits email only

**Rationale**: The OpenAPI schema allows `school_id`, but the clarified
frontend feature does not have an approved public school selector. Email-only
submission preserves non-enumerating behavior and avoids tenant disclosure or
client-side tenant inference.

**Alternatives considered**:

- Public school selector: rejected until a public, non-enumerating, approved
  source exists.
- Reuse cached active school: rejected because signed-out recovery must not
  depend on stale authenticated tenant context.
- Free-form school code: rejected because no approved contract exists.

## Decision: Action visibility waits for approved permissions or capabilities

**Rationale**: Backend docs define platform account administrators and school
user administrators but do not publish exact frontend permission codes in this
spec. Frontend action visibility must not invent authorization semantics.
Planning and tasks must verify approved permission codes or session capability
flags before enabling admin actions.

**Alternatives considered**:

- Use `users.view` and `users.manage`: rejected because account lifecycle is
  security-sensitive and may require separate authority.
- Create local permission names: rejected because permissions must come from
  published session data and approved contracts.

## Decision: Confirmation reason rules follow current OpenAPI request bodies

**Rationale**: `lockAccount` requires `reason`. `reactivateAccount` uses
`AccountRecoveryRequest`, where `reason` is optional. `unlockAccount` has no
request body. The UI should enforce only what OpenAPI supports and avoid
forcing a contract change inside frontend work.

**Alternatives considered**:

- Require reason for every admin action: rejected because unlock has no request
  body.
- Do not require any frontend reason: rejected because lock requires reason and
  should validate before submission.

## Decision: Route-local composables coordinate sensitive form and token state

**Rationale**: Lifecycle token values, plaintext passwords, reasons, and
temporary validation payloads are sensitive and should remain in route-local
state. Composables can coordinate pending state, stale-response protection, and
service calls without turning components into transport layers.

**Alternatives considered**:

- Persist token/password state in Pinia: rejected due to security risk and no
  cross-route requirement.
- Put service calls directly in pages or components: rejected by frontend
  architecture and service isolation rules.

## Decision: Use shared auth feedback for token and denial states

**Rationale**: Existing auth/session UI already handles invalid credentials,
lockout, expired session, unauthorized, forbidden, inactive-user,
inactive-school, tenant-mismatch, temporary-unavailable, and neutral
confirmation states. Account lifecycle UI should extend these patterns with
invalid lifecycle token and lifecycle success states rather than create
inconsistent messages.

**Alternatives considered**:

- Resource-specific error messages in each page: rejected because it increases
  account enumeration and translation drift risks.
- Raw backend messages: rejected because safe denial messaging must not expose
  account, tenant, token, role, or permission internals.
