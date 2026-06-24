# Feature Specification: Administration Foundation UI

**Feature Branch**: `018-administration-foundation-ui`
**Created**: 2026-06-24
**Status**: Draft
**Input**: User description: "Define the Administration Foundation UI for schools, users, roles, permissions, academic years, academic periods, and guardians. Cover reusable list/create pages, filters, data tables, forms, pagination, empty states, validation errors, authorization denials, and not-found handling, consuming only approved and implemented OpenAPI operations."

## Clarifications

### Session 2026-06-24

- Q: When the active school changes with an unsaved form, what behavior applies? → A: Require confirmation before switching schools.
- Q: How should the guardian form select optional student associations? → A: Use the approved same-school student-profile list as a lookup.
- Q: Where should list filters, sorting, and pagination state persist? → A: Persist approved list state in the URL query.
- Q: What create workflow pattern should all resources use? → A: Use dedicated create routes.
- Q: How should unsaved form data be handled when the user leaves a create route? → A: Require confirmation before every route exit.
- Q: What happens when feature 017 cannot resolve an active school? → A:
  Keep tenant-owned administration blocked; this feature does not introduce or
  populate school selection without an approved user-authorized source.
- Q: How can create forms reach selectable references beyond the first API
  page? → A: Use paginated remote selectors that retain selected options while
  traversing every available page with operation-approved parameters.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage Schools from the Platform Workspace (Priority: P1)

A system administrator can review the schools visible to them, narrow the list
with approved controls, move through paginated results, and create a school
tenant from the administration workspace.

**Why this priority**: Schools are the tenant roots for all school-owned
administration. Platform administrators need a reliable list/create foundation
before deeper school lifecycle or cross-school workflows can be exposed.

**Independent Test**: Sign in as an authorized system administrator, open the
school administration page, use each approved list control, navigate between
pages, and create a school with both valid and invalid input.

**Acceptance Scenarios**:

1. **Given** an authorized system administrator, **When** they open the schools
   page, **Then** they see only schools returned by the approved list operation,
   with pagination and status filtering; no school sort control is shown until
   the backend implementation honors the published sort parameter.
2. **Given** the school list has no matching records, **When** loading
   completes, **Then** the page shows a clear empty state and an authorized
   create action.
3. **Given** an authorized system administrator enters valid school
   information, **When** they submit the create form, **Then** the school is
   created once, success feedback is shown, and the list reflects the result.
4. **Given** the create request fails validation, **When** the response is
   received, **Then** matching fields show actionable messages and entered
   values remain available for correction.
5. **Given** the requester lacks school administration permission, **When**
   they navigate directly to a school administration route, **Then** protected
   data and create controls remain hidden and the documented forbidden state is
   shown.

---

### User Story 2 - Manage Tenant Users and Access Definitions (Priority: P2)

A school administrator can list and create users and roles within the active
school, and can browse the permission definitions available for role creation.

**Why this priority**: Users, roles, and permissions establish who can use every
later school workflow and what they are allowed to see or do.

**Independent Test**: Enter an active authorized school context, list users,
roles, and permissions, create a role from compatible permissions, then create a
user assigned to an available role while verifying denied and invalid states.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator with an active school context,
   **When** they open users, roles, or permissions, **Then** each page shows only
   records returned for that permitted scope.
2. **Given** a user or role list spans multiple pages, **When** the administrator
   changes page, page size, status filter, or supported sort, **Then** the list
   reloads with the approved parameters and preserves the current controls.
3. **Given** an administrator creates a role, **When** they choose active,
   school-scope permissions and submit valid information, **Then** a
   school-scoped role is created for the active school and becomes available to
   approved user creation flows.
4. **Given** an administrator creates a user, **When** they choose valid roles
   from the current permitted scope, **Then** the user is created without
   exposing or accepting direct per-user permission assignment.
5. **Given** role or user input is rejected, **When** validation errors are
   returned, **Then** field and form-level messages explain what must be
   corrected without exposing records from another school or authorization
   internals.
6. **Given** the active school is missing, inactive, mismatched, or unauthorized,
   **When** an administration request is attempted, **Then** tenant-owned
   content remains hidden and the established tenant recovery or denial state is
   shown.
7. **Given** available roles or permissions span multiple pages, **When** the
   administrator traverses lookup pages, **Then** every permitted option is
   reachable and already selected values remain selected.

