# Data Model: Teacher Workflow Workspace

## TeacherWorkspaceScopeView

**Purpose**: Frontend-safe scope context for teacher workspace lists and
actions.

**Fields**: `schoolId`, `academicPeriodId`, `academicPeriodLabel`,
`classSectionId`, `teacherAssignmentId`, `teacherUserId`, `routeQuery`,
`scopeStatus`, `feedbackState`.

**Source**: Existing authenticated session state, active school context,
approved academic-period read contracts, `listTeacherAssignments`, approved
class-section reads, and `listClassSectionMemberships` for selected roster
membership eligibility.

**Rules**:

- Teacher-facing workspace screens default to the current active academic
  period and the teacher's active rosters.
- Explicit academic-period and roster selection is preserved in route/query
  state.
- If no current active period or active teacher roster exists, scoped data
  loading is blocked and the UI shows safe unavailable feedback.
- Eligible student choices for roster-aware learning sets, grades, and
  attendance come from active same-school memberships returned for the selected
  roster; broad student profile lists are not an allowed eligibility source.
- The frontend must not infer teacher assignment, roster, or school context
  outside authenticated approved responses.

## TeacherContentListView

**Purpose**: Paginated teacher content collection for teacher upload review,
scan status scanning, navigation, and safe download entry.

**Fields**: `id`, `schoolId`, `ownerUserId`, `folderId`, `title`,
`description`, `contentType`, `declaredContentType`, `detectedContentType`,
`fileSizeBytes`, `scanStatus`, `status`, `downloadAvailable`,
`feedbackState`.

**Source**: `TeacherContentItem` from `listTeacherContent` and
`getTeacherContent`.

**Rules**:

- Records are visible only within active permitted school context.
- Supported list controls are OpenAPI-documented page and per-page parameters
  unless a later contract adds filters.
- `scanStatus` is `pending`, `clean`, or `failed`.
- Lifecycle `status` is `active`, `inactive`, or `deleted`.
- Download controls are enabled only for authorized clean content in an allowed
  lifecycle state.
- Private storage paths are never represented in frontend state.

## TeacherContentUploadDraft

**Purpose**: Route-local state for approved instructional file upload.

**Fields**: `folderId`, `title`, `description`, `contentType`, `file`,
`fileName`, `fileSizeBytes`, `declaredContentType`, `pending`,
`validationErrors`, `feedbackState`.

**Source**: `TeacherContentCreateRequest` submitted to `createTeacherContent`.

**Rules**:

- Required fields are `title`, `content_type`, and `file`.
- `content_type` is `pdf`, `image`, `text`, or `office_document`.
- File size must not exceed 25 MB.
- Additional request fields are not submitted.
- Draft state is route-local and is cleared after success or navigation.

## TeacherContentLifecycleDraft

**Purpose**: Route-local state for approved teacher content metadata,
lifecycle, delete, restore, and download workflows.

**Fields**: `contentItemId`, `title`, `description`, `folderId`, `status`,
`pending`, `validationErrors`, `feedbackState`, `downloadUrl`, `downloadExpiresAt`.

**Source**: `TeacherContentUpdateRequest`,
`TeacherWorkflowStatusUpdateRequest`, `TeacherWorkflowRestoreRequest`, and
`TeacherContentDownloadResult`.

**Rules**:

- Editable metadata is limited to `folder_id`, `title`, and `description`.
- Status transitions accept `active` or `inactive`; delete and restore use
  dedicated operations.
- Restore returns records to `inactive`.
- Download metadata is transient and must not be persisted beyond the current
  user action.
- Used content edits that change historical student-facing meaning map to
  conflict feedback.

## QuestionnaireListView

**Purpose**: Paginated teacher-authored questionnaire collection for authoring
and learning-set attachment.

**Fields**: `id`, `schoolId`, `ownerUserId`, `title`, `description`,
`status`, `questions`, `feedbackState`.

**Source**: `Questionnaire` from `listQuestionnaires` and
`getQuestionnaire`.

**Rules**:

