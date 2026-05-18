# Feature Specification: Backend Teacher Workflow Foundation

**Feature Branch**: `004-backend-teacher-workflows`  
**Created**: 2026-05-18  
**Status**: Draft  
**Input**: User description: "Define the next SchoolMaster backend implementation slice after `003-backend-school-admin`. The backend must implement the P2 teacher workflow foundation from the approved platform specification and OpenAPI contract: teacher content folders and uploaded content, questionnaires, learning sets assigned directly to selected active `StudentProfile` records, grade records, and attendance records. Preserve contract-first delivery, tenant-by-column isolation with `School` as tenant root and `school_id` for school-owned records, explicit platform versus school authorization, API-only `/api/v1` behavior, OpenAPI-aligned response envelopes, validation, inactive-status handling, upload validation and sanitization, malware-scan-gated content availability, academic-period enforcement, selected-student same-school validation, and regression coverage. Do not include student self-service, reporting, classroom/course/section/roster workflows, or frontend implementation in this slice."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage Teacher Content and Questionnaires (Priority: P1)

A teacher in an active school tenant uploads instructional content and creates questionnaires that can later be used in learning sets.

**Why this priority**: Content and questionnaires are prerequisites for learning sets, student assignments, and later student-facing access.

**Independent Test**: Authenticate as a teacher with an active resolved school context, upload one allowed instructional file, verify it is unavailable until the malware scan marks it clean, create one questionnaire using only v1 question types, and confirm both lists remain tenant-scoped.

**Acceptance Scenarios**:

1. **Given** an authenticated teacher has an active resolved school context, **When** they list teacher content and questionnaires, **Then** only records visible in that school context are returned with the approved response envelope.
2. **Given** a teacher uploads a PDF, image, text file, or office document up to 25 MB, **When** the file passes declared type, detected type, size, tenant, and permission checks, **Then** the content item is stored privately for the resolved school with scan status `pending`.
3. **Given** uploaded content has scan status `pending` or `failed`, **When** a workflow tries to expose the file for teacher or student use, **Then** the file remains unavailable until an authorized malware-scan workflow marks it `clean`.
4. **Given** a teacher creates a questionnaire, **When** the questions use only `multiple_choice`, `true_false`, or `short_text`, **Then** the questionnaire is created for the resolved school with the submitted question sequence.

---

### User Story 2 - Publish Learning Sets for Selected Students (Priority: P2)

A teacher creates a learning set for an active academic period, orders content and questionnaire entries, and assigns the learning set directly to selected active student profiles from the same school.

**Why this priority**: Learning sets are the instructional bridge between teacher-created materials and later student learning timelines.

**Independent Test**: Create clean content and an active questionnaire in one school, create a learning set for an active academic period with ordered entries, assign it to selected active `StudentProfile` records from the same school, and verify cross-tenant, inactive, unclean, or wrong-period references reject the request atomically.

**Acceptance Scenarios**:

1. **Given** a teacher has clean content, an available questionnaire, an active academic period, and selected active students in the resolved school, **When** they create a learning set, **Then** the set, entries, and assignments are created for that school only.
2. **Given** a learning set references content or questionnaires, **When** any referenced item is inactive, cross-tenant, missing, or unavailable because scan status is not `clean`, **Then** the request fails without partial learning-set persistence.
3. **Given** a learning set includes student assignments, **When** any selected `StudentProfile` is inactive, missing, cross-tenant, or not valid for the selected academic period, **Then** the request fails without partial assignments.

---

### User Story 3 - Record Grades and Attendance (Priority: P3)

A teacher records grades and attendance for selected active student profiles during an active academic period.

**Why this priority**: Grades and attendance are core teacher operations and provide the data that later student views and reports will consume.

**Independent Test**: Authenticate as a teacher, resolve an active school and academic period, record one grade and one attendance entry for an active same-school student profile, list the records, and verify inactive, cross-tenant, closed-period, or invalid-value attempts are rejected.

