# Feature Specification: Account Lifecycle Workflows UI

**Feature Branch**: `021-account-lifecycle-ui`  
**Created**: 2026-06-28  
**Status**: Draft  
**Input**: User description: "Specify Account Lifecycle Workflows UI for schoolmaster-frontend. Define SPA invitation, password setup, password reset, account reactivation, account lock, and account recovery flows, including token handling, safe expired/invalid token states, inactive or locked account messaging, validation errors, safe denial messages, actor permission visibility, login/session integration, admin user-management entry points, and approved OpenAPI operation IDs required before frontend implementation."

## Clarifications

### Session 2026-06-28

- Q: Should invitation UI create users directly, or start only from existing user administration records? → A: Invitation starts from existing user create/detail flow; account lifecycle UI manages invite/setup and approved resend state only.
- Q: What should frontend use for administrative account lifecycle action visibility when exact permission codes are not approved? → A: Block action visibility until approved permission codes or capability flags are confirmed before enabling actions.
- Q: Which administrative account lifecycle actions require a reason in the UI? → A: Lock requires reason; reactivation and recovery allow optional reason; unlock uses no reason.
- Q: Should public password reset request collect school context? → A: Password reset request submits email only; no public school selector in this feature.
- Q: How should admin invitation resend work when the current resend operation requires an invitation token? → A: Block admin resend UI until OpenAPI provides non-secret resend by invitation or user identifier.
- Q: When should exact account lifecycle permission codes or capability flags be confirmed? → A: Planning documents the gate; implementation confirms approved permission codes or capability flags before enabling actions.
- Q: Should invitation creation submit delivery metadata? → A: Do not submit `delivery_metadata` in invitation creation for this feature.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Invite users and complete first password setup (Priority: P1)

An authorized platform account administrator or school user administrator can
invite an eligible same-scope user from an existing user create or detail flow,
and the invited user can complete first password setup from the invitation link
without any secret token being exposed in visible UI, logs, or reusable state.

**Why this priority**: Account onboarding is blocked until invitations and
first password setup are usable by administrators and invited users.

**Independent Test**: Can be fully tested by signing in as an authorized
administrator, creating an invitation for an eligible target user, verifying
admin resend is blocked until a non-secret resend contract exists, opening the
invitation setup flow with valid and invalid token states, submitting valid and
invalid passwords, and verifying the user reaches the approved sign-in recovery
path only after successful setup.

**Acceptance Scenarios**:

1. **Given** an authorized administrator is creating or reviewing an eligible
   existing user record in the correct platform or active school scope,
   **When** they create an invitation, **Then** the UI submits only approved
   invitation fields and shows invitation status, expiry, and email delivery
   request metadata without exposing the reusable invitation token.
2. **Given** an invitation exists and resend is still allowed, **When** the
   administrator needs to resend, **Then** the admin UI remains blocked unless
   OpenAPI provides a non-secret resend operation by invitation or user
   identifier.
3. **Given** invitation sending has reached the approved same-user and
   same-scope limit, **When** the administrator attempts another invitation
   creation or any future approved resend, **Then** the UI shows validation or
   conflict feedback and does not claim email delivery happened.
4. **Given** an invited user opens a valid unexpired invitation setup link,
   **When** they submit a password that satisfies the approved password rules,
   **Then** setup completes, the invitation cannot be reused, and the user is
   guided to sign in.
5. **Given** an invitation token is expired, reused, superseded, revoked,
   malformed, missing, mismatched, or scope-incompatible, **When** the user
   opens or submits the setup flow, **Then** the UI shows the approved invalid
   token state without exposing account, school, role, or token details.

---

### User Story 2 - Complete password reset safely (Priority: P2)

A signed-out user can start password recovery from the existing forgot-password
entry flow, then complete password reset from a valid reset token while all
externally visible messages avoid account enumeration.

**Why this priority**: Password recovery is a core self-service workflow and
directly affects the login experience defined by the authentication foundation.

**Independent Test**: Can be fully tested by submitting password reset requests
for eligible, missing, inactive, locked, deleted, and unauthorized identifiers,
then completing reset with valid and invalid tokens and verifying that only
approved neutral, invalid-token, validation, and success states appear.

**Acceptance Scenarios**:

