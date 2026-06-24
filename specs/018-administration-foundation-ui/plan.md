# Implementation Plan: Administration Foundation UI

**Branch**: `018-administration-foundation-ui` | **Date**: 2026-06-24 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/018-administration-foundation-ui/spec.md`

## Summary

Implement the first CRUD-heavy frontend slice for schools, users, roles,
permissions, academic years, academic periods, and guardians in
`schoolmaster-frontend`. Each resource receives a protected list route;
creatable resources receive dedicated create routes. Approved list state is
URL-query-driven, tenant-owned requests use the authenticated active school,
guardian student associations use the approved `listStudentProfiles` lookup,
and unsaved create forms guard every route exit.

The implementation extends the existing admin shell and auth/session
foundation. Route pages remain composition surfaces. Reusable admin
components handle page framing, filters, tables, pagination, feedback, and
forms; feature composables coordinate query and draft state; services own Axios
calls and OpenAPI mapping. No backend or OpenAPI change is planned.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains out of scope.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, and existing OpenAPI-backed administration operations.  
**Storage**: No backend or database change. List state persists only in route query parameters. Unsaved form drafts remain in memory and are discarded only after confirmation.  
**Testing**: Vitest and Vue Test Utils for route modules, services, contract mappers, composables, shared CRUD components, list/create pages, tenant changes, permission denials, stale requests, and unsaved-route guards; moderated UAT with five representative administrators completing six create workflows each. Redocly/OpenAPI validation is not required unless contract files change.  
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming published `/api/v1` contracts from `schoolmaster-specs`.  
**Project Type**: Frontend SPA feature with specification and cross-repository delivery artifacts.  
**Performance Goals**: With the app bootstrapped and mocked list service latency capped at 1.5 seconds, each list renders data, empty state, or recoverable error within 2 seconds from route navigation or committed query change; stale requests are cancelled or ignored; one resource action must not block unrelated shell behavior.  
**Constraints**: No undocumented endpoint, field, filter, sort, or action; no direct Axios in pages/components/router; no lifecycle/detail/edit/delete flows; no direct per-user permission assignment; no client-side tenant inference; dedicated create routes; URL-query list state; school sort hidden until backend honors it; active-only guardian student lookup; WCAG 2.1 AA target at 390px, 768px, and 1440px; PascalCase Element Plus tags; centralized reusable text.  
**Scale/Scope**: Seven list modules, six create workflows, one student-profile lookup, shared CRUD primitives, route metadata/navigation, URL query coordination, validation/error mapping, responsive layouts, and focused frontend tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. The feature consumes only existing
  `listSchools`, `createSchool`, `listUsers`, `createUser`, `listRoles`,
  `createRole`, `listPermissions`, `listAcademicYears`,
  `createAcademicYear`, `listAcademicPeriods`, `createAcademicPeriod`,
  `listGuardians`, `createGuardian`, and `listStudentProfiles` operations.
- PASS: Repository impacts are separated. `schoolmaster-specs` defines this
  plan; `schoolmaster-frontend` implements it; `schoolmaster-backend` remains
  unchanged unless contract verification finds a gap.
- PASS: Backend architecture requirements are N/A because no Laravel code,
  schema, route, Policy, Request, Resource, Service, DTO, Repository, or public
  identifier change is approved.
- PASS: Frontend design uses Vue 3 Composition API, Pinia only for existing
  shared session/shell state, Vue Router, Tailwind CSS, Element Plus, Axios
  service modules, feature folders, and service-isolated API access.
- PASS: MySQL and soft-delete behavior are unchanged. Tenant-owned modules use
  the authenticated active `school_id`; school changes clear stale tenant data
  after unsaved-change confirmation.
- PASS: API compatibility, authentication, authorization, pagination,
  filtering, sorting, success envelopes, error mapping, and exact frontend
  permission requirements are documented.
- PASS: Vitest covers changed critical frontend flows. OpenAPI validation is
  required only if contract review results in a separate contract change.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/018-administration-foundation-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── administration-foundation-ui-contract.md
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
├── specs/018-administration-foundation-ui/
└── AGENTS.md

# Frontend repository target shape
schoolmaster-frontend/
├── src/
│   ├── components/
│   │   ├── ui/admin/
│   │   │   ├── AdminListPage.vue
│   │   │   ├── AdminFilterBar.vue
│   │   │   ├── AdminDataTable.vue
│   │   │   ├── AdminPagination.vue
│   │   │   ├── AdminFormPage.vue
│   │   │   └── AdminFeedbackState.vue
│   │   └── admin-system/
│   │       ├── schools/
│   │       ├── users/
│   │       ├── roles/
│   │       ├── permissions/
│   │       ├── academic-years/
│   │       ├── academic-periods/
│   │       └── guardians/
│   ├── composables/admin-system/
│   │   ├── useAdminListQuery.js
│   │   ├── useAdminList.js
│   │   ├── useAdminCreateForm.js
│   │   ├── useUnsavedChangesGuard.js
│   │   └── useStudentProfileLookup.js
│   ├── contracts/admin-system/
│   │   ├── administration.js
│   │   ├── schools.js
│   │   ├── users.js
│   │   ├── access.js
│   │   ├── academics.js
│   │   └── guardians.js
│   ├── locales/
│   │   └── administration.js
│   ├── pages/admin-system/
│   │   ├── schools/
│   │   ├── users/
│   │   ├── roles/
│   │   ├── permissions/
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
│       ├── permissions.js
│       ├── academic-years.js
│       ├── academic-periods.js
│       ├── guardians.js
│       └── student-profiles.js
└── tests/unit/admin-system/administration/
    ├── contracts/
    ├── services/
    ├── composables/
    ├── components/
    ├── pages/
    └── routes/
```

