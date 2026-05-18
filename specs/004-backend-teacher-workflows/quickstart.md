# Quickstart: Backend Teacher Workflow Foundation

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-teacher-workflows.md](./contracts/backend-teacher-workflows.md) for the approved operation boundary.
- Review `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` for concrete request and response schemas.

## Delivery Boundary

Implement only these operation IDs in this slice:

- `listTeacherContent`
- `createTeacherContent`
- `listQuestionnaires`
- `createQuestionnaire`
- `listLearningSets`
- `createLearningSet`
- `listGrades`
- `createGrade`
- `listAttendance`
- `createAttendance`

Do not implement student self-service, reporting, classroom/course/section/roster workflows, public content-folder CRUD, detail, update, delete, deactivate, restore, download, bulk import, or correction behavior until the specification and OpenAPI contracts are expanded.

## Validation Walkthrough

### 1. Contract readiness

- Confirm both aggregate and platform OpenAPI contracts contain the operation IDs listed above.
- Confirm every implemented route uses the documented `/api/v1` path and request schema.
- Confirm list operations use documented pagination and response envelopes only.
- Confirm no backend route exposes fields, filters, sorts, status codes, or error responses absent from OpenAPI.
- Confirm public folder management remains blocked unless OpenAPI is expanded first.

### 2. Tenant and authorization validation

- Confirm all teacher workflow requests resolve an active `school_id` context before data access or file storage.
- Confirm missing, inactive, mismatched, and unauthorized tenant contexts fail before service logic runs.
- Confirm platform access does not imply teacher workflow access.
- Confirm teachers cannot list, create, or reference records for another school.

### 3. Teacher content and upload validation

- Confirm allowed files are limited to PDF, images, text, and office documents up to 25 MB.
- Confirm executable and archive files are rejected.
- Confirm declared and detected content types are validated.
- Confirm filenames, metadata, and storage paths are sanitized before persistence.
- Confirm uploaded files are stored in private tenant-scoped storage.
- Confirm new uploads initialize scan status and remain unavailable until marked `clean`.
- Confirm `pending` or `failed` content cannot be used in learning sets or exposed through file-access workflows.

### 4. Questionnaire validation

- Confirm questionnaires are created only inside the resolved school.
- Confirm supported question types are `multiple_choice`, `true_false`, and `short_text`.
- Confirm invalid question shapes and unsupported types are rejected.
- Confirm question sequence is preserved and duplicate sequences are rejected.

### 5. Learning set validation

- Confirm learning sets reference an active same-school academic period.
- Confirm entries reference same-school available content or questionnaires.
- Confirm content entries require `scan_status = clean`.
- Confirm selected student profile assignments are active, same-school, and valid for the selected academic period.
- Confirm invalid references reject the entire request without partial learning sets, entries, or assignments.

### 6. Grade and attendance validation

- Confirm grade values are numeric from 0 to 100.
- Confirm attendance statuses use only `present`, `absent`, `late`, `excused`, `remote`, and `suspended`.
- Confirm student profile, academic period, recorder, and school context are same-school and active where required.
- Confirm closed periods are read-only unless a future correction workflow is documented.

### 7. Response-shape validation

- Confirm successful creates use `SuccessEnvelope`.
- Confirm lists use `PaginatedEnvelope`.
- Confirm validation, unauthorized, forbidden, tenant-mismatch, token-rejection or inactive-user/inactive-school, and not-found cases use documented error envelopes or codes.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
php artisan test
```

Implementation PRs should record the contract validation result, test result, feature id `004-backend-teacher-workflows`, and the operation IDs implemented.

## Exit Criteria for Planning

- The implementation boundary maps to the approved OpenAPI operation IDs.
- Tenant and authorization rules are explicit for every affected entity.
- Upload, scan-status, learning-set, grade, attendance, academic-period, and student-profile rules are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, tenant isolation, authorization, validation, inactive statuses, upload safety, malware scan gating, academic rules, and selected-student references.
