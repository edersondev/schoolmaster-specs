# Quickstart: Backend Teacher Workflow Lifecycle and Corrections

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-teacher-workflow-lifecycle.md](./contracts/backend-teacher-workflow-lifecycle.md) for the proposed operation boundary and contract expansion requirements.
- Expand `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` before backend implementation exposes teacher workflow lifecycle, correction, download, import, delete, restore, or detail behavior.

## Delivery Boundary

## Feature-to-Repository Implementation Notes

- Run `/speckit-implement` from the `schoolmaster-backend` repository root only after tasks are generated.
- Backend implementation paths in `tasks.md` will be relative to `schoolmaster-backend`.
- Specification and OpenAPI paths use the backend repository's `specs` symlink, which points to `../schoolmaster-specs`.
- Contract changes must be made in both `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` before matching backend routes are exposed.

Implement only OpenAPI-approved operations for:

- teacher content detail, update, lifecycle status, delete, restore, and download
- questionnaire detail, update, lifecycle status, delete, and restore
- learning-set detail, update, lifecycle status, delete, restore, and roster-aware assignment maintenance where approved
- grade detail, correction, lifecycle status, delete, restore, and create-only JSON grade imports
- attendance detail, correction, lifecycle status, delete, restore, and create-only JSON attendance imports
- correction records, import runs, and audit events required by approved operations

Do not implement frontend screens, guardian self-service, report lifecycle expansion, platform support access, advanced assessment types, content/questionnaire versioning, permanent purge, legal hold, anonymization, messaging, billing, imports for content/questionnaires/learning sets, approval queues for closed-period corrections, new direct selected-student assignment writes, or undocumented APIs.

## Route Traceability

Every backend route added or expanded for this feature must map to an OpenAPI operation ID:

| Route | Method | Operation ID |
| --- | --- | --- |
| `/api/v1/teacher-content` | GET | `listTeacherContent` |
| `/api/v1/teacher-content` | POST | `createTeacherContent` |
| `/api/v1/teacher-content/{contentItemId}` | GET | `getTeacherContent` |
| `/api/v1/teacher-content/{contentItemId}` | PATCH | `updateTeacherContent` |
| `/api/v1/teacher-content/{contentItemId}/status` | PATCH | `updateTeacherContentStatus` |
| `/api/v1/teacher-content/{contentItemId}` | DELETE | `deleteTeacherContent` |
| `/api/v1/teacher-content/{contentItemId}/restore` | POST | `restoreTeacherContent` |
| `/api/v1/teacher-content/{contentItemId}/download` | GET | `downloadTeacherContent` |
| `/api/v1/questionnaires` | GET | `listQuestionnaires` |
| `/api/v1/questionnaires` | POST | `createQuestionnaire` |
| `/api/v1/questionnaires/{questionnaireId}` | GET | `getQuestionnaire` |
| `/api/v1/questionnaires/{questionnaireId}` | PATCH | `updateQuestionnaire` |
| `/api/v1/questionnaires/{questionnaireId}/status` | PATCH | `updateQuestionnaireStatus` |
| `/api/v1/questionnaires/{questionnaireId}` | DELETE | `deleteQuestionnaire` |
| `/api/v1/questionnaires/{questionnaireId}/restore` | POST | `restoreQuestionnaire` |
| `/api/v1/learning-sets` | GET | `listLearningSets` |
| `/api/v1/learning-sets` | POST | `createLearningSet` |
| `/api/v1/learning-sets/{learningSetId}` | GET | `getLearningSet` |
| `/api/v1/learning-sets/{learningSetId}` | PATCH | `updateLearningSet` |
| `/api/v1/learning-sets/{learningSetId}/status` | PATCH | `updateLearningSetStatus` |
| `/api/v1/learning-sets/{learningSetId}` | DELETE | `deleteLearningSet` |
| `/api/v1/learning-sets/{learningSetId}/restore` | POST | `restoreLearningSet` |
| `/api/v1/grades` | GET | `listGrades` |
| `/api/v1/grades` | POST | `createGrade` |
| `/api/v1/grades/{gradeId}` | GET | `getGrade` |
| `/api/v1/grades/{gradeId}` | DELETE | `deleteGrade` |
| `/api/v1/grades/{gradeId}/correction` | PATCH | `correctGrade` |
| `/api/v1/grades/{gradeId}/status` | PATCH | `updateGradeStatus` |
| `/api/v1/grades/{gradeId}/restore` | POST | `restoreGrade` |
| `/api/v1/grades/imports` | POST | `importGrades` |
| `/api/v1/attendance` | GET | `listAttendance` |
| `/api/v1/attendance` | POST | `createAttendance` |
| `/api/v1/attendance/{attendanceId}` | GET | `getAttendance` |
| `/api/v1/attendance/{attendanceId}` | DELETE | `deleteAttendance` |
| `/api/v1/attendance/{attendanceId}/correction` | PATCH | `correctAttendance` |
| `/api/v1/attendance/{attendanceId}/status` | PATCH | `updateAttendanceStatus` |
| `/api/v1/attendance/{attendanceId}/restore` | POST | `restoreAttendance` |
| `/api/v1/attendance/imports` | POST | `importAttendance` |
| `/api/v1/student/learning-sets` | GET | `listStudentLearningSets` |
| `/api/v1/student/grades` | GET | `listStudentGrades` |
| `/api/v1/student/attendance` | GET | `listStudentAttendance` |
| `/api/v1/student/teacher-content/{contentItemId}/download` | GET | `downloadStudentTeacherContent` |

