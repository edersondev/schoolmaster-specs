# Data Model: Guardian Self-Service UI

## GuardianWorkspaceContext

**Purpose**: Frontend-safe context required before any guardian self-service
screen loads tenant-owned guardian or student data.

**Fields**: `schoolId`, `guardianAccessState`, `selectedStudentProfileId`,
`academicPeriodId`, `workspaceStatus`, `defaultRoute`, `feedbackState`.

**Source**: Approved authenticated session, active school context, current-user
or permission context, approved guardian access behavior, and approved current
active academic-period context.

**Rules**:

- Guardian self-service screens require authenticated access.
- Active permitted school context is required before guardian data requests.
- Active same-school guardian-user link and active guardian record are
  required. Missing link becomes a no-guardian-link state only when approved
  session or access behavior safely identifies it.
- Current active academic period is required for guardian academic summary.
- Default route is linked students when authenticated access and active school
  context are present.
- Missing school, missing guardian link, no linked students, missing current
  period, denied target, and unavailable summary are distinct states.

## GuardianStudentListView

**Purpose**: Guardian workspace landing surface for students visible to the
authenticated guardian in the active school.

**Fields**: `items`, `pagination`, `loading`, `emptyState`, `feedbackState`,
`staleRequestKey`.

**Source**: `listGuardianStudents`.

**Rules**:

- Request uses tenant context, page, and per-page only.
- Data loads only after authenticated access and active school context are
  confirmed.
- Returned students are shown with contract-approved limited summary fields.
- Empty result is no-linked-students, not authorization denial.
- Stale results are ignored when route, selected student, active school,
  pagination, authentication, or session state changes.

## GuardianStudentSummaryView

**Purpose**: Guardian-visible linked student item used in list, selected
student header, academic summary, and contact view context.

**Fields**: `id`, `schoolId`, `registrationNumber`, `fullName`, `status`,
`enrolledAt`, `relationshipLabel`, `currentAcademicYearId`.

**Source**: `GuardianStudentSummary`.

**Rules**:

- Student appears only when returned by the approved guardian list or related
  guardian operation.
- Relationship label is guardian-visible and same-school.
- School-only notes, disciplinary data, other guardian data, and private
  internal identifiers are not represented.

## GuardianStudentDetailView

**Purpose**: Guardian-visible limited profile and enrollment summary for one
permitted student.

**Fields**: `id`, `firstName`, `lastName`, `fullName`, `dateOfBirth`,
`registrationNumber`, `relationshipLabel`, `status`, `enrollmentSummary`,
`feedbackState`.

**Source**: `getGuardianStudent`.

**Rules**:

- Request uses tenant context and selected `studentProfileId` only.
- Target-specific missing, unassociated, inactive, transferred, deleted, and
  cross-tenant students map to the same not-found state.
- Detail does not expose detailed academic rows, other guardian records,
  school-only notes, report output, private teacher content, or fields not in
  contract.

## GuardianAcademicSummaryView

**Purpose**: Current-active-period academic summary for one linked student.

**Fields**: `student`, `academicPeriodId`, `gradeSummary`,
`attendanceSummary`, `learningSets`, `loading`, `feedbackState`,
`staleRequestKey`.

**Source**: `getGuardianStudentAcademics`.

**Rules**:

- Request uses tenant context, selected `studentProfileId`, and current active
  `academic_period_id`.
- Manual period switching and alternate period sources are outside this slice.
- No request is sent without current active same-school academic period.
- Empty or missing summary values show unavailable-summary or true empty state,
  not target-denial.
- Detailed grade rows, attendance rows, correction history, teacher content,
  questionnaire answers, report output, ranking, and custom reporting are not
  represented.

## GuardianGradeSummaryView

**Purpose**: Guardian-visible current grade summary for one linked student and
academic period.

**Fields**: `status`, `average`, `scale`, `lastUpdatedAt`.

**Source**: `GuardianGradeSummary`.

**Rules**:

- Fields display only as returned by contract.
- Null values render as safe missing or unavailable values.
- No client-side grade calculations beyond display formatting.
- Private correction details and teacher-only metadata are excluded.

## GuardianAttendanceSummaryView

**Purpose**: Guardian-visible attendance summary for one linked student and
academic period.

**Fields**: `status`, `totalAbsences`, `totalTardies`, `attendanceRate`,
`lastUpdatedAt`.

**Source**: `GuardianAttendanceSummary`.

**Rules**:

- Fields display only as returned by contract.
- Null attendance rate renders as safe missing or unavailable value.
- No detailed attendance rows, correction history, or actor metadata appear.

## GuardianLearningSetSummaryView

**Purpose**: Limited guardian-facing learning-set progress or status summary.

**Fields**: `learningSetId`, `title`, `status`, `progressPercent`,
`lastActivityAt`.

**Source**: `GuardianLearningSetSummary`.

**Rules**:

- Summary displays only title, status, progress, and last activity values
  approved by contract.
- No teacher content download, questionnaire answer key, student submission, or
  private teacher note appears.

## GuardianContactView

**Purpose**: Contact view for authenticated guardian contact values,
relationship label, and selected student's primary school-approved contact
details.

**Fields**: `student`, `guardianContact`, `relationshipLabel`,
`studentPrimaryContact`, `loading`, `feedbackState`, `staleRequestKey`.

**Source**: `getGuardianStudentContacts`.

**Rules**:

- Request uses tenant context and selected `studentProfileId` only.
- Contact fields are display-only.
- Missing optional contact values show safe missing-value state.
- Other guardian records, non-primary student contacts, restricted emergency
  details, custody notes, legal-document details, school-only notes, and
  unapproved contact fields are excluded.

## GuardianFeedbackState

**Purpose**: Canonical guardian UI state used across list, detail, academic,
contact, and workspace surfaces.

**Fields**: `kind`, `messageKey`, `recoverable`, `safeDetails`.

**States**:

```text
idle
loading
empty
unauthorized
forbidden
tenant_mismatch
inactive_school
no_active_school
no_guardian_link
no_linked_students
no_academic_period
unavailable_summary
validation
not_found
unsupported_page_size
temporary_unavailable
stale_response
```

**Rules**:

- `safeDetails` may include safe operation names, field-level validation labels,
  current route name, or safe correlation/request ID only.
- Unassociated student identifiers, other guardian data, non-primary contacts,
  school-only notes, correction details, teacher-private data, report data,
  token values, role internals, and cross-tenant details are excluded.
