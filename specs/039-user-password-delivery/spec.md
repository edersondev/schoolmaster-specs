# Feature Specification: Secure User Password Delivery

**Feature Branch**: `039-user-password-delivery`
**Created**: 2026-08-26
**Status**: Draft
**Input**: User description: "About the users, add secure password-setup/reset delivery after active creation."

## Clarifications

### Session 2026-08-26

- Q: For an already active user, who determines whether the credential link is
  setup or reset? → A: One administrator action; the system determines setup
  versus reset from credential state.
- Q: What administrator password-link delivery limit applies? → A: Maximum 3
  deliveries per user and scope within 24 hours.
- Q: How does password delivery behave for a locked user? → A: Locked users are
  ineligible until separately unlocked or recovered.
- Q: Which delivery channel is in scope? → A: Email only.
- Q: What happens when mail submission fails? → A: No usable new link is
  issued; the administrator may retry deliberately.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Send setup or reset delivery after active creation (Priority: P1)

An authorized administrator can explicitly request one secure password link for
an eligible, already active user after that user has been created, without
seeing or handling a reusable credential token. The system determines whether
the link performs setup or reset from the user's credential state.

**Why this priority**: An active user needs a safe path to establish or replace
credentials without requiring the administrator to know, set, or transmit a
password.

**Independent Test**: Create an active user, request credential delivery as an
authorized same-scope administrator, and verify one accepted delivery result
with no secret shown. Repeat for a denied, ineligible, or rate-limited target.

**Acceptance Scenarios**:

1. **Given** an authorized administrator has just created an eligible active
   user in the permitted scope, **When** they explicitly request a password
   link, **Then** the system sends one secure system-selected setup or reset
   link by email to the user's current address and shows only approved
   delivery-request feedback.
2. **Given** delivery succeeds, **When** the administrator views the result,
   **Then** no password, reusable token, setup URL, mailbox status, or
   tenant-private delivery diagnostic is shown or retained.
3. **Given** a target is inactive, invited, deleted, locked where credential
   delivery is disallowed, outside the actor's scope, or otherwise ineligible,
   **When** an administrator requests delivery, **Then** the system applies the
   documented denial or conflict behavior without issuing a link.

---

### User Story 2 - Complete delivered credential setup or reset (Priority: P2)

A recipient can use a valid, single-use delivery link to choose a compliant new
password and then continue through the approved sign-in path.

**Why this priority**: Delivery is useful only when recipients can safely
complete the credential change without account or tenant disclosure.

**Independent Test**: Use a valid link to set a password, then exercise
missing, expired, reused, revoked, malformed, and rate-limited link outcomes.

**Acceptance Scenarios**:

1. **Given** a recipient opens a valid unexpired link, **When** they submit a
   compliant password, **Then** the password is accepted once, existing trusted
   sessions are invalidated as required, and the recipient is guided to sign
   in.
2. **Given** a link is missing, invalid, expired, reused, revoked, or
   superseded, **When** the recipient opens or submits the completion flow,
   **Then** the system shows the approved neutral invalid-link state without
   revealing user, school, role, or account details.
3. **Given** a recipient retries after a validation failure, **When** the link
   remains valid, **Then** field-level password feedback is shown without
   exposing the link value or retaining the attempted password beyond the
   active form.

---

### User Story 3 - Preserve privacy and delivery controls (Priority: P3)

Administrators and recipients receive safe feedback when delivery is denied,
unavailable, or rate limited, while public password-reset behavior remains
non-enumerating.

**Why this priority**: Credential delivery must not reveal account existence,
tenant membership, or delivery infrastructure details.

**Independent Test**: Exercise authorized and unauthorized administrative
requests plus public reset requests for existing and non-existing identifiers,
and verify the documented feedback and zero secret exposure.

**Acceptance Scenarios**:

1. **Given** an actor lacks required authority or active tenant context, **When**
   they open or submit password-delivery controls, **Then** the control is not
   available or the request is safely denied without revealing target details.
2. **Given** delivery limits are reached or mail submission cannot be accepted,
   **When** a request is made, **Then** the system reports only documented safe
   feedback and does not claim that an email was delivered.
3. **Given** a signed-out person requests a password reset, **When** the email
   is eligible, missing, inactive, locked, deleted, or rate limited, **Then**
   the public response remains indistinguishable and non-enumerating.

### Edge Cases

- An administrator submits delivery twice, retries after a temporary failure,
  navigates away, changes school, loses permission, or signs out while a
  delivery request is pending.
- A user address changes between active-user creation and delivery request.
- A delivery link is opened in another browser or after a newer delivery link
  supersedes it.
- A recipient uses password-manager input, paste, or browser autofill.
- An error payload, browser URL, page, log, analytics event, or storage entry
  contains a password, reusable link value, target identifier, or private mail
  diagnostic.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Add the approved, authorization- and
  tenant-scoped credential-delivery behavior for eligible active users,
  including delivery limits, invalidation of superseded links, mail submission,
  safe outcomes, audit-relevant records, and automated coverage.
- **Frontend repository impact**: Add authorized post-create and user-detail
  delivery controls, safe request/result states, delivered-link completion,
  stale-context protection, and component, service, and browser coverage.
