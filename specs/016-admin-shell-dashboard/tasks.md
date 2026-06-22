# Tasks: System Administrator Shell and Dashboard Foundation

**Input**: Design documents from `specs/016-admin-shell-dashboard/`
**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/, quickstart.md

**Tests**: Frontend behavior changes require Vitest coverage for route metadata,
permission visibility, shell state, responsive navigation, dashboard
placeholders, quick actions, and shell-level denial states. No PHPUnit or
OpenAPI contract tests are required because this slice changes no backend
behavior and no REST contract.

**Organization**: Tasks are grouped by user story to enable independent
implementation and testing of each story.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (US1, US2, US3)
- Include exact file paths in descriptions

## Path Conventions

- Run specification tasks from the `schoolmaster-specs` repository root.
- Run implementation tasks from the `schoolmaster-frontend` repository root.
- Feature artifacts live under `specs/016-admin-shell-dashboard/` in
  `schoolmaster-specs`.
- Frontend implementation paths below are relative to `schoolmaster-frontend`.
- This feature does not modify `schoolmaster-backend` or `api/openapi.yaml`.

---

## Phase 1: Setup (Shared Frontend Shell Scope)

**Purpose**: Confirm source artifacts, target repository boundaries, and
frontend baseline before implementation starts.

- [ ] T001 Confirm active feature artifacts exist in `specs/016-admin-shell-dashboard/spec.md`, `specs/016-admin-shell-dashboard/plan.md`, `specs/016-admin-shell-dashboard/research.md`, `specs/016-admin-shell-dashboard/data-model.md`, `specs/016-admin-shell-dashboard/contracts/admin-shell-dashboard-contract.md`, and `specs/016-admin-shell-dashboard/quickstart.md`
- [ ] T002 [P] Review frontend baseline dependencies in `specs/015-frontend-architecture-baseline/spec.md`, `docs/frontend-architecture.md`, `docs/frontend-guidelines.md`, `docs/frontend-admin-system-architecture.md`, and `docs/naming-conventions.md`
- [ ] T003 [P] Confirm roadmap item 2 sequencing and placeholder-only live-data boundary in `docs/frontend-feature-roadmap.md`
- [ ] T004 Verify this feature requires no backend code and no OpenAPI change in `api/openapi.yaml`

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish shared frontend shell contracts, route metadata, and
test scaffolding that all user stories depend on.

**CRITICAL**: No user story work should begin until this phase is complete.

- [ ] T005 Create admin-system route module scaffold in `src/router/modules/admin-system.routes.js`
- [ ] T006 Create System Administrator layout shell scaffold in `src/layouts/admin-system/AdminSystemLayout.vue`
- [ ] T007 [P] Create admin-system shell state store scaffold in `src/stores/admin-system/shell.store.js`
- [ ] T008 [P] Create admin-system shell contract definitions in `src/contracts/admin-system/shell.js`
- [ ] T009 [P] Create admin-system navigation contract definitions in `src/contracts/admin-system/navigation.js`
- [ ] T010 [P] Create admin-system dashboard contract definitions in `src/contracts/admin-system/dashboard.js`
- [ ] T011 [P] Create reusable permission visibility composable scaffold in `src/composables/admin-system/useAdminShellPermissions.js`
- [ ] T012 [P] Create responsive shell state composable scaffold in `src/composables/admin-system/useAdminShellState.js`
- [ ] T013 [P] Create admin-system i18n message scaffold for shell and dashboard text in `src/locales/admin-system.js`
- [ ] T014 Create frontend Vitest fixture helpers for admin shell route metadata, permissions, and viewport states in `tests/unit/admin-system/shell.fixtures.js`

**Checkpoint**: Shared route, state, contract, composable, i18n, and test
fixtures are ready.

---

## Phase 3: User Story 1 - Navigate the Admin Shell (Priority: P1) MVP

**Goal**: A System Administrator can use one consistent admin shell with
sidebar, header, content region, page context, responsive navigation,
permission-aware hidden items, and shell-level denial states.

**Independent Test**: Render the shell with allowed and denied route metadata,
desktop/tablet/mobile viewport states, and unauthorized/forbidden/session
expired states. Confirm navigation visibility, drawer/sidebar behavior, content
region, and feedback states work without any dashboard live data or CRUD pages.

