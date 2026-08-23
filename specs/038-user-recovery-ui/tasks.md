# Tasks: User Recovery UI

**Input**: Design documents from `specs/038-user-recovery-ui/`  
**Prerequisites**: `plan.md`, `spec.md`, `research.md`, `data-model.md`,
`contracts/user-recovery-ui-contract.md`, `quickstart.md`

**Tests**: Service, composable, component, mounted-page, and complete-workflow
tests are required by the specification. Tests are written before their
corresponding implementation and must demonstrate the intended failure before
the implementation task begins.

**Organization**: Tasks are grouped by user story so each story has an explicit
goal and independent verification boundary.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: May run in parallel because it changes a different file and has no
  dependency on an incomplete task in the same phase.
- **[Story]**: Maps the task to User Story 1, 2, or 3.
- Every task includes an exact repository-relative file path.

## Path Conventions

- Run specification tasks from `schoolmaster-specs`.
- Run frontend implementation and frontend tests from `schoolmaster-frontend`.
- Feature artifacts live in `specs/038-user-recovery-ui/` in the specs
  repository.
- Frontend production paths use `src/`; unit tests use `tests/unit/`; browser
  workflows use `e2e/`.
- No backend source, migration, route, or OpenAPI implementation task is needed;
  Feature 038 consumes the unchanged Feature 037 contract.

## Delivery Ownership & Traceability

- `schoolmaster-specs` leads delivery and must approve Feature 038 before any
  frontend implementation task begins.
- `schoolmaster-frontend` follows on a Feature 038 branch or pull request that
  links the approved specification change.
- `schoolmaster-backend` remains a verified prerequisite and changes only if
  contract verification finds an existing defect.

## Phase 1: Setup (Shared Verification and Fixtures)

**Purpose**: Confirm the published dependency and create shared safe fixtures
before changing frontend behavior.

- [ ] T001 Confirm Feature 038 specification approval, record the linked specs and frontend branch or pull request identifiers, verify the unchanged `createUser`, `restoreUser`, and `getUser` contract with Redocly, and record the gate result in `specs/038-user-recovery-ui/quickstart.md`
- [ ] T002 Create reusable exact, malformed, generic, and status-matrix response fixtures after T001 passes in `tests/unit/admin-system/user-recovery/fixtures/recoveryFeedback.js`

---

## Phase 2: Foundational (Blocking Request Safety)

**Purpose**: Make create and lifecycle requests safely invalidatable before any
recovery reference can control the page.

**⚠️ CRITICAL**: No user story implementation begins until this phase passes.

- [ ] T003 [P] Add failing reset/context-change and late-resolution tests to `tests/unit/admin-system/administration/composables/useAdminCreateForm.spec.js`
- [ ] T004 Implement request-generation invalidation, pending cleanup, and stale-result suppression in `src/composables/admin-system/useAdminCreateForm.js`
- [ ] T005 [P] Add failing close/reset/in-flight invalidation tests to `tests/unit/admin-system/administration-lifecycle/composables/useAdminLifecycleAction.spec.js`
- [ ] T006 Implement an explicit invalidate/reset action that advances the request sequence and makes late lifecycle results inert in `src/composables/admin-system/useAdminLifecycleAction.js`
- [ ] T007 [P] Extend restore service regression coverage for method, path, request body, and `X-School-Id` in `tests/unit/admin-system/administration/services/users.spec.js`
- [ ] T008 [P] Add the internal `restore-user` recovery action constant without changing existing actions in `src/contracts/admin-system/administration.js`

**Checkpoint**: Create and lifecycle promises are single-flight and stale-safe;
the existing restore service contract is verified.

---

## Phase 3: User Story 1 - Recognize a Recoverable Identity (Priority: P1) 🎯 MVP

**Goal**: Show one safe, localized, accessible recovery warning and action only
for the exact valid recoverable conflict, without rendering the recovery UUID,
backend message, or retained-user details.

**Independent Test**: Submit a create request returning the exact eligible
conflict. Verify fixed localized guidance and one restore action, no generic
assertive conflict, no hidden identity data, unchanged focus, natural keyboard
order, preserved creation draft, and immediate clearing after any email edit.

