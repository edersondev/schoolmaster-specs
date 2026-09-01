# Tasks: Student Guardian Tabs

**Input**: Design documents from `specs/040-student-guardian-tabs/`
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/student-guardian-tabs-contract.md`, and `quickstart.md`

**Tests**: Critical REST contract, backend PHPUnit, frontend Vitest, build,
responsive, keyboard, timed usability, performance timing, and diagnostics evidence is required. Complete all
OpenAPI, backend implementation, and backend verification tasks before any
frontend implementation task.

**Organization**: Tasks are grouped by user story so each story can be
implemented and tested independently after the backend gate.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel because the task changes different files and
  does not depend on another incomplete task.
- **[Story]**: Maps the task to a user story from `spec.md`.
- Setup, Foundational, and Polish phases have no story label.

## Phase 1: Setup

**Purpose**: Confirm repository branches, baseline contracts, and source
locations before contract or implementation work starts.

- [X] T001 Confirm `040-student-guardian-tabs` is checked out or created in `schoolmaster-backend/` and `schoolmaster-frontend/`, then record branch readiness in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T002 [P] Review current student create and guardian contracts for `guardian_associations`, `createStudentProfile`, `listGuardians`, and `createGuardian` in `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfileCreateRequest.yaml`, `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfileGuardianInput.yaml`, `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfile.yaml`, and `schoolmaster-specs/api/paths/guardians/index.yaml`
- [X] T003 [P] Review existing backend student, guardian, and association request/service/resource/policy files to identify exact class names in `schoolmaster-backend/app/Http/Requests/`, `schoolmaster-backend/app/Services/`, `schoolmaster-backend/app/Policies/`, and `schoolmaster-backend/app/Http/Resources/`
- [X] T004 [P] Review existing frontend student and guardian route/service/component files to identify exact route names and component boundaries in `schoolmaster-frontend/src/router/modules/`, `schoolmaster-frontend/src/services/admin-system/`, `schoolmaster-frontend/src/pages/admin-system/`, and `schoolmaster-frontend/src/components/admin-system/`

---

## Phase 2: Foundational Backend Contract and Gate

**Purpose**: Publish and verify the API behavior that blocks all frontend user
stories.

**CRITICAL**: No frontend implementation task can begin until this phase is
complete.

### Contract Tests and Updates

- [X] T005 [P] Add OpenAPI schema coverage notes for zero to two guardian entries, new guardian mode, existing guardian mode, duplicate relationship-label acceptance, and atomic response expectations in `schoolmaster-specs/specs/040-student-guardian-tabs/contracts/student-guardian-tabs-contract.md`
- [X] T006 Update `StudentProfileCreateRequest` to cap `guardian_associations` at two entries and reference the expanded guardian entry schema in `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfileCreateRequest.yaml`
- [X] T007 Add or update the guardian entry schema with exactly-one-mode validation fields for existing `guardian_id` or new guardian identity/contact data in `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfileGuardianInput.yaml`
- [X] T008 [P] Add reusable new-guardian payload schema for full name, contact email, and contact phone in `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfileNewGuardianInput.yaml`
- [X] T009 Update `createStudentProfile` response and validation documentation for maximum guardians, duplicate existing guardian, duplicate relationship-label acceptance, missing permission, cross-school reference, inactive guardian, and atomic rollback outcomes, and update `StudentProfile.guardian_associations` to use an association-level response schema such as `StudentProfileGuardianAssociation` in `schoolmaster-specs/api/paths/student-profiles/index.yaml`, `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfile.yaml`, and `schoolmaster-specs/api/components/schemas/student-profiles/StudentProfileGuardianAssociation.yaml`
- [X] T010 Run Redocly lint and bundle for the changed OpenAPI contract and record results in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`

### Backend Tests

