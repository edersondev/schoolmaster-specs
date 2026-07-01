# Data Model: Student Self-Service UI

## StudentWorkspaceContext

**Purpose**: Frontend-safe context required before any student self-service
screen loads tenant-owned student data.

**Fields**: `schoolId`, `studentProfileId`, `academicPeriodId`,
`workspaceStatus`, `defaultRoute`, `feedbackState`.

**Source**: Approved authenticated session, active school context, current-user
or permission context, and approved session metadata for current active
academic period.

**Rules**:

- Student self-service screens require authenticated access.
- Active permitted school context is required before student data requests.
- Active same-school linked student profile is required before student data
  requests.
- Current active academic period is required for Assigned Learning Sets and
  period-scoped academic overview.
- Default route is Assigned Learning Sets when all required context is present.
- Missing school, missing student profile, and missing current period are
  distinct states.

## AssignedLearningSetsLanding

**Purpose**: Default student workspace landing surface for current active
academic period assignments.

**Fields**: `academicPeriodId`, `items`, `pagination`, `loading`,
`emptyState`, `feedbackState`, `staleRequestKey`.

**Source**: `listStudentLearningSets`.

**Rules**:

- Request uses tenant context, required `academic_period_id`, page, and
  per-page only.
- Data loads only after active school, active student profile, and current
  active academic period are confirmed.
- Returned learning sets are shown in contract order.
- Empty result is a true empty state, not authorization denial.
- Manual period switching is outside this slice.
- Stale results are ignored when route, pagination, active school, current
  active period, authentication, or session state changes.

## StudentLearningSetView

**Purpose**: Student-visible assigned learning-set card and loaded-list-backed
detail source.

**Fields**: `id`, `academicPeriodId`, `title`, `status`, `publishedAt`,
`entries`, `feedbackState`.

**Source**: `StudentLearningSetTimelineItem` from `listStudentLearningSets`.

**Rules**:

- `status` is student-visible `published`.
- Detail surfaces use loaded timeline data only.
- A stale direct link without loaded matching record shows not-found or
  contract-unavailable state.
- No standalone student learning-set detail request is sent in this slice.

## StudentLearningSetEntryView

**Purpose**: Ordered student-visible entry inside an assigned learning set.

**Fields**: `entryType`, `entryReferenceId`, `sequence`, `title`,
`contentItem`, `entryState`.

**Source**: `StudentLearningSetEntry`.

**Rules**:

- `entryType` is `content_item` or `questionnaire`.
- Entries are displayed in returned `sequence`.
- Content entries expose only student content metadata returned by contract.
- Questionnaire entries are read-only; response submit/review UI is outside
  this slice.

## StudentContentMetadataView

**Purpose**: Student-visible content metadata and download availability.

**Fields**: `id`, `title`, `contentType`, `fileSizeBytes`, `scanStatus`,
`downloadAvailable`, `feedbackState`.

**Source**: `TeacherContentStudentMetadata` inside returned learning-set
entries.

**Rules**:

- Download action is enabled only when `downloadAvailable` is true.
- Pending-scan, failed-scan, inactive, missing, unassigned, cross-school,
  unauthorized, or otherwise unavailable content shows unavailable-content or
  not-found feedback.
- Private file paths, storage keys, and scan internals are never represented in
  frontend state or diagnostics.

## StudentContentDownloadAction

**Purpose**: Transient state for one authorized student content download.

**Fields**: `contentItemId`, `pending`, `resultState`, `errorState`.

**Source**: `downloadStudentTeacherContent`.

**Rules**:

- Request uses tenant context and returned `contentItemId` only.
- Response is binary and not modeled as persisted JSON state.
- Download failures map to safe unauthorized, tenant-mismatch, validation,
  unavailable-content, or not-found feedback.
- No private URL, file path, or storage key is persisted.

## StudentGradeListView

**Purpose**: Current-period list and loaded-record detail source for the
authenticated student's own grades.

**Fields**: `items`, `pagination`, `academicPeriodId`, `loading`,
`emptyState`, `feedbackState`, `staleRequestKey`.

**Source**: `listStudentGrades`.

**Rules**:

- Request uses tenant context, page, per-page, and required current active
  `academic_period_id`.
- Manual period switching and unscoped grade lists are outside this slice.
- Empty result is a true empty state.
- Detail surfaces use loaded list records only.
- No correction, import, restore, authoring, or other-student controls exist.

## StudentGradeRecordView

**Purpose**: Student-visible grade record for list and read-only detail.

**Fields**: `id`, `schoolId`, `studentProfileId`, `academicPeriodId`,
`recordedByUserId`, `gradeValue`, `gradeLabel`, `status`, `recordedAt`.

**Source**: `StudentGradeRecord`.

**Rules**:

- `gradeValue` is 0 to 100 and reflects current student-visible value.
- `gradeLabel` may be null.
- Record belongs to the authenticated student's own active same-school profile.
- No private correction notes or teacher-only metadata are displayed.

## StudentAttendanceListView

**Purpose**: Current-period list and loaded-record detail source for the
authenticated student's own attendance.

**Fields**: `items`, `pagination`, `academicPeriodId`, `loading`,
`emptyState`, `feedbackState`, `staleRequestKey`.

**Source**: `listStudentAttendance`.

**Rules**:

- Request uses tenant context, page, per-page, and required current active
  `academic_period_id`.
- Manual period switching and unscoped attendance lists are outside this slice.
- Empty result is a true empty state.
- Detail surfaces use loaded list records only.
- No correction, import, restore, authoring, or other-student controls exist.

## StudentAttendanceRecordView

**Purpose**: Student-visible attendance record for list and read-only detail.

**Fields**: `id`, `schoolId`, `studentProfileId`, `academicPeriodId`,
`recordedByUserId`, `attendanceDate`, `attendanceStatus`, `status`.

**Source**: `StudentAttendanceRecord`.

**Rules**:

- `attendanceStatus` is the current student-visible attendance status after
  accepted corrections.
- Record belongs to the authenticated student's own active same-school profile.
- No private correction notes or teacher-only metadata are displayed.

## AcademicSummaryView

**Purpose**: Read-only student orientation view assembled from approved
student self-service responses.

**Fields**: `assignedLearningSetCount`, `downloadableContentCount`,
`unavailableContentCount`, `gradeStatusCounts`, `attendanceStatusCounts`,
`feedbackState`.

**Source**: Loaded data from `listStudentLearningSets`, `listStudentGrades`,
and `listStudentAttendance`.

**Rules**:

- Summary uses counts and statuses only.
- Calculated GPA, calculated attendance rate, rankings, trends, transcripts,
  report runs, and report downloads are outside this slice.
- Summary renders only after required active school, student profile, and
  current active period context is valid.

## StudentFeedbackState

**Purpose**: Canonical student UI state used across lists, detail surfaces,
download action, and overview.

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
no_student_profile
no_current_period
unavailable_content
validation
not_found
unsupported_page_size
temporary_unavailable
stale_response
```

**Rules**:

- `safeDetails` may include safe operation names or field-level validation
  labels only.
- Private paths, storage keys, token values, role internals, scan internals,
  guardian data, other-student data, and cross-tenant details are excluded.
