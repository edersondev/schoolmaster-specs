# Tasks: Administration Lifecycle UI

**Input**: Design documents from `specs/020-administration-lifecycle-ui/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/administration-lifecycle-ui-contract.md`, `quickstart.md`

**Tests**: Required. This feature changes frontend routes, services,
contracts, composables, shared components, authorization visibility, lifecycle
confirmations, bulk behavior, and error handling.

**Organization**: Tasks are grouped by user story so each story can be
implemented and tested independently. Paths prefixed with
`schoolmaster-frontend/` are implemented from the frontend repository. Paths
under `specs/020-administration-lifecycle-ui/` are maintained in
`schoolmaster-specs`.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because the task changes different files and
  does not depend on another incomplete task.
- **[Story]**: Maps the task to a user story from `spec.md`.
- Setup, Foundational, and Polish phases have no story label.
- No backend or OpenAPI changes are planned unless contract verification finds
  a blocking mismatch before frontend exposure.

## Phase 1: Setup

**Purpose**: Confirm implementation scope, approved operations, and local
frontend scaffolding before shared lifecycle work starts.

- [ ] T001 Verify every approved lifecycle operation ID, request field, response envelope, permission requirement, and no-school-bulk constraint against `api/openapi.yaml` and record any blocking mismatch in `specs/020-administration-lifecycle-ui/quickstart.md`
- [ ] T002 [P] Create lifecycle test fixture scaffold for sessions, tenants, permissions, records, lifecycle outcomes, bulk outcomes, and error envelopes in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/fixtures/lifecycle.fixtures.js`
- [ ] T003 [P] Create centralized lifecycle locale scaffold for status tags, detail pages, edit forms, confirmations, bulk actions, conflicts, empty states, and discard dialogs in `schoolmaster-frontend/src/locales/administration-lifecycle.js`
- [ ] T004 Register lifecycle locale messages through the existing i18n assembly used by `schoolmaster-frontend/src/main.js` and `schoolmaster-frontend/src/locales/administration-lifecycle.js`
- [ ] T005 [P] Create administration lifecycle contract test directory marker in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/contracts/.gitkeep`
- [ ] T006 [P] Create administration lifecycle service test directory marker in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/.gitkeep`
- [ ] T007 [P] Create administration lifecycle composable test directory marker in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/.gitkeep`
- [ ] T008 [P] Create administration lifecycle page-flow test directory marker in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/.gitkeep`

---

## Phase 2: Foundational

**Purpose**: Shared lifecycle contracts, mappers, service helpers,
authorization eligibility, components, and composables required by every user
story.

**CRITICAL**: No user story implementation begins before this phase completes.

### Foundational Tests

- [ ] T009 [P] Add Vitest coverage for shared lifecycle operation registry, resource capabilities, non-status update field allowlists, bulk-enabled resources, and blocked permissions/schools bulk behavior in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/contracts/lifecycle.contract.spec.js`
- [ ] T010 [P] Add Vitest coverage for validation, forbidden, tenant-mismatch, inactive-context, not-found, conflict, temporary-unavailable, unknown, safe diagnostics, and reason-text privacy mapping in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/administration-lifecycle-error-mapper.spec.js`
- [ ] T011 [P] Add Vitest coverage for detail loading, latest-request protection, route/tenant/session/permission generation keys, safe denial states, retry, and return-query preservation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/useAdminDetail.spec.js`
- [ ] T012 [P] Add Vitest coverage for update dirty state, status-field omission, field-error mapping, pending duplicate-submit prevention, success reset, conflict handling, and entered-value preservation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/useAdminUpdateForm.spec.js`
- [ ] T013 [P] Add Vitest coverage for lifecycle dialog state, required reason, today-or-past effective date, future-date rejection, duplicate-submit prevention, temporary-failure retry with reason preserved until deliberate cancel/close, stale-response ignore, and reason cleanup on close in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/useAdminLifecycleAction.spec.js`
- [ ] T014 [P] Add Vitest coverage for bulk selection limit 1 to 50, unique IDs, tenant/resource/filter/page-size/session/permission resets, all-or-nothing failures, and selected-summary retention in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/useAdminBulkLifecycle.spec.js`
- [ ] T015 [P] Add Vitest coverage for action eligibility across statuses, permissions, tenant context, restore visibility, school no-bulk behavior, and permission read-only behavior in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/useAdminActionEligibility.spec.js`
- [ ] T016 [P] Add Vitest coverage for browser and in-app discard confirmation from dirty edit forms, open lifecycle dialogs, list return, sidebar navigation, and active-school switch in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/composables/useUnsavedChangesGuard.spec.js`
- [ ] T017 [P] Add component tests for detail frame loading, base empty, filtered empty, denial, not-found, conflict, action slots, status tag, validation summary, and list-return behavior in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/AdminDetailPage.spec.js`
- [ ] T018 [P] Add component tests for status tag variants, accessible names, compact table rendering, and responsive behavior at 390px, 768px, and 1440px in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/AdminStatusTag.spec.js`
- [ ] T019 [P] Add component tests for row action visibility, disabled states, keyboard operation, focus management, and hidden unauthorized actions in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/AdminRowActions.spec.js`
- [ ] T020 [P] Add component tests for lifecycle confirmation content, soft-delete wording, future-date validation, required reason, pending state, safe validation summary, and keyboard operation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/AdminLifecycleDialog.spec.js`
- [ ] T021 [P] Add component tests for bulk action bar selected count, action filtering, all-or-nothing confirmation copy, clear selection, max-selection warning, and no school bulk controls in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/AdminBulkActionBar.spec.js`
- [ ] T022 [P] Add component tests for stale, dependency, ineligible transition, batch conflict, forbidden, tenant, not-found, and temporary failure feedback without sensitive data in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/AdminConflictFeedback.spec.js`
- [ ] T023 [P] Add route metadata tests for detail/edit routes, auth requirement, active-school gating, exact permission matrices, hidden navigation, direct-route denial, and unresolved-school request blocking in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/routes/administration-lifecycle.routes.spec.js`

