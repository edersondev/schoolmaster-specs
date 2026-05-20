# Contract Boundary: Backend Student and Reporting Foundation

## Purpose

This feature does not create a new public API surface. It defines the backend implementation boundary for the P3 student and reporting operations already published in the SchoolMaster OpenAPI contracts.

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
| `listStudentLearningSets` | `GET /api/v1/student/learning-sets` | List learning sets assigned to the authenticated student's active same-school profile for a selected academic period |
| `downloadStudentTeacherContent` | `GET /api/v1/student/teacher-content/{contentItemId}/download` | Download one assigned same-school teacher content file when available and scan-clean |
| `listStudentGrades` | `GET /api/v1/student/grades` | List grade records for the authenticated student's active same-school profile |
| `listStudentAttendance` | `GET /api/v1/student/attendance` | List attendance records for the authenticated student's active same-school profile |
| `listReports` | `GET /api/v1/reports` | List report runs visible in the resolved school scope |
| `requestReport` | `POST /api/v1/reports` | Request an asynchronous school-scoped report run for launch-scope report types and filters |
| `downloadReport` | `GET /api/v1/reports/{reportRunId}/download` | Download an authorized generated report output file in a documented format while unexpired |

## Required Response Shapes

Backend implementation must follow the response statuses, content types, and components declared on each approved OpenAPI operation:

- `SuccessEnvelope`
- `PaginatedEnvelope`
- `ErrorEnvelope`
- `ValidationError`
- `Unauthorized`
- `TenantMismatch`
- `NotFound`
- `OutputExpired`
- `application/octet-stream` binary file responses for authorized downloads

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented sort behavior, or undocumented download format is approved in this slice.

## Tenant Behavior

- Student and report operations use the documented `X-School-Id` tenant context behavior when the authenticated session is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context must fail before school-owned data access, file storage, report filtering, output lookup, or persistence.
- Platform-scope users do not receive implicit permission to perform school-scoped student self-view or reporting workflows without an explicit permitted school context.

## Authorization Behavior

- Student self-view operations require authenticated access, active tenant context, active user status, and an active same-school `StudentProfile` linked to the authenticated user.
- Student learning-set, grade, attendance, and content download queries must be constrained to the authenticated student's profile.
- Student content download requires an active same-school assignment through a learning set and `scan_status = clean` on the content item.
- Report operations require authenticated school-scoped report permission in the resolved school.
- Report run, report output, academic period, student profile, and user filter references must all resolve to the same permitted school context where relevant.

## Validation Behavior

- Student timeline rejects missing, invalid, inactive, or cross-tenant academic-period filters.
- Student grade and attendance lists reject unsupported filters, unsupported sort values, invalid pagination, and cross-tenant academic-period filters.
- Student content download rejects missing, inactive, cross-tenant, unassigned, pending-scan, failed-scan, or unavailable teacher content without exposing file storage details.
- Report requests reject unsupported report types, unsupported filters, invalid date ranges, cross-tenant academic periods, cross-tenant student profiles, cross-tenant users, and undocumented fields.
- Report downloads reject unsupported formats, unauthorized report runs, missing outputs, failed or inactive runs, expired outputs, and cross-tenant output access.

## Report Retention and Regeneration Behavior

- Report generation is asynchronous. A successful `requestReport` creates a `ReportRun` and returns status and output availability metadata.
- Launch-scope report types are `attendance`, `grades`, `academic_structure`, and `school_activity`.
- Documented report output formats are `pdf` and `csv`.
- Generated report output files remain available for 90 days after generation.
- `ReportRun` metadata remains available after output files expire.
- `downloadReport` must return the documented expired-output response for expired files.
- Expired downloads must not regenerate files, enqueue regeneration, or mutate the expired `ReportRun`.
- Regeneration requires a new `requestReport` call with the same filters, creating a new `ReportRun`.

## Blocked Until Contract Expansion

These behaviors are outside this implementation boundary until OpenAPI documents them:

- frontend student or reporting implementation
- teacher workflow writes or corrections
- report designer or custom report definitions
- platform-wide reports or support-user report overrides
- report deletion, restore, retry, cancellation, or manual status mutation endpoints
- automatic expired-output regeneration during download
- classroom, course, section, group, roster, or teacher assignment workflows
- guardian self-service or guardian student views
- student profile creation, update, transfer, or enrollment management
- additional filters, sorting options, response fields, download formats, or authorization exceptions

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of the aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- binary response checks for authorized student content and report downloads
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- student ownership failures for other-student, inactive-profile, unassigned, and cross-tenant access attempts
- authorization failures for platform/school scope separation and report permissions
- validation failures for academic-period filters, report filters, report types, report formats, date ranges, content scan status, report expiry, and invalid references
