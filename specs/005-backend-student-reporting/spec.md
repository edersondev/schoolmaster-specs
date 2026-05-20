# Feature Specification: Backend Student and Reporting Foundation

**Feature Branch**: `005-backend-student-reporting`  
**Created**: 2026-05-20  
**Status**: Draft  
**Input**: User description: "Define the next SchoolMaster backend implementation slice after `004-backend-teacher-workflows`. The backend must implement the P3 student and reporting foundation from the approved platform specification and OpenAPI contract: authenticated student learning timeline filtered by academic period, authorized student teacher-content download after malware scan passes, student grade and attendance self-view, school-scoped asynchronous report requests, report listing, report output download, 90-day output retention, expired-output handling, and explicit regeneration through a new `ReportRun` with the same filters. Preserve contract-first delivery, tenant-by-column isolation with `School` as tenant root and `school_id` for school-owned records, explicit platform versus school authorization, API-only `/api/v1` behavior, OpenAPI-aligned response envelopes, validation, inactive-status handling, selected academic-period enforcement, private file authorization, report filter scoping, output availability semantics, and regression coverage. Do not include frontend implementation, teacher workflow writes, report designer/custom report definitions, platform-wide reporting, automatic expired-output regeneration during download, classroom/course/section/roster workflows, or undocumented APIs in this slice."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Review Student Learning Timeline (Priority: P1)

An authenticated student in an active school tenant views learning sets assigned to their own active `StudentProfile` for a selected academic period, including ordered entries and metadata for clean assigned teacher content.

**Why this priority**: Student access to assigned learning material is the core student-facing outcome of the teacher workflow foundation and validates that student self-view remains tenant-scoped and assignment-scoped.

**Independent Test**: Authenticate as a student with one active same-school `StudentProfile`, request the learning timeline for an academic period with assigned learning sets, and verify only the student's own assigned learning sets are returned, ordered by publish date with entries ordered by teacher-defined sequence.

**Acceptance Scenarios**:

1. **Given** an authenticated student has an active resolved school context and an active same-school `StudentProfile`, **When** they list assigned learning sets for an academic period, **Then** only their own assigned learning sets for that school and period are returned with the documented paginated response envelope.
2. **Given** a student's assigned learning set contains content and questionnaire entries, **When** the timeline is returned, **Then** learning sets are ordered by publish date and entries are ordered by teacher-defined sequence.
3. **Given** a content entry is not malware-scan clean or is otherwise unavailable, **When** the student timeline is returned, **Then** the item does not expose a downloadable file and its availability metadata follows the published contract.

---

### User Story 2 - Access Own Academic Records and Authorized Content (Priority: P2)

A student reviews their own grades and attendance and downloads only assigned teacher content that is clean, private, same-school, and authorized for that student.

**Why this priority**: Students need direct visibility into their academic status and safe access to assigned material without gaining access to other students' records or unapproved files.

**Independent Test**: Authenticate as a student, list grades and attendance for their own profile, download one assigned clean content item, and verify attempts to access cross-tenant, unassigned, inactive, pending-scan, failed-scan, or other-student records are rejected.

**Acceptance Scenarios**:

1. **Given** an authenticated student has grade records in the resolved school, **When** they list grades with an optional academic-period filter, **Then** only their own grade records in the permitted school scope are returned.
2. **Given** an authenticated student has attendance records in the resolved school, **When** they list attendance with an optional academic-period filter, **Then** only their own attendance records in the permitted school scope are returned.
3. **Given** an assigned teacher content file is private, same-school, linked through a learning set assigned to the student, and scan status is `clean`, **When** the student downloads the content, **Then** the backend returns only the authorized file stream.
4. **Given** a content item is missing, cross-tenant, unassigned to the student, inactive, scan status `pending` or `failed`, or outside the resolved school context, **When** the student attempts to download it, **Then** the request is rejected without revealing protected file storage details.

---

### User Story 3 - Request and Retrieve School Reports (Priority: P3)

A school-scoped administrator requests asynchronous school reports for an academic period, reviews report-run status, downloads generated PDF or CSV outputs while available, and requests a new report run when previous output files have expired.

**Why this priority**: Reporting provides operational oversight from the completed P1 and P2 workflows while preserving tenant boundaries, private output storage, and explicit retention behavior.