- Records are visible only within active permitted school context.
- Supported list controls are OpenAPI-documented page and per-page parameters
  unless a later contract adds filters.
- Student response review, response grading, student response file downloads,
  and response corrections are not represented in this model.

## QuestionnaireDraft

**Purpose**: Route-local state for approved questionnaire create/update.

**Fields**: `questionnaireId`, `title`, `description`, `questions`,
`pending`, `validationErrors`, `feedbackState`.

**Source**: `QuestionnaireCreateRequest` and `QuestionnaireUpdateRequest`.

**Rules**:

- Create requires `title` and at least one question.
- This UI slice exposes authoring for `multiple_choice`, `true_false`, and
  `short_text` questions only.
- Question sequence is preserved as submitted.
- Edits that change historical student-facing meaning after use map to
  conflict feedback.
- Additional request fields are not submitted.

## LearningSetListView

**Purpose**: Paginated learning-set collection for selected teacher workspace
scope.

**Fields**: `id`, `schoolId`, `ownerUserId`, `academicPeriodId`, `title`,
`description`, `publishedAt`, `status`, `entries`, `assignments`,
`feedbackState`.

**Source**: `LearningSet` from `listLearningSets` and `getLearningSet`.

**Rules**:

- Lists load only after active school, selected academic period, approved
  teacher roster scope, and approved list filter support are available.
- Legacy direct selected-student assignments remain read-only when present.
- New writes use roster-aware assignment behavior only where the approved
  create or update contract provides roster assignment support.
- Lifecycle `status` is `active`, `inactive`, or `deleted`.

## LearningSetDraft

**Purpose**: Route-local state for learning-set create/update workflows.

**Fields**: `learningSetId`, `academicPeriodId`, `title`, `description`,
`entries`, `rosterAssignment`, `dueAt`, `pending`, `validationErrors`,
`feedbackState`.

**Source**: `LearningSetCreateRequest` and `LearningSetUpdateRequest`.

**Rules**:

- Entries must include at least one ordered content item or questionnaire.
- Entry references must be same-school, active or allowed by the contract, and
  content entries must be clean before use where required.
- New roster-aware writes use `roster_assignment.class_section_id` where the
  approved operation supports it.
- Direct selected-student assignment creation remains blocked for this UI
  slice even if legacy responses contain direct assignments.
- Learning-set create controls remain blocked until OpenAPI adds or confirms a
  roster-aware create request shape.
- Dependency conflicts do not show partial success.

**States**:

```text
draft -> active
active -> inactive
inactive -> active
active -> deleted
deleted -> inactive
any -> conflict
```

## GradeRecordView

**Purpose**: Frontend-safe academic grade record for list/detail/current value
display and correction workflows.

**Fields**: `id`, `schoolId`, `studentProfileId`, `academicPeriodId`,
`recordedByUserId`, `originalRecordedByUserId`, `gradeValue`,
`currentValue`, `originalValue`, `gradeLabel`, `correctionHistory`, `status`,
`recordedAt`, `feedbackState`.

**Source**: `GradeRecord` from `listGrades`, `createGrade`, `getGrade`,
`correctGrade`, `updateGradeStatus`, `restoreGrade`, and `deleteGrade`.

**Rules**:

- `grade_value` and `current_value` are numeric values from 0 to 100.
- Teacher-facing creation requires selected academic period and eligible
  same-school student context from approved roster scope.
- List filtering by academic period or roster requires approved OpenAPI filter
  support before implementation sends those query parameters.
- Correction history displays only tenant-safe fields allowed for the actor.
- Closed-period corrections are school-administrator-only where approved.
- Restore returns deleted records to `inactive`.

## GradeDraft

**Purpose**: Route-local grade create or correction input state.

**Fields**: `studentProfileId`, `academicPeriodId`, `gradeValue`,
`gradeLabel`, `correctionReason`, `pending`, `validationErrors`,
`feedbackState`.

**Source**: `GradeCreateRequest` and `GradeCorrectionRequest`.

**Rules**:

- Create requires `student_profile_id`, `academic_period_id`, and
  `grade_value`.
