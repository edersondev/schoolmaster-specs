# Feature Specification: Backend Administration Lifecycle Management

**Feature Branch**: `007-administration-lifecycle`  
**Created**: 2026-05-25  
**Status**: Draft  
**Input**: User description: "Define the next SchoolMaster backend implementation slice after student profile and enrollment management. The backend must add administration lifecycle management for existing school and school-administration resources: individual detail, update, activate/deactivate, delete/restore operations for schools, users, roles, academic years, academic periods, and guardians where allowed, plus selected bulk operations only for school-owned users, roles, academic years, academic periods, and guardians. Preserve contract-first delivery, tenant-by-column isolation with School as tenant root and school_id for school-owned records, explicit platform versus school authorization, API-only /api/v1 behavior, OpenAPI-aligned response envelopes, validation, inactive-status handling, soft-delete and history preservation, conflict rules, authorization boundaries, and regression coverage. Do not include invitations, password setup or reset, classroom/course/section/roster workflows, teacher correction workflows, guardian self-service, report lifecycle expansion, frontend implementation, billing, messaging, or undocumented APIs in this slice."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Maintain Administrative Records (Priority: P1)

An authorized administrator views details and updates allowed fields for existing school, user, role, academic year, academic period, and guardian records without creating undocumented lifecycle behavior or crossing tenant boundaries.

**Why this priority**: Existing backend slices provide list and create operations for most school-administration resources. Administrators need safe detail and update behavior before operational data can be maintained without direct database changes or ad hoc backend endpoints.

**Independent Test**: Authenticate as an administrator with the required platform or school scope, retrieve a single existing record, update only documented mutable fields, and verify the response reflects the update while records outside the permitted scope remain inaccessible.

**Acceptance Scenarios**:

1. **Given** a school administrator has an active resolved school context, **When** they retrieve or update a same-school user, role, academic year, academic period, or guardian using documented fields, **Then** the record is returned or updated through the approved response envelope.
2. **Given** a platform administrator updates a school record through a platform-scoped operation, **When** the submitted fields are valid and permitted, **Then** the school record is updated without granting implicit access to school-owned module records.
3. **Given** an update request includes immutable ownership fields, undocumented fields, unsupported relationships, or cross-tenant references, **When** the request is submitted, **Then** the request is rejected before persistence.
4. **Given** a requester lacks the required platform or school permission, **When** they attempt detail or update behavior, **Then** access is rejected without revealing whether a forbidden cross-scope record exists.

---

### User Story 2 - Control Activation and Recoverable Removal (Priority: P2)

An authorized administrator activates, deactivates, deletes, and restores administrative records where those transitions are allowed, while preserving history and preventing invalid dependency states.

**Why this priority**: Schools need operational maintenance for records that become inactive or were removed by mistake. Lifecycle transitions must be explicit so dependent teacher, student, reporting, and authorization workflows do not infer inconsistent state.

**Independent Test**: Authenticate with the required scope, deactivate a same-school guardian or academic period, verify new operational use is blocked where documented, restore the record, and confirm historical references remain intact.

**Acceptance Scenarios**:

1. **Given** an active record has no blocking active dependency, **When** an authorized administrator deactivates or soft-deletes it with documented reason and effective date where required, **Then** the record is no longer available for new operational use but remains retained for authorized history.
2. **Given** an inactive or soft-deleted record is eligible for restoration, **When** an authorized administrator restores it, **Then** the record becomes available according to the documented restored status and tenant scope.
3. **Given** a requested lifecycle transition would orphan active users, invalidate academic records, expose cross-tenant history, remove the current school context, or violate published dependency rules, **When** the request is submitted, **Then** the transition is rejected with the documented conflict response.
4. **Given** a platform administrator deactivates, activates, deletes, or restores a school, **When** the school status changes, **Then** school-scoped workflows for that tenant follow the documented inactive-school rules without granting platform users school-scoped bypass.

---

### User Story 3 - Apply Selected Bulk Lifecycle Actions (Priority: P3)

An authorized administrator applies selected lifecycle actions to a bounded set of same-scope records so repetitive maintenance can be performed consistently without partial tenant leakage or ambiguous results.

