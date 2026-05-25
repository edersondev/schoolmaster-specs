# Contract Boundary: Backend Student Profile and Enrollment Management

## Purpose

This feature creates a new public API surface for school-scoped student profile and enrollment management. Backend implementation must wait until OpenAPI documents the exact operations, parameters, request schemas, response schemas, errors, and operation IDs.

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

| Proposed Operation | Method and Path | Boundary |
|--------------------|-----------------|----------|
| `listStudentProfiles` | `GET /api/v1/student-profiles` | List student profiles visible in the resolved school scope with documented pagination, filters, and sorting |
| `createStudentProfile` | `POST /api/v1/student-profiles` | Create one same-school student profile and optional same-school guardian associations atomically |
| `getStudentProfile` | `GET /api/v1/student-profiles/{studentProfileId}` | Retrieve one same-school student profile using the documented response shape |
| `updateStudentProfileStatus` | `PATCH /api/v1/student-profiles/{studentProfileId}/status` | Apply an approved lifecycle status transition and write enrollment history atomically |
| `transferStudentProfile` | `POST /api/v1/student-profiles/{studentProfileId}/transfer` | Record source-school transfer behavior without exposing or moving source-school academic history across tenants |

No backend implementation may expose these or adjacent routes until OpenAPI documents them.

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths
- request schemas for create, status update, and transfer
- response schemas for profile summary, profile detail, enrollment history summary, and lifecycle outcomes
- documented pagination, filters, and sort options for profile listing
- validation errors for duplicate identifiers, invalid guardians, invalid status transitions, invalid effective dates, cross-tenant references, inactive references, and unsupported fields
- authorization and tenant-context errors for school-scoped access
- conflict response behavior for duplicate or incompatible lifecycle requests
- not-found behavior that does not reveal cross-tenant existence

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelope for create, detail, status update, and transfer outcomes
- paginated envelope for profile listing
- validation error envelope
- unauthorized response
- forbidden response for valid tenant but insufficient permission
- tenant mismatch or inactive tenant response
- conflict response for duplicate identifiers or incompatible lifecycle state
- not-found response that does not disclose cross-tenant records

For the student profile and enrollment operations in this slice, OpenAPI must document `403` outcomes that let clients distinguish a tenant-context failure (`error.code = tenant_mismatch`) from a valid-tenant permission failure (`error.code = forbidden`) without introducing undocumented status codes.

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented sort behavior, or undocumented status value is approved in this slice.

## Tenant Behavior

- Student profile operations use the documented `X-School-Id` tenant context behavior when the authenticated session is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context must fail before student profile lookup, guardian lookup, enrollment history lookup, transfer evaluation, persistence, or response shaping.
- Source-school historical academic records remain owned by the source school after transfer.
- Destination-school behavior is permitted only when OpenAPI documents it and the requester has explicit permission for that destination school context.

## Authorization Behavior

- All operations require authenticated access and active user status.
- All operations require an active permitted school context.
- School-scoped student administration permission is required for profile lifecycle operations.
- Platform-scope users do not receive implicit school-scoped profile or transfer authority.
- Teachers and students do not receive student profile administration access through teacher workflow or student self-view permissions.
- Guardian self-service access is not introduced in this slice.

## Validation Behavior

- Profile creation rejects duplicate same-school identifiers, malformed identity fields, unsupported statuses, undocumented fields, cross-tenant references, and invalid guardian references.
- Guardian references must resolve to active same-school guardians before any profile or association is created.
- Guardian association input must reject duplicate guardian references by `guardian_id`, even when the duplicate objects use different `relationship_type` values.
- Profile listing accepts only documented filters and sort options.
- Profile detail rejects missing, inactive, cross-tenant, unauthorized, or not-found profiles using documented error semantics.
- Status updates accept only documented lifecycle transitions and require effective dates, reasons, and actor context where documented.
- Transfers require an active source profile, documented transfer input, and explicit destination permissions when destination behavior is requested.
- Transfer must not copy source grades, attendance, learning sets, private content, report outputs, or guardian links across schools.

## Blocked Until Contract Expansion

These behaviors are outside this implementation boundary until OpenAPI documents them:

- frontend student administration implementation
- classroom, course, section, group, roster, or teacher assignment workflows
- guardian self-service or guardian student views
- student academic record correction workflows
- report request, report output, or reporting contract changes
- bulk import, merge, anonymization, permanent deletion, restore, or purge workflows
- account invitation, password setup, or password reset behavior
- additional filters, sorting options, response fields, status values, transfer modes, or authorization exceptions

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of the aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- authorization failures for platform/school scope separation and student administration permission
- validation failures for duplicate identifiers, invalid guardians, invalid lifecycle transitions, invalid effective dates, unsupported filters, unsupported sorts, undocumented fields, inactive references, and cross-tenant references
- atomicity checks for profile creation plus guardian associations
- atomicity checks for status or transfer changes plus enrollment history
- history preservation checks after inactivation and transfer
