# Tasks: Advanced Assessment Frontend UX

**Input**: Design documents from `specs/028-advanced-assessment-ux/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/advanced-assessment-ux-contract.md`, `quickstart.md`

**Tests**: Frontend behavior changes require Vitest service, composable, route/component integration, upload validation, stale-response, safe-diagnostics, and contract-readiness coverage. Backend implementation is out of scope unless a separate backend/API feature approves missing behavior.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after shared advanced-assessment foundations are complete.

**Repository Scope**: `schoolmaster-specs` owns this task list, OpenAPI readiness checks, contract mirror checks, and acceptance evidence. `schoolmaster-frontend` is the lead implementation repository for Vue 3 SPA routes, services, composables, components, and Vitest coverage. `schoolmaster-backend` remains out of scope except for separate backend/API work if required advanced assessment contracts from `014-advanced-assessment-content` are missing or not contract-compliant.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because it touches different files or can be completed without depending on another incomplete task.
- **[Story]**: User story label for story phases only.
- Every task includes an exact file path.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Confirm contract scope, target frontend structure, and shared implementation locations before story work begins.

- [X] T001 Review advanced assessment frontend scope and blocked actions in `specs/028-advanced-assessment-ux/spec.md`
- [X] T002 Verify approved advanced assessment operation IDs in `api/openapi.yaml` and record coverage or gaps in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T003 Verify advanced assessment operation parity in `specs/001-schoolmaster-platform/contracts/openapi.yaml` and record coverage or gaps in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T004 [P] Confirm frontend advanced-assessment target folders in `src/pages/assessments/`, `src/components/assessments/`, `src/composables/assessments/`, `src/services/assessments/`, and `src/contracts/assessments/`
- [X] T005 [P] Confirm frontend test target folders in `tests/advanced-assessment/fixtures/`, `tests/advanced-assessment/services/`, `tests/advanced-assessment/composables/`, and `tests/advanced-assessment/components/`
- [X] T006 [P] Add advanced assessment i18n namespace placeholder in `src/i18n/modules/advanced-assessment.js`
- [X] T007 [P] Confirm advanced assessment route naming and navigation placement against protected shell routes in `src/router/modules/assessments.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared contract mapping, access gates, feedback states, stale-response guards, and workspace shell required by all advanced assessment stories.

**Critical**: No user story work can begin until this phase is complete.

### Contract and Readiness Tasks

- [X] T008 Add or verify questionnaire advanced question OpenAPI coverage for `createQuestionnaire`, `updateQuestionnaire`, and `getQuestionnaire` in `api/openapi.yaml`
- [X] T009 Add or verify student response OpenAPI coverage for `submitStudentQuestionnaireResponse` and `getStudentQuestionnaireResponse` in `api/openapi.yaml`
- [X] T010 Add or verify review and grading OpenAPI coverage for `listQuestionnaireResponses`, `getQuestionnaireResponse`, and `gradeQuestionnaireResponse` in `api/openapi.yaml`
- [X] T011 Add or verify clean answer-file download OpenAPI coverage for `downloadQuestionnaireResponseFile` in `api/openapi.yaml`
- [X] T012 Add or verify report aggregate field coverage for advanced assessment counts, completion status, grading status, and score summaries in `api/openapi.yaml`
- [X] T013 Mirror verified advanced assessment operation coverage in `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- [X] T014 Run Redocly validation for aggregate and platform contracts and record result in `specs/028-advanced-assessment-ux/quickstart.md`

### Tests for Foundation