- [X] T011 [P] Add PHPUnit feature coverage for `createStudentProfile` with zero guardians, one new guardian, two new guardians, and created association response shape in `schoolmaster-backend/tests/Feature/Api/V1/StudentProfileCreateWithGuardiansTest.php`
- [X] T012 [P] Add PHPUnit feature coverage for existing same-school guardian linking, mixed new/existing entries, association-label response values that can differ from guardian-level metadata, duplicate existing guardian rejection, and duplicate relationship-label acceptance in `schoolmaster-backend/tests/Feature/Api/V1/StudentProfileCreateWithExistingGuardiansTest.php`
- [X] T013 [P] Add PHPUnit feature coverage for maximum two guardians, invalid mode combinations, missing relationship, invalid contact, inactive guardian, deleted guardian, cross-school guardian, tenant mismatch, and missing guardian management authority in `schoolmaster-backend/tests/Feature/Api/V1/StudentProfileGuardianValidationTest.php`
- [X] T014 [P] Add PHPUnit unit coverage for guardian entry DTO normalization, exactly-one-mode validation support, and duplicate detection in `schoolmaster-backend/tests/Unit/StudentProfiles/StudentProfileGuardianEntryDataTest.php`
- [X] T015 Add PHPUnit transaction coverage proving no student, guardian, or association persists when any submitted guardian entry fails in `schoolmaster-backend/tests/Feature/Api/V1/StudentProfileCreateAtomicityTest.php`

### Backend Implementation

