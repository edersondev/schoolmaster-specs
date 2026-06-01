# Contract Boundary: Backend Teacher Workflow Lifecycle and Corrections

## Purpose

This feature expands the public API surface for teacher workflow lifecycle, correction, download, and import behavior. Backend implementation must wait until OpenAPI documents exact operations, parameters, request schemas, response schemas, errors, and operation IDs.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend implementation guidance: `docs/backend-guidelines.md`
- Multi-tenant guidance: `docs/multi-tenant.md`
- Security guidance: `docs/security.md`
- Tenant decision: `decisions/004-use-tenant-by-column.md`

## Proposed Operation Boundary

The exact operation IDs must be finalized in OpenAPI, but the backend slice should be limited to this surface:

| Resource | Proposed Operations | Boundary |
|----------|---------------------|----------|
| `TeacherContentItem` | detail, update, activate/deactivate, delete, restore, download | Teacher owner or school administrator; download requires clean scan status and active allowed lifecycle state; used content rejects historical-meaning edits; successful and denied downloads are audited |
| `Questionnaire` | detail, update, activate/deactivate, delete, restore | Teacher owner or school administrator; v1 question types only; used questionnaires reject historical-meaning edits |
| `LearningSet` | detail, update, activate/deactivate, delete, restore | Teacher owner or school administrator; new assignment writes use roster memberships; legacy direct assignments remain read-only |
| `GradeRecord` | detail, update/correct, activate/deactivate, delete, restore, import | Teacher owner/recorder in open periods where approved; closed-period corrections and imports are school-administrator-only |
| `AttendanceRecord` | detail, update/correct, activate/deactivate, delete, restore, import | Teacher owner/recorder in open periods where approved; closed-period corrections and imports are school-administrator-only |
| `CorrectionRecord` | side-effect history for accepted grade and attendance corrections | Preserve original values, current values, 10-500 character free-text reason, actor, timestamp, target, and tenant-safe visibility metadata |
| `ImportRun` | submit import and return submitted import outcome | Create-only JSON payload imports for grades and attendance only, school-administrator-only, maximum 500 rows, all-or-nothing |
| `AuditEvent` | side-effect recording for lifecycle, correction, import, download, conflict, and blocked access | Tenant-safe metadata only; no private files, paths, credentials, full payloads, or unauthorized cross-tenant details |

No backend implementation may expose these or adjacent routes until OpenAPI documents them.

## Proposed Route-to-Operation Mapping

OpenAPI may approve a different path or operation set, but backend implementation must remain limited to the approved contract.

