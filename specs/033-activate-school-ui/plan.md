# Implementation Plan: School Context Selection UI

**Branch**: `033-activate-school-ui` | **Date**: 2026-07-29 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/033-activate-school-ui/spec.md`

## Summary

Deliver a System Administrator-only school-context selector through the
dedicated missing-context route and a persistent authenticated-shell control.
The Vue SPA reuses the existing platform school list with active-only,
server-side name/INEP filtering and pagination, confirms the exact choice
through `GET /api/v1/auth/me` with `X-School-Id`, clears prior-tenant state
before confirmation, and restores only safe routes. The Laravel API first
corrects its current-session serialization so the already-resolved tenant is
returned as the documented `resolved_school`; the OpenAPI schema makes that
always-present nullable property explicit. Current context also clears without
polling when bootstrap, a local lifecycle action, or a matching authoritative
tenant error proves the selected school invalid.
US1 renders that shell context read-only; US2 enables deliberate shell
switching only after cleanup, stale-response, and pending-work protections are
installed.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13 backend; JavaScript with Vue 3.5 Composition API and `<script setup>` frontend  
**Primary Dependencies**: Laravel API Resources and existing `TenantContextResolver`; Vue Router 5, Pinia 3, Axios service modules, Element Plus 2.14, Vue I18n 11, Tailwind CSS 4  
**Storage**: Existing MySQL school/user data; existing browser session metadata for one last-confirmed school identifier; no migration or new persistent entity  
**Testing**: PHPUnit feature tests; Vitest unit/component tests with Vue Test Utils; Playwright browser tests; Redocly OpenAPI lint; production frontend build  
**Target Platform**: Laravel API on Linux and modern desktop/mobile browsers consuming `/api/v1`  
**Project Type**: Multi-repository web application: Laravel API, Vue SPA, and shared specification/OpenAPI repository  
**Performance Goals**: With at least 100 active-school fixtures and deterministic 300 ms school-list/current-session responses, at least 95 of 100 selection attempts and 95 of 100 restoration attempts reach confirmed or recoverable state within 2 seconds  
**Constraints**: System Administrator only; active schools only; no CNPJ in selector UI; no first-result auto-selection; exact backend confirmation before school-owned loading; stale responses cannot repopulate prior context; context invalidation is event-driven with no polling; no new package or endpoint  
**Scale/Scope**: One backend confirmation correction, one OpenAPI schema tightening, selector/search/pagination, shell entry point, route-compatibility policy, session persistence cleanup, authoritative context invalidation, lifecycle integration, and focused cross-repository tests

## Constitution Check

- PASS: OpenAPI impact is explicit. `resolved_school` already exists; the
  contract is tightened to require the nullable property before backend and
  frontend delivery. No route, field, envelope, or API version is added.
- PASS: Delivery order is specification/OpenAPI, Laravel conformance fix, then
  Vue integration. Each repository's files and verification are identified.
- PASS: Laravel retains a thin controller, reuses `AuthService`,
  `TenantContextResolver`, `TenantContext`, and `AuthSessionResource`, and adds
  focused feature coverage. No Form Request, Policy, Repository, new DTO, UUID,
  or database change is needed because this is authenticated read
  serialization of an already-validated context.
- PASS: Vue uses Composition API and `<script setup>`, Pinia for confirmed
  session/context state, Vue Router for blocking and safe route restoration,
  Tailwind/Element Plus for UI, and the existing Axios service layer. The
  repository's JavaScript convention is retained instead of creating a mixed
  TypeScript island.
- PASS: MySQL and soft-delete behavior are unchanged. Active, non-deleted
  schools are discovered platform-wide only for exact System Administrator;
  school-owned data remains scoped to one backend-confirmed school.
- PASS: Authentication and authorization remain server-authoritative. Expected
  200, 401, 403, 422, conflict, and transport failures retain existing
  envelopes; selection never converts listing access into tenant authority.
  Context invalidation uses explicit error codes plus the matching school
  header, never generic status codes or client inference.
- PASS: PHPUnit proves current-session tenant serialization; Vitest covers
  mapping, service, store, router, components, persistence, lifecycle,
  authoritative invalidation, and stale-response behavior; Playwright, build,
  and OpenAPI lint cover delivery.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/033-activate-school-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── frontend-school-context-selection-contract.md
└── tasks.md                     # Created by /speckit-tasks
```

### Source Code (target repositories)

