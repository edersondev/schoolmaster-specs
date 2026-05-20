# Tasks: Backend Student and Reporting Foundation

**Input**: Design documents from `specs/005-backend-student-reporting/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/backend-student-reporting.md`, `quickstart.md`  
**Feature ID**: `005-backend-student-reporting`

**Tests**: Required. This slice changes critical backend behavior for tenant isolation, student ownership, private file authorization, scan-gated content download, school-scoped report permissions, asynchronous report generation, 90-day output expiry, no download-time regeneration, and response/binary contract behavior.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared foundations are in place.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Task can run in parallel with other marked tasks in the same phase when prerequisites are met.
- **[Story]**: User-story label for story phases only.
- Every task includes an exact target file path relative to the `schoolmaster-backend` repository root unless it explicitly references a shared specification file.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Align backend implementation with the approved student/reporting specification and published contract before code changes.

- [ ] T001 Create feature implementation notes with approved operation IDs and blocked scope in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T002 [P] Verify the backend specs mount references `specs/specs/005-backend-student-reporting/quickstart.md` in `AGENTS.md`
- [ ] T003 Run Redocly contract validation for the aggregate and platform contracts and record the result in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T004 Confirm all seven approved student/reporting operation IDs exist in the mounted OpenAPI contract and record the inventory in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T005 Confirm no undocumented student, report, report retry, report deletion, guardian self-service, classroom/course/section/roster, or teacher correction routes exist and document the route inventory in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T006 Record blocked contract gaps for frontend delivery, custom reports, platform-wide reports, report lifecycle mutations, automatic expired-output regeneration, and student profile management in `docs/implementation-notes/005-backend-student-reporting.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared backend persistence, permissions, tenant, report output, authorization, and response infrastructure required by every student/reporting story.

**Critical**: No user story work should begin until this phase is complete.

- [ ] T007 Inspect existing backend migrations for `StudentProfile`, `LearningSet`, `LearningSetEntry`, `LearningSetAssignment`, `TeacherContentItem`, `GradeRecord`, `AttendanceRecord`, `ReportRun`, and `ReportOutput` UUID/status/tenant/soft-delete columns and document the inventory in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T008 Add only missing report-run and report-output persistence changes with tenant indexes, expiry metadata, and status fields in `database/migrations/2026_05_20_000001_create_report_run_output_tables.php`
- [ ] T009 [P] Seed baseline student self-view and school report permission definitions in `database/seeders/PermissionSeeder.php`
- [ ] T010 [P] Configure private tenant-scoped generated report output storage without public URLs in `config/filesystems.php`
- [ ] T011 [P] Define shared student self-view authorization helper methods in `app/Services/Concerns/AuthorizesStudentSelfView.php`
- [ ] T012 [P] Define shared school report authorization helper methods in `app/Services/Concerns/AuthorizesSchoolReports.php`
- [ ] T013 [P] Define shared student/report tenant context assertion helpers in `app/Services/Concerns/AssertsSchoolTenantScope.php`
- [ ] T014 [P] Define documented student list-query validation rules for academic-period and pagination filters in `app/Services/StudentSelfView/StudentSelfViewListQuery.php`
- [ ] T015 [P] Define documented report filter validation rules for report type, academic period, optional filters, date ranges, and format values in `app/Services/Reports/ReportFilterValidator.php`
- [ ] T016 [P] Define report output availability and 90-day expiry helper in `app/Services/Reports/ReportOutputAvailability.php`
- [ ] T017 [P] Add student/reporting model factories for tests in `database/factories/StudentReportingFactory.php`
- [ ] T018 Define the student and report route groups with auth and tenant middleware only, without operation-specific route registration, in `routes/api.php`
- [ ] T019 Register report generation queue/job configuration dependencies needed by this slice in `config/queue.php`

**Checkpoint**: Persistence, permissions, private report storage, tenant helpers, student ownership helpers, report filter validation, and route boundaries are ready for story work.

---

## Phase 3: User Story 1 - Review Student Learning Timeline (Priority: P1) MVP

**Goal**: A student can list assigned learning sets for a selected academic period, with entries ordered for student consumption and clean-content availability metadata only.

**Independent Test**: Authenticate as a student with one active same-school `StudentProfile`, request the learning timeline for an academic period with assigned learning sets, and verify only the student's own assigned learning sets are returned, ordered by publish date with entries ordered by teacher-defined sequence.

### Tests for User Story 1

- [ ] T020 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/student/learning-sets` success, pagination, required academic-period filtering, and response shape in `tests/Feature/Api/V1/StudentLearningTimelineTest.php`
- [ ] T021 [P] [US1] Add PHPUnit feature tests for missing, inactive, mismatched, and unauthorized tenant context on `GET /api/v1/student/learning-sets` in `tests/Feature/Api/V1/StudentLearningTimelineTenantTest.php`
- [ ] T022 [P] [US1] Add PHPUnit feature tests for other-student, inactive-profile, inactive-assignment, cross-tenant, and unassigned learning-set rejection in `tests/Feature/Api/V1/StudentLearningTimelineOwnershipTest.php`
- [ ] T023 [P] [US1] Add unit tests for publish-date and entry-sequence ordering in `tests/Unit/Services/StudentLearningTimelineOrderingTest.php`
- [ ] T024 [P] [US1] Add unit tests for student-visible teacher content metadata and scan-status availability flags in `tests/Unit/Services/StudentLearningTimelineContentMetadataTest.php`

