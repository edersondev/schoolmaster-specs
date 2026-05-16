# Data Model: Backend API Foundation

## Modeling Principles

- This feature models backend readiness boundaries and the first product slice; it does not create product entities beyond the active platform spec unless needed for authentication security or audit behavior clarified in this feature.
- Public and cross-boundary product identifiers use UUIDs as defined in OpenAPI and the active platform data model.
- `School` is the v1 tenant root. School-owned records use `school_id` unless a future spec explicitly documents a different ownership path.
- Status and inactive handling are product-visible behavior and must be represented consistently in models, policies, services, resources, and tests when implemented.
- Audit records must avoid plaintext credentials, bearer token values, and cross-tenant leakage.

## Entities

### BackendReadinessBoundary

- **Purpose**: Repository-level readiness agreement for backend implementation.
- **Core attributes**:
  - source_of_truth_paths: `/specs/AGENTS.md`, `/specs/specs`, `/specs/api/openapi.yaml`, `/specs/docs`, `/specs/decisions`
  - api_only_scope: product behavior exposed only through documented `/api/v1` API contracts
  - setup_status: not_ready, ready_for_foundation, ready_for_first_slice
  - validation_status: unchecked, passed, failed
- **Relationships**:
  - references `ApiContract`
  - references `TenantContext`
  - governs first-slice implementation boundaries
- **Validation rules**:
  - all required `/specs` inputs must be present before implementation planning completes
  - any conflict must be resolved in `/specs`, not locally in backend code
  - product endpoints outside approved OpenAPI scope are invalid
- **State transitions**:
  - `not_ready -> ready_for_foundation` after source-truth files and setup expectations are documented
  - `ready_for_foundation -> ready_for_first_slice` after contract validation and backend setup checks pass
  - any missing source-truth or contract mismatch returns the boundary to `not_ready`

### ApiContract

