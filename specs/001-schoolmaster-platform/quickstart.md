# Quickstart: SchoolMaster Platform Foundation

## Purpose

Use this guide to validate the design intent for the SchoolMaster v1 foundation
before implementation begins in the backend and frontend repositories.

## Prerequisites

- Review [spec.md](./spec.md) for product scope and acceptance outcomes.
- Review [plan.md](./plan.md) for architecture and constitution gates.
- Review [research.md](./research.md) for planning decisions.
- Review [data-model.md](./data-model.md) for initial entity boundaries.
- Review [contracts/openapi.yaml](./contracts/openapi.yaml) for the initial API
  surface.

## Suggested Delivery Order

1. Confirm the business rules and API contract in `schoolmaster-specs`.
2. Define backend feature slices in `schoolmaster-backend` that match the
   contract modules and tenant boundaries.
3. Define frontend feature slices in `schoolmaster-frontend` that consume only
   the contract-approved `/api/v1` endpoints.
4. Add verification for contract, backend behavior, and frontend behavior
   before merging each slice.

## Cross-Repository Traceability

- Use `001-schoolmaster-platform` in related branches, issues, pull requests,
  and implementation notes across `schoolmaster-specs`,
  `schoolmaster-backend`, and `schoolmaster-frontend`.
- Contract changes land in `schoolmaster-specs` before dependent backend or
  frontend implementation work.
- Backend work links to the OpenAPI operation IDs it implements and includes
  PHPUnit coverage for tenant, authorization, validation, and response-shape
  behavior.
- Frontend work links to the OpenAPI operation IDs it consumes and includes
  Vitest coverage for affected services, stores, composables, and route guards.

When backend or frontend repositories consume this repository as a git
submodule, follow the repository-level guide in
[`docs/repository-consumption.md`](../../docs/repository-consumption.md).

## Contract Readiness

- The active feature contract is `contracts/openapi.yaml`.
- The repository-level aggregate contract is `../../api/openapi.yaml`.
- Backend and frontend implementation should consume the feature contract until
  approved operations are promoted into the aggregate contract.
- Any implementation PR that relies on unpromoted API behavior must link to the
  feature contract operation IDs it uses.

## Validation Walkthrough

### 1. Tenant and actor validation

- Confirm one `School` represents one tenant boundary.
- Confirm only system administrators can create or activate schools.
- Confirm school administrators, teachers, and students are restricted to one
  school context in normal workflows.

### 2. Academic structure validation

- Confirm each school can create one or more academic years.
- Confirm academic periods belong to an academic year and are used by grades,
  attendance, learning sets, and reports.

### 3. Instructional workflow validation

- Confirm a teacher can create folders, upload content, create questionnaires,
  and compose learning sets in a defined order.
- Confirm learning set items remain tenant-scoped and tied to the teacher's
  school.
- Confirm uploaded teacher content accepts only PDF, image, text, and office
  document files up to 25 MB, rejects executables and archives, and remains
  unavailable until malware scanning succeeds.
- Confirm questionnaires accept only multiple-choice, true-or-false, and
  short-text questions for v1.
- Confirm learning sets are assigned directly to selected active
  `StudentProfile` records in the same school and academic period.
- Confirm teacher workflows do not depend on undocumented class, course,
  section, roster, or group APIs in v1.

### 4. Academic record validation

- Confirm grades and attendance are always tied to a student, academic period,
  recorder, and school.
- Confirm grade and attendance entry targets selected active `StudentProfile`
  records directly for v1.
- Confirm grade values are numeric decimals from 0 to 100 with optional display
  labels.
- Confirm attendance uses only `present`, `absent`, `late`, `excused`,
  `remote`, or `suspended`.
- Confirm students can view only their own assigned learning sets, grades, and
  attendance.
- Confirm student learning timelines require `academic_period_id`, order
  assigned learning sets by publish date, and order entries by teacher-defined
  sequence.
- Confirm students can view metadata and download teacher content only when it
  belongs to their school, is included in their assigned learning set, and has
  passed malware scanning.

### 5. Reporting validation

- Confirm school administrators can request reports for their school only.
- Confirm report filters cannot expand outside tenant scope.
- Confirm every report request requires `academic_period_id`.
- Confirm report requests allow only the approved optional filter set:
  `student_profile_id`, `user_id`, `status`, `start_date`, and `end_date`.
- Confirm every launch-scope report request is asynchronous and returns a
  `ReportRun` status.
- Confirm generated reports provide both PDF and CSV outputs after status is
  `generated`.
- Confirm generated report output files remain downloadable for 90 days after
  generation and then expire while `ReportRun` metadata remains visible.
- Confirm expired report output downloads return the documented expired-output
  error and do not automatically start regeneration.
- Confirm administrators can request a new `ReportRun` with the same filters to
  generate fresh output files after expiry.

### 6. Contract validation

- Confirm all protected endpoints declare authentication expectations.
- Confirm school-scoped endpoints document tenant-context behavior.
- Confirm success, validation, forbidden, inactive-record, tenant-mismatch, and
  not-found responses use the shared response envelope.
- Confirm P1 operations expose concrete request and response schemas for
  schools, users, roles, permissions, academic years, academic periods, and
  guardians before backend or frontend implementation starts.
- Confirm P2 operations expose concrete request and response schemas for
  teacher content, questionnaires, learning sets, grades, and attendance before
  backend or frontend implementation starts.
- Confirm P3 operations expose concrete request and response schemas for
  student learning timelines, student grades, student attendance, authorized
  content downloads, report requests, and report output downloads before
  backend or frontend implementation starts.
- Confirm the feature OpenAPI contract and repository aggregate contract both
  pass executable validation before backend or frontend implementation merges.
- Confirm response-shape verification covers success, validation failure,
  forbidden, not found, inactive-record, tenant-mismatch, scan-pending/failed,
  and expired-output responses.

## Verification Commands

Run the concrete commands in the target repositories once those repositories
contain the implementation. Run the contract commands in this repository before
dependent backend or frontend work merges:

```bash
# schoolmaster-backend
docker exec schoolmaster-backend-app-1 php artisan test

# schoolmaster-frontend
npm test

# schoolmaster-specs
npx @redocly/cli lint specs/001-schoolmaster-platform/contracts/openapi.yaml
npx @redocly/cli lint api/openapi.yaml
```

Record the pass/fail result in the implementation PR or release note for the
feature. If aggregate publication is intentionally deferred, document the
approved reason and the feature contract operation IDs consumed by backend or
frontend work.

## Exit Criteria for Planning

- Product modules, actors, and non-goals are stable enough for task breakdown.
- Initial OpenAPI paths cover the launch-scope module families and define P1
  through P3 request, response, security, tenant, pagination, filtering,
  sorting, and standard error semantics.
- Data model boundaries are sufficient to start backend and frontend task
  generation.
- No constitution check violations remain unresolved.
