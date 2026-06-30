# Quickstart: Student Enrollment and Classroom Roster UI

## Prerequisites

- Feature 016 System Administrator Shell and Dashboard Foundation is
  implemented.
- Feature 017 Authentication and Session Foundation UI is implemented.
- Feature 018 Administration Foundation UI is implemented.
- Feature 020 Administration Lifecycle UI is implemented.
- Feature 021 Account Lifecycle Workflows UI is implemented.
- Backend student enrollment operations from
  `specs/006-backend-student-enrollment/` are deployed and
  contract-compliant.
- Backend classroom roster operations from
  `specs/009-classroom-roster-foundation/` are deployed and
  contract-compliant.
- Implementation confirms approved permission codes or session capability
  flags before student, roster, membership, transfer, and teacher-assignment
  action visibility is enabled.

## Contract Review

Before frontend implementation:

1. Confirm these student profile operations exist in `api/openapi.yaml`:
   - `listStudentProfiles`
   - `createStudentProfile`
   - `getStudentProfile`
   - `updateStudentProfileStatus`
   - `transferStudentProfile`
2. Confirm no approved `updateStudentProfile` operation exists; keep general
   profile edit blocked unless OpenAPI adds one before implementation.
3. Confirm these class-section operations exist:
   - `listClassSections`
   - `createClassSection`
   - `getClassSection`
   - `updateClassSection`
   - `updateClassSectionStatus`
4. Confirm these roster membership operations exist:
   - `listClassSectionMemberships`
   - `batchAddClassSectionMemberships`
   - `batchEndClassSectionMemberships`
5. Confirm these teacher assignment operations exist:
   - `listTeacherAssignments`
   - `createTeacherAssignment`
   - `getTeacherAssignment`
   - `updateTeacherAssignmentStatus`
6. Confirm the approved academic-period read operation exists:
   - `listAcademicPeriods`
7. Confirm the approved same-school user list operation exists for teacher
   assignment candidate selection:
   - `listUsers`
8. Confirm `listUsers` supports only documented tenant context, page, per-page,
   status, and sort parameters for this slice; do not add undocumented role or
   search filters for teacher candidates.
9. Confirm class-section and teacher-assignment lists support
   `academicPeriodId` and `status` filters only.
10. Confirm `listTeacherAssignments` has no `classSectionId` filter and class
   section detail has no assignment include; keep section-scoped assignment
   lists blocked unless OpenAPI adds one.
11. Confirm roster membership batch add and end accept 1 to 100 requested
   changes and reject invalid batches all-or-nothing.
12. Confirm validation, unauthorized, forbidden, tenant-mismatch,
   inactive-school, inactive-record, not-found, conflict, unsupported filter,
   unsupported sort, unsupported page-size, and temporary-unavailable envelopes
   are documented for consumed actions.

## Manual Scenario Review

### Student Profiles

- Sign in as an authorized school administrator.
- Select or confirm active school context.
- Open student profile list.
- Verify only same-school students appear.
- Verify pagination, status filter, search, sort, loading, empty, and denied
  states.
- Create a student with valid required fields.
- Submit invalid required fields, duplicate same-school registration number,
  invalid guardian association, and unsupported field attempts.
- Open student detail.
- Verify active, inactive, transferred, deleted or unavailable, and
  historical-only states are visually distinct.
- Verify general profile edit is hidden or blocked.

### Enrollment Status and Transfer

- Open an active same-school student profile.
- Submit approved inactive or active status transition with required effective
  date and reason.
- Verify status, effective date, and history cues update.
- Attempt invalid lifecycle transition and verify conflict or validation
  feedback.
- Start transfer with required effective date and reason.
- Verify unauthorized, inactive, missing, or incompatible destination feedback.
- Verify success marks source profile transferred and preserves historical
  source-school visibility.
- Verify source-school academic records, guardian links, private content, and
  report outputs are not exposed to destination context.

### Class Sections

- Open class-section list.
- Verify list defaults to current active academic period.
- Change selected academic period and verify route/query persistence across
  refresh and back navigation.
