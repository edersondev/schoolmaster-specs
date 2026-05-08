---

description: "Task list template for feature implementation"
---

# Tasks: [FEATURE NAME]

**Input**: Design documents from `/specs/[###-feature-name]/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Tests**: Critical business flow tests are REQUIRED. Include PHPUnit, Vitest,
and API contract tasks whenever the feature changes backend behavior,
frontend behavior, or REST contracts. Add extra test tasks whenever the
specification explicitly requests broader coverage.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- **Backend repository**: `app/`, `routes/`, `tests/Feature/`, `tests/Unit/`
- **Frontend repository**: `src/modules/`, `src/router/`, `src/services/`,
  `src/stores/`, `tests/`
- **Contracts**: OpenAPI and feature artifacts under `specs/` or the designated
  contract location from plan.md
- Paths shown below assume separate backend and frontend repositories - adjust
  to the concrete structure captured in plan.md

<!-- 
  ============================================================================
  IMPORTANT: The tasks below are SAMPLE TASKS for illustration purposes only.
  
  The /speckit-tasks command MUST replace these with actual tasks based on:
  - User stories from spec.md (with their priorities P1, P2, P3...)
  - Feature requirements from plan.md
  - Entities from data-model.md
  - Endpoints from contracts/
  
  Tasks MUST be organized by user story so each story can be:
  - Implemented independently
  - Tested independently
  - Delivered as an MVP increment
  
  DO NOT keep these sample tasks in the generated tasks.md file.
  ============================================================================
-->

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Contract definition, repository targeting, and basic structure

- [ ] T001 Update or create the OpenAPI contract for the affected `/api/v1` scope
- [ ] T002 Identify backend, frontend, and specification repository targets per implementation plan
- [ ] T003 [P] Configure project tooling and quality gates required by each repository

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

Examples of foundational tasks (adjust based on your project):

- [ ] T004 Implement Laravel route, controller, request, resource, policy, and service scaffolding for the feature
- [ ] T005 [P] Establish tenant-scoping, UUID, cross-tenant access rules, and soft-delete support for affected entities
- [ ] T006 [P] Setup Vue module, router, store, and service scaffolding for the feature
- [ ] T007 Create shared models, DTOs, or entities that all stories depend on
- [ ] T008 Configure authentication, authorization, error handling, and consistent JSON response behavior
- [ ] T009 Setup cross-repository traceability and contract validation workflows

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 1 - [Title] (Priority: P1) 🎯 MVP

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 1 (REQUIRED for critical flows) ⚠️

> **NOTE: Write these tests FIRST, ensure they FAIL before implementation**

- [ ] T010 [P] [US1] Contract verification for the changed `/api/v1` endpoint in the designated OpenAPI or contract test location
- [ ] T011 [P] [US1] Backend feature or unit test for the user journey in `tests/Feature/` or `tests/Unit/`
- [ ] T012 [P] [US1] Frontend Vitest coverage for the impacted service, store, or composable in the frontend repository

### Implementation for User Story 1

- [ ] T013 [P] [US1] Create or update Laravel models and UUID-ready persistence for affected entities
- [ ] T014 [US1] Implement backend business logic in the appropriate Service Layer class and supporting DTO or Repository code when required
- [ ] T015 [US1] Implement Form Request, Policy, API Resource, and `/api/v1` controller wiring
- [ ] T016 [US1] Implement frontend service integration and module-level state updates
- [ ] T017 [US1] Implement the user-facing Vue module or route changes without direct component HTTP calls
- [ ] T018 [US1] Add validation, error handling, tenant-safety checks, and repository coordination notes

**Checkpoint**: At this point, User Story 1 should be fully functional and testable independently

---

## Phase 4: User Story 2 - [Title] (Priority: P2)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 2 (REQUIRED for critical flows) ⚠️

- [ ] T019 [P] [US2] Contract verification for the changed `/api/v1` endpoint
- [ ] T020 [P] [US2] Backend PHPUnit coverage for the user journey
- [ ] T021 [P] [US2] Frontend Vitest coverage for the impacted service, store, or composable

### Implementation for User Story 2

- [ ] T022 [P] [US2] Create or update affected Laravel entities and supporting persistence
- [ ] T023 [US2] Implement backend service and HTTP-layer changes for the story
- [ ] T024 [US2] Implement frontend service and module updates for the story
- [ ] T025 [US2] Integrate with User Story 1 behavior while preserving independent testability

**Checkpoint**: At this point, User Stories 1 AND 2 should both work independently

---

## Phase 5: User Story 3 - [Title] (Priority: P3)

**Goal**: [Brief description of what this story delivers]

**Independent Test**: [How to verify this story works on its own]

### Tests for User Story 3 (REQUIRED for critical flows) ⚠️

- [ ] T026 [P] [US3] Contract verification for the changed `/api/v1` endpoint
- [ ] T027 [P] [US3] Backend PHPUnit coverage for the user journey
- [ ] T028 [P] [US3] Frontend Vitest coverage for the impacted service, store, or composable

### Implementation for User Story 3

- [ ] T029 [P] [US3] Create or update affected Laravel entities and supporting persistence
- [ ] T030 [US3] Implement backend service and HTTP-layer changes for the story
- [ ] T031 [US3] Implement frontend service and module updates for the story

**Checkpoint**: All user stories should now be independently functional

---

[Add more user story phases as needed, following the same pattern]

---

## Phase N: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] TXXX [P] Documentation and OpenAPI updates across affected repositories
- [ ] TXXX Code cleanup and refactoring
- [ ] TXXX Performance optimization across all stories
- [ ] TXXX [P] Additional PHPUnit or Vitest coverage for non-critical supporting behavior
- [ ] TXXX Security hardening
- [ ] TXXX Run quickstart.md validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 → P2 → P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational (Phase 2) - No dependencies on other stories
- **User Story 2 (P2)**: Can start after Foundational (Phase 2) - May integrate with US1 but should be independently testable
- **User Story 3 (P3)**: Can start after Foundational (Phase 2) - May integrate with US1/US2 but should be independently testable

### Within Each User Story

- Contract, backend, and frontend tests for critical flows MUST be written and fail before implementation
- Models and persistence changes before services
- Services before controllers, resources, and frontend integration
- Core implementation before cross-story integration
- Story complete before moving to next priority

### Parallel Opportunities

- All Setup tasks marked [P] can run in parallel
- All Foundational tasks marked [P] can run in parallel (within Phase 2)
- Once Foundational phase completes, all user stories can start in parallel (if team capacity allows)
- All tests for a user story marked [P] can run in parallel
- Models within a story marked [P] can run in parallel
- Different user stories can be worked on in parallel by different team members

---

## Parallel Example: User Story 1

```bash
# Launch all tests for User Story 1 together:
Task: "Contract verification for the changed /api/v1 endpoint"
Task: "Backend PHPUnit coverage for the user journey"
Task: "Frontend Vitest coverage for the impacted service, store, or composable"

# Launch all persistence work for User Story 1 together:
Task: "Create or update the first affected Laravel entity"
Task: "Create or update the second affected Laravel entity"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 1
4. **STOP and VALIDATE**: Test User Story 1 independently
5. Deploy/demo if ready

### Incremental Delivery

1. Complete Setup + Foundational → Foundation ready
2. Add User Story 1 → Test independently → Deploy/Demo (MVP!)
3. Add User Story 2 → Test independently → Deploy/Demo
4. Add User Story 3 → Test independently → Deploy/Demo
5. Each story adds value without breaking previous stories

### Parallel Team Strategy

With multiple developers:

1. Team completes Setup + Foundational together
2. Once Foundational is done:
   - Developer A: User Story 1
   - Developer B: User Story 2
   - Developer C: User Story 3
3. Stories complete and integrate independently

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Verify required critical-flow tests fail before implementing
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- Avoid: vague tasks, same file conflicts, cross-story dependencies that break independence