### Foundational Implementation

- [ ] T024 Define shared lifecycle operation registry, resource capability metadata, update field allowlists, action names, status categories, bulk limits, and safe diagnostics contracts in `schoolmaster-frontend/src/contracts/admin-system/lifecycle.js`
- [ ] T025 Implement normalized lifecycle error mapping with no raw Axios exposure and no form value, reason text, email, token, tenant, or permission payload leakage in `schoolmaster-frontend/src/services/admin-system/administration-error-mapper.js`
- [ ] T026 Implement lifecycle detail orchestration with latest-request protection, route/tenant/session/permission generation keys, safe denial states, retry, and return-query validation in `schoolmaster-frontend/src/composables/admin-system/useAdminDetail.js`
- [ ] T027 Implement update form orchestration with non-status payload projection, dirty tracking, field-error mapping, duplicate-submit prevention, conflict handling, success reset, and list/detail reconciliation hooks in `schoolmaster-frontend/src/composables/admin-system/useAdminUpdateForm.js`
- [ ] T028 Implement single-record lifecycle orchestration with confirmation state, today-or-past effective-date validation, required reason, duplicate-submit prevention, temporary-failure retry that preserves reason until deliberate cancel/close, stale-response ignore, result reconciliation, and reason cleanup in `schoolmaster-frontend/src/composables/admin-system/useAdminLifecycleAction.js`
- [ ] T029 Implement bulk lifecycle orchestration with unique selected IDs, selected summaries, limit 1 to 50, all-or-nothing failure handling, tenant/resource/filter reset rules, and selection reconciliation in `schoolmaster-frontend/src/composables/admin-system/useAdminBulkLifecycle.js`
- [ ] T030 Implement permission, status, tenant, resource capability, restore visibility, and bulk eligibility derivation in `schoolmaster-frontend/src/composables/admin-system/useAdminActionEligibility.js`
- [ ] T031 Extend reusable unsaved-change protection for dirty update forms, open lifecycle confirmations, list return, sidebar navigation, browser exit, and active-school switch in `schoolmaster-frontend/src/composables/admin-system/useUnsavedChangesGuard.js`
- [ ] T032 [P] Implement shared detail page frame with header, status, sections, action slots, validation summary, base empty, filtered empty, denial/error states, and list-return controls in `schoolmaster-frontend/src/components/ui/admin/AdminDetailPage.vue`
- [ ] T033 [P] Implement shared lifecycle status tag variants with accessible labels and compact responsive table rendering in `schoolmaster-frontend/src/components/ui/admin/AdminStatusTag.vue`
- [ ] T034 [P] Implement shared row/detail action menu with explicit props/emits, permission-aware visibility, keyboard support, and no transport logic in `schoolmaster-frontend/src/components/ui/admin/AdminRowActions.vue`
- [ ] T035 [P] Implement shared lifecycle confirmation dialog with resource identity, current status, expected outcome, required effective date, required reason, pending state, validation summary, and soft-delete wording in `schoolmaster-frontend/src/components/ui/admin/AdminLifecycleDialog.vue`
- [ ] T036 [P] Implement shared bulk lifecycle action bar with selected count, action filtering, all-or-nothing messaging, clear selection, and max-selection feedback in `schoolmaster-frontend/src/components/ui/admin/AdminBulkActionBar.vue`
- [ ] T037 [P] Implement shared conflict and denial feedback for stale, dependency, ineligible, batch, forbidden, tenant, not-found, unavailable, and unknown outcomes in `schoolmaster-frontend/src/components/ui/admin/AdminConflictFeedback.vue`
- [ ] T038 Add shared lifecycle route metadata helpers and return-query helpers used by `schoolmaster-frontend/src/router/modules/schools.routes.js`, `schoolmaster-frontend/src/router/modules/access-administration.routes.js`, `schoolmaster-frontend/src/router/modules/academics.routes.js`, and `schoolmaster-frontend/src/router/modules/guardians.routes.js`

