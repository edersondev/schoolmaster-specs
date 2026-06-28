# Implementation Plan: Administration Lifecycle UI

**Branch**: `020-administration-lifecycle-ui` | **Date**: 2026-06-28 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/020-administration-lifecycle-ui/spec.md`

## Summary

Implement roadmap item 5 in `schoolmaster-frontend`: administration detail,
non-status update, single-record lifecycle, soft-delete/restore, and approved
bulk lifecycle workflows for schools, users, roles, academic years, academic
periods, and guardians. Permission definitions remain read-only. The UI
consumes only the lifecycle OpenAPI operations already approved by backend
Administration Lifecycle Management, keeps edit forms separate from status
transitions, requires confirmation with today-or-past effective date and reason
for every lifecycle action, and treats bulk lifecycle as all-or-nothing.

The implementation extends the feature modules created by Administration
Foundation UI. Route pages remain thin composition surfaces. Services own Axios
calls and OpenAPI mapping. Composables coordinate detail state, update drafts,
lifecycle confirmations, bulk selection, stale-response protection, tenant
clearing, and unsaved-change guards. No backend or OpenAPI change is planned.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains out of scope.
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, and existing OpenAPI-backed administration lifecycle operations.
**Storage**: No backend or database change. Detail, update draft, lifecycle dialog, and bulk selection state remain route-local/in-memory. List state remains URL-query-driven through the existing administration foundation.
**Testing**: Vitest and Vue Test Utils for route modules, contract mappers, services, composables, status tags, action menus, confirmations, update forms, list/detail pages, bulk selection, tenant changes, permission denials, conflict handling, stale responses, and unsaved-route guards. Redocly/OpenAPI validation is required only if contract files change.
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming published `/api/v1` contracts from `schoolmaster-specs`.
**Project Type**: Frontend SPA feature with specification and cross-repository delivery artifacts.
**Performance Goals**: With the app bootstrapped and mocked detail or lifecycle services settling within 1.5 seconds, detail routes and lifecycle feedback render stable content, not-found/denial, or recoverable error within 2 seconds; stale requests are cancelled or ignored; bulk selection and row-action rendering do not block unrelated shell navigation.
**Constraints**: No undocumented endpoint, field, filter, status, action, or error dependency; no direct Axios in pages/components/router; no status field in edit forms even though update schemas publish it; no scheduled future lifecycle actions; no bulk school lifecycle; no partial-success bulk UI; no account lifecycle, invitation, password, account lock, guardian user-link, student management, or reporting workflows; no client-side tenant inference; unresolved school selection remains blocked; WCAG 2.1 AA target at 390px, 768px, and 1440px; PascalCase Element Plus tags; centralized reusable text.
**Scale/Scope**: Six lifecycle-enabled resource groups, six detail route families, six non-status update forms, single-record lifecycle controls for six resources, bulk lifecycle controls for five school-owned resources, shared status/action/confirmation/selection primitives, service and contract mapping, route metadata/navigation, and focused frontend tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. This frontend feature consumes only
  existing `get*`, `update*`, `activate*`, `deactivate*`, `delete*`,
  `restore*`, and approved `bulkLifecycle*` operation IDs. No contract change
  is planned.
- PASS: Repository impacts are separated. `schoolmaster-specs` defines this
  plan; `schoolmaster-frontend` implements it; `schoolmaster-backend` remains
  unchanged unless contract verification finds a lifecycle gap.
- PASS: Backend architecture requirements are N/A because no Laravel route,
  Request, Policy, Resource, Service, DTO, Repository, schema, or public
  identifier change is approved.
- PASS: Frontend design uses Vue 3 Composition API, Pinia only for existing
  shared session/shell state, Vue Router, Tailwind CSS, Element Plus, Axios
  service modules, feature folders, and service-isolated API access.
- PASS: MySQL and database soft-delete behavior are unchanged. UI soft-delete
  and restore controls reflect backend lifecycle contracts; tenant-owned
  resources use the authenticated active `school_id`.
- PASS: API compatibility, authentication, authorization, success envelopes,
  validation, conflict, forbidden, tenant, not-found, and lifecycle outcome
  expectations are documented.
- PASS: Vitest covers changed critical frontend flows. OpenAPI validation is
  required only if contract review creates a separate contract change.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/020-administration-lifecycle-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── administration-lifecycle-ui-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
├── api/
│   ├── openapi.yaml
│   ├── paths/
│   └── components/
├── docs/
│   ├── frontend-architecture.md
│   ├── frontend-admin-system-architecture.md
│   ├── frontend-guidelines.md
│   └── frontend-feature-roadmap.md
├── specs/020-administration-lifecycle-ui/
└── AGENTS.md

# Frontend repository target shape
schoolmaster-frontend/
├── src/
│   ├── components/
│   │   ├── ui/admin/
│   │   │   ├── AdminDetailPage.vue
│   │   │   ├── AdminStatusTag.vue
│   │   │   ├── AdminRowActions.vue
│   │   │   ├── AdminLifecycleDialog.vue
│   │   │   ├── AdminBulkActionBar.vue
│   │   │   └── AdminConflictFeedback.vue
│   │   └── admin-system/
│   │       ├── schools/
│   │       ├── users/
│   │       ├── roles/
│   │       ├── academic-years/
│   │       ├── academic-periods/
│   │       └── guardians/
│   ├── composables/admin-system/
│   │   ├── useAdminDetail.js
│   │   ├── useAdminUpdateForm.js
│   │   ├── useAdminLifecycleAction.js
│   │   ├── useAdminBulkLifecycle.js
│   │   ├── useAdminActionEligibility.js
│   │   └── useUnsavedChangesGuard.js
│   ├── contracts/admin-system/
│   │   ├── lifecycle.js
│   │   ├── schools.js
│   │   ├── users.js
│   │   ├── access.js
│   │   ├── academics.js
│   │   └── guardians.js
│   ├── locales/
│   │   └── administration-lifecycle.js
│   ├── pages/admin-system/
│   │   ├── schools/
│   │   ├── users/
│   │   ├── roles/
│   │   ├── academic-years/
│   │   ├── academic-periods/
│   │   └── guardians/
│   ├── router/modules/
│   │   ├── administration.routes.js
│   │   ├── schools.routes.js
│   │   ├── access-administration.routes.js
│   │   ├── academics.routes.js
│   │   └── guardians.routes.js
│   └── services/admin-system/
│       ├── administration-error-mapper.js
│       ├── schools.js
│       ├── users.js
│       ├── roles.js
│       ├── academic-years.js
│       ├── academic-periods.js
│       └── guardians.js
└── tests/unit/admin-system/administration-lifecycle/
    ├── contracts/
    ├── services/
    ├── composables/
    ├── components/
    ├── pages/
    └── routes/
```