1. **Given** a signed-out user enters a valid email address on the password
   reset request screen, **When** they submit the request, **Then** the UI shows
   the same neutral accepted confirmation regardless of account existence,
   inactive state, locked state, deleted state, authorization state, or request
   rate limit.
2. **Given** a signed-out user enters invalid reset request input, **When** they
   submit the form, **Then** the UI shows field-level validation and does not
   infer whether any account exists.
3. **Given** a valid unexpired reset token, **When** the user submits a
   replacement password that satisfies approved password rules, **Then** reset
   completes, active sessions for the affected user are no longer trusted, and
   the user is guided to sign in with the new password.
4. **Given** a reset token is missing or locally malformed, **When** the user
   opens the reset completion flow, **Then** the UI shows the approved invalid
   token state without revealing account or tenant details.
5. **Given** a reset token is expired, reused, superseded, revoked, mismatched,
   or scope-incompatible, **When** the user submits the reset completion flow,
   **Then** the UI relies on the documented `completePasswordReset` response and
   shows the approved invalid token state without revealing account or tenant
   details.
6. **Given** failed reset-token completion attempts or reset requests exceed
   approved limits, **When** the user retries, **Then** the UI preserves the
   non-enumerating or invalid-token behavior defined by contract and does not
   claim credentials were changed.

---

### User Story 3 - Recover, lock, unlock, and reactivate accounts (Priority: P3)

An authorized account administrator can review account lock state and perform
approved lock, unlock, recovery, or reactivation actions for same-scope users
without bypassing active-school, user lifecycle, role dependency, or tenant
authorization rules.

**Why this priority**: Account support workflows are less frequent than
onboarding and password reset but are security-sensitive and affect operational
support and audit behavior.

**Independent Test**: Can be fully tested by signing in with and without the
required account administration permissions, opening user-management entry
points for platform and school users, reviewing lock state, performing allowed
actions, and verifying forbidden, not-found, tenant-denial, conflict,
validation, and success states.

**Acceptance Scenarios**:

1. **Given** an administrator has permission for the target account scope,
   **When** they open account lifecycle controls from an approved user
   management entry point, **Then** the UI shows only actions allowed by the
   current account state, tenant context, and actor permissions.
2. **Given** an account can be administratively locked, **When** the
   administrator confirms the lock with a required reason, **Then** the UI shows
   the resulting lock state and does not continue to treat prior sessions for
   that user as usable.
3. **Given** an administrative lock can be cleared, **When** an authorized actor
   unlocks the account without a reason or recovers the account with an optional
   reason, **Then** the UI shows the approved cleared or recovered state and
   explains any next sign-in step.
4. **Given** an inactive account is eligible for reactivation, **When** an
   authorized actor reactivates it with an optional reason, **Then** the UI shows
   that authentication may resume only after user, school, role assignment,
   password setup, lock state, and scope requirements are valid.
5. **Given** an action would bypass inactive-school, soft-deleted-user, missing
   role dependency, unresolved invitation, unresolved password setup, platform
   versus school scope, or tenant authorization rules, **When** the action is
   submitted, **Then** the UI shows approved conflict or denial feedback without
   changing visible account state.

---

### User Story 4 - Preserve safe login and session recovery behavior (Priority: P4)

Users and administrators receive consistent, safe guidance when account
lifecycle actions intersect with sign-in, inactive-user, locked-account,
expired-session, or invalid-token states.

**Why this priority**: The authentication foundation already owns login and
session bootstrap. Account lifecycle UI must extend that behavior without
creating contradictory recovery paths or unsafe account-state disclosure.

**Independent Test**: Can be fully tested by forcing inactive-user,
locked-account, invalid-token, expired-session, forbidden, tenant-mismatch, and
temporary-unavailable states across invitation, reset, and account
administration flows and verifying each state renders only the approved message
and recovery actions.

**Acceptance Scenarios**:

1. **Given** a signed-in user opens a guest-only invitation setup or password
   reset completion link, **When** the flow requires token proof instead of an
   authenticated session, **Then** the UI uses the approved guest lifecycle flow
   and does not mix it with the current session state.
2. **Given** a signed-out user completes password setup or password reset,
   **When** the success state appears, **Then** the only next authentication
   path is the approved sign-in route unless OpenAPI explicitly documents
   automatic session creation.