| Method | Route | Proposed OpenAPI operation ID |
|--------|-------|-------------------------------|
| `GET` | `/api/v1/teacher-content/{contentItemId}` | `getTeacherContent` |
| `PATCH` | `/api/v1/teacher-content/{contentItemId}` | `updateTeacherContent` |
| `PATCH` | `/api/v1/teacher-content/{contentItemId}/status` | `updateTeacherContentStatus` |
| `DELETE` | `/api/v1/teacher-content/{contentItemId}` | `deleteTeacherContent` |
| `POST` | `/api/v1/teacher-content/{contentItemId}/restore` | `restoreTeacherContent` |
| `GET` | `/api/v1/teacher-content/{contentItemId}/download` | `downloadTeacherContent` |
| `GET` | `/api/v1/questionnaires/{questionnaireId}` | `getQuestionnaire` |
| `PATCH` | `/api/v1/questionnaires/{questionnaireId}` | `updateQuestionnaire` |
| `PATCH` | `/api/v1/questionnaires/{questionnaireId}/status` | `updateQuestionnaireStatus` |
| `DELETE` | `/api/v1/questionnaires/{questionnaireId}` | `deleteQuestionnaire` |
| `POST` | `/api/v1/questionnaires/{questionnaireId}/restore` | `restoreQuestionnaire` |
| `GET` | `/api/v1/learning-sets/{learningSetId}` | `getLearningSet` |
| `PATCH` | `/api/v1/learning-sets/{learningSetId}` | `updateLearningSet` |
| `PATCH` | `/api/v1/learning-sets/{learningSetId}/status` | `updateLearningSetStatus` |
| `DELETE` | `/api/v1/learning-sets/{learningSetId}` | `deleteLearningSet` |
| `POST` | `/api/v1/learning-sets/{learningSetId}/restore` | `restoreLearningSet` |
| `GET` | `/api/v1/grades/{gradeId}` | `getGrade` |
| `PATCH` | `/api/v1/grades/{gradeId}/correction` | `correctGrade` |
| `PATCH` | `/api/v1/grades/{gradeId}/status` | `updateGradeStatus` |
| `DELETE` | `/api/v1/grades/{gradeId}` | `deleteGrade` |
| `POST` | `/api/v1/grades/{gradeId}/restore` | `restoreGrade` |
| `POST` | `/api/v1/grades/imports` | `importGrades` |
| `GET` | `/api/v1/attendance/{attendanceId}` | `getAttendance` |
| `PATCH` | `/api/v1/attendance/{attendanceId}/correction` | `correctAttendance` |
| `PATCH` | `/api/v1/attendance/{attendanceId}/status` | `updateAttendanceStatus` |
| `DELETE` | `/api/v1/attendance/{attendanceId}` | `deleteAttendance` |
| `POST` | `/api/v1/attendance/{attendanceId}/restore` | `restoreAttendance` |
| `POST` | `/api/v1/attendance/imports` | `importAttendance` |

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths for every approved lifecycle, detail, correction, download, import, delete, and restore operation
- request schemas for editable fields on teacher content, questionnaires, learning sets, grade corrections, attendance corrections, status transitions, restore actions, and JSON import submissions
- immutable and editable field matrices for teacher content, questionnaires, learning sets, grades, and attendance
- lifecycle states `active`, `inactive`, and `deleted` for teacher content, questionnaires, learning sets, grades, and attendance
- restore behavior that returns records to `inactive`
- activation from `inactive` as a separate transition requiring current validation
- teacher owner/creator authority and school-administrator same-school override
- same-school teacher denial when the actor is not the creator or owner
- school-administrator-only closed-period grade and attendance correction behavior
- required 10-500 character free-text reason for grade and attendance corrections
- school-administrator-only grade and attendance imports
- imports create new grade or attendance records only; import-based correction or update is rejected
- maximum 500 rows per grade or attendance import
- JSON payload as the only v1 grade and attendance import submission format
- all-or-nothing import validation and import rejection response shape
- roster-aware learning-set assignment write behavior and read-only legacy direct-assignment compatibility
- rejection behavior for content or questionnaire edits that change historical student-facing meaning after use
- clean-scan and active-lifecycle requirements for teacher content download
- current student self-view behavior that shows active records only and hides inactive/deleted records except explicitly documented historical labels
- audit requirements for successful and denied teacher content downloads
- tenant-context errors for missing, inactive, mismatched, or unauthorized school context
- not-found behavior that does not reveal cross-tenant records
- response envelopes for success, paginated lists where applicable, validation errors, unauthorized, forbidden, tenant mismatch, not found, conflict, import validation, correction history, download delivery, and restore/status outcomes

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelopes for detail, update, status, delete, restore, correction, and import outcomes
- binary/file response or signed/private delivery response for teacher content download where approved
- validation error envelope
- unauthorized response
- forbidden response for valid school context but insufficient permission
- tenant mismatch or inactive tenant response for school-scoped operations
- not-found response that does not disclose cross-tenant records
- conflict response for immutable-field edits, dependency conflicts, lifecycle conflicts, concurrent writes, deleted/inactive dependency conflicts, historical-meaning edit attempts, and restore activation conflicts
- import-validation response for oversized, malformed, unauthorized, duplicate, cross-tenant, or dependency-invalid imports

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented include expansion, undocumented sort behavior, undocumented lifecycle state, undocumented lifecycle action, undocumented correction state, undocumented import shape, undocumented download behavior, or authorization exception is approved in this slice.

