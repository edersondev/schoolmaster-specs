# Feature Specification: Authentication and Session Foundation UI

**Feature Branch**: `017-auth-session-ui`  
**Created**: 2026-06-23  
**Status**: Draft  
**Input**: User description: "Define the Authentication and Session Foundation UI for the SPA: login, forgot-password, authenticated session bootstrap, current-user and permission hydration, session expiration handling, and layout selection. Cover UI behavior for session-expired, unauthorized, forbidden, inactive-user, and tenant-mismatch states while consuming only approved auth, current-user, permission, and tenant-context endpoints."

## Clarifications

### Session 2026-06-23

- Q: How should the SPA resolve active school context when a user can access more than one school? → A: Restore the last approved active school if still authorized; otherwise require school selection before tenant-owned content renders.
- Q: After session expiration, where should the user go after signing in again? → A: Send the user to sign in and return them to the originally requested route only if still authorized.
- Q: How should direct access to a protected route behave for signed-out users? → A: Preserve the originally requested protected route and return there after sign-in only if still authorized.
- Q: Where should school selection choices come from when the user must choose an active school? → A: Choices must come from an approved OpenAPI operation that returns only schools authorized for the current user; if no such contract is confirmed, implementation is blocked until OpenAPI is updated.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Sign in to the correct workspace (Priority: P1)

An unauthenticated user can access the sign-in screen, submit valid credentials,
and land in the correct authenticated workspace for their account and tenant
context.

**Why this priority**: Protected administration and self-service surfaces cannot
be used until users can authenticate and enter the correct workspace.

**Independent Test**: Can be fully tested by starting from a signed-out state,
submitting valid and invalid sign-in attempts, and verifying the resulting
authenticated or denied state.

**Acceptance Scenarios**:

1. **Given** a signed-out user with a valid active account, **When** they submit
   valid credentials, **Then** they are signed in and taken to the correct
   authenticated workspace for their role and tenant context.
2. **Given** a signed-out user, **When** they submit invalid credentials, **Then**
   no session is created and the user sees a safe denial message without account
   enumeration.
3. **Given** a signed-out user with an inactive account, **When** they submit
   otherwise valid credentials, **Then** they are blocked and shown the
   inactive-user state.

---

### User Story 2 - Resume an existing session (Priority: P2)

A returning authenticated user can open the application, have their current
user, permissions, and tenant context loaded, and see the correct layout without
manually signing in again.

**Why this priority**: The admin shell and future protected pages depend on
reliable session bootstrap before rendering role-aware navigation or content.

**Independent Test**: Can be fully tested by starting with an existing valid
session, opening a protected surface, and verifying the user context,
permissions, tenant context, and layout selection are resolved before protected
content is shown.

**Acceptance Scenarios**:

1. **Given** a user has a valid session, **When** they open a protected route,
   **Then** the application loads the current user, permissions, and tenant
   context before showing protected content.
2. **Given** a user has a valid session but lacks permission for a protected
   surface, **When** they navigate to that surface, **Then** they see the
   forbidden state and protected content remains hidden.
3. **Given** a user has a valid account but the tenant context is not valid for
   the requested surface, **When** session bootstrap completes, **Then** the user
   sees the tenant-mismatch state instead of tenant-owned content.
4. **Given** a user has a valid session but the selected or resolved school is
   inactive, **When** session bootstrap completes, **Then** the user sees the
   inactive-school state and tenant-owned content remains hidden.
5. **Given** a user can access more than one school, **When** the last approved
   active school is still authorized, **Then** that school is restored before
   tenant-owned content renders.
6. **Given** a user can access more than one school and no last approved active
   school is authorized, **When** session bootstrap completes, **Then** the user
   must select an authorized school before tenant-owned content renders.
7. **Given** school selection is required, **When** the frontend displays
   selectable schools, **Then** those choices come only from an approved
   OpenAPI operation that returns schools authorized for the current user.
8. **Given** school selection is required but no approved school-selection
   contract is confirmed, **When** implementation is reviewed, **Then** the
   school-selection UI is blocked until OpenAPI defines the source contract.

---

### User Story 3 - Recover from session and authorization failures (Priority: P3)

A user whose session expires or whose request is denied receives clear recovery
guidance and can return to a valid sign-in or authorized area without losing
orientation.

**Why this priority**: Users need predictable recovery behavior when sessions
expire or access is denied, especially inside protected operational workflows.

**Independent Test**: Can be fully tested by forcing expired, unauthorized,
forbidden, inactive-user, inactive-school, and tenant-mismatch states and
verifying the displayed message, recovery action, and protected-content
behavior for each state.

**Acceptance Scenarios**:

1. **Given** an authenticated user whose session has expired, **When** they
   attempt to view protected content, **Then** the session-expired state is
   shown and the user can return to sign in.