- [X] T016 Add guardian entry DTOs for new guardian and existing guardian modes in `schoolmaster-backend/app/DTOs/StudentProfiles/StudentProfileGuardianEntryData.php` and `schoolmaster-backend/app/DTOs/StudentProfiles/NewGuardianData.php`
- [X] T017 Update the student profile create Form Request to validate zero to two guardian entries, exactly one guardian mode, required relationship type, new guardian fields, existing guardian UUIDs, and duplicate existing guardian references in `schoolmaster-backend/app/Http/Requests/StudentProfiles/CreateStudentProfileRequest.php`
- [X] T018 Update student profile policy authorization so guardian entries require guardian management authority while zero-guardian student creation remains permitted in `schoolmaster-backend/app/Policies/StudentProfilePolicy.php`
- [X] T019 Update student profile creation service to resolve tenant context first, validate existing guardians as active same-school records, create new guardians, create associations, allow duplicate relationship labels, and wrap student plus guardian work in one transaction in `schoolmaster-backend/app/Services/StudentProfiles/StudentProfileCreationService.php`
- [X] T020 Update or add guardian association validator helpers for duplicate guardian reference handling and same-school active guardian checks in `schoolmaster-backend/app/Services/StudentProfiles/GuardianAssociationValidator.php`
- [X] T021 Update student profile API resource output to expose created student and zero to two association-level guardian response records with relationship/status values through the documented response shape in `schoolmaster-backend/app/Http/Resources/StudentProfiles/StudentProfileResource.php`
- [X] T022 Run the focused backend test files and full `php artisan test`, then record backend gate results in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`

**Checkpoint**: OpenAPI and backend gate passed. Frontend implementation may
begin only after T022 is complete.

---

## Phase 3: User Story 1 - Remove Standalone Guardian Navigation (Priority: P1) MVP

**Goal**: The administration sidebar no longer shows a standalone Guardians
item, and direct guardian administration routes redirect to Create Student
without loading guardian list data.

**Independent Test**: Sign in as an authorized administrator, open the
administration shell, confirm no Guardians sidebar item appears for any
permission set, and open former guardian routes to confirm redirect to Create
Student.

### Tests for User Story 1

- [X] T023 [P] [US1] Add Vitest route metadata coverage proving the Guardians sidebar item is absent for all permission combinations in `schoolmaster-frontend/tests/unit/student-guardian-tabs/routes/guardianNavigation.routes.spec.js`
- [X] T024 [P] [US1] Add Vitest route coverage proving former guardian list, create, detail, lifecycle, and bulk routes redirect to Create Student in `schoolmaster-frontend/tests/unit/student-guardian-tabs/routes/guardianRedirect.routes.spec.js`
- [X] T025 [P] [US1] Add Vitest service-spy coverage proving redirected guardian routes do not call guardian list, detail, create, update, lifecycle, or bulk services in `schoolmaster-frontend/tests/unit/student-guardian-tabs/routes/guardianRedirectNoLoad.routes.spec.js`

### Implementation for User Story 1

- [X] T026 [US1] Remove visible Guardians sidebar metadata while retaining safe route records needed for redirect behavior in `schoolmaster-frontend/src/router/modules/guardians.routes.js`
- [X] T027 [US1] Add redirect handling from former standalone guardian administration routes to Create Student in `schoolmaster-frontend/src/router/modules/guardians.routes.js`
- [X] T028 [US1] Update administration route assembly so hidden guardian routes do not appear in sidebar navigation lists in `schoolmaster-frontend/src/router/modules/administration.routes.js`
- [X] T029 [US1] Add or update localized redirect and navigation fallback text for former guardian routes and register the locale module in `schoolmaster-frontend/src/locales/student-guardian-tabs.js` and `schoolmaster-frontend/src/main.js`

**Checkpoint**: User Story 1 is independently functional and testable as MVP.

---

## Phase 4: User Story 2 - Create Student With Guardian Tabs (Priority: P2)

**Goal**: Create Student becomes one tabbed workflow with Student and Guardians
tabs, supports new or existing same-school guardians, preserves state across
tabs, and submits only through the approved student create service.

**Independent Test**: Open Create Student with student create and guardian
management authority, switch tabs, create a new guardian or link an existing
same-school guardian, submit the combined workflow, and confirm tab-scoped
validation without data loss.

### Tests for User Story 2

- [X] T030 [P] [US2] Add Vitest contract tests for StudentCreateWorkflow, StudentDraft, GuardianEntry, NewGuardianDraft, ExistingGuardianReference, GuardianStudentAssociation, SafeFeedback, and camelCase to OpenAPI payload mapping in `schoolmaster-frontend/tests/unit/student-guardian-tabs/contracts/studentGuardianTabs.contract.spec.js`
- [X] T031 [P] [US2] Add Vitest service tests for student create payloads with zero guardians, new guardian entries, existing guardian references, mixed entries, association-level guardian response mapping, abort signal, tenant header, and safe error mapping in `schoolmaster-frontend/tests/unit/student-guardian-tabs/services/studentProfilesCreateWithGuardians.service.spec.js`
- [X] T032 [P] [US2] Add Vitest service tests for existing guardian lookup with approved list filters, active status, pagination, stale cancellation, no unsupported fields, and availability for actors with student create authority plus `guardians.manage` without standalone guardian list visibility in `schoolmaster-frontend/tests/unit/student-guardian-tabs/services/guardianLookup.service.spec.js`
- [X] T033 [P] [US2] Add Vitest composable tests for tab state, dirty state, guardian mode switching, value preservation, tab error mapping, submit pending state, and stale-response protection in `schoolmaster-frontend/tests/unit/student-guardian-tabs/composables/useStudentCreateWorkflow.spec.js`
- [X] T034 [P] [US2] Add component tests for Student tab fields, Guardians tab mode selector, new guardian fields, existing guardian selector, props-down/events-up behavior, and validation summary in `schoolmaster-frontend/tests/unit/student-guardian-tabs/components/StudentCreateTabs.spec.js`
- [X] T035 [P] [US2] Add page-flow tests for Create Student tabs, zero guardian submission, one new guardian submission, one existing guardian submission, validation failure preservation, dirty leave guard, and success state in `schoolmaster-frontend/tests/unit/student-guardian-tabs/pages/StudentCreatePage.spec.js`

### Implementation for User Story 2

- [X] T036 [US2] Define frontend contracts and mappers for StudentCreateWorkflow, GuardianEntry modes, SafeFeedback, and expanded `guardian_associations` payloads in `schoolmaster-frontend/src/contracts/admin-system/student-guardian-tabs.js`
- [X] T037 [US2] Update the student profile create service to submit zero to two new or existing guardian entries through `createStudentProfile` only in `schoolmaster-frontend/src/services/admin-system/studentProfiles.js`
- [X] T038 [US2] Add guardian lookup service support for active same-school existing guardian selection using only the limited `listGuardians` filters and pagination allowed for student create authority plus `guardians.manage` in `schoolmaster-frontend/src/services/admin-system/guardians.js`
- [X] T039 [US2] Implement route-local tabbed workflow orchestration with minimal reactive state, computed tab errors, explicit actions, lookup cancellation, submit cancellation, and dirty guard state in `schoolmaster-frontend/src/composables/admin-system/useStudentCreateWorkflow.js`
- [X] T040 [P] [US2] Implement Student tab presentational component with approved student fields, field errors, pending state, and submit/cancel emits in `schoolmaster-frontend/src/components/admin-system/students/StudentCreateStudentTab.vue`
- [X] T041 [P] [US2] Implement Guardians tab presentational component with new/existing mode controls, guardian lookup UI, validation summary, selected guardian retention, and no transport logic in `schoolmaster-frontend/src/components/admin-system/students/StudentCreateGuardiansTab.vue`
- [X] T042 [P] [US2] Implement reusable guardian entry editor with stable entry keys, relationship field, new guardian fields, existing guardian selection, remove action, and props/emits contract in `schoolmaster-frontend/src/components/admin-system/students/GuardianEntryEditor.vue`
- [X] T043 [US2] Compose the Create Student page with Student and Guardians tabs, workflow composable, permission-aware guardian capture, existing-guardian lookup availability for student create authority plus `guardians.manage`, field focus recovery, dirty leave guard, success feedback, and list return behavior in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfileCreatePage.vue`
- [X] T044 [US2] Add localized Student tab, Guardians tab, new guardian, existing guardian, lookup, validation, pending, success, and dirty-leave text in the registered locale module in `schoolmaster-frontend/src/locales/student-guardian-tabs.js` and `schoolmaster-frontend/src/main.js`