- [X] T015 [P] Add advanced assessment fixture builders for questionnaires, response attempts, scan states, grading outcomes, report aggregates, and error envelopes in `tests/advanced-assessment/fixtures/advancedAssessmentFixtures.js`
- [X] T016 [P] Add advanced assessment service mapper tests for approved operations, contract-unavailable behavior, undocumented-field dropping, and no staged-upload or backend-draft service functions in `tests/advanced-assessment/services/advancedAssessmentService.test.js`
- [X] T017 [P] Add advanced assessment access-gate composable tests for active school, authoring, student submission, student result, review, grading, clean-file download, and reporting permissions in `tests/advanced-assessment/composables/useAdvancedAssessmentAccess.test.js`
- [X] T018 [P] Add stale-response and safe-diagnostics tests for route, active school, questionnaire, response, student, filters, due date, scan state, and grading state changes in `tests/advanced-assessment/composables/useAdvancedAssessmentRequestGuards.test.js`
- [X] T019 [P] Add shared advanced assessment feedback and status component tests for empty, denied, unavailable, conflict, scan, stale, and contract-unavailable states in `tests/advanced-assessment/components/AdvancedAssessmentSharedStates.test.js`

### Implementation for Foundation

- [X] T020 Create advanced assessment contract constants and mappers in `src/contracts/assessments/advancedAssessmentContract.js`
- [X] T021 Implement approved advanced assessment HTTP service functions in `src/services/assessments/advancedAssessmentService.js`
- [X] T022 Implement advanced assessment error-to-feedback normalization in `src/services/assessments/advancedAssessmentErrorMapper.js`
- [X] T023 Implement advanced assessment access gate composable with centralized contract-derived permission identifiers in `src/composables/assessments/useAdvancedAssessmentAccess.js`
- [X] T024 Implement advanced assessment stale-response and safe-diagnostics guards in `src/composables/assessments/useAdvancedAssessmentRequestGuards.js`
- [X] T025 Implement protected advanced assessment workspace route shell in `src/pages/assessments/AdvancedAssessmentWorkspacePage.vue`
- [X] T026 Wire authoring, student response, review, grading, student result, and reporting child routes in `src/router/modules/assessments.js`
- [X] T027 [P] Implement shared feedback, empty, denied, unavailable, conflict, scan, stale, and contract-unavailable states in `src/components/assessments/AdvancedAssessmentFeedbackState.vue`
- [X] T028 [P] Implement shared question-type, grading-state, scan-state, and due-date badges in `src/components/assessments/AdvancedAssessmentStatusBadges.vue`
- [X] T029 [P] Implement shared safe status update region for stale-response and contract-state feedback in `src/components/assessments/AdvancedAssessmentStatusRegion.vue`
- [X] T030 Add shared advanced assessment display text for foundation states in `src/i18n/modules/advanced-assessment.js`

**Checkpoint**: Foundation ready. User stories can now proceed in priority order or in parallel with separate owners.

---

## Phase 3: User Story 1 - Author Advanced Questionnaires (Priority: P1) MVP

**Goal**: Authorized teachers or school administrators can author questionnaires with existing question types plus approved `long_text` and `file_response` questions, see schema guidance and validation, and respect lifecycle locks.

**Independent Test**: Sign in as an authorized teacher or school administrator, create a questionnaire with existing questions plus `long_text` and `file_response`, verify schema guidance and validation feedback, and confirm unsupported question types or historical-meaning edits are blocked.

### Tests for User Story 1

- [X] T031 [P] [US1] Add questionnaire authoring service mapper tests for advanced question schemas, lifecycle lock state, validation errors, unsupported question types, and hidden-field exclusion in `tests/advanced-assessment/services/questionnaireAuthoringService.test.js`
- [X] T032 [P] [US1] Add authoring composable tests for schema controls, lifecycle locks, validation mapping, permission gates, and stale responses in `tests/advanced-assessment/composables/useAdvancedQuestionnaireAuthoring.test.js`
- [X] T033 [P] [US1] Add advanced question editor component tests for `long_text`, `file_response`, existing question type preservation, props-down/events-up updates, and no unsupported controls in `tests/advanced-assessment/components/AdvancedQuestionEditor.test.js`
- [X] T034 [P] [US1] Add questionnaire authoring page tests for locked questionnaires, denied/not-found states, empty states, and safe validation output in `tests/advanced-assessment/components/AdvancedQuestionnaireAuthoringPage.test.js`

### Implementation for User Story 1