**Checkpoint**: Shared lifecycle contracts, services, composables, components,
routes, permission visibility, stale-response handling, privacy mapping, and
unsaved guards are testable before resource pages begin.

---

## Phase 3: User Story 1 - Review and Edit Administrative Records (Priority: P1) MVP

**Goal**: Authorized administrators can open protected detail routes for each
approved administrative resource, edit only approved non-status fields, handle
validation/conflict states, and return to the original list query.

**Independent Test**: For each approved resource, sign in with the required
view/manage permissions, open a detail route from the list, submit valid and
invalid edits, verify validation and conflict feedback, and return to the same
list query.

### Tests for User Story 1

- [ ] T039 [P] [US1] Add school detail/update contract tests for `getSchool`, `updateSchool`, address/contact payload mapping, status omission, and no bulk school capability in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/contracts/schools-lifecycle.contract.spec.js`
- [ ] T040 [P] [US1] Add user detail/update contract tests for `getUser`, `updateUser`, full-name/email/role IDs mapping, status omission, and no direct permission assignment in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/contracts/users-lifecycle.contract.spec.js`
- [ ] T041 [P] [US1] Add role detail/update contract tests for `getRole`, `updateRole`, permission IDs mapping, status omission, fixed scope display, and `permissions.view` requirement in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/contracts/roles-lifecycle.contract.spec.js`
- [ ] T042 [P] [US1] Add academic year and academic period detail/update contract tests for `getAcademicYear`, `updateAcademicYear`, `getAcademicPeriod`, `updateAcademicPeriod`, date/sequence mapping, status omission, and no period-year reassignment in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/contracts/academics-lifecycle.contract.spec.js`
- [ ] T043 [P] [US1] Add guardian detail/update contract tests for `getGuardian`, `updateGuardian`, contact/relationship mapping, status omission, and no guardian user-link fields in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/contracts/guardians-lifecycle.contract.spec.js`
- [ ] T044 [P] [US1] Add school service tests for detail/update parameters, approved payloads, validation, forbidden, not-found, conflict, temporary failure, stale cancellation, and no raw endpoint use outside service in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/schools-lifecycle.spec.js`
- [ ] T045 [P] [US1] Add user service tests for detail/update tenant headers, role IDs, validation, forbidden, tenant mismatch, not-found, conflict, temporary failure, and stale cancellation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/users-lifecycle.spec.js`
- [ ] T046 [P] [US1] Add role service tests for detail/update tenant headers, permission IDs, role-scope immutability, validation, forbidden, conflict, and stale cancellation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/roles-lifecycle.spec.js`
- [ ] T047 [P] [US1] Add academic service tests for detail/update tenant headers, date and sequence payloads, validation, forbidden, tenant mismatch, conflict, and stale cancellation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/academics-lifecycle.spec.js`
- [ ] T048 [P] [US1] Add guardian service tests for detail/update tenant headers, contact and relationship payloads, validation, forbidden, tenant mismatch, conflict, and stale cancellation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/guardians-lifecycle.spec.js`
- [ ] T049 [P] [US1] Add resource detail component tests for display-only status, permitted fields, validation summaries, no protected data in denied states, and responsive sections in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/ResourceDetailSections.spec.js`
- [ ] T050 [P] [US1] Add page-flow tests for school detail/edit navigation from list to detail in no more than two interactions, list-query return, valid update, invalid update, conflict refresh path, and unsaved discard confirmation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/SchoolDetailEditPage.spec.js`
- [ ] T051 [P] [US1] Add page-flow tests for user and role detail/edit navigation from list to detail in no more than two interactions, role lookup/permission lookup requirements, valid update, validation preservation, direct denial, and unsaved school switch in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/AccessDetailEditPages.spec.js`
- [ ] T052 [P] [US1] Add page-flow tests for academic year and academic period detail/edit navigation from list to detail in no more than two interactions, valid update, date/sequence validation, no period-year reassignment, conflict feedback, and list return in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/AcademicDetailEditPages.spec.js`
- [ ] T053 [P] [US1] Add page-flow tests for guardian detail/edit navigation from list to detail in no more than two interactions, valid update, validation preservation, no user-link workflow, tenant denial, conflict feedback, and list return in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/GuardianDetailEditPage.spec.js`

### Implementation for User Story 1

- [ ] T054 [P] [US1] Extend school contracts with detail, update, status display, lifecycle capability metadata, and non-status update payload mapping in `schoolmaster-frontend/src/contracts/admin-system/schools.js`
- [ ] T055 [P] [US1] Extend user contracts with detail, update, status display, lifecycle capability metadata, role assignment mapping, and blocked direct-permission fields in `schoolmaster-frontend/src/contracts/admin-system/users.js`
- [ ] T056 [P] [US1] Extend access contracts with role detail/update mapping, permission assignment mapping, read-only permission metadata, and blocked scope editing in `schoolmaster-frontend/src/contracts/admin-system/access.js`
- [ ] T057 [P] [US1] Extend academic contracts with year/period detail and update mapping, date/sequence helpers, status display, and blocked period-year reassignment in `schoolmaster-frontend/src/contracts/admin-system/academics.js`
- [ ] T058 [P] [US1] Extend guardian contracts with detail, update, status display, lifecycle capability metadata, and blocked user-link fields in `schoolmaster-frontend/src/contracts/admin-system/guardians.js`
- [ ] T059 [US1] Implement explicit `getSchool` and `updateSchool` service functions with approved parameters, status omission, and normalized error mapping in `schoolmaster-frontend/src/services/admin-system/schools.js`
- [ ] T060 [US1] Implement explicit `getUser` and `updateUser` service functions with tenant context, role IDs, status omission, and normalized error mapping in `schoolmaster-frontend/src/services/admin-system/users.js`
- [ ] T061 [US1] Implement explicit `getRole` and `updateRole` service functions with tenant context, permission IDs, no scope editing, status omission, and normalized error mapping in `schoolmaster-frontend/src/services/admin-system/roles.js`
- [ ] T062 [US1] Implement explicit `getAcademicYear`, `updateAcademicYear`, `getAcademicPeriod`, and `updateAcademicPeriod` service functions with tenant context, date/sequence mapping, status omission, and normalized error mapping in `schoolmaster-frontend/src/services/admin-system/academic-years.js` and `schoolmaster-frontend/src/services/admin-system/academic-periods.js`
- [ ] T063 [US1] Implement explicit `getGuardian` and `updateGuardian` service functions with tenant context, contact/relationship mapping, status omission, and normalized error mapping in `schoolmaster-frontend/src/services/admin-system/guardians.js`
- [ ] T064 [P] [US1] Implement school detail and edit field sections with display-only status and no transport logic in `schoolmaster-frontend/src/components/admin-system/schools/SchoolDetailSections.vue` and `schoolmaster-frontend/src/components/admin-system/schools/SchoolEditFields.vue`
- [ ] T065 [P] [US1] Implement user detail and edit field sections with role assignment only and no direct permission controls in `schoolmaster-frontend/src/components/admin-system/users/UserDetailSections.vue` and `schoolmaster-frontend/src/components/admin-system/users/UserEditFields.vue`
- [ ] T066 [P] [US1] Implement role detail and edit field sections with permission assignment, display-only scope, and no status control in `schoolmaster-frontend/src/components/admin-system/roles/RoleDetailSections.vue` and `schoolmaster-frontend/src/components/admin-system/roles/RoleEditFields.vue`
- [ ] T067 [P] [US1] Implement academic year and academic period detail/edit sections with date/sequence fields and no academic-period year reassignment in `schoolmaster-frontend/src/components/admin-system/academic-years/AcademicYearDetailSections.vue`, `schoolmaster-frontend/src/components/admin-system/academic-years/AcademicYearEditFields.vue`, `schoolmaster-frontend/src/components/admin-system/academic-periods/AcademicPeriodDetailSections.vue`, and `schoolmaster-frontend/src/components/admin-system/academic-periods/AcademicPeriodEditFields.vue`
- [ ] T068 [P] [US1] Implement guardian detail and edit field sections with contact/relationship fields and no user-link workflow in `schoolmaster-frontend/src/components/admin-system/guardians/GuardianDetailSections.vue` and `schoolmaster-frontend/src/components/admin-system/guardians/GuardianEditFields.vue`
- [ ] T069 [US1] Compose school detail and edit pages with detail/update composables, permission visibility, conflict feedback, unsaved guard, and list-query return in `schoolmaster-frontend/src/pages/admin-system/schools/SchoolDetailPage.vue` and `schoolmaster-frontend/src/pages/admin-system/schools/EditSchoolPage.vue`
- [ ] T070 [US1] Compose user detail and edit pages with role lookup, tenant gating, detail/update composables, conflict feedback, unsaved guard, and list-query return in `schoolmaster-frontend/src/pages/admin-system/users/UserDetailPage.vue` and `schoolmaster-frontend/src/pages/admin-system/users/EditUserPage.vue`
- [ ] T071 [US1] Compose role detail and edit pages with permission lookup, `permissions.view` update gating, detail/update composables, conflict feedback, unsaved guard, and list-query return in `schoolmaster-frontend/src/pages/admin-system/roles/RoleDetailPage.vue` and `schoolmaster-frontend/src/pages/admin-system/roles/EditRolePage.vue`
- [ ] T072 [US1] Compose academic year and period detail/edit pages with tenant gating, detail/update composables, conflict feedback, unsaved guard, and list-query return in `schoolmaster-frontend/src/pages/admin-system/academic-years/AcademicYearDetailPage.vue`, `schoolmaster-frontend/src/pages/admin-system/academic-years/EditAcademicYearPage.vue`, `schoolmaster-frontend/src/pages/admin-system/academic-periods/AcademicPeriodDetailPage.vue`, and `schoolmaster-frontend/src/pages/admin-system/academic-periods/EditAcademicPeriodPage.vue`
- [ ] T073 [US1] Compose guardian detail and edit pages with tenant gating, detail/update composables, conflict feedback, unsaved guard, and list-query return in `schoolmaster-frontend/src/pages/admin-system/guardians/GuardianDetailPage.vue` and `schoolmaster-frontend/src/pages/admin-system/guardians/EditGuardianPage.vue`
- [ ] T074 [US1] Add school detail/edit routes with `schools.view` and `schools.view` plus `schools.manage` permissions, platform scope, breadcrumbs, titles, and return-list metadata in `schoolmaster-frontend/src/router/modules/schools.routes.js`
- [ ] T075 [US1] Add user, role, and permission read-only route updates with exact detail/edit permission matrices and no permission detail/edit/lifecycle routes in `schoolmaster-frontend/src/router/modules/access-administration.routes.js`
- [ ] T076 [US1] Add academic year and period detail/edit routes with school-context gating, permission matrices, breadcrumbs, titles, and return-list metadata in `schoolmaster-frontend/src/router/modules/academics.routes.js`
- [ ] T077 [US1] Add guardian detail/edit routes with school-context gating, permission matrices, breadcrumbs, titles, and return-list metadata in `schoolmaster-frontend/src/router/modules/guardians.routes.js`

**Checkpoint**: Detail and non-status update workflows are independently
functional for schools, users, roles, academic years, academic periods, and
guardians; permissions remain read-only.

---

## Phase 4: User Story 2 - Perform Single-Record Lifecycle Actions (Priority: P2)

**Goal**: Authorized administrators can activate, deactivate, soft delete, and
restore one eligible administrative record through audited confirmation
dialogs when status, permissions, tenant context, and contracts allow it.

**Independent Test**: For each resource and each eligible status, open the list
or detail action, confirm with an effective date and reason, and verify
success, validation, conflict, forbidden, not-found, and tenant-denial
outcomes.

### Tests for User Story 2

- [ ] T078 [P] [US2] Add school single lifecycle contract and service tests for `activateSchool`, `deactivateSchool`, `deleteSchool`, `restoreSchool`, soft-delete wording, payload mapping, and outcome reconciliation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/schools-single-lifecycle.spec.js`
- [ ] T079 [P] [US2] Add user single lifecycle contract and service tests for `activateUser`, `deactivateUser`, `deleteUser`, `restoreUser`, tenant headers, self-action conflict handling, and outcome reconciliation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/users-single-lifecycle.spec.js`
- [ ] T080 [P] [US2] Add role single lifecycle contract and service tests for `activateRole`, `deactivateRole`, `deleteRole`, `restoreRole`, dependency conflict handling, and outcome reconciliation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/roles-single-lifecycle.spec.js`
- [ ] T081 [P] [US2] Add academic single lifecycle contract and service tests for year and period activate/deactivate/delete/restore operations, dependency conflicts, tenant mismatch, and outcome reconciliation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/academics-single-lifecycle.spec.js`
- [ ] T082 [P] [US2] Add guardian single lifecycle contract and service tests for `activateGuardian`, `deactivateGuardian`, `deleteGuardian`, `restoreGuardian`, tenant mismatch, and outcome reconciliation in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/guardians-single-lifecycle.spec.js`
- [ ] T083 [P] [US2] Add list row action tests for visible eligible actions, hidden unauthorized actions, stale-client backend denial, restore visibility, no permission lifecycle actions, and no account lifecycle actions in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/components/ListRowLifecycleActions.spec.js`
- [ ] T084 [P] [US2] Add detail action tests for lifecycle dialog launch, reason/effective date validation, future-date rejection, duplicate-submit prevention, cancellation, dirty navigation confirmation, and reason preservation after temporary failure until deliberate cancel/close in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/DetailLifecycleActions.spec.js`
- [ ] T085 [P] [US2] Add lifecycle outcome page-flow tests for status tag refresh, row action refresh, lifecycle history refresh, list membership reconciliation, forbidden/not-found/tenant-safe states, and temporary failure retry without losing lifecycle reason in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/SingleLifecycleOutcomes.spec.js`
- [ ] T086 [P] [US2] Add privacy tests proving lifecycle reasons are not written to route query, shared diagnostics, support notifications, or long-lived stores in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/LifecyclePrivacy.spec.js`

