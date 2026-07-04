# Data Model: Advanced Assessment Frontend UX

## AdvancedAssessmentWorkspaceContext

**Purpose**: Frontend-safe context required before advanced assessment
authoring, submission, review, grading, result, or reporting screens load.

**Fields**: `actorId`, `activeSchoolId`, `actorRoleContext`,
`authoringAccessState`, `studentSubmissionAccessState`, `reviewAccessState`,
`gradingAccessState`, `reportingAccessState`, `selectedQuestionnaireId`,
`selectedResponseAttemptId`, `selectedStudentId`, `workspaceStatus`,
`feedbackState`.

**Source**: Approved authenticated session, current-user/permission context,
active school context, and approved advanced assessment contracts.

**Rules**:

- All screens require authenticated active actor state and active permitted
  school context.
- Missing permission, inactive actor, missing active school, unavailable
  contract, validation, conflict, not-found, denied, stale response, and
  temporary-unavailable remain distinct states.
- School-owned records remain scoped to active school.
- Guardian, platform, and support contexts do not unlock detailed advanced
  answer or file access.

## AdvancedQuestionAuthoringView

**Purpose**: UI representation of an existing or advanced questionnaire
question.

**Fields**: `questionId`, `questionType`, `prompt`, `sequence`,
`answerSchema`, `gradingRule`, `fileRule`, `visibilityState`,
`lifecycleLockState`, `validationState`, `feedbackState`.

**Source**: `createQuestionnaire`, `updateQuestionnaire`, and
`getQuestionnaire`.

**Rules**:

- Supported question types are existing `multiple_choice`, `true_false`,
  `short_text`, plus approved `long_text` and `file_response`.
- Unsupported question types and answer schemas are not represented as
  editable controls.
- Lifecycle lock state blocks edits that would change historical
  student-facing meaning.
- Validation feedback maps only documented contract errors.

## LongTextAnswerEntryView

**Purpose**: Student-facing long-text answer entry state.

**Fields**: `questionId`, `value`, `characterCount`, `minLength`,
`maxLength`, `localDraftState`, `validationState`, `feedbackState`.

**Source**: Approved assigned questionnaire metadata and local-only browser
state before final submission.

**Rules**:

- Valid length is 1 to 10,000 characters.
- Blank, whitespace-only, invalid encoding, and unsafe control-character
  states are blocked or shown from validation feedback.
- Unsubmitted text drafts may be local-only; no backend draft response or
  partial answer state is created.
- Text is treated as plain text, never markup.

## FileResponseSelectionView

**Purpose**: Student-facing file-response selection before final submission.

**Fields**: `questionId`, `selectedFileName`, `selectedFileCategory`,
`selectedFileSize`, `allowedCategories`, `maxFileSize`, `fileCount`,
`selectionState`, `validationState`, `feedbackState`.

**Source**: Approved assigned questionnaire metadata and local selected file
reference.

**Rules**:

- One file is allowed per `file_response` question.
- Allowed categories are PDF, image, text, and office files.
- Maximum size is 25 MB.
- Unsafe filename, unsupported category, oversized file, multiple files, and
  declared/detected mismatch feedback uses approved validation states.
- Files upload only with final assessment submission; staged file uploads are
  not represented.

## StudentAdvancedResponseView

**Purpose**: UI representation of one student's assigned questionnaire
response workflow.

**Fields**: `questionnaireId`, `learningSetId`, `studentId`, `dueDate`,
`attemptState`, `oneAttemptState`, `questions`, `longTextEntries`,
`fileSelections`, `submissionReadiness`, `submittedAt`, `feedbackState`,
`staleRequestKey`.

**Source**: Approved learning-set/questionnaire assignment context and
`submitStudentQuestionnaireResponse`.

**Rules**:

- Submission requires active same-school student profile, assigned active
  learning set, active questionnaire, due date not passed, and no prior
  submitted attempt.
- Final submission includes text answers and selected files together.
- Duplicate, after-due-date, unassigned, malformed, stale, unsafe upload, or
  partial-success states are blocked.
- Successful submission removes resubmission actions.

## ResponseFileAvailabilityView

**Purpose**: UI representation of a submitted file answer after backend scan
state exists.

**Fields**: `fileId`, `responseAttemptId`, `questionId`, `displayName`,
`category`, `size`, `scanStatus`, `availabilityState`, `downloadAllowed`,
`feedbackState`.

**Source**: Student response detail, teacher/admin response detail, and
`downloadQuestionnaireResponseFile`.

**Rules**:

- Scan states are shown only where documented.
- Pending, failed, unavailable, blocked, inactive, deleted, unauthorized, and
  stale files are not downloadable.
