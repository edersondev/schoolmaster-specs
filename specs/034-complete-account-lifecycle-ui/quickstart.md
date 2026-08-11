# Quickstart: Complete Administrator Account Lifecycle UI

## Preconditions

- Specs, backend, and frontend sibling repositories are available.
- Dependencies are installed; preserve unrelated dirty worktrees.
- Backend test environment has a dedicated configured MySQL database; migration
  evidence from SQLite alone does not satisfy SC-010.
- Test data includes platform/school lifecycle permissions, System
  Administrator, ordinary authorized/unauthorized actors, active/inactive/
  invited/locked/self/cross-tenant users, and active/inactive schools.

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

Implementation checklist:

1. Use active raw scoped permissions plus System Administrator role.
2. Unmount all lifecycle sections and send zero requests when denied.
3. Select exactly one route lookup mode from validated route/list intent,
   otherwise active school, otherwise platform authority; never cross-fallback.
4. Persist invitation-ready user first; freeze draft; expose explicit invitation.
5. Render only invitation `status`, `expires_at`, `delivery_channel`, and
   `delivery_requested_at`; never send request `delivery_metadata`.
6. Keep failure retryable for same persisted user; never expose resend/token.
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
