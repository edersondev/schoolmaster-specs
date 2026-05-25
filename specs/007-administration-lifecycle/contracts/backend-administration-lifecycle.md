# Contract Boundary: Backend Administration Lifecycle Management

## Purpose

This feature creates a public API surface for administration lifecycle management across existing platform and school-administration resources. Backend implementation must wait until OpenAPI documents the exact operations, parameters, request schemas, response schemas, errors, and operation IDs.

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
| `School` | get, update, activate, deactivate, soft delete, restore | Platform-scoped tenant-root lifecycle only; does not grant school-owned module access |
| `User` | get, update, activate, deactivate, soft delete, restore, selected bulk lifecycle | School-scoped administrative lifecycle without invitation, password, recovery, token, or direct permission-assignment behavior |
| `Role` | get, update, activate, deactivate, soft delete, restore, selected bulk lifecycle | Preserve platform/school role scope and permission compatibility |
| `AcademicYear` | get, update, activate, deactivate, soft delete, restore, selected bulk lifecycle | Preserve date rules, child period compatibility, and academic history |
| `AcademicPeriod` | get, update, activate, deactivate, soft delete, restore, selected bulk lifecycle | Preserve parent year containment, sequence uniqueness, and academic history |
| `Guardian` | get, update, activate, deactivate, soft delete, restore, selected bulk lifecycle | Preserve same-school student profile associations and authorized history |

No backend implementation may expose these or adjacent routes until OpenAPI documents them.

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths for every approved resource/action pair
- request schemas for update, activate, deactivate, soft delete, restore, and selected bulk lifecycle actions
- response schemas for single-resource detail, lifecycle outcomes, lifecycle history summary where exposed, and bulk lifecycle outcomes
- documented mutable fields for each resource update operation
- immutable fields that requests must not submit or change
- lifecycle status values and allowed transitions for each affected resource
- required reason, effective date, and actor-context behavior for lifecycle operations
- school lifecycle request rules that state which school activation, deactivation, soft-delete, and restore operations require a reason and effective date
- documented dependency conflict codes and messages for blocked transitions
- validation errors for invalid transitions, invalid effective dates, duplicate identifiers, dependency conflicts, unsupported fields, unsupported actions, mixed-scope bulk requests, inactive references, and cross-tenant references
- authorization and tenant-context errors for platform-scoped and school-scoped access
- bulk request maximum count, all-or-nothing semantics, and duplicate-identifier behavior
- not-found behavior that does not reveal cross-tenant existence

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelope for detail, update, activation, deactivation, soft delete, restore, and bulk lifecycle outcomes
- validation error envelope
- unauthorized response
- forbidden response for valid scope but insufficient permission
- tenant mismatch or inactive tenant response for school-owned operations
- inactive-record response where the contract distinguishes inactive resource state
- conflict response for dependency-blocked or incompatible lifecycle state
- not-found response that does not disclose cross-tenant records

OpenAPI must document `403` outcomes that let clients distinguish a tenant-context failure (`error.code = tenant_mismatch`) from a valid-scope permission failure (`error.code = forbidden`) without introducing undocumented status codes.

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented sort behavior, undocumented status value, or undocumented lifecycle action is approved in this slice.

## Tenant Behavior

- School-owned resource operations use the documented `X-School-Id` tenant context behavior when the authenticated session is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context must fail before school-owned detail lookup, update validation, dependency checks, lifecycle evaluation, bulk evaluation, persistence, or response shaping.
- `School` lifecycle operations are platform-scoped tenant-root operations and must not expose school-owned module records.
- Platform-scope users do not receive implicit permission to perform school-owned user, role, academic, guardian, student, teacher, report, or content lifecycle actions.

## Authorization Behavior

- All operations require authenticated access and active user status.
- School lifecycle operations require platform-scoped school administration permission.
- School-owned resource lifecycle operations require an active permitted school context and school-scoped administration permission for the affected resource family.
- User lifecycle operations do not grant invitation, password setup, password reset, recovery, lock recovery, token refresh, or direct per-user permission assignment behavior.
- Teachers, students, and guardians do not receive administration lifecycle access through teacher workflow, student self-view, or guardian association permissions.
- Any future platform support override must be separately specified, documented in OpenAPI, and covered by regression tests.

## Validation Behavior

- Detail operations accept only documented identifiers and return only records visible in the current permitted scope.
- Update operations accept only documented mutable fields and reject immutable identifiers, ownership fields, role scope changes, parent ownership changes, undocumented fields, unsupported relationships, and cross-tenant references.
- Activation, deactivation, soft delete, and restore operations accept only documented transitions for the current resource state.
- Lifecycle operations require reason, effective date, and actor context where OpenAPI documents them.
- Dependency checks must reject transitions that would orphan active users, invalidate role assignments, break academic periods, invalidate grades or attendance, expose guardian or student history, invalidate report references, or violate inactive-school rules.
- Restore operations must validate active parent context, uniqueness constraints, dependency eligibility, and tenant or platform scope before making records available for new operational use.
- Bulk lifecycle operations reject mixed resource types, mixed actions, mixed tenant scopes, duplicate identifiers, over-limit record sets, unsupported actions, dependency-blocked records, unauthorized records, missing records, and cross-tenant records.
- Bulk lifecycle operations use all-or-nothing behavior: any invalid selected record prevents every selected record from changing.

## Blocked Until Contract Expansion

These behaviors are outside this implementation boundary until OpenAPI documents them:

- account invitation, password setup, password reset, account recovery, lock recovery, token refresh, or direct per-user permission assignment
- classroom, course, section, group, roster, or teacher assignment workflows
- teacher grade, attendance, questionnaire, learning-set, or content correction workflows
- guardian self-service or guardian student views
- student academic record correction workflows
- report retry, cancellation, deletion, restore, designer, custom reports, additional formats, or report output lifecycle changes
- platform support-user access to school-owned records
- frontend implementation
- permanent purge, anonymization, legal hold, or retention management
- billing, messaging, notifications, parent portal behavior, or undocumented APIs
- additional filters, sorting options, response fields, status values, lifecycle actions, bulk modes, or authorization exceptions

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of the aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- authorization failures for platform/school scope separation and resource-family permissions
- validation failures for immutable fields, invalid lifecycle transitions, invalid effective dates, unsupported fields, duplicate identifiers, unsupported filters/sorts where applicable, inactive references, dependency conflicts, and cross-tenant references
- dependency conflict checks for users, roles, academic years, academic periods, guardians, and schools
- atomicity checks for lifecycle state change plus lifecycle history write
- soft-delete and restore checks
- bulk all-or-nothing checks
- history preservation checks after deactivation, soft delete, and restore