---

### User Story 3 - Configure the Academic Calendar (Priority: P3)

A school administrator can list and create academic years and academic periods
for the active school, including filtering periods by an approved academic year.

**Why this priority**: Teacher, student, attendance, grading, and reporting
workflows depend on a valid academic calendar.

**Independent Test**: In one active school, create an academic year, create
ordered periods within it, filter periods by year, and verify date, sequence,
validation, empty, and tenant-denial states.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator, **When** they open academic
   years or academic periods, **Then** the page shows the active school's
   records with approved pagination and status filters.
2. **Given** the administrator selects an academic year filter on the periods
   page, **When** results load, **Then** only periods returned for that academic
   year are shown.
3. **Given** valid dates and status information, **When** an academic year is
   submitted, **Then** it is created for the active school and success feedback
   is shown.
4. **Given** a same-school academic year and valid dates and sequence, **When**
   an academic period is submitted, **Then** it is created under that year.
5. **Given** dates, sequence, status, or parent-year input is invalid, **When**
   submission fails, **Then** the form preserves input and shows the documented
   validation messages beside the affected controls.
6. **Given** a referenced academic year is unavailable or not found in the
   permitted scope, **When** period creation cannot continue, **Then** the form
   shows a safe not-found state and does not infer whether a cross-tenant record
   exists.
7. **Given** permitted academic years span multiple pages, **When** the
   administrator traverses the parent-year lookup, **Then** every permitted
   option is reachable and the selected year remains available.

---

### User Story 4 - Create and List Guardians (Priority: P4)

A school administrator can list guardians for the active school and create a
guardian record, including approved same-school student associations when
available.

**Why this priority**: Guardian contact records support student administration,
but depend on the tenant, access, and reusable CRUD foundations established by
the higher-priority stories.

**Independent Test**: List guardians in an active school, use the approved
same-school student-profile lookup, create a guardian with valid contact and
relationship data, and verify invalid, missing, cross-tenant, empty, and denied
states.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator, **When** they open guardians,
   **Then** they see a paginated list limited to the active school with the
   approved status filter.
2. **Given** no guardians match the current page or filter, **When** loading
   completes, **Then** a clear empty state and authorized create action are
   shown.
3. **Given** valid guardian information and any approved same-school student
   references, **When** the form is submitted, **Then** one guardian record is
   created and the list reflects the successful result.
4. **Given** guardian creation allows optional student associations and the
   administrator has `student_profiles.view`, **When** they search or page
   through available students, **Then** the choices come only from the approved
   student-profile list filtered to active profiles in the active school.
5. **Given** a student reference is missing, inactive, unavailable, or outside
   the permitted school, **When** creation is rejected, **Then** no partial
   result is presented and a safe validation or not-found message is shown.
6. **Given** the requester lacks guardian administration permission, **When**
   they open the route directly, **Then** guardian data and create controls
   remain hidden and the documented forbidden state is shown.

### Edge Cases

- A list request completes after the user changes route, active school, filter,
  page, or sort; stale results must not replace the current view.
- The current page becomes empty after a successful creation or filter change;
  pagination must resolve to an available page without looping requests.
- A user submits a create form more than once while the first request is still
  pending; the UI must prevent accidental duplicate submissions.
- A list or create request returns unauthorized because the session expired;
  the authentication foundation owns recovery and protected data is cleared.
- Permission state changes after a route or create control was visible; a
  backend denial remains authoritative and the denied data is not retained.
- A school switch is requested while a tenant-owned form contains unsaved data;
  the user must confirm discarding the changes before the switch proceeds, and
  the old data must never submit into the new school context.
- A user leaves a create route through browser navigation, sidebar navigation,
  list return, or another in-app destination while the form has unsaved data;
  the user must confirm discarding the changes before navigation proceeds.
- A selected role, permission, academic year, or student reference becomes
  inactive or unavailable before submission; the form must show the returned
  validation or not-found outcome and require a valid replacement.
- The service is temporarily unavailable; the current list controls or form
  input remain recoverable and the user receives a retry path.
- A list envelope contains an empty final page after records changed elsewhere;
  the UI must recover to the nearest valid page.
- A list route opens with unsupported or malformed query values; the UI must
  normalize them to approved defaults without sending undocumented parameters.
- A multi-school session has no restorable active school and no approved
  user-authorized school source; tenant-owned administration remains blocked
  and no administration list or lookup request is sent.