**Acceptance Scenarios**:

1. **Given** a teacher has permission in an active school context, **When** they record a grade with a numeric value from 0 to 100 for an active same-school student profile in an active academic period, **Then** the grade is stored with the teacher as recorder and appears only in permitted school-scoped lists.
2. **Given** a teacher has permission in an active school context, **When** they record attendance using `present`, `absent`, `late`, `excused`, `remote`, or `suspended`, **Then** the attendance record is stored for the selected student and academic period.
3. **Given** the selected student, academic period, teacher, or school context is inactive, missing, unauthorized, or cross-tenant, **When** grade or attendance creation is attempted, **Then** the request is rejected before school-owned data is exposed or persisted.

### Edge Cases

- How does the backend reject missing, inactive, mismatched, or unauthorized `X-School-Id` tenant context before teacher workflow data access?
- What happens when uploaded instructional files are unsupported, exceed 25 MB, have a mismatched declared versus detected content type, or are executable/archive files?
- What happens when malware scanning fails, remains pending, or attempts to expose content before the scan is clean?
- How does the backend validate a learning set that references inactive, missing, unclean, or cross-tenant content and questionnaires?
- What happens when learning set assignments include inactive, missing, cross-tenant, or wrong-period `StudentProfile` records?
- How does the backend enforce active academic periods for learning sets, grades, and attendance while keeping closed periods read-only?
- How does the backend avoid implementing student self-service, reports, classroom/course/section/roster workflows, update/delete operations, or public folder management that are not approved for this slice?

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Implement the backend-only teacher workflow slice for the approved `/api/v1` teacher content, questionnaire, learning set, grade, and attendance list/create operations using Laravel services, Form Requests, Policies, API Resources, tenant context, private file storage, scan-status workflow, and regression tests.
- **Frontend repository impact**: No frontend implementation is included in this feature. Frontend teacher views and stores remain dependent on a later frontend slice that consumes only the published OpenAPI operations.
- **Specification or contract repository impact**: This feature records the backend implementation boundary for the P2 teacher workflow operations already published in `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml`. Any folder CRUD, detail, update, delete, teacher download, correction, classroom/course/section/roster, student self-service, or reporting behavior must be added to OpenAPI before backend exposure.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines the slice and contract boundary first, `schoolmaster-backend` implements only the approved operations next, and `schoolmaster-frontend` consumes the behavior only after backend verification remains contract-compliant.

### API Contract Impact

- **OpenAPI update required**: No for the currently published list/create operations in this slice. Yes before exposing public content-folder management, detail endpoints, update, delete, downloads, corrections, classroom/course/section/roster workflows, student self-service, or reports.
- **Versioned endpoints affected**: `/api/v1/teacher-content`, `/api/v1/questionnaires`, `/api/v1/learning-sets`, `/api/v1/grades`, and `/api/v1/attendance`.
- **JSON response impact**: Responses must use the documented success, paginated, validation, unauthorized, forbidden, tenant-mismatch, token-rejection or inactive-user/inactive-school, and not-found envelopes or codes. No backend-local envelope, ad hoc error code, undocumented field, undocumented filter, or undocumented sort behavior is approved.
- **Authentication/authorization impact**: All operations require authenticated access and an active permitted school context. Teacher workflow permissions must remain school-scoped; platform administration must not imply teacher workflow access without an explicit permitted school context.
- **Compatibility impact**: The slice is additive against the current aggregate contract. Any operation beyond the published OpenAPI surface requires a contract revision before implementation.

### Data & Tenancy Impact

