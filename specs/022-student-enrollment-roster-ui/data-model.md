# Data Model: Student Enrollment and Classroom Roster UI

## AcademicPeriodScopeView

**Purpose**: Frontend-safe period context used by roster and
teacher-assignment lists.

**Fields**: `academicPeriodId`, `label`, `status`, `isCurrent`, `routeQuery`.

**Source**: Existing academic period administration contracts and approved
class-section or teacher-assignment filters.

**Rules**:

- Roster and teacher-assignment lists default to the current active academic
  period.
- Explicit selection is preserved in route/query state.
- If no current active period exists, scoped roster and assignment loading is
  blocked until an approved period is selected or created elsewhere.
- The frontend must not infer a period outside authenticated school-owned data.

## StudentProfileListView

**Purpose**: Paginated school-owned student collection for administrator
search, status scanning, and navigation to detail.

**Fields**: `id`, `schoolId`, `userId`, `registrationNumber`, `firstName`,
`lastName`, `fullName`, `status`, `enrolledAt`, `statusEffectiveAt`.

**Source**: `StudentProfileSummary` from `listStudentProfiles`.

**Rules**:

- List entries are visible only within the active permitted school context.
- Supported list controls are OpenAPI-documented page, per-page, status,
  search, and sort parameters.
- General edit controls are not shown from the list.
- Missing, forbidden, tenant-mismatch, inactive-school, unsupported filter, and
  unsupported sort outcomes map to safe feedback.

## StudentProfileCreateDraft

**Purpose**: Route-local administrator form state for approved same-school
student profile creation.

**Fields**: `userId`, `registrationNumber`, `firstName`, `lastName`,
`dateOfBirth`, `contactEmail`, `contactPhone`, `currentAcademicYearId`,
`status`, `enrolledAt`, `guardianAssociations`, `validationErrors`, `pending`,
`feedbackState`.

**Source**: `StudentProfileCreateRequest` submitted to
`createStudentProfile`.

**Rules**:

- Required fields are `registrationNumber`, `firstName`, `lastName`, and
  `enrolledAt`.
- Initial `status` may be `active` or `inactive` where OpenAPI permits; default
  display follows backend default.
- Optional guardian associations must contain same-school guardians and no
  duplicate `guardian_id` values.
- Additional request fields are not submitted.
- Draft state is not persisted after navigation.

## StudentProfileDetailView

**Purpose**: Frontend-safe student detail for profile review, lifecycle status,
guardian-compatible summary, and history cues.

**Fields**: Student list fields plus `dateOfBirth`, `contactEmail`,
`contactPhone`, `currentAcademicYearId`, `guardianAssociations`,
`enrollmentHistory`.

**Source**: `StudentProfile` from `getStudentProfile`.

**Rules**:

- Detail retrieval must not reveal cross-school existence.
- General student profile edit is hidden or blocked unless OpenAPI adds an
  approved update operation.
- Historical-only, inactive, transferred, deleted or unavailable, and active
  eligibility states must be visually distinct.
- Guardian and history details are shown only when present in approved
  response data.

## StudentLifecycleActionDraft

**Purpose**: Route-local state for approved non-transfer status changes.

**Fields**: `studentProfileId`, `status`, `effectiveAt`, `reason`, `pending`,
`validationErrors`, `feedbackState`.

**Source**: `StudentProfileStatusUpdateRequest` submitted to
`updateStudentProfileStatus`.

**Rules**:

- `status` is `active` or `inactive`.
- `effectiveAt` and `reason` are required.
- Backend remains authoritative for allowed transitions.
- Success updates detail state and history cues; stale responses are ignored.

**States**:

```text
active -> inactive
inactive -> active
active -> conflict
inactive -> conflict
```

## StudentTransferDraft

**Purpose**: Route-local state for recording approved source-school transfer
behavior.

**Fields**: `studentProfileId`, `effectiveAt`, `reason`,
`destinationSchoolId`, `destinationStudentProfileId`, `pending`,
`validationErrors`, `feedbackState`.

**Source**: `StudentProfileTransferRequest` submitted to
`transferStudentProfile`.

**Rules**:

- `effectiveAt` and `reason` are required.
- Destination fields are optional and only submitted when approved and selected.
- Transfer must not copy or expose source-school academic records, private
  content, guardian links, report outputs, or hidden tenant data.
- Success marks the source profile as transferred and updates active
  eligibility cues.

**States**:

```text
active -> transferred
active -> validation
active -> forbidden
active -> conflict
transferred -> conflict
```

## ClassSectionListView

**Purpose**: Paginated class-section or roster collection for the selected
academic period.

**Fields**: `id`, `schoolId`, `academicPeriodId`, `code`, `name`, `course`,
`classroom`, `section`, `group`, `status`, `inactiveReason`,
`inactiveEffectiveAt`.

**Source**: `ClassSection` from `listClassSections`.

**Rules**:

- List defaults to current active academic period and persists selected period
  in route/query state.