- A paginated role, permission, or academic-year lookup changes page after a
  value is selected; the selected option must remain visible and submittable.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No backend behavior is introduced. The feature
  consumes only list/create behavior already implemented and approved for
  schools, users, roles, permissions, academic years, academic periods, and
  guardians.
- **Frontend repository impact**: Add administration route modules, navigation
  metadata, resource pages, service and contract mappings, reusable list,
  filter, table, form, pagination, empty, loading, validation, denial, and
  not-found patterns, plus focused state and tests where state crosses
  components.
- **Specification or contract repository impact**: Add this frontend behavior
  specification and a frontend consumption contract during planning. OpenAPI
  changes are not required unless review finds a missing behavior that the UI
  cannot safely omit.
- **Delivery ownership and sequencing**: `schoolmaster-specs` approves the
  frontend behavior first. `schoolmaster-frontend` implements it against the
  already implemented backend operations. Missing contract or backend behavior
  blocks only the affected surface and must be resolved before consumption.

### API Contract Impact

- **OpenAPI update required**: No. Any missing field, filter, sort, error, or
  operation must be added and implemented before the frontend consumes it.
- **Versioned endpoints affected**: `GET` and `POST` on `/api/v1/schools`,
  `/api/v1/users`, `/api/v1/roles`, `/api/v1/academic-years`,
  `/api/v1/academic-periods`, and `/api/v1/guardians`, plus `GET
  /api/v1/permissions` and `GET /api/v1/student-profiles` for the guardian
  association lookup.
- **JSON response impact**: No changes. The frontend must map the published
  success, paginated, validation, unauthorized, forbidden, tenant-mismatch, and
  not-found envelopes without depending on undocumented fields.
- **Authentication/authorization impact**: All routes are protected. School
  administration is platform-scoped; tenant-owned administration requires an
  active permitted school context. Client visibility checks improve usability
  but do not replace backend authorization.
- **Permission matrix**:
  - School list: `schools.view`; school create: `schools.view` and
    `schools.manage`.
  - User list: `users.view`; user create: `users.view`, `users.manage`, and
    `roles.view`.
  - Role list: `roles.view`; role create: `roles.view`, `roles.manage`, and
    `permissions.view`.
  - Permission list: `permissions.view`.
  - Academic-year list: `academic_years.view`; academic-year create:
    `academic_years.view` and `academic_years.manage`.
  - Academic-period list: `academic_periods.view`; academic-period create:
    `academic_periods.view`, `academic_periods.manage`, and
    `academic_years.view`.
  - Guardian list: `guardians.view`; guardian create: `guardians.view` and
    `guardians.manage`; optional student-association lookup:
    `student_profiles.view`.
  All listed permissions are required for the corresponding frontend route or
  action. Backend authorization remains authoritative.
- **Compatibility impact**: Frontend-only and additive. Detail, update,
  activation, deactivation, deletion, restoration, bulk lifecycle, invitation,
  password, and guardian user-link workflows remain outside this feature.

### Data & Tenancy Impact

- **Tenant scoping impact**: Schools are platform-managed tenant roots. Users,
  school roles, academic years, academic periods, and guardians are displayed
  and created only within the active permitted `school_id` context. Permission
  definitions are shown only as returned for the requester and active context.
- **Cross-tenant or platform access impact**: The UI must not merge tenant-owned
  results across schools, infer hidden record existence, or treat platform
  administration as an implicit school-owned access bypass.
- **Soft delete impact**: Deleted-record browsing, restoration, and lifecycle
  controls are outside scope. Lists display only the records and statuses
  returned by the approved foundation operations.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The frontend MUST provide protected list routes for schools,
  users, roles, permissions, academic years, academic periods, and guardians.
- **FR-002**: The frontend MUST provide create workflows for schools, users,
  roles, academic years, academic periods, and guardians through dedicated
  protected create routes, and MUST keep permissions read-only.
- **FR-003**: Every route, navigation item, and create action MUST use the exact
  permission matrix in this specification and MUST be hidden when the current
  session lacks any permission required for that route or action.
- **FR-004**: A direct visit to an unauthorized administration route MUST show
  the established unauthorized or forbidden state without rendering protected
  resource data.
- **FR-005**: Tenant-owned administration routes MUST require a valid active
  school context before requesting or rendering resource data.
