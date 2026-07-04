# Feature Specification: Advanced Assessment Frontend UX

**Feature Branch**: `028-advanced-assessment-ux`  
**Created**: 2026-07-04  
**Status**: Draft  
**Input**: User description: "Specify feature 028 Advanced Assessment Frontend UX. Define frontend-only SPA surfaces for questionnaire authoring, long-text and file-response submission, manual grading, student result visibility, and related reporting views. Consume only approved advanced assessment contracts. Specify question schemas, answer validation, file-response restrictions, malware-scan visibility, grading states, student/report display behavior, authorization boundaries, empty states, loading states, stale-response handling, and safe error behavior. Keep scope aligned with Vue 3 SPA patterns and OpenAPI-backed contracts."

## Clarifications

### Session 2026-07-04

- Q: When should file-response files be uploaded? → A: Upload file-response files only with final assessment submission.
- Q: How should unsubmitted answer drafts behave? → A: Keep local-only text drafts; files are selected only at final submit.
- Q: How should authorized reviewers access clean file-response files? → A: Download clean files only; no inline preview.
- Q: Which grading controls should appear for failed-scan file answers? → A: Show both zero-score and exempt actions for failed-scan file answers.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Author Advanced Questionnaires (Priority: P1)

An authorized teacher or school administrator creates and maintains questionnaires that include approved advanced question types while preserving existing question behavior and historical meaning.

**Why this priority**: Authoring defines the question schemas, validation messages, editable states, and downstream student and grading behavior. The remaining UX depends on a clear advanced-question authoring surface.

**Independent Test**: Can be fully tested by signing in as an authorized teacher or school administrator, creating a questionnaire with existing questions plus approved `long_text` and `file_response` questions, reviewing schema guidance and validation feedback, and confirming unsupported question types or historical-meaning edits are blocked.

**Acceptance Scenarios**:

1. **Given** an authorized same-school teacher or school administrator opens questionnaire authoring, **When** they add existing question types plus approved `long_text` and `file_response` questions, **Then** the UI shows only approved schema fields, answer constraints, grading expectations, and file rules.
2. **Given** an advanced question has invalid prompt, answer schema, grading rule, file rule, sequence, or visibility settings, **When** the actor attempts to save, **Then** the UI shows field-specific validation without submitting unsupported or malformed data.
3. **Given** a questionnaire has been published, assigned, answered, graded, or otherwise locked by lifecycle rules, **When** an actor attempts to change advanced question type, prompt, answer schema, grading rule, file rule, sequence, or visibility, **Then** the UI blocks or explains the locked state without changing historical student-facing meaning.
4. **Given** the actor lacks authoring permission, school context, or access to the target questionnaire, **When** they navigate directly to advanced authoring, **Then** the UI renders a safe denied or not-found state without exposing protected record existence.

---

### User Story 2 - Submit Advanced Assessment Responses (Priority: P2)

An authenticated student answers assigned advanced questionnaire questions using long-text responses or one approved file per file-response question before the learning-set due date.

**Why this priority**: Student submission is the core learner-facing workflow and introduces the highest UX risk around answer validation, upload restrictions, scan status, duplicate attempts, and after-due-date handling.

**Independent Test**: Can be fully tested by signing in as a student with an assigned same-school learning set, submitting valid long-text and file-response answers, reviewing pending scan visibility, and verifying duplicate, expired, invalid, oversized, unsupported, blank, cross-tenant, and stale submissions are blocked safely.

**Acceptance Scenarios**:

1. **Given** a student has an active assigned questionnaire with advanced questions, **When** they open the response screen, **Then** the UI shows each question, required answer constraints, due-date state, one-attempt warning, local-only text draft behavior where approved, and clear submit readiness without uploading files before final submission.
2. **Given** a `long_text` answer is blank, whitespace-only, over 10,000 characters, invalidly encoded, or contains unsafe control characters, **When** the student attempts to submit, **Then** the UI identifies the affected question and prevents submission.
3. **Given** a `file_response` question allows one PDF, image, text, or office file up to 25 MB, **When** the student selects an unsupported file, multiple files, oversized file, unsafe filename, or mismatched file category, **Then** the UI blocks submission and explains the allowed rule.
4. **Given** the student submits valid answers before the due date and has no prior submitted attempt, **When** submission succeeds, **Then** the UI shows submitted status, pending scan visibility for uploaded files, and no resubmission action.
5. **Given** the due date passes, the learning set becomes unavailable, the student already submitted, or authorization changes while the page is open, **When** the student attempts to submit, **Then** the UI prevents duplicate or stale submission and shows safe feedback.

