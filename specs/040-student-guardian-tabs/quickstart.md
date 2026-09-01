# Quickstart: Student Guardian Tabs

Use this checklist to implement or review feature 040.

## 1. Confirm Scope

- Read `spec.md`, `plan.md`, `research.md`, `data-model.md`, and
  `contracts/student-guardian-tabs-contract.md`.
- Keep Guardians absent from the administration sidebar.
- Keep Create Student as the only user-facing guardian capture entry point.
- Allow student creation with zero, one, or two guardians.
- Support both new guardian creation and existing same-school guardian linking.
- Allow duplicate relationship labels for the two guardian entries.
- Require guardian management authority before a user can capture guardians.
- Do not add packages unless separately approved.

## 2. Backend Contract Gate

Before backend implementation, update and validate OpenAPI:

- `POST /api/v1/student-profiles` supports zero to two
  `guardian_associations`.
- Each guardian entry accepts either an existing active same-school
  `guardian_id` or new guardian fields.
- New guardian fields are limited to approved identity/contact data.
- `relationship_type` is required for every guardian entry.
- Duplicate relationship labels are accepted when guardian identities or
  references are otherwise valid.
- Student-with-guardians create is atomic.
- `GET /api/v1/guardians` remains the only existing-guardian lookup source for
  this feature.
- Standalone `POST /api/v1/guardians` is not used by the frontend as a chained
  step in this workflow.

Suggested contract checks from `schoolmaster-specs`:

```bash
npx @redocly/cli lint api/openapi.yaml
npx @redocly/cli bundle api/openapi.yaml --output /tmp/feature-040-openapi.yaml
rg "maxItems: 2|guardian_associations|StudentProfileGuardianInput|createStudentProfile|listGuardians" api
```

## 3. Backend Implementation Gate

In `schoolmaster-backend`:

- Create matching branch `040-student-guardian-tabs`.
- Update student profile create validation for zero to two guardian entries.
- Validate each entry uses exactly one mode: existing guardian or new guardian.
- Validate existing guardian references are active and same-school.
- Validate guardian management authority when guardian entries are present.
- Create student, any new guardians, and associations in one transaction.
- Roll back the whole operation when any student or guardian validation fails.
- Shape success through approved resources.
- Keep controllers thin; place workflow logic in services.

Suggested backend checks:

```bash
php artisan test
```

Focused tests should cover zero, one, and two guardians, mixed new/existing
entries, duplicate references, third guardian rejection, missing permission,
duplicate relationship label acceptance, tenant mismatch, inactive school,
inactive guardian, invalid contact, invalid relationship, rollback on failure,
and response shape.

## 4. Frontend Implementation Gate

In `schoolmaster-frontend` after backend readiness:

- Create matching branch `040-student-guardian-tabs`.
- Remove Guardians from sidebar navigation metadata.
- Redirect former guardian administration routes to Create Student.
- Convert Create Student to a Student and Guardians tab workflow.
- Keep student fields in the Student tab.
- Let Guardians tab add zero to two entries.
- Support new guardian and existing same-school guardian modes.
- Use `listGuardians` only for existing guardian lookup.
- Submit through the student create service only.
- Keep route-local form state and stale-response protection.
- Show tab-level validation indicators and field errors.
- Block guardian capture for users without guardian management authority while
  leaving student-only creation available.

Suggested frontend checks:

```bash
npm run test:unit
npm run build
rg "axios" src/pages src/components src/composables src/router
rg "/api/v1/" src/pages src/components src/composables src/router
rg "Guardians" src/router src/components src/pages
```

Expected:

- tests and build pass
- no direct Axios outside services/API client
- no endpoint strings in pages/components/composables/router guards
- no visible Guardians sidebar entry
- direct guardian routes redirect without loading guardian list data

## 5. Manual Scenario Review

- Sign in as an administrator with student create authority and guardian
  management authority.
- Confirm Guardians is absent from the sidebar.
- Open Create Student and confirm Student and Guardians tabs.
- Add no guardians, submit valid student data, and verify success.
- Add one new guardian, submit, and verify association display.
- Add two guardians, one new and one existing same-school guardian, submit, and
  verify both associations.
- Add two guardians with the same relationship label and verify submission is
  accepted when both guardian identities are valid.
- Attempt a third guardian and confirm it is blocked before submit.
- Submit invalid student and guardian data and confirm affected tab/field
  feedback while values remain.
- Sign in as a user with student create authority but without guardian
  management authority and confirm student-only creation is available while
  guardian capture is unavailable.
- Open former guardian list/create/detail/lifecycle URLs and confirm redirect
  to Create Student without guardian list data.
- Switch active school during a dirty workflow and confirm existing discard
  behavior and tenant reset.

## 6. Acceptance Evidence

Record in implementation PRs:

- OpenAPI diff showing maximum two guardian entries and new/existing guardian
  entry shape.
- Backend tests proving atomic rollback and permission behavior.
- Backend response-shape verification for created student and guardians.
- Frontend evidence that sidebar excludes Guardians for all permission sets.
- Frontend evidence that direct guardian routes redirect to Create Student.
- Frontend tests for tab state, tab validation, zero/one/two guardian cases,
  blocked third guardian, new/existing guardian modes, and permission gating.
