# Tasks: Backend Guardian Self-Service

**Input**: Design documents from `specs/011-guardian-self-service/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/

**Tests**: Critical business flow tests are required. This feature changes REST contracts and backend behavior, so include OpenAPI validation and PHPUnit feature/unit coverage. Frontend implementation is explicitly deferred by the plan, so Vitest tasks are not included in this slice.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Every task includes an exact file path

## Path Conventions

- Specification and contract paths are in `schoolmaster-specs/`.
- Backend implementation paths are in `schoolmaster-backend/`.
- Frontend implementation is out of scope for this feature.

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish the contract-first backend slice and shared implementation structure.

- [ ] T001 Update guardian self-service OpenAPI root path references in `schoolmaster-specs/api/openapi.yaml`
- [ ] T002 [P] Add guardian self-service operation stubs in `schoolmaster-specs/api/paths/guardian/students/index.yaml`
- [ ] T003 [P] Add guardian self-service operation stubs in `schoolmaster-specs/api/paths/guardian/students/student.yaml`
- [ ] T004 [P] Add guardian academic summary operation stub in `schoolmaster-specs/api/paths/guardian/students/academics.yaml`
- [ ] T005 [P] Add guardian contact operation stub in `schoolmaster-specs/api/paths/guardian/students/contacts.yaml`
- [ ] T006 Add guardian self-service schemas and response components under `schoolmaster-specs/api/components/schemas/guardians/`
- [ ] T007 Mirror approved guardian self-service contract surface in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T008 Create backend feature directories in `schoolmaster-backend/app/Services/GuardianSelfService/`
- [ ] T009 [P] Create backend DTO directory in `schoolmaster-backend/app/DTOs/GuardianSelfService/`
- [ ] T010 [P] Create backend request directory in `schoolmaster-backend/app/Http/Requests/Api/V1/Guardian/`
- [ ] T011 [P] Create backend resource directory in `schoolmaster-backend/app/Http/Resources/Guardian/`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core data, authorization, tenancy, and audit foundations that MUST be complete before any user story can be implemented.

**Critical**: No user story work can begin until this phase is complete.

- [ ] T012 Create guardian-user link migration in `schoolmaster-backend/database/migrations/2026_06_04_000001_create_guardian_user_links_table.php`
- [ ] T013 Create GuardianUserLink model with UUID and school ownership in `schoolmaster-backend/app/Models/GuardianUserLink.php`
- [ ] T014 [P] Add guardian user-link relationships to `schoolmaster-backend/app/Models/Guardian.php`
- [ ] T015 [P] Add guardian user-link relationships to `schoolmaster-backend/app/Models/User.php`
- [ ] T016 [P] Add guardian association relationships to `schoolmaster-backend/app/Models/StudentProfile.php`
- [ ] T017 Create GuardianSelfServicePolicy for read-only access boundaries in `schoolmaster-backend/app/Policies/GuardianSelfServicePolicy.php`
- [ ] T018 Register GuardianSelfServicePolicy in `schoolmaster-backend/app/Providers/AuthServiceProvider.php`
- [ ] T019 Create guardian actor resolution DTO in `schoolmaster-backend/app/DTOs/GuardianSelfService/GuardianActorContext.php`
- [ ] T020 Create guardian target student DTO in `schoolmaster-backend/app/DTOs/GuardianSelfService/GuardianStudentTarget.php`
- [ ] T021 Create GuardianAccessResolver service for tenant, user-link, and association proof in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAccessResolver.php`
- [ ] T022 Create GuardianVisibilityService for approved field visibility rules in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianVisibilityService.php`
- [ ] T023 Create GuardianAuditService for tenant-safe access and denial audit events in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAuditService.php`
- [ ] T024 Create GuardianSelfServiceController shell in `schoolmaster-backend/app/Http/Controllers/Api/V1/Guardian/GuardianSelfServiceController.php`
- [ ] T025 Register guardian self-service routes in `schoolmaster-backend/routes/api.php`
- [ ] T026 Create common guardian self-service unauthorized/not-found response helpers in `schoolmaster-backend/app/Http/Resources/Guardian/GuardianErrorResource.php`
- [ ] T027 Add guardian self-service factory support in `schoolmaster-backend/database/factories/GuardianUserLinkFactory.php`
- [ ] T028 [P] Add PHPUnit feature tests for missing, mismatched, inactive, and unauthorized guardian tenant context across list, detail, academics, and contacts in `schoolmaster-backend/tests/Feature/GuardianSelfService/GuardianTenantContextTest.php`
- [ ] T077 [P] Add school-admin guardian-user-link OpenAPI contract coverage in `schoolmaster-specs/api/paths/guardians/user-links.yaml`
- [ ] T078 [P] Add guardian-user-link admin schemas and envelopes under `schoolmaster-specs/api/components/schemas/guardians/`
- [ ] T079 [P] Add PHPUnit feature tests for school-admin guardian-user-link create and deactivate in `schoolmaster-backend/tests/Feature/Api/V1/GuardianUserLinkManagementTest.php`
- [ ] T080 [P] Implement guardian-user-link create validation in `schoolmaster-backend/app/Http/Requests/Api/V1/CreateGuardianUserLinkRequest.php`
- [ ] T081 [P] Implement guardian-user-link deactivate validation in `schoolmaster-backend/app/Http/Requests/Api/V1/DeactivateGuardianUserLinkRequest.php`
- [ ] T082 [P] Implement guardian-user-link response shape in `schoolmaster-backend/app/Http/Resources/Api/V1/GuardianUserLinkResource.php`
- [ ] T083 [P] Extend guardian authorization policy for guardian-user-link lifecycle in `schoolmaster-backend/app/Policies/GuardianPolicy.php`
- [ ] T084 Implement school-admin guardian-user-link create and deactivate service in `schoolmaster-backend/app/Services/Guardians/GuardianUserLinkService.php`
- [ ] T085 Implement school-admin guardian-user-link controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/GuardianController.php`
- [ ] T086 Wire school-admin guardian-user-link routes in `schoolmaster-backend/routes/api.php`

**Checkpoint**: Foundation ready - user story implementation can now begin.

---

## Phase 3: User Story 1 - View Linked Student Overview (Priority: P1) MVP

**Goal**: Authenticated guardians can list only active same-school students with active school-approved associations and retrieve a limited student profile/enrollment summary.

**Independent Test**: Authenticate as a guardian with an explicit active guardian-user link and active same-school association, list guardian-visible students, retrieve one linked student summary, and verify inactive, unassociated, missing, and cross-tenant targets do not disclose protected records.

### Tests for User Story 1

- [ ] T029 [P] [US1] Add OpenAPI contract coverage for `listGuardianStudents` and `getGuardianStudent` in `schoolmaster-specs/api/paths/guardian/students/index.yaml` and `schoolmaster-specs/api/paths/guardian/students/student.yaml`
- [ ] T030 [P] [US1] Add PHPUnit feature tests for guardian student listing in `schoolmaster-backend/tests/Feature/GuardianSelfService/ListGuardianStudentsTest.php`
- [ ] T031 [P] [US1] Add PHPUnit feature tests for guardian student detail not-found non-enumeration in `schoolmaster-backend/tests/Feature/GuardianSelfService/GetGuardianStudentTest.php`
- [ ] T032 [P] [US1] Add PHPUnit unit tests for GuardianAccessResolver proof checks in `schoolmaster-backend/tests/Unit/GuardianSelfService/GuardianAccessResolverTest.php`
- [ ] T033 [P] [US1] Add PHPUnit feature tests for source-school and destination-school guardian student transfer visibility in `schoolmaster-backend/tests/Feature/GuardianSelfService/GuardianStudentTransferVisibilityTest.php`

### Implementation for User Story 1

- [ ] T034 [P] [US1] Implement ListGuardianStudentsRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Guardian/ListGuardianStudentsRequest.php`
- [ ] T035 [P] [US1] Implement GetGuardianStudentRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Guardian/GetGuardianStudentRequest.php`
- [ ] T036 [P] [US1] Implement GuardianStudentListResource in `schoolmaster-backend/app/Http/Resources/Guardian/GuardianStudentListResource.php`
- [ ] T037 [P] [US1] Implement GuardianStudentResource limited summary fields in `schoolmaster-backend/app/Http/Resources/Guardian/GuardianStudentResource.php`
- [ ] T038 [US1] Implement student list query logic in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianStudentService.php`
- [ ] T039 [US1] Implement target-specific same not-found behavior in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAccessResolver.php`
- [ ] T040 [US1] Wire list and detail controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/Guardian/GuardianSelfServiceController.php`
- [ ] T041 [US1] Add tenant-safe audit writes for student list and detail reads in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAuditService.php`
- [ ] T042 [US1] Verify guardian student routes map to approved OpenAPI operation IDs in `schoolmaster-backend/routes/api.php`

**Checkpoint**: User Story 1 is fully functional and independently testable.

---

## Phase 4: User Story 2 - Review Permitted Academic Information (Priority: P2)

**Goal**: Authenticated guardians can view summary-only academic information for a permitted student and explicit same-school academic period without receiving detailed rows, correction history, teacher content, questionnaire answers, reports, or student self-view capabilities.

**Independent Test**: Authenticate as a guardian linked to one active student, request that student's academic summary for an explicit same-school academic period, and verify only approved summaries are visible while detailed records and private metadata are redacted.

### Tests for User Story 2

- [ ] T043 [P] [US2] Add OpenAPI contract coverage for `getGuardianStudentAcademics` in `schoolmaster-specs/api/paths/guardian/students/academics.yaml`
- [ ] T044 [P] [US2] Add PHPUnit feature tests for guardian academic summary success in `schoolmaster-backend/tests/Feature/GuardianSelfService/GetGuardianStudentAcademicsTest.php`
- [ ] T045 [P] [US2] Add PHPUnit feature tests for academic-period validation failures in `schoolmaster-backend/tests/Feature/GuardianSelfService/GetGuardianStudentAcademicsPeriodTest.php`
- [ ] T046 [P] [US2] Add PHPUnit unit tests for academic summary aggregation redaction in `schoolmaster-backend/tests/Unit/GuardianSelfService/GuardianAcademicSummaryServiceTest.php`
- [ ] T047 [P] [US2] Add PHPUnit feature tests for guardian academic summary transfer isolation in `schoolmaster-backend/tests/Feature/GuardianSelfService/GetGuardianStudentAcademicsTransferTest.php`

### Implementation for User Story 2

- [ ] T048 [P] [US2] Implement GetGuardianStudentAcademicsRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Guardian/GetGuardianStudentAcademicsRequest.php`
- [ ] T049 [P] [US2] Implement GuardianAcademicSummaryResource in `schoolmaster-backend/app/Http/Resources/Guardian/GuardianAcademicSummaryResource.php`
- [ ] T050 [P] [US2] Create academic summary query DTO in `schoolmaster-backend/app/DTOs/GuardianSelfService/GuardianAcademicSummaryQuery.php`
- [ ] T051 [US2] Implement grade summary aggregation in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAcademicSummaryService.php`
- [ ] T052 [US2] Implement attendance summary aggregation in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAcademicSummaryService.php`
- [ ] T053 [US2] Implement learning-set progress/status summary aggregation in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAcademicSummaryService.php`
- [ ] T054 [US2] Enforce detailed grade, attendance, correction, teacher content, questionnaire, and report redaction in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianVisibilityService.php`
- [ ] T055 [US2] Wire academic summary controller action in `schoolmaster-backend/app/Http/Controllers/Api/V1/Guardian/GuardianSelfServiceController.php`
- [ ] T056 [US2] Add tenant-safe audit writes for academic summary reads and denials in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAuditService.php`

**Checkpoint**: User Story 2 is fully functional and independently testable.

---

## Phase 5: User Story 3 - Review Contact and School-Approved Relationship Information (Priority: P3)

**Goal**: Authenticated guardians can view only their own contact fields, school-approved relationship labels, and the student's primary school-approved contact details for permitted students.

**Independent Test**: Authenticate as a guardian, retrieve the contact view for an approved student association, and verify other guardians, non-primary student contacts, emergency handling details, school-only notes, and unapproved contacts are hidden.

### Tests for User Story 3

- [ ] T057 [P] [US3] Add OpenAPI contract coverage for `getGuardianStudentContacts` in `schoolmaster-specs/api/paths/guardian/students/contacts.yaml`
- [ ] T058 [P] [US3] Add PHPUnit feature tests for guardian contact view success in `schoolmaster-backend/tests/Feature/GuardianSelfService/GetGuardianStudentContactsTest.php`
- [ ] T059 [P] [US3] Add PHPUnit feature tests for contact-field redaction in `schoolmaster-backend/tests/Feature/GuardianSelfService/GetGuardianStudentContactsRedactionTest.php`
- [ ] T060 [P] [US3] Add PHPUnit unit tests for ContactVisibilityRule behavior in `schoolmaster-backend/tests/Unit/GuardianSelfService/GuardianContactVisibilityTest.php`

### Implementation for User Story 3

- [ ] T061 [P] [US3] Implement GetGuardianStudentContactsRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Guardian/GetGuardianStudentContactsRequest.php`
- [ ] T062 [P] [US3] Implement GuardianStudentContactsResource in `schoolmaster-backend/app/Http/Resources/Guardian/GuardianStudentContactsResource.php`
- [ ] T063 [P] [US3] Create contact view query DTO in `schoolmaster-backend/app/DTOs/GuardianSelfService/GuardianContactViewQuery.php`
- [ ] T064 [US3] Implement contact visibility field selection in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianContactService.php`
- [ ] T065 [US3] Enforce other-guardian, non-primary contact, emergency detail, and school-only note redaction in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianVisibilityService.php`
- [ ] T066 [US3] Wire contact controller action in `schoolmaster-backend/app/Http/Controllers/Api/V1/Guardian/GuardianSelfServiceController.php`
- [ ] T067 [US3] Add tenant-safe audit writes for contact view reads and denials in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAuditService.php`

**Checkpoint**: User Story 3 is fully functional and independently testable.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Contract validation, regression hardening, documentation, and release readiness across all stories.

- [ ] T068 Add PHPUnit feature tests for guardian audit events covering allowed reads, denied access, blocked cross-tenant attempts, and payload redaction in `schoolmaster-backend/tests/Feature/GuardianSelfService/GuardianAuditEventsTest.php`
- [ ] T069 [P] Run Redocly lint for aggregate contract and record results in `schoolmaster-specs/specs/011-guardian-self-service/quickstart.md`
- [ ] T070 [P] Run Redocly lint for platform contract and record results in `schoolmaster-specs/specs/011-guardian-self-service/quickstart.md`
- [ ] T071 Run backend PHPUnit suite and record results in `schoolmaster-specs/specs/011-guardian-self-service/quickstart.md`
- [ ] T072 [P] Review OpenAPI route-to-operation traceability in `schoolmaster-specs/specs/011-guardian-self-service/contracts/backend-guardian-self-service.md`
- [ ] T073 [P] Review backend route-to-operation traceability in `schoolmaster-backend/routes/api.php`
- [ ] T074 [P] Review audit payload redaction across all stories in `schoolmaster-backend/app/Services/GuardianSelfService/GuardianAuditService.php`
- [ ] T075 [P] Review response resource field redaction across all stories in `schoolmaster-backend/app/Http/Resources/Guardian/`
- [ ] T076 Update backend implementation notes for guardian self-service in `schoolmaster-backend/README.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies.
- **Phase 2 Foundational**: Depends on Phase 1 and blocks all user stories.
- **Phase 3 US1**: Depends on Phase 2. This is the MVP.
- **Phase 4 US2**: Depends on Phase 2 and may reuse US1 access-resolution behavior.
- **Phase 5 US3**: Depends on Phase 2 and may reuse US1 access-resolution behavior.
- **Phase 6 Polish**: Depends on all desired user stories being complete.