3. **Given** login is blocked by inactive or locked account state, **When** the
   user receives guidance, **Then** the UI shows safe recovery instructions that
   do not disclose whether a different identifier, tenant, or role would work.
4. **Given** an administrative account lifecycle request receives
   unauthorized, forbidden, tenant-mismatch, inactive-school, not-found, or
   temporary-unavailable feedback, **When** the response is displayed, **Then**
   protected account details remain hidden and the user receives the approved
   retry, sign-in, or return path.

### Edge Cases

- Invitation or reset completion response arrives after the user navigates away,
  signs in, signs out, or changes active school; stale results must not replace
  the current view.
- A reset token is missing or locally malformed in the URL; the UI may show the
  invalid-token state before submission without calling an undocumented
  validation endpoint.
- A reset token appears syntactically valid in the URL but is expired, reused,
  superseded, revoked, mismatched, or scope-incompatible; the UI must submit
  `completePasswordReset` and rely on its documented response before showing the
  server-known invalid-token state.
- A user copies a setup or reset URL, opens it in another browser, or reuses it
  after successful completion.
- A browser auto-fill tool or password manager inserts a password; the UI must
  allow paste and password-manager input while enforcing validation feedback.
- A reset request targets an eligible, missing, inactive, locked, deleted,
  unauthorized, or over-limit account identifier; the UI must show the same
  neutral accepted confirmation.
- A signed-out user expects to choose a school during password reset request;
  this feature must not show a public school selector or submit school context.
- Invitation sending or resending is over the approved 3-per-user-and-scope
  limit within 24 hours.
- Admin resend is desired but the only approved operation requires an
  invitation token; admin resend UI must remain hidden or blocked until OpenAPI
  provides a non-secret resend operation by invitation or user identifier.
- Invitation completion reaches the approved failed-attempt threshold and the
  invitation becomes revoked.
- Reset-token completion reaches the approved failed-attempt threshold and new
  reset token creation is suppressed without treating the whole account as
  administratively locked.
- Administrative lock remains active across time until an authorized unlock or
  recovery action clears it.
- A platform account administrator attempts school-scoped account lifecycle
  actions without explicit same-school authorization and active school context.
- A school user administrator attempts account lifecycle actions for a platform
  user or another school's user.
- A target user is inactive, deleted, missing active role assignments, tied to
  an inactive school, or still has unresolved invitation or password setup
  state.
- An administrator attempts to lock, unlock, recover, or reactivate their own
  account.
- Temporary service failure occurs after the user has entered a password,
  recovery reason, or account lifecycle confirmation; sensitive values must not
  be written to diagnostics or reused unexpectedly.
- A token, password, email address, reason, tenant identifier, role assignment,
  or permission payload appears in an error response or diagnostic payload; the
  UI must not echo secrets or tenant-private details into visible feedback or
  client diagnostics.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is defined by this
  frontend specification. Backend readiness is required for the approved account
  lifecycle operations, token semantics, password rules, authorization rules,
  tenant behavior, lock behavior, conflict outcomes, and non-enumerating reset
  behavior.
- **Frontend repository impact**: Add or extend account lifecycle screens and
  user-management entry points for invitation creation, blocked admin resend
  state until a non-secret resend contract exists, invitation setup, password
  reset request and completion, account lock review, account lock, unlock,
  recovery, reactivation, safe feedback states, and tests.
- **Specification or contract repository impact**: Add this frontend
  specification and, during planning, add a frontend account lifecycle
  consumption contract that maps UI surfaces to the approved OpenAPI operation
  IDs. OpenAPI changes are required only if review discovers missing fields,
  statuses, errors, or operations needed before implementation.
- **Delivery ownership and sequencing**: `schoolmaster-specs` approves this
  frontend behavior first. `schoolmaster-backend` account lifecycle behavior and
  OpenAPI operation IDs must be confirmed before `schoolmaster-frontend` exposes
  each account lifecycle surface.

### API Contract Impact

- **OpenAPI update required**: No new OpenAPI change is currently planned
  because account lifecycle paths are already present. Planning must still
  verify that every consumed request field, response field, status, token error,
  conflict envelope, forbidden outcome, tenant outcome, and validation envelope
  is approved before implementation.