- [X] T035 [P] [US1] Implement questionnaire authoring service mapper in `src/services/assessments/questionnaireAuthoringService.js` and authoring state, schema mapping, lifecycle lock mapping, and validation state in `src/composables/assessments/useAdvancedQuestionnaireAuthoring.js`
- [X] T036 [P] [US1] Build advanced question editor controls for existing, `long_text`, and `file_response` question types in `src/components/assessments/AdvancedQuestionEditor.vue`
- [X] T037 [P] [US1] Build long-text question schema controls and validation display in `src/components/assessments/LongTextQuestionSettings.vue`
- [X] T038 [P] [US1] Build file-response question rule controls for category, size, one-file, and grading expectations in `src/components/assessments/FileResponseQuestionSettings.vue`
- [X] T039 [P] [US1] Build questionnaire lifecycle lock and historical-meaning warning component in `src/components/assessments/QuestionnaireLifecycleLockNotice.vue`
- [X] T040 [US1] Build questionnaire authoring page composition in `src/pages/assessments/AdvancedQuestionnaireAuthoringPage.vue`
- [X] T041 [US1] Wire authoring route access and direct-route denied/not-found feedback in `src/router/modules/assessments.js`
- [X] T042 [US1] Add US1 authoring, schema, lifecycle lock, validation, unsupported question type, and denied text in `src/i18n/modules/advanced-assessment.js`

**Checkpoint**: User Story 1 is independently functional and testable as the MVP.

---

## Phase 4: User Story 2 - Submit Advanced Assessment Responses (Priority: P2)

**Goal**: Authenticated students can answer assigned advanced questionnaires with local-only text drafts and final-submit file-response selections, while duplicate, after-due-date, invalid, unsafe, and stale submissions are blocked.

**Independent Test**: Sign in as a student with an assigned same-school learning set, submit valid long-text and file-response answers, review pending scan visibility, and verify duplicate, expired, invalid, oversized, unsupported, blank, cross-tenant, and stale submissions are blocked safely.

### Tests for User Story 2

- [X] T043 [P] [US2] Add student response submission service mapper tests for final-submit text/files, multipart payloads, pending scan state, duplicate attempt, after-due-date, validation, forbidden, and conflict responses in `tests/advanced-assessment/services/studentResponseSubmissionService.test.js`
- [X] T044 [P] [US2] Add student response composable tests for assigned questions, one-attempt state, due-date state, local-only text drafts, final-submit file selection, and stale responses in `tests/advanced-assessment/composables/useStudentAdvancedResponse.test.js`
- [X] T045 [P] [US2] Add long-text answer entry tests for blank, whitespace-only, over-10,000 characters, invalid encoding, unsafe control characters, and plain-text display in `tests/advanced-assessment/components/LongTextAnswerEntry.test.js`
- [X] T046 [P] [US2] Add file-response selector tests for one-file rule, PDF/image/text/office categories, 25 MB limit, unsafe filename, multiple files, mismatch feedback, and no upload before final submit in `tests/advanced-assessment/components/FileResponseSelector.test.js`
- [X] T047 [P] [US2] Add student response page tests for submit readiness, submitted state, no resubmission action, safe feedback, and no partial answer state in `tests/advanced-assessment/components/StudentAdvancedResponsePage.test.js`

### Implementation for User Story 2

- [X] T048 [P] [US2] Implement student response submission service mapper in `src/services/assessments/studentResponseSubmissionService.js` and assigned response state, one-attempt gate, due-date gate, local draft state, and stale-response keys in `src/composables/assessments/useStudentAdvancedResponse.js`
- [X] T049 [P] [US2] Implement final-submit payload builder for text answers and selected files in `src/contracts/assessments/studentResponsePayloadMapper.js`
- [X] T050 [P] [US2] Build long-text answer entry component with local-only draft behavior in `src/components/assessments/LongTextAnswerEntry.vue`
- [X] T051 [P] [US2] Build file-response selector component with category, size, filename, one-file, and no-staged-upload feedback in `src/components/assessments/FileResponseSelector.vue`
- [X] T052 [P] [US2] Build assigned question response list component in `src/components/assessments/StudentAdvancedQuestionList.vue`
- [X] T053 [P] [US2] Build student submission readiness and due-date status component in `src/components/assessments/StudentResponseSubmitStatus.vue`
- [X] T054 [US2] Build student advanced response page composition in `src/pages/assessments/StudentAdvancedResponsePage.vue`
- [X] T055 [US2] Wire student response route access, ownership, and direct-route safe feedback in `src/router/modules/assessments.js`
- [X] T056 [US2] Add US2 submission, local draft, file selection, due-date, duplicate attempt, pending scan, and safe feedback text in `src/i18n/modules/advanced-assessment.js`

