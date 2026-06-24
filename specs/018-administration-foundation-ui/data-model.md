# Data Model: Administration Foundation UI

This feature adds no database entities. Models below define frontend state,
forms, service mappings, and relationships to approved OpenAPI records.

## AdministrationRoute

**Fields**:

- `name`: Stable route name.
- `resource`: School, user, role, permission, academic year, academic period,
  or guardian.
- `mode`: List or create.
- `requiresAuth`: Always true.
- `requiresSchoolContext`: False for schools; true for tenant-owned modules.
- `permissions`: Exact permission codes required by the route or action:
  `schools.view`, `schools.manage`, `users.view`, `users.manage`, `roles.view`,
  `roles.manage`, `permissions.view`, `academic_years.view`,
  `academic_years.manage`, `academic_periods.view`,
  `academic_periods.manage`, `guardians.view`, `guardians.manage`, or
  `student_profiles.view`.
- `layout`: Existing admin-system layout.
- `titleKey`, `breadcrumb`, `sidebar`: Localized shell metadata.

**Rules**:

- Create routes exist for every resource except permission.
- Unauthorized navigation remains hidden, but direct-route denial is handled.
- Tenant-owned routes cannot load before active school resolution.
- Missing unresolved school context remains blocked by feature 017; routes do
  not populate school selection from `listSchools`.

## AdministrationListQuery

**Fields**:

- `page`: Positive integer; default 1.
- `perPage`: Approved page-size value.
- `status`: Approved status value where operation supports it.
- `sort`: Approved sort value where operation supports it.
- `academicYearId`: UUID used only for academic-period lists.

**Relationships**:

- Parsed from and serialized to route query.
- Mapped to snake_case API parameters by resource service.
- Produces `AdministrationListState`.

**Rules**:

- Unsupported or malformed values normalize to defaults.
- Unsupported keys are not forwarded.
- Filter or page-size changes reset page to 1.
- Query values contain no tenant authorization state.
- School list queries omit `sort` until backend implementation honors it.

## AdministrationListState

**Fields**:

- `items`: Current normalized resource summaries.
- `meta`: `page`, `perPage`, and `total`.
- `status`: Idle, loading, ready, empty, filtered-empty, forbidden,
  tenant-mismatch, unavailable, or error.
- `error`: Normalized `AdministrationFeedback`.
- `requestKey`: Current operation, route query, and tenant generation.

**Rules**:

- Only latest request matching current route and tenant may update state.
- School change clears tenant-owned items before next request.
- Empty final pages navigate to nearest valid page once.

## AdministrationFormState

**Fields**:

- `initialValues`: Contract-shaped empty model.
- `values`: Current user-entered model.
- `fieldErrors`: Contract field to localized messages.
- `formError`: Optional normalized non-field error.
- `status`: Idle, submitting, succeeded, validation-error, forbidden,
  tenant-mismatch, not-found, unavailable, or error.
- `isDirty`: Derived comparison with initial values.
- `submitted`: Successful-submit flag that disables leave warning.
- `returnQuery`: Validated list query restored after success/cancel.

**Rules**:

- Duplicate submit is blocked while submitting.
- Every dirty route exit requires discard confirmation.
- Successful submit clears dirty/error state before list navigation.
- Tenant switch uses same discard confirmation and clears draft after approval.

## AdministrationLookupState

**Fields**:

- `items`: Current normalized lookup page.
- `selectedItems`: Selected options retained independently from the current
  page.
- `meta`: `page`, `perPage`, and `total`.
- `status`: Idle, loading, ready, empty, unavailable, or error.
- `requestKey`: Current operation, approved lookup parameters, and tenant
  generation.

**Rules**:

- Role, permission, and academic-year selectors expose server-driven page
  traversal because their approved operations are paginated and do not publish
  general search.
- Lookup requests send only operation-approved page, page-size, and supported
  status parameters.
- Every returned page is reachable; implementations must not assume the first
  page contains all selectable references.
