# Contract Boundary: Backend Classroom Roster Foundation

## Purpose

This feature creates a public API surface for school-owned classroom roster foundations. Backend implementation must wait until OpenAPI documents the exact operations, parameters, request schemas, response schemas, errors, and operation IDs.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend implementation guidance: `docs/backend-guidelines.md`
- Multi-tenant guidance: `docs/multi-tenant.md`
- Security guidance: `docs/security.md`
- Tenant decision: `decisions/004-use-tenant-by-column.md`

## Proposed Operation Boundary

The exact operation IDs must be finalized in OpenAPI, but the backend slice should be limited to this surface:

| Resource | Proposed Operations | Boundary |
|----------|---------------------|----------|
| `ClassSection/Roster` | list, create, detail, update, inactivate where approved | School-owned; active permitted school context required; creation creates `active` records only; inactive creation and inactive-roster reactivation rejected in v1; `code` unique per school and academic period; structured course/classroom/section/group metadata blocks with optional `code` and `name` only; no separate internal Course/Classroom/Section/Group tables; inactivation requires a reason and no active memberships or active teacher assignments |
| `RosterMembership` | list, batch add, batch end where approved | School-administrator-only writes; all-or-nothing batch behavior capped at 100 requested changes; explicit effective start date required on creation; active same-school student enrollment must cover the membership effective start date; no overlapping active membership for same student/roster/period; ending requires a reason and end effective date on or after the membership effective start date |
| `TeacherAssignment` | list, detail, create, deactivate where approved | School-administrator-only writes; teachers may list and retrieve only their own active assignments; explicit effective start date required on creation; active same-school teacher-compatible role required on the assignment effective start date; no duplicate active assignment for same teacher/roster/period; deactivation requires a reason and deactivation effective date on or after the assignment effective start date |
| `LegacyDirectAssignment` | read compatibility only where existing teacher workflow contracts already expose it | No new direct-assignment write behavior in this slice; migration/backfill/removal requires a future spec |
| `AuditEvent` | side-effect recording for approved transitions | Required fields: actor user ID, school ID, target type and ID, action, outcome, lifecycle reason when present, and tenant-safe summary metadata only; no private student, teacher, credential, full request payload, or unauthorized cross-tenant details |

No backend implementation may expose these or adjacent routes until OpenAPI documents them.

## Proposed Route-to-Operation Mapping

The backend implementation should expose only routes mapped to approved OpenAPI operation IDs in both authoritative contracts.

| Method | Route | Proposed OpenAPI operation ID |
|--------|-------|-------------------------------|
| `GET` | `/api/v1/class-sections` | `listClassSections` |
| `POST` | `/api/v1/class-sections` | `createClassSection` |
| `GET` | `/api/v1/class-sections/{classSectionId}` | `getClassSection` |
| `PATCH` | `/api/v1/class-sections/{classSectionId}` | `updateClassSection` |
| `PATCH` | `/api/v1/class-sections/{classSectionId}/status` | `updateClassSectionStatus` |
| `GET` | `/api/v1/class-sections/{classSectionId}/memberships` | `listClassSectionMemberships` |
| `POST` | `/api/v1/class-sections/{classSectionId}/memberships` | `batchAddClassSectionMemberships` |
| `PATCH` | `/api/v1/class-sections/{classSectionId}/memberships` | `batchEndClassSectionMemberships` |
| `GET` | `/api/v1/teacher-assignments` | `listTeacherAssignments` |
| `POST` | `/api/v1/teacher-assignments` | `createTeacherAssignment` |
| `GET` | `/api/v1/teacher-assignments/{teacherAssignmentId}` | `getTeacherAssignment` |
| `PATCH` | `/api/v1/teacher-assignments/{teacherAssignmentId}/status` | `updateTeacherAssignmentStatus` |

