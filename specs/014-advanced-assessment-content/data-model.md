# Data Model: Advanced Assessment and Content Types

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned records continue to use `school_id`.
- Public identifiers crossing API boundaries use UUIDs.
- Existing `multiple_choice`, `true_false`, and `short_text` questionnaire behavior remains compatible.
- Approved advanced question types are limited to `long_text` and `file_response`.
- `long_text` answers allow 1-10,000 characters and reject blank or whitespace-only content.
- `file_response` answers allow one PDF, image, text, or office file up to 25 MB per question.
- Student answer files use private tenant-scoped storage and remain unavailable until scan status is `clean`.
- Failed-scan file-response answers remain unavailable and may be graded zero or exempted by authorized teachers or school administrators.
- A student may submit one attempt per assigned questionnaire.
- Response submission closes at the assigned learning set due date; there is no separate assessment window in this slice.
- `long_text` and `file_response` answers are manually graded on a 0-100 point scale.
- Students may see score/status and teacher feedback summary; private grading notes remain hidden.
- Successful and denied answer-file download attempts are audited with tenant-safe metadata.
- Clean answer-file downloads are limited to owning or assigned teachers and school administrators with same-school assessment review authority.
- Reports expose only assessment counts, completion status, grading status, and score summaries.
- Raw answer text, answer files, file links, private file metadata, storage paths, answer keys, private grading notes, and unauthorized cross-tenant details must not appear in reports, guardian views, platform support views, or audit metadata.

## Entities

### AdvancedQuestionnaireQuestion

- **Purpose**: Existing questionnaire question expanded to represent approved advanced question types and answer schema metadata.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `questionnaire_id`
  - `question_type` (`multiple_choice`, `true_false`, `short_text`, `long_text`, `file_response`)
  - prompt
  - sequence
  - answer schema metadata
  - grading rule metadata
  - visibility metadata
  - lifecycle compatibility metadata
  - timestamps
- **Relationships**:
  - belongs to `Questionnaire`
  - has many `AssessmentAnswer`
- **Validation rules**:
  - `long_text` question schema must declare 1-10,000 character answer bounds.
  - `file_response` question schema must declare one allowed file and must use approved file categories and 25 MB limit.
  - unsupported question types are rejected.
  - question type, prompt, answer schema, grading rule, file rule, visibility, or sequence changes that alter historical student-facing meaning are rejected after publication, assignment, response submission, grading, or report generation.

### AssessmentAnswerSchema

- **Purpose**: Contract-defined validation shape for each approved question type.
- **Core fields**:
  - question type
  - required answer fields
  - allowed value rules
  - long-text length bounds
  - file category rules
  - file count and size rules
  - grading eligibility
  - student visibility rules
  - report visibility rules
- **Relationships**:
  - belongs to `AdvancedQuestionnaireQuestion`
- **Validation rules**:
  - must be expressible in OpenAPI before backend exposure
  - must reject unsupported fields and malformed shapes
  - report visibility for advanced responses is limited to counts, completion status, grading status, and score summaries

### AssessmentResponseAttempt

- **Purpose**: School-owned submission attempt by one student for one assigned questionnaire through one learning set and academic period.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `student_profile_id`
  - `questionnaire_id`
  - `learning_set_id`
  - `academic_period_id`
  - submission state (`submitted`, `scan_blocked`, `scan_failed`, `needs_review`, `partially_graded`, `graded`, `returned`, `exempted`)
  - submitted timestamp
  - grading status summary
  - score summary where available
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to `StudentProfile`
  - belongs to `Questionnaire`
  - belongs to `LearningSet`
  - belongs to `AcademicPeriod`
  - has many `AssessmentAnswer`
  - has many `AssessmentGradingOutcome`
  - has many `AuditEvent`
- **Validation rules**:
  - requires active same-school student profile
  - requires active assigned learning set and active questionnaire entry
  - requires a learning-set due date that has not passed
  - must reject duplicate submitted attempts for the same student and assigned questionnaire
  - must reject unassigned, cross-tenant, inactive, deleted, after-due-date, malformed, or unsupported answer payloads without partial persistence

### AssessmentAnswer