**Checkpoint**: User Stories 1 and 2 work independently after the backend gate.

---

## Phase 5: User Story 3 - Add Up To Two Guardians For One Student (Priority: P3)

**Goal**: The Guardians tab allows zero, one, or two guardian entries, blocks a
third entry, allows duplicate relationship labels, rejects duplicate guardian
identity/reference cases, and keeps no-partial-success behavior visible.

**Independent Test**: Add one guardian, add a second guardian with same or
different relationship label, confirm both stay editable, verify a third cannot
be added or submitted, and verify invalid mixed submissions do not present
partial success.

### Tests for User Story 3

- [X] T045 [P] [US3] Add Vitest composable tests for zero, one, and two guardian entry limits, third-entry blocking, duplicate relationship-label acceptance, duplicate existing guardian rejection, and selected entry removal in `schoolmaster-frontend/tests/unit/student-guardian-tabs/composables/useGuardianEntryLimit.spec.js`
- [X] T046 [P] [US3] Add component tests for add guardian button disabled state, maximum-two explanation, duplicate relationship-label acceptance, duplicate existing guardian feedback, and keyboard operation in `schoolmaster-frontend/tests/unit/student-guardian-tabs/components/GuardianEntryLimit.spec.js`
- [X] T047 [P] [US3] Add page-flow tests for two guardians with same relationship label, two mixed-mode guardians, third guardian attempt, invalid second guardian rollback feedback, and no partial success messaging in `schoolmaster-frontend/tests/unit/student-guardian-tabs/pages/StudentGuardianLimitFlow.spec.js`
- [X] T048 [P] [US3] Add backend regression coverage for duplicate relationship-label acceptance, third guardian rejection, duplicate existing guardian rejection, and atomic rollback in `schoolmaster-backend/tests/Feature/Api/V1/StudentProfileGuardianLimitRegressionTest.php`

### Implementation for User Story 3

