# Teacher Workflow Workspace UI Contract

This contract defines what `schoolmaster-frontend` may implement for Teacher
Workflow Workspace.

## Scope Contract

This feature approves:

- Teacher-facing workspace routes scoped to active school, current active
  academic period, and active teacher roster assignment by default.
- Selected academic period and roster preserved in route/query state.
- Teacher content list, upload, detail, metadata update, lifecycle status,
  soft-delete, restore, and clean authorized download UI.
- Questionnaire list, create, detail, update, lifecycle status, soft-delete,
  restore, and learning-set attachment UI.
- Learning-set list, create, detail, update, lifecycle status, soft-delete,
  restore, ordered entries, roster-aware audience display, and read-only legacy
  direct assignment display once approved roster-aware create support and
  scoped list filters are available.
- Grade and attendance list, create, detail, lifecycle status, soft-delete,
  restore, correction, and tenant-safe correction history UI.
- Same-school administrator read/detail observation for teacher workflow
  records, plus admin-only closed-period corrections where approved.
- Admin-only grade and attendance structured JSON import UI capped at 500 rows
  with all-or-nothing feedback.
- Safe loading, empty, validation, unauthorized, forbidden, tenant-mismatch,
  inactive-school, inactive-record, not-found, conflict, unsupported filter,
  unsupported sort, unsupported page-size, scan-unavailable, download-denied,
  import-validation, stale-record, and temporary-unavailable feedback states.
- Route-local sensitive state and service-isolated API consumption.
- No-sensitive-data diagnostics verification.

This feature does not approve:

- Student questionnaire response review, teacher grading of questionnaire
  responses, student response file downloads, or response correction workflows.
- CSV, spreadsheet, archive, or file-upload grade/attendance imports.
- Student self-service, guardian self-service, reporting workspace, platform
  support, billing, messaging, notification-center, permanent purge, legal
  hold, data anonymization, or undocumented behavior.
- Client-side tenant inference or platform override of school-owned records.
- Private file path display or persistence.

## OpenAPI Consumption Contract

Frontend services may consume only these approved operations for this slice:

