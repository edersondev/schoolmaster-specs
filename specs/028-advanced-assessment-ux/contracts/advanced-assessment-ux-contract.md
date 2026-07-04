# UI Contract: Advanced Assessment Frontend UX

## Purpose

This contract maps Advanced Assessment Frontend UX surfaces to approved
OpenAPI operations, frontend state boundaries, and blocked behavior. It is a
frontend consumption contract, not a backend API change.

## Contract Readiness Gate

Frontend implementation must confirm each consumed advanced assessment
operation is present in `api/openapi.yaml` and
`specs/001-schoolmaster-platform/contracts/openapi.yaml` before any runtime
route calls it. Feature `014-advanced-assessment-content` remains the product
truth for allowed advanced assessment behavior.

## Approved Operations

| UI surface | Operation ID | Method/path | Required context |
|------------|--------------|-------------|------------------|
| Questionnaire authoring | `createQuestionnaire` | `POST /api/v1/questionnaires` | Authenticated active teacher or school administrator, active school, authoring permission |
| Questionnaire authoring update | `updateQuestionnaire` | `PATCH /api/v1/questionnaires/{questionnaireId}` | Same-school authoring permission, editable lifecycle state |
| Questionnaire detail | `getQuestionnaire` | `GET /api/v1/questionnaires/{questionnaireId}` | Same-school actor with approved visibility |
| Student response submission | `submitStudentQuestionnaireResponse` | `POST /api/v1/student/questionnaire-responses` | Authenticated student, active same-school profile, assigned active learning set, before due date, no prior submitted attempt |
| Student own response | `getStudentQuestionnaireResponse` | `GET /api/v1/student/questionnaire-responses/{responseAttemptId}` | Authenticated student owner |
| Review queue | `listQuestionnaireResponses` | `GET /api/v1/questionnaire-responses` | Owning/assigned teacher or same-school school administrator |
| Review detail | `getQuestionnaireResponse` | `GET /api/v1/questionnaire-responses/{responseAttemptId}` | Owning/assigned teacher or same-school school administrator |
| Manual grading | `gradeQuestionnaireResponse` | `POST /api/v1/questionnaire-responses/{responseAttemptId}/grading` | Authorized grader, eligible response state |
| Clean file download | `downloadQuestionnaireResponseFile` | `GET /api/v1/questionnaire-responses/{responseAttemptId}/files/{fileId}/download` | Authorized reviewer, clean scan state |
| Reporting aggregate fields | Approved report catalog/report definition/report request operations | Existing `/api/v1/reports...` operations | Reporting permission and aggregate-only fields |

## Explicitly Not Consumed

| Behavior | Reason |
|----------|--------|
| Staged file uploads | No approved frontend or backend staged answer-file contract |
| Backend draft attempts or partial answer saves | One submitted attempt is the approved response model |
| Inline answer-file preview | Approved file operation is download-only |
| Resubmission, reopened attempts, attempt versioning | Outside advanced assessment contract |
| Advanced question types beyond `long_text` and `file_response` | Outside approved question type set |
| Guardian advanced assessment visibility | Not approved in this feature |
| Raw answer reporting or file links in reports | Reporting is aggregate-only |
| Platform/support detailed answer or file access | Support access remains minimized |

## Route Surfaces

| Route intent | Behavior |
|--------------|----------|
| Advanced questionnaire authoring | Create/update approved advanced questions and render lifecycle lock state |
| Student advanced response | Render assigned questions, local-only text drafts, final-submit file selection, validation, and submission state |
| Assessment review queue | List authorized same-school response attempts with documented filters and returned scan/grading states |
| Response grading | Review eligible answers, download clean files, grade long-text/file-response answers, and choose zero/exempt for failed scans |
| Student assessment result | Show own safe submission, grading, score, feedback summary, and file availability metadata |
| Reporting summaries | Show aggregate-only advanced assessment report fields |
| Direct target route | Show safe unavailable, not-found, denied, conflict, scan, stale, or tenant-safe feedback without exposing unauthorized target existence |

## Service Boundary

All HTTP access must go through advanced assessment service modules. Components
and route views must not call Axios directly.

Service functions:

- `createQuestionnaire(payload)`
- `updateQuestionnaire({ questionnaireId, payload })`
- `getQuestionnaire({ questionnaireId })`
- `submitStudentQuestionnaireResponse(formDataOrPayload)`
- `getStudentQuestionnaireResponse({ responseAttemptId })`
- `listQuestionnaireResponses({ page, perPage, filters, sort })`
- `getQuestionnaireResponse({ responseAttemptId })`
- `gradeQuestionnaireResponse({ responseAttemptId, payload })`
- `downloadQuestionnaireResponseFile({ responseAttemptId, fileId })`
- Existing reporting service functions only where approved aggregate
  assessment fields are returned.

Mapping requirements:

- Submit only documented parameters and request fields.
- For `listQuestionnaireResponses`, submit only documented server-side filters:
  `questionnaire_id`, `learning_set_id`, and `grading_status` until OpenAPI
  approves additional response-status or scan-status query parameters.
- Use multipart only for final assessment submission where files are selected.
- Parse paginated envelopes through shared pagination mappers.
- Normalize validation, unauthorized, forbidden, tenant-mismatch, not-found,
  conflict, scan-pending, scan-failed, unavailable-file, contract-unavailable,
  temporary-unavailable, and stale-response states into safe feedback.
- Drop undocumented response fields.
- Preserve question type, answer schema, lifecycle lock, response attempt,
  scan, grading, failed-scan, result, and report aggregate state.
