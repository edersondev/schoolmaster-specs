# Tasks: Advanced Assessment and Content Types

**Input**: Design documents from `specs/014-advanced-assessment-content/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/backend-advanced-assessment-content.md, quickstart.md

**Tests**: Critical business flow tests are required. This feature changes REST contracts, tenant-scoped backend behavior, file handling, grading, reporting visibility, and audit coverage, so tasks include OpenAPI validation and PHPUnit coverage. Frontend tasks are intentionally excluded because frontend implementation is out of scope for this slice.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or has no dependency on incomplete tasks
- **[Story]**: User story label for story phases only
- Every task includes an exact repository-relative file path

## Phase 1: Setup (Shared Contract and Repository Preparation)

**Purpose**: Establish OpenAPI folders, backend feature folders, and traceability before backend implementation.

- [ ] T001 Create advanced assessment OpenAPI path directory in `schoolmaster-specs/api/paths/questionnaire-responses/`
- [ ] T002 Create student questionnaire response collection OpenAPI path file in `schoolmaster-specs/api/paths/student/questionnaire-responses.yaml`
- [ ] T003 Create student questionnaire response item OpenAPI path file in `schoolmaster-specs/api/paths/student/questionnaire-response.yaml`
- [ ] T004 Create advanced assessment schema directory in `schoolmaster-specs/api/components/schemas/assessments/`
- [ ] T005 Create advanced assessment response component directory in `schoolmaster-specs/api/components/responses/assessments/`
- [ ] T006 [P] Create backend assessment DTO directory in `schoolmaster-backend/app/DTOs/Assessment/`
- [ ] T007 [P] Create backend assessment service directory in `schoolmaster-backend/app/Services/Assessment/`
- [ ] T008 [P] Create backend assessment request directory in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/`
- [ ] T009 [P] Create backend assessment resource directory in `schoolmaster-backend/app/Http/Resources/Assessment/`
- [ ] T010 [P] Create backend assessment controller directory in `schoolmaster-backend/app/Http/Controllers/Api/V1/Assessment/`
- [ ] T011 [P] Create backend student assessment controller directory in `schoolmaster-backend/app/Http/Controllers/Api/V1/Student/`
- [ ] T012 Create backend assessment test directories in `schoolmaster-backend/tests/Feature/Assessment/` and `schoolmaster-backend/tests/Unit/Assessment/`
- [ ] T013 Document that frontend implementation is out of scope for this slice in `schoolmaster-specs/specs/014-advanced-assessment-content/quickstart.md`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core contract, persistence, tenant, authorization, audit, file, and route foundations that MUST be complete before any user story can be implemented.

**CRITICAL**: No user story work can begin until this phase is complete.

