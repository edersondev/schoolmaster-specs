# Feature Specification: Backend Guardian Self-Service

**Feature Branch**: `011-guardian-self-service`  
**Created**: 2026-06-04  
**Status**: Ready for Implementation  
**Input**: User description: "Guardian Self-Service: Allow guardians to view permitted student profile, academic, and contact information without gaining school-admin or student self-view powers. Define guardian authentication, student association proof, data visibility limits, consent or school approval rules, and cross-tenant behavior. Item 5 is already implemented."

## Clarifications

### Session 2026-06-04

- Q: How is a guardian record linked to an authenticated account for self-service access? → A: School administrators explicitly link an existing guardian record to an authenticated user account before self-service access is allowed.
- Q: What academic detail level should guardians see in v1? → A: Academic summaries only: current grade summary, attendance totals/status, and learning-set progress/status where documented.
- Q: How should guardian academic summaries be scoped by academic period? → A: Guardian academic summary requests must include an explicit same-school academic period.
- Q: Which contact fields should guardians see in v1? → A: The authenticated guardian's own contact fields, relationship labels, and the student's primary school-approved contact details.
- Q: What response should target-specific denied guardian student requests return? → A: Missing, unassociated, inactive, or cross-tenant target students return the same not-found envelope.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - View Linked Student Overview (Priority: P1)

An authenticated guardian views only the students that the school has approved for that guardian in the active school context, including limited profile and enrollment summary information.

**Why this priority**: Guardian access is only valuable if the system can prove the guardian-student relationship and keep the view separate from school administration, student self-view, and other guardians.

**Independent Test**: Authenticate as a guardian with an active account explicitly linked by a school administrator to an active same-school guardian record, request the guardian student list, and verify only active same-school students with active school-approved guardian associations are returned.

**Acceptance Scenarios**:

1. **Given** an authenticated guardian has an active account explicitly linked by a school administrator to an active same-school guardian record, an active resolved school context, and an active association to one student profile in that school, **When** they list guardian-visible students, **Then** only that permitted student appears with the documented limited profile summary.
2. **Given** the same guardian is associated with students in two different schools, **When** they use one active school context, **Then** only associations from that school are returned.
3. **Given** a guardian has no active approved student association in the resolved school, **When** they request guardian-visible students, **Then** the response contains no student records and does not reveal whether other student profiles exist.
4. **Given** a guardian account is inactive, the school is inactive, the guardian record is inactive or deleted, or the student profile is inactive, transferred, or deleted, **When** the guardian requests self-service access, **Then** access is rejected or the affected student is omitted according to the published contract.

---

### User Story 2 - Review Permitted Academic Information (Priority: P2)

An authenticated guardian reviews permitted academic summary information for a linked active student, such as current grade summary, attendance totals or status, and learning-set progress or status where approved, without receiving the student's full self-view capabilities.

**Why this priority**: Guardians need enough academic visibility to support a student, but the feature must not grant student-only downloads, private teacher content access, report output access, correction history, or administrative powers.

**Independent Test**: Authenticate as a guardian linked to one active student, request that student's guardian academic summary view, and verify current active grade summary, attendance totals or status, and documented learning-set progress or status are visible while detailed grade records, detailed attendance records, teacher content downloads, private correction notes, other-student records, inactive records, and cross-tenant records are blocked.

**Acceptance Scenarios**:

1. **Given** a guardian has an active approved association to a student, **When** they request that student's academic summary view for an explicit same-school academic period, **Then** only documented current active academic summaries for that student, school, and academic period are returned.
2. **Given** grade or attendance records have correction history from the teacher workflow lifecycle slice, **When** the guardian views academic summary information, **Then** aggregate or current-status summary values may be shown while detailed record rows, private correction reasons, internal actor metadata, and prior values are hidden unless OpenAPI explicitly documents a guardian-visible label.
3. **Given** a learning set includes private teacher content, questionnaires, or file downloads, **When** the guardian views academic information, **Then** only guardian-approved summary status is returned and the guardian cannot download teacher content or answer student activities.
4. **Given** the guardian requests academic records for a missing, unassociated, inactive, or cross-tenant student, **When** the request is submitted, **Then** the same not-found envelope is returned without exposing protected record existence.
5. **Given** the guardian requests academic records with a missing academic period, an unsupported period, or an unsupported include, **When** the request is submitted, **Then** the request is rejected according to the documented validation or not-found envelope without exposing protected record existence.