**Structure Decision**: Extend the existing parallel frontend boundaries
instead of introducing a new monolithic `modules/` root. Shared CRUD
presentation lives under `components/ui/admin`; resource-specific fields and
columns live under `components/admin-system/<resource>`. Route pages compose
components and composables. URL query is list-state source of truth, so no
resource Pinia store is planned. Existing session and shell stores continue to
own authenticated tenant and navigation state. Resource route modules remain
separate so story implementation can proceed in parallel;
`administration.routes.js` only assembles them.

## Component Map

- `AdminListPage.vue`: Frames title, authorized create action, filters, table,
  feedback, and pagination through slots and explicit props/events.
- `AdminFilterBar.vue`: Renders approved filters and emits normalized filter or
  reset intent; it does not call services.
- `AdminDataTable.vue`: Wraps Element Plus remote table state, loading, empty
  rendering, responsive column slots, and sort events.
- `AdminPagination.vue`: Maps paginated envelope metadata to page/page-size
  events.
- `AdminFormPage.vue`: Frames dedicated create routes, validation summary,
  submit/cancel controls, and pending state.
- `AdminFeedbackState.vue`: Renders normalized loading, empty, unavailable,
  forbidden, tenant-mismatch, not-found, and retry states.
- Resource filter/table/form components: Define only resource fields, columns,
  and approved options; data and pending state arrive through props, and user
  intent leaves through emits.
- Route pages: Compose resource components with `useAdminList` or
  `useAdminCreateForm`; they remain thin and contain no direct HTTP logic.

## Permission Matrix

| Surface | Required session permissions |
|---------|------------------------------|
| School list | `schools.view` |
| School create | `schools.view`, `schools.manage` |
| User list | `users.view` |
| User create | `users.view`, `users.manage`, `roles.view` |
| Role list | `roles.view` |
| Role create | `roles.view`, `roles.manage`, `permissions.view` |
| Permission list | `permissions.view` |
| Academic-year list | `academic_years.view` |
| Academic-year create | `academic_years.view`, `academic_years.manage` |
| Academic-period list | `academic_periods.view` |
| Academic-period create | `academic_periods.view`, `academic_periods.manage`, `academic_years.view` |
| Guardian list | `guardians.view` |
| Guardian create | `guardians.view`, `guardians.manage` |
| Guardian student lookup | `student_profiles.view` |

Role creation in this feature is school-only. UI exposes no scope selector and
the role service mapper submits the contract-required `scope` value as
`school`. Platform-role creation remains outside this frontend slice.

## Phase 0: Research

Research output is captured in [research.md](research.md). No unresolved
technical clarifications remain.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/administration-foundation-ui-contract.md](contracts/administration-foundation-ui-contract.md)
- [quickstart.md](quickstart.md)

## Post-Design Constitution Check

- PASS: Design preserves contract-first consumption and names every approved
  operation, request parameter, envelope, and blocked lifecycle category.
- PASS: Repository ownership remains explicit; no backend implementation or
  contract mutation is hidden in frontend work.
- PASS: Pages/components remain presentation and composition boundaries;
  services own transport; composables own route/form coordination; existing
  Pinia stores own only shared session and shell state.
- PASS: Tenant changes, denied states, stale requests, guardian lookup scope,
  and cross-tenant privacy are explicit and testable.
- PASS: Route and action visibility use exact implemented permission codes, and
  role creation is constrained to school scope.
- PASS: School sorting is blocked until backend contract implementation is
  compliant; active guardian lookup, deterministic latency measurement, and
  concrete responsive viewports are defined.
- PASS: Vitest coverage maps to services, query normalization, route guards,
  reusable components, forms, validation, and critical user flows.
- PASS: No complexity exception is introduced.

## Complexity Tracking

No constitution violations.