- [ ] T014 Expand questionnaire question type and advanced question schemas in `schoolmaster-specs/api/components/schemas/questionnaires/QuestionType.yaml`
- [ ] T015 Expand questionnaire question input and response schemas for `long_text` and `file_response` in `schoolmaster-specs/api/components/schemas/questionnaires/QuestionnaireQuestionInput.yaml`
- [ ] T016 Expand questionnaire create and update schemas for advanced answer schema validation in `schoolmaster-specs/api/components/schemas/questionnaires/QuestionnaireCreateRequest.yaml`
- [ ] T017 Add response attempt, answer, file attachment, grading, scan status, and summary schemas in `schoolmaster-specs/api/components/schemas/assessments/AssessmentResponseAttempt.yaml`
- [ ] T018 Add assessment validation, scan-pending, scan-failed, and unavailable-file response components in `schoolmaster-specs/api/components/responses/assessments/AssessmentResponses.yaml`
- [ ] T019 Add advanced assessment operation references to the aggregate contract in `schoolmaster-specs/api/openapi.yaml`
- [ ] T020 Mirror advanced assessment schemas and operations in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T021 Run contract-first Redocly validation with `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` and record results in `schoolmaster-specs/specs/014-advanced-assessment-content/quickstart.md`
- [ ] T022 Create assessment response attempt migration in `schoolmaster-backend/database/migrations/2026_06_11_000001_create_assessment_response_attempts_table.php`
- [ ] T023 Create assessment answers migration in `schoolmaster-backend/database/migrations/2026_06_11_000002_create_assessment_answers_table.php`
- [ ] T024 Create assessment file attachments migration in `schoolmaster-backend/database/migrations/2026_06_11_000003_create_assessment_file_attachments_table.php`
- [ ] T025 Create assessment grading outcomes migration in `schoolmaster-backend/database/migrations/2026_06_11_000004_create_assessment_grading_outcomes_table.php`
- [ ] T026 [P] Create AssessmentResponseAttempt model in `schoolmaster-backend/app/Models/AssessmentResponseAttempt.php`
- [ ] T027 [P] Create AssessmentAnswer model in `schoolmaster-backend/app/Models/AssessmentAnswer.php`
- [ ] T028 [P] Create AssessmentFileAttachment model in `schoolmaster-backend/app/Models/AssessmentFileAttachment.php`
- [ ] T029 [P] Create AssessmentGradingOutcome model in `schoolmaster-backend/app/Models/AssessmentGradingOutcome.php`
- [ ] T030 Add assessment response relationships to Questionnaire model in `schoolmaster-backend/app/Models/Questionnaire.php`
- [ ] T031 Add assessment response relationships to LearningSet model in `schoolmaster-backend/app/Models/LearningSet.php`
- [ ] T032 Add assessment response relationships to StudentProfile model in `schoolmaster-backend/app/Models/StudentProfile.php`
- [ ] T033 Create AssessmentPolicy authorization boundaries in `schoolmaster-backend/app/Policies/AssessmentPolicy.php`
- [ ] T034 Register assessment policy mappings in `schoolmaster-backend/app/Providers/AuthServiceProvider.php`
- [ ] T035 Create AssessmentActorContext DTO for school, actor, role, and authority resolution in `schoolmaster-backend/app/DTOs/Assessment/AssessmentActorContext.php`
- [ ] T036 Create AssessmentTenantScopeService for pre-lookup school resolution and tenant-safe denial behavior in `schoolmaster-backend/app/Services/Assessment/AssessmentTenantScopeService.php`
- [ ] T037 Create AssessmentAuditService for tenant-safe authoring, submission, file, scan, grading, download, report, denial, validation, and conflict events in `schoolmaster-backend/app/Services/Assessment/AssessmentAuditService.php`
- [ ] T038 Create AssessmentFileRuleService for category, size, filename, content-type, private storage, and scan-state validation in `schoolmaster-backend/app/Services/Assessment/AssessmentFileRuleService.php`
- [ ] T039 Create AssessmentFileScanOutcomeService for authorized malware-scan clean/failed transitions and scan outcome auditing in `schoolmaster-backend/app/Services/Assessment/AssessmentFileScanOutcomeService.php`
- [ ] T040 Create AssessmentResponseStateService for response, file, and grading state transitions in `schoolmaster-backend/app/Services/Assessment/AssessmentResponseStateService.php`
- [ ] T041 Create AssessmentQueryRepository for same-school response, file, grading, and report-safe lookup patterns in `schoolmaster-backend/app/Repositories/AssessmentQueryRepository.php`
- [ ] T042 Create AssessmentController shell without exposing undocumented operations in `schoolmaster-backend/app/Http/Controllers/Api/V1/Assessment/AssessmentController.php`
- [ ] T043 Create StudentAssessmentController shell without exposing undocumented operations in `schoolmaster-backend/app/Http/Controllers/Api/V1/Student/StudentAssessmentController.php`

**Checkpoint**: Foundation ready. User story implementation can now begin.

---

## Phase 3: User Story 1 - Author Advanced Assessment Questions (Priority: P1) MVP

**Goal**: Authorized teachers and school administrators can create and maintain questionnaires containing `long_text` and `file_response` questions while existing v1 question behavior and historical-meaning locks remain compatible.

**Independent Test**: Authenticate with an active permitted school context, create a questionnaire with existing v1 questions plus advanced questions, retrieve it, update editable fields while unused, attach it to a learning set, then verify unsupported question shapes, answer schemas, cross-tenant references, and historical-meaning edits are rejected.

### Tests for User Story 1

- [ ] T044 [P] [US1] Verify OpenAPI path coverage for expanded questionnaire create, update, and retrieve operations in `schoolmaster-specs/api/paths/questionnaires/index.yaml`
- [ ] T045 [P] [US1] Verify OpenAPI path coverage for questionnaire historical-meaning conflict responses in `schoolmaster-specs/api/paths/questionnaires/questionnaire.yaml`
- [ ] T046 [P] [US1] Verify platform OpenAPI parity for advanced questionnaire schemas in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T047 [P] [US1] Add PHPUnit feature tests for successful mixed v1 and advanced questionnaire authoring in `schoolmaster-backend/tests/Feature/Assessment/AdvancedQuestionnaireAuthoringTest.php`
- [ ] T048 [P] [US1] Add PHPUnit feature tests for unsupported question type, malformed schema, invalid grading rule, unsafe file rule, and cross-tenant authoring rejection in `schoolmaster-backend/tests/Feature/Assessment/AdvancedQuestionnaireValidationTest.php`
- [ ] T049 [P] [US1] Add PHPUnit feature tests for historical-meaning edit locks after publication, assignment, response submission, grading, and report generation in `schoolmaster-backend/tests/Feature/Assessment/AdvancedQuestionnaireLifecycleLockTest.php`
- [ ] T050 [P] [US1] Add PHPUnit unit tests for advanced questionnaire schema validation in `schoolmaster-backend/tests/Unit/Assessment/AssessmentQuestionSchemaValidatorTest.php`