```text
schoolmaster-specs/
├── api/components/schemas/auth/AuthSession.yaml
└── specs/033-activate-school-ui/

schoolmaster-backend/
├── app/
│   ├── Http/Controllers/Api/V1/AuthController.php
│   ├── Http/Resources/AuthSessionResource.php
│   └── Services/AuthService.php
└── tests/Feature/CurrentUserApiTest.php

schoolmaster-frontend/
├── src/
│   ├── components/
│   │   ├── auth/
│   │   │   ├── SchoolSelectionSearch.vue
│   │   │   ├── SchoolSelectionList.vue
│   │   │   └── SchoolContextSelector.vue
│   │   └── admin-system/shell/
│   │       ├── AdminShellHeader.vue
│   │       └── SchoolContextControl.vue
│   ├── composables/
│   │   ├── auth/
│   │   │   ├── useSchoolSelection.js
│   │   │   └── useSchoolContextSwitch.js
│   │   └── admin-system/useUnsavedChangesGuard.js
│   ├── contracts/
│   │   ├── auth/authSession.contract.js
│   │   └── admin-system/schools.js
│   ├── layouts/admin-system/AdminSystemLayout.vue
│   ├── main.js
│   ├── pages/
│   │   ├── auth/SchoolSelectionPage.vue
│   │   └── admin-system/schools/
│   │       ├── SchoolsListPage.vue
│   │       └── SchoolDetailPage.vue
│   ├── router/
│   │   ├── authGuards.js
│   │   ├── schoolContextRoutes.js
│   │   └── modules/
│   ├── services/
│   │   ├── api/schoolContextInvalidation.js
│   │   ├── auth/schoolSelectionService.js
│   │   ├── admin-system/administration-service.js
│   │   └── admin-system/schools.js
│   ├── stores/auth/sessionStore.js
│   ├── locales/
│   │   ├── auth.js
│   │   └── admin-system.js
│   └── modules/schools/routes/SchoolEditPage.vue
├── tests/unit/school-context-selection/
└── e2e/school-context-selection.spec.js
```

**Structure Decision**: Correct the existing Laravel auth-session boundary and
extend existing Vue auth, school service, router, shell, lifecycle, and shared
authenticated-response boundaries. The selection service delegates to the
existing school-list service and adds no HTTP client. The application installs
the invalidation observer with explicit Pinia/router dependencies so service
modules do not import UI state or navigation. Route pages remain thin
composition boundaries; async search state belongs to a composable, confirmed
identity and school context remain in Pinia, and presentational components use
explicit props and emits.

## Implementation Approach

### Phase 0: Contract and Conformance

- Require the existing nullable `resolved_school` property in `AuthSession` so
  callers can distinguish an explicitly unresolved platform session from a
  malformed response.
- Preserve the `TenantContext` returned by `TenantContextResolver` in
  `AuthService::currentUser()` and pass its resolved school explicitly through
  the controller to `AuthSessionResource`; login continues to serialize the
  authenticated user's school.
- Add PHPUnit regressions for System Administrator with active header, no
  header, inactive/unknown header, and ordinary school-bound user mismatch.
- Send `per_page=25` explicitly from the selector, avoiding dependence on the
  pre-existing backend default mismatch without expanding this feature into an
  unrelated list-default change.

### Phase 1: Selection Source and State

- Replace the blocked System Administrator selection placeholder with a
  selection service that delegates to `schoolsService.listSchools()` using
  fixed `status=1`, explicit page size, and separate name/INEP controls because
  backend filters combine with AND.
- Normalize numeric/string school statuses and expose camel-case `inepCode`,
  `city`, and `state` at the contract boundary. Never render `document` or
  CNPJ in the selector.
- Keep query, page, results, request generation, loading, empty, and retry state
  local to `useSchoolSelection`; cancel/ignore stale list requests.
- Keep active school, context generation, authenticated identity, and
  last-confirmed preference in `sessionStore`. Persist only after exact active
  confirmation; clear on logout, expiry, token rejection, lifecycle cleanup,
  identity change, or authoritative invalidation.
- Add one idempotent `invalidateSchoolContext` store action that verifies the
  affected school, increments context generation, clears active school and all
  tenant-owned state plus persisted preference, preserves valid
  identity/token/roles/permissions and authenticated status, and records safe
  context feedback. School-owned readiness remains false because active school
  is null; platform-wide access and System Administrator recognition remain
  available.
- In US1, render confirmed current school in the authenticated shell as a
  read-only indicator. Do not emit selector navigation or accept a shell switch
  until the US2 reset pipeline and pending-work guards are installed.
- When bootstrap with a persisted school receives an explicit
  `tenant_mismatch` or `inactive_school`, clear that preference/context and
  retry `/auth/me` once without `X-School-Id`; never loop. Token rejection
  still clears identity, while transient failures retain existing safe error
  behavior.
