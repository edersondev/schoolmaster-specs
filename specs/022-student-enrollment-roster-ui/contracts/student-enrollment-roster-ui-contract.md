# Student Enrollment and Classroom Roster UI Contract

This contract defines what `schoolmaster-frontend` may implement for Student
Enrollment and Classroom Roster UI.

## Scope Contract

This feature approves:

- School administrator student profile list, create, and detail views.
- Student enrollment status display and approved status update actions.
- Student transfer dialog and transfer result display.
- Class-section or roster list, create, detail, metadata update, and status
  actions.
- Current active academic-period default for roster and assignment lists, with
  selected period preserved in route/query state.
- Roster membership list and all-or-nothing batch add/end UI capped at 100
  requested changes.
- Admin teacher-assignment list, detail, create, and deactivate UI scoped by
  academic period. Class-section detail may link to assignment workflows or
  launch create with known class-section context, but it must not display a
  section-scoped assignment list unless OpenAPI adds a documented section
  filter or include.
- Safe loading, empty, validation, unauthorized, forbidden, tenant-mismatch,
  inactive-school, inactive-record, not-found, conflict, unsupported filter,
  unsupported sort, unsupported page-size, oversized-batch, and
  temporary-unavailable feedback states.
- Route-local sensitive state and service-isolated API consumption.
- No-sensitive-data diagnostics verification.

This feature does not approve:

- General student profile edit without an approved `updateStudentProfile`
  operation.
- Teacher-facing own-assignment routes or Teacher Workflow Workspace screens.
- Guardian self-service, student self-service, reporting workspace, teacher
  correction workflows, bulk import, billing, messaging, notification-center,
  permanent purge, restore, merge, anonymization, or undocumented behavior.
- Separate Course, Classroom, Section, or Group lifecycle resources.
- Client-side tenant inference or platform override of school-owned records.

## OpenAPI Consumption Contract

Frontend services may consume only these approved operations for this slice:

| Operation ID | Method and Path | UI use |
|--------------|-----------------|--------|
| `listAcademicPeriods` | `GET /api/v1/academic-periods` | Resolve current active academic period and populate the period selector for roster and assignment lists. |
| `listStudentProfiles` | `GET /api/v1/student-profiles` | Student list with approved pagination, status/search/sort controls, loading, empty, and denied states. |
| `createStudentProfile` | `POST /api/v1/student-profiles` | Create same-school student profile with optional approved guardian associations. |
| `getStudentProfile` | `GET /api/v1/student-profiles/{studentProfileId}` | Retrieve same-school student detail, guardian summary, and enrollment history. |
| `updateStudentProfileStatus` | `PATCH /api/v1/student-profiles/{studentProfileId}/status` | Apply approved non-transfer lifecycle status change. |
| `transferStudentProfile` | `POST /api/v1/student-profiles/{studentProfileId}/transfer` | Record source-school transfer behavior without exposing source history across tenants. |
| `listClassSections` | `GET /api/v1/class-sections` | List class sections or rosters for active school and selected academic period. |
| `createClassSection` | `POST /api/v1/class-sections` | Create active class section or roster. |
| `getClassSection` | `GET /api/v1/class-sections/{classSectionId}` | Retrieve class-section or roster detail. |
| `updateClassSection` | `PATCH /api/v1/class-sections/{classSectionId}` | Update approved class-section metadata only. |
| `updateClassSectionStatus` | `PATCH /api/v1/class-sections/{classSectionId}/status` | Inactivate eligible class section or roster. |
| `listClassSectionMemberships` | `GET /api/v1/class-sections/{classSectionId}/memberships` | List active and ended memberships for a class section. |
| `batchAddClassSectionMemberships` | `POST /api/v1/class-sections/{classSectionId}/memberships` | Add up to 100 eligible student memberships all-or-nothing. |
| `batchEndClassSectionMemberships` | `PATCH /api/v1/class-sections/{classSectionId}/memberships` | End up to 100 memberships all-or-nothing with required reason and effective date. |
| `listTeacherAssignments` | `GET /api/v1/teacher-assignments` | Admin assignment list for selected academic period and status. |
| `createTeacherAssignment` | `POST /api/v1/teacher-assignments` | Assign eligible teacher to active class section or roster. |
| `getTeacherAssignment` | `GET /api/v1/teacher-assignments/{teacherAssignmentId}` | Retrieve admin-visible assignment detail where needed. |
| `updateTeacherAssignmentStatus` | `PATCH /api/v1/teacher-assignments/{teacherAssignmentId}/status` | Deactivate eligible teacher assignment with documented lifecycle information. |

Frontend implementation must not add route aliases, request fields, response
fields, filters, sort values, include expansion, page-size behavior, lifecycle
actions, batch modes, status values, permission names, or capability names that
are not documented in OpenAPI or the approved session contract.

## Request Contract

### Academic Period List

`listAcademicPeriods` may send only documented tenant context, page, per-page,
status, and academic-year filters. Feature 022 uses it only to identify the
current active period and populate the period selector for roster and teacher
assignment lists. It must not create, update, activate, deactivate, restore, or
bulk-change academic periods.

### Student Profile List

`listStudentProfiles` may send only documented tenant context, page, per-page,
status, search, and sort parameters.

Unsupported filters, unsupported sort values, unsupported page sizes, or
include expansion must not be generated by UI controls.

### Student Profile Create

`createStudentProfile` may send only fields from
`StudentProfileCreateRequest`:

