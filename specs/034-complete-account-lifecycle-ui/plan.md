# Implementation Plan: Complete Administrator Account Lifecycle UI

**Branch**: `034-complete-account-lifecycle-ui` | **Date**: 2026-08-10 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/034-complete-account-lifecycle-ui/spec.md`

## Summary

Complete account lifecycle administration as a contract-first, three-repository
delivery. OpenAPI adds an optional invitation-ready user creation mode and a
user-specific invited status. Laravel preserves current active creation by
default, supports the explicit create-then-invite sequence, provisions
`account_lifecycle.manage` in platform and school scopes, and closes self-action
and tenant-enumeration gaps. The Vue SPA then replaces the hardcoded gate with
scope-aware session authorization, keeps unauthorized sections unmounted,
supports a persisted post-create invitation phase, activates lock/unlock/recover/
reactivate controls, and rejects stale identity/permission/tenant responses.
Token-path admin resend remains excluded.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13; JavaScript with Vue 3.5 Composition API and `<script setup>`  
**Primary Dependencies**: Laravel Form Requests, DTOs, Policies, API Resources, existing account lifecycle services/repository; Vue Router 5.1, Pinia 3.0, Axios 1.18 service modules, Element Plus 2.14, Vue I18n 11.4, Tailwind CSS 4.3  
**Storage**: Existing MySQL users/roles/permissions; one permission index migration from code-only to `(code, scope)` uniqueness; no user-table migration or new frontend persistence  
**Testing**: Redocly contract lint; PHPUnit feature/unit tests including migration/seeder verification against the configured MySQL test database; Vitest 4.1 with Vue Test Utils; Playwright 1.61 mocked browser tests; production frontend build  
**Target Platform**: Laravel JSON API on Linux and modern desktop/mobile browsers consuming `/api/v1`  
**Project Type**: Multi-repository web application: shared specs/OpenAPI, Laravel API, Vue SPA  
**Performance Goals**: Unauthorized/missing-context views issue zero lifecycle requests; each accepted user-create or lifecycle submission is single-flight; context changes prevent stale commits and follow-up requests  
**Constraints**: Contract before backend before frontend; backend Policy and tenant-safe lookup tests pass before UI activation; preserve omitted/active create behavior; no new endpoint/package/store; no secret token use; no admin resend; school requests require exact active context; platform targets use explicit platform-only lookup mode with no artificial school or cross-mode fallback; route pages stay thin  
**Scale/Scope**: One additive create field, one user status schema, one permission index migration, existing lifecycle service security corrections, two user pages, three lifecycle panels/dialog, two feature composables, scoped route behavior, layered tests, and Feature 008/021 documentation updates

## Constitution Check

*GATE: PASS before Phase 0; re-checked after Phase 1 design.*

- PASS: OpenAPI changes lead delivery. No endpoint or API version is added;
  `createUser` gains an additive field, `User` gains a documented status, and
  existing `listUsers`/`getUser` tenant lookup semantics are made explicit.
- PASS: Specification/OpenAPI, backend, and frontend impacts and sequencing are
  separate and linked by Feature 034; Feature 008 and 021 remain synchronized.
- PASS: Laravel retains thin controllers, `CreateUserRequest`,
  `CreateUserData`, `UserService`, `AccountLifecyclePolicy`, existing lifecycle
  services/repository, and `UserResource`. Authorization remains in
  `AccountLifecyclePolicy`; services invoke it before business transition rules
  and protected lookup. No new repository is introduced. Public identifiers
  remain UUIDs.
- PASS: Vue retains JavaScript Composition API/`<script setup>` by repository
  convention, existing Pinia session authority, Vue Router, Tailwind/Element
  Plus, and Axios-only services. Components receive props and emit actions;
  composables own state and effects.
- PASS: MySQL permission identity becomes scope-aware. Tenant context is
  authorized before scoped lookup; soft-delete behavior and user storage shape
  remain unchanged.
- PASS: Compatibility, authentication, authorization, invited-state, success,
  conflict, validation, tenant, not-found, and transport behavior are explicit.
- PASS: Redocly, PHPUnit, Vitest, Playwright, and build verification cover each
  changed critical flow. Mocked browser tests are not represented as live API
  integration or human UAT.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/034-complete-account-lifecycle-ui/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── complete-account-lifecycle-ui-contract.md
└── tasks.md                     # Created by /speckit-tasks
```