### Tests for User Story 1

- [ ] T015 [P] [US1] Add Vitest coverage for admin route metadata layout selection, title, breadcrumb, sidebar placement, and auth requirement in `tests/unit/admin-system/admin-system.routes.spec.js`
- [ ] T016 [P] [US1] Add Vitest coverage for hidden unauthorized sidebar navigation items in `tests/unit/admin-system/useAdminShellPermissions.spec.js`
- [ ] T017 [P] [US1] Add Vitest coverage for desktop/tablet collapsed sidebar and mobile overlay drawer state in `tests/unit/admin-system/useAdminShellState.spec.js`
- [ ] T018 [P] [US1] Add Vitest coverage that dashboard content and available navigation destinations are reachable in no more than two interactions in `tests/unit/admin-system/AdminSystemLayout.spec.js`
- [ ] T019 [P] [US1] Add Vitest coverage for shell-level unauthorized, forbidden, and session-expired states in `tests/unit/admin-system/AdminSystemLayout.spec.js`

### Implementation for User Story 1

- [ ] T020 [US1] Define admin shell route metadata for dashboard route, layout selection, permissions, breadcrumb, and sidebar placement in `src/router/modules/admin-system.routes.js`
- [ ] T021 [US1] Implement shell state actions for sidebar collapsed state, mobile drawer open state, active route key, and feedback state in `src/stores/admin-system/shell.store.js`
- [ ] T022 [US1] Implement permission visibility helpers that hide unauthorized or unapproved navigation entries in `src/composables/admin-system/useAdminShellPermissions.js`
- [ ] T023 [US1] Implement responsive shell state behavior that uses desktop/tablet collapsed sidebar and mobile overlay drawer in `src/composables/admin-system/useAdminShellState.js`
- [ ] T024 [US1] Implement `AdminSystemLayout.vue` with sidebar, top header, route content region, page context, global feedback placement, and no direct Axios calls in `src/layouts/admin-system/AdminSystemLayout.vue`
- [ ] T025 [US1] Implement reusable shell navigation component with hidden unauthorized items and active route state in `src/components/admin-system/shell/AdminShellSidebar.vue`
- [ ] T026 [US1] Implement reusable top header component with page context, sidebar/drawer controls, and placeholder notification affordance in `src/components/admin-system/shell/AdminShellHeader.vue`
- [ ] T027 [US1] Implement shell feedback component for loading, empty, error, forbidden, unauthorized, session-expired, tenant-mismatch, and unavailable states in `src/components/admin-system/shell/AdminShellFeedback.vue`
- [ ] T028 [US1] Add localized shell labels, feedback messages, and accessible control text in `src/locales/admin-system.js`

**Checkpoint**: User Story 1 is independently functional and testable as the
MVP shell.

---

## Phase 4: User Story 2 - Review Dashboard Status (Priority: P2)

**Goal**: A System Administrator can open the dashboard landing surface and see
placeholder-only summary, recent activity, notification, and feedback regions
without consuming live dashboard data.

**Independent Test**: Render the dashboard inside the admin shell and confirm
summary cards, recent activity, and notification regions use only placeholder,
empty, unavailable, loading, error, or absent states with no live metric,
activity, notification, or Axios usage.

### Tests for User Story 2

- [ ] T029 [P] [US2] Add Vitest coverage for placeholder-only dashboard summary card states in `tests/unit/admin-system/AdminDashboardSummary.spec.js`
- [ ] T030 [P] [US2] Add Vitest coverage for placeholder-only recent activity and notification regions in `tests/unit/admin-system/AdminDashboardPage.spec.js`
- [ ] T031 [P] [US2] Add Vitest guard coverage rejecting live dashboard data fields such as `summaryMetric`, `recentActivity`, and `notificationCount` in `tests/unit/admin-system/dashboard.contract.spec.js`

### Implementation for User Story 2