- Treat selection 401 as identity loss. Keep signed-in identity on recoverable
  403/422/conflict/transport failures while committing no proposed school and
  keeping school-owned content blocked.

### Phase 2: UI, Navigation, and Invalidation Integration

- Compose accessible search, paginated results, feedback, and explicit select
  actions in the dedicated route. Do not select the first result.
- Enable the persistent shell control to navigate to the dedicated selector
  only after `useSchoolContextSwitch`, prior-tenant reset ordering, stale-result
  rejection, and pending-work guards are active. Navigating before switching
  lets existing route-leave, before-unload, and lifecycle-dialog guards confirm
  discard before context is cleared; cancel leaves current context/data intact.
- Wire lifecycle-dialog open state and unsaved form state into the existing
  route-leave guard on selector entry surfaces that do not yet register it; do
  not introduce a second global discard mechanism.
- Capture the requested route on missing-context redirects. Add explicit route
  metadata for safe generic school workspaces/lists. Platform routes retain
  their route and platform scope; safe school routes retain name only and
  reload; detail/edit/action/subject-bound or parameterized routes default to
  the school dashboard.
- Consume and clear requested-route state after successful selection. Recheck
  that the named route remains registered and enabled by the existing
  release/feature-gate mechanism, then recheck permissions, required context, and
  compatibility instead of replaying an old `fullPath`.
- Install one request/response observer on shared
  `administrationHttpClient` from `main.js`, injecting store and router
  callbacks rather than importing them into service modules. Stamp each
  school-owned request with current context generation. Normalize codes through
  `normalizeSchoolContextErrorCode(error)`, which reads only canonical
  `response.data.error.code` or documented legacy `response.data.code` and
  returns a lowercase code or `null`. Trigger invalidation only when that value
  is `tenant_mismatch` or `inactive_school` and request `X-School-Id` plus
  stamped generation exactly match current context; never trigger from HTTP
  status alone, another raw envelope shape, platform requests without the
  header, or late old-context responses.
- Let the original API error continue to its local error mapper after
  invalidation. Route to the selector only when current route requires school
  context; retain platform-wide routes with an unresolved indicator. Repeated
  or concurrent invalidation becomes a no-op after the first context clear.
- Invoke the same store action after successful local deactivation/deletion
  when target ID matches current school. Activation/restore only makes a school
  discoverable after deliberate selector refresh or new search. No polling or
  background status timer is introduced.

### Phase 3: Verification and Delivery Evidence

- Add focused backend, contract, service, store, router, composable, component,
  lifecycle, persistence, stale-response, and error-state tests.
- Cover every FR-021 feedback state explicitly, including distinct filtered
  no-results and unfiltered no-active-schools presentations.
- Prove error normalization for canonical nested, documented legacy top-level,
  unsupported, lowercase, and uppercase code inputs. Prove authoritative
  invalidation for normalized matching code, current header/generation,
  bootstrap fallback, and local lifecycle success;
  prove no invalidation for generic 401/403, headerless platform requests,
  mismatched/stale generation, transient errors, duplicate responses, or
  successful activation/restore.
- Add Playwright scenarios at 390px, 768px, and 1440px, including keyboard
  interaction, duplicate names, pagination, delayed responses, external
  current-school invalidation, selector refresh after activation, and no CNPJ
  in rendered selector.
- Run 100 timed selection attempts and 100 timed restoration attempts with the
  production build, at least 100 active fixtures, and deterministic 300 ms
  school-list/current-session responses; score each flow independently against
  the 95-within-2-seconds threshold.
- Run PHPUnit, focused and full Vitest, frontend build, browser tests, and
  Redocly lint. Record manual screen-reader and representative-administrator
  usability evidence separately. Moderate at least 10 intended users or
  approved role-representative proxies without feature-specific coaching and
  require at least 9 successes for each usability outcome; automated checks do
  not claim those outcomes.

## Post-Design Constitution Check

- PASS: Research, data model, frontend contract, and quickstart keep the API
  change contract-first and identify exact three-repository delivery order.
- PASS: The backend correction reuses existing tenant resolution, service, DTO,
  and API Resource boundaries without database, Policy, Request, or Repository
  expansion.
- PASS: The frontend design isolates transport, preserves one Pinia authority
  for confirmed context, injects app dependencies at composition root, uses
  explicit route/error safety, and clears tenant-owned state before new context
  loads.
- PASS: Test coverage spans backend confirmation, OpenAPI, frontend state/UI,
  route/lifecycle/error integration, accessibility semantics, responsiveness,
  and stale cross-tenant prevention.
- PASS: No constitution deviation emerged after design.

## Complexity Tracking

No constitution violations or exceptional complexity are required.
