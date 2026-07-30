# Frontend Contract: School Context Selection

This contract defines the Vue behavior that consumes the versioned School and
AuthSession API contracts. It does not authorize a school client-side.

## Actor Gate

Interactive discovery and switching are available only when the current
session contains an active role with:

```text
name = System Administrator
scope = platform
```

The actor must remain authenticated while selection is pending. Non-System
Administrator users retain their existing single-school presentation or safe
blocked state; this feature does not call the platform school list for them.

## Discovery Operation

Use existing `GET /api/v1/schools` through the existing school service.

Required query:

```text
status=1
page=<current page, minimum 1>
per_page=25
```

Optional query:

```text
name=<trimmed school-name text>
inep_code=<normalized exact INEP code>
```

Name and INEP combine with AND. The UI therefore exposes distinct fields and
does not send one ambiguous term as both filters.

The UI maps every response item to:

```js
{
  id,
  name,
  inepCode,
  city,
  state,
  status, // normalized to 'active' or 'inactive'
}
```

Only normalized `active` items may invoke selection. `document`/CNPJ must not
be rendered in the selector, its accessible name, analytics, or feedback.

Pagination derives page count from response `meta.page`, `meta.per_page`, and
`meta.total`. Filter changes return to page 1. A newer search cancels or
supersedes older requests; stale results never replace newer results.

## Confirmation Operation

Use existing `GET /api/v1/auth/me` with:

```text
Authorization: Bearer <current token>
X-School-Id: <chosen School UUID>
```

The OpenAPI `AuthSession.resolved_school` property is required and nullable:

- no header for a platform session may return `resolved_school: null`;
- a valid active header returns the exact resolved School;
- inactive, missing, or invalid context returns existing `403 tenant_mismatch`;
- invalid/rejected token returns the existing 401 token-rejection response.

The frontend commits the choice only if the response:

1. is from the latest confirmation generation;
2. contains non-null `resolved_school`;
3. contains an exact `resolved_school.id` match; and
4. contains a status that normalizes to `active`.

No school-owned route or request starts before all four checks pass.

## Context Switch Ordering

For an approved switch from School A to School B:

1. Complete route-leave/unsaved-work/lifecycle-dialog preflight before changing
   context.
2. On cancel, keep School A and all state unchanged.
3. On approval, increment context generation.
4. Clear the active school and all registered/known School A-owned state.
5. Send the confirmation request for School B.
6. Ignore late search or confirmation results from earlier generations.
7. Commit School B only after exact active confirmation.
8. Reload a compatible route or navigate to the school dashboard.

If confirmation fails after step 4, School A is not silently restored. The
selector remains available, identity is retained unless the token was rejected,
and school-owned content stays blocked.

## Route Compatibility

The centralized resolver applies these rules after confirmation:

| Route type | Result after switch |
|---|---|
| Platform route (`requiresSchoolContext !== true`) | Retain named route; keep platform scope |
| Explicit safe generic school workspace/list (`schoolContextSwitch: 'retain'`) with no tenant-owned params | Retain route name, clear old query by default, reload under new context |
| Detail, edit, create, action, subject-bound, parameterized, unregistered, unreleased, or newly unauthorized route | School dashboard |

For this feature, a released route is a registered named route that the
existing release/feature-gate mechanism reports enabled when confirmation
finishes. Route intent never overrides that fresh result.

Requested route intent is captured before a missing-context redirect or shell
navigation to the selector, revalidated after confirmation, and consumed once.
Raw old `fullPath` values and tenant-owned IDs are never replayed by default.

## Preference Lifetime

The last-confirmed UUID is written only after successful confirmation. During
bootstrap it is a request for fresh `/auth/me` confirmation, never proof.

Clear it when any of these occurs:

- explicit logout;
- session expiry;
- token rejection/revocation;
- lifecycle session cleanup;
- authenticated user identity changes;
- selected school is confirmed inactive, deleted, or unavailable.

## Authoritative Context Invalidation

One application-installed observer watches rejected responses from the shared
authenticated `administrationHttpClient`. It requests context invalidation only
when all conditions hold:

1. response error code, normalized case-insensitively, is
   `tenant_mismatch` or `inactive_school`;
2. failed request contains `X-School-Id`;
3. that header exactly matches current confirmed school ID;
4. request's stamped school-context generation equals current generation; and
5. current context has not already been invalidated.

HTTP status alone is never enough. Generic 401/403 responses, headerless
platform requests, temporary failures, and responses carrying an older school
ID continue through normal local feedback without clearing current context.
The observer rethrows/rejects the original response after triggering cleanup so
existing service error mappers still operate.

`normalizeSchoolContextErrorCode(error)` is the sole raw-shape boundary. It
reads canonical `response.data.error.code` first, then documented legacy
`response.data.code`, and returns a lowercase string or `null`. It ignores all
other raw shapes. The authoritative classifier consumes only this normalized
value and does not treat `no_active_school`, `inactive_record`, or another
semantically similar code as context invalidity.

`invalidateSchoolContext` performs one ordered, idempotent transition:

