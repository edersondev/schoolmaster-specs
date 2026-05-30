# Quickstart: Backend Classroom Roster Foundation

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-classroom-roster-foundation.md](./contracts/backend-classroom-roster-foundation.md) for the proposed operation boundary and contract expansion requirements.
- Expand `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` before backend implementation exposes classroom roster foundation behavior.

## Delivery Boundary

## Feature-to-Repository Implementation Notes

- Run `/speckit-implement` from the `schoolmaster-backend` repository root after tasks are generated.
- Backend implementation paths in `tasks.md` will be relative to `schoolmaster-backend`.
- Specification and OpenAPI paths use the backend repository's `specs` symlink, which points to `../schoolmaster-specs`.
- Contract changes must be made in both `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml` before the matching backend routes are exposed.

Implement only OpenAPI-approved operations for:

- `ClassSection/Roster` list, create, detail, update, and inactivation where approved
- roster membership list and all-or-nothing batch add/end behavior capped at 100 requested changes
- teacher assignment list/detail, create, and status changes where approved
- audit events required by roster foundation transitions and conflict outcomes
- read-only compatibility for existing direct learning-set student assignments where existing contracts already expose them

Do not implement separate top-level or internal Course, Classroom, Section, or Group lifecycle resources, teacher workflow corrections, guardian self-service, report lifecycle expansion, platform support-user roster access, frontend behavior, messaging, notifications, billing, permanent purge, anonymization, merge/restore behavior, or undocumented APIs until the specification and OpenAPI contracts are expanded.

## Validation Walkthrough

### 1. Contract readiness

- Confirm OpenAPI contains approved operation IDs for every class-section, roster membership, and teacher-assignment operation implemented.
- Confirm every implemented route uses the documented `/api/v1` path, request schema, parameters, response status, and content type.
- Confirm OpenAPI documents required `ClassSection/Roster.code`, repeatable names, optional structured course/classroom/section/group metadata blocks with optional `code` and `name` only, and code uniqueness per school and academic period.
- Confirm OpenAPI documents lifecycle states: `ClassSection/Roster active/inactive`, `RosterMembership active/ended`, and `TeacherAssignment active/inactive`.
- Confirm OpenAPI documents that `ClassSection/Roster` creation starts `active`, inactive creation is rejected, and inactive records cannot be reactivated in v1.
- Confirm OpenAPI documents teacher list and detail visibility as read-only access to the authenticated teacher's own active assignments only.
- Confirm OpenAPI documents `GET /api/v1/teacher-assignments/{teacherAssignmentId}` or whichever teacher-assignment detail route OpenAPI approves.
- Confirm OpenAPI documents that roster membership creation and teacher assignment creation require explicit effective start dates.
- Confirm OpenAPI documents that lifecycle effective dates and creation effective start dates must be today-or-past and inside the selected academic period using the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone.
- Confirm OpenAPI documents roster membership eligibility as active same-school enrollment covering the membership effective start date.
- Confirm OpenAPI documents teacher assignment eligibility as active same-school teacher-compatible role on the assignment effective start date.
- Confirm OpenAPI documents that membership end effective dates and teacher assignment deactivation effective dates must be on or after their effective start dates.
- Confirm OpenAPI documents required reasons for roster inactivation, roster membership ending, and teacher assignment deactivation.
- Confirm OpenAPI documents roster inactivation rejection while active memberships or active teacher assignments exist.
- Confirm OpenAPI documents all-or-nothing roster membership batch rejection and the 100-change batch limit.
- Confirm OpenAPI documents transactional conflict responses for concurrent duplicate-code, membership, teacher-assignment, lifecycle, and dependency-sensitive writes.
- Confirm OpenAPI documents duplicate conflict responses for class-section code, overlapping active membership, and duplicate active teacher assignment.
- Confirm OpenAPI documents list pagination with default page size 25 and maximum page size 100, plus `academicPeriodId` and `status` filters only.
- Confirm OpenAPI does not approve include expansion, sort behavior, extra filters, or page sizes above 100 for v1 roster foundation lists.
- Confirm no backend route exposes fields, filters, include expansion, sorts, status codes, error responses, status values, lifecycle actions, batch modes, page-size behavior, or authorization exceptions absent from OpenAPI.

### 2. Tenant and authorization validation

- Confirm school-scoped roster requests resolve an active `school_id` context before class-section, membership, assignment, student, teacher, academic-period, duplicate, lifecycle, audit, or persistence logic.
- Confirm missing, inactive, mismatched, and unauthorized tenant contexts fail before service logic runs.
- Confirm school administrators can manage class sections, roster memberships, and teacher assignments only inside active permitted school context.
- Confirm teachers cannot manage teacher assignments, roster memberships, or class sections in this slice.
- Confirm platform administration does not imply school-owned roster access.
- Confirm students and guardians cannot use roster foundation administration operations.

### 3. ClassSection/Roster validation