### Implementation for User Story 2

- [ ] T087 [US2] Implement explicit school single lifecycle service functions for activate, deactivate, soft delete, and restore with normalized outcomes in `schoolmaster-frontend/src/services/admin-system/schools.js`
- [ ] T088 [US2] Implement explicit user single lifecycle service functions for activate, deactivate, soft delete, and restore with tenant context and normalized outcomes in `schoolmaster-frontend/src/services/admin-system/users.js`
- [ ] T089 [US2] Implement explicit role single lifecycle service functions for activate, deactivate, soft delete, and restore with tenant context and normalized outcomes in `schoolmaster-frontend/src/services/admin-system/roles.js`
- [ ] T090 [US2] Implement explicit academic year and period single lifecycle service functions for activate, deactivate, soft delete, and restore with tenant context and normalized outcomes in `schoolmaster-frontend/src/services/admin-system/academic-years.js` and `schoolmaster-frontend/src/services/admin-system/academic-periods.js`
- [ ] T091 [US2] Implement explicit guardian single lifecycle service functions for activate, deactivate, soft delete, and restore with tenant context and normalized outcomes in `schoolmaster-frontend/src/services/admin-system/guardians.js`
- [ ] T092 [US2] Integrate shared row lifecycle actions into school list/detail surfaces with no bulk school controls in `schoolmaster-frontend/src/pages/admin-system/schools/SchoolsListPage.vue` and `schoolmaster-frontend/src/pages/admin-system/schools/SchoolDetailPage.vue`
- [ ] T093 [US2] Integrate shared row lifecycle actions into user list/detail surfaces while keeping account lifecycle, invitation, password, lock, and reactivation workflows absent in `schoolmaster-frontend/src/pages/admin-system/users/UsersListPage.vue` and `schoolmaster-frontend/src/pages/admin-system/users/UserDetailPage.vue`
- [ ] T094 [US2] Integrate shared row lifecycle actions into role list/detail surfaces while keeping permission definitions read-only and role scope display-only in `schoolmaster-frontend/src/pages/admin-system/roles/RolesListPage.vue` and `schoolmaster-frontend/src/pages/admin-system/roles/RoleDetailPage.vue`
- [ ] T095 [US2] Integrate shared row lifecycle actions into academic year and academic period list/detail surfaces with dependency conflict feedback in `schoolmaster-frontend/src/pages/admin-system/academic-years/AcademicYearsListPage.vue`, `schoolmaster-frontend/src/pages/admin-system/academic-years/AcademicYearDetailPage.vue`, `schoolmaster-frontend/src/pages/admin-system/academic-periods/AcademicPeriodsListPage.vue`, and `schoolmaster-frontend/src/pages/admin-system/academic-periods/AcademicPeriodDetailPage.vue`
- [ ] T096 [US2] Integrate shared row lifecycle actions into guardian list/detail surfaces while keeping guardian user-link workflows absent in `schoolmaster-frontend/src/pages/admin-system/guardians/GuardiansListPage.vue` and `schoolmaster-frontend/src/pages/admin-system/guardians/GuardianDetailPage.vue`
- [ ] T097 [US2] Reconcile successful single lifecycle outcomes into status tags, row actions, detail summaries, lifecycle history, list rows, and list membership without undocumented response fields in `schoolmaster-frontend/src/composables/admin-system/useAdminLifecycleAction.js`
- [ ] T098 [US2] Add lifecycle action metadata, soft-delete labels, restore visibility rules, and hidden out-of-scope actions to the shared route/navigation assembly in `schoolmaster-frontend/src/router/modules/administration.routes.js`

