# Tasks: Guardian Self-Service UI

**Input**: Design documents from `specs/025-guardian-self-service-ui/`
**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md), [research.md](research.md), [data-model.md](data-model.md), [contracts/guardian-self-service-ui-contract.md](contracts/guardian-self-service-ui-contract.md), [quickstart.md](quickstart.md)

**Tests**: Required by FR-020 and quickstart verification. Write focused Vitest service, composable, route, and component tests before implementation tasks in each story.

**Organization**: Tasks are grouped by user story so each story can be implemented and tested independently after setup and foundation.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish frontend module boundaries, route/i18n scaffolds, and contract traceability before shared implementation starts.

- [ ] T001 Confirm approved guardian self-service operation IDs and blocked contract gaps in `specs/025-guardian-self-service-ui/contracts/guardian-self-service-ui-contract.md`
- [ ] T002 Create guardian feature folders in `src/pages/guardian/`, `src/components/guardian/`, `src/composables/guardian/`, `src/services/guardian/`, and `src/contracts/guardian/`
- [ ] T003 [P] Create focused test folders in `tests/guardian-self-service/services/`, `tests/guardian-self-service/composables/`, `tests/guardian-self-service/components/`, and `tests/guardian-self-service/routes/`
- [ ] T004 [P] Add guardian self-service i18n namespace scaffold in `src/i18n/modules/guardianSelfService.js`
- [ ] T005 [P] Add guardian route module scaffold in `src/router/modules/guardian.js`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Shared contract, context, feedback, stale-response, and diagnostics infrastructure required by every story.

**Critical**: No user story work starts until this phase is complete.

- [ ] T006 [P] Add foundation tests for authenticated access, active school, no-guardian-link safe signal, generic denial without no-guardian-link inference, current period, feedback mapping, stale guard, and diagnostics redaction in `tests/guardian-self-service/composables/guardianSelfServiceFoundation.spec.js`
- [ ] T007 Create approved operation and blocked-capability map in `src/contracts/guardian/guardianSelfServiceContract.js`
- [ ] T008 Create guardian response mappers for paginated envelopes, student summaries, student detail, academic summary, contact view, and safe dropped fields in `src/contracts/guardian/guardianSelfServiceMappers.js`
- [ ] T009 Create guardian self-service Axios wrapper and exported service methods in `src/services/guardian/guardianSelfServiceService.js`
- [ ] T010 [P] Create shared feedback-state normalization for unauthorized, forbidden, tenant-mismatch, inactive-school, no-active-school, no-guardian-link, no-linked-students, no-academic-period, unavailable-summary, validation, not-found, unsupported-page-size, stale-response, and temporary-unavailable states in `src/services/guardian/guardianSelfServiceFeedbackMapper.js`
- [ ] T011 [P] Create safe diagnostics redaction for unassociated student identifiers, other guardian data, non-primary contacts, school-only notes, correction details, teacher-private data, report data, tokens, role internals, raw denial reasons, and cross-tenant details in `src/services/guardian/guardianSelfServiceDiagnostics.js`
- [ ] T012 Create stale-response guard composable for route, selected student, active school, academic period, pagination, authentication, and session changes in `src/composables/guardian/useGuardianSelfServiceStaleGuard.js`
- [ ] T013 Create guardian workspace context composable for active school, safe no-guardian-link signal, current active academic period, default landing, and blocking gates in `src/composables/guardian/useGuardianWorkspaceContext.js`
- [ ] T014 [P] Create shared guardian feedback-state component for loading, empty, denied, unavailable, no-active-school, no-guardian-link, no-linked-students, no-academic-period, unavailable-summary, not-found, and temporary-unavailable states in `src/components/guardian/GuardianFeedbackState.vue`
- [ ] T015 [P] Create shared guardian status and pagination components for relationship labels, profile status, summary status, and paginated list controls in `src/components/guardian/GuardianStatusControls.vue`
- [ ] T016 Wire guardian route module into application router in `src/router/index.js`
- [ ] T017 Add shared guardian workspace and feedback display text to `src/i18n/modules/guardianSelfService.js`

**Checkpoint**: Foundation ready. User story implementation can begin.

---

## Phase 3: User Story 1 - Review Linked Students (Priority: P1) MVP

**Goal**: Guardian workspace opens Linked Students by default and lists only approved same-school linked students with pagination, relationship labels, safe gates, and true no-linked-students empty state.

**Independent Test**: Sign in as a guardian with active school, active guardian-user link, and linked students; open guardian workspace root; verify Linked Students loads only same-school linked students, no-linked-students is distinct from denial, and no-active-school or safe no-guardian-link gates block data requests.