**Why this priority**: Bulk lifecycle actions are useful after single-record transitions are defined, but they must remain secondary because they multiply authorization, validation, conflict, and partial-failure risks.

**Independent Test**: Submit a documented bulk deactivate request for a small set of same-school eligible records, verify every requested record is processed atomically according to the contract, and verify any invalid record causes the documented failure behavior without cross-tenant changes.

**Acceptance Scenarios**:

1. **Given** all selected records are in the same permitted scope and eligible for the same documented action, **When** an authorized administrator submits a bulk lifecycle request, **Then** the action is applied consistently and the response identifies every affected record.
2. **Given** the selected records include a cross-tenant, missing, inactive-ineligible, dependency-blocked, unsupported resource, or duplicate identifier, **When** the bulk request is submitted, **Then** the request fails according to the documented all-or-nothing semantics.
3. **Given** a bulk request attempts mixed resource types, mixed tenant scopes, unsupported fields, undocumented actions, or more records than the contract permits, **When** the request is submitted, **Then** no lifecycle state changes are persisted.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before school-owned detail, update, lifecycle, or bulk data access?
- What happens when a platform administrator tries to update school-owned records without an explicit permitted school context?
- What happens when an update attempts to change immutable identifiers, `school_id`, platform/school role scope, academic parent ownership, or historical ownership?
- How does the backend prevent deactivating, deleting, or restoring records that have active dependencies or would invalidate existing academic, authorization, guardian, enrollment, report, or audit history?
- What happens when a lifecycle action targets an already inactive, active, deleted, restored, missing, or cross-tenant record?
- How are school activation, deactivation, deletion, and restoration constrained so inactive-school rejection remains consistent across protected workflows?
- How does a bulk request handle duplicate identifiers, unsupported resource types, mixed tenant scopes, and records that fail dependency checks?
- How does the backend avoid implementing invitations, password setup/reset, account recovery, classroom/course/section/roster workflows, teacher corrections, guardian self-service, report lifecycle expansion, frontend behavior, billing, messaging, or undocumented API behavior in this slice?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement an API-only lifecycle slice for individual detail, update, activation, deactivation, soft delete, and restore actions for users, roles, academic years, academic periods, guardians, and schools where OpenAPI approves the operation. Selected bulk lifecycle actions are limited to school-owned users, roles, academic years, academic periods, and guardians. Backend work must include request validation, service-layer transition rules, authorization policies, resources, tenant context enforcement, conflict handling, audit-relevant history preservation, and regression tests.
- **Frontend repository impact**: No frontend implementation is included in this feature. Future admin maintenance screens must consume only the OpenAPI operations approved by this slice.
- **Specification or contract repository impact**: OpenAPI must be expanded before backend exposure because current completed backend slices do not publish lifecycle operations for most affected resources.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines lifecycle boundaries first, `schoolmaster-backend` implements only approved operations next, and `schoolmaster-frontend` consumes the behavior only after backend verification remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: Yes. The active feature contract and aggregate contract must define operation IDs, paths, request schemas, response schemas, status semantics, conflict responses, and bulk all-or-nothing behavior before backend implementation begins.
- **Versioned endpoints affected**: Proposed contract surface is limited to individual detail, update, activate, deactivate, delete, and restore operations under `/api/v1/schools`, `/api/v1/users`, `/api/v1/roles`, `/api/v1/academic-years`, `/api/v1/academic-periods`, and `/api/v1/guardians`; selected bulk lifecycle operations are limited to `/api/v1/users`, `/api/v1/roles`, `/api/v1/academic-years`, `/api/v1/academic-periods`, and `/api/v1/guardians` unless OpenAPI approves a different path set.
- **JSON response impact**: Responses must use documented success, paginated where applicable, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, not-found, and bulk-result envelopes. No backend-local envelope, ad hoc error code, undocumented status code, undocumented field, undocumented filter, or undocumented sort behavior is approved.
- **Authentication/authorization impact**: All operations require authenticated access. School-owned resource operations require an active permitted school context and school-scoped administration permission. School lifecycle operations require platform-scoped school administration permission. Platform administration does not imply access to school-owned lifecycle actions.
- **Compatibility impact**: This is additive against the current backend foundation. Any behavior that changes existing create/list, authentication, tenant resolution, student enrollment, teacher workflow, student reporting, or guardian association semantics requires explicit contract revision and compatibility review.

