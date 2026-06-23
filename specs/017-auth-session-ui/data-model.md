# Data Model: Authentication and Session Foundation UI

This feature does not introduce database entities. The model below defines
frontend concepts, session state, and OpenAPI consumption boundaries for the
SPA authentication and session foundation.

## AuthSessionState

**Purpose**: Shared frontend state representing the authenticated session and
bootstrap lifecycle.

**Fields**:

- `status`: Signed-out, bootstrapping, authenticated, expired, unauthorized,
  forbidden, inactive-user, inactive-school, tenant-mismatch, selecting-school,
  or unavailable.
- `tokenExpiresAt`: Expiration timestamp from the approved auth contract.
- `currentUser`: Current user identity once confirmed.
- `roles`: Role array from the approved auth contract.
- `permissions`: Permission array from the approved auth contract.
- `activeSchool`: Resolved active school when confirmed by an authenticated
  response.
- `lastApprovedSchoolId`: Approved persisted school identifier used only to
  request restoration and only if still authorized.
- `requestedRoute`: Originally requested protected route pending sign-in or
  re-authentication.
- `feedbackState`: Current auth/session feedback state for the UI.

**Relationships**:

- Owns `CurrentUser`, `PermissionSet`, `TenantContext`, `RequestedRoute`, and
  `AuthFeedbackState`.
- Uses `AuthApiContract` service mapping.
- Feeds route guards and layout selection.

**Validation rules**:

- Protected content must not render while `status` is `bootstrapping`,
  `selecting-school`, or any denial state.
- `activeSchool` must come from an authenticated response, not client-side
  inference.
- `lastApprovedSchoolId` may request restoration but must not itself authorize
  tenant-owned rendering.
- Sensitive token material must not be exposed to presentation components.

## CurrentUser

**Purpose**: Signed-in person and account status used by the frontend.

**Fields**:

- `id`: Public UUID from the contract.
- `fullName`: Display name.
- `email`: Email address.
- `status`: Account status.
- `schoolId`: Optional school association from the contract.
- `roles`: Role summaries exposed on the user record.

**Relationships**:

- Belongs to `AuthSessionState`.
- Contributes to header or session affordance display.

**Validation rules**:

- Inactive users must not enter protected content.
- UI messages must not reveal account existence during failed login or password
  reset request entry.

## PermissionSet

**Purpose**: Permission values used for client-side route and navigation
visibility.

**Fields**:

- `permissions`: Array of permission records with id, code, name, scope, and
  status.
- `roles`: Array of role records with id, scope, name, status, and permissions.
- `hasPermission`: Derived client-side visibility check.

**Relationships**:

- Derived from `AuthSessionState`.
- Used by `ProtectedRouteRule`, admin shell navigation, and future module route
  metadata.

**Validation rules**:

- Client-side permission checks are visibility and navigation controls only.
- Backend authorization remains authoritative.
- Inactive or absent permission values must not grant access.

## TenantContext

**Purpose**: Active school or platform context for the current session.

**Fields**:

- `activeSchool`: Confirmed resolved school or null for platform context.
- `requestedSchoolId`: Optional school id sent through approved tenant context
  semantics.
- `requiresSchoolSelection`: Whether the user must choose an authorized school
  before tenant-owned content can render.
- `schoolSelectionSource`: Approved OpenAPI operation used to fetch only
  schools authorized for the current user.
- `tenantStatus`: Resolved, missing, inactive, mismatch, or selecting.

**Relationships**:

- Belongs to `AuthSessionState`.
- Uses the `X-School-Id` tenant context header where the OpenAPI contract
  requires an explicit school context.
- Gates tenant-owned protected routes.

**Validation rules**:

- Restore the last approved active school only if the authenticated response
  confirms authorization.
- Require school selection when no authorized last approved school can be
  restored for a multi-school user.
- School selection choices must come only from an approved OpenAPI operation
  that returns schools authorized for the current user.
- School-selection UI must remain blocked until the approved source operation is
  confirmed in OpenAPI.
- Tenant mismatch must clear tenant-owned content from view.

## ProtectedRouteRule

**Purpose**: Route metadata and guard decision input for protected frontend
surfaces.

**Fields**:

- `requiresAuth`: Whether the route requires a valid session.
- `requiresSchoolContext`: Whether the route requires a confirmed active
  school.
