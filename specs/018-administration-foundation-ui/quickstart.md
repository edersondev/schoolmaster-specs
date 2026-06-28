# Quickstart: Administration Foundation UI

Use this checklist to implement or review roadmap item 4 in
`schoolmaster-frontend`.

## 1. Confirm Scope

- Read `spec.md`, `plan.md`, `research.md`, `data-model.md`, and
  `contracts/administration-foundation-ui-contract.md`.
- Reuse implemented features 015, 016, and 017.
- Keep tenant-owned routes blocked when feature 017 cannot confirm an active
  school. Do not use `listSchools` to populate school selection.
- Do not add detail, edit, lifecycle, bulk, account, guardian-link, or student
  management actions.
- Do not add packages unless separately approved.

## 2. Confirm Contract Inventory

Verify all 14 approved operation IDs in aggregate OpenAPI and backend behavior:

```text
listSchools             createSchool
listUsers               createUser
listRoles               createRole
listPermissions
listAcademicYears       createAcademicYear
listAcademicPeriods     createAcademicPeriod
listGuardians           createGuardian
listStudentProfiles
```

Block only affected UI when contract or backend implementation differs.

Confirm frontend permission metadata exactly matches:

```text
schools list/create              schools.view / schools.view + schools.manage
users list/create                users.view /
                                 users.view + users.manage + roles.view
roles list/create                roles.view /
                                 roles.view + roles.manage + permissions.view
permissions list                 permissions.view
academic years list/create       academic_years.view /
                                 academic_years.view + academic_years.manage
academic periods list/create     academic_periods.view /
                                 academic_periods.view +
                                 academic_periods.manage + academic_years.view
guardians list/create            guardians.view /
                                 guardians.view + guardians.manage
guardian student lookup          student_profiles.view
```

Role create UI must not expose scope. Its service payload must submit
`scope=school`.

## 3. Build Shared Foundations

- Add route-query parser/serializer with per-resource allowlists.
- Add normalized administration error mapper.
- Add shared list page, filter bar, remote data table, pagination, form page,
  and feedback components.
- Add list/create composables and unsaved-change guard.
- Add shared paginated lookup orchestration with selected-option retention and
  tenant reset for roles, permissions, and academic years.
- Keep route pages thin.
- Keep Axios inside services.
- Split story routes into `schools.routes.js`,
  `access-administration.routes.js`, `academics.routes.js`, and
  `guardians.routes.js`; assemble them in `administration.routes.js`.

## 4. Build Resource Modules

- Schools: list, status, pagination, create; no sort control until backend
  honors the published sort parameter.
- Users: list, status, sort, pagination, create with paginated role choices.
- Roles: list, status, pagination, create with paginated permission choices.
- Permissions: read-only paginated list.
- Academic years: list, status, pagination, create.
- Academic periods: list, status, year filter, pagination, create with
  paginated year choice.
- Guardians: list, status, pagination, create with remote active-student
  lookup.

## 5. Verify Query and Navigation

- Refresh preserves approved list state.
- Browser back/forward restores list state.
- Invalid query values normalize without backend forwarding.
- Filter/page-size changes reset page.
- Successful create returns to previous validated list query.
- Dirty create routes confirm every exit.
- School switch confirms dirty tenant form, then clears old tenant state.
- Role, permission, and academic-year selectors can reach every API page and
  retain selected options while paging.

## 6. Verify Security and Errors

- Hidden controls do not replace backend authorization.
- No protected rows render for forbidden, mismatch, inactive, or stale school.
- Not-found messages reveal no cross-tenant existence.
- Session expiration delegates to feature 017.
- Missing unresolved school context renders the feature 017 blocked state and
  sends no tenant-owned administration request.
- Field errors preserve valid form values.
- Logs contain no form payloads, emails, tokens, or permission data.

## 7. Suggested Commands

From `schoolmaster-frontend`:

```bash
npm run test:unit -- --run
npm run build
rg "axios" src/pages src/components src/composables src/router
rg "<el-" src
rg "/api/v1/" src/pages src/components src/composables
rg "listSchools|createSchool|listUsers|createUser|listRoles|createRole|listPermissions|listAcademicYears|createAcademicYear|listAcademicPeriods|createAcademicPeriod|listGuardians|createGuardian|listStudentProfiles" src/services tests
```

Expected:

- tests and build pass
- no direct Axios outside services/API client
- no kebab-case Element Plus tags
- no endpoint strings in pages/components/composables
- every consumed operation has service and test coverage

## 8. Manual Review

- Test 390px mobile, 768px tablet, and 1440px desktop widths.
- Keyboard through filters, tables, pagination, forms, validation summary, and
  discard confirmations.
- Confirm loading/error feedback is announced.
- With the app already bootstrapped and mocked list service latency set to 1.5
  seconds, measure from route navigation or committed query change and confirm
  stable data, empty state, or recoverable error appears within 2 seconds.
- Confirm all resource routes are reachable within two navigation interactions.
- Run moderated create-flow UAT with five representative
  administrators. Each participant attempts school, user, role, academic-year,
  academic-period, and guardian creation once with valid scenario data. Record
  first-attempt completion and facilitator intervention; pass when at least 27
  of 30 attempts complete without guidance.

## 9. Implementation Record

Automated implementation verification completed on 2026-06-25:

- All 14 approved operation IDs were confirmed in aggregate OpenAPI path
  definitions.
- All required frontend permission codes were confirmed in
  `PermissionSeeder.php`.
- Feature 017 unresolved school selection remains intentionally blocked;
  tenant-owned administration and lookups send no requests until an active
  school is confirmed.
- Backend routes for all approved list/create operations were confirmed.
- Administration-focused Vitest suite passed.
- Full frontend unit suite passed with 112 tests across 61 files.
- ESLint and Oxlint passed.
- Production build passed. Existing third-party pure-annotation and bundle-size
  warnings remain non-blocking.
- Source audits found no direct Axios or endpoint use outside services, no
  kebab-case Element Plus tags, no sensitive administration logging, and no
  lifecycle actions outside feature scope.
- Playwright browser verification passed at 390px, 768px, and 1440px with no
  horizontal overflow. School list, validation, create, query-state reset,
  discard confirmation, permission denial without a protected request, and
  permission-aware post-login routing were verified against the local backend.
- Extended keyboard review confirmed keyboard reachability for create, status
  filtering, pagination, validation recovery, and discard confirmation. The
  validation summary now receives focus after a failed submission, and the
  create destination is a single interactive element.
- An already-bootstrapped 1.5-second mocked list request settled in 1.589
  seconds, within the 2-second target.
- Automated surrogate UAT completed 30 of 30 mocked create attempts across
  five isolated browser contexts and all six create workflows without scripted
  intervention. This is repeatable engineering evidence, not moderated human
  usability evidence.

Keyboard/announcement review is complete. Moderated UAT with five
representative human administrators remains pending and cannot be replaced by
the automated surrogate.