- Clean authorized files are download-only; no inline preview is represented.
- Private storage paths, scanner internals, and file contents are not kept in
  UI state.

## AssessmentReviewQueueView

**Purpose**: Teacher or school-administrator view of submitted advanced
responses that require review or grading.

**Fields**: `items`, `pagination`, `filters`, `sorting`, `loading`,
`emptyState`, `feedbackState`, `staleRequestKey`.

**Source**: `listQuestionnaireResponses`.

**Rules**:

- Request requires owning/assigned teacher authority or same-school
  school-administrator assessment review authority.
- Filters and sorting use only documented contract fields.
- Safe student identity fields may display where authorized.
- Empty and no-filter-results are distinct from denied, validation, not-found,
  conflict, unavailable, and stale-response states.

## ManualGradingView

**Purpose**: UI representation of grading actions for manually graded advanced
answers.

**Fields**: `responseAttemptId`, `answerId`, `questionId`, `questionType`,
`gradingState`, `score`, `studentFeedbackSummary`, `failedScanAction`,
`conflictState`, `feedbackState`, `staleRequestKey`.

**Source**: `getQuestionnaireResponse` and `gradeQuestionnaireResponse`.

**Rules**:

- `long_text` and clean `file_response` answers use manual 0-100 scoring.
- Failed-scan file-response answers expose only zero-score and exempt actions.
- Pending, clean, unavailable, unauthorized, inactive, deleted, or stale file
  states do not expose failed-scan zero/exempt controls.
- Private grading notes, hidden answer keys, rubric internals, scanner
  internals, storage paths, full payloads, and unauthorized identifiers are
  not represented.

## StudentAssessmentResultView

**Purpose**: Student-facing result summary for the authenticated student's own
advanced assessment response.

**Fields**: `responseAttemptId`, `questionnaireId`, `submissionStatus`,
`gradingStatus`, `scoreSummary`, `teacherFeedbackSummary`,
`fileAvailabilityMetadata`, `feedbackState`, `staleRequestKey`.

**Source**: `getStudentQuestionnaireResponse`.

**Rules**:

- Only the authenticated student's own same-school response is represented.
- Private grading notes, raw file contents, private storage paths, hidden
  answer keys, and unauthorized response details are not represented.
- Ungraded, scan-pending, scan-failed, unavailable, and hidden states remain
  visibly distinct.

## AdvancedAssessmentReportingView

**Purpose**: Reporting-facing advanced assessment aggregate view model.

**Fields**: `catalogFields`, `definitionFields`, `requestFields`,
`resultSummary`, `counts`, `completionStatus`, `gradingStatus`,
`scoreSummary`, `feedbackState`, `staleRequestKey`.

**Source**: Approved report catalog, report definition, report request,
history, and result operations that expose advanced assessment aggregate
fields.

**Rules**:

- Allowed fields are assessment counts, completion status, grading status, and
  score summaries.
- Raw answer text, answer keys, uploaded files, file links, private file
  metadata, storage paths, private grading notes, unsupported joins, arbitrary
  query text, and cross-tenant references are blocked.
- Reporting permissions remain required.

## AdvancedAssessmentFeedbackState

**Purpose**: Canonical advanced assessment UI feedback state used across
authoring, submission, review, grading, student results, and reporting.

**Fields**: `kind`, `messageKey`, `recoverable`, `safeDetails`.

**States**:

```text
idle
loading
empty
unauthorized
forbidden
tenant-mismatch
validation
not-found
conflict
after-due-date
duplicate-attempt
scan-pending
scan-failed
unavailable-file
unsupported-action
contract-unavailable
temporary-unavailable
stale-response
```

**Rules**:

- Feedback never includes raw answer text, uploaded file contents, storage
  paths, answer keys, private notes, credentials, full payloads, or
  unauthorized record existence.
- True empty states are not used for authorization, validation, conflict,
  unavailable, scan, or stale-response outcomes.

## AdvancedAssessmentRefreshState

**Purpose**: Stale-response guard for requests whose results can become invalid
when context changes.

**Fields**: `requestKey`, `startedAt`, `activeRoute`, `activeSchoolId`,
`selectedQuestionnaireId`, `selectedResponseAttemptId`, `selectedStudentId`,
`filters`, `authVersion`, `permissionVersion`, `dueDateState`, `scanState`,
`gradingState`.

**Rules**:

- Responses are ignored when route, questionnaire, response, student, report
  filters, active school, authentication, permission, assignment, due date,
  scan state, or grading state changes before response application.
- Returned contract state remains authoritative.
- Stale responses must not overwrite current visible state.
