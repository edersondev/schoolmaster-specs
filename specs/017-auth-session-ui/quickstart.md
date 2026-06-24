# Quickstart: Authentication and Session Foundation UI

Use this checklist when planning or reviewing `schoolmaster-frontend`
implementation for roadmap item 3.

## 1. Confirm Scope

- Start from `specs/017-auth-session-ui/spec.md`.
- Use `specs/015-frontend-architecture-baseline/` and
  `specs/016-admin-shell-dashboard/` as the governing frontend baselines.
- Use `docs/frontend-architecture.md`, `docs/frontend-guidelines.md`,
  `docs/frontend-admin-system-architecture.md`, and
  `docs/naming-conventions.md` as supporting guidance.
- Do not add password reset completion, password setup, invitations,
  reactivation, account lock administration, backend code, TypeScript, or
  undocumented API consumption.

## 2. Verify Approved API Operations

Frontend auth services may consume only:

- `POST /api/v1/auth/login`
- `GET /api/v1/auth/me`
- `POST /api/v1/auth/logout`
- `POST /api/v1/auth/password-reset-requests`

Confirm each service maps the published OpenAPI request, response, and error
envelope before any page, component, store, or route guard consumes the result.

### Implementation Review — 2026-06-23

- Confirmed `login`, `getCurrentUser`, `logout`, and
  `requestPasswordReset` exist with the documented operation IDs, request
  schemas, success envelopes, and auth-specific error responses.
- Confirmed `getCurrentUser` accepts the optional `X-School-Id` header and
  returns the shared `AuthSession` schema.
- School selection remains blocked. The existing `listSchools` operation is
  documented for platform-scope authorization and is not confirmed as a
  user-authorized school source across the intended user types.
- No OpenAPI change is made by this frontend slice.

## 3. Verify Session Bootstrap

The implementation should:

- keep protected content hidden while bootstrap is pending
- hydrate current user, roles, permissions, and resolved school from
  `AuthSession`
- choose authenticated or unauthenticated layout from session and route state
- clear stale protected state after token rejection, inactive-user,
  inactive-school, or tenant-mismatch outcomes
- treat client-side permission checks as navigation/visibility only

## 4. Verify Active School Context

- Restore the last approved active school only if the authenticated response
  confirms it remains authorized.
- Require school selection before tenant-owned content renders when no
  authorized active school can be restored.
- Populate school selection choices only from an approved OpenAPI operation
  that returns schools authorized for the current user.
- Block school-selection implementation when no approved source operation is
  confirmed.
- Use the documented tenant context header only as a request for explicit school
  context.
- Do not infer tenant access client-side.

## 5. Verify Protected Route Preservation

- Signed-out direct access to a protected route preserves the originally
  requested route through sign-in.
- Expired-session recovery preserves the originally requested route through
  re-authentication.
- Restore the requested route only when current user, permissions, and tenant
  context still authorize it.
- Send the user to an allowed authenticated workspace when the requested route
  is no longer authorized.

## 6. Verify Forgot-Password Entry

- Valid email submissions show neutral accepted confirmation.
- Invalid email format shows field-level validation guidance.
- The UI must not reveal whether an account exists, is inactive, is locked, is
  deleted, is unauthorized, or is over a request threshold.
- Password reset completion is not part of this feature.

## 7. Verify Error and Feedback States

The UI should support:

- validation
- invalid credentials
- lockout
- expired session
- unauthorized
- forbidden
- inactive user
- inactive school
- tenant mismatch
- temporary unavailable
- neutral confirmation

Reusable messages must be centralized for Vue I18n and must not expose tenant,
permission, account-existence, password, or token details.

## 8. Suggested Review Commands

After implementation exists in `schoolmaster-frontend`, use checks like:

```bash
rg "axios" src/pages src/components src/layouts src/router
rg "<el-" src
rg "password-reset|auth/me|auth/login|auth/logout" src/services src/stores src/pages
rg "localStorage|sessionStorage" src/stores src/services src/router
rg "token|password|Authorization" src/pages src/components src/layouts
```

Expected review result:

- no direct Axios usage in pages/components/layouts/router guards
- no kebab-case Element Plus tags
- approved auth endpoint usage is isolated to services
- persistence follows the approved authentication contract and security
  guidance; non-sensitive tenant metadata is limited to last approved active
  school restoration
- token/password values are not exposed in presentation components

Affected auth services, session store, route guards, tenant context restoration,
denied-state mapping, and forgot-password entry behavior should have Vitest
coverage when implemented in `schoolmaster-frontend`.

## 9. Implementation Results — 2026-06-23

- `npm run test:unit -- --run`: PASS, 24 test files and 62 tests.
- Auth/session subset: PASS, 14 test files and 42 tests.
- `npm run build`: PASS. Vite reported only dependency annotation and existing
  large-chunk advisory warnings.
- `npx eslint src tests/unit/auth --no-cache`: PASS.
- `npx oxlint src tests/unit/auth`: PASS.
- Sign-in and bootstrap tests include assertions against the 30-second target.
- Password reset request tests include assertions against the 15-second target.
- Automated route-recovery tests confirm authorized restoration and fallback
  behavior. The SC-006 90% participant UAT target remains a manual product UAT
  measure because no participant panel is available in this implementation
  environment.

## 10. Architecture and Security Review — 2026-06-23

- No direct Axios usage exists in auth pages, components, layouts, or guards.
- All Element Plus tags in auth Vue files use PascalCase.
- Auth endpoint strings exist only in `src/services/auth/authService.js`.
- Client persistence is limited to
  `schoolmaster.auth.lastApprovedSchoolId`; passwords, bearer tokens, and
  Authorization headers are not persisted.
- Presentation files contain password form handling only; they do not expose
  tokens or Authorization headers.
- Shared auth/session text is centralized in `src/locales/auth.js`.
- Contract review confirms login, current-user, logout, password-reset request,
  session errors, requested-route recovery, and tenant-context gating.
- School-selection choices remain blocked because no approved operation is
  confirmed for all intended user types.
- OpenAPI was not changed, so Redocly validation is not applicable.