- **FR-005a**: When feature 017 cannot confirm an active school, tenant-owned
  administration MUST remain blocked and MUST NOT use `listSchools` or another
  unapproved source to populate school selection.
- **FR-006**: After a school switch is approved by the general unsaved-change
  route guard, the frontend MUST clear tenant-owned resource data, selections,
  pending results, and form context before the new school's data is shown.
- **FR-007**: List pages MUST use a reusable administration page structure for
  title, approved actions, filters, loading feedback, table content, empty
  state, pagination, and recoverable errors.
- **FR-008**: List controls MUST send only parameters approved for that
  operation and MUST persist the current approved filter, sort, page, and page
  size state in the URL query so browser navigation and refresh restore the
  same list view.
- **FR-008a**: List routes MUST ignore or normalize unsupported and malformed
  URL query values to approved defaults and MUST NOT forward undocumented
  parameters to the backend.
- **FR-009**: School lists MUST support status, pagination, and page-size
  controls but MUST NOT expose sorting until backend implementation honors the
  published sort parameter. User lists MUST support approved status,
  pagination, page-size, and sort controls. Role, academic year, and guardian
  lists MUST support approved status and pagination controls.
- **FR-010**: Permission lists MUST support approved pagination controls and
  MUST NOT expose create, edit, assignment-to-user, or lifecycle actions.
- **FR-011**: Academic period lists MUST support approved pagination, status,
  and academic-year filtering.
- **FR-012**: Empty states MUST distinguish between no records in the permitted
  scope and no records matching the current filters, and MUST offer only
  authorized, approved recovery actions.
- **FR-013**: Create forms MUST use only fields and selectable references
  published by the corresponding request contract.
- **FR-013a**: Role, permission, and academic-year selectors MUST provide
  server-driven page traversal using only parameters approved for their list
  operations, MUST make every permitted page reachable, and MUST retain
  selected options across page changes and tenant-safe reloads.
- **FR-014**: User creation MUST allow only approved role selection and MUST NOT
  offer direct permission assignment.
- **FR-015**: Role creation in this feature MUST always submit `scope=school`,
  MUST NOT expose an editable scope control, and MUST allow only active
  school-scope permission selections returned for the active school context.
- **FR-016**: Academic period creation MUST select a permitted academic year and
  MUST preserve the documented parent-year, date, and sequence constraints in
  its guidance and validation display.
- **FR-017**: Guardian creation MUST accept only the approved contact,
  relationship, and optional student-reference fields; optional student choices
  MUST be shown only when the session has `student_profiles.view`, MUST come
  from the approved student-profile list with `status=active` for the active
  school, and the UI MUST not present a partially successful result when any
  association is rejected.
- **FR-018**: Validation responses MUST map field errors to matching controls,
  preserve correct user input, show form-level errors when no field match
  exists, and focus or summarize errors accessibly.
- **FR-019**: While a create request is pending, the form MUST prevent duplicate
  submission and MUST keep cancel or navigation behavior predictable.
- **FR-019a**: Every create route MUST detect unsaved form changes and require
  confirmation before any browser or in-app route exit discards them; no
  confirmation is required after successful submission or when the form is
  unchanged.
- **FR-020**: After successful creation, the UI MUST show success feedback,
  clear stale form errors, return to the relevant list route with its approved
  URL query state restored, and update or reload that list without assuming an
  undocumented response field.
- **FR-021**: Unauthorized, forbidden, tenant-mismatch, inactive-context,
  not-found, validation, and temporary service failures MUST use consistent
  shared feedback patterns while preserving the distinct recovery action for
  each state.
- **FR-022**: Not-found messages MUST not reveal whether an identifier exists in
  another tenant or inaccessible scope.
- **FR-023**: Stale list or create responses MUST NOT overwrite state belonging
  to a newer route, filter, pagination, permission, session, or tenant context.
- **FR-024**: Administration pages MUST remain operable at 390px mobile, 768px
  tablet, and 1440px desktop viewport widths, including accessible table
  alternatives or responsive presentation for primary record fields and
  actions.
- **FR-025**: Shared administration controls MUST provide keyboard operation,
  visible focus, programmatic labels, announced loading and error feedback, and
  contrast consistent with the project's accessibility target.
- **FR-026**: Reusable interface text for tables, forms, empty states,
  pagination, validation, and feedback MUST be centrally managed for
  consistency and localization readiness.
