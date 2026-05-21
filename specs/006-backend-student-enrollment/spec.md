# Feature Specification: Backend Student Profile and Enrollment Management

**Feature Branch**: `006-backend-student-enrollment`  
**Created**: 2026-05-21  
**Status**: Draft  
**Input**: User description: "Define the next SchoolMaster backend implementation slice after 005-backend-student-reporting. The backend must implement student profile and enrollment management for school administrators: create, list, view, update status, transfer within allowed school lifecycle rules, and maintain enrollment history for StudentProfile records. Preserve contract-first delivery, tenant-by-column isolation with School as tenant root and school_id for school-owned records, explicit platform versus school authorization, API-only /api/v1 behavior, OpenAPI-aligned response envelopes, validation, inactive-status handling, guardian association compatibility, academic-period consistency, audit/history preservation, and regression coverage. Do not include frontend implementation, classroom/course/section/roster workflows, guardian self-service, student academic record correction workflows, report changes, bulk import, billing, messaging, or undocumented APIs in this slice."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Register Student Profiles (Priority: P1)

A school administrator creates and views student profiles inside one active school tenant so students can be associated with guardians, teacher assignments, academic records, and student self-service access without relying on undocumented profile data.

**Why this priority**: Student profiles are a prerequisite for guardian association, teacher learning-set assignment, grade and attendance records, and student self-view. The backend needs an explicit, contract-governed way to create and inspect these records.

**Independent Test**: Authenticate as a school administrator with an active school context, create a student profile with required identity and enrollment fields, retrieve it by identifier, and verify the profile is visible only inside the resolved school.

**Acceptance Scenarios**:

1. **Given** a school administrator has an active resolved school context, **When** they create a student profile with valid required fields, **Then** the student profile is created in that school and can be retrieved through the documented response envelope.
2. **Given** student profiles exist in multiple schools, **When** the administrator lists or views profiles, **Then** only profiles from the resolved school are returned.
3. **Given** profile creation includes guardian references, **When** every guardian belongs to the same active school tenant, **Then** the associations are accepted and the profile remains available for guardian-compatible workflows.
4. **Given** profile creation references a missing, inactive, or cross-tenant guardian, **When** the request is submitted, **Then** the request fails without creating a partial student profile or partial association.

---

### User Story 2 - Maintain Enrollment Status and History (Priority: P2)

A school administrator updates a student's operational status and enrollment lifecycle while preserving history needed for audit, academic records, reports, and student self-service access rules.

**Why this priority**: Existing reporting and student self-view rules depend on active, inactive, and transferred student states. Those states must be managed explicitly before later correction or roster workflows can be safely introduced.

**Independent Test**: Authenticate as a school administrator, change a same-school student profile from active to inactive according to documented lifecycle rules, and verify historical academic records remain retained while active self-service access is removed where applicable.

**Acceptance Scenarios**:

1. **Given** an active student profile in the resolved school, **When** an administrator marks the profile inactive with a documented effective date and reason, **Then** the profile no longer participates in new operational workflows but historical records remain available to authorized school administrators.
2. **Given** an inactive or transferred student profile, **When** a student user attempts active self-service access for that school, **Then** access is rejected according to the documented student self-view rules.
3. **Given** a profile status change is submitted, **When** the status transition is unsupported, missing required lifecycle information, or references another school, **Then** the request is rejected before changing profile state.
4. **Given** a student profile has existing guardian associations, grades, attendance, learning-set assignments, or report references, **When** status changes, **Then** those historical links remain intact for authorized history and audit views.

---

### User Story 3 - Transfer Students Between School Contexts (Priority: P3)

An authorized administrator records a student transfer out of one school and, when permitted, creates or links a destination-school profile without exposing cross-tenant records or moving historical academic data across tenants.

**Why this priority**: Transfers are less frequent than creation and status management, but they are necessary to avoid ad hoc inactive-profile handling and to preserve tenant-safe history.

**Independent Test**: Transfer a same-school student profile using documented transfer inputs, verify the source profile becomes transferred with retained history, and verify no source-school academic records or private associations are exposed to the destination school.

**Acceptance Scenarios**:

1. **Given** a school administrator manages an active student profile, **When** they record a transfer out with required effective date and destination information, **Then** the source profile is marked transferred and remains retained for source-school history.
2. **Given** a transfer requires a destination-school profile, **When** the requester has explicit permission for the destination school context, **Then** the destination profile is created or linked only through documented contract behavior.
3. **Given** a transfer attempts to expose source-school grades, attendance, learning sets, private content, guardian links, or report outputs to another school, **When** the transfer is processed, **Then** those source-school records remain isolated and are not copied unless a future specification explicitly approves that behavior.
4. **Given** the requester lacks destination-school permission or provides an inactive, missing, or unauthorized school context, **When** they attempt a transfer, **Then** the transfer is rejected before cross-tenant state changes occur.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before student profile or enrollment data access?
- What happens when student profile creation would duplicate an existing same-school identifier or conflict with an existing active profile?
- What happens when guardian references are inactive, missing, duplicated, or outside the resolved school?
- How does the backend prevent profile updates from changing immutable tenant ownership or historical academic-record ownership?
- What happens when an inactive or transferred profile still has historical grades, attendance, learning-set assignments, report references, or guardian associations?
- How does the backend prevent a platform administrator from bypassing school-scoped student enrollment permissions without an explicit permitted school context?
- What happens when transfer inputs reference a destination school where the requester has no permission?
- How does the backend avoid introducing classroom, course, section, roster, guardian self-service, correction, bulk import, report, frontend, or undocumented API behavior in this slice?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement the backend-only student profile and enrollment management slice for school-scoped student profile creation, listing, detail retrieval, lifecycle status changes, transfer recording, guardian-compatible associations, enrollment history, request validation, policy authorization, service-layer rules, resources, tenant context enforcement, and regression tests.
- **Frontend repository impact**: No frontend implementation is included in this feature. Future student administration screens must consume only the OpenAPI operations approved by this slice.
- **Specification or contract repository impact**: OpenAPI must be expanded before backend exposure because current completed backend slices only validate existing `StudentProfile` records and do not publish student profile creation, detail, status, or transfer operations.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines the student profile and enrollment boundary first, `schoolmaster-backend` implements only the approved operations next, and `schoolmaster-frontend` consumes the behavior only after backend verification remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: Yes. The active feature contract and aggregate contract must define student profile and enrollment management operations before backend implementation begins.
- **Versioned endpoints affected**: Proposed contract surface is limited to `/api/v1/student-profiles`, `/api/v1/student-profiles/{studentProfileId}`, `/api/v1/student-profiles/{studentProfileId}/status`, and `/api/v1/student-profiles/{studentProfileId}/transfer` unless OpenAPI approves a different path set.
- **JSON response impact**: Student profile and enrollment responses must use documented success, paginated, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, and not-found envelopes. No backend-local envelope, ad hoc error code, undocumented status code, undocumented field, undocumented filter, or undocumented sort behavior is approved.
- **Authentication/authorization impact**: All operations require authenticated access, an active permitted school context, and school-scoped student administration permission. Platform administration does not imply student profile or enrollment access without an explicit permitted school context.
- **Compatibility impact**: This is additive against the current backend foundation. Any behavior that changes existing guardian, teacher workflow, student self-view, or reporting semantics requires explicit contract revision and compatibility review.

### Data & Tenancy Impact

- **Tenant scoping impact**: `StudentProfile`, enrollment history, lifecycle status, guardian associations, and transfer metadata are school-owned records governed by `school_id`.
- **Cross-tenant or platform access impact**: Source-school academic records, learning-set assignments, private content access, guardian associations, and reports remain isolated to their original school. Cross-school transfer behavior must be explicit, permission-checked, and limited to documented profile or transfer metadata.
- **Soft delete impact**: Student profiles and enrollment history must preserve audit and academic history. Public permanent deletion, restore, merge, anonymization, or purge behavior is not part of this slice unless OpenAPI and specifications are updated first.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend implementation slice MUST be limited to documented student profile list, create, detail, status update, and transfer operations unless OpenAPI is updated first.
- **FR-002**: Every operation in this slice MUST resolve an active permitted school context before validation that depends on school-owned records, authorization decisions, persistence, guardian references, lifecycle transitions, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any student profile, guardian, enrollment history, academic record, assignment, report, or cross-school transfer data is accessed.
- **FR-004**: Student profile creation MUST create records only inside the resolved school context and MUST validate required identity, contact, enrollment, status, and association fields according to OpenAPI.
- **FR-005**: Student profile creation MUST reject duplicate same-school profile identifiers, malformed required fields, unsupported status values, undocumented request fields, and cross-tenant references.
- **FR-006**: Student profile creation that includes guardian associations MUST validate every referenced guardian as active and same-school before creating the profile; any invalid guardian reference MUST reject the whole request without partial writes.
- **FR-007**: Student profile listing MUST return only profiles visible in the resolved school context and MUST support only documented pagination, filtering, and sorting behavior.
- **FR-008**: Student profile detail retrieval MUST return only same-school profiles visible to the requester and MUST preserve documented response shape for active, inactive, transferred, and not-found cases.
- **FR-009**: Student profile status updates MUST allow only documented non-transfer lifecycle transitions and MUST require documented effective date, reason, and actor context where applicable.
- **FR-010**: Student profile status updates MUST preserve historical grades, attendance, learning-set assignments, report references, guardian associations, and audit history while preventing inactive or transferred profiles from participating in new active workflows where prohibited.
- **FR-011**: Student transfer recording MUST be the only approved way to mark the source-school profile as transferred, MUST follow documented transfer lifecycle rules, and MUST preserve source-school academic and audit history.
- **FR-012**: Student transfer behavior involving a destination school MUST require explicit permission for the destination school context and MUST NOT expose source-school academic records, learning sets, private content, reports, or guardian links across tenants.
- **FR-013**: Enrollment history MUST record enough information to explain current and historical student profile status, including effective dates and reasons needed for audit, reporting, and support review.
- **FR-014**: Backend authorization MUST keep platform-scope administration, school-scoped student administration, teacher workflows, student self-view, and reporting permissions separate.
- **FR-015**: Backend validation MUST reject undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, invalid lifecycle transitions, invalid effective dates, inactive references, duplicate references, and cross-tenant profile, guardian, school, or user references.
- **FR-016**: Backend responses MUST match the published OpenAPI success and error semantics declared for each approved operation. The backend MUST NOT emit undocumented status semantics unless OpenAPI first documents them on the relevant operation.
- **FR-017**: Backend implementation MUST NOT expose classroom, course, section, group, roster, teacher assignment, guardian self-service, academic-record correction, report lifecycle, report output, frontend, bulk import, billing, messaging, notification, or undocumented API behavior in this slice.
- **FR-018**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, duplicate prevention, guardian association failures, lifecycle transition failures, transfer permission failures, inactive school/user/profile handling, history preservation, and response shape for every operation in this slice.