1. increment confirmation generation so in-flight old-context responses are
   stale;
2. clear active school, student/subject/academic-period and all registered
   school-owned state;
3. clear the persisted preference for that school;
4. preserve authenticated user, token, roles, and permissions when token
   remains valid;
5. preserve authenticated status with safe context feedback, while active
   school remains null and school-owned readiness is false; and
6. navigate to selector only if current route requires school context.

Platform-wide routes remain open and platform-scoped with unresolved school
indicator. Concurrent matching errors after the first clear are no-ops.

When bootstrap sends a persisted school and receives explicit
`tenant_mismatch`/`inactive_school`, it invokes the same cleanup and retries
`/auth/me` once without `X-School-Id`. A second failure does not retry. Token
rejection still follows identity-loss behavior.

## Lifecycle Integration

- The selector never lists inactive schools.
- Activation and restoration continue through existing School administration
  lifecycle actions and do not auto-select a school.
- After successful activation/restore, a deliberate selector refresh or new
  search may show the school.
- Successful deactivation/deletion of the current school clears context and
  school-owned state through the same invalidation action, then routes to
  selection when school context is needed.
- Authoritative invalidation from another browser/process is detected by the
  next matching backend rejection; no poll or background timer is permitted.

## Feedback Contract

| Condition | Required frontend behavior |
|---|---|
| Loading/searching | `aria-busy`, polite announcement, preserve layout |
| Selecting | Disable duplicate submission; announce progress |
| No filter matches | Search-specific empty state and clear-filter action |
| No active schools | Active-school empty state and authorized School administration path |
| Validation (422) | Safe field/query feedback; preserve identity |
| Forbidden / inactive / tenant mismatch (403) | Commit nothing; explain unavailable choice without leaking hidden details; allow refresh/other choice |
| Conflict/lifecycle race | Commit nothing; refresh results; preserve identity |
| Temporary/network/server failure | Retry action; preserve identity |
| Token rejected/expired (401) | Clear identity and preference; follow existing sign-in/session-expired path |
| Exact response mismatch/inactive status | Treat as safe confirmation failure; commit nothing |
| Current-school `tenant_mismatch`/`inactive_school` with matching request header and generation | Preserve valid identity, invalidate context/preference/data, and require selection before school-owned work |
| Generic 401/403, platform request, or stale old-school header | Do not infer context invalidity; use existing local feedback |

All visible and assistive strings use existing i18n modules. Result accessible
names include school name, INEP, city, and state. Keyboard focus is visible,
selection works with native Enter/Space semantics, status changes are announced,
and error focus/retry behavior is deterministic.

## Component and State Boundaries

- `SchoolSelectionPage.vue`: route composition and post-confirmation navigation.
- `SchoolContextSelector.vue`: selection feature container; no direct HTTP.
- `SchoolSelectionSearch.vue`: labeled filter controls and explicit events.
- `SchoolSelectionList.vue`: presentational choices; props down/events up.
- `SchoolContextControl.vue`: current-school presentation and navigate-to-selector
  event; it does not switch context itself. US1 renders it read-only, and US2
  enables the event only after reset ordering and pending-work guards exist.
- `useSchoolSelection.js`: query, pagination, async, cancellation, stale-search,
  and retry state.
- `useSchoolContextSwitch.js`: confirmation orchestration and reset ordering.
- `sessionStore.js`: authenticated identity, confirmed active school,
  confirmation generation, idempotent invalidation, resetters, and preference
  lifetime.
- `schoolContextInvalidation.js`: pure response classification, request
  generation stamping, and observer installation using injected store/router
  callbacks; no direct Pinia/router import.
- `main.js`: installs observer after Pinia/router creation.
- `schoolSelectionService.js`: constrained delegation to existing
  `schoolsService.listSchools()`; no Axios instance and no duplicated endpoint.

## Verification Contract

Automated coverage must include:

- exact actor gate and non-System Administrator blocking;
- active-only discovery, separate name/INEP queries, explicit pagination;
- normalized numeric status and name/INEP/city/state display;
- no CNPJ in selector DOM or accessible text;
- no first-result auto-selection and duplicate-submit prevention;
- distinct loading, filtered no-results, unfiltered no-active-schools,
  validation, unauthorized, forbidden, inactive-school, tenant-mismatch,
  expired-session, conflict, and temporary-unavailable feedback;
- exact active confirmation and backend System Administrator regression;
- restore, logout/expiry/rejection/identity-change clearing;
- old-context clearing, stale list/confirmation rejection, recoverable failures;
- missing-context route capture, safe route retain/reload, unsafe dashboard
  fallback, platform scope preservation, and requested-intent consumption;
- unsaved-work/lifecycle guard cancellation and approval ordering;
- activation refresh and current-school deactivation/deletion invalidation;
- authoritative matching error invalidation, one-shot bootstrap fallback, and
  no invalidation for generic status, platform, stale-header/generation,
  transient, or duplicate responses;
- keyboard, focus, announcements, and layouts at 390px, 768px, and 1440px.

Manual evidence remains required for real screen-reader behavior and the
representative-administrator usability success criteria.