### Source Code (target repositories)

```text
schoolmaster-specs/
├── api/
│   ├── components/schemas/users/
│   │   ├── User.yaml
│   │   ├── UserCreateRequest.yaml
│   │   └── UserStatus.yaml
│   └── paths/
│       ├── users/index.yaml
│       ├── users/user.yaml
│       ├── users/account-lock.yaml
│       ├── users/account-reactivation.yaml
│       ├── account-lifecycle/invitations.yaml
│       └── account-lifecycle/invitations-setup.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
├── specs/008-account-lifecycle-workflows/
├── specs/021-account-lifecycle-ui/
└── specs/034-complete-account-lifecycle-ui/

schoolmaster-backend/
├── app/
│   ├── DTOs/Users/CreateUserData.php
│   ├── Http/
│   │   ├── Requests/Api/V1/CreateUserRequest.php
│   │   └── Resources/UserResource.php
│   ├── Models/User.php
│   ├── Policies/AdministrationLifecyclePolicy.php
│   ├── Policies/AccountLifecyclePolicy.php
│   ├── Repositories/AccountLifecycleRepository.php
│   └── Services/
│       ├── Users/UserService.php
│       ├── AccountLifecycle/
│       │   ├── AccountInvitationService.php
│       │   ├── AccountLockService.php
│       │   └── AccountRecoveryService.php
│       └── AdministrationLifecycle/
│           └── AdministrationDetailService.php
├── database/
│   ├── migrations/
│   └── seeders/PermissionSeeder.php
└── tests/
    ├── Feature/AccountLifecycle/
    ├── Feature/Api/V1/
    └── Unit/

schoolmaster-frontend/
├── src/
│   ├── components/admin-system/users/
│   │   ├── UserInvitationPanel.vue
│   │   ├── AccountLockPanel.vue
│   │   └── AccountLifecycleActions.vue
│   ├── components/ui/admin/AdminAccountLifecycleDialog.vue
│   ├── composables/admin-system/
│   │   ├── useAccountInvitation.js
│   │   ├── useAccountLifecycleActions.js
│   │   └── useAdministrationCreatePage.js
│   ├── contracts/admin-system/account-lifecycle.js
│   ├── locales/account-lifecycle.js
│   ├── pages/admin-system/users/
│   │   ├── CreateUserPage.vue
│   │   └── UserDetailPage.vue
│   ├── router/modules/access-administration.routes.js
│   ├── services/admin-system/accountLifecycle.js
│   └── stores/auth/sessionStore.js
├── tests/unit/account-lifecycle/
└── e2e/account-lifecycle.spec.js
```

**Structure Decision**: Extend existing account lifecycle feature boundaries.
Laravel controllers remain unchanged orchestration surfaces; request/DTO/service/
resource boundaries own create-mode behavior, while existing policy/repository/
services own security. Vue pages compose existing panels and new feature
composables. No component calls Axios, no new Pinia store is created, and the
repository remains JavaScript rather than introducing a TypeScript island.

## Component and Composable Map

| Surface | Single responsibility | Inputs / outputs |
|---|---|---|
| `CreateUserPage.vue` | Compose editing and refresh-safe persisted invitation phases | Route/session/context in; create/invite events and navigation out |
| `UserDetailPage.vue` | Compose authorized detail lifecycle surfaces | Route/session/target in; reload callback out |
| `UserInvitationPanel.vue` | Render explicit invitation action and safe result | User/result/pending/error in; `create`/`retry` out |
| `AccountLockPanel.vue` | Render safe lock loading/empty/current state | Lock/loading/error in; no mutation |
| `AccountLifecycleActions.vue` | Render only currently eligible actions | Eligibility/pending in; `action`/`refresh` out |
| `AdminAccountLifecycleDialog.vue` | Capture/validate action reason and confirmation | Action/reason/errors/pending in; submit/cancel out |
| `useAccountInvitation.js` | Own invitation single-flight, retry, abort, and stale-context state | Persisted target/session/context/service in; readonly state/actions out |
| `useAccountLifecycleActions.js` | Own lock load, action orchestration, refresh, and stale guards | Target/session/context/service/reload callback in; readonly state/actions out |

