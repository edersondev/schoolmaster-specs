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

### 4. Academic record validation

- Confirm grades and attendance are always tied to a student, academic period,
  recorder, and school.
- Confirm students can view only their own assigned learning sets, grades, and
  attendance.

### 5. Reporting validation

- Confirm school administrators can request reports for their school only.
- Confirm report filters cannot expand outside tenant scope.

### 6. Contract validation

- Confirm all protected endpoints declare authentication expectations.
- Confirm school-scoped endpoints document tenant-context behavior.
- Confirm success, validation, forbidden, inactive-record, tenant-mismatch, and
  not-found responses use the shared response envelope.
- Confirm P1 operations expose concrete request and response schemas for
  schools, users, roles, permissions, academic years, academic periods, and
  guardians before backend or frontend implementation starts.

## Verification Commands

Run the concrete commands in the target repositories once those repositories
contain the implementation:

```bash
# schoolmaster-backend
php artisan test

# schoolmaster-frontend
npm test

# schoolmaster-specs
# Use the repository-approved OpenAPI validator for contracts/openapi.yaml.
```

## Exit Criteria for Planning

- Product modules, actors, and non-goals are stable enough for task breakdown.
- Initial OpenAPI paths cover the launch-scope module families and define P1
  request, response, security, tenant, pagination, filtering, sorting, and
  standard error semantics.
- Data model boundaries are sufficient to start backend and frontend task
  generation.
- No constitution check violations remain unresolved.
