# Backend Guidelines

## Target

Backend implementation targets the `schoolmaster-backend` Laravel API
repository.

## Working Principles

- Keep controllers thin
- Place business logic in services
- Use requests for validation
- Use API resources for response shaping
- Align behavior with the published OpenAPI contract

## Multi-Tenancy

Tenant-aware backend code must follow the tenant-by-column strategy defined in
repository decisions. For v1 school-owned records, the concrete tenant column
is `school_id`; use `tenant_id` only as a generic architecture term unless a
future ADR introduces a different tenant root.

Backend implementation must enforce tenant scope in services, query scopes or
repositories, policies, API resources, and tests. Requests with missing,
mismatched, inactive, or unauthorized school context must fail before
module-specific business logic runs.

## Authentication and Authorization

- Use Laravel-native authentication mechanisms aligned to the published
  OpenAPI contract.
- Keep platform-scope and school-scope authorization paths explicit.
- Identify System Administrator only through the existing active
  platform-scoped role named exactly `System Administrator`.
- Centralize the permission-only override in `User::hasPermission()` and
  `User::hasSchoolPermission()`. Do not use `Gate::before`, because policies and
  services must still evaluate tenant, ownership, lifecycle, approval, and
  safety gates.
- For school-owned operations, resolve active `X-School-Id` context before the
  override is useful and keep every query and response selected-school scoped.
- Record `master_access_used: true` through existing audit pipelines for System
  Administrator writes and lifecycle actions.
- Expose role and permission information through API resources only as defined
  by the contract.

## Module Boundaries

- Keep controllers thin and route work through feature services.
- Use Form Requests for validation and API Resources for response shaping.
- Use DTOs when request input or service input has multiple coordinated fields.
- Use repositories or explicit query objects for complex tenant-scoped access.

## Classroom Roster Foundation

- `ClassSection/Roster` is the only v1 teaching-structure resource; do not add
  separate Course, Classroom, Section, Group, or Roster lifecycle resources.
- Store course, classroom, section, and group values as structured metadata
  blocks on `ClassSection/Roster`; each block may contain only optional `code`
  and `name` fields.
- Roster membership and teacher assignment writes must go through classroom
  roster services with transaction boundaries for duplicate, dependency, and
  all-or-nothing batch conflicts.
- Keep existing direct learning-set assignment behavior read-compatible; do not
  add new direct assignment write behavior in this slice.
- Expose only the operation IDs documented for feature
  `009-classroom-roster-foundation`.

## Teacher Workflow Lifecycle

- Expose teacher workflow lifecycle behavior only through the approved
  `/api/v1` OpenAPI operations for detail, update, status, delete, restore,
  download, correction, and import flows.
- Keep teacher workflow controllers thin and route lifecycle, correction,
  download, and import logic through dedicated services under
  `App\Services\TeacherWorkflow`.
- Use Form Requests to reject undocumented fields, unsupported lifecycle
  transitions, unsupported filters or sorts, immutable-field edits, and
  cross-tenant references before service logic mutates state.
- Use API Resources to shape teacher content, questionnaire, learning-set,
  grade, attendance, correction-history, and import-run responses exactly as
  defined in OpenAPI.
- Use DTOs for status transitions, restores, corrections, and import payloads
  when request fields must be validated and consumed together.
- Do not expose backend-only lifecycle shortcuts, direct file paths, private
  audit payloads, or undocumented route aliases.

## Teacher Workflow Business Rules

- Creating or owning teachers may manage only their own same-school teacher
  workflow records unless OpenAPI documents an explicit exception.
- School administrators may manage same-school teacher workflow records within
  an active permitted school context.
- Same-school non-owner teachers, students, guardians, and platform users
  without documented school-scoped authority must be denied teacher workflow
  management access.
- Teacher content downloads require clean scan status, an allowed active
  lifecycle state, same-school authority, and tenant-safe private delivery.
- Used content and questionnaires must reject edits that would change
  historical student-facing meaning in v1.
- New learning-set assignment writes must use roster-aware assignment scope;
  legacy direct selected-student assignments remain readable only where
  existing contracts already expose them.
- Grade and attendance corrections require a 10-500 character free-text reason,
  preserve original values and history, and allow closed-period corrections
  only for school administrators.
- Grade and attendance imports are school-administrator-only, JSON-only,
  create-only, capped at 500 rows, and must be all-or-nothing.

## Teacher Workflow Auditing

- Audit successful and denied teacher content downloads, lifecycle transitions,
  restores, corrections, imports, validation rejections, conflict outcomes,
  and blocked cross-tenant attempts.
- Audit records must store tenant-safe metadata only. Do not store private file
  contents, private storage paths, credentials, full request payloads, or
  unauthorized cross-tenant details.

## Backend Test Command

Run the Laravel test suite inside the backend application container:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Do not run `php artisan test` directly on the host for documented backend
verification, because the backend runtime and database drivers are provided by
the Docker container.

## Response and Error Conventions

Backend responses must follow the approved OpenAPI envelopes for successful
responses, validation failures, authorization failures, inactive-record
handling, tenant mismatches, and not-found outcomes.
