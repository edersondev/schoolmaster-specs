# Feature Specification: Advanced Assessment and Content Types

**Feature Branch**: `014-advanced-assessment-content`  
**Created**: 2026-06-11  
**Status**: Draft  
**Input**: User description: "Run speckit-specify for backend roadmap item 9: Advanced Assessment and Content Types. Add questionnaire types beyond multiple choice, true/false, and short text; support long-text or file-response answers; expand content processing where needed. Define answer schemas, grading rules, file handling, malware scanning, storage visibility, and student/reporting impact. Preserve contract-first API-only `/api/v1` behavior, School as the v1 tenant root, `school_id` as the concrete tenant column for school-owned records, explicit platform versus school authorization separation, tenant-safe file handling, OpenAPI expansion before backend implementation, and compatibility with completed teacher workflow, student self-view, report lifecycle, and platform support access slices."

## Clarifications

### Session 2026-06-11

- Q: Which file categories, size limit, and count should file-response answers allow? → A: PDF, image, text, and office files; maximum 25 MB; one file per question.
- Q: How should advanced assessment answers be graded? → A: `long_text` and `file_response` answers are manually graded on a 0-100 point scale; legacy auto-gradable answers keep existing behavior.
- Q: How many submission attempts may a student make for one assigned advanced questionnaire? → A: One submission attempt per student per assigned questionnaire; no resubmission after submit.
- Q: What advanced assessment data may reports expose? → A: Reports expose assessment counts, completion status, grading status, and score summaries only.
- Q: What length limits apply to long-text answers? → A: Long-text answers allow 1-10,000 characters; blank or whitespace-only answers are rejected.
- Q: What submission window applies to advanced questionnaire responses? → A: Submission closes at the learning-set due date; there is no separate assessment window in this slice.
- Q: What feedback may students see after advanced assessment grading? → A: Students see score/status plus teacher feedback summary; private grading notes remain hidden.
- Q: How should failed malware scans affect file-response answers? → A: Failed scan blocks the file-response answer; teacher or school administrator may grade it as zero or exempt it.
- Q: Which answer-file download attempts should be audited? → A: Audit both successful and denied answer-file download attempts.
- Q: Who may download clean answer files for review? → A: Owning or assigned teacher and school administrator may download clean answer files.
- Q: What does unsafe long-text answer content mean in this slice? → A: Long-text answers are stored and returned as plain text, never interpreted as markup or executable content; invalid encoding or unsafe control characters are rejected, while content moderation is out of scope.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Author Advanced Assessment Questions (Priority: P1)

An authorized teacher or school administrator creates and maintains questionnaires that include advanced question types while preserving existing v1 multiple-choice, true/false, and short-text behavior.

**Why this priority**: Advanced assessment authoring is the source of new answer schemas, validation rules, grading behavior, and downstream student/reporting impact. Nothing else can safely consume advanced assessments until the contract defines the allowed question types.

**Independent Test**: Authenticate with an active permitted school context, create a questionnaire with existing v1 questions plus advanced questions, retrieve it, update editable fields while unused, attach it to a learning set, then verify unsupported question shapes, answer schemas, cross-tenant references, and historical-meaning edits are rejected.

**Acceptance Scenarios**:

1. **Given** an authorized teacher has an active school context, **When** they create a questionnaire using existing v1 question types and approved advanced question types, **Then** the questionnaire is stored for that school with ordered questions, validated schemas, and no unsupported fields.
2. **Given** a questionnaire contains advanced questions, **When** it is listed or retrieved by an authorized same-school actor, **Then** responses include only the OpenAPI-approved question metadata for that actor and do not expose hidden grading keys or private file storage details.
3. **Given** a questionnaire has been used by a published or assigned learning set, **When** an actor attempts to change a question type, prompt, answer schema, grading rule, file rule, or sequence in a way that changes historical student-facing meaning, **Then** the change is rejected according to existing lifecycle rules.
4. **Given** a request submits an unsupported question type, malformed answer schema, invalid grading rule, unsafe file-response rule, or cross-tenant reference, **When** questionnaire creation or update is attempted, **Then** validation rejects the request before persistence.

---

### User Story 2 - Submit Student Advanced Assessment Responses (Priority: P2)

An authenticated student submits advanced questionnaire answers only for assigned same-school learning sets, including long-text answers and file-response answers where allowed by the questionnaire.

