# Frontend Guidelines

## Target

Frontend implementation targets the `schoolmaster-frontend` Vue 3 SPA
repository.

For the baseline application structure, see
[frontend-architecture.md](frontend-architecture.md).

## Working Principles

- Use Vue 3 Composition API
- Use `<script setup>` for Vue SFCs
- Use JavaScript consistently across the frontend codebase
- Use Pinia for shared feature state
- Use Vue Router with route modules and lazy-loaded feature pages
- Keep API access in a service layer
- Use Element Plus as the primary UI component library
- Use Element Plus Icons from `@element-plus/icons-vue` as the default icon set
- Use Vue I18n for centralized reusable UI text
- Consume only published OpenAPI-backed endpoints
- Represent JavaScript contract shapes with JSDoc typedefs plus service
  mapping helpers in `src/contracts/`
- Keep business rules sourced from approved specifications
- Favor production-ready code and reusable patterns over mock-only examples

## Contract Alignment

The frontend must treat OpenAPI as the backend contract and should not depend
on undocumented payloads or routes.

## State and Module Conventions

- Organize feature code by domain or feature across `src/pages/`,
  `src/components/`, `src/stores/`, `src/services/`, and `src/contracts/`.
- Apply clean architecture principles to frontend organization by keeping
  layouts, pages, components, composables, stores, services, and shared
  contract definitions separated by responsibility.
- Use `src/contracts/` for JSDoc typedefs and service mapping helpers, not
  TypeScript types or documentation-only shapes.
- Keep HTTP access in `src/services/` and keep services free of UI state.
- Use Pinia stores for session state, tenant context, role-aware navigation,
  and feature state that is shared across views.
- Restore tenant context only from authenticated API responses or approved
  persisted session metadata; do not invent tenant scope client-side.
- Create reusable primitives for CRUD-heavy admin workflows so tables, filters,
  forms, pagination, confirm dialogs, loading states, empty states, and error
  states are not reimplemented independently per module.
- Keep reusable UI text centralized through Vue I18n instead of hardcoding
  labels, actions, validation labels, empty states, and feedback text inside
  shared components.

## UI Composition Standards

- Prefer Element Plus components for forms, tables, dialogs, navigation,
  feedback, and data display primitives.
- Prefer Element Plus Icons for navigation, actions, feedback, and shared UI
  primitives before adding custom icon assets.
- Always use PascalCase Element Plus component tags in Vue templates, such as
  `ElButton`, `ElForm`, and `ElTable`; do not use kebab-case equivalents.
- Use Tailwind CSS for layout, spacing, responsive rules, and restrained
  visual customization around Element Plus.
- Treat the System Administrator panel as the reference implementation for
  reusable admin layout and CRUD patterns.
- Build school administration, teacher, student, and reporting screens from the
  approved product workflows and permissions.
- Treat client-side role checks as navigation and visibility aids only; backend
  authorization remains authoritative.
- Keep reusable composables focused on view coordination, not business rules
  that belong in specifications or backend services.

## Layout and Navigation Baseline

- This baseline approves reusable shell responsibilities, not a concrete
  System Administrator shell implementation.
- A later shell or dashboard feature spec must define the exact layout
  component names, sidebar/header behavior, navigation entries, responsive
  rules, dashboard content, and API contracts before implementation.
- Reusable shell patterns should provide a route content region, layout
  selection through route metadata, permission-aware navigation visibility,
  global feedback surfaces, and responsive state boundaries.
- Navigation items must support active-route styling, nested children, and
  permission-based visibility when a consuming feature approves concrete
  navigation behavior.

## Theme Baseline

- Element Plus provides component primitives.
- Tailwind provides spacing, responsive behavior, and minor visual refinement.
- The admin panel uses the TailAdmin palette baseline documented in
  [frontend-admin-system-architecture.md](frontend-admin-system-architecture.md).

## Reference Blueprint

Use [frontend-admin-system-architecture.md](frontend-admin-system-architecture.md)
as the source for:

- complete recommended folder structure
- reusable shell responsibilities
- reusable dashboard composition boundaries
- reusable CRUD foundation components
- initial router, store, service, and contract organization
- illustrative feature-module structure that does not approve resource-specific
  behavior

## Error and Loading Patterns

- Map API error envelopes consistently for validation, unauthorized, forbidden,
  inactive-record, tenant-mismatch, and not-found outcomes.
- Preserve enough error detail for users to correct validation problems without
  exposing tenant or authorization internals.
- Loading states should be scoped to the service/store action being performed
  so one module does not block unrelated tenant-safe workflows.