## Implementation Approach

### Phase 0: Contract and Source-of-Truth Alignment

- Add `account_setup_mode: active|invitation` to `UserCreateRequest`, defaulting
  to `active`. Document that invitation mode persists an invited user but creates
  no invitation, token, or delivery request.
- Add `UserStatus` (`active`, `inactive`, `invited`) and use it only for `User`;
  do not expand shared status schemas used by other resources.
- Document `invited` list filtering, invited-to-active setup ownership,
  platform-only versus exact-school `listUsers`/`getUser` lookup, self-target
  denial, tenant-before-target ordering, invitation creation and completion
  semantics, and unchanged resend block in both modular invitation paths.
- Synchronize aggregate OpenAPI, Feature 008 backend rules, Feature 021 UI rules,
  and Feature 034 contract before backend changes.

### Phase 1: Backend Invitation-Ready Creation and Permission Provisioning

- Extend `CreateUserRequest` and `CreateUserData` with account setup mode.
  `UserService` maps active/omitted to current behavior and invitation to
  `status=invited`, preserving school, roles, generated unusable password, and
  normal `201 User` output without invitation side effects.
- Keep create-user school tenancy and `users.manage` authorization unchanged.
  Platform user lifecycle actions remain supported for existing platform users;
  this feature does not invent platform user creation through `createUser`.
- Ensure generic update/activation/reactivation rules cannot transition invited
  users to active; only successful invitation setup may do so.
- Replace permission code-only uniqueness with `(code, scope)`, update seeder
  natural keys, seed platform and school `account_lifecycle.manage`, and make
  ordinary permission checks filter both code and scope. Preserve System
  Administrator master behavior.
- Treat the permission migration as forward-only unless rollback explicitly
  reconciles duplicate scoped codes before restoring code-only uniqueness. The
  implementation task must not promise an unconditional reversible rollback.
- Run the migration and seeder verification against the configured MySQL test
  database, including a second seeder run, duplicate composite-key rejection, and
  guarded rollback behavior; SQLite-only evidence does not satisfy this gate.

### Phase 2: Backend Lifecycle Security and Verification

- Make `AccountLifecyclePolicy` the authorization owner for scoped permission,
  exact System Administrator master access, self-target denial, and resolved
  tenant prerequisites. Services invoke the policy before transition rules,
  protected state reads, or target mutation.
- Resolve/authorize active tenant context before school-owned target lookup and
  scope repository queries so unknown and cross-school identifiers share the
  documented non-disclosing outcome.
- Make existing `listUsers` and `getUser` reads use the same preselected lookup
  mode: exact-school queries require the active school header, while headerless
  platform queries require platform `schools.view` or System Administrator master
  access and constrain users to `school_id IS NULL`; exact-school queries retain
  school `users.view`. Lifecycle panels separately require matching
  `account_lifecycle.manage` or master access. `AdministrationDetailService` and
  `AdministrationLifecyclePolicy` must not perform cross-mode fallback.
- Deny actor-equals-target lock review, lock, unlock, recover, and reactivate
  before state read or mutation. Preserve existing 401/403/404/409/422 envelopes.
- Keep unlock (`DELETE`, no body), recovery (`action=unlock`, optional reason),
  reactivation (`action=reactivate`, optional reason), audit, token revocation,
  and resend behavior unchanged.
- Against the configured MySQL test database, test migration/seeder idempotence,
  both scoped permission rows, duplicate composite-key rejection, guarded
  rollback, System Administrator and ordinary actors, invitation-mode
  compatibility, setup-only activation, user list/detail lookup modes,
  self-denial, tenant ordering, transitions, conflicts, and response shapes.
- Treat this phase and its focused policy/tenant test checkpoint as a blocking
  prerequisite for all frontend action and invitation activation.

### Phase 3: Frontend Scoped Authorization and Route Readiness

- Remove the hardcoded permission-source flag. Derive authority from active raw
  permission objects (`code` plus `scope`) and exact active platform System
  Administrator role. Do not use flattened permission codes as scope authority.
