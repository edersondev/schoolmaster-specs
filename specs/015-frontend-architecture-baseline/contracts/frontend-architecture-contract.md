# Frontend Architecture Contract

This contract defines what future SchoolMaster frontend specs and implementations may rely on from the Frontend Architecture Baseline.

## Package Baseline

Future `schoolmaster-frontend` implementation must use:

- `vue`
- `vue-router`
- `pinia`
- `axios`
- `element-plus`
- `@element-plus/icons-vue`
- `vue-i18n`
- `tailwindcss`

Supporting build-time tooling may be added in the frontend repository when needed for the selected Vite or Element Plus registration strategy, but it must not change the approved runtime architecture.

## Language and File Contract

- Vue SFCs use JavaScript and `<script setup>`.
- TypeScript is not required or approved by this baseline.
- Vue components and pages use PascalCase filenames.
- Composables use `use*.js`.
- Pinia stores use `*.store.js`.
- Router modules use lowercase hyphenated filenames such as `admin-system.routes.js`.
- Services use `.js` files and expose contract-focused functions.
- Frontend contract files use JSDoc typedefs plus service mapping helpers under `src/contracts/`.

## Folder Responsibility Contract

Future frontend work must organize product code by feature module across the
approved responsibility boundaries. Shared layer folders are allowed only for
cross-feature infrastructure, reusable primitives, and framework boundaries.

Application code should use this top-level structure:

```text
src/
  app/
  assets/
  components/
  composables/
  constants/
  contracts/
  layouts/
  pages/
  router/
  services/
  stores/
  utils/
```

Responsibility boundaries:

- `layouts/`: route shells and shared frame composition.
- `pages/`: route-facing views.
- `components/`: reusable shared UI and feature-local components.
- `composables/`: reusable view coordination.
- `stores/`: Pinia shared state.
- `services/`: Axios-backed API access and response/error mapping.
- `contracts/`: JSDoc typedefs and service mapping helpers for JavaScript contract shapes.
- `constants/`: stable app constants.
- `utils/`: pure utility helpers with no transport or UI ownership.

Feature modules must remain traceable across pages, components, composables,
stores, services, contracts, and route modules. Shared folders must not become a
catch-all for feature-owned behavior.

## API Consumption Contract

- Frontend behavior may consume only approved OpenAPI-backed `/api/v1` endpoints.
- Services own all Axios calls.
- Pages, layouts, and components must not call Axios directly.
- Stores may call services but must not bypass them.
- Frontend code must not use undocumented response fields, filters, routes, status meanings, error codes, tenant semantics, or authorization behavior.
- If a future frontend feature needs missing contract behavior, implementation must block until the spec and OpenAPI contract approve it.

## UI Primitive Contract

- Element Plus is the primary component system for forms, tables, dialogs, menus, cards, pagination, feedback, skeletons, and empty states.
- Element Plus component tags must use PascalCase in templates, such as `ElContainer`, `ElAside`, `ElHeader`, `ElMain`, `ElMenu`, `ElSubMenu`, `ElMenuItem`, `ElCard`, `ElRow`, `ElCol`, `ElButton`, `ElDropdown`, `ElInput`, `ElTable`, `ElDialog`, `ElPagination`, `ElForm`, `ElFormItem`, `ElSelect`, `ElSkeleton`, and `ElEmpty`.
- Kebab-case Element Plus tags such as `el-button`, `el-form`, and `el-table` are not allowed.
- Tailwind CSS owns layout, spacing, responsiveness, utility classes, and restrained visual refinements around Element Plus.
- Tailwind must not replace Element Plus with a competing unapproved component system.

## Icon Contract

- Use Element Plus Icons from `@element-plus/icons-vue` by default.
- Use package icons for navigation, actions, feedback, empty states, and shared UI primitives.
- Use custom icon assets only when no suitable package icon exists and the consuming feature documents the reason.

## Internationalization Contract

- Use Vue I18n for centralized reusable UI text.
- Shared labels, actions, validation labels, empty states, loading text, and feedback messages must not be hardcoded inside shared components.
- This baseline requires localization readiness and one launch language only; full multilingual delivery is a later product decision.

## Accessibility Contract

Reusable layouts, forms, tables, dialogs, navigation, feedback states, and CRUD primitives must target WCAG 2.1 AA.

At minimum, future frontend implementation must preserve:

- keyboard-reachable navigation and actions
- visible focus states
- accessible labels for form controls and icon-only actions
- semantic feedback for loading, empty, error, validation, unauthorized, forbidden, tenant-mismatch, not-found, inactive-record, and conflict states
- dialog focus management consistent with Element Plus behavior

## CRUD Pattern Contract

Reusable admin CRUD patterns must cover:

- list pages
- search and filters
- data tables
- pagination
- create and edit forms
- confirmation dialogs
- loading states
- empty states
- validation states
- error states

Concrete resource behavior, editable fields, permissions, lifecycle transitions, and business rules remain owned by future feature specs.

## State Contract

Pinia stores may own:

- authentication/session state
- user session metadata
- active tenant or school context from approved sources
- permissions for visibility and navigation
- sidebar state
- notifications
- feature state shared across views

Stores must not become transport layers or containers for unrelated local component state.

## Observability Contract

Future frontend implementation must provide reusable boundaries for:

- client error capture
- normalized service errors
- user-action trace points

This baseline does not require a specific telemetry vendor or external analytics integration.

## Blocked by This Contract

This baseline does not approve:

- TypeScript requirements
- concrete System Administrator layout behavior
- authentication screens
- business module workflows
- backend implementation
- OpenAPI changes
- undocumented API behavior
- reporting behavior
- assessment behavior
- platform support behavior
