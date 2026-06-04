# Contract Boundary: Backend Guardian Self-Service

## Purpose

This feature adds the public API surface for guardian self-service read access. Backend implementation must wait until OpenAPI documents exact operations, parameters, request schemas, response schemas, errors, field visibility, tenant behavior, and operation IDs.

This feature also depends on a documented same-school school-admin guardian-user-link provisioning path so administrators can create and deactivate the link required for guardian self-service access.

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

| Resource/View | Proposed Operations | Boundary |
|---------------|---------------------|----------|
| Guardian student list | list permitted students for the authenticated guardian in the resolved school | Requires authenticated active user, explicit active guardian-user link, active same-school guardian record, active school context, and active guardian-student associations |
| Guardian student detail | retrieve limited profile and enrollment summary for a permitted student | Target-specific missing, unassociated, inactive, or cross-tenant students return the same not-found envelope |
| Guardian academic summary | retrieve current grade summary, attendance totals/status, and learning-set progress/status for a permitted student and explicit academic period | Requires explicit same-school academic period; no detailed rows, correction history, teacher content, student submissions, or report data |
| Guardian contact view | retrieve authenticated guardian contact fields, relationship labels, and student's primary school-approved contact details | No other guardian records, non-primary student contacts, restricted emergency handling details, or school-only notes |
| AuditEvent | side-effect recording for allowed reads, denied attempts, blocked cross-tenant attempts, and visibility-sensitive access | Tenant-safe metadata only; no private payloads, credentials, file paths, report outputs, school-only notes, or unauthorized cross-tenant details |

No backend implementation may expose these or adjacent routes until OpenAPI documents them.

## Proposed Route-to-Operation Mapping

OpenAPI may approve a different path or operation set, but backend implementation must remain limited to the approved contract.

| Method | Route | Proposed OpenAPI operation ID |
|--------|-------|-------------------------------|
| `GET` | `/api/v1/guardian/students` | `listGuardianStudents` |
| `GET` | `/api/v1/guardian/students/{studentProfileId}` | `getGuardianStudent` |
| `GET` | `/api/v1/guardian/students/{studentProfileId}/academics` | `getGuardianStudentAcademics` |
| `GET` | `/api/v1/guardian/students/{studentProfileId}/contacts` | `getGuardianStudentContacts` |

## Provisioning Prerequisite Mapping

If the existing school-admin contract does not already publish guardian-user-link lifecycle operations, this feature must add the minimal same-school admin surface required to provision guardian self-service:

| Method | Route | Proposed OpenAPI operation ID |
|--------|-------|-------------------------------|
| `POST` | `/api/v1/guardians/{guardianId}/user-links` | `createGuardianUserLink` |
| `POST` | `/api/v1/guardians/{guardianId}/user-links/{guardianUserLinkId}/deactivate` | `deactivateGuardianUserLink` |

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths for every approved guardian self-service operation
- required authentication and active school context behavior
- required `X-School-Id` tenant context behavior where applicable
- school-admin guardian-user-link provisioning or deactivation operations if they are not already available in the published admin contract
- explicit guardian-user link requirement
- active guardian record requirement
- active school-approved guardian-student association requirement
- request parameters for target student UUIDs, pagination, supported filters, supported sorts, and explicit academic period UUID where required
- response schemas for guardian student list items, guardian student detail summary, guardian academic summary, and guardian contact view
- academic summary fields limited to current grade summary, attendance totals/status, and learning-set progress/status where approved
- contact fields limited to authenticated guardian-owned contact fields, school-approved relationship labels, and the student's primary school-approved contact details
- hidden field rules for other guardians, non-primary contact details, restricted emergency handling details, school-only notes, detailed grade rows, detailed attendance rows, correction history, internal actor metadata, report outputs, teacher content, questionnaire answer keys, private file paths, malware-scan internals, and unauthorized cross-tenant identifiers
- uniform not-found envelope for target-specific missing, unassociated, inactive, or cross-tenant students
- empty-result behavior for guardian student listing when the guardian has no active approved associations in the resolved school
- tenant-context errors for missing, inactive, mismatched, or unauthorized school context
- validation errors for missing or unsupported academic period, unsupported include expansion, unsupported filters, unsupported sort values, and invalid payload shapes
- audit requirements for allowed reads, denied attempts, blocked cross-tenant attempts, and visibility-sensitive reads

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelopes for guardian student detail, academic summary, and contact view
- paginated success envelope for guardian student listing if pagination is approved
- empty-result success envelope for list operations where the guardian has no permitted students
- validation error envelope
- unauthorized response for unauthenticated or inactive actor access
- forbidden response for authenticated actors without guardian self-service permission where record existence is not target-specific
- tenant mismatch or inactive tenant response for school-scoped operations
- uniform not-found response for missing, unassociated, inactive, or cross-tenant target students
- conflict response only where OpenAPI explicitly documents a guardian self-service conflict condition

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented include expansion, undocumented sort behavior, undocumented denial detail, undocumented lifecycle rule, undocumented academic summary shape, undocumented contact field, or authorization exception is approved in this slice.

