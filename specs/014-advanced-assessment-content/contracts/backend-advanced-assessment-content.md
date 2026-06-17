# Contract Boundary: Advanced Assessment and Content Types

## Purpose

This feature adds a public API surface for advanced questionnaire question types, student advanced assessment response submission, file-response upload handling, response review, manual grading, student-safe assessment summaries, report-safe assessment aggregates, and audit coverage. Backend implementation must wait until OpenAPI documents exact operations, parameters, request schemas, response schemas, errors, tenant behavior, authorization behavior, state values, scan behavior, conflict semantics, and operation IDs.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend roadmap: `docs/backend-feature-roadmap.md`
- Feature spec: `specs/014-advanced-assessment-content/spec.md`

## Current Assessment and Content Baseline

Existing questionnaire behavior supports `multiple_choice`, `true_false`, and `short_text`; teacher content uploads already use private tenant storage and scan gating; student learning timelines and content downloads are assignment-scoped; report catalog and custom report definitions exclude private fields by default; platform support access is minimized and does not grant implicit school-owned detail access.

## Approved Operation Boundary

OpenAPI must define operation IDs and routes for this backend slice before implementation. Proposed operation mapping:

| Method | Route | Proposed Operation ID | Boundary |
|--------|-------|-----------------------|----------|
| `POST` | `/api/v1/questionnaires` | `createQuestionnaire` | Expand existing schema to allow `long_text` and `file_response` question definitions |
| `PATCH` | `/api/v1/questionnaires/{questionnaireId}` | `updateQuestionnaire` | Expand existing schema while preserving historical-meaning locks |
| `GET` | `/api/v1/questionnaires/{questionnaireId}` | `getQuestionnaire` | Return actor-appropriate advanced question metadata without hidden answer keys or storage details |
| `POST` | `/api/v1/student/questionnaire-responses` | `submitStudentQuestionnaireResponse` | Submit one response attempt for an assigned same-school questionnaire |
| `GET` | `/api/v1/student/questionnaire-responses/{responseAttemptId}` | `getStudentQuestionnaireResponse` | Retrieve own student-safe response status, grading status, score summary, teacher feedback summary, and file availability metadata |
| `GET` | `/api/v1/questionnaire-responses` | `listQuestionnaireResponses` | List same-school responses visible to authorized teacher or school administrator |
| `GET` | `/api/v1/questionnaire-responses/{responseAttemptId}` | `getQuestionnaireResponse` | Retrieve same-school response review data for authorized teacher or school administrator |
| `POST` | `/api/v1/questionnaire-responses/{responseAttemptId}/grading` | `gradeQuestionnaireResponse` | Record manual 0-100 grading for `long_text` and `file_response` answers |
| `GET` | `/api/v1/questionnaire-responses/{responseAttemptId}/files/{fileId}/download` | `downloadQuestionnaireResponseFile` | Download a clean answer file for owning or assigned teacher and school administrator review only |
| Existing | Report catalog operations and schemas | Existing report catalog operation IDs | Expand report catalog schemas only if OpenAPI exposes assessment aggregate fields through existing report catalog operations |

The exact route layout may differ, but implementation must map to documented OpenAPI operation IDs before backend routes are exposed.

## Required Contract Expansion

OpenAPI must define, at minimum:

- `QuestionType` expansion limited to `long_text` and `file_response`
- advanced question input schemas and response schemas
- long-text validation: 1-10,000 characters, blank and whitespace-only rejected
- file-response validation: PDF, image, text, and office files only; 25 MB maximum; one file per question
- declared/detected content-type validation and safe filename handling
- private file scan status and availability values
- failed-scan file-response handling with zero or exemption outcome only
- one-attempt-per-student-per-assigned-questionnaire behavior
- learning-set due date as the response submission deadline
- response attempt state values and transitions
- grading request and response schemas for 0-100 manual grading
- student-safe response summary schema with score/status and teacher feedback summary
- teacher/school-administrator response review schemas
- report catalog assessment aggregate fields limited to counts, completion status, grading status, and score summaries
- validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, scan-pending, scan-failed, unavailable-file, and unsupported-field response envelopes
- tenant-safe audit event expectations
- successful and denied answer-file download audit expectations
- explicit exclusions for guardian visibility, raw answer reporting, generated report file packaging, public file URLs, platform/support detail access, and undocumented APIs

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelopes for questionnaire, response attempt, response review, grading, and assessment summary responses
- accepted or created responses for response submission and file upload where applicable
- binary/file response only for authorized clean response-file review downloads where OpenAPI explicitly approves it
- validation error envelope
- unauthorized response for unauthenticated or inactive actor access
- forbidden response for authenticated actors without required same-school permission
- not-found or tenant-safe denial response for missing, unauthorized, mismatched, or cross-tenant targets according to contract
- conflict response for duplicate attempts, attempts after the learning-set due date, historical-meaning edit locks, stale lifecycle state, or concurrent grading/submission conflicts
- scan-pending, scan-failed, or unavailable-file responses for unsafe answer files

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented question type, undocumented answer field, undocumented report field, undocumented include expansion, undocumented sort/filter behavior, undocumented file access path, or authorization exception is approved in this slice.