- Derive target scope from the persisted target, require matching active school
  only for school targets, reject target-school mismatch, actor self-target, and
  invalid target states, and never require school context for platform targets.
- Select one route lookup mode before loading user detail: validated route/list
  intent first, otherwise school mode for an active school, otherwise platform
  mode for platform authority, otherwise no request. School mode requires the
  exact active-school header. Platform mode sends no school header, queries
  platform-owned users only, and never retries as a school lookup.
- Make platform list/detail route access scope-aware rather than inheriting a
  universal school requirement. Keep school create flow tenant-owned.
- Do not mount invitation, lock, or lifecycle-action sections for unauthorized,
  missing/inactive/mismatched context. Because composables are not started,
  denied views issue zero lifecycle calls.

### Phase 4: Explicit Create-Then-Invite Journey

- Create school users with `account_setup_mode=invitation` on the account
  onboarding flow. Retain returned mapped user and replace editable form with a
  persisted-user summary and explicit invitation action; do not auto-invite or
  auto-navigate to list.
- Store only the non-secret created user UUID in route intent. On navigation or
  reload, re-fetch that UUID under the current authorized tenant before
  restoring the invitation phase; fail closed when authorization, tenant,
  target, or invited-state validation fails. Never rebuild target data from
  draft fields or route-supplied user details.
- `useAccountInvitation` builds the documented request from persisted user,
  passes exact school header, omits request `delivery_metadata`, renders only
  `status`, `expires_at`, `delivery_channel`, and `delivery_requested_at`,
  prevents duplicate submission, and ignores/aborts stale identity, permission,
  target, or school responses.
- Invitation failure leaves “user created” truth and same-user retry visible.
  Resend stays absent; no token-path request is reachable.

### Phase 5: Lifecycle Action Activation

- `useAccountLifecycleActions` uses context snapshots and abort/request
  generations for lock load and mutations. Identity, permission, target, or
  school change clears protected state and invalidates both current and
  follow-up requests.
- Do not show actions until lock load resolves. Validate requested action again
  at launch and submit; keep lock reason 1–500, unlock body absent, and optional
  recovery/reactivation reason at most 500.
- After success or state conflict, refresh both authoritative target and lock.
  Remove obsolete actions immediately after refreshed state; render normalized,
  localized, secret-safe feedback without assuming undocumented 5xx envelopes.
- Keep panels presentational with props down/events up and page components as
  composition surfaces.

### Phase 6: Verification, Documentation, and Evidence

- Rewrite tests that assert the stale hardcoded block. Add contract, service,
  composable, component, mounted page, and mocked Playwright coverage for the
  complete permission/tenant/state matrix and create/invite/action journeys.
- Browser tests install stateful API routes before navigation, record requests,
  fail unexpected lifecycle/resend calls, and verify 390/768/1440 layouts plus
  the FR-026/SC-011 keyboard, live-feedback, overflow, focus containment/return,
  cancel, and pending-submit behavior across Chromium, Firefox, and WebKit.
- Run contract lint, focused/full PHPUnit, focused/full Vitest, build, focused
  cross-browser Playwright, and full browser regression. Record mocked browser
  evidence accurately; human administrator/screen-reader evidence remains
  pending unless actually performed.
- Update every stale gate statement and command/evidence entry in Feature 021;
  preserve blocked resend and outstanding human UAT.

## Post-Design Constitution Check

- PASS: Research and design resolve all plan-time unknowns and define contract
  changes before backend/frontend consumption.
- PASS: Laravel design uses migration, Form Request, DTO, Service, Policy,
  existing Repository, Resource, and feature tests without controller logic;
  lifecycle authorization remains in `AccountLifecyclePolicy` and completes
  before frontend activation.
- PASS: Vue design uses one-way component flow, focused composables, scope-aware
  session inputs, service isolation, and no new global state/package.
- PASS: Permission and tenant storage/access rules, soft deletes, invited state,
  compatibility order, and UUID boundaries are explicit.
- PASS: Layered tests distinguish backend policy proof, mocked SPA E2E, and
  genuinely manual evidence.
- PASS: No constitution deviation emerged after design.

## Complexity Tracking

No constitution violations or exceptional complexity are required.