- Confirm creation accepts only documented code, name, academic period, lifecycle status, and optional structured course/classroom/section/group metadata blocks with optional `code` and `name` only.
- Confirm creation creates `active` ClassSection/Roster records only and rejects requests to create inactive records.
- Confirm no separate internal Course/Classroom/Section/Group tables or top-level lifecycle resources are created in v1.
- Confirm `code` is required and unique per school and academic period.
- Confirm names may repeat in the same school and academic period.
- Confirm unsupported metadata shapes and metadata fields beyond optional `code` and `name` are rejected.
- Confirm missing, inactive, closed, cross-tenant, or incompatible academic periods are rejected.
- Confirm status values outside `active` and `inactive` are rejected.
- Confirm unsupported lifecycle transitions and dependency conflicts are rejected.
- Confirm inactive ClassSection/Roster reactivation is rejected in v1.
- Confirm inactivation requires a reason.
- Confirm inactivation is rejected while active memberships or active teacher assignments exist.

### 4. Roster membership validation

- Confirm roster membership creation accepts one or more documented student profile references.
- Confirm roster membership creation requires an explicit effective start date.
- Confirm every referenced student profile is active, same-school, not deleted, not transferred out, and has active same-school enrollment covering the membership effective start date.
- Confirm overlapping active memberships for the same student, roster, and academic period are rejected without changing the existing membership.
- Confirm batch membership changes are all-or-nothing when the request contains more than 100 requested changes or one member is invalid, inactive, transferred, deleted, cross-tenant, duplicate, overlapping, or lacks active same-school enrollment covering the membership effective start date.
- Confirm concurrent conflicting membership writes return the documented conflict response without partial state.
- Confirm ending a membership changes status to `ended` while preserving history, effective dates, actor context, required reason, and audit events.
- Confirm membership effective dates are today-or-past and inside the selected academic period using the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone.
- Confirm membership end effective dates before the membership effective start date are rejected.

### 5. Teacher assignment validation

- Confirm teacher assignment creation requires school administrator permission.
- Confirm teacher assignment creation requires an explicit effective start date.
- Confirm the assigned teacher user is active, same-school, not deleted, and has an active same-school teacher-compatible role on the assignment effective start date.
- Confirm the target class section/roster and academic period are active and same-school.
- Confirm duplicate active teacher assignments for the same teacher, class section/roster, and academic period are rejected without changing the existing assignment.
- Confirm teachers may list and retrieve only their own active teacher assignments.
- Confirm teachers cannot view other teachers' assignments.
- Confirm concurrent conflicting teacher-assignment writes return the documented conflict response without partial state.
- Confirm teacher assignment deactivation requires a reason.
- Confirm teacher assignment effective dates are today-or-past and inside the selected academic period using the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone.
- Confirm teacher assignment deactivation effective dates before the assignment effective start date are rejected.
- Confirm status values outside `active` and `inactive` are rejected.
- Confirm teachers, lead teachers, department managers, students, guardians, and platform administrators without school authorization cannot manage teacher assignments.

### 6. Legacy compatibility validation

- Confirm existing direct learning-set student assignments remain readable where current contracts already expose them.
- Confirm this slice does not add new direct student-assignment write behavior.
- Confirm roster foundation changes do not alter existing grades, attendance, student self-view, reporting, teacher correction, guardian self-service, or report lifecycle behavior.
- Confirm migration, backfill, or removal of legacy direct-assignment reads is not implemented without a future specification and OpenAPI update.

### 7. Audit and response-shape validation

- Confirm creation, update, roster inactivation, membership add/end, teacher assignment add/deactivate, required lifecycle reasons, duplicate conflict, oversized batch rejection, all-or-nothing batch rejection, concurrent conflict, blocked cross-tenant attempts, and lifecycle conflict outcomes are audited with actor user ID, school ID, target type and ID, action, outcome, lifecycle reason when present, and tenant-safe summary metadata.
- Confirm audit events do not store private student, teacher, credential, full request payload, or unauthorized cross-tenant details.
- Confirm successful JSON responses use documented success envelopes.
- Confirm list operations use documented paginated envelopes with default page size 25 and maximum page size 100.
- Confirm validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, unsupported filter/include/sort/page-size, oversized batch rejection, all-or-nothing batch rejection, and not-found cases use only the error envelopes or codes documented on each affected OpenAPI operation.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Implementation PRs should record the contract validation result, test result, feature id `009-classroom-roster-foundation`, and the operation IDs implemented.

## Exit Criteria for Planning

- The implementation boundary maps to approved OpenAPI operation IDs.
- Tenant and authorization rules are explicit for school-administrator-only management writes.
- `ClassSection/Roster`, active-only creation, inactive-roster reactivation rejection, structured metadata blocks, metadata field limits, roster membership, teacher assignment, student enrollment eligibility, teacher role eligibility, teacher read visibility, lifecycle state, lifecycle reason, explicit effective start dates, effective-date timezone behavior, date-order behavior, dependency conflict, duplicate conflict, transactional conflict handling, all-or-nothing batch, batch size limit, list filters, pagination limits, academic-period, audit, and legacy direct-assignment compatibility behavior are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, tenant isolation, authorization, validation, inactive statuses, inactive creation, inactive-roster reactivation, duplicate code conflicts, unsupported metadata fields, overlapping membership conflicts, duplicate teacher assignment conflicts, teacher own-active-assignment visibility, student enrollment effective-start eligibility, teacher role effective-start eligibility, missing effective start dates, invalid effective dates with timezone behavior, lifecycle-ending dates before effective starts, missing required lifecycle reasons, roster inactivation dependency conflicts, unsupported filter/include/sort/page-size behavior, oversized and invalid all-or-nothing batch rejection, transactional conflict handling, lifecycle history, audit event fields, and read-only legacy compatibility.
