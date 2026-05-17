# Feature Specification: Backend School Administration Foundation

**Feature Branch**: `003-backend-school-admin`  
**Created**: 2026-05-17  
**Status**: Draft  
**Input**: User description: "Define the next SchoolMaster backend implementation slice after the implemented backend API foundation. The backend must now implement the remaining P1 school administration foundation from the approved platform specification and OpenAPI contract: tenant-scoped user management, scoped roles, permission listing, academic years, academic periods, and guardian records. Preserve contract-first delivery, tenant-by-column isolation with School as the tenant root and school_id for school-owned records, explicit platform versus school authorization, API-only /api/v1 behavior, OpenAPI-aligned response envelopes, validation, inactive-status handling, and regression coverage. Do not include P2 teacher workflows, student self-service, or reports in this slice."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Administer tenant users and roles (Priority: P1)

A school administrator manages the initial school staff and student access foundation by listing users, creating users, listing available permissions, listing roles, and creating school-scoped roles inside the active school tenant.

**Why this priority**: Users, roles, and permissions are prerequisites for every later school workflow. Teacher, student, academic, guardian, and reporting capabilities cannot be delivered safely until tenant-scoped identities and authorization profiles are available.

**Independent Test**: This story can be tested independently by authenticating as a school administrator, resolving one active school tenant, creating a school-scoped role with school permissions, creating a user assigned to that role, and confirming listed users, roles, and permissions remain limited to the permitted tenant and scope.

**Acceptance Scenarios**:

1. **Given** an authenticated school administrator has an active resolved school context, **When** they list users and roles, **Then** only records visible in that school context are returned with the approved response envelope.
2. **Given** a school administrator creates a user, **When** the submitted role assignments are valid for the same school scope, **Then** the user is created in that school tenant and receives only the submitted roles.
3. **Given** a school administrator creates a school-scoped role, **When** the submitted permissions are active school-scope permissions, **Then** the role is created for the resolved school and does not include platform-only permissions.
4. **Given** a system administrator has platform access, **When** they attempt a school-scoped user or role action without an explicit permitted school context, **Then** access is rejected rather than treated as an implicit cross-tenant bypass.

---

### User Story 2 - Define school academic structure (Priority: P2)

A school administrator creates academic years and academic periods inside the school tenant so grades, attendance, learning sets, and reports can later reference a valid academic calendar.

**Why this priority**: Academic structure is required before teacher recording, student progress, and reporting workflows can be tied to the correct school period.

**Independent Test**: This story can be tested independently by creating one academic year for an active school, creating ordered periods inside that year, listing both collections, and verifying date, sequence, tenant, and inactive-context rules.

**Acceptance Scenarios**:

1. **Given** a school administrator has an active resolved school context, **When** they create an academic year with a valid date range, **Then** the year is stored for that school only and appears in that school's academic year list.
2. **Given** an academic year exists in the resolved school, **When** the school administrator creates an academic period inside it, **Then** the period belongs to the same school, fits within the academic year range, and has a unique sequence within that year.
3. **Given** an academic year belongs to another school or an inactive school context, **When** a requester tries to list or create related periods, **Then** the request is rejected before cross-tenant or inactive data is exposed.

---

### User Story 3 - Maintain guardian contact foundation (Priority: P3)

A school administrator creates and lists guardian records within the school tenant so student support contacts can be associated with student profiles in later workflows.

**Why this priority**: Guardian records support student administration but depend on users and academic setup being available first.

**Independent Test**: This story can be tested independently by creating a guardian with required relationship information and contact details, optionally linking existing student profiles from the same school when available, and confirming no cross-tenant student association is accepted.

**Acceptance Scenarios**:

