# Quickstart: Complete Administrator Account Lifecycle UI

## Preconditions

- Specs, backend, and frontend sibling repositories are available.
- Dependencies are installed; preserve unrelated dirty worktrees.
- Backend test environment has a dedicated configured MySQL database; migration
  evidence from SQLite alone does not satisfy SC-010.
- Test data includes platform/school lifecycle permissions, System
  Administrator, ordinary authorized/unauthorized actors, active/inactive/
  invited/locked/self/cross-tenant users, and active/inactive schools.

## Implementation Starting State

Recorded on 2026-08-11 before implementation changes:

- `schoolmaster-specs`: branch `034-complete-account-lifecycle-ui`, revision
  `4d2c94d59ac3d1d199e3a4163201cdcd30a7bc63`, clean worktree.
- `schoolmaster-backend`: branch `034-complete-account-lifecycle-ui`, revision
  `74ae95e39b0a5d7f53db0460f6602ba9f0435a56`, clean worktree.
- `schoolmaster-frontend`: branch `034-complete-account-lifecycle-ui`, revision
  `4674da2e59a2350c5f9485f01548149c3229fa85`, with unrelated existing changes
  in `src/components/admin-system/roles/RoleFilters.vue` and
  `tests/unit/admin-system/administration/components/RoleFilters.spec.js`.
  Feature 034 work must preserve those files unchanged.

## Delivery Order

1. Update Feature 008/021/034 and modular/aggregate OpenAPI.
2. Deliver Laravel create-mode, permission migration/seeding, invited-state,
   self-action, tenant-ordering, and backend tests.
3. Deliver Vue scoped gate, routes, create/invite flow, actions, and tests.
4. Update Feature 021 evidence only with results actually run.

## Contract Verification

From `schoolmaster-specs`:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Confirm no new endpoint, secret response, resend surface, or shared-status
pollution. Confirm `listUsers`/`getUser` document exact-school versus
platform-only lookup and modular and aggregate contracts match.

### Phase 1 Evidence — 2026-08-11

- Feature 008 and Feature 021 source documents now align invitation-ready user
  creation, invited-to-active ownership, exact scoped lifecycle authority,
  master-access limits, self-action denial, tenant-first lookup, hidden client
  denial, and blocked administrator resend with Feature 034.
- Modular OpenAPI adds `account_setup_mode`, user-specific `UserStatus`, exact
  school/platform user lookup modes, invitation eligibility, scoped lifecycle
  authorization, self-action denial, and setup-only invited activation.
- Aggregate contract regenerated from `api/openapi.yaml` with:
  `npx @redocly/cli bundle api/openapi.yaml --output specs/001-schoolmaster-platform/contracts/openapi.yaml`.
- `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1`: PASS. Both API
  descriptions valid; 9 pre-existing `no-unused-components` warnings remain in
  aggregate assessment `$defs`, unrelated to Feature 034.

## Backend Verification

Implementation checklist:

1. Add optional account setup mode to Request/DTO/Service.
2. Preserve omitted/active create; invitation mode returns invited user and no
   invitation/token/delivery record.
3. Prevent generic activation/reactivation of invited users.
4. Migrate permission uniqueness and seed both scopes by `(code, scope)`.
5. Enforce scope, master access, tenant prerequisites, and self-denial through
   `AccountLifecyclePolicy` before services read protected state.
6. Use school-scoped lookup only after exact tenant authorization and
   platform-only lookup only after platform authorization; never cross-fallback.
7. Make `listUsers` and `getUser` use the same exact-school/platform-only rule and
   return the same non-disclosing outcome for unknown or opposite-mode targets;
   school mode uses school `users.view`, platform mode uses platform
   `schools.view` or System Administrator master access, and lifecycle panels
   still require matching `account_lifecycle.manage` or master access.
8. Run the permission migration and seeder twice against MySQL, prove duplicate
   composite rejection, and exercise guarded rollback.
9. Pass focused Policy and tenant-ordering tests before frontend activation.

Run from `schoolmaster-backend`:

```bash
vendor/bin/pint --dirty --format agent
DB_CONNECTION=mysql php artisan test tests/Feature/AccountLifecycle/AccountLifecyclePermissionProvisioningTest.php
php artisan test tests/Feature/AccountLifecycle
php artisan test tests/Feature/Api/V1/AdministrationLifecycle/UserDetailUpdateTest.php
php artisan test --filter=UserManagementTest
php artisan test
```

## Frontend Verification

### Phase 2 Evidence — 2026-08-11

- Database connection: configured MySQL test database (`DB_CONNECTION=mysql`,
  container host `dbmysql_test`, database `dbschoolmaster_test`).