- **Purpose**: One student's answer to one questionnaire question.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `response_attempt_id`
  - `question_id`
  - `question_type`
  - answer value metadata
  - validation outcome
  - grading status
  - visibility state
  - timestamps
- **Relationships**:
  - belongs to `AssessmentResponseAttempt`
  - belongs to `AdvancedQuestionnaireQuestion`
  - may have one `AssessmentFileAttachment`
  - may have one or more `AssessmentGradingOutcome`
- **Validation rules**:
  - `long_text` answer must contain 1-10,000 non-whitespace characters
  - `file_response` answer must have at most one same-school file attachment
  - unsupported answer fields are rejected
  - answer content must not appear in reports, guardian views, platform support diagnostics, or audit metadata

### AssessmentFileAttachment

- **Purpose**: Private tenant-scoped uploaded student answer file.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `assessment_answer_id`
  - original sanitized filename
  - declared content type
  - detected content type
  - file category
  - file size bytes
  - private storage reference
  - scan status (`pending`, `clean`, `failed`)
  - availability state
  - uploaded timestamp
  - scanned timestamp where available
- **Relationships**:
  - belongs to `AssessmentAnswer`
  - belongs to `School`
- **Validation rules**:
  - allowed categories: PDF, image, text, office
  - maximum file size: 25 MB
  - one file per question
  - executable/archive files are rejected
  - declared and detected type mismatch is rejected
  - unsafe filenames are sanitized or rejected according to OpenAPI
  - `pending` and `failed` files are unavailable to teacher review, student display, report output, platform summary, support diagnostics, and download
  - failed-scan file-response answers may receive zero or exempt grading outcomes without exposing the file
  - `clean` files may be downloaded only by owning or assigned teachers and school administrators with same-school assessment review authority
  - private storage paths and keys are never exposed in API responses

### AssessmentGradingOutcome

- **Purpose**: Teacher or school-administrator grading result for an advanced answer or response attempt.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `response_attempt_id`
  - `assessment_answer_id`
  - grader user ID
  - grading status (`needs_review`, `graded`, `returned`, `exempted`)
  - score, 0-100 where applicable
  - student-visible teacher feedback summary
  - private grading note metadata where retained internally
  - graded timestamp
- **Relationships**:
  - belongs to `AssessmentResponseAttempt`
  - belongs to `AssessmentAnswer`
  - belongs to grader `User`
  - has audit events
- **Validation rules**:
  - `long_text` and `file_response` grading uses 0-100 points
  - scores outside 0-100 are rejected
  - requires authorized teacher ownership/assignment or explicit same-school school-administrator authority
  - pending scan files block file-response grading that depends on file content
  - failed scan files allow only zero or exempt grading outcomes unless a future specification approves replacement upload behavior
  - teacher feedback summary is student-visible for the owning student
  - private grading notes are not exposed in student, guardian, report, platform, support, or audit payloads

### Questionnaire

- **Purpose**: Existing school-owned teacher-authored assessment container.
- **Relationships**:
  - has many `AdvancedQuestionnaireQuestion`
  - may appear in `LearningSetEntry`
  - has many `AssessmentResponseAttempt`
- **Validation rules**:
  - existing lifecycle states and restore-to-inactive behavior remain authoritative
  - after use by published or assigned learning sets, edits that change historical student-facing meaning are rejected
  - unsupported question type additions are rejected

### LearningSet

- **Purpose**: Existing school-owned assignment context that determines whether a student may submit answers for a questionnaire entry.
- **Relationships**:
  - contains questionnaire entries
  - has roster-aware or legacy student assignments according to existing workflow rules
  - gates `AssessmentResponseAttempt` eligibility
- **Validation rules**:
  - response submission requires active same-school assignment
  - inactive or deleted learning sets block new submissions
  - existing lifecycle and historical visibility rules remain compatible

### StudentProfile

- **Purpose**: Existing school-owned student profile linked to authenticated student user.
- **Relationships**:
  - has many `AssessmentResponseAttempt`
- **Validation rules**:
  - response submission and student self-view require authenticated user linked to active same-school profile
  - other-student response access is rejected without exposing protected record existence

### TeacherAssignment

