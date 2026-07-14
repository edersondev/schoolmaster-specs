# Administration Lifecycle UI Contract

## Scope

Frontend may implement detail pages, non-status update forms, single-record
activate/deactivate/soft-delete/restore confirmations, and approved
all-or-nothing bulk lifecycle workflows for schools, users, roles, academic
years, academic periods, and guardians.

Permissions remain read-only. Account lifecycle, invitation, password,
account-lock, guardian user-link, student management, reporting, platform
support, permanent purge, and undocumented backend routes remain blocked.

## Approved Operations

| Operation | Method and path | Frontend use |
|-----------|-----------------|--------------|
| `getSchool` | `GET /api/v1/schools/{schoolId}` | Platform school detail |
| `updateSchool` | `PATCH /api/v1/schools/{schoolId}` | School non-status update |
| `activateSchool` | `POST /api/v1/schools/{schoolId}/activate` | School activation confirmation |
| `deactivateSchool` | `POST /api/v1/schools/{schoolId}/deactivate` | School deactivation confirmation |
| `deleteSchool` | `DELETE /api/v1/schools/{schoolId}` | School soft-delete confirmation |
| `restoreSchool` | `POST /api/v1/schools/{schoolId}/restore` | School restore confirmation |
| `getUser` | `GET /api/v1/users/{userId}` | Active-school user detail |
| `updateUser` | `PATCH /api/v1/users/{userId}` | User non-status update |
| `activateUser` | `POST /api/v1/users/{userId}/activate` | User activation confirmation |
| `deactivateUser` | `POST /api/v1/users/{userId}/deactivate` | User deactivation confirmation |
| `deleteUser` | `DELETE /api/v1/users/{userId}` | User soft-delete confirmation |
| `restoreUser` | `POST /api/v1/users/{userId}/restore` | User restore confirmation |
| `bulkLifecycleUsers` | `POST /api/v1/users/bulk-lifecycle` | User all-or-nothing bulk lifecycle |
| `getRole` | `GET /api/v1/roles/{roleId}` | Active-school role detail |
| `updateRole` | `PATCH /api/v1/roles/{roleId}` | Role non-status update |
| `activateRole` | `POST /api/v1/roles/{roleId}/activate` | Role activation confirmation |
| `deactivateRole` | `POST /api/v1/roles/{roleId}/deactivate` | Role deactivation confirmation |
| `deleteRole` | `DELETE /api/v1/roles/{roleId}` | Role soft-delete confirmation |
| `restoreRole` | `POST /api/v1/roles/{roleId}/restore` | Role restore confirmation |
| `bulkLifecycleRoles` | `POST /api/v1/roles/bulk-lifecycle` | Role all-or-nothing bulk lifecycle |
| `getAcademicYear` | `GET /api/v1/academic-years/{academicYearId}` | Active-school academic-year detail |
| `updateAcademicYear` | `PATCH /api/v1/academic-years/{academicYearId}` | Academic-year non-status update |
| `activateAcademicYear` | `POST /api/v1/academic-years/{academicYearId}/activate` | Academic-year activation confirmation |
| `deactivateAcademicYear` | `POST /api/v1/academic-years/{academicYearId}/deactivate` | Academic-year deactivation confirmation |
| `deleteAcademicYear` | `DELETE /api/v1/academic-years/{academicYearId}` | Academic-year soft-delete confirmation |
| `restoreAcademicYear` | `POST /api/v1/academic-years/{academicYearId}/restore` | Academic-year restore confirmation |
| `bulkLifecycleAcademicYears` | `POST /api/v1/academic-years/bulk-lifecycle` | Academic-year all-or-nothing bulk lifecycle |
| `getAcademicPeriod` | `GET /api/v1/academic-periods/{academicPeriodId}` | Active-school academic-period detail |
| `updateAcademicPeriod` | `PATCH /api/v1/academic-periods/{academicPeriodId}` | Academic-period non-status update |
| `activateAcademicPeriod` | `POST /api/v1/academic-periods/{academicPeriodId}/activate` | Academic-period activation confirmation |
| `deactivateAcademicPeriod` | `POST /api/v1/academic-periods/{academicPeriodId}/deactivate` | Academic-period deactivation confirmation |
| `deleteAcademicPeriod` | `DELETE /api/v1/academic-periods/{academicPeriodId}` | Academic-period soft-delete confirmation |
| `restoreAcademicPeriod` | `POST /api/v1/academic-periods/{academicPeriodId}/restore` | Academic-period restore confirmation |
| `bulkLifecycleAcademicPeriods` | `POST /api/v1/academic-periods/bulk-lifecycle` | Academic-period all-or-nothing bulk lifecycle |
| `getGuardian` | `GET /api/v1/guardians/{guardianId}` | Active-school guardian detail |
| `updateGuardian` | `PATCH /api/v1/guardians/{guardianId}` | Guardian non-status update |
| `activateGuardian` | `POST /api/v1/guardians/{guardianId}/activate` | Guardian activation confirmation |
| `deactivateGuardian` | `POST /api/v1/guardians/{guardianId}/deactivate` | Guardian deactivation confirmation |
| `deleteGuardian` | `DELETE /api/v1/guardians/{guardianId}` | Guardian soft-delete confirmation |
| `restoreGuardian` | `POST /api/v1/guardians/{guardianId}/restore` | Guardian restore confirmation |
| `bulkLifecycleGuardians` | `POST /api/v1/guardians/bulk-lifecycle` | Guardian all-or-nothing bulk lifecycle |