### Implementation for User Story 1

- [ ] T051 [P] [US1] Implement AssessmentQuestionSchema DTO for v1, `long_text`, and `file_response` shapes in `schoolmaster-backend/app/DTOs/Assessment/AssessmentQuestionSchema.php`
- [ ] T052 [P] [US1] Implement AssessmentQuestionInput DTO for questionnaire authoring inputs in `schoolmaster-backend/app/DTOs/Assessment/AssessmentQuestionInput.php`
- [ ] T053 [P] [US1] Implement QuestionnaireAdvancedQuestionResource for actor-safe question metadata in `schoolmaster-backend/app/Http/Resources/Assessment/QuestionnaireAdvancedQuestionResource.php`
- [ ] T054 [US1] Implement advanced question schema validation for `long_text`, `file_response`, and legacy v1 compatibility in `schoolmaster-backend/app/Services/Assessment/AssessmentQuestionSchemaService.php`
- [ ] T055 [US1] Implement advanced questionnaire lifecycle lock checks for type, prompt, schema, grading rule, file rule, visibility, and sequence changes in `schoolmaster-backend/app/Services/Assessment/AssessmentQuestionnaireLifecycleService.php`
- [ ] T056 [US1] Extend questionnaire create request validation for advanced question schemas in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/StoreQuestionnaireRequest.php`
- [ ] T057 [US1] Extend questionnaire update request validation for advanced question schemas and lifecycle conflicts in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/UpdateQuestionnaireRequest.php`
- [ ] T058 [US1] Wire advanced questionnaire create, update, and retrieve behavior in `schoolmaster-backend/app/Http/Controllers/Api/V1/Assessment/QuestionnaireController.php`
- [ ] T059 [US1] Register or update questionnaire route bindings for advanced schema behavior in `schoolmaster-backend/routes/api.php`
- [ ] T060 [US1] Add tenant-safe authoring, validation failure, and lifecycle conflict audit writes in `schoolmaster-backend/app/Services/Assessment/AssessmentAuditService.php`

**Checkpoint**: User Story 1 is independently functional and is the MVP.

---

## Phase 4: User Story 2 - Submit Student Advanced Assessment Responses (Priority: P2)

**Goal**: Authenticated students can submit one valid response attempt for assigned same-school questionnaires, including long-text answers and private scan-gated file-response answers.

**Independent Test**: Authenticate as a student with an active same-school profile and assigned learning set, submit allowed long-text and file-response answers, verify pending file scans prevent teacher grading and reporting exposure until clean, and verify unassigned, cross-tenant, expired, malformed, blank, duplicate, or unsafe submissions are rejected.

### Tests for User Story 2

- [ ] T061 [P] [US2] Verify OpenAPI path coverage for `submitStudentQuestionnaireResponse` in `schoolmaster-specs/api/paths/student/questionnaire-responses.yaml`
- [ ] T062 [P] [US2] Verify OpenAPI schema coverage for student submission payloads, scan states, duplicate-attempt conflicts, and assignment denials in `schoolmaster-specs/api/components/schemas/assessments/AssessmentResponseAttempt.yaml`
- [ ] T063 [P] [US2] Verify platform OpenAPI parity for student response submission in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T064 [P] [US2] Add PHPUnit feature tests for successful assigned same-school long-text and file-response submission in `schoolmaster-backend/tests/Feature/Assessment/StudentAssessmentSubmissionTest.php`
- [ ] T065 [P] [US2] Add PHPUnit feature tests for unassigned, cross-tenant, inactive-profile, inactive-learning-set, inactive-questionnaire, after-due-date, and other-student submission rejection in `schoolmaster-backend/tests/Feature/Assessment/StudentAssessmentSubmissionAuthorizationTest.php`
- [ ] T066 [P] [US2] Add PHPUnit feature tests for duplicate attempt, malformed answer schema, partial persistence rollback, and concurrent submission conflict behavior in `schoolmaster-backend/tests/Feature/Assessment/StudentAssessmentSubmissionConflictTest.php`
- [ ] T067 [P] [US2] Add PHPUnit feature tests for long-text blank, whitespace-only, over-10000-character, invalid-encoding, unsafe-control-character, and plain-text handling behavior in `schoolmaster-backend/tests/Feature/Assessment/LongTextAnswerValidationTest.php`
- [ ] T068 [P] [US2] Add PHPUnit feature tests for file-response allowed categories, 25 MB limit, one-file rule, unsafe filename, executable/archive rejection, and declared/detected type mismatch in `schoolmaster-backend/tests/Feature/Assessment/FileResponseValidationTest.php`
- [ ] T069 [P] [US2] Add PHPUnit unit tests for assignment and due-date eligibility checks in `schoolmaster-backend/tests/Unit/Assessment/AssessmentSubmissionEligibilityServiceTest.php`