- Correction requires `grade_value` and `correction_reason`.
- `correction_reason` must be 10 to 500 characters.
- Draft state is not persisted after submission or navigation.

## AttendanceRecordView

**Purpose**: Frontend-safe attendance record for list/detail/current value
display and correction workflows.

**Fields**: `id`, `schoolId`, `studentProfileId`, `academicPeriodId`,
`recordedByUserId`, `originalRecordedByUserId`, `attendanceDate`,
`attendanceStatus`, `currentValue`, `originalValue`, `correctionHistory`,
`status`, `feedbackState`.

**Source**: `AttendanceRecord` from `listAttendance`, `createAttendance`,
`getAttendance`, `correctAttendance`, `updateAttendanceStatus`,
`restoreAttendance`, and `deleteAttendance`.

**Rules**:

- `attendance_status` and `current_value` use `present`, `absent`, `late`,
  `excused`, `remote`, or `suspended`.
- Teacher-facing creation requires selected academic period and eligible
  same-school student context from approved roster scope.
- List filtering by academic period or roster requires approved OpenAPI filter
  support before implementation sends those query parameters.
- Correction history displays only tenant-safe fields allowed for the actor.
- Closed-period corrections are school-administrator-only where approved.
- Restore returns deleted records to `inactive`.

## AttendanceDraft

**Purpose**: Route-local attendance create or correction input state.

**Fields**: `studentProfileId`, `academicPeriodId`, `attendanceDate`,
`attendanceStatus`, `correctionReason`, `pending`, `validationErrors`,
`feedbackState`.

**Source**: `AttendanceCreateRequest` and `AttendanceCorrectionRequest`.

**Rules**:

- Create requires `student_profile_id`, `academic_period_id`,
  `attendance_date`, and `attendance_status`.
- Correction requires `attendance_status` and `correction_reason`.
- `correction_reason` must be 10 to 500 characters.
- Draft state is not persisted after submission or navigation.

## TeacherWorkflowImportDraft

**Purpose**: Admin-only route-local state for grade or attendance JSON import.

**Fields**: `importType`, `rows`, `rowCount`, `acceptedRowCount`,
`rejectedRowCount`, `status`, `errorSummary`, `pending`, `validationErrors`,
`feedbackState`.

**Source**: `GradeImportRequest`, `AttendanceImportRequest`, and `ImportRun`
from `importGrades` or `importAttendance`.

**Rules**:

- `importType` is `grade` or `attendance`.
- `rows` must contain 1 to 500 structured JSON rows.
- Grade rows require `student_profile_id`, `academic_period_id`, and
  `grade_value`.
- Attendance rows require `student_profile_id`, `academic_period_id`,
  `attendance_date`, and `attendance_status`.
- CSV, spreadsheet, archive, and file-upload imports are not exposed.
- Rejected imports show no partial success.
- Full import payloads are not persisted in diagnostics.

## CorrectionRecordView

**Purpose**: Tenant-safe correction history summary for grade and attendance
details.

**Fields**: `id`, `actorUserId`, `originalValue`, `newValue`, `reasonSummary`,
`correctedAt`, `visibility`, `feedbackState`.

**Source**: Approved correction history fields embedded in grade and
attendance records.

**Rules**:

- Private correction notes and unauthorized actor metadata are redacted.
- Current value is displayed separately from original value.
- Correction history is read-only in the frontend.

## AdminTeacherWorkflowObservationView

**Purpose**: Same-school administrator observation surface for teacher
workflow records.

**Fields**: `resourceType`, `resourceId`, `ownerUserId`, `schoolId`,
`academicPeriodId`, `status`, `summary`, `detail`, `feedbackState`.

**Source**: Approved teacher workflow detail/list operations visible to the
administrator.

**Rules**:

- Observation is read/detail by default.
- Admin-only imports and closed-period corrections may be exposed where
  approved.
- Teacher-owned create, update, lifecycle, download, and learning-set
  management controls remain hidden unless the approved contract explicitly
  requires administrator action.