Existing foundation list operations remain approved for navigation,
post-action refresh, and selectors: `listSchools`, `listUsers`, `listRoles`,
`listPermissions`, `listAcademicYears`, `listAcademicPeriods`,
`listGuardians`, and `listStudentProfiles` where already used.

## Route Contract

- All routes require authentication and existing session bootstrap.
- School lifecycle routes are platform-scoped.
- Users, roles, academic years, academic periods, guardians, and selectors
  require confirmed active school context.
- If feature 017 cannot confirm an active school, tenant-owned lifecycle
  routes remain blocked and send no tenant-owned administration request.
- Route metadata defines layout, title, breadcrumb, sidebar placement,
  permission codes, auth requirement, school-context requirement, and return
  list route.
- Unauthorized navigation items, row actions, detail actions, edit actions,
  lifecycle actions, and bulk controls are hidden.
- Direct denied routes show established forbidden or tenant feedback without
  rendering protected resource data.
- Dirty edit routes and dirty lifecycle confirmations use the existing
  unsaved-change guard for every browser or in-app exit.

## Permission Contract

| Surface | Required session permissions |
|---------|------------------------------|
| School detail | `schools.view` |
| School update/lifecycle | `schools.view`, `schools.manage` |
| User detail | `users.view` |
| User update | `users.view`, `users.manage`, `roles.view` |
| User lifecycle/bulk | `users.view`, `users.manage` |
| Role detail | `roles.view` |
| Role update | `roles.view`, `roles.manage`, `permissions.view` |
| Role lifecycle/bulk | `roles.view`, `roles.manage` |
| Permission reference choices | `permissions.view` |
| Academic-year detail | `academic_years.view` |
| Academic-year update/lifecycle/bulk | `academic_years.view`, `academic_years.manage` |
| Academic-period detail | `academic_periods.view` |
| Academic-period update/lifecycle/bulk | `academic_periods.view`, `academic_periods.manage` |
| Guardian detail | `guardians.view` |
| Guardian update/lifecycle/bulk | `guardians.view`, `guardians.manage` |

Every permission listed for a surface is required for frontend visibility and
route access. Backend authorization remains authoritative.

System Administrator exception: the active platform `System Administrator`
role satisfies the listed feature-specific permission checks. Authentication,
active tenant context, tenant isolation, lifecycle dependencies, confirmation,
validation, and audit requirements remain enforceable. Each System
Administrator update or lifecycle action must record
`master_access_used: true`. Frontend adoption is deferred.

## Update Contract

- Update forms submit only documented non-status request fields.
- School update fields: `name`, `contact_email`, `contact_phone`, `address`.
- User update fields: `full_name`, `email`, `role_ids`.
- Role update fields: `name`, `permission_ids`.
- Academic-year update fields: `name`, `start_date`, `end_date`.
- Academic-period update fields: `name`, `sequence`, `start_date`,
  `end_date`.
- Guardian update fields: `full_name`, `relationship_type`, `contact_email`,
  `contact_phone`.
- Status is display-only in edit forms even where the OpenAPI update schema
  publishes a status property.