### Implementation for User Story 2

- [ ] T070 [P] [US2] Implement AssessmentResponseSubmissionData DTO in `schoolmaster-backend/app/DTOs/Assessment/AssessmentResponseSubmissionData.php`
- [ ] T071 [P] [US2] Implement AssessmentAnswerInput DTO for long-text and file-response answers in `schoolmaster-backend/app/DTOs/Assessment/AssessmentAnswerInput.php`
- [ ] T072 [P] [US2] Implement SubmitStudentQuestionnaireResponseRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/SubmitStudentQuestionnaireResponseRequest.php`
- [ ] T073 [P] [US2] Implement StudentAssessmentResponseResource with safe submission, grading, score, feedback, and file availability metadata in `schoolmaster-backend/app/Http/Resources/Assessment/StudentAssessmentResponseResource.php`
- [ ] T074 [US2] Implement student assignment, active profile, active learning set, active questionnaire, due-date, and single-attempt eligibility checks in `schoolmaster-backend/app/Services/Assessment/AssessmentSubmissionEligibilityService.php`
- [ ] T075 [US2] Implement atomic student response attempt persistence and rollback-on-validation-failure behavior in `schoolmaster-backend/app/Services/Assessment/AssessmentSubmissionService.php`
- [ ] T076 [US2] Implement long-text answer validation, invalid-encoding rejection, unsafe-control-character rejection, plain-text persistence, and non-executable output handling in `schoolmaster-backend/app/Services/Assessment/LongTextAnswerService.php`
- [ ] T077 [US2] Implement file-response private storage, metadata persistence, one-file enforcement, and pending scan initialization in `schoolmaster-backend/app/Services/Assessment/FileResponseSubmissionService.php`
- [ ] T078 [US2] Enforce pending and failed file scan invisibility for student submission outputs in `schoolmaster-backend/app/Services/Assessment/AssessmentResponseVisibilityService.php`
- [ ] T079 [US2] Wire student response submission controller action in `schoolmaster-backend/app/Http/Controllers/Api/V1/Student/StudentAssessmentController.php`
- [ ] T080 [US2] Register student questionnaire response submission route in `schoolmaster-backend/routes/api.php`
- [ ] T081 [US2] Add audit writes for submission, upload, validation failure, duplicate conflict, due-date conflict, and blocked cross-tenant submission in `schoolmaster-backend/app/Services/Assessment/AssessmentAuditService.php`

**Checkpoint**: User Stories 1 and 2 work independently with assignment, file, and scan gates.

---

## Phase 5: User Story 3 - Grade and Review Advanced Responses (Priority: P3)

**Goal**: Authorized teachers and school administrators can review same-school advanced responses, download only clean answer files, and record manual 0-100 grading or failed-scan zero/exempt outcomes.

**Independent Test**: Create assigned advanced responses, mark file submissions clean, review student answers as an authorized teacher or school administrator, apply manual grading from 0 to 100 points, verify auto-gradable legacy questions still grade according to existing rules, and verify unauthorized, unclean, cross-tenant, or stale grading attempts are rejected.

### Tests for User Story 3

- [ ] T082 [P] [US3] Verify OpenAPI path coverage for `listQuestionnaireResponses` and `getQuestionnaireResponse` in `schoolmaster-specs/api/paths/questionnaire-responses/index.yaml`
- [ ] T083 [P] [US3] Verify OpenAPI path coverage for `gradeQuestionnaireResponse` in `schoolmaster-specs/api/paths/questionnaire-responses/grading.yaml`
- [ ] T084 [P] [US3] Verify OpenAPI path coverage for `downloadQuestionnaireResponseFile` binary delivery and unsafe-file errors in `schoolmaster-specs/api/paths/questionnaire-responses/file-download.yaml`
- [ ] T085 [P] [US3] Verify platform OpenAPI parity for response review, grading, and answer-file download operations in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T086 [P] [US3] Add PHPUnit feature tests for teacher and school administrator response review authorization in `schoolmaster-backend/tests/Feature/Assessment/AssessmentResponseReviewTest.php`
- [ ] T087 [P] [US3] Add PHPUnit feature tests for clean answer-file download success, denied download attempts, and audit creation in `schoolmaster-backend/tests/Feature/Assessment/AssessmentFileDownloadTest.php`
- [ ] T088 [P] [US3] Add PHPUnit feature tests for manual 0-100 grading, invalid score rejection, legacy auto-grading compatibility, and stale state rejection in `schoolmaster-backend/tests/Feature/Assessment/AssessmentManualGradingTest.php`
- [ ] T089 [P] [US3] Add PHPUnit feature tests for pending-scan block, failed-scan zero-only grading, failed-scan exemption, and unclean file exposure denial in `schoolmaster-backend/tests/Feature/Assessment/AssessmentScanGatingTest.php`
- [ ] T090 [P] [US3] Add PHPUnit feature tests for unauthorized, cross-tenant, unassigned teacher, and inactive actor grading denial in `schoolmaster-backend/tests/Feature/Assessment/AssessmentGradingAuthorizationTest.php`
- [ ] T091 [P] [US3] Add PHPUnit unit tests for grading state transition rules in `schoolmaster-backend/tests/Unit/Assessment/AssessmentGradingStateServiceTest.php`

### Implementation for User Story 3

- [ ] T092 [P] [US3] Implement GradeAssessmentResponseRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/GradeAssessmentResponseRequest.php`
- [ ] T093 [P] [US3] Implement ListQuestionnaireResponsesRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/ListQuestionnaireResponsesRequest.php`
- [ ] T094 [P] [US3] Implement DownloadAssessmentFileRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/DownloadAssessmentFileRequest.php`
- [ ] T095 [P] [US3] Implement AssessmentResponseReviewResource with hidden storage paths, hidden answer keys, hidden private notes, and safe file metadata in `schoolmaster-backend/app/Http/Resources/Assessment/AssessmentResponseReviewResource.php`
- [ ] T096 [P] [US3] Implement AssessmentGradingResource with student-visible feedback boundary and private note exclusion from responses in `schoolmaster-backend/app/Http/Resources/Assessment/AssessmentGradingResource.php`
- [ ] T097 [US3] Implement teacher owner, teacher assignment, school administrator, and same-school grading authorization checks in `schoolmaster-backend/app/Services/Assessment/AssessmentReviewAuthorizationService.php`
- [ ] T098 [US3] Implement response review listing and detail retrieval with tenant-safe filtering in `schoolmaster-backend/app/Services/Assessment/AssessmentResponseReviewService.php`
- [ ] T099 [US3] Implement manual grading, legacy auto-grading compatibility checks, failed-scan zero/exempt handling, and state transitions in `schoolmaster-backend/app/Services/Assessment/AssessmentGradingService.php`
- [ ] T100 [US3] Implement clean answer-file lookup, private download delivery, unsafe-file denial, and successful/denied download auditing in `schoolmaster-backend/app/Services/Assessment/AssessmentFileDownloadService.php`
- [ ] T101 [US3] Wire review, grading, and file-download controller actions in `schoolmaster-backend/app/Http/Controllers/Api/V1/Assessment/AssessmentController.php`
- [ ] T102 [US3] Register questionnaire response review, grading, and file-download routes in `schoolmaster-backend/routes/api.php`
- [ ] T103 [US3] Add audit writes for review, grading, scan outcome, successful file download, denied file download, stale conflict, and unauthorized grading outcomes in `schoolmaster-backend/app/Services/Assessment/AssessmentAuditService.php`