**Why this priority**: Student submission rules define the privacy, storage, scanning, ownership, and answer lifecycle behavior that advanced assessment introduces beyond authored questionnaire metadata.

**Independent Test**: Authenticate as a student with an active same-school profile and assigned learning set, submit allowed long-text and file-response answers, verify pending file scans prevent teacher grading and reporting exposure until clean, and verify unassigned, cross-tenant, expired, malformed, blank, duplicate, or unsafe submissions are rejected.

**Acceptance Scenarios**:

1. **Given** an authenticated student has an active same-school profile and an assigned active learning set containing an advanced questionnaire, **When** they submit valid answers within the allowed window, **Then** the response attempt is recorded for that student, questionnaire, learning set, and school.
2. **Given** a file-response answer is allowed, **When** the student uploads an approved file, **Then** the file is stored privately for the resolved school with scan status `pending` and is unavailable for teacher review, reporting, or download until an authorized scan workflow marks it `clean`.
3. **Given** a submitted answer references a questionnaire not assigned to the student, another student's response, another school, an inactive/deleted learning set, or an unavailable question, **When** submission is attempted, **Then** the request is denied or rejected without exposing protected record existence.
4. **Given** a student already submitted one attempt for an assigned questionnaire, attempts to submit after the learning-set due date, uploads an unsupported file, exceeds response limits, or submits a malformed answer schema, **When** the request is processed, **Then** no partial answer state is persisted.

---

### User Story 3 - Grade and Review Advanced Responses (Priority: P3)

An authorized teacher or school administrator reviews advanced assessment responses, applies manual grading where required, and keeps answer files, grading notes, and audit history tenant-safe.

**Why this priority**: Long-text and file-response questions cannot rely on the existing narrow automatic grading model. Explicit grading rules are needed before grades, student views, and reports can trust advanced responses.

**Independent Test**: Create assigned advanced responses, mark file submissions clean, review student answers as an authorized teacher or school administrator, apply manual grading from 0 to 100 points, verify auto-gradable legacy questions still grade according to existing rules, and verify unauthorized, unclean, cross-tenant, or stale grading attempts are rejected.

**Acceptance Scenarios**:

1. **Given** a teacher owns or is assigned to the relevant same-school learning set or a school administrator has same-school assessment review authority, **When** they review submitted advanced responses, **Then** they can see only authorized same-school responses and download clean file-response attachments.
2. **Given** a `long_text` or `file_response` answer requires manual grading, **When** an authorized actor submits a valid 0-100 point score, feedback, and grading status, **Then** the grading outcome is recorded with actor, timestamp, student, questionnaire, question, and school context.
3. **Given** a file-response attachment has scan status `pending` or `failed`, **When** a teacher, administrator, report, or student view attempts to expose it, **Then** the file remains unavailable and the response exposes only documented safe status metadata; failed-scan answers may be graded as zero or exempted by an authorized teacher or school administrator.
4. **Given** an actor attempts grading without teacher assignment, owner authority, school administrator authority, or correct school context, **When** the grading request is processed, **Then** access is denied without revealing cross-tenant response details.

---

### User Story 4 - Reflect Advanced Assessments in Student Views and Reports (Priority: P4)

Students, teachers, and reporting administrators see advanced assessment outcomes only through documented fields that preserve answer privacy, file security, and historical meaning.

**Why this priority**: Advanced assessment changes student experience and report inputs. Reports must avoid leaking answer keys, private files, unscanned uploads, grading notes, or cross-tenant details.

**Independent Test**: Submit and grade advanced responses, then verify student learning timeline, student grade/assessment view, teacher review views, report catalog, custom report definitions, and generated reports expose only approved advanced assessment summary fields and reject hidden/private fields.

**Acceptance Scenarios**:

1. **Given** a student's advanced response has a documented grading outcome, **When** the student views their permitted assessment or academic summary, **Then** the response shows only approved current status, score, teacher feedback summary, and file availability metadata.
2. **Given** report catalog or custom report definitions include assessment domains, **When** a reporting administrator builds or requests a report, **Then** only assessment counts, completion status, grading status, and score summaries are available; private answer content, answer keys, private attachments, storage paths, and grading notes are excluded.
3. **Given** an advanced response includes unscanned, failed-scan, deleted, inactive, or unauthorized file attachments, **When** any student, teacher, guardian, report, platform, or support view is returned, **Then** the file content and private storage metadata remain hidden.
4. **Given** platform support access is used, **When** support diagnostics reference advanced assessments, **Then** only minimized diagnostics approved by the support contract may appear; platform users receive no implicit access to student answers or files.