### Key Entities *(include if feature involves data)*

- **StudentProfile**: School-owned learner profile linked to a user identity where applicable; used by guardian associations, teacher assignments, academic records, reports, and student self-view rules.
- **EnrollmentHistory**: School-owned record of student profile lifecycle events, including creation, activation, inactivation, transfer, effective dates, reasons, and actor context needed for audit and support review.
- **StudentTransfer**: School-owned source-school transfer metadata for a student profile, including source profile, optional destination school/profile references where documented, effective date, reason, actor context, and no-copy guarantees for historical academic records, private content, guardian links, reports, and report outputs.
- **GuardianAssociation**: Same-school association between a student profile and guardian record; this slice may create or update profile-side associations only where OpenAPI documents it.
- **School**: Tenant root that owns student profiles, enrollment history, guardian associations, and historical academic records.
- **User**: Authenticated actor and optional student login identity linked to a student profile; school administrator users perform management actions under school-scoped permissions.
- **AcademicPeriod**: Existing school-owned period that may constrain enrollment effective dates or profile visibility where OpenAPI documents such constraints.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented routes in this slice to approved OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for student profile creation, listing, detail, status update, and transfer operations.
- **SC-003**: Student profile creation tests reject 100% of duplicate same-school identifiers, invalid required fields, unsupported status values, cross-tenant references, and invalid guardian associations covered by this specification.
- **SC-004**: Enrollment lifecycle tests confirm 100% of unsupported status transitions and invalid transfer attempts are rejected without partial profile, association, or history changes.
- **SC-005**: History preservation tests confirm existing grades, attendance, learning-set assignments, guardian associations, report references, and audit records remain retained after inactivation or transfer.
- **SC-006**: Authorization tests confirm platform administrators, school administrators, teachers, and students receive only the student profile and enrollment permissions explicitly documented for their active scope.
- **SC-007**: Response-shape verification confirms every operation in this slice returns the documented success or error envelope for successful, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, and not-found cases.
- **SC-008**: A school administrator can create, retrieve, list, inactivate, and transfer a student profile in one active school tenant without exposing records from another school.

## Assumptions

- `005-backend-student-reporting` has completed student self-view and reporting behavior that depends on existing `StudentProfile` records but does not create or manage those records.
- Current OpenAPI contracts do not yet publish student profile creation, detail, lifecycle status, or transfer operations; this feature must expand contracts before backend implementation.
- Student profiles remain school-owned records; transfer preserves source-school history rather than moving historical academic records across tenants.
- Guardian association compatibility means validating and preserving same-school guardian links, not creating guardian self-service or guardian-facing student views.
- Classroom, course, section, group, roster, and teacher assignment models remain unspecified and outside this slice.
- Bulk import, merge, anonymization, permanent deletion, correction workflows, report changes, frontend implementation, billing, messaging, notifications, and parent portal behavior remain outside this slice.