- **Versioned endpoints affected**:
  - `createAccountInvitation` - `POST /api/v1/account-invitations`
  - `resendAccountInvitation` - `POST /api/v1/account-invitations/{invitationToken}/resend` (not approved for admin UI while it requires an invitation token)
  - `completeAccountInvitation` - `POST /api/v1/account-invitations/{invitationToken}/setup`
  - `requestPasswordReset` - `POST /api/v1/auth/password-reset-requests`
  - `completePasswordReset` - `POST /api/v1/auth/password-resets`
  - `getAccountLock` - `GET /api/v1/users/{userId}/account-lock`
  - `lockAccount` - `POST /api/v1/users/{userId}/account-lock`
  - `unlockAccount` - `DELETE /api/v1/users/{userId}/account-lock`
  - `reactivateAccount` - `POST /api/v1/users/{userId}/account-reactivation`
- **JSON response impact**: The frontend must consume only published success,
  non-enumerating accepted, invalid-token, validation, unauthorized, forbidden,
  tenant-mismatch, inactive-school, not-found, conflict, and temporary failure
  envelopes. Visible feedback and diagnostics must not include plaintext
  passwords, lifecycle token values, bearer tokens, credential hashes, delivery
  secrets, tenant-private details, role internals, or permission payloads.
- **Authentication/authorization impact**: Invitation creation, account lock
  review, lock, unlock, recovery, and reactivation require authenticated
  authorized actors. Admin invitation resend remains blocked until an approved
  non-secret operation exists. Invitation setup and password reset
  completion are token-proven guest lifecycle flows. Platform account
  administrators may manage platform accounts only. School user administrators
  may manage same-school accounts only under active permitted school context.
  Planning documents the action-visibility gate; frontend action visibility
  remains blocked until implementation confirms the approved permission codes
  or session capability flags for platform and school account lifecycle
  administration.
- **Compatibility impact**: Additive frontend behavior over approved contracts.
  Automatic session creation after setup or reset, provider-specific messaging,
  SMS delivery, notification-center behavior, guardian account linking,
  classroom or roster workflows, platform support override, permanent purge,
  legal hold, retention management, and undocumented account lifecycle actions
  remain outside this feature.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-scoped account lifecycle actions use the
  authenticated active permitted school context before any school-owned user,
  role, invitation, recovery, lock, or reactivation state is shown or acted on.
  Platform-scoped account lifecycle actions must not expose school-owned user
  or role state.
- **Cross-tenant or platform access impact**: Platform account administration
  and school user administration remain separate. Platform access does not imply
  school-owned account lifecycle authority. Tenant-denied and not-found states
  must not reveal hidden account existence.
- **Soft delete impact**: Soft-deleted users are not recoverable through account
  lifecycle UI unless administration lifecycle rules and approved contracts first
  expose a restoration path. Account lifecycle UI must present conflict or
  blocked recovery feedback when recovery would bypass soft-delete rules.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The frontend MUST expose invitation creation only from approved
  existing user create or detail entry points for actors with account lifecycle
  authority in the target platform or active school scope.
- **FR-002**: Invitation creation UI MUST submit only documented target identity,
  account scope, role, and school fields approved by the account invitation
  contract, MUST NOT submit `delivery_metadata` in this feature, and MUST NOT
  create a user record independently of the approved user administration
  workflow.
- **FR-003**: Invitation creation feedback MUST show invitation status, expiry,
  delivery request metadata, and send limit failures without exposing reusable
  invitation token secrets.
- **FR-003a**: Admin invitation resend UI MUST remain blocked while the only
  approved resend operation requires an invitation token; frontend may expose
  admin resend only after OpenAPI provides a non-secret resend operation by
  invitation or user identifier.
- **FR-004**: Invitation setup UI MUST accept an invitation token only as proof
  for the setup flow and MUST normalize expired, reused, superseded, revoked,
  malformed, missing, mismatched, and scope-incompatible token outcomes into an
  approved invalid-token state.
- **FR-005**: Password setup UI MUST validate password requirements before
  submission, including length, common-password rejection when returned by
  validation, and paste/password-manager compatibility.
- **FR-006**: After successful invitation setup, the frontend MUST guide the
  user to the approved sign-in route unless the contract explicitly documents
  automatic session creation.