- [X] T049 [US3] Add explicit guardian entry limit helpers for add/remove eligibility, duplicate existing guardian detection, duplicate relationship-label acceptance, and maximum explanation state in `schoolmaster-frontend/src/composables/admin-system/useStudentCreateWorkflow.js`
- [X] T050 [US3] Update Guardians tab controls to block third guardian entry creation, keep two entries editable, and expose accessible maximum-two feedback in `schoolmaster-frontend/src/components/admin-system/students/StudentCreateGuardiansTab.vue`
- [X] T051 [US3] Update guardian entry editor validation display so duplicate relationship labels are accepted while duplicate existing guardian references and duplicate identity conflicts show field-level feedback in `schoolmaster-frontend/src/components/admin-system/students/GuardianEntryEditor.vue`
- [X] T052 [US3] Update Create Student page submission and feedback handling so rejected guardian entries preserve both tabs and never show partial success in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfileCreatePage.vue`
- [X] T053 [US3] Add maximum-two, duplicate-existing-guardian, duplicate-identity, duplicate-relationship-allowed, and no-partial-success text in the registered locale module in `schoolmaster-frontend/src/locales/student-guardian-tabs.js` and `schoolmaster-frontend/src/main.js`

**Checkpoint**: All user stories are independently functional.

---

## Phase 6: Polish and Cross-Cutting Validation

**Purpose**: Validate contract, backend, frontend, accessibility, responsiveness,
privacy, and documentation across all stories.

- [X] T054 Run Redocly lint and bundle, then record exact contract results in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T055 Run focused backend PHPUnit files and full `php artisan test`, then record exact backend results in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T056 Run focused frontend Vitest files under `schoolmaster-frontend/tests/unit/student-guardian-tabs/` and record exact results in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T057 Run full frontend unit suite and production build, then record `npm run test:unit` and `npm run build` results in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T058 Run source audits for direct Axios, endpoint strings outside services, hidden Guardians sidebar metadata, former guardian route redirects, and no chained `createGuardian` use in `schoolmaster-frontend/src/` and record findings in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T059 Review Create Student tabs, guardian entry controls, redirects, validation summaries, dialogs, focus order, and keyboard operation at 390px, 768px, and 1440px in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfileCreatePage.vue` and record findings in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T060 Review backend, frontend, visible errors, logs, storage, URLs, and test output for protected student, guardian, contact, token, permission, full payload, and cross-tenant data leakage, then record findings in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T061 Update final implementation evidence for operation mapping, permission behavior, backend gate, sidebar removal, direct route redirect, zero/one/two guardian submissions, duplicate relationship-label acceptance, third guardian rejection, atomic rollback, stale response handling, and no-sensitive-data diagnostics in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T062 Run at least one timed usability check using an authorized administrator or administrator-proxy account for opening Create Student, finding the Guardians tab, and creating a student with two guardians, then record tester role, timing method, results, and blockers in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`
- [X] T063 Run focused performance checks for Create Student initial route readiness, existing-guardian lookup latency, and duplicate-submit prevention under normal admin test conditions, then record timings and pass/fail evidence in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`

---

## Phase 7: User Story 4 - Student Detail With Student and Guardian Tabs (Priority: P2)

**Goal**: The Student Profile detail page presents Student and Guardians tabs;
the Guardians tab renders zero to two read-only guardian associations from the
existing documented `getStudentProfile.guardian_associations` response, with no
guardian edit, lifecycle, or association-management controls.

**Independent Test**: Open an existing student detail page with guardian
associations, switch between Student and Guardians tabs, confirm read-only
guardian fields, and confirm the empty state for a student without guardians.

### Tests for User Story 4

- [X] T064 [P] [US4] Add Vitest page coverage proving the Student Profile detail page renders Student and Guardians tabs and shows existing guardian associations in the Guardians tab in `schoolmaster-frontend/tests/unit/student-guardian-tabs/pages/StudentProfileDetailPage.spec.js`
- [X] T065 [P] [US4] Add Vitest component coverage for guardian association display fields, formatted phone, status rendering, and empty state in `schoolmaster-frontend/tests/unit/student-guardian-tabs/components/StudentGuardianAssociationsPanel.spec.js`

### Implementation for User Story 4

- [X] T066 [US4] Implement read-only guardian associations panel with relationship, full name, contact email, contact phone, status, and empty state in `schoolmaster-frontend/src/components/admin-system/students/StudentGuardianAssociationsPanel.vue`
- [X] T067 [US4] Convert the Student Profile detail page into Student and Guardians tabs using the existing detail record `guardianAssociations` in `schoolmaster-frontend/src/pages/admin-system/students/StudentProfileDetailPage.vue`
- [X] T068 [US4] Run focused Vitest files, eslint on changed frontend files, and production build, then record results in `schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`

**Checkpoint**: User Story 4 is independently functional after the Phase 2
backend gate; no backend or OpenAPI change is required.

---

## Dependencies and Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; can start immediately.
- **Foundational Backend Contract and Gate (Phase 2)**: Depends on Setup
  completion; blocks all frontend user stories.
- **User Stories (Phase 3+)**: Depend on Phase 2 backend gate completion.
- **Polish (Phase 6)**: Depends on all desired user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Starts after Phase 2; MVP sidebar and redirect change.
- **User Story 2 (P2)**: Starts after Phase 2; integrates with existing Create
  Student page and can be tested with direct route access even if US1 is not
  complete.
- **User Story 3 (P3)**: Starts after Phase 2 and after US2 workflow scaffolding
  exists; its limit and duplicate handling remain independently testable in the
  composable and page tests.