---

### User Story 3 - Grade and Review Advanced Responses (Priority: P3)

An authorized teacher or school administrator reviews submitted advanced responses, sees scan-safe file availability, and manually grades long-text or file-response answers on the approved scale.

**Why this priority**: Manual grading turns advanced submissions into usable academic outcomes and must avoid exposing unsafe files, private grading notes, unauthorized responses, or stale grading state.

**Independent Test**: Can be fully tested by opening submitted advanced responses as an authorized teacher or school administrator, grading manual-answer questions from 0 to 100, handling pending or failed scan files, adding student-visible feedback, and verifying unauthorized, stale, invalid-score, and cross-tenant grading attempts are denied safely.

**Acceptance Scenarios**:

1. **Given** an authorized owning or assigned teacher or same-school school administrator opens response review, **When** responses exist, **Then** the UI lists student response state, grading state, safe file availability, scan state, and review actions only for authorized same-school responses.
2. **Given** a long-text answer or clean file-response answer requires manual grading, **When** the grader enters a score from 0 to 100 and student-visible feedback, **Then** the UI records the grading outcome and refreshes the response state.
3. **Given** a file-response attachment is pending scan, failed scan, unavailable, deleted, inactive, or not authorized, **When** the grader reviews the response, **Then** the UI hides download actions, shows only approved safe status metadata, and exposes zero-score or exempt actions only for authorized failed-scan answers.
4. **Given** a grading action is stale, conflicts with a newer grade, lacks authority, uses an invalid score, or targets an inaccessible response, **When** the actor submits, **Then** the UI keeps the current screen safe, avoids overwriting newer state, and shows the documented denial or conflict state.

---

### User Story 4 - View Student Results and Reporting Summaries (Priority: P4)

Students, teachers, and reporting users view advanced assessment results only through approved summary fields that protect answer text, answer files, private grading notes, and cross-tenant records.

**Why this priority**: Advanced assessment outcomes affect student self-service and reporting, but these views must stay summarized and privacy-safe.

**Independent Test**: Can be fully tested by viewing graded and ungraded advanced responses through student and reporting surfaces, confirming students see only their own status, score, feedback summary, and safe file metadata, and confirming reporting users see only counts, completion status, grading status, and score summaries.

**Acceptance Scenarios**:

1. **Given** a student has submitted an advanced assessment, **When** they view their result after grading, **Then** the UI shows only submission status, grading status, score summary, student-visible teacher feedback summary, and safe file availability metadata.
2. **Given** a student response is ungraded, scan-pending, scan-failed, unavailable, or hidden by authorization, **When** the student views results, **Then** the UI shows the approved pending or unavailable state without exposing private notes, raw files, storage paths, or hidden grading details.
3. **Given** a reporting user builds or views assessment reports, **When** advanced assessment fields are available, **Then** the UI exposes only assessment counts, completion status, grading status, and score summaries.
4. **Given** a report definition, student view, teacher view, guardian view, platform view, or support view attempts to expose raw answer text, answer keys, uploaded files, private file metadata, storage paths, or private grading notes, **When** the UI receives or blocks that state, **Then** it does not display the protected data and shows safe feedback.

### Edge Cases

- Advanced assessment contracts are absent, partially promoted, unavailable, or missing required state values when the UI loads.
- Actor is authenticated but inactive, missing active school context, missing questionnaire permissions, or has only an unrelated role.
- Existing question types appear beside `long_text` and `file_response` questions and must preserve prior behavior.
- Questionnaire lifecycle state changes while an authoring page is open.
- Student loses assignment, school context, active profile, or eligibility while answering.
- Due date passes while a student response screen is open.
- Student attempts duplicate submission after one completed attempt.
- Long-text answer is blank, whitespace-only, too long, invalidly encoded, or contains unsafe control characters.
- File answer is unsupported, oversized, more than one file, mismatched by declared and detected type, unsafe by filename, scan-pending, scan-failed, unavailable, inactive, deleted, or unauthorized.
- File scan status changes while teacher review, grading, reporting, or student result views are open.
- Grading state changes concurrently while a teacher or school administrator submits scores.
- Response list, detail, report, or result requests return empty, denied, tenant-mismatch, validation, not-found, conflict, scan-pending, scan-failed, stale, or temporary-unavailable responses.
- Visible labels, filenames, validation messages, diagnostics, report columns, and test output must not expose raw answer text outside authorized grading displays, student-visible feedback, or another explicitly approved display surface, and must not expose uploaded file contents, storage paths, answer keys, private grading notes, credentials, full payloads, or unauthorized record existence.
- Guardian, platform support, messaging, notification, billing, public file sharing, generated report file packaging, legal hold, anonymization, and permanent purge behavior remain out of scope unless separately approved.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is included in this frontend feature. Advanced assessment contracts and backend behavior from `specs/014-advanced-assessment-content/` must be approved and available before runtime consumption.
- **Frontend repository impact**: Adds advanced questionnaire authoring surfaces, student advanced response submission surfaces, teacher or school-administrator response review and grading surfaces, student result surfaces, related reporting views, permission-aware navigation, loading states, empty states, denied states, conflict states, scan status feedback, stale-response handling, and safe error behavior.
- **Specification or contract repository impact**: This specification defines frontend consumption boundaries for approved advanced assessment contracts. OpenAPI changes are required before frontend implementation if advanced assessment operations or fields are absent from `api/openapi.yaml`.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines this UI boundary first. `schoolmaster-backend` and OpenAPI contract work must provide approved advanced assessment operations before `schoolmaster-frontend` exposes runtime behavior.

