# Multi-Tenant Strategy

## Current Decision

SchoolMaster multi-tenancy uses a tenant-by-column strategy. For v1, `School`
is the tenant root and school-owned records use `school_id` as the tenant
column unless ownership is inherited through a strictly school-owned parent
record with documented traversal rules.

This follows [ADR 004](../decisions/004-use-tenant-by-column.md).

## Scope

- Tenant-owned data is isolated by school identity.
- Backend queries, repositories, services, policies, and tests must enforce
  tenant boundaries.
- Frontend behavior must respect tenant-scoped data flows exposed by the API
  and must not infer authorization from client-side state alone.
- OpenAPI documents tenant-context behavior for school-scoped endpoints.

## Repository Impact

- Specifications define tenant rules and constraints
- OpenAPI documents tenant-aware contract behavior
- Laravel API enforces tenant isolation
- Vue 3 SPA consumes tenant-safe endpoints

## Tenant Context Resolution

- School-scoped requests resolve an active tenant before module-specific
  business logic runs.
- The API contract uses the `X-School-Id` header for explicit tenant context
  where a request is not already bound to exactly one active school.
- Requests with missing, mismatched, inactive, or unauthorized tenant context
  are rejected before data access.

## Classroom Roster Foundation

- `ClassSection/Roster`, `RosterMembership`, and `TeacherAssignment` records are
  school-owned and must carry `school_id`.
- `X-School-Id` resolution must complete before class-section, membership,
  assignment, student, teacher, academic-period, duplicate, lifecycle,
  persistence, audit, or response lookup.
- School administrators may manage class sections, roster memberships, and
  teacher assignments only inside an active permitted school context.
- Teachers may read only their own active teacher assignments where OpenAPI
  documents that visibility; this does not grant roster membership, class
  section, or other teacher assignment visibility.
- System Administrator may select any active school through the documented
  tenant-context mechanism. This satisfies permission checks only; roster
  lookup, tenant ownership, lifecycle, and response scoping remain bound to the
  resolved school.

## Platform Access

- System administrators operate at platform scope and may resolve any active
  school through `X-School-Id` for school-scoped work.
- The master role satisfies feature-specific school permission checks only
  after active tenant context resolves. Queries, validation, authorization,
  persistence, audit, and response shaping remain selected-school scoped.
- Cross-school output is allowed only for operations explicitly documented as
  platform-wide in the specification and OpenAPI contract.

## Shared and Tenant-Owned Resources

- `Permission` records are shared capability definitions controlled at platform
  scope and filtered by role scope.
- School-scoped roles, users, academic structures, guardians, content,
  questionnaires, learning sets, grades, attendance, and report requests are
  tenant-owned.
- Tenant-owned records use status fields and soft-delete support where recovery
  or audit history is relevant.

## Teacher Workflow Tenant Rules

- Teacher content, questionnaires, learning sets, learning-set assignments,
  grades, attendance, correction records, import runs, and teacher workflow
  audit events are school-owned through `school_id`.
- `X-School-Id` resolution must complete before teacher workflow lookup, file
  access, roster lookup, student lookup, teacher lookup, academic-period
  lookup, duplicate checks, dependency checks, authorization, persistence,
  audit writes, import validation, or response shaping.
- Same-school ownership checks are mandatory for teacher actors on teacher
  content, questionnaires, learning sets, grades, and attendance. Teacher
  scope is owner or creator bound, not school-wide.
- School administrators may manage same-school teacher workflow records only
  inside an active permitted school context.
- System Administrator satisfies teacher-workflow permission checks inside a
  resolved active school, while tenant ownership, file safety, historical
  meaning, closed-period, validation, and lifecycle rules remain enforceable.
- Cross-tenant not-found and tenant-mismatch behavior must avoid revealing
  another school's record existence through validation, authorization, download,
  import, or correction responses.