### Tests for User Story 1

- [ ] T018 [P] [US1] Add service and mapper tests for `listGuardianStudents`, tenant context, pagination, dropped fields, no-linked-students, and no undocumented parameters in `tests/guardian-self-service/services/linkedStudentsService.spec.js`
- [ ] T019 [P] [US1] Add composable tests for workspace default landing, active-school gate, safe no-guardian-link signal, generic denial without no-guardian-link inference, pagination, empty state, and stale-response handling in `tests/guardian-self-service/composables/useGuardianLinkedStudents.spec.js`
- [ ] T020 [P] [US1] Add route/component tests for workspace root, linked student list, relationship labels, no-active-school, no-guardian-link, no-linked-students, unauthorized, forbidden, tenant-mismatch, and inactive-school states in `tests/guardian-self-service/components/linkedStudentsViews.spec.js`

### Implementation for User Story 1

- [ ] T021 [US1] Implement linked student service method and request/response mapper in `src/services/guardian/guardianSelfServiceService.js`
- [ ] T022 [US1] Implement linked student state, pagination, active-school gate, safe no-guardian-link handling, empty state, stale guard, and feedback mapping in `src/composables/guardian/useGuardianLinkedStudents.js`
- [ ] T023 [P] [US1] Implement linked student list component with status, relationship label, pagination, empty, loading, and denied states in `src/components/guardian/GuardianLinkedStudentsList.vue`
- [ ] T024 [P] [US1] Implement guardian workspace route view with default Linked Students landing and context gates in `src/pages/guardian/GuardianWorkspaceView.vue`
- [ ] T025 [US1] Register guardian workspace root and linked student list routes in `src/router/modules/guardian.js`
- [ ] T026 [US1] Add linked students, no-linked-students, no-guardian-link, generic denial, inactive-school, tenant-mismatch, and workspace display text in `src/i18n/modules/guardianSelfService.js`

**Checkpoint**: User Story 1 is independently functional and testable as MVP.

---

## Phase 4: User Story 2 - View Linked Student Detail (Priority: P2)

**Goal**: Guardians can open limited linked student detail while stale, missing, unassociated, inactive, transferred, deleted, and cross-tenant targets all render the same safe not-found state.

**Independent Test**: Select a linked student from the list, open detail, verify only approved profile and enrollment summary fields appear, then attempt direct denied target routes and confirm identical not-found behavior without protected-record enumeration.

### Tests for User Story 2

- [ ] T027 [P] [US2] Add service and mapper tests for `getGuardianStudent`, limited profile fields, enrollment summary fields, dropped restricted fields, and not-found mapping in `tests/guardian-self-service/services/guardianStudentDetailService.spec.js`
- [ ] T028 [P] [US2] Add composable tests for selected student detail loading, target-specific not-found non-enumeration, stale selected-student changes, and safe diagnostics in `tests/guardian-self-service/composables/useGuardianStudentDetail.spec.js`
- [ ] T029 [P] [US2] Add route/component tests for linked student detail, restricted field absence, stale direct routes, not-found state, and return-to-list behavior in `tests/guardian-self-service/components/guardianStudentDetailViews.spec.js`

### Implementation for User Story 2

- [ ] T030 [US2] Implement guardian student detail service method and response mapper in `src/services/guardian/guardianSelfServiceService.js`
- [ ] T031 [US2] Implement selected student detail state, non-enumerating not-found mapping, stale guard, and diagnostics redaction in `src/composables/guardian/useGuardianStudentDetail.js`
- [ ] T032 [P] [US2] Implement limited student detail summary component with profile, enrollment, relationship label, and restricted-field omission in `src/components/guardian/GuardianStudentDetailSummary.vue`
- [ ] T033 [US2] Implement linked student detail route view with safe not-found and no-active-school/no-guardian-link states in `src/pages/guardian/GuardianStudentDetailView.vue`
- [ ] T034 [US2] Register linked student detail route in `src/router/modules/guardian.js`
- [ ] T035 [US2] Add linked student detail, enrollment summary, restricted field absence, and non-enumerating not-found display text in `src/i18n/modules/guardianSelfService.js`

**Checkpoint**: User Story 2 is independently functional and non-enumerating.

---

## Phase 5: User Story 3 - Review Academic Summaries (Priority: P3)

**Goal**: Guardians can view current-active-period grade, attendance, and learning-set summaries for linked students while period picker, detailed rows, reports, teacher content, questionnaires, and student activity controls remain absent.