## Validation Walkthrough

### 1. Contract readiness

- Confirm OpenAPI contains approved operation IDs for every implemented detail, update, status, delete, restore, download, correction, and import operation.
- Confirm every implemented route uses a documented `/api/v1` path, request schema, response schema, status code, and content type.
- Confirm OpenAPI documents lifecycle states `active`, `inactive`, and `deleted`.
- Confirm OpenAPI documents current student self-view as active-records-only, with inactive/deleted records hidden except explicitly documented historical labels.
- Confirm OpenAPI documents delete-to-`deleted`, restore-to-`inactive`, and separate activation from `inactive`.
- Confirm OpenAPI documents immutable and editable field matrices for teacher content, questionnaires, learning sets, grades, and attendance.
- Confirm OpenAPI documents historical-meaning edit rejection for used content and questionnaires.
- Confirm OpenAPI documents teacher owner/creator authority and school-administrator same-school override.
- Confirm OpenAPI documents same-school non-owner teacher denial.
- Confirm OpenAPI documents school-administrator-only closed-period grade and attendance corrections.
- Confirm OpenAPI documents correction reasons as required free text from 10 to 500 characters inclusive.
- Confirm OpenAPI documents school-administrator-only create-only JSON grade and attendance imports, 500-row maximum, and all-or-nothing rejection.
- Confirm OpenAPI documents clean-scan and lifecycle requirements for teacher content downloads.
- Confirm OpenAPI documents audit expectations for successful and denied teacher content downloads.
- Confirm OpenAPI documents roster-aware learning-set assignment writes and legacy direct-assignment read compatibility.
- Confirm no backend route exposes fields, filters, include expansion, sorts, status codes, status values, lifecycle actions, correction states, import shapes, download behavior, or authorization exceptions absent from OpenAPI.

### 2. Tenant and authorization validation

- Confirm every operation resolves active permitted school context before lookup, authorization, file access, validation, persistence, audit, import processing, or response shaping.
- Confirm missing, inactive, mismatched, and unauthorized school context fails before service logic accesses school-owned data.
- Confirm creating/owning teachers can manage their own same-school records where documented.
- Confirm school administrators can manage same-school records where documented.
- Confirm same-school teachers cannot manage records created or owned by another teacher.
- Confirm platform users do not receive implicit lifecycle, correction, import, download, or restore access.
- Confirm students and guardians cannot access teacher workflow management operations.

### 3. Lifecycle and restore validation

- Confirm teacher content, questionnaires, learning sets, grades, and attendance accept only `active`, `inactive`, and `deleted` lifecycle states.
- Confirm delete operations move records to `deleted`.
- Confirm restore operations move records to `inactive`, not directly to `active`.
- Confirm activation from `inactive` performs current tenant, dependency, scan, roster, academic-period, and student visibility validation.
- Confirm invalid lifecycle transitions return documented validation or conflict responses.
- Confirm active learning-set, student-visible, correction-history, audit-history, roster-eligibility, and tenant-isolation dependencies are preserved during delete and restore attempts.