**Checkpoint**: User Stories 1, 2, and 3 work independently with manual grading and file review boundaries.

---

## Phase 6: User Story 4 - Reflect Advanced Assessments in Student Views and Reports (Priority: P4)

**Goal**: Students, teachers, and reporting administrators see advanced assessment outcomes only through documented safe fields that preserve answer privacy, file security, and historical meaning.

**Independent Test**: Submit and grade advanced responses, then verify student learning timeline, student grade/assessment view, teacher review views, report catalog, custom report definitions, and generated reports expose only approved advanced assessment summary fields and reject hidden/private fields.

### Tests for User Story 4

- [ ] T104 [P] [US4] Verify OpenAPI path coverage for `getStudentQuestionnaireResponse` in `schoolmaster-specs/api/paths/student/questionnaire-response.yaml`
- [ ] T105 [P] [US4] Verify OpenAPI report catalog and report definition aggregate-only field coverage in `schoolmaster-specs/api/paths/report-catalog/index.yaml`
- [ ] T106 [P] [US4] Verify OpenAPI report output exclusion of raw answer text, file links, private metadata, feedback summaries, and grading notes in `schoolmaster-specs/api/components/schemas/reports/ReportOutput.yaml`
- [ ] T107 [P] [US4] Verify platform OpenAPI parity for student-safe and report-safe advanced assessment summaries in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T108 [P] [US4] Add PHPUnit feature tests for student own response summary status, grading status, score summary, feedback summary, and file availability metadata in `schoolmaster-backend/tests/Feature/Assessment/StudentAssessmentResponseViewTest.php`
- [ ] T109 [P] [US4] Add PHPUnit feature tests for report catalog and report definition allowed aggregate fields and rejected private advanced assessment fields in `schoolmaster-backend/tests/Feature/Assessment/AdvancedAssessmentReportCatalogTest.php`
- [ ] T110 [P] [US4] Add PHPUnit feature tests for generated report aggregate-only output and exclusion of answer text, files, links, metadata, feedback summaries, and private notes in `schoolmaster-backend/tests/Feature/Assessment/AdvancedAssessmentReportOutputTest.php`
- [ ] T111 [P] [US4] Add PHPUnit feature tests for guardian, platform, support, unauthorized, unclean-file, and cross-tenant detail access denial in `schoolmaster-backend/tests/Feature/Assessment/AdvancedAssessmentVisibilityBoundaryTest.php`