### API Contract Impact

- **OpenAPI update required**: Yes before frontend implementation if approved advanced assessment operations, state values, validation errors, scan states, grading states, response summaries, or reporting fields are absent from `api/openapi.yaml`.
- **Versioned endpoints affected**: Frontend may consume only approved questionnaire, learning-set, student questionnaire response, response review, grading, response-file availability, report catalog, report definition, report request, current-user, permission, session, and school context operations documented under `/api/v1`.
- **JSON response impact**: UI behavior depends only on documented success, accepted, paginated, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, scan-pending, scan-failed, unavailable-file, stale-response, empty, loading, grading-state, file-availability, result-summary, and report-field semantics.
- **Authentication/authorization impact**: Authoring requires approved teacher or school-administrator assessment authoring permission. Student submission and result views require the authenticated student to be linked to the active same-school assigned profile. Grading requires owning or assigned teacher authority or same-school school-administrator assessment review authority. Reporting requires approved reporting permission.
- **Compatibility impact**: Frontend delivery is additive. Existing authentication, school administration, teacher workflow, student self-service, guardian self-service, reporting workspace, platform support, and existing question behavior remain unchanged unless a separate approved specification changes them.

### Data & Tenancy Impact

- **Tenant scoping impact**: Advanced questionnaire metadata, student response attempts, answer states, file-response metadata, grading states, student result summaries, and report summaries are school-owned and must remain scoped to the active school context.
- **Cross-tenant or platform access impact**: Cross-school questionnaire, response, file, grading, result, or report access must render safe denial or not-found states without exposing protected existence. Platform and support users receive no implicit access to answer text or files.
- **Soft delete impact**: UI must reflect approved inactive, deleted, restored, unavailable, historical-meaning lock, and retained-summary states without exposing deleted school-owned details or unsupported restore/delete actions.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The UI MUST expose only advanced assessment behavior documented in approved contracts before frontend implementation begins.
- **FR-002**: The UI MUST require authenticated active school context before loading advanced questionnaire, response, grading, student result, file availability, or reporting data.
- **FR-003**: The UI MUST permission-gate advanced assessment authoring, student submission, response review, manual grading, student result visibility, and reporting views using approved current-user and permission signals.
- **FR-004**: The UI MUST preserve existing multiple-choice, true-false, and short-text question behavior while adding only approved `long_text` and `file_response` authoring and response surfaces.
- **FR-005**: The UI MUST reject or block unsupported advanced question types, unsupported answer shapes, unsupported grading rules, unsupported report fields, and unsupported file rules before submission when the approved contract provides enough state to decide.
- **FR-006**: The questionnaire authoring UI MUST show approved schema controls, answer constraints, grading expectations, file-response rules, lifecycle lock state, validation errors, empty states, loading states, and safe denied states.
- **FR-007**: The questionnaire authoring UI MUST prevent edits that would change historical student-facing meaning when the approved lifecycle state marks the questionnaire as published, assigned, answered, graded, or otherwise locked.
- **FR-008**: The student response UI MUST show each assigned advanced question, required answer constraints, due-date state, one-attempt behavior, submission readiness, and safe unavailable states.
- **FR-009**: The student response UI MUST validate long-text answers as 1 to 10,000 characters, reject blank or whitespace-only answers, and surface invalid encoding or unsafe control-character feedback where returned by contract.
- **FR-010**: The student response UI MUST allow only one file per `file_response` question, approved PDF, image, text, and office categories, and files no larger than 25 MB where the contract exposes these limits.
- **FR-011**: The student response UI MUST preserve private file behavior by showing only approved filename, category, size, upload, scan, availability, and failure status metadata.
- **FR-012**: The student response UI MUST upload file-response files only with final assessment submission and MUST NOT create staged file uploads or backend partial answer state before final submission.
- **FR-013**: The student response UI MAY preserve unsubmitted text-answer drafts locally for the current user interaction, but MUST NOT persist backend draft responses, backend partial answer state, or draft file uploads before final submission.
- **FR-014**: The student response UI MUST block duplicate submissions, after-due-date submissions, unassigned questionnaires, inactive learning sets, inactive student profiles, mismatched school context, malformed answers, unsafe uploads, stale submissions, and partial-success states.
- **FR-015**: The UI MUST show file scan states as pending, clean, failed, unavailable, or blocked only where those states are documented, MUST hide download behavior until a clean file is authorized for the current actor, and MUST NOT provide inline file preview behavior in this slice.
- **FR-016**: The teacher and school-administrator review UI MUST list submitted responses using only approved server-side filters (`questionnaire_id`, `learning_set_id`, and `grading_status` in the current contract), display scan status and safe student identity fields where returned, and MUST NOT invent response-status or scan-status query parameters until OpenAPI documents them.
- **FR-017**: The grading UI MUST support manual grading for `long_text` and `file_response` answers using the approved 0-100 score range, grading status, student-visible feedback summary, and conflict handling.
- **FR-018**: The grading UI MUST distinguish auto-gradable existing answers from manually graded advanced answers and MUST render unsubmitted, submitted, scan-blocked, scan-failed, needs-review, graded, returned, and exempted states only where approved.
- **FR-019**: The grading UI MUST expose both zero-score and exempt actions for authorized failed-scan file-response answers and MUST hide those actions for pending, clean, unavailable, unauthorized, inactive, deleted, or stale file states.
- **FR-020**: The grading UI MUST allow authorized graders to view `long_text` answer content only within approved review and grading surfaces, and MUST hide private grading notes, hidden answer keys, rubric internals, private file storage paths, scanner internals, full payloads, unauthorized cross-tenant identifiers, and any answer text outside authorized grading, student-visible feedback, or explicitly approved display surfaces.
- **FR-021**: The student result UI MUST show only the authenticated student's approved submission status, grading status, score summary, student-visible teacher feedback summary, and safe file availability metadata.
- **FR-022**: The reporting UI MUST expose only approved advanced assessment counts, completion status, grading status, and score summaries in report catalog, report definition, report request, report history, and report result surfaces.
- **FR-023**: The reporting UI MUST block raw answer text, answer keys, uploaded answer files, file links, private file metadata, private grading notes, storage paths, unsupported joins, arbitrary query text, and cross-tenant references.
- **FR-024**: The UI MUST distinguish empty, loading, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, scan-pending, scan-failed, unavailable-file, stale-response, and temporary-unavailable states without relying on undocumented response details.
- **FR-025**: The UI MUST ignore or cancel stale advanced assessment responses when route, selected questionnaire, selected response, selected student, report filters, active school, authentication state, permission state, assignment state, due date, scan state, or grading state changes before a response applies.
- **FR-026**: The UI MUST keep advanced assessment routes, page titles, navigation labels, filters, empty states, denied states, and diagnostics distinct from existing non-advanced assessment, guardian, platform support, billing, messaging, notification, legal hold, anonymization, and purge behavior.
- **FR-027**: The UI MUST include test coverage or equivalent verification for authoring schemas, lifecycle locks, long-text validation, file restrictions, one-attempt submission, due-date blocking, scan visibility, failed-scan zero/exempt actions, review lists, manual grading, student results, reporting summaries, authorization denial, stale responses, safe errors, and unsupported actions.