### Tests for User Story 1

- [ ] T009 [P] [US1] Add failing exact-response projection, valid-UUID, fixed-copy key, and raw-message/details exclusion tests to `tests/unit/admin-system/administration/services/administration-error-mapper.spec.js`
- [ ] T010 [P] [US1] Add failing safe-copy, props/emits, polite atomic status, no nested alert, unchanged-focus, normal-tab-order, and no-identifier tests in `tests/unit/admin-system/user-recovery/components/UserRecoveryAlert.spec.js`
- [ ] T011 [P] [US1] Add failing mounted-page tests for exact warning rendering, generic-feedback suppression, no automatic restore, draft preservation, and email-edit clearing in `tests/unit/admin-system/user-recovery/pages/CreateUserRecovery.spec.js`

### Implementation for User Story 1

- [ ] T012 [US1] Implement exact `409` + `recoverable_user_conflict` + valid UUID + `restore` classification and minimal safe projection in `src/services/admin-system/administration-error-mapper.js`
- [ ] T013 [P] [US1] Add fixed localized warning, restore action, generic lifecycle resource label, and recovery feedback keys in `src/locales/administration.js`
- [ ] T014 [US1] Implement the presentation-only polite atomic status region with semantic HTML, Tailwind, a normal `ElButton`, and one `restore` emit in `src/components/admin-system/users/UserRecoveryAlert.vue`
- [ ] T015 [US1] Implement readonly transient recovery acceptance, context snapshot, visibility, and explicit clear actions without persistence or discovery in `src/composables/admin-system/useUserCreationRecovery.js`
- [ ] T016 [US1] Compose exact recovery feedback and `UserRecoveryAlert` into the create form, suppress duplicate assertive feedback, preserve the draft, and clear recovery on email edit in `src/pages/admin-system/users/CreateUserPage.vue`

**Checkpoint**: User Story 1 independently distinguishes valid recoverable
feedback from the previous generic conflict and exposes no retained identity.

---

## Phase 4: User Story 2 - Restore and Continue Maintenance (Priority: P2)

**Goal**: Reuse lifecycle confirmation for an explicit same-school restore,
apply the exact retryable/terminal failure policy, discard the create draft on
success, and continue to authoritative school-mode detail without repeating
restore.

**Independent Test**: From a valid warning, open the existing confirmation,
validate reason/date, submit one restore in the original school, verify the
exact failure-disposition matrix, and on success verify draft discard and
`userDetail?user_mode=school`. If the detail GET fails, remain on that route and
retry GET only while restore call count stays one.

### Tests for User Story 2

- [ ] T017 [P] [US2] Add failing restore orchestration, single-flight, same-context, local/0/408/422/429/5xx preserve, 401/403/404/409/other-HTTP clear, and safe-terminal-feedback tests in `tests/unit/admin-system/user-recovery/composables/useUserCreationRecovery.spec.js`
- [ ] T018 [P] [US2] Add failing mounted-page tests for lifecycle dialog reuse, fixed generic label, reason/date submission, exact failure handling, draft discard, and explicit school-mode detail navigation in `tests/unit/admin-system/user-recovery/pages/CreateUserRecovery.spec.js`
- [ ] T019 [P] [US2] Add failing post-restore detail-load tests proving the detail route remains active, normal retry/return feedback is used, and retry invokes only `getUser` in `tests/unit/admin-system/user-recovery/pages/UserRecoveryDetailFailure.spec.js`
- [ ] T020 [P] [US2] Create a failing stateful conflict-to-confirmation-to-restore-to-detail browser journey that asserts no more than two deliberate actions from warning to completed confirmation, excluding reason/date entry, and enforces one restore call in `e2e/user-recovery.spec.js`

### Implementation for User Story 2