- Diagnostics review proving no protected student, guardian, contact, token,
  permission, full payload, or cross-tenant data appears in visible errors,
  logs, or test output.
- Responsive and keyboard review for 390px, 768px, and 1440px.

## Implementation Evidence

### T001 Branch Readiness

- `schoolmaster-specs`: `040-student-guardian-tabs`
- `schoolmaster-backend`: `040-student-guardian-tabs`
- `schoolmaster-frontend`: `040-student-guardian-tabs`
- Frontend had pre-existing unrelated dirty classroom roster changes before
  branch creation; implementation will not modify or revert those files.

### T010 OpenAPI Contract Gate

- `rtk npx @redocly/cli lint api/openapi.yaml`: passed.
- `rtk npx @redocly/cli bundle api/openapi.yaml --output /tmp/feature-040-openapi.yaml`: passed.

### T022 Backend Gate

- `rtk docker exec schoolmaster-backend-app-1 php artisan test tests/Feature/Api/V1/StudentProfileCreateWithGuardiansTest.php tests/Feature/Api/V1/StudentProfileCreateWithExistingGuardiansTest.php tests/Feature/Api/V1/StudentProfileGuardianValidationTest.php tests/Feature/Api/V1/StudentProfileCreateAtomicityTest.php tests/Feature/Api/V1/StudentProfileGuardianLimitRegressionTest.php tests/Unit/StudentProfiles/StudentProfileGuardianEntryDataTest.php`: passed, 13 tests and 41 assertions.
- `rtk docker exec schoolmaster-backend-app-1 php artisan test`: passed, 583 tests and 2963 assertions.

### T054 Contract Polish

- `rtk npx @redocly/cli lint api/openapi.yaml`: passed.
- `rtk npx @redocly/cli bundle api/openapi.yaml --output /tmp/feature-040-final-openapi.yaml`: passed.

### T056-T057 Frontend Gate

- `rtk npm run test:unit -- tests/unit/student-guardian-tabs`: passed, 12 files and 15 tests.
- `rtk npm run test:unit -- tests/unit/student-guardian-tabs tests/unit/admin-system/administration/routes/administration.navigation.spec.js tests/unit/admin-system/administration/routes/administration.routes.spec.js tests/unit/admin-system/administration/services/student-profiles.spec.js tests/unit/admin-system/administration/services/guardians.spec.js tests/unit/student-enrollment-roster/services/studentProfileCreate.service.spec.js`: passed, 17 files and 25 tests.
- `rtk npm run test:unit`: passed, 366 files and 813 tests.
- `rtk npm run build`: passed. Existing warnings remain for VueUse pure annotations and large chunks; no feature compile errors remain.

### T058 Source Audit

- Direct Axios usage remains inside service modules: `authService.js`, `accountLifecycle.js`, and `admin-system/administration-service.js`.
- Guardian endpoint strings remain only in `src/services/admin-system/guardians.js`.
- `createGuardian` is not called by the Create Student page, components, composable, or student profile service.
- `guardiansList` route remains only as a hidden redirect record; no Guardians sidebar item or `navigation.guardians` sidebar metadata remains.

### T059 Responsive and Keyboard Review

- Source review confirmed the Create Student workflow uses existing `AdminFormPage`, Element Plus tabs, form items, buttons, and native form submit/cancel handling.
- The tab layout has no fixed-width text containers; Student fields retain the existing responsive two-column grid and Guardians entries use one-column mobile/two-column desktop grids.
- Guardian add/remove/mode/select controls are keyboard-reachable Element Plus controls. Live browser review at 390px, 768px, and 1440px remains recommended before release because no authenticated administrator browser session was available in this run.

### T060 Privacy Review

- Backend validation errors use field paths and generic same-school guardian messages; cross-school, missing, inactive, and deleted guardian references are not distinguished by protected record detail.
- Frontend validation messages do not include guardian contact values, full request payloads, tokens, role payloads, or cross-tenant existence details.
- Route redirects preserve only the former route name in `redirected_from` and active school context is still owned by the existing auth/session flow.

### T061-T063 Final Evidence

- `createStudentProfile` now handles zero, one, or two guardian entries; `createGuardian` is not chained by the frontend workflow.
- Guardian entries require `guardians.manage`; student-only creation still requires only student profile management.
- Sidebar navigation excludes Guardians for all tested permission combinations.
- Former guardian list/create/detail/edit routes redirect to Create Student without loading guardian pages.
- Backend tests cover zero/two new guardians, mixed new/existing guardians, duplicate relationship-label acceptance, duplicate existing guardian rejection, third guardian rejection, and atomic rollback.
- Frontend tests cover route hiding/redirects, payload mapping, active lookup filtering, tab rendering, entry limits, duplicate identity/reference validation, and no partial-success contract.
- Administrator-proxy timed evidence: focused feature Vitest run completed in 3.38s; full frontend unit suite completed in 135.17s; production build completed in 1.46s. Existing-guardian lookup is covered as a single active-filtered service call; duplicate submit prevention remains inherited from `useAdminCreateForm` pending-request behavior.