- **FR-007**: Password reset request UI MUST show the same neutral accepted
  confirmation for eligible, missing, inactive, locked, deleted, unauthorized,
  and over-limit account identifiers.
- **FR-008**: Password reset request UI MUST show field-level validation for
  invalid request input without revealing account existence or account state.
- **FR-008a**: Password reset request UI MUST submit email only and MUST NOT
  show a public school selector, use cached active school context, or submit a
  school identifier in this feature.
- **FR-009**: Password reset completion UI MUST normalize missing or locally
  malformed reset tokens into an approved invalid-token state without an API
  call, and MUST normalize expired, reused, superseded, revoked, mismatched, and
  scope-incompatible reset-token outcomes from the documented
  `completePasswordReset` response into the same approved invalid-token state.
- **FR-010**: Password reset completion UI MUST validate replacement password
  requirements before submission and show approved validation feedback without
  echoing the password into diagnostics or shared feedback.
- **FR-011**: After successful password reset completion, the frontend MUST clear
  stale assumptions about existing sessions for the affected user and guide the
  user to sign in with the new password.
- **FR-012**: Account lock review UI MUST show only approved lock state fields
  for the permitted target user and MUST hide lock state when authorization,
  tenant context, or not-found responses deny access.
- **FR-013**: Account lifecycle action UI MUST show lock, unlock, recovery, and
  reactivation actions only when actor permissions, target account state, tenant
  context, and approved contract semantics allow the action.
- **FR-013a**: Administrative account lifecycle action visibility MUST remain
  blocked while planning documents the gate, and MUST remain blocked in
  implementation until approved permission codes or session capability flags are
  confirmed for platform account administration and school account
  administration.
- **FR-014**: Account lock, unlock, recovery, and reactivation actions MUST
  require explicit confirmation before submission and MUST preserve backend
  authorization as authoritative when client-side visibility is stale.
- **FR-014a**: Account lock confirmation MUST require a reason, reactivation
  and recovery confirmations MAY include an optional reason, and unlock
  confirmation MUST NOT require a reason unless OpenAPI adds one.
- **FR-015**: Reactivation feedback MUST explain that authentication can resume
  only when user lifecycle, school lifecycle, active role assignments, password
  setup state, lock state, and platform or school scope are valid.
- **FR-016**: Conflict feedback MUST cover inactive school, soft-deleted user,
  role dependency conflict, unresolved invitation, unresolved password setup,
  current-state mismatch, self-action denial, and platform-versus-school scope
  mismatch without exposing hidden tenant or role details.
- **FR-017**: Administrative account lifecycle operations MUST use active
  permitted school context for school-scoped users and MUST NOT infer tenant
  authority from visible lists, cached user records, or platform access.
- **FR-018**: Token-proven setup and reset completion flows MUST not require an
  authenticated session and MUST not mix a current signed-in user's permissions
  with the token-proven target account.
- **FR-019**: The frontend MUST not persist lifecycle token values, plaintext
  passwords, bearer tokens, recovery reasons, delivery secrets, tenant-private
  details, or permission payloads in diagnostics, shared feedback, or reusable
  frontend state beyond what is required to complete the current submitted
  action.
- **FR-020**: The frontend MUST ignore stale account lifecycle responses when
  route, session, active school, token, target user, or permission state changed
  after the request started.
- **FR-021**: The frontend MUST consume only the approved account lifecycle
  operation IDs listed in this specification and MUST block any UI surface whose
  required operation, field, error, or status is not confirmed in OpenAPI.
- **FR-022**: The frontend MUST preserve existing authentication foundation
  behavior for sign-in, session expiration, unauthorized, forbidden,
  inactive-user, inactive-school, tenant-mismatch, lockout, and
  temporary-unavailable states.
- **FR-023**: The frontend MUST provide accessible, localized, centralized text
  for account lifecycle forms, confirmations, success states, neutral
  confirmations, invalid-token states, validation states, denied states, and
  recovery actions.
- **FR-024**: The frontend MUST NOT introduce provider-specific email delivery,
  SMS delivery, notification preferences, inboxes, campaigns, guardian
  user-link management, classroom or roster workflows, platform support
  overrides, permanent purge, legal hold, retention management, or undocumented
  account lifecycle actions.

### Key Entities *(include if feature involves data)*