**Checkpoint**: User Story 2 is independently testable from an assigned student response route.

---

## Phase 5: User Story 3 - Grade and Review Advanced Responses (Priority: P3)

**Goal**: Authorized teachers and school administrators can list and review same-school advanced responses, download only clean files, manually grade eligible answers, and apply zero or exempt outcomes to failed-scan file answers.

**Independent Test**: Open submitted advanced responses as an authorized teacher or school administrator, grade manual-answer questions from 0 to 100, handle pending or failed scan files, add student-visible feedback, and verify unauthorized, stale, invalid-score, and cross-tenant grading attempts are denied safely.

### Tests for User Story 3

- [X] T057 [P] [US3] Add review queue service mapper tests for response list pagination, documented filters only (`questionnaire_id`, `learning_set_id`, `grading_status`), safe student identity, returned scan state display, grading state, auto-gradable versus manual-answer distinction, denied states, and private-field exclusion in `tests/advanced-assessment/services/assessmentReviewQueueService.test.js`
- [X] T058 [P] [US3] Add review detail and clean file download service tests for `getQuestionnaireResponse`, `downloadQuestionnaireResponseFile`, clean download, scan-pending, scan-failed, unavailable-file, forbidden, not-found, and no inline preview in `tests/advanced-assessment/services/assessmentReviewDetailService.test.js`
- [X] T059 [P] [US3] Add grading service mapper tests for 0-100 scores, failed-scan zero/exempt actions, invalid scores, conflicts, stale state, forbidden state, and returned-state authority in `tests/advanced-assessment/services/assessmentGradingService.test.js`
- [X] T060 [P] [US3] Add review queue composable tests for documented server-side filters, pagination, empty states, denied state, returned scan state display, grading states, and stale responses in `tests/advanced-assessment/composables/useAssessmentReviewQueue.test.js`
- [X] T061 [P] [US3] Add manual grading composable tests for authorized long-text answer display, score entry, feedback summary, failed-scan controls, permission gates, conflict handling, and private-note exclusion in `tests/advanced-assessment/composables/useManualAssessmentGrading.test.js`
- [X] T062 [P] [US3] Add review and grading component tests for authorized long-text answer display, clean download only, pending/failed/unavailable file states, auto-gradable existing answers, manual advanced answers, zero/exempt actions, manual grading, and safe feedback in `tests/advanced-assessment/components/AssessmentReviewAndGrading.test.js`

### Implementation for User Story 3

- [X] T063 [P] [US3] Implement review queue service mapper with documented filters only in `src/services/assessments/assessmentReviewQueueService.js` and list state, returned scan state display, pagination, safe student identity mapping, and stale-response keys in `src/composables/assessments/useAssessmentReviewQueue.js`
- [X] T064 [P] [US3] Implement review detail and clean file download service mapper in `src/services/assessments/assessmentReviewDetailService.js` and review detail loading and file availability mapping in `src/composables/assessments/useAssessmentReviewDetail.js`
- [X] T065 [P] [US3] Implement grading service mapper in `src/services/assessments/assessmentGradingService.js` and manual grading state, 0-100 score validation, feedback summary, and conflict mapping in `src/composables/assessments/useManualAssessmentGrading.js`
- [X] T066 [P] [US3] Implement failed-scan zero/exempt action mapper in `src/contracts/assessments/assessmentGradingMapper.js`
- [X] T067 [P] [US3] Build assessment review queue documented filters and returned state list in `src/components/assessments/AssessmentReviewQueue.vue`
- [X] T068 [P] [US3] Build response detail panel with safe answer and scan state display in `src/components/assessments/AssessmentResponseDetail.vue`
- [X] T069 [P] [US3] Build clean answer-file download control without inline preview in `src/components/assessments/AnswerFileDownloadControl.vue`
- [X] T070 [P] [US3] Build manual grading form with authorized long-text answer display and clean file-response answers in `src/components/assessments/ManualAssessmentGradingForm.vue`
- [X] T071 [P] [US3] Build failed-scan zero/exempt grading controls in `src/components/assessments/FailedScanGradingActions.vue`
- [X] T072 [US3] Build assessment review queue page composition in `src/pages/assessments/AssessmentReviewQueuePage.vue`
- [X] T073 [US3] Build assessment response grading page composition in `src/pages/assessments/AssessmentResponseGradingPage.vue`
- [X] T074 [US3] Wire review and grading route access, response ownership, and direct-route safe feedback in `src/router/modules/assessments.js`
- [X] T075 [US3] Add US3 review, grading, clean download, scan, failed-scan zero/exempt, conflict, and safe feedback text in `src/i18n/modules/advanced-assessment.js`