### Data & Tenancy Impact

- **Tenant scoping impact**: Users with school roles, school-scoped roles, academic years, academic periods, guardians, and guardian-student associations are school-owned records governed by `school_id`. Schools are tenant roots managed through platform-scoped operations.
- **Cross-tenant or platform access impact**: Platform users may manage school records only through documented platform operations. They do not receive implicit access to school-owned users, roles, academic structures, guardians, students, teacher records, reports, or private content.
- **Soft delete impact**: Delete behavior in this slice means recoverable soft deletion unless a future specification explicitly approves permanent deletion or purge. Soft-deleted records must remain retained for authorized history, audit, dependency checks, and restore decisions.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend implementation slice MUST be limited to documented individual detail, update, activate, deactivate, delete, and restore operations for users, roles, academic years, academic periods, guardians, and schools, plus documented selected bulk lifecycle operations for users, roles, academic years, academic periods, and guardians only, unless OpenAPI is updated first.
- **FR-002**: Every school-owned operation in this slice MUST resolve an active permitted school context before validation that depends on school-owned records, authorization decisions, lifecycle rules, persistence, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any school-owned user, role, academic, guardian, student, teacher, report, or association data is accessed.
- **FR-004**: School lifecycle operations MUST be platform-scoped and MUST NOT grant platform users implicit permission to perform school-owned lifecycle actions inside the affected school.
- **FR-005**: Detail operations MUST return only records visible to the requester in the permitted platform or school scope and MUST preserve documented response shape for active, inactive, soft-deleted where allowed, forbidden, and not-found cases.
- **FR-006**: Update operations MUST allow only documented mutable fields and MUST reject changes to immutable identifiers, tenant ownership, platform/school role scope, academic parent ownership, historical ownership, undocumented request fields, unsupported relationships, and cross-tenant references.
- **FR-007**: User lifecycle operations MUST preserve authentication and authorization separation and MUST NOT introduce invitation, password setup, password reset, account recovery, token refresh, or direct per-user permission assignment behavior.
- **FR-008**: Role lifecycle operations MUST preserve platform-scope and school-scope separation and MUST reject deactivation, deletion, or restoration that would leave active users with invalid required authorization state.
- **FR-009**: Academic year lifecycle operations MUST reject updates or transitions that would invalidate child academic periods, existing grades, attendance, learning sets, enrollment history, reports, or documented academic date rules.
- **FR-010**: Academic period lifecycle operations MUST reject updates or transitions that would move the period outside its parent academic year, duplicate sequence constraints, or invalidate existing grades, attendance, learning sets, enrollment history, or reports.
- **FR-011**: Guardian lifecycle operations MUST preserve same-school student profile associations and MUST reject updates or transitions that create missing, inactive, duplicated, or cross-tenant associations.
- **FR-012**: School lifecycle operations MUST preserve inactive-school rejection behavior for protected workflows and MUST require documented reason, effective date, and actor context where the operation affects tenant availability.
- **FR-013**: Delete operations in this slice MUST be recoverable soft deletes and MUST preserve history needed for audit, authorization review, academic records, guardian associations, student enrollment, reports, and support review.
- **FR-014**: Restore operations MUST validate the current record state, tenant or platform scope, dependency eligibility, uniqueness constraints, and active parent context before making a record available for new operational use.
- **FR-015**: Selected bulk lifecycle operations MUST be limited to one documented resource type, one documented lifecycle action, one permitted scope, and a contract-defined maximum number of unique record identifiers per request.
- **FR-016**: Selected bulk lifecycle operations MUST use documented all-or-nothing semantics: if any selected record is missing, unauthorized, cross-tenant, dependency-blocked, duplicate, invalid for the requested action, or otherwise ineligible, no selected record is changed.
- **FR-017**: Backend validation MUST reject undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, invalid lifecycle transitions, invalid effective dates, inactive references, duplicate identifiers, mixed-scope bulk requests, and cross-tenant references.
- **FR-018**: Backend responses MUST match the published OpenAPI success and error semantics declared for each approved operation. The backend MUST NOT emit undocumented status semantics unless OpenAPI first documents them on the relevant operation.
- **FR-019**: Backend implementation MUST NOT expose invitations, password setup/reset, account recovery, classroom, course, section, roster, teacher correction, guardian self-service, report lifecycle expansion, report output, frontend, billing, messaging, notification, permanent purge, or undocumented API behavior in this slice.
- **FR-020**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, immutable-field failures, dependency conflicts, lifecycle transition failures, bulk all-or-nothing failures, inactive school/user/resource handling, soft-delete and restore behavior, history preservation, and response shape for every operation in this slice.