1. **Given** a school administrator has an active resolved school context, **When** they create a guardian record with required relationship information, **Then** the guardian is created for that school and appears only in that school's guardian list.
2. **Given** guardian creation includes student profile references, **When** all referenced student profiles belong to the same active school tenant, **Then** the associations are accepted.
3. **Given** guardian creation includes a missing, inactive, or cross-tenant student profile reference, **When** the request is submitted, **Then** the request fails without creating partial cross-tenant associations.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before user, role, academic, or guardian data access?
- What happens when a role creation request mixes school-scope and platform-scope permissions?
- What happens when a user creation request assigns roles from another school, inactive roles, or platform-only roles to a school-scoped user?
- How does the backend handle duplicate user email addresses when the contract does not permit ambiguous identity resolution?
- What happens when academic year or period date ranges are invalid, overlap rules are violated, or a period falls outside its parent year?
- What happens when guardian creation references student profiles that are inactive, missing, or outside the resolved school?
- How does the backend avoid implementing update, detail, deactivate, or delete operations that are required by product intent but not yet documented in the current OpenAPI contract?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement the next API-only backend slice for tenant-scoped users, roles, permission listing, academic years, academic periods, and guardians using the approved `/api/v1` contract operations, shared response envelopes, request validation, authorization checks, tenant-context enforcement, and regression tests.
- **Frontend repository impact**: No frontend implementation is included in this feature. Frontend work remains dependent on the published OpenAPI operations and must not consume undocumented backend behavior.
- **Specification or contract repository impact**: This feature records the backend implementation boundary for the remaining P1 school administration operations already published in the aggregate and platform OpenAPI contracts. Any missing update, detail, activation, deactivation, deletion, filtering, sorting, or response behavior must be added to OpenAPI before backend implementation exposes it.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines the backend slice and contract boundary first, `schoolmaster-backend` implements only the approved operations next, and `schoolmaster-frontend` may consume those operations only after backend behavior remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: No for the currently published list/create operations in this slice. Yes before implementing any user, role, academic year, academic period, or guardian update, detail, deactivate, delete, bulk import, password-reset, invitation, student-profile creation, or assignment workflow not already documented in OpenAPI.
- **Versioned endpoints affected**: `/api/v1/users`, `/api/v1/roles`, `/api/v1/permissions`, `/api/v1/academic-years`, `/api/v1/academic-periods`, and `/api/v1/guardians`.
- **JSON response impact**: Responses must use the documented success, paginated, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, and not-found envelopes. No backend-local envelope, ad hoc error code, or undocumented response field is approved.
- **Authentication/authorization impact**: All operations require authenticated access. School-scoped operations require an active permitted school context. Role and permission behavior must preserve platform-scope and school-scope separation, and platform administration must not bypass school-scoped authorization implicitly.
- **Compatibility impact**: The slice is additive against the current aggregate contract. Any operation beyond the published OpenAPI surface requires a contract revision before implementation.

### Data & Tenancy Impact