---

### User Story 3 - Review Contact and School-Approved Relationship Information (Priority: P3)

An authenticated guardian views their own school-maintained guardian contact record, school-approved relationship details, and the linked student's primary school-approved contact details, while sensitive school-only notes and other guardian contact records remain hidden.

**Why this priority**: Guardian self-service should reduce school-office lookup needs, but contact information is privacy-sensitive and must be limited to school-approved fields.

**Independent Test**: Authenticate as a guardian, retrieve the guardian contact view for an approved student association, and verify the response includes only the guardian's own contact data, permitted relationship labels, and the student's primary school-approved contact details, not other guardians, school-only notes, staff-only emergency handling data, or unapproved student contacts.

**Acceptance Scenarios**:

1. **Given** a guardian is linked to an active same-school student association, **When** they request contact information, **Then** they see only documented guardian-owned contact fields, permitted relationship labels, and the student's primary school-approved contact details.
2. **Given** a student has multiple guardians or emergency contacts, **When** one guardian requests contact information, **Then** other guardian records and restricted contact details are hidden unless the contract explicitly marks a field as guardian-visible.
3. **Given** the school has not approved a guardian-student association or has deactivated it, **When** the guardian requests profile, academic, or contact information for that student, **Then** the same not-found envelope used for missing target students is returned.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before guardian, student, academic, or contact data is accessed?
- What happens when an authenticated user has a valid account but no active guardian record in the resolved school?
- How does the backend prove guardian access when a guardian record exists but is not linked to the authenticated user account?
- What happens when a school administrator deactivates a guardian, deactivates a guardian-student association, transfers a student, or deactivates the student profile after guardian access was previously available?
- How does the backend prevent guardian users from gaining school-admin, teacher, report, support, or student self-view capabilities through guardian self-service routes?
- What happens when a guardian attempts to enumerate student identifiers, academic-period identifiers, guardian identifiers, teacher content identifiers, report-run identifiers, or contacts from another school?
- How does the backend hide private correction reasons, teacher-only notes, private file paths, malware-scan details, report outputs, and internal actor metadata from guardian-facing responses?
- How does the backend avoid implementing self-claiming, consent-signature capture, messaging, notification-center behavior, frontend behavior, report lifecycle expansion, platform support access, billing, or undocumented APIs in this slice?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement a backend-only guardian self-service slice for authenticated guardian student listing, linked student detail summary, permitted academic summary-only view, and permitted contact view. Backend work must include request validation, authorization policies, service-layer visibility rules, response resources, tenant context enforcement, audit events for allowed and denied access, inactive-record handling, and regression tests.
- **Frontend repository impact**: No frontend implementation is included. Future guardian portal screens must consume only the OpenAPI operations approved by this slice and must not rely on school-admin, student self-view, or undocumented backend behavior.
- **Specification or contract repository impact**: OpenAPI must be expanded before backend exposure because current contracts publish school-admin guardian management, student self-view, reporting, student enrollment, and teacher workflow lifecycle behavior, but do not publish guardian self-service views.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines guardian self-service boundaries and OpenAPI first, `schoolmaster-backend` implements only approved contract behavior next, and `schoolmaster-frontend` remains deferred until backend verification remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: Yes. The active feature contract and aggregate contract must define operation IDs, paths, request schemas, response schemas, field visibility, error envelopes, authorization rules, tenant behavior, and non-enumerating denial behavior before backend implementation begins.
- **Versioned endpoints affected**: Proposed contract surface is limited to guardian self-service operations under `/api/v1/guardian/students`, `/api/v1/guardian/students/{studentProfileId}`, `/api/v1/guardian/students/{studentProfileId}/academics`, and `/api/v1/guardian/students/{studentProfileId}/contacts` unless OpenAPI approves a different path set.
- **JSON response impact**: Guardian self-service responses must use documented success, paginated, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, not-found, and empty-result envelopes. Target-specific requests for missing, unassociated, inactive, or cross-tenant students must return the same not-found envelope. No backend-local envelope, ad hoc error code, undocumented status code, undocumented field, undocumented include, undocumented filter, or undocumented sort behavior is approved.
- **Authentication/authorization impact**: All operations require authenticated guardian access, an active permitted school context, an active guardian record explicitly linked to the authenticated user by a school administrator, and an active school-approved association to the target student profile. Guardian self-service permission does not imply school administration, teacher workflow access, student self-view access, report access, platform support access, or access to unassociated students.
- **Compatibility impact**: This is additive against the current backend foundation, school administration, student reporting, student enrollment, classroom roster, account lifecycle, and teacher workflow lifecycle behavior. Any change to school-admin guardian management, student self-view, report access, teacher content download, correction visibility, student transfer, or account lifecycle semantics requires explicit contract revision and compatibility review.