**Checkpoint**: User Story 3 is independently testable from review queue and response grading routes with fixture-backed response states.

---

## Phase 6: User Story 4 - View Student Results and Reporting Summaries (Priority: P4)

**Goal**: Students and reporting users can view only approved advanced assessment summaries: student-owned status/score/feedback/file metadata and aggregate reporting counts, completion status, grading status, and score summaries.

**Independent Test**: View graded and ungraded advanced responses through student and reporting surfaces, confirm students see only their own status, score, feedback summary, and safe file metadata, and confirm reporting users see only counts, completion status, grading status, and score summaries.

### Tests for User Story 4

- [X] T076 [P] [US4] Add student result service mapper tests for own status, grading status, score summary, feedback summary, safe file metadata, scan-pending, scan-failed, forbidden, not-found, and private-field exclusion in `tests/advanced-assessment/services/studentAssessmentResultService.test.js`
- [X] T077 [P] [US4] Add reporting aggregate service mapper tests for report catalog, report definition, report request, report history, result fields, aggregate-only exposure, and blocked raw answer/file fields in `tests/advanced-assessment/services/advancedAssessmentReportingService.test.js`
- [X] T078 [P] [US4] Add student result composable tests for ownership, pending/unavailable states, stale responses, and no private grading note exposure in `tests/advanced-assessment/composables/useStudentAssessmentResult.test.js`
- [X] T079 [P] [US4] Add reporting aggregate composable tests for allowed field mapping, unsupported field blocking, permission gates, and tenant-safe denial in `tests/advanced-assessment/composables/useAdvancedAssessmentReporting.test.js`
- [X] T080 [P] [US4] Add student result and reporting component tests for safe result summaries, aggregate-only reports, and no raw answer/file disclosure in `tests/advanced-assessment/components/StudentResultAndReporting.test.js`

### Implementation for User Story 4

- [X] T081 [P] [US4] Implement student result service mapper in `src/services/assessments/studentAssessmentResultService.js` and assessment result loading, own-response gate, safe file metadata mapping, and stale-response keys in `src/composables/assessments/useStudentAssessmentResult.js`
- [X] T082 [P] [US4] Implement advanced assessment reporting service mapper in `src/services/assessments/advancedAssessmentReportingService.js` and aggregate field mapping and blocked-field handling in `src/composables/assessments/useAdvancedAssessmentReporting.js`
- [X] T083 [P] [US4] Build student assessment result summary component in `src/components/assessments/StudentAssessmentResultSummary.vue`
- [X] T084 [P] [US4] Build student result file availability component without private file disclosure in `src/components/assessments/StudentResultFileAvailability.vue`
- [X] T085 [P] [US4] Build reporting aggregate field panel for counts, completion status, grading status, and score summaries in `src/components/assessments/AdvancedAssessmentReportingPanel.vue`
- [X] T086 [US4] Build student assessment result page composition in `src/pages/assessments/StudentAssessmentResultPage.vue`
- [X] T087 [US4] Integrate advanced assessment reporting panel with existing reporting surfaces in `src/pages/assessments/AdvancedAssessmentReportingPage.vue`
- [X] T088 [US4] Wire student result and reporting route access with safe denied/not-found feedback in `src/router/modules/assessments.js`
- [X] T089 [US4] Add US4 student result, reporting aggregate, pending/unavailable, unsupported field, and privacy text in `src/i18n/modules/advanced-assessment.js`