- Do not create service functions for staged uploads or backend drafts.
- Do not preview clean files inline.

## UI State Contract

Advanced assessment surfaces must distinguish:

- loading
- empty
- unauthorized
- forbidden
- tenant-mismatch
- validation
- not-found
- conflict
- after-due-date
- duplicate-attempt
- scan-pending
- scan-failed
- unavailable-file
- contract-unavailable
- temporary-unavailable
- unsupported-action
- stale-response

True empty states must not be shown for missing permission, denial,
validation, target not-found, contract unavailable, conflict, after-due-date,
duplicate attempt, scan failure, unavailable file, or stale response.

## Capability Gates

- No advanced assessment data request before authenticated active actor and
  active school context are confirmed.
- No authoring controls without approved authoring permission.
- No student submission controls without authenticated student owner,
  assigned active learning set, due date eligibility, and no prior submitted
  attempt.
- No backend draft or staged file request before final submit.
- No review or grading controls without owning/assigned teacher authority or
  same-school school-administrator review authority.
- No clean file download until scan state is clean and actor is authorized.
- No inline file preview.
- No failed-scan zero/exempt actions except for authorized failed-scan
  file-response answers.
- No reporting field outside approved aggregate counts, completion status,
  grading status, and score summaries.

## Authoring Contract

Allowed:

- Existing question types already approved by questionnaire contracts.
- `long_text` and `file_response` question definitions.
- Contract-defined prompt, sequence, answer schema, grading rule, visibility,
  and file rule controls.
- Lifecycle lock display and validation feedback.

Blocked:

- Numeric, date, rating, media, matching, ordering, multi-file, archive,
  audio/video, AI grading, rubric-engine, and any unsupported question type or
  answer schema.
- Edits that change historical student-facing meaning when lifecycle state
  locks the questionnaire.
- Hidden answer keys, private storage details, and unsupported metadata.

## Student Submission Contract

Allowed:

- Local-only text drafts during the current interaction.
- Final-submit text answers and selected file-response files.
- One PDF, image, text, or office file per file-response question, up to
  25 MB.
- Submission before learning-set due date for the assigned same-school
  questionnaire.

Blocked:

- Backend draft response persistence.
- Backend partial answer state.
- Staged file uploads.
- Duplicate attempts.
- After-due-date attempts.
- Unassigned, inactive, cross-tenant, malformed, unsafe, stale, or
  partial-success submissions.

## File Availability Contract

Allowed:

- Safe file display metadata returned by contract.
- Scan status and availability state.
- Download action for clean authorized answer files.

Blocked:

- Inline preview.
- Download for pending, failed, unavailable, inactive, deleted, unauthorized,
  or stale file state.
- Private storage path, scanner internals, raw file content, file links in
  reports, and public file URLs.

## Grading Contract

Allowed:

- Authorized graders may view `long_text` answer content only inside approved
  review and grading surfaces for same-school responses they are allowed to
  grade.
- Manual 0-100 grading for `long_text` answers.
- Manual 0-100 grading for clean `file_response` answers.
- Zero-score and exempt actions for authorized failed-scan file-response
  answers.
- Student-visible feedback summary where approved.

Blocked:

- Manual content-based grading of pending or failed scan files.
- Failed-scan scores outside zero or exempt.
- Private grading notes in student, report, guardian, platform, support,
  diagnostics, or test output.
- Hidden answer keys, rubric internals, scanner internals, full payloads, and
  unauthorized cross-tenant identifiers.

## Student Result Contract

Allowed:

- Own submission status.
- Own grading status.
- Own score summary.
- Student-visible teacher feedback summary.
- Safe file availability metadata.

Blocked:

- Other students' responses.
- Private grading notes.
- Raw file contents.
- Private storage paths.
- Hidden answer keys.
- Unauthorized response existence.

## Reporting Contract

Allowed:

- Assessment counts.
- Completion status.
- Grading status.
- Score summaries.

Blocked:

- Raw answer text.
- Answer keys.
- Uploaded files.
- File links.
- Private file metadata.
- Storage paths.
- Private grading notes.
- Unsupported joins.
- Arbitrary query text.
- Cross-tenant references.

## Stale Response Contract

Responses must be ignored when any of these change before response
application:

- route
- active school
- authenticated actor
- permission context
- selected questionnaire
- selected response
- selected student
- report filters
- assignment state
- due date state
- scan state
- grading state

Returned contract state is authoritative and must not be overwritten by stale
responses.

## Sensitive Data Contract

Client diagnostics, validation labels, filenames, student/result/report
surfaces, guardian/platform/support views, and automated test output must not
expose:

- raw answer text outside authorized grading displays, student-visible
  feedback, or another explicitly approved display surface
- uploaded file contents
- private storage paths
- hidden answer keys
- private grading notes
- scanner internals
- credentials or token values
- full request or response payloads
- unauthorized record existence
- cross-tenant identifiers

## Verification Contract

Frontend verification must cover:

- authoring schema controls and lifecycle locks
- unsupported question type blocking
- long-text validation and local-only draft behavior
- final-submit file-response selection and upload
- one-file, category, size, filename, and mismatch validation feedback
- one-attempt and due-date blocking
- response submission success and failure states
- scan-pending, scan-failed, unavailable-file, and clean file states
- download-only clean file behavior
- failed-scan zero/exempt actions
- review queue filters and safe student identity
- manual 0-100 grading and conflict handling
- student result privacy
- reporting aggregate-only fields
- authorization and tenant-safe denial
- stale-response cancellation or ignore behavior
- no-sensitive-data diagnostics