## Tenant Behavior

- Operations use documented `X-School-Id` tenant context behavior when the authenticated actor is not already bound to exactly one active school.
- V1 school-owned records use `school_id`.
- Missing, mismatched, inactive, or unauthorized tenant context fails before guardian lookup, guardian-user link lookup, guardian-student association lookup, student lookup, academic-period lookup, academic summary aggregation, contact lookup, or response shaping. Audit-safe denial recording may occur before return using only request metadata and safely resolved tenant context details.
- A guardian associated with students in multiple schools sees only the active resolved school context.
- Platform users do not receive implicit permission to view guardian self-service data or support-only school data.

## Authorization Behavior

- All operations require authenticated access and active actor status.
- Guardian self-service requires an active same-school guardian record explicitly linked to the authenticated user by a school administrator.
- School administrators need a documented same-school path to create and deactivate guardian-user links without granting guardian self-service target visibility to other actor types.
- Guardian access to a student requires an active school-approved guardian-student association in the resolved school.
- Guardian self-service permission does not grant school administration, guardian management, student self-view, teacher workflow, report, account lifecycle administration, roster, import, correction, platform support, or content download authority.
- Students, teachers, school administrators acting without a guardian-user link, and platform users do not receive guardian self-service target visibility unless OpenAPI and authorization explicitly approve a separate actor path.

## Validation Behavior

- Guardian student listing returns only active same-school student profiles associated through active guardian-student associations for the authenticated guardian.
- Guardian student detail returns only limited profile and enrollment summary fields documented by OpenAPI.
- Guardian academic summary requires an explicit same-school academic period.
- Guardian academic summary exposes only current grade summary, attendance totals/status, and learning-set progress/status where documented.
- Guardian contact view exposes only authenticated guardian-owned contact fields, school-approved relationship labels, and the student's primary school-approved contact details.
- Target-specific missing, unassociated, inactive, or cross-tenant students return the same not-found envelope.
- Unsupported filters, includes, sort values, undocumented request fields, invalid payload shapes, inactive references, deleted references, cross-tenant references, unsupported academic periods, and unassociated target attempts are rejected according to OpenAPI.
- Student transfer ends source-school current guardian self-service access unless OpenAPI explicitly documents limited source-school historical labels; destination-school access requires a separate active destination-school association.
- Guardian-facing responses never expose detailed academic rows, private correction reasons, full correction history, internal actor metadata, private teacher content, questionnaire answer keys, report-run data, report outputs, private file paths, malware-scan internals, other guardian records, non-primary student contact details, restricted emergency handling details, school-only notes, credentials, or unauthorized cross-tenant details.

## Blocked Until Future Specification

These behaviors are outside this implementation boundary until a future spec and OpenAPI update approve them:

- frontend implementation
- guardian self-claiming
- automatic contact matching
- invitation completion creating guardian-user links
- guardian profile or contact updates
- guardian association requests
- consent-signature capture
- custody dispute workflows
- legal-document upload or review
- multi-party approval workflows
- report request, report listing, report download, or report lifecycle behavior
- teacher content download or private file access
- student activity submission or questionnaire answers
- detailed grade rows or detailed attendance rows
- full correction history or private correction notes
- messaging, notification-center behavior, parent communication, billing, payment, payroll, or accounting
- platform support-user access to guardian self-service data
- permanent purge, anonymization, legal hold, merge, or retention override workflows
- guardian-facing restore behavior

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- explicit guardian-user link proof checks
- school-admin guardian-user-link create and deactivate coverage
- active guardian-student association checks
- denial checks for missing, unassociated, inactive, and cross-tenant target students returning the same not-found envelope
- guardian student list empty-result behavior
- explicit academic-period validation for guardian academic summaries
- summary-only academic field visibility
- limited contact field visibility
- hidden private correction, teacher content, report, contact, and school-only note fields
- student transfer visibility behavior
- guardian self-service read-only authorization checks
- audit event coverage for allowed reads, denied attempts, blocked cross-tenant attempts, and visibility-sensitive reads
