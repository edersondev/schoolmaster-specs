# Frontend Architecture

## Purpose

This document defines the baseline architecture for the
`schoolmaster-frontend` Vue 3 SPA so frontend feature specifications can build
on one shared structure instead of redefining application shape per feature.

## Scope

This architecture applies to frontend delivery that consumes approved
SchoolMaster `/api/v1` contracts. It does not define product behavior on its
own; feature specs remain the source of truth for workflows, permissions, and
visible outcomes.

For the detailed System Administrator panel blueprint, including the complete
recommended folder structure, reusable layout, dashboard, CRUD primitives,
stores, router, shared contracts, and example Schools module, see
[frontend-admin-system-architecture.md](frontend-admin-system-architecture.md).

## Technology Baseline

- Vue 3 with Composition API and `<script setup>`
- Vue Router for route composition
- Pinia for shared application and feature state
- Axios for HTTP transport
- Element Plus for the primary component library
- Tailwind CSS for styling

Do not introduce alternate state, transport, or styling foundations without a
documented architecture decision.

Recommended package baseline:

- `vue`
- `vue-router`
- `pinia`
- `axios`
- `element-plus`
- `@element-plus/icons-vue` for Element Plus Icons
- `vue-i18n`
- `tailwindcss`

If the frontend repository uses on-demand Element Plus registration, supporting
build tooling such as `unplugin-vue-components` and `unplugin-auto-import` may
be added in the frontend implementation repository.

## Architectural Principles

- OpenAPI is the backend contract boundary.
- Vue SFCs use `<script setup>`.
- Feature specs define behavior; frontend code implements approved behavior.
- Frontend organization should follow clean architecture principles with
  explicit boundaries between presentation, view coordination, state,
  transport, and shared contract definitions.
- Components handle presentation and interaction, not business rules.
- Services own HTTP access and transport mapping.
- JavaScript frontend contracts use JSDoc typedefs plus service mapping helpers
  under `src/contracts/`.
- Stores coordinate shared client state and UI workflows, but do not replace
  services as transport boundaries.
- Tenant context, role visibility, and authorization-sensitive behavior must
  follow authenticated backend responses.
- Reusable CRUD-heavy admin patterns should be implemented once and reused
  across modules where the workflow shape is the same.

## Application Shape

The frontend should be organized around feature modules with a thin
application shell.

Recommended top-level structure:

```text
src/
  app/
  router/
  layouts/
  pages/
  components/
  services/
  stores/
  composables/
  contracts/
  constants/
  utils/
```

### Application Shell

The application shell is responsible for:

- bootstrapping the authenticated session
- restoring the active school context from approved persisted session metadata
  or authenticated API responses
- loading role-aware navigation
- coordinating route guards and layout selection
- exposing global feedback surfaces such as session-expired, maintenance, or
  contract-safe error states

The shell must stay small. Domain workflows belong in feature modules.

### Feature Modules

Each product area should own its route-facing pages, local components,
composables, store logic, services, and shared contracts through
feature-aligned folders.
The current frontend baseline uses parallel domain folders under
`src/pages/`, `src/components/`, `src/stores/`, `src/services/`, and
`src/contracts/` instead of forcing every feature into one monolithic module
directory.

Expected module families:

- `platform`
- `school-admin`
- `teacher`
- `student`
- `guardian`
- `reporting`

Feature areas should depend on shared services and shell state, not on each
other's internal files.

## Routing and Navigation

- Vue Router is the single route definition layer.
- Route records should be grouped by feature module and assembled into the
  application router.
- Route metadata may describe authentication, school-context requirement,
  role-aware navigation visibility, and page-level titles or breadcrumbs.
- Client-side route guards are navigation controls only. They must not be
  treated as authoritative authorization.

When a route depends on school context, the application must resolve the active
school before module-specific loading starts.

## Services and API Access

All HTTP access must go through `src/services/`.

Service responsibilities:

- call approved `/api/v1` endpoints only
- map request parameters to approved contract fields
- use `src/contracts/` JSDoc typedefs and mapping helpers for consumed data
  shapes
- normalize transport concerns such as headers, pagination primitives, and
  error envelope parsing
- return contract-shaped data to stores or composables without embedding UI
  concerns

Service responsibilities do not include:

- rendering decisions
- component state mutation
- ad hoc permission rules that contradict the feature spec or backend behavior

Components and route views must not call Axios directly.

## State Management

Pinia stores should be used for state that is shared across routes, layouts, or
multiple related views.

Use stores for:

- authenticated user session
- active school context
- role-aware navigation state
- feature state shared across lists, detail views, filters, and mutations

Do not use stores as dumping grounds for every local interaction. View-local
state should remain inside the route view or a dedicated composable when it
does not need to be shared.

## Composables

Composables should coordinate view behavior, such as:

- table filtering and pagination state
- form draft management
- optimistic UI coordination where the spec allows it
- route query synchronization

Composables must not become hidden service layers or alternate global stores.

## Auth, Session, and Tenant Context

- Authentication state must come from approved backend session or token flows.
- The frontend must not infer unauthorized tenant scope client-side.
- Active school context may be restored only from authenticated backend data or
  approved persisted session metadata.
- Platform-scoped and school-scoped navigation must remain visibly distinct.
- When backend responses indicate inactive, unauthorized, or mismatched school
  context, the UI must return to a contract-safe state instead of preserving
  stale tenant data.

## UI Composition

- Element Plus is the default component system for forms, tables, dialogs,
  navigation, feedback, and other common application UI primitives.
- In Vue templates, Element Plus components must be written with PascalCase
  tags such as `ElForm`, `ElTable`, and `ElDialog`, not kebab-case tags.
- Vue I18n is the default localization package for centralized reusable UI
  text and future multilingual support.
- Element Plus Icons from `@element-plus/icons-vue` are the default icon source
  for navigation, actions, feedback, and shared UI primitives.
- Shared visual primitives should live in shared components or layouts.
- Module-specific views should compose shared primitives with feature-local
  state and actions.
- Tailwind should be used for layout, spacing, responsive behavior, and small
  visual adjustments around Element Plus components, not to rebuild a second
  competing component system.
- Forms, tables, filters, detail panes, and status surfaces must reflect the
  approved workflow and permission model from the relevant feature spec.
- Do not expose hidden backend fields, undocumented actions, or inferred
  statuses just because they are convenient for the UI.

## Error, Empty, and Loading States

Frontend delivery must explicitly design for:

- loading
- empty
- validation error
- unauthorized
- forbidden
- tenant mismatch
- not found
- inactive or unavailable record
- conflict or stale update

These states must map to published OpenAPI semantics. Do not invent alternate
status meanings in the frontend.

Loading state should be scoped to the action in progress so one feature action
does not block unrelated tenant-safe workflows.

## Testing Boundaries

Frontend verification should cover the contract-consuming layers that can break
behavior safely and repeatedly:

- services
- stores
- composables
- route-level workflow logic where risk justifies it

Vitest coverage should focus on permission-sensitive state transitions, error
mapping, tenant-context handling, and contract-shaped response consumption.

## Relationship to Feature Specs

Frontend feature specs should reference this document for baseline structure and
only specify what changes for the feature:

- affected routes or surfaces
- user workflows
- visible states
- permissions
- contract dependencies
- repository sequencing

If a frontend feature needs to deviate from this architecture, update this
document or add an ADR before implementation starts.