### Implementation for User Story 4

- [ ] T112 [P] [US4] Implement GetStudentQuestionnaireResponseRequest validation in `schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/GetStudentQuestionnaireResponseRequest.php`
- [ ] T113 [P] [US4] Implement AssessmentReportSummaryResource for counts, completion status, grading status, and score summaries only in `schoolmaster-backend/app/Http/Resources/Assessment/AssessmentReportSummaryResource.php`
- [ ] T114 [US4] Implement student own-response retrieval and safe response shaping in `schoolmaster-backend/app/Services/Assessment/StudentAssessmentResponseViewService.php`
- [ ] T115 [US4] Implement report catalog aggregate-field registration and hidden-field rejection in `schoolmaster-backend/app/Services/Assessment/AssessmentReportCatalogService.php`
- [ ] T116 [US4] Implement generated report aggregate-only projection for advanced assessments in `schoolmaster-backend/app/Services/Assessment/AssessmentReportProjectionService.php`
- [ ] T117 [US4] Enforce guardian, platform, support, raw answer, file link, private metadata, feedback summary, and private note exclusions in `schoolmaster-backend/app/Services/Assessment/AssessmentResponseVisibilityService.php`
- [ ] T118 [US4] Wire student own-response retrieval in `schoolmaster-backend/app/Http/Controllers/Api/V1/Student/StudentAssessmentController.php`
- [ ] T119 [US4] Register student questionnaire response retrieval route in `schoolmaster-backend/routes/api.php`
- [ ] T120 [US4] Add audit writes for report field access, blocked report fields, denied detail access, and cross-tenant visibility blocks in `schoolmaster-backend/app/Services/Assessment/AssessmentAuditService.php`

**Checkpoint**: All user stories are independently functional with student-safe and report-safe visibility boundaries.

---

## Phase 7: Polish & Cross-Cutting Validation

**Purpose**: Contract validation, regression coverage, traceability, security review, and readiness checks across the full slice.

