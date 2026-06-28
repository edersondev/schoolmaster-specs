# Administration Foundation UI Contract

## Scope

Frontend may implement list pages for schools, users, roles, permissions,
academic years, academic periods, and guardians. It may implement dedicated
create pages for every listed resource except permissions. It may use student
profiles only as a guardian-association lookup.

Detail, update, activate, deactivate, delete, restore, bulk lifecycle, account
lifecycle, guardian user links, and student management remain blocked.

## Approved Operations

| Operation | Method and path | Frontend use |
|-----------|-----------------|--------------|
| `listSchools` | `GET /api/v1/schools` | Platform school list |
| `createSchool` | `POST /api/v1/schools` | Dedicated school create route |
| `listUsers` | `GET /api/v1/users` | Active-school user list and role-dependent form data |
| `createUser` | `POST /api/v1/users` | Dedicated user create route |
| `listRoles` | `GET /api/v1/roles` | Role list and user role choices |
| `createRole` | `POST /api/v1/roles` | Dedicated role create route |
| `listPermissions` | `GET /api/v1/permissions` | Read-only list and role permission choices |
| `listAcademicYears` | `GET /api/v1/academic-years` | Year list and period parent choices |
| `createAcademicYear` | `POST /api/v1/academic-years` | Dedicated year create route |
| `listAcademicPeriods` | `GET /api/v1/academic-periods` | Period list filtered by year where requested |
| `createAcademicPeriod` | `POST /api/v1/academic-periods` | Dedicated period create route |
| `listGuardians` | `GET /api/v1/guardians` | Active-school guardian list |
| `createGuardian` | `POST /api/v1/guardians` | Dedicated guardian create route |
| `listStudentProfiles` | `GET /api/v1/student-profiles` | Remote guardian association lookup only |

## Route Contract

- All routes require authentication and existing session bootstrap.
- School routes are platform-scoped.
- Users, roles, permissions, academic years, academic periods, guardians, and
  student lookup require confirmed active school context.
- If feature 017 cannot confirm an active school, tenant-owned administration
  remains blocked. This feature does not use `listSchools` or another
  unapproved source to populate school selection.
- Route metadata defines layout, title, breadcrumb, sidebar placement,
  permission codes, auth requirement, and school-context requirement.
- Unauthorized navigation items and create actions are hidden.
- Direct denied routes show established forbidden or tenant feedback without
  rendering protected resource data.
- Create routes use an unsaved-change guard for every browser or in-app exit.

### Permission Matrix

| Surface | Required session permissions |
|---------|------------------------------|
| School list | `schools.view` |
| School create | `schools.view`, `schools.manage` |
| User list | `users.view` |
| User create | `users.view`, `users.manage`, `roles.view` |
| Role list | `roles.view` |
| Role create | `roles.view`, `roles.manage`, `permissions.view` |
| Permission list | `permissions.view` |
| Academic-year list | `academic_years.view` |
| Academic-year create | `academic_years.view`, `academic_years.manage` |
| Academic-period list | `academic_periods.view` |
| Academic-period create | `academic_periods.view`, `academic_periods.manage`, `academic_years.view` |
| Guardian list | `guardians.view` |
| Guardian create | `guardians.view`, `guardians.manage` |
| Guardian student lookup | `student_profiles.view` |

Every permission listed for a surface is required for frontend visibility and
route access. Backend authorization remains authoritative.

## List Query Contract

- URL query is source of truth for approved list state.
- Common approved keys are `page`, `per_page`, `status`, and `sort`, but each
  resource exposes only keys published by its operation.
- School list UI omits `sort` because current backend behavior ignores the
  published parameter; enable it only after backend compliance is verified.
- Academic periods additionally support `academic_year_id`.
- Student-profile lookup additionally supports approved `search`, status, and
  sort parameters.
- Unsupported, malformed, or out-of-range values normalize to defaults and are
  never forwarded.
- Filters and page-size changes reset page to 1.
- Create-route return navigation restores validated list query.

## Service Contract