| Operation ID | Method and Path | UI use |
|--------------|-----------------|--------|
| `listAcademicPeriods` | `GET /api/v1/academic-periods` | Resolve current active academic period and populate period selector where approved. |
| `listClassSections` | `GET /api/v1/class-sections` | Resolve active roster/class-section options where approved by feature 022 contracts. |
| `listClassSectionMemberships` | `GET /api/v1/class-sections/{classSectionId}/memberships` | Resolve selected roster active memberships and eligible `student_profile_id` values for roster-aware learning-set audiences and grade/attendance create forms. |
| `listTeacherAssignments` | `GET /api/v1/teacher-assignments` | Resolve teacher active roster scope and admin observation where approved. |
| `getTeacherAssignment` | `GET /api/v1/teacher-assignments/{teacherAssignmentId}` | Retrieve assignment detail for active teacher workspace scope or admin observation. |
| `listTeacherContent` | `GET /api/v1/teacher-content` | Content list with approved pagination, loading, empty, and denied states. |
| `createTeacherContent` | `POST /api/v1/teacher-content` | Upload approved instructional file and metadata. |
| `getTeacherContent` | `GET /api/v1/teacher-content/{contentItemId}` | Retrieve content detail. |
| `updateTeacherContent` | `PATCH /api/v1/teacher-content/{contentItemId}` | Update approved metadata only. |
| `updateTeacherContentStatus` | `PATCH /api/v1/teacher-content/{contentItemId}/status` | Activate or deactivate content through documented transition. |
| `deleteTeacherContent` | `DELETE /api/v1/teacher-content/{contentItemId}` | Soft-delete content where approved. |
| `restoreTeacherContent` | `POST /api/v1/teacher-content/{contentItemId}/restore` | Restore deleted content to inactive. |
| `downloadTeacherContent` | `GET /api/v1/teacher-content/{contentItemId}/download` | Request transient download metadata for clean authorized content. |
| `listQuestionnaires` | `GET /api/v1/questionnaires` | Questionnaire list with approved pagination. |
| `createQuestionnaire` | `POST /api/v1/questionnaires` | Create questionnaire for supported authoring scope. |
| `getQuestionnaire` | `GET /api/v1/questionnaires/{questionnaireId}` | Retrieve questionnaire detail. |
| `updateQuestionnaire` | `PATCH /api/v1/questionnaires/{questionnaireId}` | Update editable questionnaire fields and supported questions. |
| `updateQuestionnaireStatus` | `PATCH /api/v1/questionnaires/{questionnaireId}/status` | Activate or deactivate questionnaire. |
| `deleteQuestionnaire` | `DELETE /api/v1/questionnaires/{questionnaireId}` | Soft-delete questionnaire where approved. |
| `restoreQuestionnaire` | `POST /api/v1/questionnaires/{questionnaireId}/restore` | Restore deleted questionnaire to inactive. |
| `listLearningSets` | `GET /api/v1/learning-sets` | Learning-set list with approved pagination and required period/roster filters once OpenAPI documents them. |
| `createLearningSet` | `POST /api/v1/learning-sets` | Create learning set only after OpenAPI documents roster-aware create request behavior for this UI. |
| `getLearningSet` | `GET /api/v1/learning-sets/{learningSetId}` | Retrieve learning-set detail. |
| `updateLearningSet` | `PATCH /api/v1/learning-sets/{learningSetId}` | Update editable learning-set fields and roster-aware audience where approved. |
| `updateLearningSetStatus` | `PATCH /api/v1/learning-sets/{learningSetId}/status` | Activate or deactivate learning set. |
| `deleteLearningSet` | `DELETE /api/v1/learning-sets/{learningSetId}` | Soft-delete learning set where approved. |
| `restoreLearningSet` | `POST /api/v1/learning-sets/{learningSetId}/restore` | Restore deleted learning set to inactive. |
| `listGrades` | `GET /api/v1/grades` | Grade list with approved pagination and required period/roster filters once OpenAPI documents them. |
| `createGrade` | `POST /api/v1/grades` | Record grade for eligible student and period. |
| `getGrade` | `GET /api/v1/grades/{gradeId}` | Retrieve grade detail. |
| `correctGrade` | `PATCH /api/v1/grades/{gradeId}/correction` | Correct grade with 10-500 character reason. |
| `updateGradeStatus` | `PATCH /api/v1/grades/{gradeId}/status` | Activate or deactivate grade record. |
| `deleteGrade` | `DELETE /api/v1/grades/{gradeId}` | Soft-delete grade record where approved. |
| `restoreGrade` | `POST /api/v1/grades/{gradeId}/restore` | Restore deleted grade to inactive. |
| `importGrades` | `POST /api/v1/grades/imports` | Admin-only create-only structured JSON grade import. |
| `listAttendance` | `GET /api/v1/attendance` | Attendance list with approved pagination and required period/roster filters once OpenAPI documents them. |
| `createAttendance` | `POST /api/v1/attendance` | Record attendance for eligible student and period. |
| `getAttendance` | `GET /api/v1/attendance/{attendanceId}` | Retrieve attendance detail. |
| `correctAttendance` | `PATCH /api/v1/attendance/{attendanceId}/correction` | Correct attendance with 10-500 character reason. |
| `updateAttendanceStatus` | `PATCH /api/v1/attendance/{attendanceId}/status` | Activate or deactivate attendance record. |
| `deleteAttendance` | `DELETE /api/v1/attendance/{attendanceId}` | Soft-delete attendance record where approved. |
| `restoreAttendance` | `POST /api/v1/attendance/{attendanceId}/restore` | Restore deleted attendance to inactive. |
| `importAttendance` | `POST /api/v1/attendance/imports` | Admin-only create-only structured JSON attendance import. |

Frontend implementation must not add route aliases, request fields, response
fields, filters, sort values, include expansion, page-size behavior, lifecycle
actions, correction states, import formats, status values, permission names, or
capability names that are not documented in OpenAPI or the approved session
contract.

## Request Contract

### Workspace Scope

Teacher workspace scope may use only authenticated session state, active school
context, approved academic-period reads, `listTeacherAssignments`, approved
class-section/roster reads, and `listClassSectionMemberships` for the selected
class section. Teacher-facing list pages default to the current active academic
period and active teacher rosters. Explicit selected period and roster are
preserved in route/query state.

`listClassSectionMemberships` may send only tenant context, class section ID,
page, per-page, academic-period filter, and membership status filter. Teacher
workflow forms use active same-school memberships from the selected roster as
the approved source for eligible `student_profile_id` choices. The UI must not
load broad student profile lists or infer eligibility outside the approved
membership response.

### Teacher Content

`listTeacherContent` may send only tenant context, page, and per-page
parameters unless OpenAPI adds filters before implementation.

`createTeacherContent` sends multipart form data using only:

- `folder_id`
- `title`
- `description`
- `content_type`
- `file`

`content_type` may be `pdf`, `image`, `text`, or `office_document`; `file` is
limited to approved file categories and 25 MB maximum.

`updateTeacherContent` sends only:

- `folder_id`
- `title`
- `description`

`updateTeacherContentStatus` sends only `status` as `active` or `inactive`.
Delete and restore use dedicated operations. `downloadTeacherContent` returns
transient download metadata and must not store private file paths.

### Questionnaires

`listQuestionnaires` may send only tenant context, page, and per-page
parameters unless OpenAPI adds filters before implementation.

`createQuestionnaire` and `updateQuestionnaire` send only:

- `title`
- `description`
- `questions`

