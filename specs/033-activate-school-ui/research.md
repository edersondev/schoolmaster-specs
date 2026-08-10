# Research: School Context Selection UI

## Decision 1: Correct Current-Session Tenant Serialization First

**Decision**: Keep `GET /api/v1/auth/me` as the authoritative confirmation
operation, but preserve the `TenantContext` already returned by
`TenantContextResolver` and serialize its school as `resolved_school`. Make the
existing nullable property required in the OpenAPI `AuthSession` schema.

**Rationale**: The backend currently validates `X-School-Id` but discards the
resolved context. `AuthSessionResource` then reads `$user->school`, which is
null for a platform System Administrator. The frontend's exact confirmation
check therefore cannot succeed against the published flow. Reusing the
existing operation and `TenantContext` fixes conformance without a new route,
field, authorization rule, or persistence model.

**Alternatives considered**:

- Trust the chosen list item client-side: rejected because list visibility is
  not tenant-context confirmation and would bypass server authority.
- Add a new select-school endpoint: rejected because `/auth/me` already has the
  documented header, error behavior, and `resolved_school` response.
- Mutate the platform user's `school_id`: rejected because current context is
  request/session context, not ownership of the platform identity.

## Decision 2: Reuse the Platform School List Behind a Feature Service

**Decision**: A small frontend selection service delegates to the existing
`schoolsService.listSchools()` with `status=1`, explicit `page` and
`per_page=25`, and separate optional `name` and `inepCode` inputs. It is
callable only after exact active platform `System Administrator` recognition.

**Rationale**: `GET /api/v1/schools` already provides System Administrator
master access, active filtering, name contains filtering, normalized exact
INEP filtering, pagination, abort signals, and standard envelopes. A feature
wrapper fixes allowed parameters and prevents components from depending on the
broader administration query surface.

**Alternatives considered**:

- Enable generic `authService.listAuthorizedSchools()`: rejected because no
  general authorized-school source is approved for other actor types.
- Call Axios from the composable/page: rejected because project architecture
  isolates HTTP access in services.
- Send the same search text as name and INEP: rejected because backend filters
  combine with AND, which would hide valid matches.
- Rely on omitted `per_page`: rejected because OpenAPI defaults to 25 while the
  current backend service defaults to 15. The selector sends 25 explicitly;
  aligning the global default remains outside this feature.

## Decision 3: Normalize School Choice Data at Contract Boundaries

**Decision**: Map each choice to `{ id, name, inepCode, city, state, status }`.
Normalize `1`/`"1"` to `active` and `0`/`"0"` to `inactive` in both school-list
and auth-session mappings. Do not expose CNPJ/document to selector templates.

**Rationale**: The published School schema and Laravel resource use numeric
status, while the existing auth store expects string `active`. Existing tests
used a string fixture and masked the defect. A stable frontend model keeps
snake_case and numeric representation out of UI and state decisions.

**Alternatives considered**:

- Accept both forms in every component/store comparison: rejected because it
  duplicates contract knowledge and invites inconsistent security decisions.
- Render CNPJ as the existing list does: rejected by the clarified selector
  identity rule; name, INEP, city, and state are sufficient.

## Decision 4: Separate Query State from Confirmed Session State

**Decision**: `useSchoolSelection` owns filters, page, results, pagination
metadata, loading/error/empty state, AbortController, and request generation.
`sessionStore` remains the only owner of authenticated identity, active school,
confirmation generation, tenant resetters, and persisted preference.

**Rationale**: Search results are transient page state, not global authority.
Keeping only confirmed context in Pinia minimizes global state and prevents a
listed school from being mistaken for an active tenant. Generation checks and
abort handling prevent old list or confirmation responses from winning.

**Alternatives considered**:

- Store all choices globally in Pinia: rejected because cached platform-wide
  results would outlive filters, permissions, and lifecycle changes.
- Let the selector component own confirmation directly: rejected because route
  guards and all school-owned consumers need one confirmed context authority.