**Independent Test**: Open academic summary for a linked student with current active period, verify summary-only values render, then verify no-academic-period blocks requests and denied targets use safe not-found behavior.

### Tests for User Story 3

- [ ] T036 [P] [US3] Add service and mapper tests for `getGuardianStudentAcademics`, required current `academic_period_id`, grade summary, attendance summary, learning-set summary, dropped restricted fields, and no undocumented parameters in `tests/guardian-self-service/services/guardianAcademicSummaryService.spec.js`
- [ ] T037 [P] [US3] Add composable tests for current active period gate, no-academic-period request blocking, unavailable-summary mapping, target-specific not-found, stale period/student changes, and diagnostics redaction in `tests/guardian-self-service/composables/useGuardianAcademicSummary.spec.js`
- [ ] T038 [P] [US3] Add route/component tests for grade summary, attendance summary, learning-set summary, null value rendering, no manual period picker, no detailed rows, no reports, and denied states in `tests/guardian-self-service/components/guardianAcademicSummaryViews.spec.js`

### Implementation for User Story 3

- [ ] T039 [US3] Implement guardian academic summary service method and response mapper in `src/services/guardian/guardianSelfServiceService.js`
- [ ] T040 [US3] Implement academic summary state, current-period gate, no-academic-period request blocking, unavailable-summary mapping, stale guard, and safe feedback in `src/composables/guardian/useGuardianAcademicSummary.js`
- [ ] T041 [P] [US3] Implement grade and attendance summary panels with safe null-value rendering in `src/components/guardian/GuardianAcademicSummaryPanels.vue`
- [ ] T042 [P] [US3] Implement learning-set summary list with status, progress, last activity, and no teacher content or questionnaire actions in `src/components/guardian/GuardianLearningSetSummaryList.vue`
- [ ] T043 [US3] Implement academic summary route view with current active period only and no period picker in `src/pages/guardian/GuardianAcademicSummaryView.vue`
- [ ] T044 [US3] Register guardian academic summary route in `src/router/modules/guardian.js`
- [ ] T045 [US3] Add academic summary, no-academic-period, unavailable-summary, null summary, blocked detail, blocked report, and blocked period-picker display text in `src/i18n/modules/guardianSelfService.js`

**Checkpoint**: User Story 3 is independently functional and summary-limited.

---

## Phase 6: User Story 4 - Review Contact and Relationship Information (Priority: P4)

**Goal**: Guardians can view only authenticated guardian contact fields, relationship label, and selected student's primary school-approved contact details with safe missing values and restricted contact redaction.

**Independent Test**: Open contact view for a linked student, verify approved guardian and student primary contact fields appear, null values render safely, and other guardians, non-primary contacts, emergency details, custody/legal details, school-only notes, and edit controls remain absent.

### Tests for User Story 4

- [ ] T046 [P] [US4] Add service and mapper tests for `getGuardianStudentContacts`, guardian contact fields, relationship label, student primary contact fields, safe null values, dropped restricted fields, and not-found mapping in `tests/guardian-self-service/services/guardianContactViewService.spec.js`
- [ ] T047 [P] [US4] Add composable tests for contact loading, safe missing-value state, target-specific not-found, stale selected-student changes, and diagnostics redaction in `tests/guardian-self-service/composables/useGuardianContactView.spec.js`
- [ ] T048 [P] [US4] Add route/component tests for guardian contact, relationship label, student primary contact, missing values, restricted contact absence, no edit controls, and denied states in `tests/guardian-self-service/components/guardianContactView.spec.js`

### Implementation for User Story 4

- [ ] T049 [US4] Implement guardian contact view service method and response mapper in `src/services/guardian/guardianSelfServiceService.js`
- [ ] T050 [US4] Implement contact view state, safe missing-value handling, target not-found mapping, stale guard, and diagnostics redaction in `src/composables/guardian/useGuardianContactView.js`
- [ ] T051 [P] [US4] Implement guardian contact panel with guardian-owned fields and relationship label in `src/components/guardian/GuardianContactPanel.vue`
- [ ] T052 [P] [US4] Implement student primary contact panel with safe missing-value rendering and restricted contact omission in `src/components/guardian/GuardianStudentPrimaryContactPanel.vue`
- [ ] T053 [US4] Implement contact view route with no edit controls and safe denied/not-found states in `src/pages/guardian/GuardianContactView.vue`
- [ ] T054 [US4] Register guardian contact view route in `src/router/modules/guardian.js`
- [ ] T055 [US4] Add contact view, missing contact value, relationship label, blocked contact edit, restricted contact, and not-found display text in `src/i18n/modules/guardianSelfService.js`