## Tenant Behavior

- School-owned records continue to use `school_id`.
- Response submission requires an active same-school student profile and an active assigned same-school learning set with questionnaire entry.
- Teacher review and grading require same-school ownership/assignment or explicit school-administrator authority.
- Cross-school questionnaires, responses, files, grading outcomes, report fields, and audit events are rejected before protected data or file metadata is exposed.
- Platform/support actors do not receive implicit access to advanced answers, answer files, grading notes, or detailed response records.

## Authorization Behavior

- All operations require authenticated access and active actor status.
- Questionnaire authoring requires existing teacher/school-administrator authority.
- Student submission and own response retrieval require authenticated user linked to the active same-school `StudentProfile`.
- Teacher response review and grading require owning/assigned teacher authority for the relevant same-school learning context.
- School administrators may review and grade same-school responses only where OpenAPI explicitly grants permission.
- Clean answer-file download requires owning/assigned teacher authority or same-school school-administrator assessment review authority.
- Report catalog/admin behavior requires existing report-definition/report-catalog permission and may expose only approved aggregate assessment fields.
- Guardian access remains unchanged and does not include advanced assessment visibility.

## Validation Behavior

- Advanced question types other than `long_text` and `file_response` are rejected.
- `long_text` answers shorter than 1 character, longer than 10,000 characters, blank, or whitespace-only are rejected.
- `file_response` answers reject unsupported categories, unsafe filenames, executable/archive files, files over 25 MB, multiple files for one question, declared/detected type mismatches, pending/failed scan exposure, and cross-tenant file references.
- Duplicate submitted attempts for the same student and assigned questionnaire are rejected.
- Submission attempts outside active assignment, active questionnaire entry, active learning set, learning-set due date, or same-school context are rejected without partial persistence.
- Manual grading rejects scores outside 0-100, unauthorized graders, pending-scan file answers, stale response state, unsupported grading status, and cross-tenant response references; failed-scan file-response answers allow only zero or exempt outcomes.
- Questionnaire updates that change historical student-facing meaning after publication, assignment, response submission, grading, or report generation are rejected.
- Report catalog definitions reject raw answer text, answer keys, raw uploaded files, file links, private file metadata, storage paths, teacher feedback summaries, private grading notes, unsupported joins, arbitrary query text, and cross-tenant references.
- Audit metadata must reject or redact raw answer text, uploaded file contents, private storage paths, credentials, answer keys, student-visible feedback text, private grading notes, full request payloads, and unauthorized cross-tenant details.

## Blocked Until Future Specification

These behaviors are outside this implementation boundary until a future spec and OpenAPI update approve them:

- frontend advanced assessment UI implementation
- advanced question types beyond `long_text` and `file_response`
- resubmission, draft attempts, reopened attempts, or attempt versioning
- teacher-configurable attempt limits or long-text length limits
- rubric engines, per-question grading scales, AI grading, or automatic grading for advanced answers
- audio/video/media responses, archive uploads, multiple files per question, public file URLs, or file sharing outside authorized review
- guardian advanced assessment visibility
- raw answer text, file links, uploaded files, answer keys, or private grading notes in reports
- generated report file packaging for response attachments
- platform/support access to detailed student answers or answer files
- billing, payroll, accounting, messaging, notifications, live classroom, or video conferencing behavior
- permanent purge, legal hold, anonymization, retention override, or emergency support access

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- existing v1 question types remain compatible
- `long_text` schema acceptance and rejection for blank, whitespace-only, and over-10,000-character answers
- `file_response` validation for allowed categories, 25 MB maximum, one file per question, unsafe filenames, executable/archive rejection, and declared/detected type mismatch
- malware-scan gating for pending and failed answer files
- zero/exempt-only grading behavior for failed-scan file-response answers
- same-school student assignment requirement for response submission
- duplicate attempt rejection
- after-due-date submission rejection based on learning-set due date
- manual 0-100 grading authority and invalid score rejection
- teacher owner/assignment and school-administrator authorization boundaries
- clean answer-file download allowed only for owning or assigned teachers and school administrators with same-school assessment review authority
- student own-view privacy boundaries
- report catalog and report output aggregate-only visibility
- guardian, platform, and support detail access denied unless future contract grants it
- tenant-safe audit events for authoring, submission, upload, scan, review, grading, successful answer-file download, denied answer-file download, validation, denial, conflict, and blocked cross-tenant outcomes
- student-visible feedback summary exposure to the owning student and private grading note exclusion from student/report/guardian/platform/support/audit payloads
- audit metadata redaction for raw answer text, file contents, private paths, credentials, answer keys, student-visible feedback text, private grading notes, full payloads, and unauthorized cross-tenant details