**Structure Decision**: Extend the established administration feature folders
instead of creating a new top-level module. Shared lifecycle UI primitives live
under `components/ui/admin`. Resource-specific detail sections, update fields,
and action presentation live under `components/admin-system/<resource>`.
Route pages compose reusable primitives and composables. Resource services add
explicit lifecycle functions beside the existing list/create service functions.
No new Pinia resource store is planned; existing session and shell stores
continue to own authenticated user, permissions, active school, and shell
navigation.

## Component Map

- `AdminDetailPage.vue`: Frames detail header, status tag, authorized actions,
  sections, update form region, feedback, and list-return navigation.
- `AdminStatusTag.vue`: Renders lifecycle and academic status consistently
  without owning transition rules.
- `AdminRowActions.vue`: Displays permitted row/detail actions from
  eligibility input and emits user intent upward.
- `AdminLifecycleDialog.vue`: Confirms one lifecycle action with resource
  identity, current status, expected action, today-or-past effective date,
  required reason, validation summary, and pending state.
- `AdminBulkActionBar.vue`: Coordinates selected count, all-or-nothing bulk
  action intent, and clear-selection behavior for bulk-enabled resources.
- `AdminConflictFeedback.vue`: Presents stale, dependency, ineligible, and
  batch-level conflict outcomes without exposing hidden tenant data.
- Resource detail/update components: Define only resource-specific fields and
  labels; status fields are display-only and lifecycle actions own transitions.
- Route pages: Compose resource components with detail, update, lifecycle, and
  bulk composables; pages contain no direct HTTP logic.

## Permission Matrix

| Surface | Required session permissions |
|---------|------------------------------|
| School detail | `schools.view` |
| School update and lifecycle | `schools.view`, `schools.manage` |
| User detail | `users.view` |
| User update | `users.view`, `users.manage`, `roles.view` |
| User lifecycle | `users.view`, `users.manage` |
| User bulk lifecycle | `users.view`, `users.manage` |
| Role detail | `roles.view` |
| Role update | `roles.view`, `roles.manage`, `permissions.view` |
| Role lifecycle | `roles.view`, `roles.manage` |
| Role bulk lifecycle | `roles.view`, `roles.manage` |
| Permission list/reference use | `permissions.view` |
| Academic-year detail | `academic_years.view` |
| Academic-year update and lifecycle | `academic_years.view`, `academic_years.manage` |
| Academic-year bulk lifecycle | `academic_years.view`, `academic_years.manage` |
| Academic-period detail | `academic_periods.view` |
| Academic-period update and lifecycle | `academic_periods.view`, `academic_periods.manage` |
| Academic-period bulk lifecycle | `academic_periods.view`, `academic_periods.manage` |
| Guardian detail | `guardians.view` |
| Guardian update and lifecycle | `guardians.view`, `guardians.manage` |
| Guardian bulk lifecycle | `guardians.view`, `guardians.manage` |

Backend authorization remains authoritative. Client-side visibility improves
usability but does not replace denial handling.

## Phase 0: Research

Research output is captured in [research.md](research.md). No unresolved
technical clarifications remain.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/administration-lifecycle-ui-contract.md](contracts/administration-lifecycle-ui-contract.md)
- [quickstart.md](quickstart.md)

## Post-Design Constitution Check

- PASS: Design preserves contract-first consumption and names every approved
  lifecycle operation, blocked action category, request field, outcome, and
  frontend-only restriction.
- PASS: Repository ownership remains explicit; no backend implementation or
  OpenAPI mutation is hidden in frontend work.
- PASS: Pages/components remain presentation and composition boundaries;
  services own transport; composables own detail, form, lifecycle, bulk, and
  tenant coordination; existing Pinia stores own only shared session and shell
  state.
- PASS: Tenant changes, denied states, stale requests, restore visibility,
  status-transition separation, all-or-nothing bulk behavior, and lifecycle
  reason privacy are explicit and testable.
- PASS: Route and action visibility use existing permission codes and backend
  authorization remains authoritative.
- PASS: Responsive, keyboard, validation, conflict, latency, and contract
  verification requirements are defined.
- PASS: No complexity exception is introduced.

## Complexity Tracking

No constitution violations or approved deviations.