### Key Entities *(include if feature involves data)*

- **School**: Platform-managed tenant root whose lifecycle determines whether school-scoped protected workflows may proceed.
- **User**: Authenticated identity that may be school-scoped through role assignments and status; lifecycle maintenance excludes invitation and password recovery workflows.
- **Role**: Authorization grouping with platform or school scope, status, assigned permissions, and assignment dependencies that constrain lifecycle transitions.
- **AcademicYear**: School-owned academic cycle with date range, status, child periods, and historical references from academic and reporting workflows.
- **AcademicPeriod**: School-owned subdivision of an academic year with date range, sequence, status, and references from teacher, enrollment, student, and report workflows.
- **Guardian**: School-owned responsible contact record associated with same-school student profiles where documented.
- **LifecycleHistory**: Audit-relevant record of administrative transitions, reasons, effective dates, actor context, and prior state needed for support review and restore decisions.
- **BulkLifecycleRequest**: Documented request to apply one lifecycle action to a bounded set of same-scope records with all-or-nothing semantics.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented lifecycle routes in this slice to approved OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for school-owned detail, update, lifecycle, and bulk operations.
- **SC-003**: Authorization tests confirm 100% of platform-scoped school lifecycle actions and school-scoped resource lifecycle actions remain separated by documented permissions.
- **SC-004**: Update validation tests reject 100% of immutable ownership changes, undocumented fields, invalid relationships, and cross-tenant references covered by this specification.
- **SC-005**: Lifecycle transition tests reject 100% of unsupported status changes, dependency-blocked deletes, dependency-blocked restores, and invalid effective dates covered by this specification.
- **SC-006**: Bulk lifecycle tests confirm 100% of invalid mixed-scope, mixed-resource, duplicate-identifier, over-limit, unauthorized, or dependency-blocked requests leave all selected records unchanged.
- **SC-007**: History preservation tests confirm soft-deleted or deactivated records retain authorized access to prior audit, academic, guardian, student enrollment, authorization, and reporting references where applicable.
- **SC-008**: Response-shape verification confirms every operation in this slice returns the documented success or error envelope for successful, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, bulk-result, and not-found cases.
- **SC-009**: An authorized administrator can view, update, deactivate, restore, and soft-delete eligible records in one active scope without exposing records from another school or platform scope.

## Assumptions

- `006-backend-student-enrollment` has specified student profile and enrollment management, including student lifecycle history that later admin operations must not invalidate.
- Current OpenAPI contracts do not yet publish most individual detail, update, activate, deactivate, delete, or restore lifecycle operations for users, roles, academic years, academic periods, guardians, and schools, or selected bulk lifecycle operations for users, roles, academic years, academic periods, and guardians; this feature must expand contracts before backend implementation.
- Delete means recoverable soft delete for this slice. Permanent purge, anonymization, and legal retention workflows remain outside scope until separately specified.
- Bulk operations are intentionally narrow: one resource type, one action, one permitted scope, bounded record count, and all-or-nothing behavior.
- Selected bulk lifecycle operations apply only to school-owned users, roles, academic years, academic periods, and guardians in this slice. Bulk school lifecycle operations are out of scope because schools are platform-scoped tenant roots.
- Account lifecycle workflows such as invitations, password setup, password reset, account recovery, lock recovery, and token refresh remain a separate roadmap item.
- Classroom, course, section, roster, teacher correction, guardian self-service, report lifecycle expansion, frontend implementation, billing, messaging, notifications, and parent portal behavior remain outside this slice.