### Data & Tenancy Impact

- **Tenant scoping impact**: Guardian records, guardian-user links, guardian-student associations, student profiles, academic records, contact fields, and audit events used by guardian self-service are school-owned records governed by `school_id`.
- **Cross-tenant or platform access impact**: Cross-tenant references must be rejected before data exposure. A guardian associated with students in multiple schools sees only the active resolved school context. Platform users receive no implicit guardian self-service, support, report, or school-owned visibility bypass.
- **Soft delete impact**: Deactivated or deleted guardian records, guardian-user links, guardian-student associations, student profiles, academic records, and contact records are not guardian-visible in current self-service views unless OpenAPI explicitly documents limited historical labels. Permanent purge, legal hold, anonymization, and guardian-facing restore behavior are outside this slice.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend MUST NOT expose guardian self-service behavior until OpenAPI defines operation IDs, request schemas, response schemas, field visibility, errors, authorization rules, and tenant behavior for each approved operation.
- **FR-002**: Every operation in this slice MUST resolve an active permitted school context before authorization, validation, guardian lookup, student lookup, academic-period lookup, academic-record lookup, contact lookup, audit recording, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any guardian, guardian-user link, guardian-student association, student profile, academic record, contact, report, teacher content, file, or cross-school data is accessed.
- **FR-004**: Guardian self-service operations MUST require an authenticated active user account explicitly linked by a school administrator to an active same-school guardian record before any student profile, academic, or contact information is returned.
- **FR-005**: Guardian access to a student MUST require an active school-approved guardian-student association in the resolved school. Guardian self-claiming, unverified association requests, invite acceptance as association proof by itself, and guardian-created student links are outside v1.
- **FR-006**: School approval for guardian self-service in v1 MUST be represented by school-admin maintained guardian records, explicit guardian-user links, and guardian-student associations. Automatic contact matching, invite completion as link creation, separate consent-signature capture, custody dispute workflows, legal-document uploads, and multi-party approval workflows are outside this slice unless specified later.
- **FR-007**: Guardian student listing MUST return only active same-school student profiles associated with the authenticated guardian through active approved associations and MUST use only documented pagination, filtering, and sorting behavior.
- **FR-008**: Guardian student detail MUST expose only documented limited profile and enrollment summary fields for permitted active students. It MUST NOT expose school-only notes, internal identifiers not in contract, disciplinary records, other guardian details, teacher-only notes, report outputs, or private file references.
- **FR-009**: Guardian academic view MUST require an explicit same-school academic period and MUST expose only documented current active academic summaries for the permitted student and selected academic period, limited to current grade summary, attendance totals or status, and learning-set progress or status where OpenAPI documents each summary. It MUST NOT expose detailed grade rows, detailed attendance rows, private correction reasons, full correction history, internal actor metadata, private teacher content, questionnaire answer keys, student activity submission capabilities, or report-run data.
- **FR-010**: Guardian contact view MUST expose only documented guardian-owned contact fields, school-approved relationship labels, and the student's primary school-approved contact details. Other guardian contact records, non-primary student contact details, restricted emergency handling details, and school-only contact notes MUST remain hidden unless OpenAPI explicitly documents them as guardian-visible.
- **FR-011**: Guardian self-service MUST NOT permit creating, updating, deactivating, deleting, restoring, transferring, correcting, importing, downloading teacher content, requesting reports, downloading reports, submitting student activities, or managing users, roles, guardians, students, rosters, teacher workflows, account lifecycle, or school configuration.
- **FR-012**: If a guardian, guardian-user link, guardian-student association, school, student profile, academic period, grade, attendance record, learning-set summary, or contact record becomes inactive or deleted, guardian current self-service views MUST hide or reject the affected data according to OpenAPI.
- **FR-013**: Student transfer behavior MUST preserve source-school tenant isolation. A guardian's access to a transferred student in the source school MUST end for current self-service unless OpenAPI explicitly documents limited source-school historical labels, and destination-school access MUST require a separate active destination-school guardian association.
- **FR-014**: Backend authorization MUST keep platform administration, school administration, guardian self-service, teacher workflows, student self-view, reporting, account lifecycle administration, and support access permissions separate.
- **FR-015**: Backend validation MUST reject undocumented request fields, unsupported filters, unsupported include expansion, unsupported sort values, invalid payload shapes, inactive references, deleted references, cross-tenant references, unsupported academic periods, and attempts to target unassociated students.
- **FR-016**: Backend responses MUST match the published OpenAPI success and error semantics declared for each approved operation. Target-specific requests for missing, unassociated, inactive, or cross-tenant students MUST return the same not-found envelope. The backend MUST NOT emit undocumented status semantics, fields, filters, includes, or denial details unless OpenAPI first documents them.
- **FR-017**: Guardian-facing denial behavior MUST avoid protected-record enumeration. Unauthorized, unassociated, inactive, cross-tenant, and missing target records MUST not reveal whether a protected student, guardian, academic record, contact record, teacher content item, or report exists outside the guardian's permitted view.
- **FR-018**: Backend audit events MUST record guardian self-service access grants, denied access attempts, blocked cross-tenant attempts, and visibility-sensitive reads with actor user ID when available, canonical school and target identifiers when safely resolved, outcome, reason category where safe, timestamp, and tenant-safe summary metadata only.
- **FR-019**: Backend implementation MUST NOT expose guardian self-claiming, guardian profile update, consent-signature capture, custody dispute workflows, legal-document handling, messaging, notifications, report lifecycle expansion, platform-wide support access, frontend behavior, billing, payment, permanent purge, anonymization, legal hold, or undocumented APIs in this slice.
- **FR-020**: Backend tests MUST cover successful guardian views, validation failures, authorization failures, tenant isolation failures, inactive school/user/guardian/association/student/period handling, unassociated-student rejection, non-enumerating denial behavior, correction-data redaction, contact-data redaction, student transfer visibility, audit event coverage, and response shape for every operation in this slice.