### Implementation for User Story 1

- [ ] T025 [P] [US1] Verify `StudentProfile` model user relationship, school scope, and active-status helpers in `app/Models/StudentProfile.php`
- [ ] T026 [P] [US1] Verify `LearningSet` model published status, academic-period relationship, and school scope in `app/Models/LearningSet.php`
- [ ] T027 [P] [US1] Verify `LearningSetEntry` model sequence helpers and content/questionnaire reference behavior in `app/Models/LearningSetEntry.php`
- [ ] T028 [P] [US1] Verify `LearningSetAssignment` model student relationship, active status, and school scope in `app/Models/LearningSetAssignment.php`
- [ ] T029 [P] [US1] Implement student learning-set list request validation in `app/Http/Requests/StudentSelfView/ListStudentLearningSetsRequest.php`
- [ ] T030 [P] [US1] Implement student teacher content metadata resource in `app/Http/Resources/Student/TeacherContentStudentMetadataResource.php`
- [ ] T031 [P] [US1] Implement student learning-set entry resource in `app/Http/Resources/Student/StudentLearningSetEntryResource.php`
- [ ] T032 [P] [US1] Implement student learning-set timeline resource in `app/Http/Resources/Student/StudentLearningSetTimelineResource.php`
- [ ] T033 [P] [US1] Implement student learning-set authorization policy in `app/Policies/StudentLearningSetPolicy.php`
- [ ] T034 [US1] Implement student learning timeline query service with tenant, profile, assignment, period, ordering, and content-metadata rules in `app/Services/StudentSelfView/StudentLearningTimelineService.php`
- [ ] T035 [US1] Implement student learning-set list controller action in `app/Http/Controllers/Api/V1/StudentLearningSetController.php`
- [ ] T036 [US1] Wire `listStudentLearningSets` route in `routes/api.php`

**Checkpoint**: User Story 1 is independently functional and can be validated without student download, grade, attendance, or reporting workflows.

---

## Phase 4: User Story 2 - Access Own Academic Records and Authorized Content (Priority: P2)

**Goal**: A student can list their own grade and attendance records and download only assigned clean teacher content from the resolved school.

**Independent Test**: Authenticate as a student, list grades and attendance for their own profile, download one assigned clean content item, and verify attempts to access cross-tenant, unassigned, inactive, pending-scan, failed-scan, or other-student records are rejected.

### Tests for User Story 2