### Edge Cases

- Which advanced question types are approved in this slice, and which remain rejected until a later contract expansion?
- How does the backend reject missing, inactive, mismatched, or unauthorized school context before advanced questionnaire, response, grading, file, or report data is accessed?
- What happens when advanced questionnaire updates would change historical student-facing meaning after assignment, publication, response submission, grading, or report generation?
- How does file-response handling reject unsupported file categories, executable/archive files, files over the approved limit, declared/detected content-type mismatches, malware-scan failures, duplicate uploads, and unsafe filenames?
- What happens when malware scanning is delayed, fails, blocks a file-response answer, or races with teacher review, grading, report generation, student view, delete/restore, or support diagnostics?
- How are long-text answers shorter than 1 character, longer than 10,000 characters, blank responses, whitespace-only responses, invalid encoding, unsafe control characters, plain-text output handling, and answer resubmission rules enforced?
- How does grading handle mixed auto-gradable, manually graded, ungraded, partially graded, exempted, or scan-blocked responses in the same questionnaire?
- How does the system prevent answer keys, rubric internals, private grading notes, raw student answer files, private storage paths, and unauthorized cross-tenant identifiers from appearing in student, guardian, report, platform, or support responses?
- How do advanced responses behave when the learning set, questionnaire, teacher assignment, roster membership, student profile, academic period, or school becomes inactive or deleted after submission?
- How are concurrent student submissions, file scans, grading attempts, questionnaire lifecycle changes, and report generation handled without partial or contradictory state?
- How does this slice avoid introducing frontend behavior, messaging, notifications, billing, unrestricted file sharing, public file URLs, permanent purge, legal hold, anonymization workflows, or undocumented APIs?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Adds backend-only advanced questionnaire schema validation, student response submission, private file-response handling, malware-scan gating, grading workflow, response visibility rules, audit events, and regression coverage after OpenAPI approval.
- **Frontend repository impact**: No frontend implementation in this slice. Future teacher, student, and reporting screens must consume only the approved OpenAPI contract after backend behavior is implemented.
- **Specification or contract repository impact**: Requires this feature specification and OpenAPI expansion before backend exposes advanced question types, student response submission, file-response upload/download, grading, or report-field behavior.
- **Delivery ownership and sequencing**: `schoolmaster-specs` leads product and contract definition, `schoolmaster-backend` implements only approved contract behavior next, and `schoolmaster-frontend` remains deferred until a separate frontend delivery consumes the approved API.

### API Contract Impact

- **OpenAPI update required**: Yes. Define advanced question type schemas, questionnaire request/response changes, student response submission, file-response upload handling, response review, grading, file availability, report catalog impact, audit-sensitive states, and error responses before backend implementation.
- **Versioned endpoints affected**: Existing `/api/v1/questionnaires`, `/api/v1/questionnaires/{questionnaireId}`, `/api/v1/learning-sets`, `/api/v1/student/learning-sets`, `/api/v1/reports`, `/api/v1/report-catalog`, and `/api/v1/report-definitions` behavior must remain compatible; new or expanded `/api/v1/student/questionnaire-responses`, `/api/v1/questionnaire-responses`, and documented subresources for response files or grading may be added only through OpenAPI.
- **JSON response impact**: Adds documented answer schema metadata, response attempt state, file scan/availability state, grading status, score summary, validation errors, conflict responses, tenant-safe denials, and report catalog fields. Responses must not include private storage paths, raw answer files, hidden answer keys, private grading notes, or unauthorized cross-tenant details.
- **Authentication/authorization impact**: All operations require authenticated access and active permitted school context. Teachers may manage or grade only owned/assigned same-school assessments; school administrators may review same-school assessments where explicitly permitted; students may submit and view only their own assigned assessments. Platform and support users receive no implicit access to advanced answers or files.
- **Compatibility impact**: Additive contract change. Existing v1 question types, questionnaire lifecycle behavior, learning-set timelines, student content downloads, report lifecycle behavior, and support access behavior must remain compatible unless a future approved contract explicitly changes them.

### Data & Tenancy Impact