### Key Entities *(include if feature involves data)*

- **Guardian**: Existing school-owned responsible contact record with lifecycle state and contact fields. Guardian self-service requires the record to be active and linked to the authenticated user.
- **GuardianUserLink**: School-owned proof that an authenticated user account is allowed to act as a specific guardian in one school context. The link must be explicitly created by a school administrator, school-approved, active, and same-school.
- **GuardianStudentAssociation**: School-approved same-school relationship between an active guardian and an active student profile that grants guardian self-service visibility for that student only.
- **StudentProfile**: Existing school-owned learner profile whose limited guardian-visible profile and enrollment summary may be exposed only through active approved guardian association.
- **AcademicPeriod**: Existing school-owned period required to filter guardian-visible academic summary information in v1.
- **GradeSummary**: Guardian-facing summary derived from existing school-owned grade records for a permitted active student and academic period; detailed grade rows and correction history are not guardian-visible in v1.
- **AttendanceSummary**: Guardian-facing summary derived from existing school-owned attendance records for a permitted active student and academic period; detailed attendance rows and correction history are not guardian-visible in v1.
- **LearningSetSummary**: Limited guardian-facing summary of student learning progress or status where OpenAPI documents it; it does not include private teacher content downloads, questionnaire answer keys, or student activity submission powers.
- **ContactVisibilityRule**: Contract-governed field visibility rule that limits guardian self-service contact responses to the authenticated guardian's own contact fields, school-approved relationship labels, and the student's primary school-approved contact details.
- **AuditEvent**: Tenant-safe record of guardian self-service reads, denials, and cross-tenant blocks without storing private files, full payloads, school-only notes, credentials, or unauthorized cross-tenant details.
- **School**: Tenant root whose active state and `school_id` boundary control guardian, student, academic, contact, and audit visibility.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented guardian self-service routes in this slice to approved OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for guardian student listing, student detail, academic view, and contact view.
- **SC-003**: Association-proof tests reject 100% of guardian access attempts where the authenticated user lacks an active same-school guardian-user link and active school-approved guardian-student association.
- **SC-004**: Authorization tests confirm guardian self-service grants 0 school-admin, teacher, student self-view, report, account administration, platform support, import, correction, or content-download capabilities.
- **SC-005**: Visibility tests confirm 100% of guardian responses omit detailed grade rows, detailed attendance rows, private correction reasons, full correction history, internal actor metadata, private teacher content, questionnaire answer keys, other guardian records, non-primary student contact details, restricted contact notes, report outputs, and cross-tenant identifiers covered by this specification.
- **SC-006**: Lifecycle tests confirm inactive or deleted schools, guardian records, guardian-user links, guardian-student associations, student profiles, academic periods, and academic records are hidden or rejected according to OpenAPI in 100% of covered cases.
- **SC-006a**: Academic-period validation tests reject 100% of guardian academic summary requests with missing, inactive, unsupported, cross-tenant, or unauthorized academic periods.
- **SC-007**: Student transfer tests confirm destination-school guardian access requires a separate active destination-school association and that source-school academic records are not exposed across tenants.
- **SC-008**: Non-enumeration tests confirm target-specific requests for missing, unassociated, inactive, or cross-tenant students return the same not-found envelope and do not reveal protected student, guardian, academic, contact, teacher-content, or report existence outside the guardian's permitted view.
- **SC-009**: Response-shape verification confirms every operation in this slice returns the documented success or error envelope for successful, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, empty-result, and not-found cases.
- **SC-010**: Audit verification confirms 100% of guardian self-service allowed reads, denied access attempts, and blocked cross-tenant attempts covered by this slice create tenant-safe audit events without private payloads, school-only notes, credentials, file paths, or unauthorized cross-tenant details.