- Supported filters are `academicPeriodId` and `status` only.
- Include expansion, sort behavior, and other filters are not used.
- `code` is unique per school and academic period; names may repeat.

## ClassSectionDraft

**Purpose**: Route-local form state for approved class-section create and
metadata update.

**Fields**: `academicPeriodId`, `code`, `name`, `course`, `classroom`,
`section`, `group`, `validationErrors`, `pending`, `feedbackState`.

**Source**: `ClassSectionCreateRequest` and `ClassSectionUpdateRequest`
submitted to `createClassSection` or `updateClassSection`.

**Rules**:

- Create records start as `active`.
- Inactive creation and inactive reactivation are not supported.
- Metadata blocks may contain optional `code` and `name` only.
- Status changes use `updateClassSectionStatus`, not metadata update.

**States**:

```text
none -> active
active -> inactive
inactive -> terminal-v1
```

## RosterMembershipListView

**Purpose**: Paginated roster membership history for one class section.

**Fields**: `id`, `schoolId`, `classSectionId`, `studentProfileId`,
`academicPeriodId`, `status`, `effectiveStartDate`, `effectiveEndDate`,
`endReason`, `createdByUserId`, `endedByUserId`.

**Source**: `RosterMembership` from `listClassSectionMemberships`.

**Rules**:

- Supported filters are page, per-page, `academicPeriodId`, and `status`.
- Status is `active` or `ended`.
- Ended memberships remain visible for approved history review.
- Cross-school student details must not be inferred from missing or forbidden
  membership responses.

## RosterMembershipBatchDraft

**Purpose**: Route-local state for all-or-nothing batch membership add and end.

**Fields**: `classSectionId`, `academicPeriodId`, `effectiveStartDate`,
`effectiveEndDate`, `reason`, `studentProfileIds`, `rosterMembershipIds`,
`selectionCount`, `pending`, `validationErrors`, `feedbackState`.

**Source**: `RosterMembershipBatchAddRequest` and
`RosterMembershipBatchEndRequest` submitted to
`batchAddClassSectionMemberships` or `batchEndClassSectionMemberships`.

**Rules**:

- Batch add requires `academicPeriodId`, `effectiveStartDate`, and 1 to 100
  unique `studentProfileIds`.
- Batch end requires `effectiveEndDate`, `reason`, and 1 to 100 unique
  `rosterMembershipIds`.
- Any invalid requested member rejects the whole request; the UI must not show
  partial success.
- Oversized selections are blocked before submission or mapped to documented
  oversized-batch feedback.

**States**:

```text
selecting -> confirming -> submitting -> succeeded
selecting -> confirming -> submitting -> validation
selecting -> confirming -> submitting -> conflict
selecting -> oversized-blocked
```

## TeacherAssignmentListView

**Purpose**: Admin-visible teacher assignment collection for selected academic
period and status.

**Fields**: `id`, `schoolId`, `classSectionId`, `teacherUserId`,
`academicPeriodId`, `status`, `effectiveStartDate`, `effectiveEndDate`,
`deactivationReason`.

**Source**: `TeacherAssignment` from `listTeacherAssignments` and
`getTeacherAssignment`.

**Rules**:

- Feature 022 exposes admin assignment management only.
- Teacher-facing own-assignment screens are deferred.
- Supported filters are page, per-page, `academicPeriodId`, and `status`.
- Status is `active` or `inactive`.

## TeacherAssignmentDraft

**Purpose**: Route-local state for assigning or deactivating eligible teachers.

**Fields**: `classSectionId`, `teacherUserId`, `academicPeriodId`,
`effectiveStartDate`, `effectiveEndDate`, `deactivationReason`, `pending`,
`validationErrors`, `feedbackState`.

**Source**: `TeacherAssignmentCreateRequest` and
`TeacherAssignmentStatusUpdateRequest` submitted to `createTeacherAssignment`
or `updateTeacherAssignmentStatus`.

**Rules**:

- Creation requires `classSectionId`, `teacherUserId`, `academicPeriodId`, and
  `effectiveStartDate`.
- Deactivation uses the dedicated status operation and requires documented
  lifecycle information.
- Duplicate active teacher assignments map to conflict feedback.
- Teacher eligibility and role coverage remain backend-authoritative.

**States**:

```text
none -> active
active -> inactive
active -> conflict
inactive -> conflict
```

## AdminSafeFeedbackState

**Purpose**: Canonical UI state used by student enrollment and classroom roster
pages.

**Values**: `loading`, `empty`, `validation`, `unauthorized`, `forbidden`,
`tenant-mismatch`, `inactive-school`, `inactive-record`, `not-found`,
`conflict`, `unsupported-filter`, `unsupported-sort`,
`unsupported-page-size`, `oversized-batch`, `temporary-unavailable`,
`success`.

**Rules**:

- Feedback must not reveal cross-tenant existence, protected student details,
  guardian details, teacher role internals, permission payloads, full request
  payloads, lifecycle reasons, tokens, or private academic records.
- Field-level validation may identify invalid local input fields.
- Diagnostics may include safe operation identifiers and request identifiers
  only.