- **FR-027**: The feature MUST NOT expose detail, update, activation,
  deactivation, deletion, restoration, bulk lifecycle, invitation, password,
  account lock, guardian user-link, or other lifecycle actions reserved for
  later roadmap items.
- **FR-028**: Frontend tests MUST cover contract mapping, list controls,
  pagination, URL query restoration and normalization, empty states, create
  success and validation, permission visibility, direct-route denial, tenant
  changes, unsaved-change route guards, not-found handling, duplicate
  submission prevention, paginated form lookups, blocked unresolved school
  selection, and stale-response protection.

### Key Entities *(include if feature involves data)*

- **School**: Platform-managed tenant root displayed and created through the
  system administration workspace.
- **User**: Identity displayed or created in the permitted platform or school
  scope with role assignments and status.
- **Role**: Scoped authorization grouping created from compatible permissions.
- **Permission**: Read-only capability definition available to the requester
  and used when defining roles.
- **Academic Year**: School-owned academic cycle with dates and status.
- **Academic Period**: Ordered school-owned subdivision of an academic year.
- **Guardian**: School-owned responsible contact with approved relationship,
  contact, and optional same-school student association data.
- **Student Profile Lookup**: Approved, paginated active-school source used only
  to select optional guardian associations in this feature.
- **Administration List State**: Current resource, filter, sort, page, page
  size, loading, empty, and error state for one list surface; approved filter,
  sort, page, and page-size values are represented in the URL query.
- **Administration Form State**: Current values, selectable references,
  validation messages, submission state, and tenant context for one create
  workflow.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Authorized administrators can reach every approved administration
  list and create workflow from the admin shell in no more than two navigation
  interactions.
- **SC-002**: In moderated usability verification with five representative
  administrators, each participant attempts all six create workflows once
  using valid scenario data; at least 27 of the resulting 30 first attempts
  (90%) must complete without facilitator guidance.
- **SC-003**: With the application already bootstrapped and a mocked list
  service settling within 1.5 seconds, every list surface must display its first
  usable result, empty state, or recoverable error within 2 seconds measured
  from route navigation or committed query change to stable rendered feedback.
- **SC-004**: Validation verification confirms 100% of documented field errors
  appear beside or are programmatically associated with the relevant form
  control, while entered valid values remain intact.
- **SC-005**: Authorization and tenancy verification confirms 100% of tested
  denied, mismatched, inactive, and cross-tenant scenarios render no protected
  resource data.
- **SC-006**: Pagination and filter verification confirms 100% of supported
  controls use only approved parameters and recover correctly from empty or
  invalidated pages.
- **SC-007**: Responsive and accessibility review at 390px, 768px, and 1440px
  confirms all primary list, filter, pagination, create, validation, and
  recovery actions are keyboard operable and usable.
- **SC-008**: Contract review maps 100% of frontend requests in this feature to
  the approved operation IDs with no undocumented endpoint, field, filter,
  sort, or lifecycle action.

## Assumptions

- Features `015-frontend-architecture-baseline`, `016-admin-shell-dashboard`,
  and `017-auth-session-ui` are implemented and provide the reusable
  architecture, admin shell, authenticated routing, permissions, tenant
  context, and denial-state foundations.
- Feature 017 can restore an active school only from authenticated session
  data. When it cannot do so, school selection remains blocked until a separate
  approved operation returns only schools authorized for the current user;
  this feature does not resolve that contract gap.
- Backend operations `listSchools`, `createSchool`, `listUsers`, `createUser`,
  `listRoles`, `createRole`, `listPermissions`, `listAcademicYears`,
  `createAcademicYear`, `listAcademicPeriods`, `createAcademicPeriod`,
  `listGuardians`, `createGuardian`, and `listStudentProfiles` are implemented
  and contract-compliant.
- Schools are managed through platform-scoped permissions; other resource
  pages use the active permitted school context unless the published contract
  explicitly returns a permitted platform scope.
- Search controls are not included because the current foundation list
  operations do not publish a general search parameter.
- School sorting remains hidden because the current backend implementation
  ignores the published school sort parameter and always orders by name.
- Create workflows use dedicated protected routes and must preserve accessible
  validation, list return navigation, and unsaved-state behavior.
- Detail and maintenance behavior belongs to Administration Lifecycle UI,
  roadmap item 5, even where lifecycle endpoints already exist.