**Checkpoint**: User Story 4 is independently functional and contact-limited.

---

## Phase 7: Polish and Cross-Cutting Concerns

**Purpose**: Verify feature-wide contract compliance, accessibility, diagnostics, documentation, and quality gates.

- [ ] T056 [P] Verify operation ID to UI surface traceability against `specs/025-guardian-self-service-ui/contracts/guardian-self-service-ui-contract.md`
- [ ] T057 [P] Verify WCAG 2.1 AA responsive behavior, keyboard navigation, focus visibility, labels, landmarks/headings, and contrast at 390px, 768px, and 1440px for all guardian routes in `src/pages/guardian/`
- [ ] T058 Verify no direct Axios calls exist outside guardian services in `src/pages/guardian/`, `src/components/guardian/`, and `src/composables/guardian/`
- [ ] T059 Verify no manual period switch, free-form academic period query, guardian write, profile update, association request, guardian-user-link provisioning, detailed academic rows, teacher content, questionnaire action, report UI, messaging, notification-center, school-admin, student self-service, platform support, billing, legal workflow, or undocumented behavior exists in `src/pages/guardian/`, `src/components/guardian/`, `src/composables/guardian/`, `src/services/guardian/`, `src/contracts/guardian/`, and `src/router/modules/guardian.js`
- [ ] T060 [P] Verify diagnostics and test output redaction for unassociated student identifiers, other guardian data, non-primary contacts, school-only notes, correction details, teacher-private data, report data, tokens, role internals, raw denial reasons, and cross-tenant details in `tests/guardian-self-service/`
- [ ] T061 Run focused guardian self-service tests through `npm run test:unit -- tests/guardian-self-service` using `package.json` and record results in implementation PR notes
- [ ] T062 Run full frontend unit tests through `npm run test:unit` using `package.json` and record results in implementation PR notes
- [ ] T063 Run frontend build verification through `npm run build` using `package.json` and record results in implementation PR notes
- [ ] T064 Run OpenAPI validation with `npx @redocly/cli lint api/openapi.yaml` only if `api/openapi.yaml` changed and record results in implementation PR notes
- [ ] T065 Record timed usability evidence for SC-002, SC-003, and SC-004 against `specs/025-guardian-self-service-ui/spec.md` using linked student detail under 2 minutes, academic summary under 3 minutes, and contact view under 2 minutes
- [ ] T066 Record representative guardian UAT evidence for SC-009 against `specs/025-guardian-self-service-ui/spec.md` confirming at least 90% can distinguish linked-student list, no-linked-students, student detail, academic summary, contact view, no-academic-period, unavailable-summary, and not-found states
- [ ] T067 Verify frontend timing goals from `specs/025-guardian-self-service-ui/plan.md` and record evidence in implementation PR notes for mocked service render within 1.5s, guardian route transition within 2s, and selected-student views within 2s after service resolution

---

## Dependencies and Execution Order

### Phase Dependencies

- Setup (Phase 1): no dependencies.
- Foundational (Phase 2): depends on Setup completion and blocks all user stories.
- User Story 1 (Phase 3): depends on Foundation; recommended MVP.
- User Story 2 (Phase 4): depends on Foundation and can use selected student route context from US1.
- User Story 3 (Phase 5): depends on Foundation and selected student route context; can use mocked linked-student context for independent testing.
- User Story 4 (Phase 6): depends on Foundation and selected student route context; can use mocked linked-student context for independent testing.
- Polish (Phase 7): depends on selected user stories being complete.

### User Story Dependencies

- US1: can start after Foundation; no other story dependency.
- US2: can start after Foundation; integrates with US1 list selection but remains independently testable by direct linked-student route.
- US3: can start after Foundation; requires selected student and current active period context but not US2 implementation.
- US4: can start after Foundation; requires selected student context but not US2 or US3 implementation.

### Within Each User Story

- Write tests first and confirm they fail.
- Implement service/mappers before composables.
- Implement composables before route views.
- Implement components before final route integration.
- Add route registration and i18n after primary components exist.
- Verify each story independently before starting polish.

## Parallel Opportunities