- **User Story 4 (P2)**: Starts after Phase 2; consumes the existing
  `getStudentProfile.guardian_associations` contract and requires no additional
  backend or OpenAPI gate.

### Within Each User Story

- Write Vitest or PHPUnit coverage first and confirm it fails before
  implementation.
- Contract and backend tests precede backend implementation.
- Backend implementation and verification precede frontend implementation.
- Services and mappers precede composables.
- Composables precede route pages.
- Components use props down and events up; no direct HTTP calls.
- Route pages compose components and composables; no direct Axios calls.
- Localized text is updated before page-flow verification.

### Parallel Opportunities

- T002, T003, and T004 can run in parallel.
- T005, T008, T011, T012, T013, and T014 can run in parallel after setup.
- T023, T024, and T025 can run in parallel after the backend gate.
- T030 through T035 can run in parallel after the backend gate.
- T040, T041, and T042 can run in parallel after T036 through T039.
- T045 through T048 can run in parallel after US2 workflow scaffolding exists.
- T054 through T063 run after implementation is complete; serialize each `quickstart.md` evidence update.
- T064 and T065 can run in parallel after the Phase 2 backend gate.

---

## Parallel Example: User Story 1

```bash
Task: "T023 Add route metadata coverage in schoolmaster-frontend/tests/unit/student-guardian-tabs/routes/guardianNavigation.routes.spec.js"
Task: "T024 Add guardian route redirect coverage in schoolmaster-frontend/tests/unit/student-guardian-tabs/routes/guardianRedirect.routes.spec.js"
Task: "T025 Add no-load service-spy coverage in schoolmaster-frontend/tests/unit/student-guardian-tabs/routes/guardianRedirectNoLoad.routes.spec.js"
```

## Parallel Example: User Story 2

```bash
Task: "T030 Add workflow contract tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/contracts/studentGuardianTabs.contract.spec.js"
Task: "T031 Add student create service tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/services/studentProfilesCreateWithGuardians.service.spec.js"
Task: "T032 Add guardian lookup service tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/services/guardianLookup.service.spec.js"
Task: "T033 Add workflow composable tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/composables/useStudentCreateWorkflow.spec.js"
Task: "T034 Add tab component tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/components/StudentCreateTabs.spec.js"
Task: "T035 Add Create Student page-flow tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/pages/StudentCreatePage.spec.js"
```

## Parallel Example: User Story 3

```bash
Task: "T045 Add entry-limit composable tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/composables/useGuardianEntryLimit.spec.js"
Task: "T046 Add entry-limit component tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/components/GuardianEntryLimit.spec.js"
Task: "T047 Add student guardian limit page-flow tests in schoolmaster-frontend/tests/unit/student-guardian-tabs/pages/StudentGuardianLimitFlow.spec.js"
Task: "T048 Add backend limit regression coverage in schoolmaster-backend/tests/Feature/StudentProfiles/StudentProfileGuardianLimitRegressionTest.php"
```

---

## Implementation Strategy

### MVP First

1. Complete Phase 1 setup.
2. Complete Phase 2 OpenAPI and backend gate.
3. Complete Phase 3 User Story 1.
4. Stop and validate sidebar removal plus direct route redirect.

### Incremental Delivery

1. Add User Story 1 to remove visible standalone guardian entry points.
2. Add User Story 2 to make Create Student the combined student/guardian
   workflow.
3. Add User Story 3 to harden two-guardian limit, duplicate handling, and
   no-partial-success behavior.
4. Finish cross-cutting verification, timed usability checks, performance checks, and evidence.

### Backend-First Gate

Frontend work begins only after OpenAPI changes, backend validation, backend
transaction behavior, and PHPUnit coverage pass and are recorded in
`schoolmaster-specs/specs/040-student-guardian-tabs/quickstart.md`.

## Notes

- Every implementation repository uses branch `040-student-guardian-tabs`.
- Keep backend controllers thin and place workflow logic in services.
- Use DTOs for guardian entry normalization because the input has multiple
  modes and fields.
- Do not add repositories unless implementation discovers complex shared data
  access.
- Keep Vue route pages thin, use Composition API and `<script setup>`, isolate
  API access in services, and keep tab state route-local.
- Commit after each task or logical group.