## Decision 5: Use the Dedicated Selector Route for Shell Switching

**Decision**: The persistent shell school control navigates to the same
dedicated selection route used for missing context. It captures the current
route before navigation. The actual context switch starts only after the user
chooses a school on that route.

**Rationale**: Existing unsaved-change and lifecycle-dialog protection is
route-leave based. Navigation to the selector triggers those guards before
`selectSchool()` clears context or cached tenant data. Canceling the navigation
therefore leaves the current school and its data unchanged. One selector also
avoids duplicate modal/page state, focus management, and error behavior.

**Alternatives considered**:

- Switch immediately from a header dropdown: rejected because it bypasses
  route-leave guards and can discard work before confirmation.
- Add a new global blocker registry: rejected for this scope because route
  navigation already supplies the required preflight boundary.
- Duplicate the selector in a shell dialog: rejected because the feature
  requires a persistent control, not a second UI implementation.

## Decision 6: Make Cross-School Route Safety Explicit and Default-Deny

**Decision**: Add centralized route compatibility evaluation. Platform routes
(`requiresSchoolContext !== true`) retain their named route and remain
platform-wide. School-owned routes retain only when explicitly marked as a
safe generic workspace/list and carry no tenant-owned params; otherwise route
to the school dashboard. Old query is cleared unless a route explicitly
declares context-neutral keys.

**Rationale**: Names and parameters on detail, edit, action, student, subject,
or academic-period routes belong to the old school. Guessing safety from path
strings risks cross-tenant lookup. Explicit metadata is reviewable and safe
when new routes are added.

**Alternatives considered**:

- Replay the old full path: rejected because it can carry old entity IDs.
- Preserve all list-looking route names heuristically: rejected because route
  naming is not an authorization or ownership contract.
- Always go to dashboard: safe but rejected because clarified behavior requires
  compatible platform and generic workspace/list routes to remain open.

## Decision 7: Bind Preference Lifetime to the Current Identity

**Decision**: Persist a school UUID only after exact active confirmation. Use
it solely as a fresh-confirmation request during bootstrap. Clear it whenever
logout, expiry, token rejection, lifecycle session cleanup, or identity change
clears/replaces the authenticated user.

**Rationale**: The current global localStorage key can leak a prior browser
user's selection into another login. Authorization is never inferred from the
stored value, but clearing it with identity state prevents cross-user restore
attempts and satisfies the clarified lifetime.

**Alternatives considered**:

- Store indefinitely: rejected because shared browsers can cross identities.
- Key only by user ID and retain after logout: rejected because the requirement
  ties retention to a valid session identity, not a durable user preference.

## Decision 8: Keep Selection Failures Distinct from Identity Loss

**Decision**: A 401/token rejection clears identity and preference. A denied,
invalid, inactive/mismatched, conflict, or temporary selection result commits
no school and leaves school-owned content blocked, but retains the signed-in
identity when the session remains valid so the administrator can retry.

**Rationale**: Existing `selectSchool()` applies a global denied state for all
errors, logging the operator out for recoverable selection failures. Contract-
specific feedback and retained identity provide safe recovery without treating
a proposed context as confirmed.

**Alternatives considered**:

- Clear identity on every failure: rejected because validation, tenant
  mismatch, lifecycle races, and network failures do not prove token loss.
- Restore the old active school after failed switching: rejected because prior
  school-owned data is deliberately cleared before confirmation and should not
  silently return.

## Decision 9: Lifecycle Changes Invalidate; They Never Auto-Select

**Decision**: Activation/restore only makes a school eligible after deliberate
selector refresh/new search. Successful deactivation/deletion of the current
school clears current context and tenant data, then requires selection.

**Rationale**: Lifecycle state and tenant context are separate authoritative
actions. Feature 020 already owns confirmation, reason, effective-date,
conflict, authorization, and audit behavior; feature 033 only integrates its
outcome with selector eligibility and active-context validity.