**Checkpoint**: Single-record lifecycle actions work independently on list and
detail surfaces for all lifecycle-enabled resources.

---

## Phase 5: User Story 3 - Apply Approved Bulk Lifecycle Actions (Priority: P3)

**Goal**: Authorized administrators can select multiple eligible users, roles,
academic years, academic periods, or guardians and apply one approved
all-or-nothing lifecycle action through a confirmation dialog.

**Independent Test**: On each approved bulk-enabled list, select valid and
conflicting records, attempt each published bulk action within limits, verify
confirmation content, and validate full success, batch-level validation,
conflict, and denial states.

### Tests for User Story 3

- [ ] T099 [P] [US3] Add user bulk lifecycle contract and service tests for `bulkLifecycleUsers`, one action, unique IDs, 1 to 50 limit, reason/effective date payload, all-or-nothing success, and batch failure in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/users-bulk-lifecycle.spec.js`
- [ ] T100 [P] [US3] Add role bulk lifecycle contract and service tests for `bulkLifecycleRoles`, tenant headers, unique IDs, all-or-nothing dependency conflict, and batch feedback in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/roles-bulk-lifecycle.spec.js`
- [ ] T101 [P] [US3] Add academic bulk lifecycle contract and service tests for `bulkLifecycleAcademicYears`, `bulkLifecycleAcademicPeriods`, tenant headers, unique IDs, all-or-nothing dependency conflicts, and batch feedback in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/academics-bulk-lifecycle.spec.js`
- [ ] T102 [P] [US3] Add guardian bulk lifecycle contract and service tests for `bulkLifecycleGuardians`, tenant headers, unique IDs, all-or-nothing validation/conflict outcomes, and batch feedback in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/services/guardians-bulk-lifecycle.spec.js`
- [ ] T103 [P] [US3] Add bulk list page tests for users and roles covering selection across pages, tenant reset, permission reset, filter/page-size reset, confirmation content, successful refresh, and batch failure retention in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/AccessBulkLifecyclePages.spec.js`
- [ ] T104 [P] [US3] Add bulk list page tests for academic years and periods covering selection across pages, dependency conflicts, tenant reset, confirmation content, successful refresh, and batch failure retention in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/AcademicBulkLifecyclePages.spec.js`
- [ ] T105 [P] [US3] Add bulk list page tests for guardians covering selection across pages, tenant reset, confirmation content, successful refresh, validation/conflict failure retention, and no guardian user-link behavior in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/GuardianBulkLifecyclePages.spec.js`
- [ ] T106 [P] [US3] Add absence tests proving schools and permission definitions expose no bulk lifecycle controls or bulk lifecycle service functions in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/pages/BulkLifecycleAbsence.spec.js`