OpenAPI may approve a different path or operation set, but backend implementation must remain limited to the approved contract.

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths for every approved class-section, membership, and teacher-assignment operation
- request schemas for creating and updating `ClassSection/Roster` records, including required `code`, repeatable `name`, active academic period, and optional structured course/classroom/section/group metadata blocks with optional `code` and `name` only
- response schemas for class-section detail/list, membership list, membership batch result, teacher assignment detail/list, lifecycle transition outcomes, and audit-safe metadata where exposed
- `ClassSection/Roster.code` uniqueness per school and academic period, with names allowed to repeat
- lifecycle states: `ClassSection/Roster.active`, `ClassSection/Roster.inactive`, `RosterMembership.active`, `RosterMembership.ended`, `TeacherAssignment.active`, `TeacherAssignment.inactive`
- `ClassSection/Roster` lifecycle limits: creation starts `active`, inactive creation is rejected, and inactive records cannot be reactivated in v1
- teacher list and detail visibility limited to the teacher's own active assignments
- effective-date behavior: roster membership and teacher assignment creation require explicit effective start dates, and all effective dates must fall within the selected academic period and be today-or-past only using the resolved school's configured timezone with application default timezone fallback only when the school has no configured timezone
- roster membership eligibility requiring active same-school enrollment covering the membership effective start date
- teacher assignment eligibility requiring an active same-school teacher-compatible role on the assignment effective start date
- lifecycle-ending date ordering requiring membership end and teacher assignment deactivation effective dates to be on or after their effective start dates
- lifecycle reason requirements for roster inactivation, membership ending, and teacher assignment deactivation
- roster inactivation conflict behavior when active memberships or active teacher assignments exist
- all-or-nothing batch membership behavior, the 100-change batch limit, and the response envelope for oversized or invalid batch rejection
- duplicate conflict behavior for overlapping active memberships and duplicate active teacher assignments
- transactional conflict behavior for concurrent duplicate-code, membership, teacher-assignment, lifecycle, and dependency-sensitive writes
- school-administrator-only management write behavior
- active school context and active academic-period requirements
- pagination with default page size 25 and maximum page size 100, `academicPeriodId` and `status` filters only, no include expansion, and no sort behavior unless OpenAPI later documents it
- tenant-context errors for missing, inactive, mismatched, or unauthorized school context
- validation errors for unsupported metadata shapes, invalid lifecycle states, invalid lifecycle transitions, inactive creation, inactive-roster reactivation, lifecycle-ending dates before effective start dates, inactive references, duplicate references, cross-tenant references, students without active same-school enrollment covering the membership effective start date, and teachers without an active same-school teacher-compatible role on the assignment effective start date
- compatibility behavior for existing direct learning-set student assignments as read-only legacy records
- audit event required fields and tenant-safe fields safe for response or support review
- not-found behavior that does not reveal cross-tenant records

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelope for class-section, roster membership, and teacher assignment outcomes
- paginated envelope for list operations
- validation error envelope
- unauthorized response
- forbidden response for valid scope but insufficient permission
- tenant mismatch or inactive tenant response for school-scoped operations
- inactive-record response where the contract distinguishes inactive school, academic period, class section, student, teacher, or user state
- conflict response for duplicate class-section code, overlapping active membership, duplicate active teacher assignment, concurrent write conflict, dependency conflict, or unsupported current-state transition
- conflict response for roster inactivation while active memberships or active teacher assignments exist
- validation or conflict response for inactive class-section creation, inactive-roster reactivation attempts, and lifecycle-ending dates before effective start dates as OpenAPI documents
- oversized batch rejection response and all-or-nothing batch rejection response for invalid roster membership batches
- unsupported filter, include expansion, sort, or page-size response for list requests
- not-found response that does not disclose cross-tenant records

OpenAPI must document `403` outcomes that let clients distinguish a tenant-context failure (`error.code = tenant_mismatch`) from a valid-scope permission failure (`error.code = forbidden`) without introducing undocumented status codes.

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented include expansion, undocumented sort behavior, undocumented page-size behavior, undocumented status value, undocumented lifecycle action, undocumented batch result shape, or authorization exception is approved in this slice.

## Tenant Behavior

- Roster foundation operations use the documented `X-School-Id` tenant context behavior when the authenticated actor is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context must fail before class-section lookup, membership lookup, teacher assignment lookup, student lookup, teacher lookup, authorization checks, duplicate checks, lifecycle evaluation, persistence, audit creation, or response shaping.
- Platform administrators do not receive implicit permission to list, create, update, or mutate school-owned roster records.
- Cross-school student profiles, teacher users, academic periods, teacher content, learning sets, grades, attendance, guardians, reports, and private content remain isolated to their original school.

## Authorization Behavior

- All operations require authenticated access and active actor status.
- Management writes require school administrator permission under an active permitted school context.
- Teachers may list and retrieve only teacher assignments visible to them if OpenAPI documents the operation; they may not manage teacher assignments in this slice.
- Teacher read visibility is limited to list and detail retrieval for the authenticated teacher's own active assignments and does not expose other teacher assignments or roster memberships.
- Students and guardians do not receive roster foundation access through student self-view or guardian association permissions.
- Platform users do not receive school-owned roster access unless a future support-access specification grants an explicit exception.

## Validation Behavior

