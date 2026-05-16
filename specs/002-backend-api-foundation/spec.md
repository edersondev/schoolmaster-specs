# Feature Specification: Backend API Foundation

**Feature Branch**: `001-backend-api-foundation`  
**Created**: 2026-05-14  
**Status**: Draft  
**Input**: User description: "Prepare the SchoolMaster Laravel backend repository based on the specifications mounted at /specs. This repository is the API-only backend for the SchoolMaster SaaS platform. Use /specs as the source of truth, especially /specs/AGENTS.md, /specs/specs, /specs/api/openapi.yaml, /specs/docs, and /specs/decisions. The goal of this backend specification is to define the initial backend setup and implementation boundaries before coding."

## Clarifications

### Session 2026-05-14

- Q: What token lifecycle should the first authentication slice specify? → A: Bearer tokens expire after 8 hours; logout and inactive user or school status revoke access.
- Q: What login rate-limiting behavior should the first authentication slice specify? → A: Rate-limit failed login attempts by email and IP, and document the lockout response.
- Q: What failed-login threshold should the first authentication slice specify? → A: 5 failed attempts per email or IP within 15 minutes.
- Q: What audit coverage should the first authentication and school management slice specify? → A: Audit login success and failure, logout, token rejection, and school create, update, and status changes.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Establish Backend Readiness (Priority: P1)

A backend maintainer prepares the repository so every future product endpoint is implemented from the published SchoolMaster specifications and API contract, not from local assumptions.

**Why this priority**: The backend cannot safely implement authentication, school management, or tenant-scoped workflows until the repository is aligned with the source-of-truth specifications and contract-first workflow.

**Independent Test**: This story can be tested independently by reviewing the repository setup and confirming that the backend guidance, source-of-truth links, environment expectations, API-only boundaries, and contract validation entry points are present and consistent with `/specs`.

**Acceptance Scenarios**:

1. **Given** a maintainer opens the backend repository, **When** they read the backend guidance, **Then** they can identify `/specs` as the source of truth and know that specs, OpenAPI, docs, and decisions must be read before backend behavior changes.
2. **Given** the backend repository consumes the `/specs` submodule, **When** the maintainer validates the mounted specifications, **Then** the active platform spec, aggregate OpenAPI contract, backend guidelines, security guidance, and architecture decisions are available before implementation work starts.
3. **Given** the backend is being prepared for product work, **When** repository readiness is checked, **Then** product-facing behavior is bounded to API-only `/api/v1` contracts and excludes frontend code or product Blade views.

---

### User Story 2 - Define API and Security Foundations (Priority: P2)

A backend maintainer defines the base API behavior expected for authentication, authorization, validation, responses, and tenant context so future controllers and services can be implemented consistently.

**Why this priority**: Authentication and response behavior are shared by every protected module. Inconsistent foundations would create contract drift and tenant-isolation risk across later features.

**Independent Test**: This story can be tested independently by checking that the backend setup specification identifies the approved authentication source, response envelope dependency, validation and authorization responsibilities, and tenant-context failure behavior without creating undocumented endpoints.

**Acceptance Scenarios**:

1. **Given** OpenAPI defines authentication and current-session operations, **When** the backend authentication foundation is planned, **Then** it uses the published contract and accepted Laravel-native authentication decision without inventing undocumented request fields, response fields, or token semantics.
2. **Given** a protected school-scoped request is planned, **When** tenant context is missing, mismatched, inactive, or unauthorized, **Then** the backend foundation requires rejection before module-specific business logic or data access.
3. **Given** a request fails validation or authorization, **When** the response is shaped, **Then** it follows the approved OpenAPI error envelope instead of a module-specific format.

---

### User Story 3 - Bound First Backend Product Slice (Priority: P3)

A backend maintainer prepares the first implementation boundary for authentication and school management so coding can begin only after the repository foundation and contract checks are satisfied.