**Independent Test**: Authenticate as a school administrator, request each launch-scope report type for an academic period, list report runs, simulate generated outputs, download PDF and CSV files before expiry, verify expired outputs return the documented expired-output response, and create a new `ReportRun` with the same filters for regeneration.

**Acceptance Scenarios**:

1. **Given** an authenticated school administrator has an active resolved school context, **When** they request a launch-scope report with a same-school academic period and valid filters, **Then** a `ReportRun` is accepted with status and output availability metadata for that school only.
2. **Given** report runs exist for the resolved school, **When** the administrator lists reports with documented pagination and report-type filtering, **Then** only same-school report runs are returned.
3. **Given** a report run has status `generated` and unexpired PDF or CSV output, **When** the administrator downloads a requested format, **Then** the backend returns only the authorized generated file.
4. **Given** a report output file is older than the 90-day retention period, **When** the administrator attempts to download it, **Then** the backend returns the documented expired-output response and does not regenerate the file during download.
5. **Given** a report output has expired, **When** the administrator requests a new report using the same filters, **Then** a new `ReportRun` is created while the expired run metadata remains available for history.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before student or report data access?
- What happens when an authenticated user has no active `StudentProfile` in the resolved school but attempts student self-view endpoints?
- How does the backend prevent a student from listing or downloading another student's learning sets, grade records, attendance records, or assigned content?
- What happens when assigned content is pending scan, scan failed, inactive, deleted, unassigned, cross-tenant, or not linked through a valid learning-set assignment?
- How does the backend enforce academic-period filtering for student timelines and report requests when the period is missing, inactive, closed, or belongs to another school?
- What happens when report filters reference a student, user, status, or date range outside the resolved school or outside the selected academic period?
- How does the backend expose report output availability without revealing private file paths, storage keys, or cross-tenant file existence?
- How does the backend preserve `ReportRun` metadata after output expiry while preventing automatic regeneration during download?
- How does the backend avoid implementing frontend behavior, teacher workflow writes, custom report definitions, platform-wide reporting, classroom/course/section/roster workflows, or undocumented APIs in this slice?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement the backend-only P3 student and reporting slice for student learning timeline, student content download, student grade and attendance self-view, report listing, report request, and report output download using established Laravel request validation, policy authorization, services, resources, tenant context, private file storage, asynchronous report-run workflow, retention handling, and regression tests.
- **Frontend repository impact**: No frontend implementation is included in this feature. Student and reporting SPA screens remain dependent on a later frontend slice that consumes only the published OpenAPI operations.
- **Specification or contract repository impact**: This feature records the backend implementation boundary for the P3 student and reporting operations already published in `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml`. Any report designer, custom report definition, platform-wide report, teacher correction, classroom/course/section/roster, or frontend behavior must be specified and added to OpenAPI before backend exposure.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines the slice and contract boundary first, `schoolmaster-backend` implements only the approved operations next, and `schoolmaster-frontend` consumes the behavior only after backend verification remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: No for the currently published P3 operations in this slice. Yes before exposing any report designer/custom report definitions, platform-wide reporting, report deletion, report retry, teacher correction, classroom/course/section/roster workflow, or undocumented student endpoint.
- **Versioned endpoints affected**: `/api/v1/student/learning-sets`, `/api/v1/student/teacher-content/{contentItemId}/download`, `/api/v1/student/grades`, `/api/v1/student/attendance`, `/api/v1/reports`, and `/api/v1/reports/{reportRunId}/download`.
- **JSON response impact**: Student and report JSON responses must use only the success, paginated, validation, unauthorized, tenant-mismatch, not-found, and expired-output envelopes or binary responses declared by OpenAPI for the affected operation. No backend-local envelope, ad hoc error code, undocumented status code, undocumented field, undocumented filter, or undocumented sort behavior is approved.
- **Authentication/authorization impact**: All operations require authenticated access and an active permitted school context. Student self-view operations require the authenticated user to be linked to the active same-school `StudentProfile` whose records are being exposed. Report operations require school-scoped report permission. Platform administration does not imply school report or student self-view access without an explicit permitted school context.
- **Compatibility impact**: The slice is additive against the current aggregate contract. Any operation beyond the published OpenAPI surface requires a contract revision before implementation.

### Data & Tenancy Impact