### User Story Dependencies

- **US1 View Linked Student Overview**: No dependency on other stories after foundation.
- **US2 Review Permitted Academic Information**: Depends on shared access resolution from foundation; can be developed after Phase 2, but integration is simpler after US1.
- **US3 Review Contact and Relationship Information**: Depends on shared access resolution from foundation; can be developed after Phase 2, but integration is simpler after US1.

### Within Each User Story

- Contract and PHPUnit tests first.
- Request validation and resources before controller wiring.
- Services before controller actions.
- Audit writes integrated with each story's service path.
- Route-to-operation verification before story checkpoint.

---

## Parallel Opportunities

- Setup path stubs T002-T005 can run in parallel.
- Backend directory setup T009-T011 can run in parallel.
- Foundational model relationship tasks T014-T016 can run in parallel after T013.
- Foundational tenant-context test T028 can run in parallel with model and factory scaffolding after route signatures are known.
- Foundational guardian-user-link admin tasks T077-T083 can run in parallel once the contract shape is fixed.
- US1 tests T029-T033 can run in parallel.
- US1 request/resource tasks T034-T037 can run in parallel.
- US2 tests T043-T047 can run in parallel.
- US2 request/resource/DTO tasks T048-T050 can run in parallel.
- US3 tests T057-T060 can run in parallel.
- US3 request/resource/DTO tasks T061-T063 can run in parallel.
- Polish review tasks T069-T070 and T072-T075 can run in parallel.