- [ ] T121 [P] Run final Redocly validation with `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1` and record results in `schoolmaster-specs/specs/014-advanced-assessment-content/quickstart.md`
- [ ] T122 [P] Review operation-to-route traceability for every advanced assessment operation in `schoolmaster-specs/specs/014-advanced-assessment-content/contracts/backend-advanced-assessment-content.md`
- [ ] T123 [P] Review aggregate OpenAPI publication for undocumented question types, answer fields, report fields, file access paths, states, filters, or authorization exceptions in `schoolmaster-specs/api/openapi.yaml`
- [ ] T124 [P] Review platform-local OpenAPI mirror for operation and schema parity in `schoolmaster-specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [ ] T125 [P] Add PHPUnit feature tests for inactive school, inactive user, inactive teacher assignment, and inactive academic period denial across authoring, submission, review, grading, file download, and reports in `schoolmaster-backend/tests/Feature/Assessment/AdvancedAssessmentInactiveBoundaryTest.php`
- [ ] T126 Run backend PHPUnit suite with `docker exec schoolmaster-backend-app-1 php artisan test` and record results in `schoolmaster-specs/specs/014-advanced-assessment-content/quickstart.md`
- [ ] T127 [P] Review tenant-safe audit payloads for raw answer text, file contents, private paths, credentials, hidden answer keys, feedback text, private notes, full payloads, and unauthorized cross-tenant details in `schoolmaster-backend/app/Services/Assessment/AssessmentAuditService.php`
- [ ] T128 [P] Review response resources for student, teacher, report, guardian, platform, and support redaction boundaries in `schoolmaster-backend/app/Http/Resources/Assessment/`
- [ ] T129 [P] Review file-response storage and malware-scan gating behavior against teacher-content patterns in `schoolmaster-backend/app/Services/Assessment/AssessmentFileRuleService.php`
- [ ] T130 [P] Review existing v1 questionnaire, learning-set, student self-view, report lifecycle, and platform support regression compatibility in `schoolmaster-backend/tests/Feature/Assessment/AdvancedAssessmentCompatibilityTest.php`
- [ ] T131 Update backend implementation notes for advanced assessment and content types in `schoolmaster-backend/README.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Phase 1 Setup**: No dependencies; starts immediately.
- **Phase 2 Foundation**: Depends on Phase 1; blocks all user stories and includes contract-first Redocly validation before backend route exposure.
- **Phase 3 US1**: Depends on Phase 2; MVP.
- **Phase 4 US2**: Depends on Phase 2 and the approved question schemas from US1 for real end-to-end submissions.
- **Phase 5 US3**: Depends on Phase 2 and needs submitted response attempts from US2 for full grading and file-review verification.
- **Phase 6 US4**: Depends on submitted and graded responses from US2 and US3 for complete student/report verification.
- **Phase 7 Polish**: Depends on selected user stories and contract changes being complete.

### User Story Dependencies

- **US1 Author Advanced Assessment Questions (P1)**: Can start after Foundation; required MVP and schema source for later stories.
- **US2 Submit Student Advanced Assessment Responses (P2)**: Can start after Foundation but should consume the advanced question schemas from US1.
- **US3 Grade and Review Advanced Responses (P3)**: Can start after Foundation but needs response attempts and files from US2 for full verification.
- **US4 Reflect Advanced Assessments in Student Views and Reports (P4)**: Can start after Foundation but needs submitted and graded responses from US2/US3 for full verification.

### Within Each User Story

- OpenAPI path and schema verification tasks come before backend route exposure.
- PHPUnit feature and unit tests are written before implementation and should fail for missing behavior.
- DTOs and request validation come before services.
- Services come before controllers, resources, and route registration.
- Audit and redaction checks are added before marking a story complete.

---

## Parallel Execution Examples

### User Story 1

```bash
# Contract and test tasks can run together:
Task: "Verify OpenAPI path coverage for expanded questionnaire create, update, and retrieve operations in schoolmaster-specs/api/paths/questionnaires/index.yaml"
Task: "Add PHPUnit feature tests for successful mixed v1 and advanced questionnaire authoring in schoolmaster-backend/tests/Feature/Assessment/AdvancedQuestionnaireAuthoringTest.php"
Task: "Add PHPUnit unit tests for advanced questionnaire schema validation in schoolmaster-backend/tests/Unit/Assessment/AssessmentQuestionSchemaValidatorTest.php"

# DTO/resource work can run together after contract shape is settled:
Task: "Implement AssessmentQuestionSchema DTO for v1, long_text, and file_response shapes in schoolmaster-backend/app/DTOs/Assessment/AssessmentQuestionSchema.php"
Task: "Implement QuestionnaireAdvancedQuestionResource for actor-safe question metadata in schoolmaster-backend/app/Http/Resources/Assessment/QuestionnaireAdvancedQuestionResource.php"
```

### User Story 2