- **Tenant scoping impact**: Advanced questionnaires, questions, answer schemas, response attempts, answers, file-response attachments, grading outcomes, grading feedback, scan states, report summary fields, and audit events are school-owned records governed by `school_id`.
- **Cross-tenant or platform access impact**: Cross-school questionnaires, learning sets, student profiles, response attempts, file attachments, grading outcomes, report fields, and audit events must be rejected before data exposure. Platform support access remains limited to separately approved minimized diagnostics.
- **Soft delete impact**: Questionnaire and learning-set lifecycle rules from teacher workflow lifecycle remain authoritative. Response attempts, answer files, grading outcomes, and audit events must preserve historical meaning and student/report references; permanent purge, anonymization, legal hold, and retention override are outside this slice.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST NOT expose advanced assessment question types, student response submission, file-response uploads, response review, grading, or advanced assessment report fields until OpenAPI defines operation IDs, request schemas, response schemas, state values, authorization rules, validation errors, conflict responses, and tenant behavior.
- **FR-002**: Every operation in this slice MUST resolve an active permitted school context before questionnaire lookup, response lookup, learning-set lookup, student lookup, teacher-assignment lookup, file access, grading, report catalog expansion, persistence, audit recording, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any advanced questionnaire, response, answer, file, grading, report, student-profile, teacher-assignment, learning-set, audit, or storage data is accessed.
- **FR-004**: Advanced question support MUST preserve existing `multiple_choice`, `true_false`, and `short_text` behavior and MUST reject unsupported question types or answer shapes not documented by OpenAPI.
- **FR-005**: The approved advanced question type set for this slice is limited to `long_text` and `file_response`; numeric, date, rating, multi-select, matching, ordering, media-response, and any other advanced question types remain out of scope until a future specification and OpenAPI expansion approve them.
- **FR-006**: Long-text answers MUST allow 1 to 10,000 characters, reject blank or whitespace-only answers, reject invalid encoding or unsafe control characters that cannot be safely stored, store and return answer content as plain text that is never interpreted as markup or executable content, and define documented resubmission behavior, grading eligibility, and student/report visibility before implementation; content moderation is out of scope for this slice.
- **FR-007**: File-response answers MUST allow only PDF, image, text, and office files up to 25 MB, with one file per question; they MUST define declared/detected type validation, filename sanitization, private tenant storage, scan status values, and safe failure responses before implementation.
- **FR-008**: File-response attachments MUST be unavailable for teacher review, grading, student display, report output, platform summary, support diagnostics, or download until an authorized malware-scan workflow marks them `clean`.
- **FR-009**: File-response scan status MUST reject or hide `pending`, `failed`, unavailable, inactive, deleted, cross-tenant, unauthorized, or otherwise unsafe files without exposing private storage paths or cross-tenant file existence; failed-scan file-response answers MAY be graded as zero or exempted by an authorized teacher or school administrator.
- **FR-010**: Student response submission MUST require an active same-school student profile, an active assigned learning set, an active questionnaire entry, a learning-set due date that has not passed, no prior submitted attempt for the same student and assigned questionnaire, and a valid answer schema for every submitted question.
- **FR-011**: Student response submission MUST reject unassigned questionnaires, cross-tenant learning sets, inactive/deleted records, unsupported answer schemas, unsafe file uploads, duplicate attempts, attempts after the learning-set due date, and malformed payloads without partial persistence.
- **FR-012**: Response attempts MUST preserve student, school, questionnaire, learning-set, academic-period, submitted answers, file attachment metadata, scan states, grading status, timestamps, and audit history needed for accountability.
- **FR-013**: Questionnaire updates MUST continue to reject edits that change historical student-facing meaning after use by a published or assigned learning set, and this lock MUST include advanced question type, prompt, answer schema, grading rule, file rule, sequence, and visibility changes.
- **FR-014**: Grading MUST distinguish auto-gradable legacy answers from manually graded `long_text` and `file_response` answers, and MUST expose documented states for unsubmitted, submitted, scan-blocked, scan-failed, needs-review, graded, returned, and exempted outcomes where approved by OpenAPI.
- **FR-015**: Manual grading for `long_text` and `file_response` answers MUST use a 0-100 point score, require authorized teacher ownership/assignment or explicit same-school school-administrator authority, and MUST record actor, timestamp, target response, grading outcome, score or status, student-visible teacher feedback summary, and private grading notes where supported internally.
- **FR-016**: Grading and response review MUST allow clean answer-file downloads only for the owning or assigned teacher and school administrators with same-school assessment review authority, while hiding private storage paths, scanner internals, hidden answer keys, private rubric internals, private grading notes, credentials, full audit payloads, and unauthorized cross-tenant identifiers from all responses.
- **FR-017**: Student-visible assessment responses MUST expose only approved current submission status, grading status, score summary, student-visible teacher feedback summary, and file availability metadata for the authenticated student's own same-school responses; private grading notes remain hidden.
- **FR-018**: Guardian-visible behavior MUST remain unchanged unless a separate approved guardian specification and OpenAPI contract explicitly grants advanced assessment visibility.
- **FR-019**: Report catalog and custom report definitions MAY include only assessment counts, completion status, grading status, and score summaries for advanced assessments, and MUST reject hidden answer content, answer keys, raw file attachments, private file metadata, storage paths, private grading notes, unsupported joins, arbitrary query text, and cross-tenant references.
- **FR-020**: Generated reports MUST include only assessment counts, completion status, grading status, and score summaries for advanced assessments and MUST NOT embed raw answer text, uploaded answer files, or file links unless a future report output specification explicitly approves secure file packaging.
- **FR-021**: Platform reporting and support access MUST NOT receive implicit access to advanced questionnaire answers, response files, grading notes, answer keys, or detailed student response records. Any platform/support exposure remains limited to separately approved minimized diagnostics.
- **FR-022**: Audit events MUST record advanced questionnaire authoring, response submission, file upload, scan outcome, review, grading, successful answer-file download, denied answer-file download, denied access, validation failure, lifecycle conflict, report-field access, and blocked cross-tenant outcomes with actor, action, outcome, target, school, correlation ID, and tenant-safe reason code.
- **FR-023**: Audit events MUST NOT store raw answer text, uploaded file contents, private storage paths, credentials, hidden answer keys, private grading notes, student-visible feedback text, full request payloads, or unauthorized cross-tenant details.
- **FR-024**: Concurrent submissions, file scans, grading attempts, lifecycle changes, and report generation MUST use documented conflict or first-valid-transition semantics and MUST not create partial answer, file, grading, or report state.
- **FR-025**: Backend responses MUST match the expanded OpenAPI success, accepted, binary where explicitly approved, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, scan-pending, scan-failed, and unavailable-file semantics for every approved operation.
- **FR-026**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, inactive school/user/student/teacher/period handling, advanced question schema validation, file validation, malware-scan gating, student assignment boundaries, grading authority, report catalog boundaries, audit coverage, concurrency conflicts, and response shapes.
- **FR-027**: Backend implementation MUST NOT expose frontend behavior, messaging, notifications, billing, payroll, accounting, unrestricted file sharing, public file URLs, generated report file packaging, permanent purge, legal hold, anonymization workflows, emergency support access, or undocumented APIs in this slice.