This UI slice exposes authoring for `multiple_choice`, `true_false`, and
`short_text` questions only. It must not expose student questionnaire response
review, teacher grading, student response file downloads, or response
correction workflows.

### Learning Sets

`listLearningSets` may send only tenant context, page, and per-page parameters
until OpenAPI adds academic-period and roster/class-section filters. The UI
must not send undocumented scoped filters.

`createLearningSet` or `updateLearningSet` may send only documented fields:

- `academic_period_id` where required
- `title`
- `description`
- `entries`
- `roster_assignment`
- `due_at`

`entries` contain ordered `content_item` or `questionnaire` references. New
roster-aware writes use approved roster assignment behavior. Learning-set
create controls remain blocked until OpenAPI documents roster-aware create
request behavior. Legacy direct selected-student assignments may be displayed
read-only when returned by an approved response, but the UI must not create new
direct selected-student assignment controls in this slice.

### Grades

`listGrades` may send only tenant context, page, and per-page parameters until
OpenAPI adds academic-period and roster/class-section filters. The UI must not
send undocumented scoped filters.

`createGrade` sends:

- `student_profile_id`
- `academic_period_id`
- `grade_value`
- `grade_label`

`grade_value` is 0 to 100. `correctGrade` sends:

- `grade_value`
- `grade_label`
- `correction_reason`

`correction_reason` is required and must be 10 to 500 characters.

### Attendance

`listAttendance` may send only tenant context, page, and per-page parameters
until OpenAPI adds academic-period and roster/class-section filters. The UI
must not send undocumented scoped filters.

`createAttendance` sends:

- `student_profile_id`
- `academic_period_id`
- `attendance_date`
- `attendance_status`

`attendance_status` is `present`, `absent`, `late`, `excused`, `remote`, or
`suspended`. `correctAttendance` sends:

- `attendance_status`
- `correction_reason`

`correction_reason` is required and must be 10 to 500 characters.

### Imports

`importGrades` sends:

- `rows`, 1 to 500 grade rows containing `student_profile_id`,
  `academic_period_id`, `grade_value`, and optional `grade_label`

`importAttendance` sends:

- `rows`, 1 to 500 attendance rows containing `student_profile_id`,
  `academic_period_id`, `attendance_date`, and `attendance_status`

The UI must not expose CSV, spreadsheet, archive, or file-upload imports. It
must not show partial success when an import is rejected.

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
- `scan-unavailable`
- `download-denied`
- `import-validation`
- `stale-record`
- `temporary-unavailable`

Conflict feedback must cover unclean content, historical-meaning questionnaire
edits, inactive or deleted dependencies, unauthorized teacher ownership,
roster assignment conflicts, legacy direct-assignment read-only records,
closed-period correction denial, invalid correction reason length, duplicate or
invalid grade/attendance imports, and stale state without exposing hidden
tenant, role, permission, private file, correction, import, or protected
student details.

## Tenant and Authorization Contract

- All screens require authenticated active user state.
- School-owned records require active permitted school context.
- Teacher-facing surfaces require approved teacher workflow authority and
  roster or ownership scope where applicable.
- School administrators get same-school read/detail observation plus
  admin-only imports and closed-period corrections where approved.
- Platform scope does not grant implicit teacher workflow authority.
- Client-side permission checks are visibility aids only.
- Frontend action visibility remains blocked until implementation confirms
  approved permission codes or session capability flags.
- Backend authorization remains authoritative when client state is stale.

## State Boundary Contract

Frontend route-local state may own:

- list filters and pagination
- selected academic-period route/query state
- selected roster route/query state
- current validation errors
- form draft values for the current route
- upload pending flags and selected local file reference
- transient download request feedback
- correction draft values and reason while dialog is open
- JSON import draft while import route is active
- current dialog state
- current safe feedback state

Frontend state and diagnostics must not persist:

- student private details beyond current route needs
- teacher private details beyond approved response display
- guardian private details
- tokens
- private file paths
- presigned download URLs after the current action
- full upload metadata
- full request payloads
- correction private notes
- correction reasons after submission
- full import payloads after navigation or submission
- role internals
- permission payloads
- cross-tenant details

Stale responses after route, filter, pagination, session, active school,
academic period, roster, target resource, correction dialog, import draft, or
permission changes must be ignored.

## Accessibility, Localization, and Observability Contract

- Teacher workflow screens must target WCAG 2.1 AA at 390px, 768px, and
  1440px.
- Reusable text must be centralized through Vue I18n.
- Element Plus component tags must use PascalCase.
- Upload, download, correction, import, lifecycle, and dependency conflict
  controls must expose labels, status, disabled reason, and error feedback to
  assistive technology.
- Frontend diagnostics may include safe operation identifiers and request
  identifiers, but must not log student details, teacher private details,
  guardian details, tokens, permission payloads, role internals, private file
  paths, full upload metadata, full import payloads, correction private notes,
  source-school private data, or unauthorized cross-tenant details.