- **Tenant scoping impact**: Teacher content, content folders where referenced, questionnaires, learning sets, learning set entries, learning set assignments, grade records, and attendance records are school-owned records governed by `school_id`.
- **Cross-tenant or platform access impact**: Platform-scope users do not receive implicit access to school-scoped teacher workflows. Cross-tenant content, questionnaire, academic-period, teacher, or student references must be rejected before data exposure or persistence.
- **Soft delete impact**: Recoverable instructional and academic records should preserve status and history expectations from the platform specification. Public delete, restore, or correction behavior is not part of this slice unless OpenAPI is updated first.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend implementation slice MUST be limited to the published operations `listTeacherContent`, `createTeacherContent`, `listQuestionnaires`, `createQuestionnaire`, `listLearningSets`, `createLearningSet`, `listGrades`, `createGrade`, `listAttendance`, and `createAttendance` unless OpenAPI is updated first.
- **FR-002**: Every operation in this slice MUST resolve an active permitted school context before validation that depends on school-owned records, authorization decisions, service logic, file storage, persistence, or response shaping.
- **FR-003**: Requests with missing, mismatched, inactive, or unauthorized school context MUST fail before any teacher content, questionnaire, learning set, grade, attendance, academic-period, or student-profile data is accessed.
- **FR-004**: Teacher content creation MUST accept only OpenAPI-documented content fields and file uploads that satisfy the allowed v1 file categories, 25 MB maximum size, declared and detected content-type validation, filename and metadata sanitization, tenant ownership, and authorization rules before persistence.
- **FR-005**: Teacher content creation MUST store uploaded files in private tenant-scoped storage and initialize scan status so content is unavailable until an authorized scan workflow marks it `clean`.
- **FR-006**: Teacher content listing MUST return only content visible in the resolved school context and MUST follow documented pagination and response envelopes.
- **FR-007**: Public teacher-content folder management MUST NOT be exposed in this slice unless OpenAPI first documents folder operations; any submitted `folder_id` on content creation MUST reference an allowed same-school folder if folder persistence already exists.
- **FR-008**: Questionnaire creation MUST accept only the v1 question types `multiple_choice`, `true_false`, and `short_text`, preserve submitted question sequence, and reject unsupported shapes or undocumented fields.
- **FR-009**: Questionnaire listing MUST return only questionnaires visible in the resolved school context and MUST follow documented pagination and response envelopes.
- **FR-010**: Learning set creation MUST require an active same-school academic period, ordered entries, and direct assignment to selected active `StudentProfile` records from the same resolved school.
- **FR-011**: Learning set creation MUST reject missing, inactive, cross-tenant, unclean, or unsupported content and questionnaire references without creating partial learning sets, entries, or assignments.
- **FR-012**: Learning set listing MUST return only learning sets visible in the resolved school context and MUST follow documented pagination and response envelopes.
- **FR-013**: Grade creation MUST require an active same-school `StudentProfile`, an active same-school academic period, a permitted teacher recorder, and a numeric grade value from 0 to 100 with optional label according to OpenAPI.
- **FR-014**: Attendance creation MUST require an active same-school `StudentProfile`, an active same-school academic period, a permitted teacher recorder, and one documented attendance status from `present`, `absent`, `late`, `excused`, `remote`, or `suspended`.
- **FR-015**: Grade and attendance listing MUST return only records visible in the permitted school scope and MUST follow documented pagination and response envelopes.
- **FR-016**: Backend validation MUST reject undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, unsupported question types, invalid file types, invalid scan-status transitions, and invalid academic-period or student references.
- **FR-017**: Backend authorization MUST keep platform-scope administration and school-scoped teacher workflows separate; system administrator access MUST NOT imply teacher workflow access unless a future specification and contract revision explicitly grants it.
- **FR-018**: Backend responses MUST match the published OpenAPI success and error envelopes for successful reads and writes, validation failures, unauthorized requests, forbidden requests, tenant mismatches, inactive-user or inactive-school outcomes, token rejection, and not-found outcomes.
- **FR-019**: Backend tests MUST cover successful flows, validation failures, authorization failures, tenant isolation failures, inactive school/user/student/period handling, file validation and scan gating, learning-set reference failures, grade and attendance value failures, and response shape for every operation in this slice.
- **FR-020**: Backend implementation MUST NOT expose student self-service, reporting, classroom/course/section/roster workflows, public content-folder CRUD, detail, update, deactivate, delete, restore, download, bulk import, or correction behavior in this slice unless `/specs` and OpenAPI are updated first.