### Implementation for User Story 3

- [ ] T107 [US3] Implement explicit `bulkLifecycleUsers` service function with tenant context, one action, unique IDs, reason, today-or-past effective date, and normalized all-or-nothing outcomes in `schoolmaster-frontend/src/services/admin-system/users.js`
- [ ] T108 [US3] Implement explicit `bulkLifecycleRoles` service function with tenant context, one action, unique IDs, reason, today-or-past effective date, and normalized all-or-nothing outcomes in `schoolmaster-frontend/src/services/admin-system/roles.js`
- [ ] T109 [US3] Implement explicit `bulkLifecycleAcademicYears` and `bulkLifecycleAcademicPeriods` service functions with tenant context, one action, unique IDs, reason, today-or-past effective date, and normalized all-or-nothing outcomes in `schoolmaster-frontend/src/services/admin-system/academic-years.js` and `schoolmaster-frontend/src/services/admin-system/academic-periods.js`
- [ ] T110 [US3] Implement explicit `bulkLifecycleGuardians` service function with tenant context, one action, unique IDs, reason, today-or-past effective date, and normalized all-or-nothing outcomes in `schoolmaster-frontend/src/services/admin-system/guardians.js`
- [ ] T111 [US3] Integrate bulk selection, selection clearing, all-or-nothing confirmation, batch feedback, and refresh reconciliation into user and role list pages in `schoolmaster-frontend/src/pages/admin-system/users/UsersListPage.vue` and `schoolmaster-frontend/src/pages/admin-system/roles/RolesListPage.vue`
- [ ] T112 [US3] Integrate bulk selection, selection clearing, all-or-nothing confirmation, dependency conflict feedback, and refresh reconciliation into academic year and period list pages in `schoolmaster-frontend/src/pages/admin-system/academic-years/AcademicYearsListPage.vue` and `schoolmaster-frontend/src/pages/admin-system/academic-periods/AcademicPeriodsListPage.vue`
- [ ] T113 [US3] Integrate bulk selection, selection clearing, all-or-nothing confirmation, batch feedback, and refresh reconciliation into guardian list pages in `schoolmaster-frontend/src/pages/admin-system/guardians/GuardiansListPage.vue`
- [ ] T114 [US3] Ensure school and permission resource contracts, route metadata, pages, and services expose no bulk lifecycle capability in `schoolmaster-frontend/src/contracts/admin-system/lifecycle.js`, `schoolmaster-frontend/src/router/modules/schools.routes.js`, and `schoolmaster-frontend/src/router/modules/access-administration.routes.js`

