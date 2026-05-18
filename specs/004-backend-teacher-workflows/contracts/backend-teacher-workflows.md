# Contract Boundary: Backend Teacher Workflow Foundation

## Purpose

This feature does not create a new public API surface. It defines the backend implementation boundary for the P2 teacher workflow operations already published in the SchoolMaster OpenAPI contracts.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend implementation guidance: `docs/backend-guidelines.md`
- Multi-tenant guidance: `docs/multi-tenant.md`
- Security guidance: `docs/security.md`
- Tenant decision: `decisions/004-use-tenant-by-column.md`

## Approved Operation Boundary

The backend may implement only these operation IDs in this slice unless `/specs` and OpenAPI are updated first:

| Operation ID | Method and Path | Boundary |
|--------------|-----------------|----------|
| `listTeacherContent` | `GET /api/v1/teacher-content` | List teacher content items visible in the resolved school scope |
| `createTeacherContent` | `POST /api/v1/teacher-content` | Upload one allowed private teacher content item and initialize scan status |
| `listQuestionnaires` | `GET /api/v1/questionnaires` | List questionnaires visible in the resolved school scope |
| `createQuestionnaire` | `POST /api/v1/questionnaires` | Create a questionnaire using v1 supported question types |
| `listLearningSets` | `GET /api/v1/learning-sets` | List learning sets visible in the resolved school scope |
| `createLearningSet` | `POST /api/v1/learning-sets` | Create a learning set with ordered entries and direct same-school student assignments |
| `listGrades` | `GET /api/v1/grades` | List grade records visible in the permitted school scope |
| `createGrade` | `POST /api/v1/grades` | Record one grade for an active same-school student profile and academic period |
| `listAttendance` | `GET /api/v1/attendance` | List attendance records visible in the permitted school scope |
| `createAttendance` | `POST /api/v1/attendance` | Record attendance for an active same-school student profile and academic period |

## Required Response Shapes

Backend implementation must follow the response statuses and components declared on each approved OpenAPI operation:

- `SuccessEnvelope`
- `PaginatedEnvelope`
- `ErrorEnvelope`
- `ValidationError`
- `Unauthorized`
- `TenantMismatch`
- `TeacherWorkflowForbidden`

The `TeacherWorkflowForbidden` component covers both tenant-context failures and valid-tenant permission failures for these ten operations. The shared OpenAPI contract also defines `Forbidden`, `TokenRejected`, and `NotFound` components for other operations. Those components are not approved for these ten teacher workflow operations unless OpenAPI first attaches them to the relevant operation. No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, or undocumented sort behavior is approved in this slice.

## Tenant Behavior

- Teacher workflow operations use the documented `X-School-Id` tenant context behavior when the authenticated session is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context must fail before school-owned data access, file storage, or persistence.
- Platform-scope users do not receive implicit permission to perform school-scoped teacher workflows without an explicit permitted school context.

## Authorization Behavior

- Teacher workflow operations require authenticated school-scoped permissions.
- Content, questionnaires, learning sets, grades, attendance, academic periods, teachers, and student profiles must all resolve to the same permitted school context where relevant.
- Inactive users, inactive schools, inactive student profiles, inactive content, inactive questionnaires, inactive or closed periods, and unavailable scanned content cannot be used for new operational writes.

## Validation Behavior

- Teacher content creation rejects unsupported file categories, executable/archive files, files over 25 MB, declared/detected content-type mismatches, unsafe filenames or metadata, unsafe storage paths, unauthorized folder references, and undocumented fields.
- Uploaded files remain private and unavailable until malware scan status is `clean`.
- Questionnaire creation rejects unsupported question types, invalid question shapes, duplicate sequences, empty question sets, and undocumented fields.
- Learning set creation rejects inactive, missing, cross-tenant, unclean, or unsupported content and questionnaire references; invalid academic periods; duplicate entry sequences; empty entries; empty assignments; and invalid student profile references.
- Grade creation rejects invalid grade values, inactive or cross-tenant student profiles, inactive or closed periods, unauthorized recorder context, and undocumented fields.
- Attendance creation rejects unsupported attendance statuses, inactive or cross-tenant student profiles, inactive or closed periods, unauthorized recorder context, and undocumented fields.

## Blocked Until Contract Expansion

These behaviors are outside this implementation boundary until OpenAPI documents them:

- teacher content folder list, create, update, delete, restore, or move operations
- teacher content, questionnaire, learning set, grade, or attendance detail endpoints
- update, deactivate, activate, delete, restore, bulk import, or correction workflows
- teacher or student content download endpoints not already documented for the relevant actor
- classroom, course, section, group, roster, or teacher assignment workflows
- student self-service learning timelines, student grades, student attendance, or student downloads
- report requests, report output downloads, retention, expiry, or regeneration
- additional filters, sorting options, response fields, or authorization exceptions

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of the aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- authorization failures for platform/school scope separation and teacher workflow permissions
- validation failures for uploads, malware scan gating, questionnaire question types, learning-set references, grade values, attendance statuses, active periods, and student profile references