- `docker exec schoolmaster-backend-app-1 php artisan test --compact
  tests/Feature/AccountLifecycle/AccountLifecyclePermissionProvisioningTest.php`:
  PASS, 3 tests and 8 assertions. Both scopes coexist, a second seed is
  idempotent, duplicate composite identity is rejected, and guarded rollback
  refuses code-only uniqueness while scoped duplicates exist.
- `docker exec schoolmaster-backend-app-1 php artisan test --compact
  tests/Feature/AccountLifecycle/AccountLifecycleAuthorizationTest.php
  tests/Feature/Api/V1/UserManagementTest.php
  tests/Feature/Api/V1/AdministrationLifecycle/UserDetailUpdateTest.php`:
  PASS, 18 tests and 52 assertions.
- `docker exec schoolmaster-backend-app-1 php artisan test --compact
  tests/Feature/AccountLifecycle`: PASS, 32 tests and 146 assertions.
- `npm run test:unit -- --run
  tests/unit/account-lifecycle/contracts/adminAccountLifecycle.contract.test.js
  tests/unit/auth/sessionStore.bootstrap.test.js`: PASS, 17 tests.
- The obsolete Feature 021 hardcoded-source assertion was replaced by the full
  scoped denial matrix. The final focused account-lifecycle and school-context
  unit checkpoint passes 42 files and 115 tests.

Implementation checklist:

1. Use active raw scoped permissions plus System Administrator role.
2. Unmount all lifecycle sections and send zero requests when denied.
3. Select exactly one route lookup mode from validated route/list intent,
   otherwise active school, otherwise platform authority; never cross-fallback.
4. Persist invitation-ready user first; freeze draft; expose explicit invitation.
5. Render only invitation `status`, `expires_at`, `delivery_channel`, and
   `delivery_requested_at`; never send request `delivery_metadata`.
6. Keep failure retryable for the same persisted user across navigation/reload
   by retaining UUID-only route intent and requiring an authorized tenant-scoped
   re-fetch; never rebuild from draft data or expose resend/token.
7. Abort/ignore stale loads/actions/invitations and refresh user plus lock.

Run from `schoolmaster-frontend`:

```bash
npm run test:unit -- --run tests/unit/account-lifecycle tests/unit/auth/AuthFeedbackState.test.js
npm run test:unit -- --run
npm run build
env CI=1 npm run test:e2e -- e2e/account-lifecycle.spec.js --project=chromium
env CI=1 npm run test:e2e -- e2e/account-lifecycle.spec.js --project=chromium --project=firefox --project=webkit
env CI=1 npm run test:e2e
```

Playwright uses stateful mocked `/api/v1` routes and must fail unexpected
lifecycle/resend calls. It proves SPA behavior, not live backend policy.

## Required Scenario Matrix

- Scoped permission allow/deny, inactive permission, unrelated permissions.
- System Administrator platform target without school; school target with exact
  active school; missing/mismatched school denied.
- `listUsers`/`getUser` exact-school and platform-only modes; no cross-mode fallback.
- Same-school, cross-school, unknown target, self-target, soft-deleted target.
- Invitation create mode active compatibility, explicit invite success/failure/
  retry, no draft invite, no automatic delivery, no resend.
- Active unlocked → lock; locked → unlock/recover; inactive → reactivate;
  invited → no generic activation.
- Identity, permission, target, or school changes during every request.
- 401/403/404/409/422 and unexpected transport failure with safe feedback.

## Responsive and Accessibility Review

- Treat this matrix as FR-026/SC-011 automated acceptance.
- Exercise create/detail/panels/dialogs at 390, 768, and 1440 px.
- Verify semantic headings/labels/status, no overflow, visible focus, keyboard
  reachability, dialog focus trap/return, Escape/cancel, pending disable, and
  touch-width actions.
- Browser evidence may verify mechanics. Record screen-reader and representative
  administrator review as pending unless humans actually perform it.

## Source Audits

From `schoolmaster-frontend`:

```bash
rg "axios|/api/v1/" src/pages src/components src/composables
rg "resendAccountInvitation|invitationToken" src/components/admin-system src/pages/admin-system
rg "ACCOUNT_LIFECYCLE_PERMISSION_SOURCE_CONFIRMED" src tests
```

Expected: no direct transport outside services, no admin token-path resend, and
no stale hardcoded gate.

## Feature 021 Evidence Update

Update `spec.md`, `research.md`, `data-model.md`, `plan.md`, `tasks.md`,
`contracts/account-lifecycle-ui-contract.md`, and `quickstart.md`. Name exact
permission/scope/master behavior, corrected commands and current results,
cross-repository Feature 034 linkage, and blocked resend. Keep human UAT task
pending unless completed.

## Final Implementation Evidence — 2026-08-12

### Revisions

- Backend branch `034-complete-account-lifecycle-ui`:
  `735a4311d922875f903d8eaee39fa54432845329`
  (`feat(account-lifecycle): complete administrator workflows`).
