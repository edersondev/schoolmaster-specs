# Tasks: Backend Student Profile and Enrollment Management

**Input**: Design documents from `specs/006-backend-student-enrollment/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/backend-student-enrollment.md`, `quickstart.md`  
**Feature ID**: `006-backend-student-enrollment`

**Tests**: Required. This slice changes critical backend behavior for contract expansion, tenant isolation, student profile lifecycle management, guardian association atomicity, enrollment history, transfer boundaries, platform/school authorization separation, and response-contract behavior.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared foundations are in place.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Task can run in parallel with other marked tasks in the same phase when prerequisites are met.
- **[Story]**: User-story label for story phases only.
- Every task includes an exact target file path relative to the `schoolmaster-backend` repository root, except shared specification files, which use paths relative to the `schoolmaster-specs` repository root.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Expand the approved contract and align backend implementation with the student profile/enrollment specification before backend code changes.

- [X] T001 Create feature implementation notes with approved operation boundary and blocked scope in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T002 Add student profile and enrollment operation paths, operation IDs, parameters, request schemas, response schemas, and error responses in `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T003 Mirror the approved student profile and enrollment contract behavior in `api/openapi.yaml`
- [X] T004 Run Redocly contract validation for the aggregate and platform contracts and record the result in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T005 Confirm the approved operation IDs `listStudentProfiles`, `createStudentProfile`, `getStudentProfile`, `updateStudentProfileStatus`, and `transferStudentProfile` exist in the mounted OpenAPI contract and record the inventory in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T006 Confirm no undocumented student profile, enrollment, transfer, classroom, roster, guardian self-service, correction, bulk import, report, or frontend routes exist and document the route inventory in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T007 Record blocked contract gaps for frontend student administration, classroom/course/section/roster, teacher assignment, guardian self-service, academic-record correction, report lifecycle changes, bulk import, merge, anonymization, deletion, restore, purge, account lifecycle, and additional filters in `docs/implementation-notes/006-backend-student-enrollment.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared backend persistence, permissions, tenant, lifecycle, validation, authorization, and response infrastructure required by every student profile/enrollment story.

**Critical**: No user story work should begin until this phase is complete.

- [X] T008 Inspect existing backend migrations and models for `StudentProfile`, `Guardian`, guardian association pivot records, `School`, `User`, `AcademicPeriod`, `GradeRecord`, `AttendanceRecord`, `LearningSetAssignment`, and `ReportRun` tenant/status/UUID columns and document the inventory in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T009 Add only missing student profile lifecycle, enrollment history, guardian association, and transfer metadata persistence changes with tenant indexes and UUIDs in `database/migrations/2026_05_21_000001_add_student_profile_enrollment_management.php`
- [X] T010 [P] Seed baseline school-scoped student administration permission definitions in `database/seeders/PermissionSeeder.php`
- [X] T011 [P] Define shared student enrollment tenant assertion helpers in `app/Services/Concerns/AssertsStudentEnrollmentTenantScope.php`
- [X] T012 [P] Define shared student administration authorization helper methods in `app/Services/Concerns/AuthorizesStudentAdministration.php`
- [X] T013 [P] Define documented student profile list-query validation rules for pagination, filters, and sorting in `app/Services/StudentProfiles/StudentProfileListQuery.php`
- [X] T014 [P] Define documented student profile lifecycle transition validation rules in `app/Services/StudentProfiles/StudentProfileLifecycleRules.php`
- [X] T015 [P] Define documented guardian association validation rules for profile creation in `app/Services/StudentProfiles/GuardianAssociationValidator.php`
- [X] T016 [P] Define documented transfer validation rules for source and destination school behavior in `app/Services/StudentProfiles/StudentTransferValidator.php`
- [X] T017 [P] Implement or verify `StudentProfile` model UUIDs, school scope, user relationship, status helpers, guardian associations, and history relationship in `app/Models/StudentProfile.php`
- [X] T018 [P] Implement `EnrollmentHistory` model UUIDs, school scope, profile relationship, actor relationship, event type helpers, and immutable-history guard in `app/Models/EnrollmentHistory.php`
- [X] T019 [P] Implement or verify student guardian association model or pivot behavior with school scope and active status in `app/Models/GuardianAssociation.php`
- [X] T020 [P] Implement `StudentTransfer` model UUIDs, source school scope, source profile relationship, destination references, and actor relationship in `app/Models/StudentTransfer.php`
- [X] T021 [P] Add student enrollment model factories for tests in `database/factories/StudentEnrollmentFactory.php`
- [X] T022 Define the student profile route group with auth and tenant middleware only, without operation-specific route registration, in `routes/api.php`
- [X] T023 Register student profile and enrollment policies in `app/Providers/AuthServiceProvider.php`