- **Purpose**: Published frontend/backend communication contract consumed by backend implementation.
- **Core attributes**:
  - aggregate_contract_path: `specs/api/openapi.yaml`
  - active_feature_contract_path: `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
  - approved_operations: `login`, `getCurrentUser`, `listSchools`, `createSchool`, `getSchool`, `updateSchool` for the first product slice
  - response_envelopes: success, paginated, validation error, unauthorized, forbidden, tenant mismatch, not found
  - required_auth_updates: token expiry, logout revocation, inactive-context token rejection, login lockout response
- **Relationships**:
  - defines `AuthenticationSession`
  - defines `School` API shape
  - defines `TenantContext` header behavior
  - defines client-visible auth failure responses
- **Validation rules**:
  - must pass executable OpenAPI validation before dependent implementation merges
  - backend must not accept or emit fields absent from the applicable schema
  - product routes must use the documented `/api/v1` path and operation semantics
  - clarified auth security responses must be documented before coding

### AuthenticationSession

- **Purpose**: Authenticated identity returned by the published authentication contract.
- **Core attributes**:
  - token
  - token_expires_at or equivalent documented expiry semantics
  - user
  - resolved_school
  - roles
  - permissions
- **Relationships**:
  - belongs to one `User`
  - may resolve one active `School`
  - includes zero or more `Role` records
  - includes permissions inherited through active roles
  - produces `AuditEvent` records for login success, logout, and token rejection
- **Validation rules**:
  - inactive users cannot authenticate or continue protected workflows
  - users tied to inactive schools cannot authenticate or continue school-scoped workflows
  - bearer tokens expire after 8 hours
  - logout revokes access
  - expired, logged-out, inactive-user, and inactive-school tokens are rejected
- **State transitions**:
  - unauthenticated -> authenticated when credentials, lockout, status, and tenant checks pass
  - authenticated -> expired after 8 hours
  - authenticated -> revoked on logout
  - authenticated -> rejected if user or resolved school becomes inactive

### LoginAttemptControl

- **Purpose**: Failed-login control used to protect accounts and source IPs.
- **Core attributes**:
  - submitted_email
  - source_ip
  - failed_attempt_count
  - window_started_at
  - locked_until
  - outcome
- **Relationships**:
  - may reference `User` when the submitted email maps to a known user without leaking existence in responses
  - produces `AuditEvent` records for login failure and lockout outcomes
- **Validation rules**:
  - failed attempts are counted by both submitted email and source IP
  - lockout occurs after 5 failed attempts for either key within 15 minutes
  - lockout response must be documented in OpenAPI before coding
  - responses must not reveal whether a submitted email belongs to a tenant or user
- **State transitions**:
  - clear -> counting after first failed attempt
  - counting -> locked after 5 failed attempts within 15 minutes
  - locked -> clear after lockout window or approved reset behavior defined by contract

### TenantContext

- **Purpose**: Active school context required before school-scoped behavior accesses tenant-owned data.
- **Core attributes**:
  - school_id
  - source: authenticated session, `X-School-Id` header, or route-bound school operation as documented by OpenAPI
  - resolution_status: resolved, missing, mismatched, inactive, unauthorized
- **Relationships**:
  - resolves to one active `School`
  - applies to school-scoped `User`, `Role`, and future school-owned records
  - produces `AuditEvent` records when token rejection is caused by tenant status or mismatch where safe to record
- **Validation rules**:
  - missing, mismatched, inactive, or unauthorized context is rejected before module-specific service logic
  - tenant mismatch responses must not reveal cross-tenant resource existence
  - system administrator platform access is not an implicit school-scope bypass
- **State transitions**:
  - unresolved -> resolved after authenticated user and tenant checks pass
  - resolved -> rejected if school or user becomes inactive or loses permission

### School

- **Purpose**: V1 tenant root and first product-management entity.
- **Core attributes**:
  - id
  - name
  - code
  - status
  - contact_email
  - contact_phone
  - address_summary
- **Relationships**:
  - owns school-scoped users, roles, academic structures, guardians, content, questionnaires, learning sets, grades, attendance, and report requests in later slices
  - may be returned as `resolved_school` in an authentication session
  - produces `AuditEvent` records for create, update, activation, and deactivation
- **Validation rules**:
  - name and code are required on creation
  - code uniqueness follows the active platform contract and data model
  - status uses the OpenAPI `active` / `inactive` catalog for this first slice
  - inactive schools reject school-scoped operational workflows and invalidate continued access for users tied to that school
- **State transitions**:
  - created -> active when provisioned for normal operation
  - active -> inactive when suspended or archived
  - inactive -> active when reactivated by authorized platform-scope action

### User

- **Purpose**: Authenticated platform identity for system administrators and school-scoped actors.
- **Core attributes**:
  - id
  - school_id
  - full_name
  - email
  - status
  - roles
- **Relationships**:
  - belongs to a `School` when school-scoped
  - has many `Role` assignments
  - is included in `AuthenticationSession`
  - is actor for `AuditEvent` when authenticated
- **Validation rules**:
  - inactive users cannot authenticate or continue protected workflows
  - school-scoped users must remain constrained to their permitted school context
  - direct per-user permission assignment is outside v1 scope
- **State transitions**:
  - active -> inactive disables protected access while preserving history

### Role

- **Purpose**: Scoped authorization profile.
- **Core attributes**:
  - id
  - school_id
  - scope
  - name
  - status
  - permissions
- **Relationships**:
  - has many `Permission` definitions
  - may be assigned to `User`
- **Validation rules**:
  - platform roles may grant platform capabilities only
  - school roles may grant school capabilities only and require tenant context
  - inactive roles must not grant new effective access

### Permission

- **Purpose**: Shared capability definition exposed through roles.
- **Core attributes**:
  - id
  - code
  - name
  - scope
  - status
- **Relationships**:
  - belongs to many `Role` records
  - is exposed in `AuthenticationSession` only as defined by OpenAPI
- **Validation rules**:
  - scope must be platform or school
  - inactive permissions must not grant new effective access

### AuditEvent

- **Purpose**: Security and administrative event record for first-slice authentication and school lifecycle review.
- **Core attributes**:
  - id
  - event_type
  - actor_user_id
  - school_id
  - affected_resource_type
  - affected_resource_id
  - outcome
  - source_ip
  - tenant_safe_metadata
  - occurred_at
- **Relationships**:
  - may belong to `User` as actor when known
  - may reference `School` for school lifecycle and school-scoped auth events
  - may reference affected resource by type and id
- **Validation rules**:
  - must record login success, login failure, logout, token rejection, school create, school update, activation, and deactivation
  - must not store plaintext passwords, bearer token values, or sensitive credential material
  - tenant-safe metadata must not leak cross-tenant resource existence
- **State transitions**:
  - append-only once recorded unless a future audit retention rule explicitly defines redaction or retention behavior

### ApiResponseEnvelope

- **Purpose**: Contract-defined response shape for all product API responses.
- **Core attributes**:
  - data
  - meta
  - error
  - error.code
  - error.message
  - error.details
- **Relationships**:
  - wraps `AuthenticationSession`, `School`, and collection responses
  - represents validation, unauthorized, forbidden, tenant mismatch, lockout, token rejection, and not-found outcomes after OpenAPI is updated
- **Validation rules**:
  - success responses include `data` and `meta`
  - paginated responses include pagination metadata as defined in OpenAPI
  - failure responses include the OpenAPI error object
  - backend resources and exception handling must not produce undocumented response shapes