**Alternatives considered**:

- Auto-select after activation: rejected because activation is not a context
  selection confirmation.
- Leave a deactivated current school in frontend state until next request:
  rejected because it presents a known-invalid tenant as current.

## Decision 10: Accessible Paginated List, Not a Remote Select Dropdown

**Decision**: Use a labeled name input, labeled INEP input, explicit search and
refresh actions, a keyboard-operable result list, loading/empty/error regions,
and Element Plus pagination. All visible and assistive strings live in i18n.

**Rationale**: Each choice needs four identity fields, clear per-item selection
feedback, 100+ result pagination, and responsive wrapping. A full list is more
legible and testable than squeezing identity details into remote select
options. Element Plus pagination uses controlled current-page state; loading
and empty states remain explicit.

**Alternatives considered**:

- Remote `ElSelect`: technically supports filtering, loading, debounce, empty
  text, and end-reached, but is less suitable for multi-line identity and
  explicit server pagination.
- Client-side filtering of one page: rejected because it treats partial data as
  the full authorized choice set.

## Decision 11: Verification Separates Automation from Usability Evidence

**Decision**: PHPUnit, Redocly, Vitest, Playwright, build, and source audits
prove contract and behavior. Playwright runs selector scenarios at 390, 768,
and 1440 pixels and configured browsers. Manual NVDA/VoiceOver and moderated
representative-administrator sessions remain recorded evidence for the
screen-reader and 30-second/2-minute success criteria.

**Rationale**: Semantic DOM assertions and repeatable timing surrogates are
valuable but cannot honestly substitute for a screen reader or moderated user
outcome.

**Alternatives considered**:

- Claim automated accessibility assertions satisfy screen-reader review:
  rejected because no axe dependency exists and automation does not reproduce
  assistive-technology behavior.

## Decision 12: Invalidate Context from Matching Authoritative Events

**Decision**: Add one idempotent `invalidateSchoolContext` Pinia action and one
application-wired request/response observer on the shared authenticated
administration Axios client. Stamp requests with current school-context
generation. Normalize raw errors through one
`normalizeSchoolContextErrorCode(error)` boundary that reads only canonical
`response.data.error.code` or documented legacy `response.data.code` and
returns a lowercase code or `null`. Invalidate only when that normalized code
is `tenant_mismatch` or `inactive_school` and the failed request's
`X-School-Id` plus stamped generation exactly match current context. Preserve
authenticated status with active school cleared. Bootstrap with an invalid
persisted school clears it and retries current-session resolution once without
the header. Successful local deactivation/deletion of the current school calls
the same action. No polling is added.

**Rationale**: The shared client carries administration, teacher, student,
guardian, reporting, assessment, and support requests, so one observer covers
authoritative school-owned failures without duplicating page logic. Matching
normalized explicit error code, request school, and generation prevents ordinary
permission 403s, token failures, platform requests, and late responses for an
old or earlier same-school context from clearing current context. Preserving
authenticated status keeps platform routes and exact System Administrator
recognition usable while school-owned readiness stays false. Installing the
observer from `main.js` with store and router callbacks avoids
service-to-Pinia/router import cycles. Reusing one store action keeps lifecycle,
bootstrap, and response-driven cleanup ordered and testable.

**Alternatives considered**:

- Invalidate on every 401 or 403: rejected because these statuses also mean
  expired/revoked token or ordinary forbidden access and do not prove current
  school invalidity.
- Handle tenant errors independently in every page/composable: rejected because
  coverage would be incomplete and behavior would drift across workspaces.
- Import the Pinia store/router directly into the Axios service: rejected due
  startup ordering, circular dependency, and isolated service-test problems.
- Poll school status: rejected by clarification and because it adds traffic,
  lifecycle race windows, and another source of authority.
- Clear context for any stale failed school header: rejected because a late
  School A response must not clear already-confirmed School B.