**Why this priority**: Authentication and school tenant management are the first platform capabilities needed before users, roles, academic structures, teacher workflows, student views, and reports can be safely delivered.

**Independent Test**: This story can be tested independently by verifying that the first backend slice maps only to approved OpenAPI operations for authentication and school management, with explicit tenant, authorization, validation, response, and test expectations.

**Acceptance Scenarios**:

1. **Given** the first backend product slice is selected, **When** its scope is reviewed, **Then** it includes only authentication and school management operations already present in `/specs/api/openapi.yaml` or the active feature contract.
2. **Given** school management is planned, **When** platform and school-scoped access are reviewed, **Then** system administrator school provisioning is separated from school-scoped module permissions.
3. **Given** backend tests are planned for the first slice, **When** acceptance coverage is reviewed, **Then** it includes successful flows, validation failures, authorization failures, inactive status handling, and tenant-isolation failure cases.

### Edge Cases

- What happens when the `/specs` submodule is missing, uninitialized, or does not contain the active platform spec and aggregate OpenAPI contract?
- What happens when backend setup expectations conflict with `/specs` decisions, such as a generic `tenant_id` wording conflicting with v1 `school_id` tenant ownership?
- How does the backend avoid exposing web product views while still allowing framework health checks or operational endpoints?
- How does the backend prevent uncontracted request fields, response fields, route prefixes, filters, sorting behavior, or authorization semantics from entering implementation?
- How does the backend reject expired bearer tokens, logged-out sessions, and tokens tied to inactive users or inactive schools?
- How does the backend prevent repeated failed login attempts from one source or against one account identifier?
- How does the backend retain enough audit context for authentication and school lifecycle security review without exposing sensitive credentials or token values?
- How does the backend ensure inactive schools and inactive users cannot proceed into protected workflows?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Establish the API-only backend setup boundary, source-of-truth guidance, environment expectations, authentication foundation, response envelope dependency, tenant foundation, and first product slice boundary for authentication and school management.
- **Frontend repository impact**: No frontend implementation is included. Frontend work remains dependent on the published OpenAPI contracts and must not consume backend behavior that is not documented in `/specs`.
- **Specification or contract repository impact**: This feature adds a backend-readiness specification. It does not create new product behavior beyond the active platform specification; any missing behavior discovered during implementation planning must be updated in `/specs` and OpenAPI before backend code changes.
- **Delivery ownership and sequencing**: Specification and contract validation occur first, backend foundation setup follows, then authentication and school management implementation may begin as the first backend slice.

### API Contract Impact

- **OpenAPI update required**: Yes. The backend readiness feature clarifies product-visible authentication behavior that is not fully documented yet. OpenAPI must be updated before authentication implementation to document logout revocation, token-expiry rejection, inactive user or school token rejection, failed-login lockout, and audit-relevant response semantics.
- **Versioned endpoints affected**: The first implementation boundary is limited to `/api/v1/auth/login`, `/api/v1/auth/me`, `/api/v1/auth/logout`, `/api/v1/schools`, and `/api/v1/schools/{schoolId}` after OpenAPI documents logout revocation behavior.
- **JSON response impact**: Backend setup must adopt the success, paginated, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, and not-found envelopes documented in OpenAPI.
- **Authentication/authorization impact**: Authentication follows the accepted Laravel-native authentication decision and OpenAPI contract. Authorization must keep platform-scope school provisioning separate from school-scoped permissions.
- **Compatibility impact**: This is a greenfield backend foundation. Once implemented, externally visible changes must be additive unless a future approved contract revision defines a migration path.

### Data & Tenancy Impact

