# Contract Boundary: Backend API Foundation

## Purpose

This feature does not create a separate public API surface. It defines how the backend repository consumes and updates existing SchoolMaster contracts before implementation begins.

## Authoritative Contracts

- Aggregate contract: `specs/api/openapi.yaml`
- Active platform feature contract: `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `specs/AGENTS.md`
- Backend implementation guidance: `specs/docs/backend-guidelines.md`
- Multi-tenant guidance: `specs/docs/multi-tenant.md`
- Security guidance: `specs/docs/security.md`
- ADRs: `specs/decisions/*.md`

## First Product Slice Operations

The first backend product slice may implement only these published operations unless `/specs` is updated first:

| Operation ID | Method and Path | Boundary |
|--------------|-----------------|----------|
| `login` | `POST /api/v1/auth/login` | Authenticate a platform or school-scoped user according to the published request and `AuthSession` response shape |
| `getCurrentUser` | `GET /api/v1/auth/me` | Return authenticated user, resolved tenant, roles, and permissions |
| `logout` | `POST /api/v1/auth/logout` | Revoke continued access for the current bearer token after OpenAPI documents logout behavior |
| `listSchools` | `GET /api/v1/schools` | List schools visible to the requester through platform-scope authorization |
| `createSchool` | `POST /api/v1/schools` | Create a school tenant through platform-scope authorization |
| `getSchool` | `GET /api/v1/schools/{schoolId}` | Review one school tenant in permitted scope |
| `updateSchool` | `PATCH /api/v1/schools/{schoolId}` | Update school profile or operational status in permitted scope |

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

No backend-local success or error envelope may be introduced for product APIs without updating OpenAPI first.

## Required Authentication Contract Updates Before Coding

The clarified spec requires OpenAPI to document these before authentication implementation merges:

- bearer tokens expire after 8 hours
- logout operation and response semantics
- logout revokes continued token access
- inactive user status rejects authentication and continued access
- inactive school status rejects school-scoped authentication and continued access
- failed login attempts are tracked by both submitted email and source IP
- lockout occurs after 5 failed attempts for either key within 15 minutes
- lockout response code, status, envelope, and retry metadata
- token rejection response code, status, and envelope for expired, revoked, inactive-user, and inactive-school cases

## Required Tenant Behavior

- School-scoped requests use the OpenAPI `X-School-Id` tenant context behavior when the authenticated session is not already bound to exactly one active school.
- V1 school-owned records use `school_id` as the concrete tenant column.
- Missing, mismatched, inactive, or unauthorized tenant context returns the documented tenant-mismatch response before tenant-owned data access.
- Platform-scope school provisioning does not grant implicit school-scope access.

## Required Audit Boundary

The first backend slice must record audit events for:

- login success
- login failure
- logout
- token rejection
- school create
- school update
- school activation
- school deactivation

Audit records must include actor where available, affected school where applicable, action, outcome, timestamp, source IP where relevant, and tenant-safe metadata. Audit records must not store plaintext credentials or bearer token values.

## Non-Goals

- No setup-only endpoint is added.
- No frontend contract is created.
- No product route outside `/api/v1` is approved.
- No class, course, section, roster, teacher-content, questionnaire, learning-set, grade, attendance, or report endpoint is included in this first backend slice.
- Token rotation and refresh behavior remain out of scope unless `/specs` and OpenAPI are updated first.
