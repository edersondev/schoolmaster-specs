# Quickstart: Administration Lifecycle UI

Use this checklist to implement or review roadmap item 5 in
`schoolmaster-frontend`.

## 1. Confirm Scope

- Read `spec.md`, `plan.md`, `research.md`, `data-model.md`, and
  `contracts/administration-lifecycle-ui-contract.md`.
- Reuse implemented features 015, 016, 017, and 018.
- Keep tenant-owned routes blocked when feature 017 cannot confirm an active
  school. Do not use `listSchools` to populate school selection.
- Do not add account lifecycle, invitation, password, account-lock, guardian
  user-link, student management, reporting, platform support, permanent purge,
  or undocumented API behavior.
- Do not add packages unless separately approved.

## 2. Confirm Contract Inventory

Verify approved operation IDs in aggregate OpenAPI and backend behavior:

```text
schools:
getSchool updateSchool activateSchool deactivateSchool deleteSchool restoreSchool

users:
getUser updateUser activateUser deactivateUser deleteUser restoreUser
bulkLifecycleUsers

roles:
getRole updateRole activateRole deactivateRole deleteRole restoreRole
bulkLifecycleRoles

academic years:
getAcademicYear updateAcademicYear activateAcademicYear
deactivateAcademicYear deleteAcademicYear restoreAcademicYear
bulkLifecycleAcademicYears

academic periods:
getAcademicPeriod updateAcademicPeriod activateAcademicPeriod
deactivateAcademicPeriod deleteAcademicPeriod restoreAcademicPeriod
bulkLifecycleAcademicPeriods

guardians:
getGuardian updateGuardian activateGuardian deactivateGuardian
deleteGuardian restoreGuardian bulkLifecycleGuardians
```

Existing list operations remain required for navigation and post-action
refreshes.

## 3. Confirm Permission Metadata

```text
schools detail/update/lifecycle            schools.view /
                                           schools.view + schools.manage
users detail/update/lifecycle/bulk         users.view /
                                           users.view + users.manage
roles detail                               roles.view
roles update                               roles.view + roles.manage +
                                           permissions.view
roles lifecycle/bulk                       roles.view + roles.manage
role permission choices                    permissions.view
academic years detail/update/lifecycle/bulk
                                           academic_years.view /
                                           academic_years.view +
                                           academic_years.manage
academic periods detail/update/lifecycle/bulk
                                           academic_periods.view /
                                           academic_periods.view +
                                           academic_periods.manage
guardians detail/update/lifecycle/bulk     guardians.view /
                                           guardians.view + guardians.manage
```

Backend authorization remains authoritative for every action.

## 4. Build Shared Foundations

- Add explicit service functions for detail, update, lifecycle, restore, and
  bulk lifecycle operations.
- Add lifecycle contract mappers for detail, non-status update payloads,
  lifecycle requests, lifecycle outcomes, and bulk outcomes.
- Add shared detail, status tag, row action, lifecycle confirmation, conflict
  feedback, and bulk action components.
- Add detail, update form, lifecycle action, bulk lifecycle, and action
  eligibility composables.
- Keep route pages thin.
- Keep Axios inside services.
- Extend existing resource route modules and assemble them through
  `administration.routes.js`.

## 5. Build Resource Modules

- Schools: detail, non-status update, activate, deactivate, soft delete,
  restore; no bulk lifecycle.
- Users: detail, non-status update, activate, deactivate, soft delete,
  restore, all-or-nothing bulk lifecycle; no account lifecycle.
- Roles: detail, non-status update, activate, deactivate, soft delete,
  restore, all-or-nothing bulk lifecycle; no scope editing.
- Permissions: stay read-only with no lifecycle UI.
- Academic years: detail, non-status update, activate, deactivate, soft
  delete, restore, all-or-nothing bulk lifecycle.
- Academic periods: detail, non-status update, activate, deactivate, soft
  delete, restore, all-or-nothing bulk lifecycle; no academic-year
  reassignment.
- Guardians: detail, non-status update, activate, deactivate, soft delete,
  restore, all-or-nothing bulk lifecycle; no user-link management.

## 6. Verify Lifecycle Rules

- Edit forms do not submit `status`.
- Status transitions happen only through lifecycle confirmations.
- Every lifecycle confirmation requires reason and today-or-past effective
  date.
- Future effective dates are rejected or mapped to validation feedback.
- Soft-delete wording is recoverable and never implies permanent deletion.
- Restore controls appear only when approved responses expose restorable
  records.
- Bulk lifecycle submits one resource type, one action, 1 to 50 unique record
  IDs, one today-or-past effective date, and one reason.
- Bulk lifecycle failure is all-or-nothing and shows batch-level feedback.
- Selection clears or revalidates on tenant, session, permission, resource,
  filter, or page-size changes.

## 7. Verify Security and Errors

- Hidden controls do not replace backend authorization.
- No protected detail data renders for forbidden, mismatch, inactive, or stale
  school context.
- Not-found messages reveal no cross-tenant existence.
- Session expiration delegates to feature 017.
- Field errors preserve valid form values.
- Conflict messages distinguish stale, dependency, ineligible transition, and
  batch failure when available.
- Logs and diagnostics contain no form payloads, lifecycle reason text,
  emails, tokens, permission payloads, tenant data, or hidden selected IDs.

## 8. Suggested Commands

From `schoolmaster-frontend`:

```bash
npm run test:unit -- --run
npm run build
rg "axios" src/pages src/components src/composables src/router
rg "<el-" src
rg "/api/v1/" src/pages src/components src/composables
rg "status" src/components/admin-system src/pages/admin-system src/contracts/admin-system src/services/admin-system
rg "bulkLifecycle|activate.*|deactivate.*|delete.*|restore.*|update.*|get.*" src/services tests
```

Expected:

- tests and build pass
- no direct Axios outside services/API client
- no kebab-case Element Plus tags
- no endpoint strings in pages/components/composables
- no update-form status submission
- every consumed operation has service and test coverage

## 9. Manual Review

- Test 390px mobile, 768px tablet, and 1440px desktop widths.
- Keyboard through detail pages, edit forms, row actions, status tags,
  lifecycle confirmations, bulk selection, validation summary, and discard
  confirmations.
- Confirm loading/error feedback is announced.
- With the app already bootstrapped and mocked detail/lifecycle service
  latency set to 1.5 seconds, measure from route navigation or action submit
  and confirm stable detail, result, denial, conflict, or recoverable error
  appears within 2 seconds.
- Run moderated lifecycle UAT with five representative administrators. Each
  participant attempts one valid update and one valid single lifecycle action
  across assigned resource scenarios. Pass when at least 90% complete without
  facilitator guidance.

## 10. Implementation Record

Record during implementation:

- OpenAPI operation IDs verified.
- Backend behavior verified for each enabled action.
- Unit test command and result.
- Production build result.
- Source audits for direct Axios, endpoint strings, Element Plus tag casing,
  update-form status omission, and out-of-scope lifecycle behavior.
- Responsive, keyboard, latency, denial, conflict, and privacy review results.