- `ClassSection/Roster` creation and update accept only documented code, name, academic period, status, and optional structured course/classroom/section/group metadata blocks with optional `code` and `name` only.
- `ClassSection/Roster` creation creates `active` records only; requests to create inactive records are rejected.
- `ClassSection/Roster.code` must be unique per school and academic period; names may repeat.
- Course, classroom, section, and group values are structured metadata blocks with optional `code` and `name` only and must not create separate internal tables or top-level lifecycle resources in this slice.
- `ClassSection/Roster` status is limited to `active` and `inactive`.
- Inactive `ClassSection/Roster` records cannot be reactivated in v1; administrators create a new roster if needed.
- Roster membership status is limited to `active` and `ended`.
- Teacher assignment status is limited to `active` and `inactive`.
- Roster inactivation requires a reason and is rejected while active memberships or active teacher assignments exist.
- Roster membership ending requires a reason.
- Teacher assignment deactivation requires a reason.
- Roster membership creation and teacher assignment creation require explicit effective start dates.
- Effective dates must fall within the selected academic period and must be today-or-past only using the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone.
- Roster membership end effective dates must be on or after the membership effective start date.
- Teacher assignment deactivation effective dates must be on or after the assignment effective start date.
- Roster membership creation requires active, same-school, non-deleted, non-transferred student profiles with active same-school enrollment covering the membership effective start date.
- Roster membership batches reject the whole request if the request contains more than 100 requested changes or if any member is invalid, inactive, transferred, deleted, cross-tenant, duplicate, overlapping, or lacks active same-school enrollment covering the membership effective start date.
- Overlapping active memberships for the same student, roster, and academic period are rejected without changing the existing membership.
- Teacher assignment creation requires active, same-school, non-deleted users with an active same-school teacher-compatible role on the assignment effective start date, plus an active same-school class section/roster and academic period.
- Duplicate active assignments for the same teacher, class section/roster, and academic period are rejected without changing the existing assignment.
- Lifecycle transitions reject unsupported transitions, inactive-roster reactivation, future effective dates, effective dates outside the selected academic period, lifecycle-ending effective dates before the target record's effective start date, missing required reasons, dependency conflicts, and attempts that would orphan active memberships or assignments.
- Concurrent conflicting roster code, membership, teacher-assignment, lifecycle, or dependency-sensitive writes are resolved transactionally so one request succeeds and conflicting requests return the documented conflict response without partial state.
- List operations support pagination with default page size 25 and maximum page size 100, plus `academicPeriodId` and `status` filters only. Unsupported filters, include expansion, sort values, and page-size values above 100 are rejected.
- Existing direct learning-set student assignments remain readable as legacy records, but no new direct student-assignment write behavior is approved by this slice.

## Blocked Until Contract Expansion

These behaviors are outside this implementation boundary until OpenAPI documents them:

- separate top-level or internal Course, Classroom, Section, or Group lifecycle resources
- teacher workflow correction behavior for grades, attendance, questionnaires, learning sets, or content
- roster-aware learning-set assignment writes beyond the foundation contract
- guardian self-service or guardian student views
- student academic record correction workflows
- report retry, cancellation, deletion, restore, designer, custom reports, additional formats, or report output lifecycle changes
- platform support-user access to school-owned roster records
- frontend implementation
- messaging, notifications, parent portal communication, billing, or undocumented APIs
- permanent purge, anonymization, legal hold, merge, or restore behavior
- additional filters, include expansion, sorting options, response fields, status values, lifecycle actions, batch modes, page-size behavior, or authorization exceptions
- inactive-roster reactivation, inactive class-section creation, lifecycle-ending dates before effective start dates, whole-period enrollment requirements, or whole-period teacher role requirements

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of the aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- authorization failures for school-administrator-only management writes
- validation failures for duplicate class-section codes, unsupported metadata fields, invalid lifecycle states, invalid lifecycle transitions, inactive class-section creation, inactive-roster reactivation, unsupported request fields, unsupported filters, unsupported include expansion, unsupported sort values, unsupported page-size values above 100, inactive references, students without active same-school enrollment covering the membership effective start date, teachers without an active same-school teacher-compatible role on the assignment effective start date, duplicate references, and cross-tenant references
- validation failures for missing effective start dates, future effective dates evaluated with school timezone or application default fallback, effective dates outside the selected academic period, lifecycle-ending effective dates before the target record's effective start date, and missing required lifecycle reasons
- oversized roster membership batch rejection checks
- transactional conflict checks for concurrent duplicate-code, membership, teacher-assignment, lifecycle, and dependency-sensitive writes
- roster inactivation dependency conflict checks
- teacher own-active-assignment read visibility checks
- all-or-nothing roster membership batch rejection checks
- overlapping active membership conflict checks
- duplicate active teacher assignment conflict checks
- lifecycle history preservation checks
- read-only legacy direct-assignment compatibility checks
- audit event coverage for actor user ID, school ID, target type and ID, action, outcome, lifecycle reason when present, and tenant-safe summary metadata without private student, teacher, credential, full request payload, or unauthorized cross-tenant details