**Checkpoint**: Approved bulk lifecycle workflows work independently for users,
roles, academic years, academic periods, and guardians; schools and permissions
have no bulk lifecycle UI.

---

## Phase 6: Polish and Cross-Cutting Verification

**Purpose**: Full-feature validation, accessibility, privacy, scope control,
performance, and documentation traceability.

- [ ] T115 [P] Add end-to-end route assembly tests for all administration lifecycle route modules registered through the main router in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/routes/administration-lifecycle-registration.spec.js`
- [ ] T116 [P] Add responsive and keyboard-operability coverage for detail, edit, row action, confirmation, selection, bulk, validation, and recovery controls at 390px, 768px, and 1440px in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/accessibility/lifecycle-responsive-keyboard.spec.js`
- [ ] T117 [P] Add performance-oriented tests proving mocked detail and lifecycle service latency up to 1.5 seconds renders stable content, denial, conflict, or recoverable error within 2 seconds in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/performance/lifecycle-latency.spec.js`
- [ ] T118 [P] Add scope guard tests proving no account lifecycle, invitation, password setup/reset, account lock/reactivation, guardian user-link, student management, reporting, platform support, permanent purge, or undocumented route behavior appears in `schoolmaster-frontend/tests/unit/admin-system/administration-lifecycle/scope/lifecycle-scope-absence.spec.js`
- [ ] T119 Run `npm run test:unit -- --run` from `schoolmaster-frontend` and record the result in `specs/020-administration-lifecycle-ui/quickstart.md`
- [ ] T120 Run `npm run build` from `schoolmaster-frontend` and record the result in `specs/020-administration-lifecycle-ui/quickstart.md`
- [ ] T121 Run source audits for direct Axios outside services, endpoint strings outside services, kebab-case Element Plus tags, update-form status submission, lifecycle reason leakage, and out-of-scope actions, then record findings in `specs/020-administration-lifecycle-ui/quickstart.md`
- [ ] T122 Record manual responsive review, keyboard review, latency review, denial/conflict/privacy review, OpenAPI operation ID verification, backend readiness verification, and five-administrator UAT outcome in `specs/020-administration-lifecycle-ui/quickstart.md`

---

## Dependencies and Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks all user stories.
- **Phase 3 US1**: Depends on Phase 2 and is the MVP.
- **Phase 4 US2**: Depends on Phase 2; can proceed in parallel with US1 after shared foundations, but list/detail integration is easier after US1 pages exist.
- **Phase 5 US3**: Depends on Phase 2; benefits from US2 lifecycle services and dialogs but remains independently testable through bulk service/page tests.
- **Phase 6 Polish**: Depends on completed story slices selected for delivery.

### User Story Dependencies

- **US1 Review and Edit Administrative Records**: Independent after foundational work.
- **US2 Perform Single-Record Lifecycle Actions**: Independent after foundational work; integrates with US1 detail pages when present.
- **US3 Apply Approved Bulk Lifecycle Actions**: Independent after foundational work for bulk-enabled lists; uses the same confirmation and error foundations as US2.

### Within Each Story

- Write and run story tests first; confirm they fail before implementation.
- Contracts and services precede composables and pages.
- Shared components and composables stay transport-free.
- Services own Axios calls and approved OpenAPI operation mapping.
- Route pages compose contracts, services, composables, and components only.
- Each story reaches its checkpoint before relying on it for a later story.

## Parallel Examples

### User Story 1

```bash
# Contract tests can be developed in parallel:
T039 T040 T041 T042 T043