**Checkpoint**: Persistence, permissions, tenant helpers, lifecycle rules, guardian validation, transfer validation, models, factories, and route boundaries are ready for story work.

---

## Phase 3: User Story 1 - Register Student Profiles (Priority: P1) MVP

**Goal**: A school administrator can create, list, and view same-school student profiles with atomic same-school guardian association validation.

**Independent Test**: Authenticate as a school administrator with an active school context, create a student profile with required identity and enrollment fields, retrieve it by identifier, list profiles, and verify only same-school profiles and guardian associations are visible.

### Tests for User Story 1

- [X] T024 [P] [US1] Add PHPUnit feature tests for `POST /api/v1/student-profiles` success, response shape, initial enrollment history, and same-school guardian association behavior in `tests/Feature/Api/V1/StudentProfileCreateTest.php`
- [X] T025 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/student-profiles` pagination, filters, sorting, same-school visibility, and response shape in `tests/Feature/Api/V1/StudentProfileListTest.php`
- [X] T026 [P] [US1] Add PHPUnit feature tests for `GET /api/v1/student-profiles/{studentProfileId}` success, inactive/transferred visibility, not-found behavior, and response shape in `tests/Feature/Api/V1/StudentProfileDetailTest.php`
- [X] T027 [P] [US1] Add PHPUnit feature tests for missing, inactive, mismatched, and unauthorized tenant context across create, list, and detail operations in `tests/Feature/Api/V1/StudentProfileTenantTest.php`
- [X] T028 [P] [US1] Add PHPUnit feature tests for duplicate same-school identifiers, invalid required fields, unsupported status values, undocumented fields, cross-tenant user references, and cross-tenant guardian references in `tests/Feature/Api/V1/StudentProfileValidationTest.php`
- [X] T029 [P] [US1] Add unit tests for atomic profile creation, guardian association validation, and initial enrollment history creation in `tests/Unit/Services/StudentProfileCreationTest.php`
- [X] T030 [P] [US1] Add OpenAPI response-shape regression coverage for `listStudentProfiles`, `createStudentProfile`, and `getStudentProfile` in `tests/Feature/Api/V1/StudentProfileContractTest.php`

### Implementation for User Story 1

- [X] T031 [P] [US1] Implement student profile creation input DTO in `app/DTOs/StudentProfiles/CreateStudentProfileData.php`
- [X] T032 [P] [US1] Implement student profile list request validation in `app/Http/Requests/StudentProfiles/ListStudentProfilesRequest.php`
- [X] T033 [P] [US1] Implement student profile create request validation in `app/Http/Requests/StudentProfiles/CreateStudentProfileRequest.php`
- [X] T034 [P] [US1] Implement student profile detail request validation in `app/Http/Requests/StudentProfiles/GetStudentProfileRequest.php`
- [X] T035 [P] [US1] Implement student profile summary resource using the published list shape in `app/Http/Resources/StudentProfiles/StudentProfileSummaryResource.php`
- [X] T036 [P] [US1] Implement student profile detail resource using the published detail shape in `app/Http/Resources/StudentProfiles/StudentProfileResource.php`
- [X] T037 [P] [US1] Implement enrollment history summary resource using the published history shape in `app/Http/Resources/StudentProfiles/EnrollmentHistoryResource.php`
- [X] T038 [P] [US1] Implement student profile authorization policy for list, create, and view behavior in `app/Policies/StudentProfilePolicy.php`
- [X] T039 [US1] Implement student profile listing service with tenant, permission, pagination, filters, sorting, and response-scope rules in `app/Services/StudentProfiles/StudentProfileListService.php`
- [X] T040 [US1] Implement student profile creation service with tenant, duplicate, user reference, guardian association, lifecycle status, and initial history rules in `app/Services/StudentProfiles/StudentProfileCreationService.php`
- [X] T041 [US1] Implement student profile detail service with tenant, permission, inactive/transferred visibility, and not-found behavior in `app/Services/StudentProfiles/StudentProfileDetailService.php`
- [X] T042 [US1] Implement student profile list, create, and detail controller actions in `app/Http/Controllers/Api/V1/StudentProfileController.php`
- [X] T043 [US1] Wire `listStudentProfiles`, `createStudentProfile`, and `getStudentProfile` routes in `routes/api.php`
- [X] T044 [US1] Update implementation notes with final create/list/detail operation IDs, route inventory, test commands, and blocked scope in `docs/implementation-notes/006-backend-student-enrollment.md`

**Checkpoint**: User Story 1 is independently functional and can be validated without lifecycle status updates or transfers.

---

## Phase 4: User Story 2 - Maintain Enrollment Status and History (Priority: P2)

**Goal**: A school administrator can change a same-school student profile's lifecycle status while preserving academic, guardian, assignment, report, and audit history.

**Independent Test**: Authenticate as a school administrator, mark a same-school student profile inactive according to documented lifecycle rules, and verify history is written atomically while active self-service and new operational participation are removed where prohibited.

### Tests for User Story 2

- [X] T045 [P] [US2] Add PHPUnit feature tests for `PATCH /api/v1/student-profiles/{studentProfileId}/status` active-to-inactive success, response shape, effective date, reason, and history write behavior in `tests/Feature/Api/V1/StudentProfileStatusUpdateTest.php`
- [X] T046 [P] [US2] Add PHPUnit feature tests for unsupported lifecycle transitions, transfer status rejection through the status endpoint, missing lifecycle fields, invalid effective dates, cross-tenant profiles, inactive tenant context, and undocumented fields in `tests/Feature/Api/V1/StudentProfileStatusValidationTest.php`
- [X] T047 [P] [US2] Add PHPUnit feature tests confirming historical grades, attendance, learning-set assignments, guardian associations, report references, and audit records remain retained after status changes in `tests/Feature/Api/V1/StudentProfileHistoryPreservationTest.php`
- [X] T048 [P] [US2] Add PHPUnit feature tests confirming inactive or transferred profiles cannot use active student self-view or new active workflow participation where prohibited in `tests/Feature/Api/V1/StudentProfileLifecycleAccessTest.php`
- [X] T049 [P] [US2] Add unit tests for non-transfer lifecycle transition rules, enrollment history event generation, and atomic rollback on lifecycle failure in `tests/Unit/Services/StudentProfileLifecycleRulesTest.php`
- [X] T050 [P] [US2] Add OpenAPI response-shape regression coverage for `updateStudentProfileStatus` in `tests/Feature/Api/V1/StudentProfileStatusContractTest.php`

### Implementation for User Story 2

- [X] T051 [P] [US2] Implement student profile status update input DTO in `app/DTOs/StudentProfiles/UpdateStudentProfileStatusData.php`
- [X] T052 [P] [US2] Implement student profile status update request validation in `app/Http/Requests/StudentProfiles/UpdateStudentProfileStatusRequest.php`
- [X] T053 [P] [US2] Extend student profile authorization policy for lifecycle status update behavior in `app/Policies/StudentProfilePolicy.php`
- [X] T054 [P] [US2] Implement lifecycle status response resource using the published lifecycle outcome shape in `app/Http/Resources/StudentProfiles/StudentProfileLifecycleResource.php`
- [X] T055 [US2] Implement non-transfer student profile lifecycle service with transition, effective-date, reason, actor, history, and atomic rollback rules in `app/Services/StudentProfiles/StudentProfileLifecycleService.php`
- [X] T056 [US2] Implement student profile status update controller action in `app/Http/Controllers/Api/V1/StudentProfileController.php`
- [X] T057 [US2] Wire `updateStudentProfileStatus` route in `routes/api.php`
- [X] T058 [US2] Update implementation notes with final lifecycle operation ID, transition matrix, history behavior, self-view impact, and test commands in `docs/implementation-notes/006-backend-student-enrollment.md`

**Checkpoint**: User Story 2 is independently functional after shared foundations and User Story 1 profile retrieval behavior.

---

## Phase 5: User Story 3 - Transfer Students Between School Contexts (Priority: P3)

**Goal**: An authorized administrator can record a source-school transfer without moving or exposing source-school academic history across tenants.

**Independent Test**: Transfer a same-school student profile using documented transfer inputs, verify source profile status and history, and verify no source-school grades, attendance, learning sets, private content, guardian links, reports, or output files are copied or exposed to another school.

### Tests for User Story 3

- [X] T059 [P] [US3] Add PHPUnit feature tests for `POST /api/v1/student-profiles/{studentProfileId}/transfer` source-school transfer success, response shape, source profile status, and enrollment history write behavior in `tests/Feature/Api/V1/StudentProfileTransferTest.php`
- [X] T060 [P] [US3] Add PHPUnit feature tests for destination-school profile creation or linking when documented and explicitly authorized in `tests/Feature/Api/V1/StudentProfileTransferDestinationTest.php`
- [X] T061 [P] [US3] Add PHPUnit feature tests for missing destination permission, inactive destination school, invalid source profile status, cross-tenant source profile, invalid effective date, unsupported transfer mode, and undocumented fields in `tests/Feature/Api/V1/StudentProfileTransferValidationTest.php`
- [X] T062 [P] [US3] Add PHPUnit feature tests confirming source-school grades, attendance, learning sets, private content, guardian links, report runs, and report outputs are not copied or exposed to destination school during transfer in `tests/Feature/Api/V1/StudentProfileTransferTenantIsolationTest.php`
- [X] T063 [P] [US3] Add unit tests for transfer validation, destination permission checks, source-profile lifecycle changes, and atomic rollback on transfer failure in `tests/Unit/Services/StudentProfileTransferServiceTest.php`
- [X] T064 [P] [US3] Add OpenAPI response-shape regression coverage for `transferStudentProfile` in `tests/Feature/Api/V1/StudentProfileTransferContractTest.php`

### Implementation for User Story 3

- [X] T065 [P] [US3] Implement student transfer input DTO in `app/DTOs/StudentProfiles/TransferStudentProfileData.php`
- [X] T066 [P] [US3] Implement student transfer request validation in `app/Http/Requests/StudentProfiles/TransferStudentProfileRequest.php`
- [X] T067 [P] [US3] Extend student profile authorization policy for transfer behavior and destination-school permission requirements in `app/Policies/StudentProfilePolicy.php`
- [X] T068 [P] [US3] Implement student transfer response resource using the published transfer outcome shape in `app/Http/Resources/StudentProfiles/StudentTransferResource.php`
- [X] T069 [US3] Implement student transfer service with source-school lifecycle, destination permission, destination profile, transfer metadata, no-copy, history, and atomic rollback rules in `app/Services/StudentProfiles/StudentProfileTransferService.php`
- [X] T070 [US3] Implement student transfer controller action in `app/Http/Controllers/Api/V1/StudentProfileController.php`
- [X] T071 [US3] Wire `transferStudentProfile` route in `routes/api.php`
- [X] T072 [US3] Update implementation notes with final transfer operation ID, destination permission behavior, tenant-isolation guarantees, no-copy evidence, and test commands in `docs/implementation-notes/006-backend-student-enrollment.md`

**Checkpoint**: User Story 3 is independently functional after shared foundations and profile lifecycle behavior.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final contract compliance, verification, documentation, and cleanup across all selected stories.

- [X] T073 [P] Add response-shape regression coverage for all five student profile/enrollment operation IDs, including success, validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, and not-found outcomes exactly as declared by OpenAPI, in `tests/Feature/Api/V1/StudentProfileEnrollmentResponseShapeTest.php`
- [X] T074 [P] Add validation-contract regression coverage for undocumented request fields, unsupported filters, unsupported sort values, invalid payload shapes, duplicate identifiers, invalid guardians, invalid lifecycle transitions, invalid effective dates, unsupported transfer modes, inactive references, and invalid cross-tenant references in `tests/Feature/Api/V1/StudentProfileEnrollmentValidationContractTest.php`
- [X] T075 [P] Add tenant-isolation regression coverage across profile creation, listing, detail, status update, transfer, guardian association, history lookup, and destination-school checks in `tests/Feature/Api/V1/StudentProfileEnrollmentTenantIsolationTest.php`
- [X] T076 [P] Add authorization matrix regression coverage for platform administrators, school administrators, teachers, students, inactive users, unauthorized school context, and cross-tenant school context across all five student profile/enrollment operation IDs in `tests/Feature/Api/V1/StudentProfileEnrollmentAuthorizationTest.php`
- [X] T077 [P] Add blocked-operation regression coverage for classroom/course/section/roster, teacher assignment, guardian self-service, correction workflows, report lifecycle changes, bulk import, merge, anonymization, permanent deletion, restore, purge, frontend-only behavior, and account lifecycle routes in `tests/Feature/Api/V1/StudentProfileEnrollmentBlockedOperationsTest.php`
- [X] T078 [P] Add end-to-end student profile/enrollment happy-path coverage from same-school profile creation through guardian association, list/detail retrieval, status change, transfer recording, and history preservation in `tests/Feature/Api/V1/StudentProfileEnrollmentHappyPathTest.php`
- [X] T079 Review implemented backend routes against the blocked-operation list in `routes/api.php`
- [X] T080 Run backend PHP syntax checks and record result in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T081 Run backend style checks and record result in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T082 Run backend PHPUnit suite and record result in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T083 Run Redocly validation and record result in `docs/implementation-notes/006-backend-student-enrollment.md`
- [X] T084 Update implementation notes with final operation IDs, test commands, tenant rules, guardian association behavior, lifecycle behavior, transfer behavior, history preservation behavior, and blocked follow-up contract gaps in `docs/implementation-notes/006-backend-student-enrollment.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks every user story.
- **Phase 3 US1**: Depends on Phase 2; recommended MVP.
- **Phase 4 US2**: Depends on Phase 2 and uses profile detail behavior from US1 for validation visibility.
- **Phase 5 US3**: Depends on Phase 2 and uses lifecycle behavior from US2 for source profile status changes.
- **Phase 6 Polish**: Depends on all selected user stories for the implementation increment.