- Selected options remain visible and submittable while another page is shown.
- Only the latest request for the current tenant may update options.
- School change clears current and selected tenant-owned options.

## School

**Fields**: `id`, `name`, `code`, `status`, `contactEmail`, `contactPhone`,
`addressSummary`.

## SchoolCreateForm

**Fields**: `name`, `code`, `contactEmail`, `contactPhone`, `addressSummary`.

**Rules**: Name and code required; email must be valid when supplied.

## User

**Fields**: `id`, `schoolId`, `fullName`, `email`, `status`, `roles`.

## UserCreateForm

**Fields**: `fullName`, `email`, `roleIds`.

**Relationships**: Role choices come from paginated, active permitted
`listRoles` results coordinated through `AdministrationLookupState`.

**Rules**: Full name, email, and at least the contract-required role selection
are submitted; direct permission assignment is absent.

## Role

**Fields**: `id`, `schoolId`, `scope`, `name`, `status`, `permissions`.

## RoleCreateForm

**Fields**: `name`, `permissionIds`.

**Relationships**: Permission choices come from paginated `listPermissions`
results coordinated through `AdministrationLookupState`.

**Rules**: Only active school-scope permissions are selectable. The UI has no
scope control. The service mapper adds contract-required `scope=school` and
uses active school tenant context. Platform-role creation is outside scope.

## Permission

**Fields**: `id`, `code`, `name`, `scope`, `status`.

**Rules**: Read-only in this feature; no create, edit, lifecycle, or direct user
assignment action.

## AcademicYear

**Fields**: `id`, `schoolId`, `name`, `startDate`, `endDate`, `status`.

## AcademicYearCreateForm

**Fields**: `name`, `startDate`, `endDate`.

**Rules**: All fields required; end date cannot precede start date in client
guidance, while backend validation remains authoritative.

## AcademicPeriod

**Fields**: `id`, `schoolId`, `academicYearId`, `name`, `sequence`,
`startDate`, `endDate`, `status`.

## AcademicPeriodCreateForm

**Fields**: `academicYearId`, `name`, `sequence`, `startDate`, `endDate`.

**Relationships**: Academic-year choices come from active-school
`listAcademicYears` pages coordinated through `AdministrationLookupState`.

**Rules**: Parent year and all fields required; sequence is integer; displayed
date guidance uses selected year bounds; backend remains authoritative.

## Guardian

**Fields**: `id`, `schoolId`, `fullName`, `relationshipType`, `contactEmail`,
`contactPhone`, `status`.

## GuardianCreateForm

**Fields**: `fullName`, `relationshipType`, `contactEmail`, `contactPhone`,
`studentProfileIds`.

**Relationships**: `studentProfileIds` come from paginated, remote
`StudentProfileLookupOption` results in active school.

**Rules**: Full name and relationship required; contact email validated when
supplied; failed association means no partial success is shown.

## StudentProfileLookupOption

**Fields**: `id`, `schoolId`, `registrationNumber`, `fullName`, `status`,
`enrolledAt`, `label`.

**Rules**:

- Loaded only through `listStudentProfiles` with `status=active`.
- Search, status, page, page size, and sort use approved parameters.
- Old-school or old-search results cannot update current options.
- Selected values submit UUIDs, not labels.

## AdministrationFeedback

**Fields**:

- `type`: Validation, unauthorized, forbidden, tenant-mismatch,
  inactive-context, not-found, unavailable, conflict, or unknown.
- `messageKey`: Centralized locale key.
- `fieldErrors`: Optional field map.
- `recoveryAction`: Sign in, return, choose school, retry, reset filters, or
  none.
- `operationId`: Diagnostic operation name.
- `requestId`: Optional safe correlation identifier.

**Rules**:

- Must not expose cross-tenant existence or raw sensitive payloads.
- Unauthorized delegates session recovery to feature 017.
- Validation preserves form values.