- **Tenant scoping impact**: Users with school roles, school-scoped roles, academic years, academic periods, guardians, and guardian-student associations are school-owned records governed by the v1 `school_id` tenant boundary.
- **Cross-tenant or platform access impact**: Platform-scope permissions may list or manage platform-scoped authorization definitions only where documented. School-scoped administration requires explicit resolved school context and must not create cross-tenant records or associations.
- **Soft delete impact**: Recoverable administrative and academic records should preserve status and history expectations from the platform specification. Public delete or restore behavior is not part of this slice unless OpenAPI is updated first.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend implementation slice MUST be limited to the published operations `listUsers`, `createUser`, `listRoles`, `createRole`, `listPermissions`, `listAcademicYears`, `createAcademicYear`, `listAcademicPeriods`, `createAcademicPeriod`, `listGuardians`, and `createGuardian` unless OpenAPI is updated first.
- **FR-002**: Every school-scoped operation in this slice MUST resolve an active permitted school context before validation that depends on school-owned records, authorization decisions, service logic, persistence, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any school-owned user, role, academic, guardian, or student-profile data is accessed.
- **FR-004**: User creation MUST create school-scoped users only when submitted school context, role assignments, user status, and identity fields satisfy the published contract and same-tenant authorization rules.
- **FR-005**: User creation MUST reject role assignments that are missing, inactive, platform-only where school-scope is required, or owned by another school.
- **FR-006**: User listing MUST return only users visible to the requester in the permitted platform or school scope and MUST follow documented pagination, filtering, sorting, and response envelopes.
- **FR-007**: Role creation MUST preserve scope integrity: platform roles cannot be created through school-scoped flows, school roles require the resolved school context, and roles cannot include permissions from an incompatible scope.
- **FR-008**: Role listing MUST expose roles and inherited permissions only in the requester's permitted scope.
- **FR-009**: Permission listing MUST expose active permission definitions available to the requester without allowing direct per-user permission assignment.
- **FR-010**: Academic year creation MUST require a valid school-owned date range and MUST reject records that violate documented status, date, or tenant constraints.
- **FR-011**: Academic period creation MUST require an academic year from the same resolved school, a valid period date range within that academic year, and a sequence that is unique within the academic year.
- **FR-012**: Academic year and period listing MUST return only records for the resolved school context and MUST follow documented pagination, status filtering, and response envelopes.
- **FR-013**: Guardian creation MUST create guardian records only within the resolved school context and MUST validate required relationship information and contact fields according to OpenAPI.
- **FR-014**: Guardian creation with student profile references MUST accept only active student profiles from the same resolved school and MUST reject invalid references without creating partial cross-tenant associations.
- **FR-015**: Backend validation MUST reject undocumented request fields, unsupported filters, unsupported sort values, and payload shapes not present in OpenAPI.
- **FR-016**: Backend authorization MUST keep platform-scope school provisioning and school-scoped school administration separate; system administrator access MUST NOT imply school-scope access unless a future specification and contract revision explicitly grants it.
- **FR-017**: Backend responses MUST match the published OpenAPI success and error envelopes for successful reads and writes, validation failures, unauthorized requests, forbidden requests, tenant mismatches, inactive records, and not-found outcomes.
- **FR-018**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, inactive school or user handling, incompatible role and permission scope, academic date and sequence failures, guardian association failures, and response shape for every operation in this slice.
- **FR-019**: Backend implementation MUST NOT expose update, detail, deactivate, delete, restore, bulk import, invitation, password management, student-profile creation, teacher workflow, student self-service, or reporting behavior in this slice unless `/specs` and OpenAPI are updated first.

### Key Entities *(include if feature involves data)*

- **User**: Authenticated platform identity that may be school-scoped through `school_id`, status, and role assignments.
- **Role**: Authorization grouping with platform or school scope, optional `school_id` for school-scoped roles, status, and assigned permissions.
- **Permission**: Shared capability definition exposed through role assignment and filtered by platform or school scope.
- **AcademicYear**: School-owned academic cycle with name, date range, status, and child academic periods.
- **AcademicPeriod**: School-owned subdivision of an academic year with sequence, date range, status, and later academic workflow references.
- **Guardian**: School-owned responsible contact record that may be associated with student profiles in the same school.
- **StudentProfile**: Existing or future school-owned student profile reference used only for guardian association validation in this slice, not for creating student self-service behavior.
- **Tenant Context**: The active school context required before school-owned administration workflows can access, create, or list records.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented routes in this slice to the listed OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for users, roles, academic years, academic periods, and guardians.
- **SC-003**: Authorization tests reject 100% of incompatible platform/school role and permission combinations defined in this specification.
- **SC-004**: Academic structure tests reject 100% of invalid date ranges, periods outside their parent academic year, and duplicate period sequences within the same academic year.
- **SC-005**: Guardian association tests reject 100% of missing, inactive, and cross-tenant student profile references without creating partial guardian associations.
- **SC-006**: Response-shape verification confirms every operation in this slice returns the documented success or error envelope for successful, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, and not-found cases.
- **SC-007**: A school administrator can create a role, create a user assigned to that role, create an academic year with periods, and create a guardian in one active school tenant without exposing records from any other school.

## Assumptions

- `002-backend-api-foundation` has already established backend readiness, authentication, current-session behavior, school tenant management, response-envelope usage, and baseline tenant middleware behavior in the backend repository.
- The next backend slice should finish the remaining P1 school administration foundation before P2 teacher workflows, student self-service, or reporting are implemented.
- Current OpenAPI contracts already publish list/create operations for users, roles, academic years, academic periods, and guardians, plus permission listing; implementation is restricted to those operations.
- Product-level maintain, activate, deactivate, detail, delete, invitation, and password-management expectations remain real product needs but are not approved for backend exposure until OpenAPI documents them.
- Direct per-user permission assignment is outside v1 scope; permissions are granted through roles.
- Student profile creation is outside this slice except where existing student profile identifiers may be validated for guardian association.