```bash
# Submission validation tests can run together:
Task: "Add PHPUnit feature tests for unassigned, cross-tenant, inactive-profile, inactive-learning-set, inactive-questionnaire, after-due-date, and other-student submission rejection in schoolmaster-backend/tests/Feature/Assessment/StudentAssessmentSubmissionAuthorizationTest.php"
Task: "Add PHPUnit feature tests for long-text blank, whitespace-only, and over-10000-character answer rejection in schoolmaster-backend/tests/Feature/Assessment/LongTextAnswerValidationTest.php"
Task: "Add PHPUnit feature tests for file-response allowed categories, 25 MB limit, one-file rule, unsafe filename, executable/archive rejection, and declared/detected type mismatch in schoolmaster-backend/tests/Feature/Assessment/FileResponseValidationTest.php"

# Independent input and resource classes can run together:
Task: "Implement AssessmentResponseSubmissionData DTO in schoolmaster-backend/app/DTOs/Assessment/AssessmentResponseSubmissionData.php"
Task: "Implement SubmitStudentQuestionnaireResponseRequest validation in schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/SubmitStudentQuestionnaireResponseRequest.php"
Task: "Implement StudentAssessmentResponseResource with safe submission, grading, score, feedback, and file availability metadata in schoolmaster-backend/app/Http/Resources/Assessment/StudentAssessmentResponseResource.php"
```

### User Story 3

```bash
# Review, grading, and file-download tests can run together:
Task: "Add PHPUnit feature tests for teacher and school administrator response review authorization in schoolmaster-backend/tests/Feature/Assessment/AssessmentResponseReviewTest.php"
Task: "Add PHPUnit feature tests for clean answer-file download success, denied download attempts, and audit creation in schoolmaster-backend/tests/Feature/Assessment/AssessmentFileDownloadTest.php"
Task: "Add PHPUnit feature tests for manual 0-100 grading, invalid score rejection, legacy auto-grading compatibility, and stale state rejection in schoolmaster-backend/tests/Feature/Assessment/AssessmentManualGradingTest.php"

# HTTP-layer classes can run together:
Task: "Implement GradeAssessmentResponseRequest validation in schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/GradeAssessmentResponseRequest.php"
Task: "Implement ListQuestionnaireResponsesRequest validation in schoolmaster-backend/app/Http/Requests/Api/V1/Assessment/ListQuestionnaireResponsesRequest.php"
Task: "Implement AssessmentResponseReviewResource with hidden storage paths, hidden answer keys, hidden private notes, and safe file metadata in schoolmaster-backend/app/Http/Resources/Assessment/AssessmentResponseReviewResource.php"
```

### User Story 4

```bash
# Visibility tests can run together:
Task: "Add PHPUnit feature tests for student own response summary status, grading status, score summary, feedback summary, and file availability metadata in schoolmaster-backend/tests/Feature/Assessment/StudentAssessmentResponseViewTest.php"
Task: "Add PHPUnit feature tests for report catalog and report definition allowed aggregate fields and rejected private advanced assessment fields in schoolmaster-backend/tests/Feature/Assessment/AdvancedAssessmentReportCatalogTest.php"
Task: "Add PHPUnit feature tests for guardian, platform, support, unauthorized, unclean-file, and cross-tenant detail access denial in schoolmaster-backend/tests/Feature/Assessment/AdvancedAssessmentVisibilityBoundaryTest.php"

# Report and student-safe services can run together:
Task: "Implement student own-response retrieval and safe response shaping in schoolmaster-backend/app/Services/Assessment/StudentAssessmentResponseViewService.php"
Task: "Implement report catalog aggregate-field registration and hidden-field rejection in schoolmaster-backend/app/Services/Assessment/AssessmentReportCatalogService.php"
Task: "Implement generated report aggregate-only projection for advanced assessments in schoolmaster-backend/app/Services/Assessment/AssessmentReportProjectionService.php"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 setup.
2. Complete Phase 2 foundation, including OpenAPI expansion and Redocly validation.
3. Complete Phase 3 User Story 1.
4. Validate US1 independently with mixed v1 and advanced questionnaire authoring, unsupported schema rejection, tenant rejection, and historical-meaning lock checks.
5. Stop before submission, grading, or report exposure unless MVP scope is accepted.

### Incremental Delivery

1. Deliver US1 to make advanced question authoring contract-safe.
2. Deliver US2 to accept one student response attempt with file scan gates.
3. Deliver US3 to enable authorized review, clean file download, and manual grading.
4. Deliver US4 to expose only student-safe and report-safe summaries.
5. Run Phase 7 validation before implementation readiness sign-off.

### Team Parallel Strategy

1. One engineer owns OpenAPI schema and path work in `schoolmaster-specs/api/`.
2. One engineer owns backend persistence, models, tenant scope, and audit foundations.
3. Story engineers work on tests first, then DTO/request/resource/service/controller layers by story.
4. Security review focuses on file-response validation, scan gating, tenant isolation, report exclusions, and audit redaction before merge.
