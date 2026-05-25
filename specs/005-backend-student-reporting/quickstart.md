# Quickstart: Backend Student and Reporting Foundation

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-student-reporting.md](./contracts/backend-student-reporting.md) for the approved operation boundary.
- Review `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` for concrete request and response schemas.

## Delivery Boundary

Implement only these operation IDs in this slice:

- `listStudentLearningSets`
- `downloadStudentTeacherContent`
- `listStudentGrades`
- `listStudentAttendance`
- `listReports`
- `requestReport`
- `downloadReport`

Do not implement frontend behavior, teacher workflow writes, report designer/custom report definitions, platform-wide reporting, report deletion, report retry, automatic expired-output regeneration during download, classroom/course/section/roster workflows, correction workflows, guardian self-service, student profile management, or undocumented APIs until the specification and OpenAPI contracts are expanded.

## Validation Walkthrough

### 1. Contract readiness

- Confirm both aggregate and platform OpenAPI contracts contain the operation IDs listed above.
- Confirm every implemented route uses the documented `/api/v1` path, request schema, parameters, response status, and content type.
- Confirm list operations use documented pagination and response envelopes only.
- Confirm download operations use only the documented binary responses and documented error responses.
- Confirm no backend route exposes fields, filters, sorts, status codes, error responses, or download formats absent from OpenAPI.

### 2. Tenant and authorization validation

- Confirm all student and report requests resolve an active `school_id` context before data access or file storage.
- Confirm missing, inactive, mismatched, and unauthorized tenant contexts fail before service logic runs.
- Confirm platform access does not imply student self-view or school report access.
- Confirm report operations require school-scoped report permission.
- Confirm students cannot list, download, or infer records for another school or another student.

### 3. Student learning timeline validation

- Confirm the authenticated user has an active same-school `StudentProfile`.
- Confirm the learning timeline requires a same-school academic-period filter.
- Confirm returned learning sets are assigned directly to the authenticated student's active profile.
- Confirm learning sets are ordered by publish date and entries by teacher-defined sequence.
- Confirm content entries expose only documented student metadata.
- Confirm pending-scan, failed-scan, inactive, deleted, or unassigned content is not downloadable.

### 4. Student grade and attendance validation

- Confirm grade lists return only records for the authenticated student's same-school profile.
- Confirm attendance lists return only records for the authenticated student's same-school profile.
- Confirm optional academic-period filters reference the resolved school.
- Confirm unsupported filters, unsupported sorts, invalid pagination, and cross-tenant references are rejected.

### 5. Student content download validation

- Confirm content download requires same-school ownership, active assignment through a learning set, and `scan_status = clean`.
- Confirm unassigned, cross-tenant, inactive, missing, pending-scan, or failed-scan content is rejected.
- Confirm private storage paths, scanner internals, and cross-tenant file existence are not exposed.

### 6. Report request and list validation

- Confirm report runs are listed only inside the resolved school scope.
- Confirm supported report types are `attendance`, `grades`, `academic_structure`, and `school_activity`.
- Confirm every report request includes a same-school `academic_period_id`.
- Confirm optional `student_profile_id`, `user_id`, `status`, `start_date`, and `end_date` filters are accepted only when relevant and same-school.
- Confirm report requests create asynchronous `ReportRun` records and do not wait for generated files.

### 7. Report output and expiry validation

- Confirm generated outputs support only PDF and CSV.
- Confirm generated files are stored privately under tenant scope.
- Confirm authorized report downloads return the requested generated format while unexpired.
- Confirm generated output files expire after 90 days.
- Confirm expired output downloads return the documented expired-output response.
- Confirm expired downloads do not regenerate files or mutate the expired report run.
- Confirm creating a new report request with the same filters creates a new `ReportRun`.

### 8. Response-shape validation

- Confirm successful JSON responses use the documented `SuccessEnvelope` or `PaginatedEnvelope`.
- Confirm downloads use the documented binary content type.
- Confirm validation where applicable, unauthorized, tenant-mismatch, not-found, and expired-output cases use only the error envelopes or codes documented on each affected OpenAPI operation.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Implementation PRs should record the contract validation result, test result, feature id `005-backend-student-reporting`, and the operation IDs implemented.

## Exit Criteria for Planning

- The implementation boundary maps to the approved OpenAPI operation IDs.
- Tenant and authorization rules are explicit for every affected entity.
- Student ownership, content assignment, scan-gated file access, report filter scoping, asynchronous report generation, output expiry, and no download-time regeneration are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, binary downloads, tenant isolation, authorization, validation, inactive statuses, scan gating, academic-period filters, report retention, and report regeneration through a new `ReportRun`.