- T003, T004, and T005 can run in parallel after T002 is agreed.
- T006, T010, T011, T014, and T015 can run in parallel after T007 and T008 are drafted.
- US1 tests T018, T019, and T020 can run in parallel.
- US1 components T023 and T024 can run in parallel after T022 exists.
- US2 tests T027, T028, and T029 can run in parallel.
- US3 tests T036, T037, and T038 can run in parallel.
- US3 components T041 and T042 can run in parallel after T040 exists.
- US4 tests T046, T047, and T048 can run in parallel.
- US4 components T051 and T052 can run in parallel after T050 exists.
- Polish checks T056, T057, and T060 can run in parallel after selected stories are implemented.

## Parallel Example: User Story 1

```bash
Task: "T018 [P] [US1] Add linked student service tests in tests/guardian-self-service/services/linkedStudentsService.spec.js"
Task: "T019 [P] [US1] Add linked student composable tests in tests/guardian-self-service/composables/useGuardianLinkedStudents.spec.js"
Task: "T020 [P] [US1] Add linked student component tests in tests/guardian-self-service/components/linkedStudentsViews.spec.js"
Task: "T023 [P] [US1] Implement linked student list component in src/components/guardian/GuardianLinkedStudentsList.vue"
Task: "T024 [P] [US1] Implement guardian workspace route view in src/pages/guardian/GuardianWorkspaceView.vue"
```

## Parallel Example: User Story 2

```bash
Task: "T027 [P] [US2] Add student detail service tests in tests/guardian-self-service/services/guardianStudentDetailService.spec.js"
Task: "T028 [P] [US2] Add student detail composable tests in tests/guardian-self-service/composables/useGuardianStudentDetail.spec.js"
Task: "T029 [P] [US2] Add student detail component tests in tests/guardian-self-service/components/guardianStudentDetailViews.spec.js"
Task: "T032 [P] [US2] Implement student detail summary component in src/components/guardian/GuardianStudentDetailSummary.vue"
```

## Parallel Example: User Story 3

```bash
Task: "T036 [P] [US3] Add academic summary service tests in tests/guardian-self-service/services/guardianAcademicSummaryService.spec.js"
Task: "T037 [P] [US3] Add academic summary composable tests in tests/guardian-self-service/composables/useGuardianAcademicSummary.spec.js"
Task: "T038 [P] [US3] Add academic summary component tests in tests/guardian-self-service/components/guardianAcademicSummaryViews.spec.js"
Task: "T041 [P] [US3] Implement summary panels in src/components/guardian/GuardianAcademicSummaryPanels.vue"
Task: "T042 [P] [US3] Implement learning-set summary list in src/components/guardian/GuardianLearningSetSummaryList.vue"
```

## Parallel Example: User Story 4

```bash
Task: "T046 [P] [US4] Add contact service tests in tests/guardian-self-service/services/guardianContactViewService.spec.js"
Task: "T047 [P] [US4] Add contact composable tests in tests/guardian-self-service/composables/useGuardianContactView.spec.js"
Task: "T048 [P] [US4] Add contact component tests in tests/guardian-self-service/components/guardianContactView.spec.js"
Task: "T051 [P] [US4] Implement guardian contact panel in src/components/guardian/GuardianContactPanel.vue"
Task: "T052 [P] [US4] Implement student primary contact panel in src/components/guardian/GuardianStudentPrimaryContactPanel.vue"
```

## Implementation Strategy

### MVP First

1. Complete Phase 1 and Phase 2.
2. Complete Phase 3 for User Story 1.
3. Run focused US1 tests and manual checks for Linked Students, default landing, no-active-school, safely identified no-guardian-link, no-linked-students, tenant-mismatch, and generic denial without no-guardian-link inference.
4. Stop and demo MVP before adding student detail, academic summary, or contact view.

### Incremental Delivery

1. Complete Setup and Foundation.
2. Add US1 Linked Students as MVP.
3. Add US2 Linked Student Detail.
4. Add US3 Academic Summary.
5. Add US4 Contact View.
6. Run polish checks and quickstart validation.

### Parallel Team Strategy

1. Team completes Setup and Foundation together.
2. Developer A implements US1 linked student list.
3. Developer B implements US2 student detail after selected-student route contract is stable.
4. Developer C implements US3 academic summary with mocked selected-student context.
5. Developer D implements US4 contact view with mocked selected-student context.
6. Team finishes polish checks together.

## Notes

- [P] tasks use different files or test scopes and can run in parallel after dependencies are met.
- [US1], [US2], [US3], and [US4] labels map to spec user stories.
- This is a frontend-only implementation task list unless a future contract gap creates separate backend/OpenAPI work.
- No task may introduce undocumented API behavior or direct Axios calls outside services.
- Verify required critical-flow tests fail before implementing each story.
