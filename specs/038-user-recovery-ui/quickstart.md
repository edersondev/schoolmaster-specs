# Quickstart: User Recovery UI

## Prerequisites

- Feature 037 backend recovery behavior and OpenAPI contract are available.
- The specs repository is on branch `038-user-recovery-ui`.
- The frontend implementation uses its matching feature branch.
- Existing repository dependencies are installed. Do not add packages for this
  feature.

## 1. Verify the Published Contract

From `schoolmaster-specs`:

```bash
rtk npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Expected: no new contract errors. Existing unrelated warnings must be recorded,
not represented as Feature 038 regressions. Do not regenerate or modify OpenAPI
because this feature changes no wire behavior.

Optional backend prerequisite evidence, when the backend container is running:

```bash
rtk docker exec schoolmaster-backend-app-1 php artisan test --compact \
  tests/Unit/ApiResponseTest.php \
  tests/Feature/Api/V1/UserDuplicateEmailRecoveryTest.php \
  tests/Feature/Api/V1/AdministrationLifecycle/UserLifecycleTransitionTest.php \
  tests/Feature/Api/V1/AdministrationLifecycle/UserDetailUpdateTest.php
```

This verifies the existing dependency; it is not evidence of frontend behavior.

## 2. Implement in Dependency Order

From `schoolmaster-frontend`:

1. Add the internal recovery action and exact error-mapper projection.
2. Add stale-result invalidation to create and lifecycle composables.
3. Add the route-local recovery composable with the exact disposition matrix:
   preserve local/422/network/408/429/5xx; clear 401/403/404/409 and all other
   non-allowlisted HTTP statuses.
4. Add localized recovery copy and a semantic `UserRecoveryAlert.vue` polite
   status region. Do not use `ElAlert`, automatic focus, or custom tab order.
5. Compose the alert and existing lifecycle dialog in `CreateUserPage.vue`.
6. Reset the draft and navigate to `userDetail?user_mode=school` only after a
   current-context restore succeeds.
7. Add unit, component, mounted-page, and Playwright coverage.

Do not add a store, endpoint, direct component HTTP call, raw error rendering,
pre-restore detail lookup, or automatic status/role/profile change.

## 3. Run Focused Frontend Tests

```bash
rtk npm run test:unit -- --run \
  tests/unit/admin-system/administration/services/administration-error-mapper.spec.js \
  tests/unit/admin-system/administration/services/users.spec.js \
  tests/unit/admin-system/administration/composables/useAdminCreateForm.spec.js \
  tests/unit/admin-system/administration-lifecycle/composables/useAdminLifecycleAction.spec.js \
  tests/unit/admin-system/user-recovery/composables/useUserCreationRecovery.spec.js \
  tests/unit/admin-system/user-recovery/components/UserRecoveryAlert.spec.js \
  tests/unit/admin-system/user-recovery/pages/CreateUserRecovery.spec.js \
  tests/unit/account-lifecycle/pages/CreateUserAccountInvitation.test.js
```

If a listed new test path is adjusted to match an existing feature convention,
update this quickstart and the task list together.

## 4. Run the Workflow Test

```bash
rtk env CI=1 npm run test:e2e -- e2e/user-recovery.spec.js --project=chromium
```

The browser workflow must prove:

- exact conflict shows safe localized guidance and one action;
- warning-to-completed-confirmation requires no more than two deliberate actions,
  excluding reason and effective-date entry;
- generic/malformed conflicts never show recovery;
- reason/date validation and explicit confirmation;
- local/422/network/408/429/5xx failures preserve the dialog and draft for a
  deliberate retry, while 401/403/404/409 clear recovery;
- only one restore submission;
- successful restore discards the draft and opens authoritative school-mode
  detail;
- if the detail GET then fails, the app stays on the detail route with existing
  safe retry/return feedback; retry calls only `getUser` and the restore request
  count remains exactly one;
- email, school, session, permission, or route changes invalidate recovery;
- stale in-flight results do not show feedback or navigate;
- no recovery UUID or hidden user detail appears before success;
- the warning uses one polite atomic status region with no nested assertive
  alert, does not move the focused element, and exposes restore in normal
  keyboard order;
- keyboard operation and layouts at approximately 390, 768, and 1440 pixels.

## 5. Run Full Frontend Gates

```bash
rtk npm run test:unit -- --run
rtk npm run build
```

Run the existing non-mutating lint/check command if the frontend repository
defines one. Do not use an auto-fixing lint command as a verification-only gate.

## 6. Run Privacy and Architecture Audits

From `schoolmaster-frontend`:

```bash
rtk rg -n "axios|fetch\(" src/pages/admin-system/users/CreateUserPage.vue src/components/admin-system/users/UserRecoveryAlert.vue
rtk rg -n "localStorage|sessionStorage|useStorage|recoveryUserId|user_id" src/components/admin-system/users src/pages/admin-system/users/CreateUserPage.vue
rtk rg -n "error\.message|response\.data\.error\.message" src/components/admin-system/users src/pages/admin-system/users/CreateUserPage.vue src/composables/admin-system/useUserCreationRecovery.js
rtk rg -n "ElAlert|role=\"alert\"|autofocus|tabindex" src/components/admin-system/users/UserRecoveryAlert.vue
```

Expected:

- no direct HTTP access in page or alert;
- no browser persistence;
- no recovery identifier passed to or rendered by the alert/page template;
- no backend message used as recovery copy;
- no `ElAlert`, nested `role="alert"`, autofocus, or custom/positive tab order in
  `UserRecoveryAlert.vue`.

Review route and storage output in the Playwright trace before restore success.
The recovery UUID must not appear in query parameters, browser storage, or DOM.

## 7. Record Evidence

Fill this table during implementation; do not pre-mark results:

| Gate | Result | Evidence/notes |
|---|---|---|
| Redocly contract lint | Pending | |
| Focused Vitest suite | Pending | |
| Full Vitest suite | Pending | |
| Playwright recovery workflow | Pending | |
| Production build | Pending | |
| Privacy/source audit | Pending | |
| Responsive, live-region, focus, and keyboard review | Pending | |
| Moderated administrator acceptance (SC-007) | Pending | Record predefined cohort size (minimum 10), binary passes, and pass percentage (minimum 90%) |

## 8. Run Moderated Acceptance

Preselect at least 10 authorized school administrators before testing. Present
the recoverable warning and restore journey without explaining the intended
action. Count a participant as passing only when, without prompting, they both:

1. identify that the existing identity must be restored rather than a new
   identity created; and
2. select `Restore existing user` as the correct next action.

Record the cohort size, pass count, and pass percentage. The criterion passes
only at 90% or greater.

Acceptance notes must remain privacy-safe: do not record submitted emails,
recovery identifiers, lifecycle reasons, tenant data, or retained-user details.