- [ ] T032 [US2] Implement dashboard page composition without direct Axios calls or live dashboard data in `src/pages/admin-system/dashboard/AdminDashboardPage.vue`
- [ ] T033 [US2] Implement placeholder summary card component with empty, unavailable, loading, and error states in `src/components/admin-system/dashboard/AdminDashboardSummaryCard.vue`
- [ ] T034 [US2] Implement placeholder recent activity component with empty, unavailable, loading, and error states in `src/components/admin-system/dashboard/AdminRecentActivityPanel.vue`
- [ ] T035 [US2] Implement placeholder notification region or absent-state indicator in `src/components/admin-system/dashboard/AdminNotificationPlaceholder.vue`
- [ ] T036 [US2] Add JSDoc placeholder dashboard contract shapes and blocked live-data notes in `src/contracts/admin-system/dashboard.js`
- [ ] T037 [US2] Add localized dashboard placeholder, empty, unavailable, loading, and error text in `src/locales/admin-system.js`

**Checkpoint**: User Story 2 is independently functional and testable as a
placeholder-only dashboard.

---

## Phase 5: User Story 3 - Launch Approved Quick Actions (Priority: P3)

**Goal**: A System Administrator can use dashboard quick actions only when the
target route and workflow are approved and the user has permission.

**Independent Test**: Render quick actions with approved route metadata,
unapproved route metadata, and missing permissions. Confirm approved actions
navigate normally and all unapproved or unauthorized actions are hidden.

### Tests for User Story 3

- [ ] T038 [P] [US3] Add Vitest coverage for hidden quick actions when permission is absent in `tests/unit/admin-system/useAdminQuickActions.spec.js`
- [ ] T039 [P] [US3] Add Vitest coverage for hidden quick actions when route or workflow approval is missing in `tests/unit/admin-system/useAdminQuickActions.spec.js`
- [ ] T040 [P] [US3] Add Vitest coverage for approved quick action navigation behavior in `tests/unit/admin-system/AdminQuickActions.spec.js`

### Implementation for User Story 3

- [ ] T041 [US3] Create quick-action visibility and approved-route filtering composable in `src/composables/admin-system/useAdminQuickActions.js`
- [ ] T042 [US3] Add QuickAction JSDoc contract with permission, route approval, workflow approval, and hidden-state fields in `src/contracts/admin-system/navigation.js`
- [ ] T043 [US3] Implement quick actions component that renders only approved and authorized actions in `src/components/admin-system/dashboard/AdminQuickActions.vue`
- [ ] T044 [US3] Integrate approved quick actions into dashboard composition in `src/pages/admin-system/dashboard/AdminDashboardPage.vue`
- [ ] T045 [US3] Add localized quick-action labels and accessible icon-only text in `src/locales/admin-system.js`

**Checkpoint**: All user stories are independently functional and testable.

---

## Phase 6: Polish & Cross-Cutting Concerns

**Purpose**: Final verification across shell, dashboard, quick actions,
contracts, and specification artifacts.

- [ ] T046 [P] Run quickstart scope and API-boundary checks from `specs/016-admin-shell-dashboard/quickstart.md`
- [ ] T047 [P] Scan frontend implementation for direct Axios usage in layouts, pages, and components with `rg "axios" src/layouts src/pages src/components`
- [ ] T048 [P] Scan frontend implementation for kebab-case Element Plus tags with `rg "<el-" src`
- [ ] T049 [P] Scan frontend implementation for blocked coming-soon quick actions and live dashboard data names with `rg "Coming soon|coming soon|notificationCount|recentActivity|summaryMetric" src`
- [ ] T050 Review WCAG 2.1 AA expectations for shell navigation, header controls, dashboard placeholders, quick actions, focus behavior, and feedback states in `src/layouts/admin-system/AdminSystemLayout.vue`
- [ ] T051 Run frontend unit test suite for admin-system shell and dashboard behavior in `tests/unit/admin-system/`
- [ ] T052 Verify follow-up frontend and contract dependencies are documented in `specs/016-admin-shell-dashboard/contracts/admin-shell-dashboard-contract.md`, `specs/016-admin-shell-dashboard/quickstart.md`, and `docs/frontend-feature-roadmap.md`
- [ ] T053 Verify `docs/frontend-feature-roadmap.md` still marks item 2 as task-generated and live-data blocked
- [ ] T054 Verify task checklist lines in `specs/016-admin-shell-dashboard/tasks.md` follow `- [ ] T### [P?] [US?] Description with file path`

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies; can start immediately.
- **Foundational (Phase 2)**: Depends on Setup completion; blocks all user stories.
- **User Story 1 (Phase 3)**: Depends on Foundational completion; MVP scope.
- **User Story 2 (Phase 4)**: Depends on Foundational completion and can run after US1 shell route/content region exists.
- **User Story 3 (Phase 5)**: Depends on Foundational completion and can run after dashboard composition exists.
- **Polish (Phase 6)**: Depends on all desired user stories being complete.

