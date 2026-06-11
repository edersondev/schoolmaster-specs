# Quickstart: Advanced Assessment and Content Types

## Purpose

Use this walkthrough to verify that the advanced assessment and content types plan, contract boundary, and backend implementation remain aligned before implementation proceeds.

## Prerequisites

- Active feature branch: `014-advanced-assessment-content`
- Active spec: `specs/014-advanced-assessment-content/spec.md`
- Active plan: `specs/014-advanced-assessment-content/plan.md`
- OpenAPI changes must be completed before backend routes expose advanced assessment behavior.
- Backend implementation must remain API-only and scoped to the backend repository.
- Frontend implementation is out of scope for this slice.

## Contract Validation

After OpenAPI is expanded for this feature, validate the aggregate contract:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

If validating individual files directly, use:

```bash
npx @redocly/cli lint api/openapi.yaml
npx @redocly/cli lint specs/001-schoolmaster-platform/contracts/openapi.yaml
```

### Validation Results

- Pending: Run after OpenAPI operations and schemas are added.

Contract review must confirm:

- operation IDs exist for every approved advanced assessment operation
- all routes remain under `/api/v1`
- `QuestionType` includes only existing types plus `long_text` and `file_response`
- unsupported advanced question types are rejected
- long-text answers allow 1-10,000 characters and reject blank or whitespace-only answers
- file responses allow PDF, image, text, and office files only
- file responses allow one file per question and maximum 25 MB
- file-response scan states and unavailable-file responses are documented
- student response submission enforces one attempt per student per assigned questionnaire
- student response submission closes at the learning-set due date
- manual grading uses 0-100 points for `long_text` and `file_response`
- student-visible response summaries include score/status and teacher feedback summary, not private grading notes
- student own-view response fields are student-safe
- report catalog exposes only assessment counts, completion status, grading status, and score summaries
- raw answer text, answer keys, uploaded files, file links, private file metadata, storage paths, and private grading notes are excluded from reports
- guardian, platform, and support detail access remains excluded unless a future contract grants it
- tenant-safe audit expectations are documented

## Backend Verification

After backend implementation, run the backend test suite from the backend repository:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

### Backend Validation Results

- Pending: Run after backend implementation.

Focused backend coverage must include:

- successful questionnaire create/update with `long_text` and `file_response`
- existing `multiple_choice`, `true_false`, and `short_text` behavior remains compatible
- unsupported question types rejected
- historical-meaning edits rejected after publication, assignment, response submission, grading, or report generation
- successful student response submission for assigned same-school questionnaire
- unassigned, other-student, inactive-profile, inactive-learning-set, inactive-questionnaire, after-due-date, malformed, and cross-tenant submissions rejected
- duplicate response attempt rejected
- long-text blank, whitespace-only, and over-10,000-character answers rejected
- file-response PDF, image, text, and office files accepted under 25 MB
- unsupported file category, unsafe filename, executable/archive file, oversized file, multiple files for one question, and declared/detected type mismatch rejected
- pending and failed scan files unavailable to teacher review, grading, student display, reports, platform summaries, support diagnostics, and downloads
- failed-scan file-response answers may receive only zero or exempt grading outcomes from authorized teachers/admins
- clean answer file available only to owning or assigned teachers and school administrators through a documented review operation where OpenAPI approves file delivery
- manual grading accepts 0-100 point scores for authorized teachers/admins
- manual grading rejects scores outside 0-100, unauthorized actor, scan-blocked files, stale state, and cross-tenant responses
- student own-view exposes only approved submission status, grading status, score summary, teacher feedback summary, and safe file availability metadata
- report catalog and generated reports expose only assessment counts, completion status, grading status, and score summaries
- raw answer text, answer keys, uploaded files, file links, teacher feedback summaries, private file metadata, storage paths, and private grading notes are rejected from reports
- guardian, platform, and support users cannot access detailed student answers or answer files through this slice
- audit events written for authoring, submission, file upload, scan outcome, review, grading, denied access, validation failure, lifecycle conflict, report-field access, and blocked cross-tenant outcomes
- successful and denied answer-file download attempts create tenant-safe audit events
- audit metadata excludes raw answer text, uploaded file contents, private storage paths, credentials, hidden answer keys, student-visible feedback text, private grading notes, full request payloads, and unauthorized cross-tenant details

## Backend Architecture Checklist

- Controllers are orchestration-only.
- Business rules live in `App\Services\Assessment`.
- Request validation uses Form Requests.
- Authorization uses Policies.
- Responses use API Resources and documented envelopes.
- Multi-field inputs use DTOs where they improve clarity.
- Repositories/query objects are used only for complex response aggregation, report catalog summary exposure, answer-file lookup, and audit-safe reads.
- Public identifiers are UUIDs.
- School-owned reads and writes remain scoped by `school_id`.
- File-response attachments use private tenant storage.
- File-response availability is gated by scan status.
- No frontend behavior is implemented in this slice.

## Out-of-Scope Guardrail

Reject implementation or contract additions for:

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

Any needed behavior from this list requires a separate specification and OpenAPI update.