- **Tenant scoping impact**: SchoolMaster v1 uses tenant-by-column with `School` as the tenant root and `school_id` as the concrete tenant column for school-owned records.
- **Tenant resolution impact**: Every authenticated school-scoped request must resolve an active school context before validation that depends on school-owned data, authorization decisions, service logic, file access, or persistence.
- **Cross-tenant or platform access impact**: System administrators may provision and review school tenants through platform-scope authorization only. They do not receive an implicit bypass for school-scoped module actions.
- **Platform override impact**: Any cross-tenant override must be explicitly documented in `/specs`, OpenAPI, authorization policy expectations, and regression tests before implementation.
- **Soft delete impact**: Recoverable school-owned operational records must support status and soft-delete expectations where required by the active platform specification.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend repository MUST identify `/specs` as the source of truth for product specifications, business rules, OpenAPI contracts, architecture decisions, and implementation sequencing.
- **FR-002**: The backend repository MUST provide backend-specific guidance that aligns with `/specs/AGENTS.md`, including the mandatory read order and the rule that `/specs` wins when conflicts exist.
- **FR-003**: The backend setup MUST verify that the `/specs` submodule exposes the active platform specification, aggregate OpenAPI contract, backend guidelines, multi-tenant guidance, security guidance, and accepted architecture decisions before implementation begins.
- **FR-004**: The backend MUST be bounded as an API-only product application and MUST NOT introduce frontend implementation or product Blade views.
- **FR-005**: Product routes MUST be limited to RESTful `/api/v1` endpoints that are defined in `/specs/api/openapi.yaml` or an approved feature-level contract.
- **FR-006**: Backend environment expectations MUST identify MySQL as the primary transactional datastore and MUST keep secrets in environment configuration.
- **FR-007**: Backend setup MUST define a repeatable local readiness path for dependency installation, environment configuration, application key generation, database migration, test execution, and OpenAPI validation.
- **FR-008**: Backend implementation boundaries MUST require UUID identifiers for entities exchanged across product boundaries or external references, as defined by `/specs`.
- **FR-009**: Backend implementation boundaries MUST use tenant-by-column multi-tenancy with `School` as the v1 tenant root and `school_id` as the concrete column for school-owned records.
- **FR-010**: Backend implementation boundaries MUST reject school-scoped requests when tenant context is missing, mismatched, inactive, or unauthorized before module-specific business logic runs.
- **FR-011**: Backend implementation boundaries MUST use policies for authorization and MUST keep platform-scope and school-scope authorization paths explicit.
- **FR-012**: Backend implementation boundaries MUST use request validation for client input and MUST avoid accepting undocumented request fields.
- **FR-013**: Backend implementation boundaries MUST use response resources or equivalent response-shaping boundaries so externally visible JSON matches OpenAPI.
- **FR-014**: Backend implementation boundaries MUST route business behavior through service classes and keep controllers limited to request orchestration, authorization, service invocation, and response selection.
- **FR-015**: Backend setup MUST define the base API response pattern by referencing the OpenAPI success and error envelopes rather than inventing module-specific response formats.
- **FR-016**: Backend authentication foundation MUST follow the accepted Laravel-native authentication decision and the published OpenAPI authentication operations.
- **FR-017**: Authentication implementation MUST issue bearer tokens that expire after 8 hours and MUST revoke access when a user logs out.
- **FR-017a**: Authentication implementation MUST prevent inactive users and users tied to inactive schools from authenticating or continuing protected workflows.
- **FR-017b**: Authentication implementation MUST rate-limit failed login attempts by both email and IP address, with lockout after 5 failed attempts for either key within 15 minutes; the lockout response MUST be documented in OpenAPI before coding.
- **FR-017c**: Authentication implementation MUST audit login success, login failure, logout, and token rejection events without storing plaintext credentials or bearer token values.
- **FR-018**: School foundation implementation MUST represent schools as tenant roots with active and inactive status handling aligned to `/specs`.
- **FR-019**: School management implementation MUST separate system administrator platform-scope school provisioning from school-scoped school profile management.
- **FR-019a**: School management implementation MUST audit school create, update, activation, and deactivation events with actor, school, action, and timestamp context.
- **FR-020**: The first backend product slice MUST be limited to authentication and school management unless `/specs` and OpenAPI are updated first.
- **FR-021**: Backend tests MUST include feature coverage for API behavior, validation, authorization, tenant isolation, inactive status handling, and response shape for every implemented contract operation.
- **FR-022**: Backend tests MUST include unit coverage for isolated service or domain logic where behavior is not fully covered by feature tests.
- **FR-023**: Backend implementation planning MUST identify any missing contract behavior before coding and update `/specs` and OpenAPI before implementation proceeds.
- **FR-024**: Backend code MUST NOT create endpoints, payload fields, filters, sorting behavior, response fields, permission semantics, or tenant behavior that are absent from `/specs`.

