# Implementation Plan: System Administrator Shell and Dashboard Foundation

**Branch**: `016-admin-shell-dashboard` | **Date**: 2026-06-22 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/016-admin-shell-dashboard/spec.md`

## Summary

Define the first concrete System Administrator frontend shell and placeholder
dashboard foundation for `schoolmaster-frontend`. The plan consumes the
completed frontend architecture baseline and specifies a reusable admin layout
frame with sidebar navigation, top header, route content region, dashboard
placeholder regions, hidden permission-gated navigation and quick actions,
desktop/tablet collapsible sidebar behavior, mobile overlay drawer behavior,
and shell-level unauthorized, forbidden, and session-expired feedback states.

Live dashboard summary, recent activity, and notification data are explicitly
blocked in this slice until later specs and OpenAPI contracts approve them.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains out of scope.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS.  
**Storage**: No backend or database storage change. Frontend shell state may use Pinia and approved client persistence only for non-sensitive UI preferences such as collapsed sidebar state.  
**Testing**: Specification review in `schoolmaster-specs`; future `schoolmaster-frontend` implementation should use Vitest for route metadata, permission visibility, shell state, dashboard placeholders, and denial-state behavior.  
**Target Platform**: `schoolmaster-frontend` Vue 3 SPA consuming contracts from `schoolmaster-specs`.  
**Project Type**: Frontend SPA feature specification and cross-repository planning artifact.  
**Performance Goals**: Shell and dashboard route interactions should remain usable without live data dependencies; future implementation should render placeholder dashboard content and navigation state within normal SPA interaction expectations.  
**Constraints**: No backend implementation; no OpenAPI change; no live dashboard data consumption; no authentication screens; no concrete CRUD pages; no reporting or platform-support workflows; unauthorized navigation and quick actions are hidden; mobile navigation uses an overlay drawer; desktop/tablet navigation uses a collapsible sidebar.  
**Scale/Scope**: Applies to the System Administrator shell, dashboard placeholder surface, permission-aware navigation visibility, static quick actions for approved routes only, shell feedback states, and route metadata that later admin, reporting, and support modules attach to.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. This slice introduces no OpenAPI change and blocks live dashboard, recent activity, and notification data until later contracts approve it.
- PASS: Repository impacts are separated. `schoolmaster-specs` leads this plan, `schoolmaster-frontend` consumes it for implementation, and `schoolmaster-backend` remains unchanged.
- PASS: Backend architecture requirements are N/A because this feature does not change Laravel code, services, requests, policies, resources, models, public identifiers, or routes.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind CSS, Axios service boundaries where later API consumption exists, feature-aligned organization, and service-isolated API access.
- PASS: MySQL, tenant-scoping, cross-tenant access, and soft-delete impacts are documented as unchanged. Tenant or school context displayed by the shell must come from approved authenticated session behavior.
- PASS: API compatibility, authentication, authorization, and error-state impacts are documented. Client-side permission checks hide surfaces only; backend authorization remains authoritative.
- PASS: Verification is scoped to specification review now, with future Vitest coverage required for affected frontend routes, stores, composables, and shell/dashboard behavior.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/016-admin-shell-dashboard/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/
│   └── admin-shell-dashboard-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
├── docs/
│   ├── frontend-admin-system-architecture.md
│   ├── frontend-architecture.md
│   ├── frontend-feature-roadmap.md
│   ├── frontend-guidelines.md
│   └── naming-conventions.md
├── specs/
│   └── 016-admin-shell-dashboard/
│       ├── spec.md
│       ├── plan.md
│       ├── research.md
│       ├── data-model.md
│       ├── quickstart.md
│       └── contracts/
│           └── admin-shell-dashboard-contract.md
└── AGENTS.md

# Frontend repository target shape consumed later by schoolmaster-frontend
schoolmaster-frontend/
├── src/
│   ├── layouts/
│   │   └── admin-system/
│   │       └── AdminSystemLayout.vue
│   ├── pages/
│   │   └── admin-system/
│   │       └── dashboard/
│   │           └── AdminDashboardPage.vue
│   ├── components/
│   │   └── admin-system/
│   │       ├── shell/
│   │       └── dashboard/
│   ├── composables/
│   │   └── admin-system/
│   ├── contracts/
│   │   └── admin-system/
│   ├── locales/
│   │   └── admin-system.js
│   ├── router/
│   │   └── modules/
│   │       └── admin-system.routes.js
│   └── stores/
│       └── admin-system/
└── tests/
    └── unit/
```

**Structure Decision**: This planning slice changes specification artifacts in
`schoolmaster-specs` and defines target frontend files for later
`schoolmaster-frontend` implementation. It does not change backend code,
OpenAPI, or API behavior. Shell and dashboard implementation must remain under
admin-system frontend boundaries and must not introduce concrete CRUD module
pages or live dashboard services in this slice.

## Phase 0: Research

Research output is captured in [research.md](research.md). All technical
context items are resolved.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/admin-shell-dashboard-contract.md](contracts/admin-shell-dashboard-contract.md)
- [quickstart.md](quickstart.md)

## Post-Design Constitution Check

- PASS: The design artifacts preserve the no-OpenAPI-change boundary and block live dashboard, recent activity, and notification data until later approved contracts exist.
- PASS: The frontend contract defines route metadata, hidden permission-gated navigation and quick actions, static approved quick actions only, placeholder dashboard regions, responsive shell behavior, shell feedback states, and state ownership boundaries.
- PASS: Data and tenancy impact remain frontend-only with no backend schema, tenant policy, cross-tenant access, or soft-delete changes.
- PASS: Future verification expectations are captured without creating implementation tasks in this slice.
- PASS: No complexity exception is introduced.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No constitution violations.