- [ ] T037 [P] [US2] Add PHPUnit feature tests for `GET /api/v1/student/grades` success, optional academic-period filtering, pagination, and response shape in `tests/Feature/Api/V1/StudentGradeSelfViewTest.php`
- [ ] T038 [P] [US2] Add PHPUnit feature tests for `GET /api/v1/student/attendance` success, optional academic-period filtering, pagination, and response shape in `tests/Feature/Api/V1/StudentAttendanceSelfViewTest.php`
- [ ] T039 [P] [US2] Add PHPUnit feature tests for `GET /api/v1/student/teacher-content/{contentItemId}/download` success and binary response behavior in `tests/Feature/Api/V1/StudentTeacherContentDownloadTest.php`
- [ ] T040 [P] [US2] Add PHPUnit feature tests for other-student, inactive-profile, cross-tenant, invalid academic-period, unsupported filter, and tenant-mismatch rejection across student academic record endpoints in `tests/Feature/Api/V1/StudentAcademicRecordAuthorizationTest.php`
- [ ] T041 [P] [US2] Add PHPUnit feature tests for unassigned, inactive, missing, cross-tenant, pending-scan, failed-scan, and unavailable content download rejection in `tests/Feature/Api/V1/StudentTeacherContentDownloadAuthorizationTest.php`
- [ ] T042 [P] [US2] Add unit tests for assignment-based student content download authorization in `tests/Unit/Services/StudentTeacherContentDownloadAuthorizationTest.php`

### Implementation for User Story 2

- [ ] T043 [P] [US2] Verify `GradeRecord` model student, academic-period, recorder, status, and school-scope relationships in `app/Models/GradeRecord.php`
- [ ] T044 [P] [US2] Verify `AttendanceRecord` model student, academic-period, recorder, status, and school-scope relationships in `app/Models/AttendanceRecord.php`
- [ ] T045 [P] [US2] Verify `TeacherContentItem` model scan-status, private storage, operational status, and school-scope helpers in `app/Models/TeacherContentItem.php`
- [ ] T046 [P] [US2] Implement student grade list request validation in `app/Http/Requests/StudentSelfView/ListStudentGradesRequest.php`
- [ ] T047 [P] [US2] Implement student attendance list request validation in `app/Http/Requests/StudentSelfView/ListStudentAttendanceRequest.php`
- [ ] T048 [P] [US2] Implement student teacher content download request validation in `app/Http/Requests/StudentSelfView/DownloadStudentTeacherContentRequest.php`
- [ ] T049 [P] [US2] Implement student grade response resource using the published `GradeRecord` shape in `app/Http/Resources/Student/StudentGradeRecordResource.php`
- [ ] T050 [P] [US2] Implement student attendance response resource using the published `AttendanceRecord` shape in `app/Http/Resources/Student/StudentAttendanceRecordResource.php`
- [ ] T051 [P] [US2] Implement student grade authorization policy in `app/Policies/StudentGradeRecordPolicy.php`
- [ ] T052 [P] [US2] Implement student attendance authorization policy in `app/Policies/StudentAttendanceRecordPolicy.php`
- [ ] T053 [P] [US2] Implement student teacher content download authorization policy in `app/Policies/StudentTeacherContentPolicy.php`
- [ ] T054 [US2] Implement student grade self-view query service with tenant, profile, academic-period, pagination, and response-scope rules in `app/Services/StudentSelfView/StudentGradeSelfViewService.php`
- [ ] T055 [US2] Implement student attendance self-view query service with tenant, profile, academic-period, pagination, and response-scope rules in `app/Services/StudentSelfView/StudentAttendanceSelfViewService.php`
- [ ] T056 [US2] Implement student teacher content download service with assignment, scan-clean, status, private file, and no-storage-detail exposure rules in `app/Services/StudentSelfView/StudentTeacherContentDownloadService.php`
- [ ] T057 [US2] Implement student grade list controller action in `app/Http/Controllers/Api/V1/StudentGradeController.php`
- [ ] T058 [US2] Implement student attendance list controller action in `app/Http/Controllers/Api/V1/StudentAttendanceController.php`
- [ ] T059 [US2] Implement student teacher content download controller action in `app/Http/Controllers/Api/V1/StudentTeacherContentController.php`
- [ ] T060 [US2] Wire `listStudentGrades`, `listStudentAttendance`, and `downloadStudentTeacherContent` routes in `routes/api.php`