- **Tenant scoping impact**: Learning sets, assignments, teacher content metadata, private teacher content files, grade records, attendance records, report runs, report filters, and generated report files are school-owned records governed by `school_id`.
- **Cross-tenant or platform access impact**: Platform-scope users do not receive implicit access to school-scoped student views or reports. Cross-tenant content, assignment, grade, attendance, academic-period, student-profile, user, report-run, or output-file references must be rejected before data exposure or file access.
- **Soft delete impact**: Student-visible academic records and report metadata should preserve historical and audit expectations. Public report deletion, output purge controls, record correction, restore behavior, or permanent deletion workflows are not part of this slice unless OpenAPI is updated first.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend implementation slice MUST be limited to the published operations `listStudentLearningSets`, `downloadStudentTeacherContent`, `listStudentGrades`, `listStudentAttendance`, `listReports`, `requestReport`, and `downloadReport` unless OpenAPI is updated first.
- **FR-002**: Every operation in this slice MUST resolve an active permitted school context before validation that depends on school-owned records, authorization decisions, report filters, file storage, persistence, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any student learning-set, teacher-content, grade, attendance, report-run, academic-period, student-profile, user, or file data is accessed.
- **FR-004**: Student self-view operations MUST require the authenticated user to be linked to an active `StudentProfile` in the resolved school and MUST expose only records belonging to that student's profile.
- **FR-005**: Student learning-set listing MUST require an academic-period filter, validate that the period belongs to the resolved school, and return only learning sets assigned to the authenticated student's active same-school `StudentProfile`.
- **FR-006**: Student learning-set listing MUST order assigned learning sets by publish date and each learning-set entry by teacher-defined sequence according to the approved contract.
- **FR-007**: Student learning-set responses MUST expose teacher content metadata only as documented for student consumption and MUST NOT expose private file paths, storage keys, scanner details, other students' assignment data, or unsupported questionnaire behavior.
- **FR-008**: Student teacher-content download MUST allow file access only when the content item belongs to the resolved school, is linked through a learning set assigned to the authenticated student's active same-school `StudentProfile`, is operationally available, and has scan status `clean`.
- **FR-009**: Student teacher-content download MUST reject missing, inactive, cross-tenant, unassigned, scan-pending, scan-failed, or otherwise unavailable content without revealing protected file-storage details.
- **FR-010**: Student grade listing MUST return only the authenticated student's same-school grade records and MUST support only the documented academic-period and pagination filters.
- **FR-011**: Student attendance listing MUST return only the authenticated student's same-school attendance records and MUST support only the documented academic-period and pagination filters.
- **FR-012**: Report listing MUST return only report runs visible in the resolved school context and MUST support only documented pagination and report-type filtering.
- **FR-013**: Report request creation MUST require a launch-scope report type, a same-school `academic_period_id`, and only documented filters that remain within the resolved school and selected period.
- **FR-014**: Report request creation MUST accept filters for `student_profile_id`, `user_id`, `status`, `start_date`, and `end_date` only where relevant to the selected report type and MUST reject unsupported or cross-tenant filter references.
- **FR-015**: Report generation MUST be asynchronous for every launch-scope report type; a successful request MUST create a `ReportRun` with requested status and output availability metadata rather than blocking on completed files.
- **FR-016**: Report outputs MUST be generated only in the documented PDF and CSV formats for launch-scope report types and stored as private tenant-scoped files.
- **FR-017**: Report download MUST allow access only to same-school generated report outputs in the requested documented format and MUST reject unauthorized, missing, failed, inactive, or cross-tenant report runs.
- **FR-018**: Generated report output files MUST remain available for 90 days after generation and MUST become unavailable after expiry while preserving `ReportRun` metadata.
- **FR-019**: Expired report output download MUST return the documented expired-output response and MUST NOT regenerate output files during the download request.
- **FR-020**: Regenerating an expired report output MUST require creating a new `ReportRun` with the same filters; the original expired `ReportRun` metadata MUST remain available for authorized history views.
- **FR-021**: Backend validation MUST reject undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, invalid report types, invalid report formats, invalid date ranges, and invalid academic-period, student-profile, user, content, or report-run references.
- **FR-022**: Backend authorization MUST keep platform-scope administration, school-scoped reporting, and student self-view access separate; system administrator access MUST NOT imply school report or student self-view access unless a future specification and contract revision explicitly grants it.
- **FR-023**: Backend responses MUST match the published OpenAPI success, binary, and error semantics declared for each approved operation. The backend MUST NOT emit undocumented status semantics for these operations unless OpenAPI first documents them on the relevant operation.
- **FR-024**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, inactive school/user/student/period handling, student ownership boundaries, scan-gated content access, report filter scoping, asynchronous status behavior, output expiry, no download-time regeneration, and response shape for every operation in this slice.
- **FR-025**: Backend implementation MUST NOT expose frontend implementation, teacher workflow writes, report designer/custom report definitions, platform-wide reporting, report deletion, report retry, classroom/course/section/roster workflows, correction workflows, or undocumented APIs in this slice unless `/specs` and OpenAPI are updated first.