2. **Given** a user signs in again after session expiration, **When** the
   originally requested route is still authorized for their current session and
   tenant context, **Then** they return to that route.
3. **Given** a user signs in again after session expiration, **When** the
   originally requested route is no longer authorized, **Then** they are taken
   to an allowed authenticated workspace instead of the denied route.
4. **Given** a signed-out user attempts to open a protected route, **When** no
   valid session exists, **Then** the unauthorized state sends the user to sign
   in while preserving the originally requested route.
5. **Given** a user reaches a denied state, **When** they follow the provided
   recovery action, **Then** they are taken to the appropriate sign-in or
   allowed workspace surface.
6. **Given** a signed-out user signs in after opening a protected route
   directly, **When** the originally requested route is authorized for their
   current session and tenant context, **Then** they return to that route.
7. **Given** a signed-out user signs in after opening a protected route
   directly, **When** the originally requested route is not authorized, **Then**
   they are taken to an allowed authenticated workspace instead of the denied
   route.

---

### User Story 4 - Start password recovery (Priority: P4)

A signed-out user who forgot their password can start the approved recovery
entry flow and receive a confirmation that does not disclose whether an account
exists.

**Why this priority**: Password recovery is important for account continuity,
but the full account lifecycle flows are covered by a later roadmap item.

**Independent Test**: Can be fully tested by submitting the recovery entry form
with different email addresses and verifying the same safe confirmation outcome.

**Acceptance Scenarios**:

1. **Given** a signed-out user on the forgot-password screen, **When** they
   submit a syntactically valid email address, **Then** they see a neutral
   confirmation message.
2. **Given** a signed-out user submits an invalid email format, **When** the
   form is validated, **Then** they see field-level guidance and no recovery
   request is sent.

### Edge Cases

- Session expires during initial bootstrap.
- Current-user data loads but permissions or tenant context cannot be confirmed.
- A user has multiple authorized schools and the last approved active school is
  unavailable or no longer authorized.
- School selection is required but no approved OpenAPI operation for
  user-authorized school choices is confirmed.
- A user has multiple possible workspaces or roles and only one is valid for the
  requested surface.
- A protected route is opened directly from a bookmark while signed out.
- A signed-in user visits a guest-only screen such as login or forgot password.
- Authentication or recovery requests fail because the service is temporarily
  unavailable.
- A denial response is received after protected content was previously visible.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is introduced by this
  specification unless contract review finds missing approved authentication,
  current-user, permission, tenant-context, or error-state behavior.
- **Frontend repository impact**: Adds the unauthenticated entry surfaces,
  protected-session bootstrap behavior, permission-aware layout selection,
  denied-state screens, and recovery entry behavior for the SPA.
- **Specification or contract repository impact**: This specification defines
  the required frontend behavior and blocks implementation on matching approved
  OpenAPI coverage for all consumed authentication, current-user, permission,
  tenant-context, and denial-state responses.
- **Delivery ownership and sequencing**: Contract review leads first, frontend
  implementation follows only after approved endpoint and error semantics are
  confirmed. Backend changes, if needed, must be tracked separately before the
  frontend consumes them.

### API Contract Impact

- **OpenAPI update required**: Yes if the existing contract does not already
  define every consumed authentication, current-user, permission, tenant-context,
  password-recovery entry, school-selection source, and denial-state response
  required by this feature.
- **Versioned endpoints affected**: No new path is defined by this spec. The
  approved versioned authentication, current-user, permission, tenant-context,
  password-recovery entry, and user-authorized school-selection operations must
  be identified during planning.
- **JSON response impact**: The frontend requires consistent success and failure
  envelopes for valid session, invalid credentials, expired session,
  unauthorized, forbidden, inactive-user, inactive-school, tenant-mismatch,
  validation, and temporary-unavailable outcomes.
- **Authentication/authorization impact**: Defines the frontend behavior for
  signed-out, signed-in, expired, unauthorized, forbidden, inactive-user,
  inactive-school, and tenant-mismatch states. It does not loosen authorization
  or tenant isolation.
- **Compatibility impact**: Frontend behavior is additive. Any backend or
  contract correction discovered during review must remain backward compatible
  unless a separate migration plan approves otherwise.

### Data & Tenancy Impact

- **Tenant scoping impact**: The selected authenticated workspace must match the
  approved tenant context before tenant-owned surfaces render.
- **Cross-tenant or platform access impact**: No cross-tenant access is added.
  Platform or support access, if present for an authenticated user, must be
  represented only through approved permission and tenant-context responses.
- **Soft delete impact**: No direct soft-delete behavior is introduced.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST provide a sign-in surface for unauthenticated
  users.
- **FR-002**: The system MUST validate sign-in input before attempting
  authentication and display field-level guidance for invalid input.
- **FR-003**: The system MUST create an authenticated frontend state only after
  authentication succeeds and the current user, permissions, and tenant context
  are confirmed.