### Key Entities *(include if feature involves data)*

- **Backend Readiness Boundary**: The repository-level agreement that defines source-of-truth usage, API-only scope, environment expectations, contract validation, and implementation prerequisites.
- **API Response Envelope**: The shared success, paginated, and error response structures documented in OpenAPI and consumed by backend and frontend repositories.
- **Authentication Session**: The authenticated user context returned by the published authentication contract, including identity, roles, permissions, and resolved school context where applicable.
- **School**: The tenant root for v1 school-owned records, with operational status and platform-level lifecycle behavior.
- **Tenant Context**: The active school context required by school-scoped requests, represented by the OpenAPI tenant-context behavior and enforced before tenant-owned data access.
- **User**: The authenticated platform identity for system administrators, school administrators, teachers, and students, with inactive-user access restrictions.
- **Role**: A platform-scoped or school-scoped authorization profile that grants permissions through role assignment.
- **Permission**: A capability definition assigned through roles and exposed only as defined by the contract.
- **Audit Event**: A security or administrative event recorded for authentication and school lifecycle review, including actor where available, event type, affected school where applicable, timestamp, outcome, and tenant-safe metadata.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A maintainer can determine the backend source of truth, active spec, aggregate contract, and first implementation boundary in under 10 minutes from a clean checkout.
- **SC-002**: Repository readiness review identifies zero product routes outside the approved `/api/v1` OpenAPI scope before the first backend feature is implemented.
- **SC-003**: Contract validation for the active feature contract and aggregate OpenAPI contract can be executed and recorded before backend implementation merges.
- **SC-004**: The first authentication and school management implementation review finds zero undocumented request fields, response fields, route prefixes, or tenant behaviors.
- **SC-005**: Tenant-isolation acceptance tests reject 100% of defined missing, mismatched, inactive, and unauthorized school-context attempts for implemented school-scoped operations.
- **SC-006**: New backend contributors can complete documented local setup and run the baseline backend test suite without private knowledge of the project.
- **SC-007**: Authentication acceptance tests reject 100% of expired tokens, logged-out tokens, inactive-user tokens, and inactive-school tokens in the first authentication slice.
- **SC-008**: Authentication acceptance tests reject 100% of login attempts that exceed 5 failed attempts within 15 minutes for either the submitted email or source IP address.
- **SC-009**: Security review confirms audit events are recorded for 100% of login success, login failure, logout, token rejection, and school lifecycle changes in the first backend slice.

## Assumptions

- `/specs` is mounted as the SchoolMaster specification repository and remains the source of truth even when implementation repository guidance is incomplete.
- The user-provided `tenant_id column strategy` is interpreted through ADR 004: v1 uses tenant-by-column generally, with `School` as tenant root and `school_id` as the concrete school-owned column.
- This feature prepares backend setup and implementation boundaries only; it does not implement product endpoints, migrations, services, controllers, policies, or tests.
- Token rotation and refresh behavior remain out of scope for the first authentication slice unless `/specs` and OpenAPI are updated before coding.
- The first backend product slice should start with authentication and school management because the active platform specification makes school tenant onboarding the prerequisite for later user, academic, teacher, student, and reporting workflows.
- Framework health checks or operational endpoints may exist outside product behavior, but all product API behavior must be versioned under `/api/v1` and documented in OpenAPI.