### User Story Dependencies

- **User Story 1 (P1)**: Can start after Foundational; no dependency on US2 or US3.
- **User Story 2 (P2)**: Can start after Foundational, but integration is cleaner after US1 provides the shell and route content region.
- **User Story 3 (P3)**: Can start after Foundational, but final integration needs US2 dashboard composition.

### Parallel Opportunities

- T002 and T003 can run in parallel during Setup.
- T007 through T013 can run in parallel during Foundational because they target different files.
- US1 tests T015 through T019 can run in parallel.
- US2 tests T029 through T031 can run in parallel.
- US3 tests T038 through T040 can run in parallel.
- Polish scans T046 through T049 can run in parallel.

---

## Parallel Example: User Story 1

```bash
Task: "Add Vitest coverage for admin route metadata layout selection, title, breadcrumb, sidebar placement, and auth requirement in tests/unit/admin-system/admin-system.routes.spec.js"
Task: "Add Vitest coverage for hidden unauthorized sidebar navigation items in tests/unit/admin-system/useAdminShellPermissions.spec.js"
Task: "Add Vitest coverage for desktop/tablet collapsed sidebar and mobile overlay drawer state in tests/unit/admin-system/useAdminShellState.spec.js"
Task: "Add Vitest coverage that dashboard content and available navigation destinations are reachable in no more than two interactions in tests/unit/admin-system/AdminSystemLayout.spec.js"
Task: "Add Vitest coverage for shell-level unauthorized, forbidden, and session-expired states in tests/unit/admin-system/AdminSystemLayout.spec.js"
```

## Parallel Example: User Story 2

```bash
Task: "Add Vitest coverage for placeholder-only dashboard summary card states in tests/unit/admin-system/AdminDashboardSummary.spec.js"
Task: "Add Vitest coverage for placeholder-only recent activity and notification regions in tests/unit/admin-system/AdminDashboardPage.spec.js"
Task: "Add Vitest guard coverage rejecting live dashboard data fields such as summaryMetric, recentActivity, and notificationCount in tests/unit/admin-system/dashboard.contract.spec.js"
```

## Parallel Example: User Story 3

```bash
Task: "Add Vitest coverage for hidden quick actions when permission is absent in tests/unit/admin-system/useAdminQuickActions.spec.js"
Task: "Add Vitest coverage for hidden quick actions when route or workflow approval is missing in tests/unit/admin-system/useAdminQuickActions.spec.js"
Task: "Add Vitest coverage for approved quick action navigation behavior in tests/unit/admin-system/AdminQuickActions.spec.js"
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup.
2. Complete Phase 2: Foundational.
3. Complete Phase 3: User Story 1.
4. Stop and validate the admin shell independently against US1 acceptance scenarios.

### Incremental Delivery

1. Complete Setup and Foundational scaffolding.
2. Deliver US1 as the minimum reusable admin shell.
3. Add US2 placeholder dashboard regions.
4. Add US3 approved quick actions.
5. Complete Polish checks before moving to the next frontend roadmap item.

### Parallel Team Strategy

With multiple contributors:

1. One contributor prepares route, store, contract, composable, and i18n scaffolds.
2. One contributor implements and tests the admin shell.
3. One contributor implements and tests placeholder dashboard regions after shell scaffolding is ready.
4. One contributor implements and tests quick actions after dashboard composition is ready.

---

## Notes

- [P] tasks target different files or independent review checks.
- This feature does not approve backend code, OpenAPI changes, live dashboard data, authentication screens, CRUD resource pages, reporting workflows, platform-support workflows, or TypeScript requirements.
- Frontend implementation must not call Axios directly from layouts, pages, or components.
- Future live dashboard data requires a later feature spec and approved OpenAPI contract.