### User Story Dependencies

- **US1 Register student profiles**: First recommended story because it establishes profile creation, listing, detail, same-school guardian associations, and initial history.
- **US2 Maintain enrollment status and history**: Can start after Phase 2, but should reuse US1 profile retrieval and resource behavior for consistent response shapes.
- **US3 Transfer students between school contexts**: Can start after Phase 2, but source-profile transfer should reuse lifecycle validation and history patterns established in US2.

### Within Each User Story

- Contract and backend tests should be written first and fail before implementation.
- Model and persistence verification precedes DTOs, requests, services, and controllers.
- Services own business rules before controllers wire routes.
- Policies and resources must be in place before marking a route complete.
- Routes must not expose blocked operations absent from OpenAPI.

## Parallel Opportunities

- T002 and T003 can run in parallel if maintainers coordinate schema naming and operation IDs.
- T010 through T021 can run in parallel after T008 and T009 establish the persistence baseline.
- Test files within each user story can be written in parallel.
- Request, resource, and policy files within a story can be implemented in parallel after model relationships are verified.
- US2 and US3 can run alongside each other after Phase 2 if implementers coordinate edits to `StudentProfilePolicy.php`, `StudentProfileController.php`, and `routes/api.php`.
- T073, T074, T075, T076, T077, and T078 can run in parallel during polish.