- **Account Invitation**: Single-use onboarding state for a platform or
  school-scoped user, including target user, scope, school when applicable,
  status, expiry, completion state, and email delivery request metadata without
  exposing reusable token secrets.
- **Invitation Token**: User-provided proof used only to complete first password
  setup; invalid-token states include expired, reused, superseded, revoked,
  malformed, missing, mismatched, and scope-incompatible tokens.
- **Password Reset Request**: Non-enumerating recovery request that accepts an
  account identifier and always presents the same accepted result for eligible,
  missing, inactive, locked, deleted, unauthorized, and over-limit accounts.
- **Password Reset Token**: User-provided proof used only to complete credential
  replacement; invalid-token states must not reveal target account or tenant
  details.
- **Account Lock**: Account access restriction state, including failed-login or
  administrative lock status where approved; administrative locks remain until
  an authorized unlock or recovery clears them.
- **Account Recovery or Reactivation Result**: Approved outcome of unlock,
  recovery, or reactivation action that reports target user, scope, status, and
  action without exposing hidden dependencies.
- **Account Administrator**: Authorized platform or school actor who may manage
  account lifecycle workflows only inside the actor's approved scope.
- **User Account**: Target identity whose invitation, password setup, reset,
  lock, recovery, reactivation, role assignment, school state, and
  authentication eligibility determine visible actions.
- **Safe Feedback State**: User-facing validation, neutral confirmation,
  invalid-token, forbidden, tenant-denial, not-found, conflict, or temporary
  failure message that avoids account enumeration and secret exposure.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of account lifecycle UI surfaces planned for implementation
  map to one of the approved OpenAPI operation IDs listed in this specification.
- **SC-002**: 100% of password reset request outcomes for eligible, missing,
  inactive, locked, deleted, unauthorized, and over-limit identifiers show the
  same neutral accepted confirmation in user testing.
- **SC-003**: Users can complete valid invitation setup or password reset
  completion in under 2 minutes from opening a valid lifecycle link.
- **SC-004**: 100% of expired, reused, superseded, revoked, malformed, missing,
  mismatched, and scope-incompatible setup or reset tokens show an invalid-token
  state without exposing account, school, role, or token details.
- **SC-005**: Authorized administrators can create an eligible invitation and
  see updated invitation status in under 30 seconds; admin resend remains
  hidden or blocked until a non-secret resend contract exists.
- **SC-006**: Authorized administrators can review lock state and complete an
  approved lock, unlock, recovery, or reactivation action in under 2 minutes
  without leaving user management.
- **SC-007**: Permission and tenant tests confirm 100% of platform-versus-school,
  cross-school, inactive-school, and insufficient-permission attempts hide
  protected account details and show only approved denial or conflict feedback.
- **SC-008**: Security review confirms account lifecycle feedback and diagnostics
  expose no plaintext passwords, lifecycle token values, bearer tokens,
  credential hashes, delivery secrets, tenant-private details, or permission
  payloads.
- **SC-009**: At least 90% of representative administrators can correctly choose
  when to invite, understand blocked resend, lock, unlock, recover, or
  reactivate an account using only the visible status and feedback text.

## Assumptions

- Backend account lifecycle contracts and implementations from
  `specs/008-account-lifecycle-workflows/` are available or will be verified
  before frontend implementation exposes each workflow.
- Existing authentication and session foundation behavior from
  `specs/017-auth-session-ui/` remains authoritative for sign-in, session
  bootstrap, session expiration, denied states, and forgot-password request
  entry.
- Existing administration lifecycle behavior from
  `specs/020-administration-lifecycle-ui/` remains authoritative for user
  detail, user lifecycle status, soft-delete rules, role dependencies, and
  active school context.
- Invitation tokens expire after 7 days and password reset tokens expire after
  30 minutes, matching the approved backend account lifecycle rules.
- Password setup and reset completion require passwords from 12 to 128
  characters, reject common passwords through validation feedback, and allow
  paste and password managers.
- Account lifecycle email behavior is limited to approved email delivery request
  metadata; provider-specific delivery status, SMS, notification preferences,
  templates, inboxes, and campaigns remain outside this feature.
- Automatic sign-in after invitation setup or password reset completion is out
  of scope unless OpenAPI explicitly documents that behavior before planning.