- User update requires `users.view`, `users.manage`, and `roles.view` because
  role assignment uses the approved role lookup. It assigns roles only and
  never assigns direct permissions.
- Role update requires `roles.view`, `roles.manage`, and `permissions.view`,
  does not expose scope editing, and allows only permitted school-scope
  permission assignments.
- Academic-period update does not expose academic-year reassignment.
- Field errors map to matching controls; unmatched errors appear in an
  accessible form summary.
- Valid entered values survive validation failure.
- Submit is disabled or ignored while request is pending.
- Successful update clears dirty/error state and reconciles detail/list state.

## Lifecycle Action Contract

- Single-record lifecycle actions are `activate`, `deactivate`, `delete`, and
  `restore`.
- Every lifecycle action uses a confirmation dialog with target resource,
  current status, action, expected outcome, required today-or-past
  `effective_at`, required `reason`, and pending state.
- Reason is 1 to 500 characters.
- Future effective dates are invalid and are not presented as scheduled
  lifecycle actions.
- Delete labels use soft-delete language. Permanent deletion, purge, and
  anonymization remain blocked.
- Restore controls appear only when an approved response exposes a restorable
  record to the requester.
- Successful lifecycle outcomes reconcile status tags, row actions, detail
  summary, lifecycle history, list membership, and selection state.

## Bulk Lifecycle Contract

- Bulk lifecycle is approved only for users, roles, academic years, academic
  periods, and guardians.
- Schools and permissions expose no bulk lifecycle controls.
- Bulk requests submit exactly one resource type, one lifecycle action, 1 to
  50 unique selected record identifiers, one today-or-past `effective_at`, and
  one reason.
- Bulk lifecycle is all-or-nothing. Validation or conflict failure keeps all
  selected records presented as unchanged and shows batch-level feedback.
- Selection is cleared or revalidated when resource, route, filters, page size,
  active school, session, or permission state changes.
- Selection may span pages only while records remain in the same resource and
  active tenant context.

## Service Contract

- Axios calls exist only in service/API-client boundaries.
- Resource services export explicit functions for each consumed lifecycle
  operation; pages, components, composables, and router guards call service
  functions, not endpoints.
- Services map camelCase frontend values to approved snake_case request
  fields.
- Services omit status from update payloads.
- Tenant headers come from confirmed session context, not route query or form
  input.
- Stale detail, update, lifecycle, and bulk requests are cancelled or ignored.
- Components, pages, composables, and router guards never inspect raw Axios
  errors.

## Feedback and Privacy Contract

Supported normalized outcomes:

- loading
- validation
- unauthorized/session expired
- forbidden
- tenant mismatch
- inactive context
- not found
- conflict
- temporary unavailable
- unknown error

Not-found and denial messages must not reveal whether inaccessible records
exist in another school. Diagnostics may include operation ID, safe error code,
route name, and request identifier, but not form values, lifecycle reason text,
emails, tokens, tenant data, selected hidden identifiers, or permission
payloads.

## Verification Contract

Vitest coverage must verify:

- route metadata, hidden navigation, hidden actions, direct-route denial, and
  tenant gating
- blocked unresolved school selection without tenant-owned lifecycle requests
- approved service functions, parameters, payload mappings, and omission of
  update status fields
- success, validation, forbidden, not-found, conflict, and temporary failure
  envelope mappings
- stale request cancellation or ignore behavior
- detail page loading, not-found, forbidden, tenant mismatch, conflict, and
  list-return behavior
- update form dirty state, validation mapping, pending state, successful
  reconciliation, and unsaved-route guards
- status tag and action eligibility behavior for each resource
- lifecycle dialog required reason, today-or-past effective date, future-date
  rejection, duplicate-submit prevention, and soft-delete wording
- restore visibility limited to approved record visibility
- bulk selection limit, tenant reset, all-or-nothing success, batch-level
  validation/conflict failure, and absence of school bulk lifecycle
- absence of permission lifecycle, account lifecycle, guardian user-link,
  student management, and undocumented route behavior
- responsive and keyboard-operable critical actions at 390px, 768px, and
  1440px
- detail feedback within 2 seconds when mocked service latency is at most 1.5
  seconds, measured from route navigation

No OpenAPI file changes are part of this contract.