## Parallel Example: User Story 1

```text
Task: "T024 Add PHPUnit feature tests for POST /api/v1/student-profiles success, response shape, initial enrollment history, and same-school guardian association behavior in tests/Feature/Api/V1/StudentProfileCreateTest.php"
Task: "T025 Add PHPUnit feature tests for GET /api/v1/student-profiles pagination, filters, sorting, same-school visibility, and response shape in tests/Feature/Api/V1/StudentProfileListTest.php"
Task: "T026 Add PHPUnit feature tests for GET /api/v1/student-profiles/{studentProfileId} success, inactive/transferred visibility, not-found behavior, and response shape in tests/Feature/Api/V1/StudentProfileDetailTest.php"
Task: "T031 Implement student profile creation input DTO in app/DTOs/StudentProfiles/CreateStudentProfileData.php"
Task: "T032 Implement student profile list request validation in app/Http/Requests/StudentProfiles/ListStudentProfilesRequest.php"
Task: "T035 Implement student profile summary resource using the published list shape in app/Http/Resources/StudentProfiles/StudentProfileSummaryResource.php"
```

## Parallel Example: User Story 2

```text
Task: "T045 Add PHPUnit feature tests for PATCH /api/v1/student-profiles/{studentProfileId}/status active-to-inactive success, response shape, effective date, reason, and history write behavior in tests/Feature/Api/V1/StudentProfileStatusUpdateTest.php"
Task: "T046 Add PHPUnit feature tests for unsupported lifecycle transitions, missing lifecycle fields, invalid effective dates, cross-tenant profiles, inactive tenant context, and undocumented fields in tests/Feature/Api/V1/StudentProfileStatusValidationTest.php"
Task: "T051 Implement student profile status update input DTO in app/DTOs/StudentProfiles/UpdateStudentProfileStatusData.php"
Task: "T052 Implement student profile status update request validation in app/Http/Requests/StudentProfiles/UpdateStudentProfileStatusRequest.php"
Task: "T054 Implement lifecycle status response resource using the published lifecycle outcome shape in app/Http/Resources/StudentProfiles/StudentProfileLifecycleResource.php"
```