- Verify no-current-active-period state blocks scoped loading until a period is
  selected or created elsewhere.
- Create a class section with approved code, name, and metadata fields.
- Update approved metadata fields.
- Attempt duplicate code, unsupported metadata shape, inactive creation, and
  inactive reactivation.
- Inactivate eligible class section with required reason.
- Attempt inactivation while active memberships or teacher assignments remain.

### Roster Memberships

- Open class-section detail.
- List active and ended memberships.
- Add up to 100 eligible active same-school students in one batch.
- Attempt oversized selection above 100 and verify submission is blocked or
  oversized-batch feedback appears.
- Include inactive, transferred, cross-school, duplicate, overlapping, deleted,
  or not-enrolled students and verify all-or-nothing rejection.
- End up to 100 active memberships with required reason and effective end date.
- Attempt invalid dates outside selected academic period, future dates, or end
  date before start date.
- Verify no partial success appears for rejected batches.

### Teacher Assignments

- Open the academic-period teacher-assignment administration list.
- Verify teacher-facing own-assignment routes are absent.
- Verify class-section detail does not scan period-wide assignment pages to
  infer section-scoped assignments.
- Verify teacher candidates come only from approved same-school active user data
  or a known teacher user ID, without undocumented role/search/autocomplete
  requests.
- Assign an eligible same-school teacher with active teacher-compatible role
  coverage on the effective start date.
- Attempt inactive, deleted, cross-school, duplicate, incompatible-period, or
  missing-role assignments.
- Deactivate an active assignment with required lifecycle information.
- Verify forbidden actors see hidden or disabled management controls and safe
  denial feedback.

## Automated Verification Expectations

Run in `schoolmaster-frontend` after implementation:

```bash
npm run test:unit
```

Focused Vitest coverage should include:

- student profile contract mappers and service calls
- class-section contract mappers and service calls
- roster membership batch add/end mappers and service calls
- teacher-assignment contract mappers and service calls
- academic-period default and route/query selected-period restoration
- student list/create/detail composables and pages
- student status and transfer composables and dialogs
- class-section list/create/detail composables and pages
- roster membership batch selection, 100-request cap, and all-or-nothing
  feedback
- teacher assignment admin-only visibility, create, and deactivate flows
- blocked section-scoped teacher assignment list behavior
- blocked general student profile edit state
- deferred teacher own-assignment route absence
- stale response protection
- validation, forbidden, tenant-mismatch, inactive-school, inactive-record,
  not-found, conflict, unsupported filter, unsupported sort, unsupported
  page-size, oversized-batch, and temporary-unavailable mapping
- no-sensitive-data diagnostics

Run build checks if available:

```bash
npm run build
```

Run OpenAPI validation only if student enrollment or roster contracts change:

```bash
npx @redocly/cli lint api/openapi.yaml
```

## Acceptance Evidence

Record in implementation PR:

- Operation ID to UI surface mapping.
- Permission code or session capability source used for each action surface.
- Evidence that `listAcademicPeriods` is the approved source for current-period
  default and period selector state.
- Evidence that teacher assignment candidate selection uses only approved
  same-school active user data or a known teacher user ID, without undocumented
  role/search/autocomplete requests.
- Evidence that general student profile edit remains blocked unless
  `updateStudentProfile` is approved.
- Evidence that teacher-facing own-assignment routes remain deferred.
- Evidence that class-section detail does not infer section-scoped teacher
  assignments from paginated period-wide assignment lists.
- Evidence that roster and assignment lists default to current active academic
  period and restore explicit route/query selection.
- Evidence that roster membership batch add/end enforce the 100-request cap
  and all-or-nothing feedback.
- Evidence that stale responses do not overwrite route, filter, pagination,
  active-school, selected-period, target, or selection changes.
- Evidence that student, guardian, token, permission, role, reason, full
  payload, private academic, and cross-tenant details are absent from
  diagnostics.
- Responsive and keyboard review for 390px, 768px, and 1440px.
