# Contract Boundary: Backend School Administration Foundation

## Purpose

This feature does not create a new public API surface. It defines the backend implementation boundary for the remaining P1 school administration operations already published in the SchoolMaster OpenAPI contracts.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend implementation guidance: `docs/backend-guidelines.md`
- Multi-tenant guidance: `docs/multi-tenant.md`
- Security guidance: `docs/security.md`
- Tenant decision: `decisions/004-use-tenant-by-column.md`

## Approved Operation Boundary

The backend may implement only these operation IDs in this slice unless `/specs` and OpenAPI are updated first:

| Operation ID | Method and Path | Boundary |
|--------------|-----------------|----------|
| `listUsers` | `GET /api/v1/users` | List users visible in the permitted platform or school scope |
| `createUser` | `POST /api/v1/users` | Create a user with valid same-scope role assignments |
| `listRoles` | `GET /api/v1/roles` | List roles and inherited permissions visible in permitted scope |
| `createRole` | `POST /api/v1/roles` | Create a platform or school role without mixing incompatible permissions |
| `listPermissions` | `GET /api/v1/permissions` | List permission definitions available to the requester |
| `listAcademicYears` | `GET /api/v1/academic-years` | List academic years for the resolved school |
| `createAcademicYear` | `POST /api/v1/academic-years` | Create an academic year in the resolved school |
| `listAcademicPeriods` | `GET /api/v1/academic-periods` | List academic periods for the resolved school, optionally filtered by academic year |
| `createAcademicPeriod` | `POST /api/v1/academic-periods` | Create a period inside a same-school academic year |
| `listGuardians` | `GET /api/v1/guardians` | List guardians for the resolved school |
| `createGuardian` | `POST /api/v1/guardians` | Create a same-school guardian record and optional student associations |

## Required Response Shapes

Backend implementation must follow the OpenAPI response components:

- `SuccessEnvelope`
- `PaginatedEnvelope`
- `ErrorEnvelope`
- `ValidationError`
- `Unauthorized`
- `Forbidden`
- `TenantMismatch`
- `NotFound`

No backend-local product envelope, ad hoc error response, undocumented field, undocumented filter, or undocumented sort behavior is approved in this slice.

## Tenant Behavior

- School-scoped operations use the documented `X-School-Id` tenant context behavior when the authenticated session is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context must fail before school-owned data access.
- Platform-scope users do not receive implicit permission to perform school-scoped administration without an explicit permitted school context.

## Authorization Behavior

- Users receive permissions through roles only.
- School-scoped roles can include only school-scope permissions and require the resolved school context.
- Platform-scoped roles can include only platform-scope permissions and must not be created through school-scoped flows.
- Inactive roles or permissions cannot be assigned to new users or roles.

## Validation Behavior

- User creation rejects cross-tenant roles, inactive roles, incompatible role scopes, duplicate ambiguous identity input, and undocumented fields.
- Academic year creation rejects invalid date ranges and tenant mismatches.
- Academic period creation rejects parent years from another school, periods outside the parent date range, duplicate period sequences within the academic year, and undocumented fields.
- Guardian creation rejects missing required relationship fields and any missing, inactive, or cross-tenant student profile reference without partial association persistence.

## Blocked Until Contract Expansion

These behaviors are outside this implementation boundary until OpenAPI documents them:

- user, role, academic year, academic period, or guardian detail endpoints
- update, deactivate, activate, delete, restore, or bulk operations
- invitation, password reset, password setup, or account lifecycle workflows
- student profile creation or student self-service
- teacher content, questionnaire, learning set, grade, attendance, or report workflows
- additional filters, sorting options, response fields, or authorization exceptions

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of the aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- authorization failures for incompatible platform and school scopes
- validation failures for roles, permissions, academic date/sequence rules, and guardian associations