## Assumptions

- `003-backend-school-admin` established school-admin guardian records and guardian-student association creation/listing foundations.
- `006-backend-student-enrollment` established student profile lifecycle and transfer boundaries that guardian visibility must respect.
- `008-account-lifecycle-workflows` established account onboarding, password setup, reset, lock, recovery, and inactive-user behavior that guardian accounts reuse without creating a separate authentication model.
- `010-teacher-workflow-lifecycle` is implemented and established current student visibility rules for active records, correction history, lifecycle states, and private correction-note redaction.
- V1 guardian self-service proof is school-approved association and an explicit school-administrator-created guardian-user link, not guardian self-claiming, automatic contact matching, or invite completion alone. School staff remain responsible for creating and maintaining guardian records, guardian-user links, and guardian-student associations.
- A guardian may have approved associations in more than one school, but every request remains scoped to exactly one active school context and one `school_id` boundary.
- Guardian self-service exposes read-only profile, academic summary, and limited contact views only. Guardian profile maintenance, association requests, detailed academic record rows, other-guardian contact visibility, non-primary student contact details, consent-signature capture, legal-document handling, custody dispute workflows, messaging, notification-center behavior, frontend implementation, report lifecycle expansion, teacher content downloads, billing, and undocumented APIs remain outside this slice.