### 4. Content and questionnaire validation

- Confirm content downloads require clean scan status and allowed lifecycle state.
- Confirm pending-scan, failed-scan, inactive, deleted, cross-tenant, and unauthorized downloads are rejected.
- Confirm successful and denied content downloads create tenant-safe audit events.
- Confirm audit events never contain private file contents, storage paths, credentials, full payloads, or unauthorized cross-tenant details.
- Confirm content or questionnaire edits that would change historical student-facing meaning after use are rejected rather than versioned.

### 5. Learning-set validation

- Confirm new learning-set assignment writes use active same-school roster membership context.
- Confirm new direct selected-student assignment writes are not exposed.
- Confirm legacy direct selected-student assignments remain readable where existing contracts expose them.
- Confirm inactive, deleted, unclean, cross-tenant, unauthorized, closed-period, or dependency-invalid updates/restores/corrections are rejected without partial state.

### 6. Grade and attendance correction validation

- Confirm open-period teacher corrections require creator/owner authority and a 10-500 character free-text correction reason.
- Confirm school administrators can correct same-school grade and attendance records according to open-period and closed-period rules.
- Confirm closed-period corrections by teachers are rejected.
- Confirm missing, blank, shorter-than-10-character, and longer-than-500-character correction reasons are rejected.
- Confirm accepted corrections preserve original value, current value, actor, reason, timestamp, target student, academic period, correction history, and audit history.
- Confirm prior corrections are never erased.
- Confirm current student self-view shows active records only, hides inactive and deleted records except explicitly documented historical labels, and does not expose private correction notes or unauthorized actor metadata.

### 7. Import validation

- Confirm grade and attendance imports are school-administrator-only.
- Confirm grade and attendance imports accept JSON payloads only.
- Confirm imports create new records only and reject rows that target existing records for update or correction.
- Confirm teacher, student, guardian, platform-without-school-context, and non-administrator import attempts are rejected.
- Confirm imports with more than 500 rows are rejected before any row changes state.
- Confirm all rows validate before any import state is committed.
- Confirm one invalid, cross-tenant, malformed, duplicate, unauthorized, closed-period-restricted, dependency-conflicting, or missing-required-field row rejects the entire import.
- Confirm import error summaries do not expose unauthorized cross-tenant student, teacher, content, roster, or academic data.
- Confirm CSV, spreadsheet, archive, and file-upload import attempts are rejected as outside v1.
- Confirm import-based correction or update attempts are rejected as outside v1.

### 8. Audit and response-shape validation

- Confirm updates, lifecycle transitions, successful and denied downloads, imports, import rejections, corrections, correction rejections, delete/restore, blocked cross-tenant attempts, and conflict outcomes are audited with actor, action, outcome, target where resolved, reason where applicable, and tenant-safe summary metadata.
- Confirm successful JSON responses use documented success envelopes.
- Confirm validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, import-validation, download-denial, restore-conflict, and correction-history cases use only documented error envelopes or codes.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

## Validation Record

- 2026-06-02: `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1`
  passed for both `api/openapi.yaml` and
  `specs/001-schoolmaster-platform/contracts/openapi.yaml`.

Implementation PRs should record the contract validation result, test result, feature id `010-teacher-workflow-lifecycle`, and operation IDs implemented.

## Exit Criteria for Planning

- The implementation boundary maps to approved OpenAPI operation IDs.
- Tenant, owner, school-administrator, JSON import, correction reason, student visibility, and download audit rules are explicit.
- Lifecycle, restore, correction, import, content/questionnaire historical-meaning, roster-aware learning-set, and legacy compatibility behavior are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, tenant isolation, authorization, lifecycle states, restore-to-inactive, editable/immutable field matrices, historical-meaning edit rejection, closed-period correction authority, imports, download scan gating, download audit, active-only current student visibility, audit fields, and legacy direct-assignment compatibility.