### Key Entities *(include if feature involves data)*

- **AdvancedQuestionnaireQuestion**: Existing questionnaire question expanded with approved advanced question type, answer schema, grading rule, visibility rule, and sequence while preserving school ownership and historical-meaning locks.
- **AssessmentAnswerSchema**: Contract-defined validation shape for each approved question type, including required fields, allowed values, answer limits, grading eligibility, and student/report visibility.
- **AssessmentResponseAttempt**: School-owned submission attempt by one student for one assigned questionnaire through one learning set and academic period, including submission state, timestamps, grading state, and audit references.
- **AssessmentAnswer**: One student's answer to one questionnaire question, including answer value metadata, validation outcome, file attachment references where allowed, grading status, and visibility state.
- **AssessmentFileAttachment**: Private tenant-scoped uploaded student answer file with allowed file metadata, declared/detected type, size, scan status, storage visibility, and safe availability state.
- **AssessmentGradingOutcome**: Teacher or administrator grading result for an answer or response attempt, including score/status, student-visible teacher feedback summary, private grading note boundary, actor, timestamp, and tenant-safe audit metadata.
- **Questionnaire**: Existing school-owned teacher-authored assessment container whose lifecycle, ownership, and historical-meaning rules remain authoritative.
- **LearningSet**: Existing school-owned assignment context that determines whether a student may submit answers for a questionnaire entry.
- **StudentProfile**: Existing school-owned student profile; response submission and student self-view require the authenticated user to be linked to the active same-school profile.
- **TeacherAssignment**: Existing roster/teaching authority record used to authorize teacher review and grading where school administrator authority is not used.
- **ReportCatalog**: Existing school-scoped catalog that may expose only assessment counts, completion status, grading status, and score summaries for advanced assessment reports.
- **AuditEvent**: Tenant-safe record of authoring, submission, upload, scan, review, grading, answer-file download, reporting, validation, authorization, and conflict outcomes.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Contract review maps 100% of new advanced assessment operations, schemas, state values, and responses to approved OpenAPI definitions before backend implementation starts.
- **SC-002**: Tenant-isolation tests reject 100% of missing-school, inactive-school, unauthorized-school, and cross-school attempts for advanced questionnaires, responses, files, grading, and report fields.
- **SC-003**: Schema validation tests reject 100% of unsupported advanced question types, malformed answer schemas, unsupported answer fields, blank or whitespace-only long-text answers, long-text answers over 10,000 characters, invalid long-text encoding or unsafe control characters, invalid grading rules, and unsupported visibility settings covered by this specification.
- **SC-004**: Student authorization tests reject 100% of unassigned, other-student, inactive-profile, duplicate-attempt, after-due-date, inactive-learning-set, inactive-questionnaire, and cross-tenant response submission attempts.
- **SC-005**: File-response tests reject 100% of unsupported file categories, unsafe filenames, executable/archive files, files over 25 MB, multiple files for one question, declared/detected content-type mismatches, and unsafe scan states covered by the contract.
- **SC-006**: Malware-scan gating tests confirm 100% of `pending` or `failed` answer files remain unavailable to teacher review, student display, reports, platform summaries, support diagnostics, and downloads; clean answer-file downloads are allowed only for owning or assigned teachers and school administrators with same-school assessment review authority; failed-scan answers can only receive zero or exempt grading outcomes from authorized actors.
- **SC-007**: Grading tests confirm 100% of manually graded advanced responses require approved actor authority, reject scores outside 0-100 points, and preserve grading actor, timestamp, status, score or outcome, and tenant-safe audit metadata.
- **SC-008**: Student visibility tests confirm students see only their own approved assessment submission status, grading status, score summary, teacher feedback summary, and safe file availability metadata, while private grading notes remain hidden.
- **SC-009**: Report catalog tests confirm only assessment counts, completion status, grading status, and score summaries are available for advanced assessment reports, while private answer text, answer keys, raw uploaded files, file links, private file metadata, storage paths, private grading notes, and unsupported advanced assessment fields are rejected from report definitions and generated report outputs.
- **SC-010**: Audit verification confirms 100% of authoring, submission, upload, scan, grading, successful answer-file download, denied answer-file download, denied access, validation, conflict, and blocked cross-tenant outcomes create tenant-safe audit records without raw answer text, file contents, private storage paths, credentials, hidden answer keys, student-visible feedback text, or private grading notes.
- **SC-011**: Compatibility review confirms existing v1 question types, questionnaire lifecycle behavior, learning-set timelines, report lifecycle behavior, and platform support access behavior remain compatible unless an approved contract explicitly changes them.

## Assumptions

- `004-backend-teacher-workflows` established teacher content, questionnaires, learning sets, scan status, tenant-scoped teacher workflow patterns, and v1 question types.
- `005-backend-student-reporting` established student learning timelines, student content download behavior, student grades and attendance self-view, and launch reporting.
- `010-teacher-workflow-lifecycle` established questionnaire lifecycle, historical-meaning edit locks, teacher ownership/assignment boundaries, and soft-delete/restore behavior.
- `012-report-lifecycle-expansion` established report catalog and custom report definition rules that must exclude private or hidden assessment fields by default.
- `013-platform-support-access` keeps platform and support access minimized; support diagnostics do not imply access to student answers or private files.
- Advanced assessment is a backend API contract feature first; frontend implementation remains out of scope for this slice.
- Existing private tenant-scoped file storage, upload validation, and malware-scan gating patterns are reused for file-response answers, with the same launch-scope file categories and 25 MB maximum used for teacher content.
- Guardian advanced assessment visibility remains out of scope unless a future guardian specification grants it.
- Permanent purge, legal hold, anonymization, public file sharing, generated report file packaging, billing, messaging, notifications, and emergency support access remain outside this slice.