- **FR-004**: The system MUST prevent protected content from rendering until
  session bootstrap has completed successfully.
- **FR-004a**: When a user can access more than one school, the system MUST
  restore the last approved active school only if it remains authorized;
  otherwise, the system MUST require school selection before tenant-owned
  content renders.
- **FR-004b**: School selection choices MUST come only from an approved OpenAPI
  operation that returns schools authorized for the current user; if no such
  contract is confirmed, school-selection implementation MUST be blocked until
  OpenAPI is updated.
- **FR-005**: The system MUST select the appropriate authenticated or
  unauthenticated layout based on session state and route access requirements.
- **FR-006**: The system MUST send signed-out users who attempt to open
  protected surfaces to the sign-in flow while preserving the originally
  requested route.
- **FR-006a**: After direct protected-route access from a signed-out state, the
  system MUST return a user to the originally requested route only when the
  authenticated session remains authorized for that route and tenant context.
- **FR-007**: The system MUST show a safe denial message for invalid credentials
  without confirming whether the account exists.
- **FR-008**: The system MUST show a distinct inactive-user state when an
  otherwise valid account is not allowed to sign in.
- **FR-008a**: The system MUST show a distinct inactive-school state when the
  authenticated session is otherwise valid but the selected or resolved school
  is inactive.
- **FR-009**: The system MUST show a distinct session-expired state when a prior
  authenticated session is no longer valid.
- **FR-009a**: After session expiration, the system MUST return a user to the
  originally requested route only when the renewed session remains authorized
  for that route and tenant context.
- **FR-010**: The system MUST show a distinct unauthorized state when no valid
  authenticated session is available for a protected surface.
- **FR-011**: The system MUST show a distinct forbidden state when an
  authenticated user lacks permission for a protected surface.
- **FR-012**: The system MUST show a distinct tenant-mismatch state when the
  authenticated context does not allow the requested tenant-scoped surface.
- **FR-013**: The system MUST provide recovery actions from session-expired,
  unauthorized, forbidden, inactive-user, inactive-school, and tenant-mismatch
  states.
- **FR-014**: The system MUST provide a forgot-password entry surface with
  neutral confirmation messaging for valid submissions.
- **FR-015**: The system MUST consume only approved authentication,
  current-user, permission, tenant-context, user-authorized school-selection,
  and password-recovery entry contracts.
- **FR-016**: The system MUST define any changed REST contract in OpenAPI before
  implementation begins.
- **FR-017**: The system MUST preserve consistent JSON responses for success and
  failure cases.
- **FR-018**: The system MUST preserve tenant isolation and document any
  intentional cross-tenant access path.
- **FR-019**: The system MUST identify all affected repositories and the delivery
  sequence when implementation spans more than one repository.

### Key Entities *(include if feature involves data)*

- **Authenticated Session**: Represents whether a user is currently signed in
  and allowed to access protected surfaces.
- **Current User**: Represents the signed-in person, including account status
  and display identity needed by the frontend.
- **Permission Set**: Represents the actions and surfaces the signed-in user is
  allowed to access.
- **Tenant Context**: Represents the school or platform context in which the
  current session is operating, including the last approved active school when
  it remains authorized.
- **Authentication State**: Represents signed-out, bootstrapping, signed-in,
  expired, unauthorized, forbidden, inactive-user, inactive-school, and
  tenant-mismatch UI states.
- **Password Recovery Request**: Represents a signed-out user's request to start
  password recovery without exposing account existence.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: At least 95% of users with valid active accounts can sign in and
  reach their correct authenticated workspace in under 30 seconds.
- **SC-002**: 100% of protected surfaces tested prevent content visibility until
  current user, permissions, and tenant context are confirmed.
- **SC-003**: 100% of denied-state tests display the correct state for expired
  session, unauthorized, forbidden, inactive-user, inactive-school, and
  tenant-mismatch outcomes.
- **SC-004**: At least 90% of tested password-recovery entry attempts with
  syntactically valid email addresses result in a neutral confirmation in under
  15 seconds.
- **SC-005**: Zero tested invalid credential or password-recovery messages reveal
  whether a specific account exists.
- **SC-006**: At least 90% of user acceptance test participants can recover from
  session expiration and return to sign-in or an allowed workspace without
  assistance.

## Assumptions

- Authentication is based on approved first-party schoolmaster account flows;
  external identity provider behavior is outside this slice unless already
  approved by contract.
- Password setup, password reset completion, invitations, account locking, and
  broader account lifecycle workflows remain in the later account lifecycle UI
  feature.
- The admin shell and dashboard foundation from
  `specs/016-admin-shell-dashboard/` is available as the protected
  administration layout target.
- Current-user, permission, and tenant-context semantics are the source of truth
  for frontend layout and route visibility.
- Contract review may identify backend or OpenAPI work before frontend
  implementation; the frontend must not consume undocumented behavior.