### Key Entities *(include if feature involves data)*

- **TeacherContentItem**: School-owned instructional content metadata and private file reference with owner, optional folder reference, content type, size, detected type, scan status, and operational status.
- **TeacherContentFolder**: Internal or pre-existing school-owned organization container for teacher content. This slice may validate submitted `folder_id` references against existing same-school folders but MUST NOT expose folder lifecycle behavior until OpenAPI documents it.
- **Questionnaire**: School-owned teacher-authored assessment or activity containing ordered v1-supported questions.
- **QuestionnaireQuestion**: Ordered question within a questionnaire using `multiple_choice`, `true_false`, or `short_text` behavior.
- **LearningSet**: School-owned instructional sequence for one academic period, owned by a teacher and composed of ordered content or questionnaire entries.
- **LearningSetEntry**: Ordered reference from a learning set to a teacher content item or questionnaire in the same school.
- **LearningSetAssignment**: Direct assignment from a learning set to selected active same-school `StudentProfile` records.
- **GradeRecord**: School-owned academic record for one student profile and academic period, recorded by an authorized teacher with a numeric value from 0 to 100 and optional label.
- **AttendanceRecord**: School-owned attendance record for one student profile, academic period, date, and documented attendance status, recorded by an authorized teacher.
- **AcademicPeriod**: Existing school-owned period that must be active for learning set publication, grade creation, and attendance creation in this slice.
- **StudentProfile**: Existing school-owned student profile used for learning set assignment, grade, and attendance validation; this slice must not create student self-service behavior.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Backend reviewers can map 100% of implemented routes in this slice to the listed OpenAPI operation IDs without finding undocumented product endpoints.
- **SC-002**: Tenant-isolation acceptance tests reject 100% of missing, mismatched, inactive, and unauthorized school-context attempts for teacher content, questionnaires, learning sets, grades, and attendance.
- **SC-003**: Upload validation tests reject 100% of unsupported file categories, executable/archive files, files over 25 MB, and declared/detected content-type mismatches covered by this specification.
- **SC-004**: Malware-scan gating tests confirm 100% of `pending` or `failed` content items remain unavailable to learning-set and file-access workflows until marked `clean`.
- **SC-005**: Learning-set tests reject 100% of missing, inactive, cross-tenant, unclean, or wrong-period content, questionnaire, academic-period, and student-profile references without partial persistence.
- **SC-006**: Grade and attendance tests reject 100% of invalid grade values, unsupported attendance statuses, inactive or closed periods, and invalid student-profile references.
- **SC-007**: Response-shape verification confirms every operation in this slice returns the documented success or error envelope for successful, validation, unauthorized, forbidden, tenant-mismatch, token-rejection or inactive-user/inactive-school, and not-found cases.
- **SC-008**: A teacher can upload clean content, create a questionnaire, publish a learning set to selected same-school students, and record grades and attendance in one active school tenant without exposing records from any other school.

## Assumptions

- `003-backend-school-admin` has already established tenant-scoped users, roles, permissions, academic years, academic periods, guardians, student profile references, response envelopes, tenant middleware, and school-admin backend patterns.
- Current OpenAPI contracts already publish list/create operations for teacher content, questionnaires, learning sets, grades, and attendance; implementation is restricted to those operations.
- Public teacher-content folder creation, listing, update, deletion, and movement are real product needs but are not approved for backend exposure until OpenAPI documents them.
- Student profile creation and enrollment workflows remain outside this slice except where existing `StudentProfile` identifiers are validated for learning set assignment, grade creation, and attendance creation.
- Classroom, course, section, group, roster, teacher assignment, correction, reporting, and student self-service workflows remain outside this slice.
- Direct teacher content download behavior is outside this backend slice unless OpenAPI documents a teacher-authorized download operation; student download remains part of the later student-facing slice.