- `requiredPermissions`: Permission codes required for visibility and route
  access.
- `layout`: Authenticated or unauthenticated layout selection.
- `fallbackRoute`: Allowed authenticated workspace when the requested route is
  no longer authorized.

**Relationships**:

- Reads `AuthSessionState`, `PermissionSet`, and `TenantContext`.
- Produces `RequestedRoute` and `AuthFeedbackState` when access is blocked.

**Validation rules**:

- Signed-out direct access must preserve the originally requested protected
  route.
- Expired-session access must preserve the originally requested protected
  route.
- The requested route may be restored only after renewed authentication confirms
  authorization and tenant context.

## RequestedRoute

**Purpose**: The protected route a user attempted to reach before sign-in or
re-authentication.

**Fields**:

- `routeName`: Intended route name.
- `routeParams`: Intended route parameters.
- `routeQuery`: Intended route query.
- `requiresSchoolContext`: Whether restoration needs an active school.
- `requiredPermissions`: Permissions needed before restoration.
- `createdFrom`: Signed-out direct access or expired-session recovery.

**Relationships**:

- Stored in `AuthSessionState` until consumed or cleared.
- Validated by `ProtectedRouteRule` after authentication.

**Validation rules**:

- Must be cleared when restored successfully.
- Must be cleared when no longer authorized and the user is sent to an allowed
  workspace.
- Must not bypass permission or tenant checks.

## LoginForm

**Purpose**: User input for sign-in.

**Fields**:

- `email`: Required email address.
- `password`: Required password with the contract minimum.
- `schoolId`: Optional school UUID when the user is signing in to a specific
  school context.

**Relationships**:

- Submitted through `AuthApiContract.login`.
- Produces `AuthSessionState` or `AuthFeedbackState`.

**Validation rules**:

- Must validate email and password fields before submission.
- Invalid credentials must use safe denial messaging without account
  enumeration.
- Lockout responses must show retry guidance without exposing account state.

## PasswordRecoveryRequest

**Purpose**: Forgot-password entry request.

**Fields**:

- `email`: Required email address.
- `schoolId`: Optional school UUID when recovery is school-context-specific.
- `confirmationState`: Neutral accepted confirmation state.

**Relationships**:

- Submitted through `AuthApiContract.requestPasswordReset`.
- Produces neutral confirmation or validation feedback.

**Validation rules**:

- Valid submissions show the same neutral accepted confirmation regardless of
  account existence, activity, lock, deletion, or rate-limit state.
- Invalid email format shows field-level validation guidance.
- Password reset completion is out of scope for this feature.

## AuthFeedbackState

**Purpose**: User-facing state for auth and session outcomes.

**Fields**:

- `state`: Validation, invalid-credentials, lockout, expired-session,
  unauthorized, forbidden, inactive-user, inactive-school, tenant-mismatch,
  unavailable, or neutral-confirmation.
- `messageKey`: Centralized reusable UI text key.
- `severity`: Informational, warning, or error.
- `recoveryAction`: Sign in, choose school, go to allowed workspace, retry, or
  neutral confirmation.

**Relationships**:

- Derived from service error mapping, route guards, or password recovery
  request result.
- Rendered by auth pages, protected-route feedback, and shell feedback areas.

**Validation rules**:

- Must not expose sensitive tenant, permission, or account-existence details.
- Must map documented error codes consistently.

## AuthApiContract

**Purpose**: Frontend service boundary for approved OpenAPI auth operations.

**Fields**:

- `login`: Maps to `POST /api/v1/auth/login`.
- `getCurrentUser`: Maps to `GET /api/v1/auth/me`.
- `logout`: Maps to `POST /api/v1/auth/logout`.
- `requestPasswordReset`: Maps to
  `POST /api/v1/auth/password-reset-requests`.
- `listAuthorizedSchools`: Maps to an approved OpenAPI operation that returns
  only schools authorized for the current user when school selection is
  required.

**Relationships**:

- Produces `AuthSessionState`, `CurrentUser`, `PermissionSet`, `TenantContext`,
  `PasswordRecoveryRequest`, and `AuthFeedbackState`.

**Validation rules**:

- Must not call undocumented endpoints.
- Must block school selection when no approved user-authorized school source is
  confirmed.
- Must normalize success and error envelopes before stores or components
  consume results.
- Components, pages, layouts, and route guards must not call Axios directly.