**Checkpoint**: User Story 2 is independently functional after shared foundations and existing teacher workflow data.

---

## Phase 5: User Story 3 - Request and Retrieve School Reports (Priority: P3)

**Goal**: A school-scoped administrator can request asynchronous reports, list report runs, download generated PDF/CSV outputs while unexpired, and request a new `ReportRun` after expiry.

**Independent Test**: Authenticate as a school administrator, request each launch-scope report type for an academic period, list report runs, simulate generated outputs, download PDF and CSV files before expiry, verify expired outputs return the documented expired-output response, and create a new `ReportRun` with the same filters for regeneration.

### Tests for User Story 3

- [ ] T061 [P] [US3] Add PHPUnit feature tests for `GET /api/v1/reports` success, pagination, report-type filtering, tenant scoping, and response shape in `tests/Feature/Api/V1/ReportRunListTest.php`
- [ ] T062 [P] [US3] Add PHPUnit feature tests for `POST /api/v1/reports` accepted response, launch-scope report types, filter validation, and same-school references in `tests/Feature/Api/V1/ReportRequestTest.php`
- [ ] T063 [P] [US3] Add PHPUnit feature tests for `GET /api/v1/reports/{reportRunId}/download` PDF/CSV success and binary response behavior in `tests/Feature/Api/V1/ReportDownloadTest.php`
- [ ] T064 [P] [US3] Add PHPUnit feature tests for expired-output `410` behavior and no download-time regeneration in `tests/Feature/Api/V1/ReportOutputExpiryTest.php`
- [ ] T065 [P] [US3] Add PHPUnit feature tests for report permission, platform/school separation, cross-tenant run, cross-tenant output, invalid format, failed run, inactive run, and not-found cases in `tests/Feature/Api/V1/ReportAuthorizationTest.php`
- [ ] T066 [P] [US3] Add unit tests for report filter relevance, date-range validation, and same-school academic-period/student/user references in `tests/Unit/Services/ReportFilterValidationTest.php`
- [ ] T067 [P] [US3] Add unit tests for report output availability, 90-day expiry calculation, and metadata retention in `tests/Unit/Services/ReportOutputAvailabilityTest.php`

### Implementation for User Story 3

- [ ] T068 [P] [US3] Implement or verify `ReportRun` model UUIDs, school scope, requester relationship, status helpers, filter summary casting, output formats, generated timestamp, expiry timestamp, and output availability helpers in `app/Models/ReportRun.php`
- [ ] T069 [P] [US3] Implement or verify `ReportOutput` model UUIDs, school scope, report-run relationship, format, private storage path, expiry timestamp, and status helpers in `app/Models/ReportOutput.php`
- [ ] T070 [US3] Implement report request input DTO in `app/DTOs/Reports/RequestReportData.php`
- [ ] T071 [P] [US3] Implement report request validation in `app/Http/Requests/Reports/RequestReportRequest.php`
- [ ] T072 [P] [US3] Implement report list request validation in `app/Http/Requests/Reports/ListReportsRequest.php`
- [ ] T073 [P] [US3] Implement report download request validation in `app/Http/Requests/Reports/DownloadReportRequest.php`
- [ ] T074 [P] [US3] Implement report run response resource using the published `ReportRun` shape in `app/Http/Resources/Reports/ReportRunResource.php`
- [ ] T075 [P] [US3] Implement report authorization policy for list, request, and download behavior in `app/Policies/ReportRunPolicy.php`
- [ ] T076 [US3] Implement report listing service with tenant, permission, pagination, and report-type filter rules in `app/Services/Reports/ReportRunListService.php`
- [ ] T077 [US3] Implement asynchronous report request service that creates `ReportRun` records without waiting for generated files in `app/Services/Reports/ReportRequestService.php`
- [ ] T078 [US3] Implement report generation job boundary for launch-scope PDF/CSV output generation in `app/Jobs/GenerateReportRunOutputs.php`
- [ ] T079 [US3] Implement report output writer for private tenant-scoped generated files in `app/Services/Reports/ReportOutputWriter.php`
- [ ] T080 [US3] Implement report download service with same-school, generated, format, private file, and unexpired-output rules in `app/Services/Reports/ReportDownloadService.php`
- [ ] T081 [US3] Implement report output retention cleanup behavior that expires files while preserving `ReportRun` metadata in `app/Services/Reports/ReportOutputRetentionService.php`
- [ ] T082 [US3] Implement report list, request, and download controller actions in `app/Http/Controllers/Api/V1/ReportController.php`
- [ ] T083 [US3] Wire `listReports`, `requestReport`, and `downloadReport` routes in `routes/api.php`