## Parallel Example: User Story 3

```text
Task: "T059 Add PHPUnit feature tests for POST /api/v1/student-profiles/{studentProfileId}/transfer source-school transfer success, response shape, source profile status, and enrollment history write behavior in tests/Feature/Api/V1/StudentProfileTransferTest.php"
Task: "T060 Add PHPUnit feature tests for destination-school profile creation or linking when documented and explicitly authorized in tests/Feature/Api/V1/StudentProfileTransferDestinationTest.php"
Task: "T065 Implement student transfer input DTO in app/DTOs/StudentProfiles/TransferStudentProfileData.php"
Task: "T066 Implement student transfer request validation in app/Http/Requests/StudentProfiles/TransferStudentProfileRequest.php"
Task: "T068 Implement student transfer response resource using the published transfer outcome shape in app/Http/Resources/StudentProfiles/StudentTransferResource.php"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 only.
3. Validate US1 independently with profile creation, guardian association atomicity, list/detail, tenant, duplicate, validation, and response-shape tests.
4. Stop before US2 or US3 if the backend needs an early review checkpoint.

### Incremental Delivery

1. Deliver US1 for student profile registration, listing, and detail.
2. Deliver US2 for enrollment status lifecycle and history preservation.
3. Deliver US3 for transfer recording and cross-tenant no-copy guarantees.
4. Run Phase 6 after the selected story set is complete.

### Scope Guardrails

- Do not add frontend implementation, classroom/course/section/group/roster workflows, teacher assignment workflows, guardian self-service, academic-record correction workflows, report lifecycle changes, bulk import, merge, anonymization, permanent deletion, restore, purge, account lifecycle behavior, billing, messaging, notifications, or undocumented APIs.
- Do not add request fields, response fields, filters, sort options, status values, transfer modes, status codes, or error envelopes absent from OpenAPI.
- Do not treat platform administrator access as an implicit school-scoped student administration permission bypass.
- Do not change `school_id` ownership for a transferred source profile.
- Do not copy source-school grades, attendance, learning sets, private content, guardian links, report runs, or report outputs to a destination school during transfer.