- Axios calls exist only in service/API-client boundaries.
- Every resource service exports explicit list/create functions rather than a
  generic unreviewed CRUD transport.
- Services map camelCase frontend values to approved snake_case request fields.
- Services map success and paginated envelopes into frontend contracts.
- Tenant headers come from confirmed session context, not route query or form
  input.
- Stale list and lookup requests are cancelled or ignored.
- Components, pages, composables, and router guards never inspect raw Axios
  errors.

## Lookup Contract

- User-role, role-permission, and period-year selectors use server-driven
  pagination over `listRoles`, `listPermissions`, and `listAcademicYears`.
- Those operations publish no general search parameter, so selectors expose
  explicit page traversal or an equivalent load-more control rather than
  silently limiting choices to the first page.
- Lookup calls send only operation-approved page, page-size, and supported
  status parameters.
- Selected options remain visible and submittable while another page is
  displayed.
- Tenant changes cancel or invalidate pending lookup requests and clear both
  current and selected tenant-owned options.

## Component Contract

- Route pages compose shared and resource-specific components.
- Shared list components expose props/slots for data, columns, filters,
  loading, empty, feedback, and pagination.
- Resource components define fields, columns, labels, and approved choices.
- Props are read-only; user intent is emitted upward.
- Element Plus components use PascalCase tags.
- Shared UI text uses Vue I18n.
- Tailwind handles spacing, layout, responsiveness, and restrained visual
  refinement.

## Form Contract

- Forms submit only documented request fields.
- Field errors map to matching controls; unmatched errors appear in accessible
  form summary.
- Valid entered values survive validation failure.
- Submit is disabled or ignored while request is pending.
- Successful creation clears dirty/error state, shows success feedback, and
  returns to the resource list with prior validated query.
- Cancel or any route exit prompts only when values differ from initial state.
- Permission remains read-only.
- User form assigns roles only and can traverse every permitted role page.
- Role form has no scope selector, assigns active school-scope permissions
  only, can traverse every permitted permission page, and service mapping
  submits `scope=school`.
- Academic-period form can traverse every permitted academic-year page.
- Guardian student choices are hidden when `student_profiles.view` is absent;
  when present, choices come from remote `listStudentProfiles` with
  `status=active`.

## State and Tenancy Contract

- Existing Pinia session store owns current user, permissions, and active school.
- Existing shell store owns shell/navigation state.
- List state lives in URL query plus route-local composable state.
- Form drafts live in route-local composable state.
- No resource store is required unless future implementation proves state must
  cross route boundaries.
- Confirmed school change clears tenant-owned data, requests, lookups, and
  drafts before new-school data renders.
- Platform access never implies school-owned access.

## Feedback and Privacy Contract

Supported normalized outcomes:

- loading
- empty
- filtered empty
- validation
- unauthorized/session expired
- forbidden
- tenant mismatch
- inactive context
- not found
- temporary unavailable
- unknown error

Not-found and denial messages must not reveal whether inaccessible records exist
in another school. Diagnostics may include operation ID, safe error code, route
name, and request identifier, but not form values, emails, tokens, tenant data,
or permission payloads.

## Verification Contract

Vitest coverage must verify:

- route metadata, hidden navigation, direct-route denial, and tenant gating
- blocked unresolved school selection without administration API requests
- query parsing, serialization, normalization, refresh, and back navigation
- approved service parameters and request payload mappings
- paginated envelope and error mappings
- stale request cancellation/ignore behavior
- reusable list/table/filter/pagination/feedback behavior
- each create form's fields, validation mapping, pending state, and success
  navigation
- paginated role, permission, academic-year, and student lookup behavior,
  including selected-option retention and tenant reset
- exact permission-matrix behavior, including multi-permission create routes
- absence of platform scope selection in role creation
- school-switch and generic unsaved-route confirmations
- responsive and keyboard-operable critical actions
- responsive behavior at 390px, 768px, and 1440px
- list feedback within 2 seconds when mocked service latency is at most 1.5
  seconds, measured from route navigation or committed query change

No OpenAPI file changes are part of this contract.
