# Research: Backend API Foundation

## Decision: Treat `/specs` as the backend source of truth

**Rationale**: Backend `AGENTS.md`, `/specs/AGENTS.md`, and the repository consumption guide all require backend work to follow the specification repository, OpenAPI contracts, docs, and decisions before implementation. This avoids backend-only business rules and keeps frontend/backend contracts synchronized.

**Alternatives considered**:

- Use backend-local README or code conventions as primary source: rejected because `/specs` explicitly wins when conflicts exist.
- Resolve conflicts inside backend implementation: rejected because `/specs` requires business-rule and API changes to be made in the specification repository first.

## Decision: Use API-only backend readiness as a separate setup boundary

**Rationale**: The current repository is a Laravel skeleton. The foundation must first establish API-only behavior, `/api/v1` routing boundaries, backend setup documentation, environment expectations, and missing architecture folders before product endpoints are coded.

**Alternatives considered**:

- Start directly with authentication endpoints: rejected because route, response, contract, tenancy, security, and setup boundaries are not yet aligned.
- Keep web routes and Blade views as harmless defaults: rejected for product behavior because backend scope says API-only and no product Blade views.

## Decision: Follow v1 `school_id` tenancy instead of generic `tenant_id`

**Rationale**: ADR 004 defines tenant-by-column generally, but v1 uses `School` as tenant root and `school_id` as the concrete tenant column for school-owned records. Backend guidelines say `tenant_id` is only a generic architecture term unless a future ADR changes the tenant root.

**Alternatives considered**:

- Use a literal `tenant_id` column for all tenant-owned tables: rejected because it conflicts with ADR 004 and the active platform spec.
- Hide tenant ownership behind only parent traversal: rejected as a default because direct `school_id` is required unless a documented parent ownership rule exists.

## Decision: Keep authentication Laravel-native and contract-bound

**Rationale**: ADR 002 requires Laravel-native authentication. The aggregate OpenAPI contract defines bearer authentication and `AuthSession.token`, while Laravel 13 documentation describes Sanctum as the native route guard pattern for API token and session-backed authentication. The backend may use Laravel-native auth mechanisms only where they match the published contract and clarified security rules.

**Alternatives considered**:

- Choose a custom token implementation now: rejected because it would add security semantics outside the accepted Laravel-native direction.
- Add or assume an extra package without approval: rejected because backend agent rules say not to install extra packages unless requested.
- Leave token lifecycle unspecified: rejected by clarification; token expiry and revocation behavior are now product rules for the first slice.

## Decision: First-slice bearer tokens expire after 8 hours and are revoked by logout or inactive context

**Rationale**: An 8-hour lifetime fits a school-day work session while limiting risk from leaked tokens. Logout revocation and inactive user/school revocation make access termination explicit and testable.

**Alternatives considered**:

- 24-hour tokens: rejected as broader exposure than needed for the first school SaaS backend slice.
- Tokens valid until logout only: rejected because it lacks a time-bounded safety limit.
- Defer token lifecycle: rejected because authentication is part of the first implementation boundary.

## Decision: Failed login attempts are rate-limited by both email and IP

**Rationale**: Email-based limits protect individual accounts from distributed attempts, while IP-based limits slow repeated abuse from one source. The clarified threshold is 5 failed attempts per email or IP within 15 minutes.

**Alternatives considered**:

- Generic API throttling only: rejected because login needs security-specific behavior and acceptance tests.
- IP-only throttling: rejected because it does not protect a targeted account from attempts across multiple IPs.
- Defer rate limiting: rejected because login is the first exposed security-sensitive operation.

## Decision: Audit authentication and school lifecycle events in the first slice

**Rationale**: Authentication and school lifecycle changes are security-sensitive tenant boundary events. Audit coverage must include login success, login failure, logout, token rejection, and school create/update/status changes, without storing plaintext credentials or bearer token values.

**Alternatives considered**:

- Audit only school changes: rejected because authentication incidents would lack traceability.
- Audit only failures and lockouts: rejected because successful login and logout events are needed for session review.
- Defer auditing: rejected because the first slice establishes security and tenant lifecycle foundations.

## Decision: Use OpenAPI envelopes as the response source

**Rationale**: OpenAPI already defines `SuccessEnvelope`, `PaginatedEnvelope`, `ErrorEnvelope`, `ValidationError`, `Unauthorized`, `Forbidden`, `TenantMismatch`, `NotFound`, and related response structures. Backend setup should reference these instead of creating module-specific response conventions.

**Alternatives considered**:

- Create a backend-local response standard first: rejected because it could drift from the frontend contract.
- Let each controller shape responses independently: rejected because the active spec requires consistent success and failure patterns.

## Decision: First product implementation boundary is auth and school management

**Rationale**: The active platform spec prioritizes tenant onboarding before other workflows. The first slice includes `login`, `getCurrentUser`, logout revocation, `listSchools`, `createSchool`, `getSchool`, and `updateSchool` once OpenAPI documents logout behavior, which establish platform identity and tenant lifecycle foundations.

**Alternatives considered**:

- Start with users and roles: rejected because they depend on tenant foundation and authenticated identity.
- Start with academic structures: rejected because school tenant management and tenant context must exist first.
- Start with teacher content or reports: rejected because they depend on authentication, authorization, tenant context, users, academic periods, and storage/report rules.

## Decision: Do not create a new OpenAPI file for this setup feature

**Rationale**: This feature defines backend readiness and implementation boundaries, not a separate product API surface. The authoritative API contract remains `specs/api/openapi.yaml` and the active feature contract remains `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`.

**Alternatives considered**:

- Generate a new OpenAPI file for backend readiness: rejected because it would duplicate or compete with the aggregate contract.
- Add setup-only endpoints to OpenAPI: rejected because setup behavior is internal and not product API behavior.

## Decision: Require OpenAPI updates before authentication coding

**Rationale**: The clarified auth behavior affects externally visible outcomes: token expiry, logout revocation, inactive-context token rejection, and lockout response. OpenAPI must document these responses before backend implementation merges.

**Alternatives considered**:

- Treat these as implementation-only security details: rejected because clients must handle expired, revoked, and locked-out authentication flows consistently.
- Implement first and update OpenAPI later: rejected by the contract-first workflow.