### Key Entities *(include if feature involves data)*

- **StudentProfile**: Existing school-owned student profile linked to a user identity; student self-view operations require an active profile in the resolved school.
- **LearningSet**: Existing school-owned instructional sequence for one academic period, visible to a student only when assigned to that student's same-school profile.
- **LearningSetEntry**: Ordered content or questionnaire entry within a learning set, displayed to students in teacher-defined sequence.
- **LearningSetAssignment**: Existing same-school assignment connecting a learning set to a selected `StudentProfile`.
- **TeacherContentItem**: Existing school-owned instructional content metadata and private file reference; student download requires assignment, availability, and clean scan status.
- **GradeRecord**: Existing school-owned academic record for one student profile and academic period, visible through student self-view only to the linked student.
- **AttendanceRecord**: Existing school-owned attendance record for one student profile, academic period, and date, visible through student self-view only to the linked student.
- **AcademicPeriod**: Existing school-owned period used to filter student timelines and constrain report requests.
- **ReportRun**: School-owned asynchronous report request containing requester, report type, selected filters, output formats, status, generation timestamp, output expiry timestamp, and output availability metadata.
- **ReportOutput**: Private tenant-scoped generated PDF or CSV file associated with a `ReportRun`; output files expire after 90 days while metadata remains.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented routes in this slice to the listed OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for student learning sets, student content downloads, student grades, student attendance, report requests, report listing, and report downloads.
- **SC-003**: Student ownership tests confirm 100% of other-student, inactive-profile, cross-tenant, and unassigned access attempts are rejected for learning timelines, academic records, and content downloads.
- **SC-004**: Content-download tests confirm 100% of pending-scan, failed-scan, inactive, missing, cross-tenant, and unassigned teacher-content downloads are unavailable to students.
- **SC-005**: Report request tests reject 100% of invalid report types, invalid filter shapes, cross-tenant academic periods, cross-tenant users, cross-tenant student profiles, and unsupported filters covered by this specification.
- **SC-006**: Report output tests confirm generated PDF and CSV outputs are downloadable before expiry and return the documented expired-output response after 90 days without automatic regeneration.
- **SC-007**: Response-shape verification confirms every operation in this slice returns the documented success, binary, or error behavior for successful, validation where applicable, unauthorized, tenant-mismatch, not-found, and expired-output cases exactly as declared by OpenAPI for that operation.
- **SC-008**: A student can review assigned learning sets, download one clean authorized content file, and review their own grades and attendance in an active school tenant without exposing records from any other student or school.
- **SC-009**: A school administrator can request, list, and download generated reports for the launch-scope report types in one active school tenant without exposing records or output files from any other school.

## Assumptions

- `004-backend-teacher-workflows` has already established teacher content, questionnaires, learning sets, learning set assignments, grade records, attendance records, scan status, and tenant-scoped teacher workflow patterns.
- Current OpenAPI contracts already publish student learning-set listing, student teacher-content download, student grade listing, student attendance listing, report listing, report request, and report download operations; implementation is restricted to those operations.
- Launch-scope report types remain attendance, grades, academic structure, and school activity summaries as defined by the approved platform specification and OpenAPI schema.
- Report generation workers or adapters may be implemented behind the backend service boundary, but no client-visible behavior beyond `ReportRun` status and output availability is approved in this slice.
- Student profile creation, enrollment changes, guardian access, teacher corrections, and school-admin data management remain outside this slice except where existing records are validated for self-view or reporting.
- Classroom, course, section, group, roster, custom report designer, platform-wide reporting, frontend implementation, and automatic expired-output regeneration remain outside this slice.