**Checkpoint**: User Story 4 is independently testable from student result and reporting aggregate surfaces.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Full-flow verification, accessibility, security, performance, documentation evidence, and contract guardrails.

- [X] T090 [P] Audit responsive layout and WCAG 2.1 AA behavior at 390px, 768px, and 1440px in `src/pages/assessments/AdvancedAssessmentWorkspacePage.vue`
- [X] T091 [P] Audit keyboard behavior for authoring, local text drafts, file selection, submit, clean download, grading, failed-scan actions, and reporting controls in `src/components/assessments/`
- [X] T092 [P] Audit no-sensitive-data diagnostics, visible errors, validation labels, filenames, report columns, and automated test fixtures in `tests/advanced-assessment/`
- [X] T093 [P] Audit blocked unsupported advanced assessment actions are absent from controls in `src/components/assessments/`
- [X] T094 [P] Audit Element Plus PascalCase usage and no direct Axios calls outside services in `src/pages/assessments/`, `src/components/assessments/`, `src/composables/assessments/`, `src/router/modules/assessments.js`, `src/stores/`, and `src/contracts/assessments/`
- [X] T095 Verify timed usability checks for SC-002, SC-003, and SC-006, then record result in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T096 Verify frontend performance goals for mocked render, protected route transition, final response submission, grading actions, and stale-response handling, then record result in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T097 Run focused frontend unit tests and record result in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T098 Run frontend build check and record result in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T099 Run OpenAPI lint for advanced assessment contract coverage and record result in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T100 Update implementation acceptance evidence in `specs/028-advanced-assessment-ux/quickstart.md`
- [X] T101 Verify all advanced assessment tasks remain traceable to approved contracts in `specs/028-advanced-assessment-ux/contracts/advanced-assessment-ux-contract.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies. Complete before shared foundation.
- **Foundational (Phase 2)**: Depends on Setup. Blocks all user stories.
- **User Stories (Phases 3-6)**: Depend on Foundational completion.
- **Polish (Phase 7)**: Depends on all implemented user stories.

### User Story Dependencies

- **US1 (P1)**: Can start after Foundational. MVP scope.
- **US2 (P2)**: Can start after Foundational and can be tested with assigned questionnaire fixtures independently of US1 runtime pages.
- **US3 (P3)**: Can start after Foundational and can be tested with review/response fixtures independently of US2 runtime submission.
- **US4 (P4)**: Can start after Foundational and can be tested with student result and report aggregate fixtures independently of US3.

### Within Each User Story

- Write service, composable, and component tests before implementation.
- Implement service and contract mappers before composables.
- Implement composables before route/page integration.
- Keep route views thin and keep Axios calls inside advanced assessment services only.
- Preserve active school, permission gates, tenant-safe denial, local-only draft, final-submit file, scan-state, stale-response, and safe-diagnostics behavior in every story.

## Parallel Opportunities

- Setup tasks T004, T005, T006, and T007 can run in parallel after T001 through T003 context review.
- Foundation contract tasks T008 through T012 can run in parallel before T013 and T014.
- Foundation tests T015 through T019 can run in parallel.
- Foundation implementation tasks T027, T028, and T029 can run in parallel after T020 through T026 define shared contracts and shell.
- Tests inside each user story can run in parallel because they target separate service, composable, and component files.
- US2, US3, and US4 can be implemented by separate owners after Foundation if fixture-backed integration points are stable.

## Parallel Example: User Story 1

```bash
# Launch US1 test tasks together:
Task: "T031 [US1] Add questionnaire authoring service mapper tests in tests/advanced-assessment/services/questionnaireAuthoringService.test.js"
Task: "T032 [US1] Add authoring composable tests in tests/advanced-assessment/composables/useAdvancedQuestionnaireAuthoring.test.js"
Task: "T033 [US1] Add advanced question editor component tests in tests/advanced-assessment/components/AdvancedQuestionEditor.test.js"
Task: "T034 [US1] Add questionnaire authoring page tests in tests/advanced-assessment/components/AdvancedQuestionnaireAuthoringPage.test.js"
```

## Parallel Example: User Story 2

```bash
# Launch US2 independent implementation tasks after tests:
Task: "T048 [US2] Implement student response submission service mapper in src/services/assessments/studentResponseSubmissionService.js and assigned response state in src/composables/assessments/useStudentAdvancedResponse.js"
Task: "T049 [US2] Implement final-submit payload builder in src/contracts/assessments/studentResponsePayloadMapper.js"
Task: "T050 [US2] Build long-text answer entry in src/components/assessments/LongTextAnswerEntry.vue"
Task: "T051 [US2] Build file-response selector in src/components/assessments/FileResponseSelector.vue"
Task: "T052 [US2] Build assigned question response list in src/components/assessments/StudentAdvancedQuestionList.vue"
Task: "T053 [US2] Build student submission readiness in src/components/assessments/StudentResponseSubmitStatus.vue"
```

## Parallel Example: User Story 3

```bash
# Launch US3 review and grading implementation tasks after tests:
Task: "T063 [US3] Implement review queue service mapper with documented filters only in src/services/assessments/assessmentReviewQueueService.js and list state in src/composables/assessments/useAssessmentReviewQueue.js"
Task: "T064 [US3] Implement review detail and clean file download service mapper in src/services/assessments/assessmentReviewDetailService.js and file availability mapping in src/composables/assessments/useAssessmentReviewDetail.js"
Task: "T065 [US3] Implement grading service mapper in src/services/assessments/assessmentGradingService.js and manual grading state in src/composables/assessments/useManualAssessmentGrading.js"
Task: "T067 [US3] Build documented review queue filters in src/components/assessments/AssessmentReviewQueue.vue"
Task: "T070 [US3] Build manual grading form with authorized long-text answer display in src/components/assessments/ManualAssessmentGradingForm.vue"
Task: "T071 [US3] Build failed-scan grading controls in src/components/assessments/FailedScanGradingActions.vue"
```

## Parallel Example: User Story 4

```bash
# Launch US4 test tasks together:
Task: "T076 [US4] Add student result service mapper tests in tests/advanced-assessment/services/studentAssessmentResultService.test.js"
Task: "T077 [US4] Add reporting aggregate service mapper tests in tests/advanced-assessment/services/advancedAssessmentReportingService.test.js"
Task: "T078 [US4] Add student result composable tests in tests/advanced-assessment/composables/useStudentAssessmentResult.test.js"
Task: "T079 [US4] Add reporting aggregate composable tests in tests/advanced-assessment/composables/useAdvancedAssessmentReporting.test.js"
Task: "T080 [US4] Add student result and reporting component tests in tests/advanced-assessment/components/StudentResultAndReporting.test.js"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Deliver Phase 3 User Story 1 as the MVP.
3. Validate that advanced questionnaire authoring uses only approved schemas, preserves existing question behavior, and blocks unsupported question types and lifecycle-locked edits.

### Incremental Delivery

1. Add User Story 2 for student long-text/file-response submission, local-only text drafts, final-submit files, due-date blocking, and one-attempt behavior.
2. Add User Story 3 for teacher/admin review, clean-file downloads, manual grading, and failed-scan zero/exempt actions.
3. Add User Story 4 for student result summaries and reporting aggregate-only fields.
4. Complete Phase 7 polish, evidence, and validation.

### Team Strategy

1. One owner completes OpenAPI readiness and service contract mapping in Foundation.
2. One owner builds authoring as MVP after Foundation.
3. One owner builds student submission after Foundation.
4. One owner builds review and grading after response fixtures stabilize.
5. One owner builds student result and reporting aggregate surfaces after result/report fixtures stabilize.
6. QA or another frontend owner runs cross-story accessibility, diagnostics, and blocked-action audits in Phase 7.