- `user_id`
- `registration_number`
- `first_name`
- `last_name`
- `date_of_birth`
- `contact_email`
- `contact_phone`
- `current_academic_year_id`
- `status`
- `enrolled_at`
- `guardian_associations`

The UI must not submit undocumented fields or tenant ownership fields.

### Student Profile Detail

`getStudentProfile` sends the approved `studentProfileId` path parameter and
tenant context only. The UI must not call a general update operation unless
OpenAPI adds one before implementation.

### Student Status Update

`updateStudentProfileStatus` sends:

- `status`: `active` or `inactive`
- `effective_at`
- `reason`

### Student Transfer

`transferStudentProfile` sends:

- `effective_at`
- `reason`
- `destination_school_id` only when selected and approved
- `destination_student_profile_id` only when selected and approved

The UI must not display or submit source-school grades, attendance, learning
sets, private content, guardian links, report outputs, or hidden tenant data.

### Class-Section List

`listClassSections` may send only tenant context, page, per-page,
`academicPeriodId`, and `status`. The list defaults to the current active
academic period and preserves explicit period selection in route/query state.

### Class-Section Create and Update

`createClassSection` and `updateClassSection` may send only approved
class-section request fields:

- `academic_period_id` where required by create
- `code`
- `name`
- `course`
- `classroom`
- `section`
- `group`

Each metadata block may include optional `code` and `name` only. Status
changes use `updateClassSectionStatus`.

### Class-Section Status

`updateClassSectionStatus` sends only documented status, effective date, and
reason fields. The UI must show conflict feedback when active memberships or
teacher assignments block inactivation.

### Roster Membership List

`listClassSectionMemberships` may send only tenant context, class-section ID,
page, per-page, `academicPeriodId`, and `status`.

### Roster Membership Batch Add

`batchAddClassSectionMemberships` sends:

- `academic_period_id`
- `effective_start_date`
- `student_profile_ids`, 1 to 100 unique UUID values

The UI must prevent or safely map oversized selections and must not show
partial success when any requested student is invalid.

### Roster Membership Batch End

`batchEndClassSectionMemberships` sends:

- `effective_end_date`
- `reason`
- `roster_membership_ids`, 1 to 100 unique UUID values

The UI must prevent or safely map oversized selections and must not show
partial success when any requested membership is invalid.

### Teacher Assignment List

`listTeacherAssignments` may send only tenant context, page, per-page,
`academicPeriodId`, and `status`. Feature 022 uses this for admin-visible
academic-period assignment screens only. It must not be used to populate a
single class-section detail assignment panel by scanning pages, because the
approved operation does not expose a `classSectionId` filter or class-section
assignment include.

### Teacher Assignment Create

`createTeacherAssignment` sends:

- `class_section_id`
- `teacher_user_id`
- `academic_period_id`
- `effective_start_date`

### Teacher Assignment Status

`updateTeacherAssignmentStatus` sends only approved status update fields for
assignment deactivation. Teacher eligibility and active role coverage remain
backend-authoritative.

## Response and Feedback Contract

Frontend services and composables must normalize documented success and error
envelopes into these UI states where applicable:

- `success`
- `loading`
- `empty`
- `validation`
- `unauthorized`
- `forbidden`
- `tenant-mismatch`
- `inactive-school`
- `inactive-record`
- `not-found`
- `conflict`
- `unsupported-filter`
- `unsupported-sort`
- `unsupported-page-size`
- `oversized-batch`
- `temporary-unavailable`

Conflict feedback must cover duplicate student identifiers, invalid guardian
associations, invalid lifecycle transitions, unauthorized transfer
destination, duplicate class-section code, active-dependency roster
inactivation conflicts, overlapping memberships, invalid batch membership
requests, duplicate teacher assignments, and stale state without exposing
hidden tenant, role, permission, or protected student details.

## Tenant and Authorization Contract

- All screens require authenticated active user state.
- School-owned records require active permitted school context.
- Platform scope does not grant implicit student, roster, membership,
  transfer, or teacher-assignment authority.
- Client-side permission checks are visibility aids only.
- Frontend action visibility remains blocked until implementation confirms
  approved permission codes or session capability flags.
- Backend authorization remains authoritative when client state is stale.

## State Boundary Contract

Frontend route-local state may own:

- list filters and pagination
- selected academic-period route/query state
- current validation errors
- form draft values for the current route
- pending flags
- selected membership IDs or student IDs for current batch actions
- current dialog state
- current safe feedback state

Frontend state must not persist:

- student private details beyond current route needs
- guardian private details beyond approved response display
- tokens
- full request payloads
- lifecycle reasons after submission
- role internals
- permission payloads
- cross-tenant details
- private academic records

Stale responses after route, filter, pagination, session, active school,
academic period, target student, class section, membership selection, or
permission changes must be ignored.

## Accessibility, Localization, and Observability Contract

- Student, roster, membership, transfer, and teacher-assignment screens must
  target WCAG 2.1 AA at 390px, 768px, and 1440px.
- Reusable text must be centralized through Vue I18n.
- Element Plus component tags must use PascalCase.
- Batch selection controls must expose selected count, max count, and disabled
  submission state to assistive technology.
- Frontend diagnostics may include safe operation identifiers and request
  identifiers, but must not log student details, guardian details, tokens,
  permission payloads, role internals, lifecycle reasons, full request payloads,
  source-school private data, or unauthorized cross-tenant details.