- **Specification or contract repository impact**: Define the delivery
  operation, request and response envelopes, eligibility, rate limits, token
  lifecycle, password completion semantics, authorization, and safe errors in
  OpenAPI before implementation.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines the
  contract first. `schoolmaster-backend` implements and verifies delivery
  behavior on branch `039-user-password-delivery`; then
  `schoolmaster-frontend` implements only the verified contract on the branch
  with the same name.

### API Contract Impact

- **OpenAPI update required**: Yes. Add a documented administrator credential
  delivery operation for an eligible active user and update only the existing
  public reset-completion contract if shared token semantics require it.
- **Versioned endpoints affected**: Existing user creation/detail, password
  reset request/completion, and a new documented `/api/v1` user credential
  delivery operation.
- **JSON response impact**: Delivery responses expose only approved status,
  delivery channel, and request time. Tokens, setup URLs, passwords, mail
  provider diagnostics, and private target details never appear.
- **Authentication/authorization impact**: Administrative delivery requires
  documented scoped account-lifecycle authority and active tenant context for
  school-owned users. Public reset remains unauthenticated and non-enumerating.
- **Compatibility impact**: Additive. Existing active creation, invitation
  creation, invitation setup, and public reset flows remain compatible.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-owned delivery requests require an active
  matching school context; platform behavior follows separately documented
  platform authority.
- **Cross-tenant or platform access impact**: No cross-tenant lookup or
  delivery is allowed. Platform actors use only explicitly authorized platform
  targets.
- **Soft delete impact**: Soft-deleted users are ineligible and receive no
  credential delivery.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST preserve active user creation without automatic
  credential delivery.
- **FR-002**: The system MUST allow only authorized administrators to
  explicitly request secure credential delivery for an eligible active user in
  their permitted scope.
- **FR-003**: The system MUST expose one administrator password-link action and
  determine setup or reset internally from the target's credential state. It
  MUST create a single-use, expiring credential link and deliver it only to the
  target user's current address; the link value MUST not be returned, displayed,
  logged, or persisted in client-visible state.
- **FR-004**: The system MUST invalidate or supersede older outstanding
  credential links when a newer delivery is successfully issued.
- **FR-004a**: This feature MUST use email only and MUST NOT add SMS,
  notification-center, or per-school delivery-channel selection.
- **FR-005**: The system MUST allow no more than 3 password-link deliveries per
  user and scope within 24 hours, apply documented limits to completion
  attempts, and MUST not claim inbox delivery when mail submission is rejected
  or unavailable. A rejected or unavailable submission MUST issue no usable new
  link and MUST allow a deliberate retry.
- **FR-006**: The system MUST allow a valid recipient to set a compliant
  password once and MUST invalidate affected active sessions on success.
- **FR-006a**: The system MUST reject password-link delivery for a locked user;
  a separately authorized unlock or recovery is required before delivery.
- **FR-007**: The system MUST provide neutral invalid-link feedback for every
  missing, malformed, expired, reused, revoked, mismatched, or superseded link.
- **FR-008**: The system MUST preserve the existing non-enumerating public
  password-reset request response across all eligibility and rate-limit states.
- **FR-009**: The system MUST reject delivery for unauthorized, cross-tenant,
  inactive, invited, soft-deleted, or otherwise ineligible users without
  issuing a link or exposing private target details.
- **FR-010**: The system MUST invalidate pending frontend delivery state when
  its target, actor, authorization, tenant context, or route changes.
- **FR-011**: The system MUST define changed REST behavior in OpenAPI before
  backend implementation and must verify backend behavior before frontend
  implementation begins.
- **FR-012**: The system MUST cover contract, backend, frontend, privacy,
  authorization, tenant, rate-limit, and stale-request outcomes with automated
  tests.

### Key Entities *(include if feature involves data)*

- **Credential Delivery Request**: An auditable request for one secure setup or
  reset link for an eligible active user, with safe status and request time.
- **Credential Link**: A single-use, expiring secret proof tied to one user and
  invalidated when consumed, revoked, or superseded.
- **Password Completion**: One successful use of a valid credential link to
  establish or replace a password and invalidate affected sessions.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In 100% of authorized, eligible delivery tests, exactly one link
  is submitted to the target address and no secret value appears in an
  administrator-visible response, page, storage entry, or diagnostic.
- **SC-002**: In 100% of denied, ineligible, cross-tenant, and rate-limited
  delivery tests, no link is issued and no private target or delivery-provider
  detail is exposed.
- **SC-003**: In 100% of valid completion tests, recipients can set a compliant
  password and are directed to sign in; replaying the same link never succeeds.
- **SC-004**: Public password-reset request responses remain indistinguishable
  for all tested account-existence and eligibility states.
- **SC-005**: Automated backend and frontend critical-flow suites cover 100% of
  documented authorization, tenant, link-lifecycle, stale-request, and
  no-secret scenarios before release.

## Assumptions

- Existing password rules and the approved guest reset-completion pattern are
  reused unless contract planning identifies a necessary additive change.
- Authorized delivery is an explicit post-create or user-detail action; active
  user creation itself never sends mail automatically.
- The administrator sees delivery-request acceptance, not inbox delivery.
- Existing invitation setup stays dedicated to invited users; this feature does
  not add token-based administrator resend or alter invitation eligibility.
- Email is the sole delivery channel; SMS and per-school channel selection are
  out of scope.
- Backend and frontend branches use `039-user-password-delivery`; backend
  implementation and verification complete before frontend implementation.