## Tenant Behavior

- Operations use documented `X-School-Id` tenant context behavior when the authenticated actor is not already bound to exactly one active school.
- V1 school-owned records use `school_id`.
- Missing, mismatched, inactive, or unauthorized tenant context fails before teacher workflow lookup, private file access, roster lookup, student lookup, teacher lookup, academic-period lookup, authorization, duplicate/dependency checks, persistence, audit creation, import processing, or response shaping.
- Platform administrators do not receive implicit permission to manage, correct, import, download, or restore school-owned teacher workflow records.

## Authorization Behavior

- All operations require authenticated access and active actor status.
- Teachers may manage records they created or own inside the resolved school context.
- School administrators may manage same-school records.
- Same-school teachers who are not the creator or owner cannot manage another teacher's content, questionnaires, learning sets, grades, or attendance.
- Closed-period grade and attendance corrections are school-administrator-only.
- Grade and attendance imports are school-administrator-only.
- Students and guardians do not receive teacher workflow lifecycle, correction, import, or teacher download access.

## Validation Behavior

- Teacher content download requires clean scan status, allowed active lifecycle state, same-school ownership or permission, and tenant-safe private delivery.
- Used content and questionnaires reject edits that change historical student-facing meaning.
- New learning-set assignment writes use active same-school roster membership context; new direct selected-student assignment writes are rejected.
- Legacy direct selected-student assignments remain readable where existing contracts expose them.
- Grade and attendance corrections require a 10-500 character free-text reason and preserve original values, current values, actor, timestamp, target, and audit history.
- Current student self-view shows active teacher workflow records only; inactive and deleted records are hidden except where OpenAPI explicitly documents historical labels.
- Teacher content, questionnaires, learning sets, grades, and attendance use `active`, `inactive`, and `deleted`.
- Delete moves records to `deleted`; restore moves records to `inactive`; activation is a separate transition.
- Restores and activations validate current tenant, ownership, dependency, scan, roster, academic-period, and student visibility rules.
- Grade and attendance imports accept JSON payloads only, create new records only, accept at most 500 rows, and reject the entire import if any row is invalid or attempts to correct/update an existing record.
- Concurrent update, correction, lifecycle, delete/restore, and import conflicts resolve without partial state.

## Blocked Until Future Specification

These behaviors are outside this implementation boundary until a future spec and OpenAPI update approve them:

- frontend implementation
- guardian self-service or guardian student views
- report retry, cancellation, deletion, restore, designer, custom reports, additional formats, or report output lifecycle changes
- platform support-user access to school-owned teacher workflow records
- teacher content folder public lifecycle expansion unless separately documented
- advanced assessment/question types beyond the current v1 types
- content/questionnaire automatic versioning
- permanent purge, anonymization, legal hold, merge, or retention override workflows
- messaging, notifications, parent communication, billing, payroll, or accounting
- new direct selected-student assignment writes
- imports for content, questionnaires, or learning sets
- CSV, spreadsheet, archive, or file-upload imports
- import-based correction or update of existing grade or attendance records
- approval queues for teacher-submitted closed-period corrections

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- owner/creator authorization and school-administrator override checks
- same-school non-owner teacher denial checks
- closed-period correction authority checks
- lifecycle state transition checks
- restore-to-inactive checks
- immutable-field and historical-meaning edit rejection checks
- roster-aware learning-set write checks
- legacy direct-assignment read compatibility checks
- teacher content clean-scan download checks
- successful and denied download audit checks
- grade and attendance import cap, all-or-nothing, and administrator-only checks
- correction history and student visibility checks
- conflict handling for concurrent lifecycle, correction, delete/restore, and import attempts