---

## Parallel Example: User Story 1

```bash
# Launch US1 tests together:
Task: "T029 OpenAPI contract coverage for listGuardianStudents and getGuardianStudent"
Task: "T030 PHPUnit feature tests for guardian student listing"
Task: "T031 PHPUnit feature tests for guardian student detail non-enumeration"
Task: "T032 PHPUnit unit tests for GuardianAccessResolver"
Task: "T033 PHPUnit feature tests for guardian student transfer visibility"

# Launch US1 HTTP shaping work together:
Task: "T034 ListGuardianStudentsRequest"
Task: "T035 GetGuardianStudentRequest"
Task: "T036 GuardianStudentListResource"
Task: "T037 GuardianStudentResource"
```

## Parallel Example: User Story 2

```bash
# Launch US2 tests together:
Task: "T043 OpenAPI contract coverage for getGuardianStudentAcademics"
Task: "T044 PHPUnit feature tests for academic summary success"
Task: "T045 PHPUnit feature tests for academic-period validation"
Task: "T046 PHPUnit unit tests for academic summary redaction"
Task: "T047 PHPUnit feature tests for academic transfer isolation"

# Launch US2 support classes together:
Task: "T048 GetGuardianStudentAcademicsRequest"
Task: "T049 GuardianAcademicSummaryResource"
Task: "T050 GuardianAcademicSummaryQuery"
```