- [ ] T021 [US2] Extend the recovery composable with lifecycle-dialog orchestration, exact numeric-status disposition, safe terminal feedback retention, deliberate retry, and same-context restore submission in `src/composables/admin-system/useUserCreationRecovery.js`
- [ ] T022 [US2] Wire the existing `restoreUser` service and `AdminLifecycleDialog` to the recovery coordinator with the original school context and no pre-restore detail lookup in `src/pages/admin-system/users/CreateUserPage.vue`
- [ ] T023 [US2] Reset the failed create draft, invalidate recovery, and navigate once to named `userDetail` with `user_mode=school` after current restore success in `src/pages/admin-system/users/CreateUserPage.vue`

**Checkpoint**: User Story 2 completes the recommended recovery flow. Restore
is explicit and single-flight; later maintenance remains separate.

---

## Phase 5: User Story 3 - Preserve Generic Duplicate Privacy (Priority: P3)

**Goal**: Keep every generic, malformed, unauthorized, unsupported,
cross-context, and stale duplicate outcome non-disclosing and non-recoverable.

**Independent Test**: Exercise generic unavailable-email validation, wrong code,
flat body, malformed UUID, missing/wrong action, extra fields, repeated submit,
email/school/session/permission/route changes, and stale in-flight results.
Verify no restore action, lookup, identifier, hidden details, persistence,
cross-tenant request, feedback replacement, or navigation survives invalidation.

### Tests for User Story 3

- [ ] T024 [P] [US3] Add failing generic, flat-body, wrong-code, malformed-UUID, missing/wrong-action, extra-field, and recovery-field-stripping tests to `tests/unit/admin-system/administration/services/administration-error-mapper.spec.js`
- [ ] T025 [P] [US3] Add failing email/school/actor/session/permission/route/cancel/newer-request invalidation and stale-create/restore tests to `tests/unit/admin-system/user-recovery/composables/useUserCreationRecovery.spec.js`
- [ ] T026 [P] [US3] Add failing mounted-page privacy tests for generic duplicate feedback, no discovery request, context changes, stale feedback suppression, and no route/storage/DOM identifier exposure in `tests/unit/admin-system/user-recovery/pages/CreateUserRecovery.spec.js`
- [ ] T027 [P] [US3] Extend browser coverage for generic and malformed privacy fallbacks, repeated submission, context switches, route departure, and stale in-flight responses in `e2e/user-recovery.spec.js`

### Implementation for User Story 3

- [ ] T028 [US3] Enforce safe generic fallback with no recovery fields for every malformed, unsupported, flat, or non-allowlisted create conflict in `src/services/admin-system/administration-error-mapper.js`
- [ ] T029 [US3] Implement cancellation and email/school/actor/session/permission/route/newer-request invalidation with safe-default clearing for non-allowlisted HTTP outcomes in `src/composables/admin-system/useUserCreationRecovery.js`
- [ ] T030 [US3] Connect administration context and route invalidation through `src/composables/admin-system/useAdministrationCreatePage.js` and `src/pages/admin-system/users/CreateUserPage.vue`, suppress stale page effects, and ensure no recovery state enters query, Pinia, storage, logs, or telemetry

**Checkpoint**: All three user stories are independently verified, and recovery
cannot weaken duplicate-email or tenant privacy.

---

## Phase 6: Polish & Cross-Cutting Verification

**Purpose**: Protect adjacent invitation behavior, finish browser accessibility
coverage, and collect release evidence.

- [ ] T031 [P] Add regression coverage proving successful invited-user creation and the existing invitation continuation still work in `tests/unit/account-lifecycle/pages/CreateUserAccountInvitation.test.js`
- [ ] T032 [P] Extend the recovery workflow for 390/768/1440 layouts, polite live announcement, unchanged focus, natural keyboard activation, dialog keyboard use, and no accessible UUID in `e2e/user-recovery.spec.js`
- [ ] T033 Run Redocly, focused and full Vitest, the recovery Playwright workflow, and the production build, then record commands and results in `specs/038-user-recovery-ui/quickstart.md`
- [ ] T034 Run the documented privacy/architecture source audits plus manual responsive, live-region, focus, and keyboard review, then record evidence in `specs/038-user-recovery-ui/quickstart.md`
- [ ] T035 Conduct moderated acceptance with a preselected cohort of at least 10 authorized administrators, count a participant as passing only when they independently identify restore-versus-recreate behavior and select `Restore existing user` as the next action, calculate the pass percentage and require at least 90%, and record privacy-safe evidence in `specs/038-user-recovery-ui/quickstart.md`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: T001 starts immediately and blocks T002 and every frontend
  implementation or test task until specification approval and contract evidence
  are recorded.