- Frontend branch `034-complete-account-lifecycle-ui`:
  `8f98b2ae685b5d933bd09fed363046030610fdf3`
  (`feat(account-lifecycle): enable administrator workflows`).
- The pre-existing frontend RoleFilters work was preserved and committed
  separately as `0b62e27bb302aa6e8a922664d33be08fa17f3398`.
- Specs branch `034-complete-account-lifecycle-ui`: final documentation is the
  commit containing this evidence record.

### Live Backend Evidence

- Configured MySQL (`DB_CONNECTION=mysql`, host `dbmysql_test`, database
  `dbschoolmaster_test`) permission migration/seeder checkpoint: 3 tests and 8
  assertions passed, including idempotent reseeding, composite uniqueness, and
  guarded rollback.
- Focused invitation regression after enforcing persisted-target-only creation:
  7 tests and 30 assertions passed.
- Full `php artisan test --compact`: 503 tests and 2,456 assertions passed.
- `vendor/bin/pint --dirty --format agent`: passed.

These tests execute Laravel policy, tenant lookup, persistence, migration, and
response behavior. They are the live backend authorization evidence.

### Frontend Evidence

- Full `npm run test:unit -- --run`: 342 files and 688 tests passed.
- Final focused account-lifecycle and school-context checkpoint: 42 files and
  115 tests passed after dynamic user lookup-route reconciliation.
- `npm run build`: passed. Existing third-party VueUse pure-annotation warnings
  and the existing large-chunk warning remain.
- Focused mocked-SPA Playwright account-lifecycle matrix: 9/9 passed across
  Chromium, Firefox, and WebKit. It covers school/platform lookup, scoped
  visibility and zero-call denial, create-then-invite recovery, stale context,
  390/768/1440 overflow, named controls/live regions, keyboard reachability,
  dialog focus containment/return, Escape/cancel, and pending behavior.
- Repaired dynamic user-detail school-switch case: 1/1 Chromium browser test
  passed.
- Full Playwright regression ran 81 tests: 70 passed, 4 skipped, 6 failed, and
  one unrelated WebKit reporting timing case was flaky and passed on retry. The
  six failures are the pre-existing `e2e/vue.spec.js` root-shell expectations:
  they expect an authenticated Dashboard at `/`, while current product routing
  renders the public `SchoolMaster` landing page. No Feature 034 browser case
  failed.

Playwright uses mocked APIs and proves SPA orchestration only; it is not a
substitute for the live Laravel policy evidence above.

### Contract and Security Audits

- Redocly lint passed for `aggregate@v1` and `schoolmaster-platform@v1`; the same
  9 unrelated assessment `$defs` warnings remain.
- A fresh Redocly bundle compared byte-for-byte equal with
  `specs/001-schoolmaster-platform/contracts/openapi.yaml`.
- Feature 008, Feature 021, Feature 034, modular/aggregate OpenAPI, Laravel
  behavior, and Vue request shapes agree on invitation-ready persistence,
  persisted-target-only invitation, setup-only activation, exact scoped
  authority, self denial, tenant-first lookup, and no cross-mode fallback.
- Source audit found no direct Axios/API transport in administrator pages,
  components, or composables. Administrator resend/token paths are absent.
  Request `delivery_metadata` appears only in negative tests/audit assertions;
  post-submit DOM and browser storage assertions reject tokens, delivery
  metadata, private permissions, tenant-private data, and submitted reasons.

### Completion Decision and Limitations

Feature 034 implementation and its scoped backend/frontend acceptance matrix are
complete. Release evidence still has two external limitations: the unrelated
stale root-shell Playwright expectations described above need their owning
feature reconciled, and the five-administrator moderated UAT plus representative
human accessibility review remain genuinely pending. No human evidence is
claimed by this implementation run.

## PR Review Reconciliation — 2026-08-12

- Expanded Feature 008/034 and OpenAPI before backend changes: school
  invitations still require a persisted invited user, while platform invitation
  atomically provisions a missing platform-owned invited user because no
  platform `createUser` operation exists.
- Shared administration lifecycle transition rules now reject activation of an
  invited user, so `POST /api/v1/users/{id}/activate` cannot bypass password
  setup.
- Platform user `role_ids` updates now accept only active platform roles with no
  school ownership; school roles remain invalid in platform mode.
- Focused red/green review regression: the three new cases initially failed for
  the reported reasons; after the fixes, 13 tests and 42 assertions passed.
- Broader account-lifecycle plus user administration regression: 50 tests and
  199 assertions passed.
- Full configured-MySQL Laravel suite: 506 tests and 2,471 assertions passed.
- Pint passed after ordering one test import. Redocly lint passed for modular and
  aggregate contracts with the same 9 unrelated assessment `$defs` warnings.
  A fresh temporary bundle is byte-for-byte identical to the committed aggregate
  contract.