## Parallel Example: User Story 3

```bash
# Launch US3 tests together:
Task: "T057 OpenAPI contract coverage for getGuardianStudentContacts"
Task: "T058 PHPUnit feature tests for contact view success"
Task: "T059 PHPUnit feature tests for contact-field redaction"
Task: "T060 PHPUnit unit tests for contact visibility"

# Launch US3 support classes together:
Task: "T061 GetGuardianStudentContactsRequest"
Task: "T062 GuardianStudentContactsResource"
Task: "T063 GuardianContactViewQuery"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup.
2. Complete Phase 2 foundational access, tenant, policy, route, and audit scaffolding.
3. Complete Phase 3 User Story 1.
4. Stop and validate linked student listing and detail summary independently.
5. Do not expose US2 or US3 routes until their OpenAPI and tests are complete.

### Incremental Delivery

1. Setup + Foundation: contract paths, guardian-user link proof, admin guardian-user-link provisioning, access resolver, visibility service, audit service.
2. US1: linked student list/detail MVP.
3. US2: academic summary-only view.
4. US3: limited contact view.
5. Polish: contract validation, backend test run, audit and resource redaction review.

### Parallel Team Strategy

1. Complete Phase 1 and Phase 2 together.
2. After foundation:
   - Developer A: US1 linked student list/detail.
   - Developer B: US2 academic summary.
   - Developer C: US3 contact view.
3. Keep each story independently testable and do not merge undocumented route behavior.

## Notes

- [P] tasks are parallelizable because they touch different files and do not depend on incomplete tasks in the same phase.
- All backend routes must map to approved OpenAPI operation IDs before implementation is considered complete.
- Frontend work is intentionally absent from this task list because the feature plan defers frontend implementation.
