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

## Exit Criteria for Planning

- Product modules, actors, and non-goals are stable enough for task breakdown.
- Initial OpenAPI paths cover the launch-scope module families.
- Data model boundaries are sufficient to start backend and frontend task
  generation.
- No constitution check violations remain unresolved.