- **Purpose**: Existing roster/teaching authority record used for teacher review and grading where school-administrator authority is not used.
- **Validation rules**:
  - teacher review and grading require ownership or active same-school teaching assignment to relevant learning context
  - school administrators may review same-school assessments only where OpenAPI grants authority

### ReportCatalog

- **Purpose**: Existing school-scoped catalog of approved report fields.
- **Validation rules**:
  - may expose only assessment counts, completion status, grading status, and score summaries for advanced assessments
  - must reject raw answer text, answer keys, raw uploaded files, file links, private file metadata, storage paths, private grading notes, unsupported joins, arbitrary query text, and cross-tenant references

### AuditEvent

- **Purpose**: Tenant-safe record of authoring, submission, upload, scan, review, grading, answer-file download, reporting, validation, authorization, and conflict outcomes.
- **Core fields**:
  - actor user ID where available
  - action
  - outcome
  - target school ID
  - target type and safe target ID
  - correlation ID
  - tenant-safe reason code
  - timestamp
  - minimized metadata
- **Validation rules**:
  - must not store raw answer text, uploaded file contents, private storage paths, credentials, hidden answer keys, private grading notes, full request payloads, or unauthorized cross-tenant details
  - denied cross-tenant attempts are audited without storing unauthorized protected target details

## State Transitions

### AssessmentResponseAttempt

```text
submitted -> scan_blocked
submitted -> scan_failed
submitted -> needs_review
scan_blocked -> needs_review
scan_blocked -> scan_failed
needs_review -> partially_graded
needs_review -> graded
partially_graded -> graded
needs_review -> returned
partially_graded -> returned
needs_review -> exempted
scan_failed -> graded
scan_failed -> exempted
```

- A submitted attempt is created once per student and assigned questionnaire.
- `scan_blocked` applies when at least one file-response attachment is pending and grading/review depends on it.
- `scan_failed` applies when at least one file-response attachment has failed malware scanning.
- `needs_review` applies when manual grading is required and all required files are clean or no file grading dependency exists.
- `partially_graded` applies when some but not all manually graded answers have accepted grading outcomes.
- `graded` applies when all required grading outcomes are complete.
- `returned` exposes only approved student-safe status, score summary, teacher feedback summary, and file availability metadata.
- `exempted` represents an approved grading outcome without a numeric score where OpenAPI allows exemption.

### AssessmentFileAttachment

```text
pending -> clean
pending -> failed
```

- `pending` and `failed` files are unavailable for review, grading, reports, platform summaries, support diagnostics, and download.
- `clean` makes the file eligible for authorized review or grading only; it does not make the file report-visible or publicly downloadable.
- `failed` allows zero or exempt grading outcomes for the affected file-response answer without exposing the file.

### AssessmentGradingOutcome

```text
needs_review -> graded
needs_review -> returned
needs_review -> exempted
graded -> returned
```

- Manual grading for `long_text` and `file_response` uses 0-100 points.
- Failed-scan file-response answers may be graded as zero or exempted.
- Grading attempts without authority, wrong school context, scan-blocked files, unsupported status, or invalid score are rejected.

## Reporting Visibility Matrix

| Data | Student Own View | Teacher/Admin Review | Reports | Guardian | Platform/Support |
|------|------------------|----------------------|---------|----------|------------------|
| Submission status | Allowed | Allowed if authorized | Summary only | Out of scope | Minimized diagnostics only if separately approved |
| Grading status | Allowed | Allowed if authorized | Summary only | Out of scope | Minimized diagnostics only if separately approved |
| Score summary | Allowed | Allowed if authorized | Summary only | Out of scope | Minimized diagnostics only if separately approved |
| Teacher feedback summary | Allowed | Allowed if authorized | Not allowed | Not allowed | Not allowed |
| Long-text answer content | Student own only where documented | Authorized review only | Not allowed | Not allowed | Not allowed |
| File attachment content | Student own safe metadata only | Clean authorized review/download only | Not allowed | Not allowed | Not allowed |
| Answer keys | Not allowed | Hidden except authoring/review rules | Not allowed | Not allowed | Not allowed |
| Private grading notes | Not allowed | Internal authorized view only | Not allowed | Not allowed | Not allowed |