**Checkpoint**: User Story 3 is independently functional after shared foundations and school-scoped report permissions.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final contract compliance, verification, documentation, and cleanup across all selected stories.

- [ ] T084 [P] Add response-shape regression coverage for all seven student/reporting operation IDs, including success, binary, validation where applicable, unauthorized, tenant-mismatch, not-found, and expired-output outcomes exactly as declared by OpenAPI, in `tests/Feature/Api/V1/StudentReportingResponseShapeTest.php`
- [ ] T085 [P] Add validation-contract regression coverage for undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, invalid report types, invalid report formats, invalid date ranges, and invalid references across all seven operation IDs in `tests/Feature/Api/V1/StudentReportingValidationContractTest.php`
- [ ] T086 [P] Add tenant-isolation regression coverage across student learning sets, student downloads, student grades, student attendance, report requests, report listing, and report downloads in `tests/Feature/Api/V1/StudentReportingTenantIsolationTest.php`
- [ ] T087 [P] Add blocked-operation regression coverage for frontend-only behavior, teacher writes, custom reports, platform-wide reports, report deletion, report retry, guardian self-service, student profile management, and classroom/course/section/roster routes in `tests/Feature/Api/V1/StudentReportingBlockedOperationsTest.php`
- [ ] T088 [P] Add end-to-end student/reporting happy-path coverage from assigned clean content through student timeline, content download, academic record self-view, report request, generated output download, expiry, and new report-run regeneration in `tests/Feature/Api/V1/StudentReportingHappyPathTest.php`
- [ ] T089 Review implemented backend routes against the blocked-operation list in `routes/api.php`
- [ ] T090 Run backend PHP syntax checks and record result in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T091 Run backend style checks and record result in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T092 Run backend PHPUnit suite and record result in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T093 Run Redocly validation and record result in `docs/implementation-notes/005-backend-student-reporting.md`
- [ ] T094 Update implementation notes with final operation IDs, test commands, student ownership behavior, content download behavior, report retention behavior, regeneration behavior, and blocked follow-up contract gaps in `docs/implementation-notes/005-backend-student-reporting.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks every user story.
- **Phase 3 US1**: Depends on Phase 2; recommended MVP.
- **Phase 4 US2**: Depends on Phase 2 and existing teacher workflow data; may integrate with US1 assignment/content rules but remains independently testable.
- **Phase 5 US3**: Depends on Phase 2 and existing school-admin/teacher workflow data for report fixtures.
- **Phase 6 Polish**: Depends on all selected user stories for the implementation increment.

### User Story Dependencies

- **US1 Review student learning timeline**: First recommended story because it validates student identity, assignment ownership, academic-period filtering, and student response shaping.
- **US2 Access own academic records and authorized content**: Can start after Phase 2, but content download uses the same assignment and scan-gating concepts proven by US1.
- **US3 Request and retrieve school reports**: Can start after Phase 2 if report implementers avoid route-file conflicts; report data depends on existing school-admin and teacher workflow records for meaningful fixtures.

### Within Each User Story

- Tests should be written first and fail before implementation.
- Model and persistence verification precedes DTOs, requests, services, and controllers.
- Services own business rules before controllers wire routes.
- Policies and resources must be in place before marking a route complete.
- Routes must not expose blocked operations absent from OpenAPI.

## Parallel Opportunities

- T002 can run in parallel with the implementation-notes setup sequence.
- T009 through T017 can run in parallel after T007 and T008 establish the persistence baseline.
- Test files within each user story can be written in parallel.
- Request, resource, and policy files within a story can be implemented in parallel after model relationships are verified.
- US2 and US3 can run alongside each other after Phase 2 if separate implementers avoid editing `routes/api.php` at the same time.
- T084, T085, T086, T087, and T088 can run in parallel during polish.

## Parallel Example: User Story 1

```text
Task: "T020 Add PHPUnit feature tests for GET /api/v1/student/learning-sets in tests/Feature/Api/V1/StudentLearningTimelineTest.php"
Task: "T021 Add PHPUnit tenant tests for GET /api/v1/student/learning-sets in tests/Feature/Api/V1/StudentLearningTimelineTenantTest.php"
Task: "T022 Add PHPUnit ownership tests for student learning timeline in tests/Feature/Api/V1/StudentLearningTimelineOwnershipTest.php"
Task: "T023 Add unit tests for timeline ordering in tests/Unit/Services/StudentLearningTimelineOrderingTest.php"
Task: "T024 Add unit tests for student content metadata in tests/Unit/Services/StudentLearningTimelineContentMetadataTest.php"
```

## Parallel Example: User Story 2

```text
Task: "T037 Add PHPUnit feature tests for student grades in tests/Feature/Api/V1/StudentGradeSelfViewTest.php"
Task: "T038 Add PHPUnit feature tests for student attendance in tests/Feature/Api/V1/StudentAttendanceSelfViewTest.php"
Task: "T039 Add PHPUnit feature tests for student content download in tests/Feature/Api/V1/StudentTeacherContentDownloadTest.php"
Task: "T046 Implement student grade list request validation in app/Http/Requests/StudentSelfView/ListStudentGradesRequest.php"
Task: "T053 Implement student teacher content download authorization policy in app/Policies/StudentTeacherContentPolicy.php"
```

## Parallel Example: User Story 3

```text
Task: "T061 Add PHPUnit feature tests for report listing in tests/Feature/Api/V1/ReportRunListTest.php"
Task: "T062 Add PHPUnit feature tests for report requests in tests/Feature/Api/V1/ReportRequestTest.php"
Task: "T063 Add PHPUnit feature tests for report downloads in tests/Feature/Api/V1/ReportDownloadTest.php"
Task: "T068 Implement ReportRun model in app/Models/ReportRun.php"
Task: "T069 Implement ReportOutput model in app/Models/ReportOutput.php"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 only.
3. Validate US1 independently with student timeline, tenant, ownership, academic-period, ordering, and response-shape tests.
4. Stop before US2 or US3 if the backend needs an early review checkpoint.

### Incremental Delivery

1. Deliver US1 for student learning timeline.
2. Deliver US2 for student grade/attendance self-view and authorized content download.
3. Deliver US3 for school reports, report downloads, expiry, and regeneration through a new `ReportRun`.
4. Run Phase 6 after the selected story set is complete.

### Scope Guardrails

- Do not add frontend implementation, teacher workflow writes, custom report definitions, platform-wide reports, report deletion, report retry, correction workflows, guardian self-service, student profile management, classroom, course, section, group, roster, or teacher assignment workflows.
- Do not add request fields, response fields, filters, sort options, report formats, status codes, or error envelopes absent from OpenAPI.
- Do not treat platform administrator access as an implicit school-scoped student self-view or report permission bypass.
- Do not expose private teacher content files unless same-school, assigned to the student, operationally available, and malware scan status is `clean`.
- Do not expose generated report files from public storage, after output expiry, or outside the resolved school context.
- Do not regenerate expired report outputs during download; create a new `ReportRun` through `requestReport` instead.