### Key Entities *(include if feature involves data)*

- **AdvancedQuestionAuthoringView**: UI representation of an advanced questionnaire question, approved schema controls, answer constraints, grading expectations, file-response rules, sequence, lifecycle lock state, and validation feedback.
- **StudentAdvancedResponseView**: UI representation of one student's assigned advanced questionnaire response, due-date state, one-attempt state, local-only text draft state, final-submit file selection, submission status, and safe feedback.
- **ResponseFileAvailabilityView**: UI representation of an uploaded answer file's approved display metadata, scan status, availability state, and actor-specific clean-file download action.
- **AssessmentReviewQueueView**: Teacher or school-administrator view of submitted advanced responses, documented review filters, returned scan states, grading states, and safe student context.
- **ManualGradingView**: UI representation of grading state for manually graded advanced answers, score, outcome, failed-scan zero or exempt action, student-visible feedback summary, conflict state, and safe private-note boundary.
- **StudentAssessmentResultView**: Student-facing result summary for own advanced assessment status, score, feedback summary, and file availability metadata.
- **AdvancedAssessmentReportingView**: Reporting-facing representation of advanced assessment counts, completion status, grading status, score summaries, report-field availability, and blocked private fields.
- **AdvancedAssessmentSafeState**: UI state category for denied, unavailable, empty, invalid, stale, scan-pending, scan-failed, conflict, unsupported, not-found, loading, or tenant-safe feedback.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of advanced assessment authoring, submission, file-response, grading, student-result, and reporting UI behavior can be traced to approved contract behavior before frontend implementation begins.
- **SC-002**: In usability checks, an authorized teacher or school administrator can create a questionnaire with one long-text question and one file-response question in under 5 minutes without assistance.
- **SC-003**: In usability checks, a student can identify answer requirements, file limits, due-date status, one-attempt behavior, local-only draft behavior, and submit an assigned advanced assessment in under 6 minutes without assistance.
- **SC-004**: 100% of tested invalid long-text answers, unsupported files, oversized files, duplicate attempts, after-due-date attempts, and unassigned questionnaire submissions are blocked before or at submission with question-specific feedback.
- **SC-005**: 100% of tested pending, failed, unavailable, unauthorized, inactive, or deleted file-response attachments remain hidden from download actions, and 100% of clean authorized files avoid inline preview.
- **SC-006**: In usability checks, an authorized grader can find a submitted advanced response, identify scan and grading status, apply a valid manual score or failed-scan zero/exempt outcome, and return student-visible feedback in under 4 minutes.
- **SC-007**: 100% of tested unauthorized, forbidden, tenant-mismatch, validation, not-found, conflict, scan-pending, scan-failed, stale, unavailable, empty, and temporary-unavailable responses show safe feedback without exposing protected record existence.
- **SC-008**: 100% of tested student result views show only the student's own status, score summary, student-visible feedback summary, and safe file availability metadata.
- **SC-009**: 100% of tested report surfaces expose only assessment counts, completion status, grading status, and score summaries for advanced assessments.
- **SC-010**: 100% of tested diagnostics, labels, filenames, student/result/report surfaces, guardian/platform/support views, and automated test output omit raw answer text outside authorized grading displays, student-visible feedback, or another explicitly approved display surface, and omit uploaded file contents, private storage paths, answer keys, private grading notes, credentials, full payloads, and unauthorized cross-tenant details.
- **SC-011**: 100% of tested stale responses caused by route, questionnaire, response, student, report filter, school, authentication, permission, assignment, due-date, scan, or grading changes do not overwrite the current visible screen state.
- **SC-012**: Review confirms no UI surface consumes undocumented advanced assessment operations or exposes guardian, platform support, messaging, notification, billing, public file sharing, generated report file packaging, legal hold, anonymization, purge, or unsupported assessment behavior through this feature.

## Assumptions

- Backend feature `014-advanced-assessment-content` is the source of product truth for advanced question types, answer schemas, long-text limits, file-response limits, scan states, grading rules, student visibility, reporting fields, and tenant boundaries.
- Approved advanced question types for this frontend slice are `long_text` and `file_response`; other advanced question types remain out of scope.
- Long-text answers allow 1 to 10,000 characters and are treated as plain text only.
- File-response answers allow one PDF, image, text, or office file per question, up to 25 MB, with private storage and scan-gated availability.
- Students receive one submitted attempt per assigned questionnaire and cannot resubmit after successful submission.
- Submission closes at the learning-set due date; no separate assessment window is introduced by this frontend slice.
- Manual grading uses a 0-100 point scale for `long_text` and `file_response` answers, and students see only score/status plus teacher feedback summary.
- Existing authentication, active-school context, role-aware navigation, stale-response handling, safe error mapping, list/detail/status patterns, upload patterns, and reporting workspace patterns are reused.
- Guardian advanced assessment visibility, platform support diagnostics beyond minimized approved summaries, generated report file packaging, messaging, notifications, billing, public file sharing, permanent purge, legal hold, and anonymization remain outside this slice.