# Resource component sections can be implemented in parallel:
T064 T065 T066 T067 T068
```

### User Story 2

```bash
# Single lifecycle service tests can be developed in parallel:
T078 T079 T080 T081 T082

# Resource list/detail integrations can be implemented in parallel:
T092 T093 T094 T095 T096
```

### User Story 3

```bash
# Bulk lifecycle service tests can be developed in parallel:
T099 T100 T101 T102

# Bulk page integrations can be implemented in parallel:
T111 T112 T113
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 setup.
2. Complete Phase 2 foundational lifecycle contracts, services, components,
   composables, route helpers, and tests.
3. Complete Phase 3 User Story 1.
4. Validate detail and non-status update workflows for all approved resources.

### Incremental Delivery

1. Deliver US1 for detail and update workflows.
2. Deliver US2 for single-record lifecycle actions.
3. Deliver US3 for approved bulk lifecycle workflows.
4. Run Phase 6 verification after each delivered increment and again before
   final release.

## Notes

- Backend authorization remains authoritative even when frontend visibility
  hides actions.
- Permission definitions remain read-only for this feature.
- Edit forms never submit `status`.
- Lifecycle effective dates are today-or-past only.
- Bulk lifecycle is all-or-nothing and has no school support.
- No account lifecycle, guardian user-link, student management, reporting,
  permanent purge, or undocumented route behavior is part of this feature.