- **Foundational (Phase 2)**: Depends on Setup and blocks every user story.
- **User Story 1 (Phase 3)**: Depends on Foundation.
- **User Story 2 (Phase 4)**: Depends on User Story 1 because it extends the
  warning and route-local coordinator with restore behavior.
- **User Story 3 (Phase 5)**: Depends on User Story 2 because it invalidates both
  create and restore workflows, including in-flight restoration.
- **Polish (Phase 6)**: Depends on all selected user stories.
- **Moderated acceptance (T035)**: Depends on the technical and manual release
  evidence in T033 and T034 so participants evaluate the release candidate.

### User Story Dependency Graph

```text
Setup
  -> Foundation
     -> US1: Recognize recoverable identity
        -> US2: Restore and continue maintenance
           -> US3: Preserve generic duplicate privacy
              -> Polish and release evidence
```

### Within Each User Story

1. Add the story's failing tests and confirm they fail for the intended reason.
2. Implement pure contract/state behavior before page composition.
3. Implement presentational components before route integration when required.
4. Keep API access in existing services and business rules in composables.
5. Run the story's focused tests and satisfy its independent-test checkpoint.

### Parallel Opportunities

- T002 begins only after T001 passes; it is not parallel with the approval and
  contract gate.
- T003, T005, T007, and T008 touch separate foundation files and can begin in
  parallel after Setup.
- T009, T010, and T011 can be written in parallel; T013 can proceed alongside
  mapper implementation after the US1 test boundary is established.
- T017, T018, T019, and T020 are separate US2 test surfaces and can be written
  in parallel.
- T024, T025, T026, and T027 are separate US3 test surfaces and can be written
  in parallel.
- T031 and T032 can run in parallel after all stories pass.

## Parallel Example: User Story 1

```text
Parallel test work:
- T009: mapper contract tests
- T010: warning component accessibility/privacy tests
- T011: mounted create-page behavior tests

After tests establish the boundary:
- T012: exact mapper projection
- T013: localized copy (parallel, separate file)
```

## Parallel Example: User Story 2

```text
Parallel test work:
- T017: recovery composable and failure matrix
- T018: mounted restore workflow
- T019: post-restore detail failure
- T020: browser success journey
```

## Parallel Example: User Story 3

```text
Parallel test work:
- T024: malformed/generic mapper fallbacks
- T025: context and stale-request invalidation
- T026: mounted-page privacy
- T027: browser privacy and context changes
```

## Implementation Strategy

### MVP First: User Story 1

1. Complete Setup and Foundation.
2. Complete User Story 1.
3. Verify exact safe guidance, accessibility, privacy, and email-edit clearing.
4. Use this as the recognition MVP checkpoint. Do not release the restore action
   as a complete production recovery journey until User Story 2 is also done.

### Production Recovery Slice

1. Complete Setup and Foundation.
2. Deliver US1 recognition.
3. Deliver US2 confirmation, restore, and authoritative continuation.
4. Validate US1 and US2 together before enabling the action in production.

### Full Increment

1. Add US3 generic privacy and stale-context defenses.
2. Complete invitation regression, responsive/accessibility coverage, and all
   release gates.
3. Run moderated acceptance against the release candidate and record all
   evidence in `quickstart.md` before review.

## Notes

- `[P]` tasks operate on different files and have no incomplete same-phase
  dependency.
- Frontend tests are required; no new backend PHPUnit task is created because
  backend behavior is unchanged and Feature 037 is the verified prerequisite.
- `408/429/5xx` are defensive frontend transport classes, not new published
  restore responses; no response body may be assumed.
- Components stay presentational, recovery state stays route-local, and direct
  component HTTP calls are forbidden.
- Commit after each logical group and stop at every checkpoint for independent
  validation.
