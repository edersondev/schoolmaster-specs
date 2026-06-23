# Research: Authentication and Session Foundation UI

## Decision: Implement this as a frontend-only auth/session slice

**Rationale**: The feature primarily defines SPA behavior: sign-in surfaces,
session bootstrap, current-user and permission hydration, active school context
selection, route guard behavior, and denied-state rendering. Existing OpenAPI
already documents the required auth and password reset request operations for
the planned frontend consumption.

**Alternatives considered**: Adding backend implementation to this slice was
rejected because no missing backend behavior is confirmed. Adding new OpenAPI
paths preemptively was rejected because the current contract already exposes
login, current user, logout, tenant context, token rejection, tenant mismatch,
and password reset request behavior.

## Decision: Consume existing OpenAPI operations for auth/session behavior

**Rationale**: The documented operations are sufficient for planning frontend
consumption:

- `POST /api/v1/auth/login` (`login`) authenticates and returns `AuthSession`.
- `GET /api/v1/auth/me` (`getCurrentUser`) returns current user, resolved
  school, roles, and permissions.
- `POST /api/v1/auth/logout` (`logout`) revokes the current bearer token.
- `POST /api/v1/auth/password-reset-requests` (`requestPasswordReset`) starts
  password recovery with non-enumerating `202` behavior.

Using these operation IDs keeps frontend services aligned to the published
contract and avoids undocumented route aliases.

**Alternatives considered**: Introducing frontend-local mock contracts was
rejected because implementation must consume OpenAPI-backed endpoints. Defining
new endpoint names in the plan was rejected because it would duplicate the
contract source of truth.

## Decision: Treat `AuthSession` as the session bootstrap source of truth

**Rationale**: `AuthSession` contains token expiration, current user, optional
resolved school, roles, and permissions. That is enough for the SPA to hydrate
session state, hide protected content until bootstrap completes, choose the
correct layout, and derive client-side visibility rules. Backend authorization
remains authoritative.

**Alternatives considered**: Splitting current-user, role, permission, and
tenant reads into separate frontend bootstrap requirements was rejected because
the current OpenAPI contract returns them together. Inferring permissions from
role names was rejected because permission arrays are explicitly exposed.

## Decision: Restore active school context only from approved sources

**Rationale**: The clarified behavior requires restoring the last approved
active school only if the current session remains authorized for it. The
frontend may use approved persisted session metadata to request a school
context, but the authenticated response must confirm the resolved school before
tenant-owned content renders. If no authorized school can be restored, the
frontend must require school selection.

**Alternatives considered**: Automatically selecting the first school was
rejected because it can place users in an unintended tenant context. Always
requiring school selection was rejected because it creates unnecessary friction
when a valid last approved school remains authorized.

## Decision: Source school selection choices only from an approved authorized-school contract

**Rationale**: When a user must choose an active school, the frontend needs a
tenant-safe source of selectable schools. Choices must come from an approved
OpenAPI operation that returns only schools authorized for the current user. If
no such operation is confirmed, school-selection implementation must block
until OpenAPI defines the source.

**Alternatives considered**: Reusing `listSchools` for every user was rejected
because its authorization scope must be confirmed before treating it as a
universal school-selection source. Manual school-id entry was rejected because
it is poor UX and increases tenant-enumeration risk. Showing tenant mismatch
until a future feature was rejected because this feature owns the school
selection gate when an approved source exists.

## Decision: Preserve requested protected routes only after authorization

**Rationale**: Both expired-session and signed-out direct-access flows should
preserve the originally requested protected route to reduce user disruption.
The route may be restored only after renewed authentication confirms user,
permission, and tenant context for that route; otherwise the user is sent to an
allowed authenticated workspace.

**Alternatives considered**: Always returning to the default dashboard was
rejected because it discards user intent. Returning to the requested route
without rechecking authorization was rejected because it can expose stale or
unauthorized route state.

## Decision: Map auth and tenant errors to explicit frontend states

**Rationale**: Existing error envelopes expose machine-readable codes such as
`validation_failed`, `forbidden`, `tenant_mismatch`, `auth_locked`,
`token_expired`, `token_revoked`, `inactive_user`, and `inactive_school`.
Frontend services and stores should normalize these into testable UI states:
validation, invalid credentials, lockout, expired session, unauthorized,
forbidden, inactive-user, inactive-school, tenant-mismatch, and temporary
unavailable.

**Alternatives considered**: Rendering raw backend messages directly was
rejected because UI text must be centralized and must not reveal sensitive
account or tenant details. Collapsing all failures into a generic error was
rejected because the spec requires distinct denial states.

## Decision: Keep forgot-password in this slice to request entry only

**Rationale**: The roadmap separates password reset completion, password setup,
invitations, reactivation, and account lock management into the later account
lifecycle workflows UI. This feature only needs the request entry screen and
neutral confirmation behavior backed by `requestPasswordReset`.

**Alternatives considered**: Implementing full password reset completion in
this slice was rejected because it belongs to the later account lifecycle UI.
Omitting forgot-password entirely was rejected because the roadmap item
explicitly includes forgot-password.

## Decision: Use frontend tests around services, store, guards, and state mapping

**Rationale**: The highest-risk implementation behavior is coordination:
service mapping of OpenAPI responses, Pinia session state transitions, route
guard preservation and authorization checks, active school context restoration,
and denial-state selection. Vitest coverage should focus on these boundaries.

**Alternatives considered**: Testing only page rendering was rejected because
it would miss route guard and state transition regressions. Deferring all tests
to end-to-end coverage was rejected because the specification requires focused
frontend service, store, and composable tests for affected behavior.
