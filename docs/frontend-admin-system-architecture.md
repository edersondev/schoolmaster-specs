# Frontend Admin-System Architecture

## Purpose

This document defines the detailed frontend architecture baseline for the
SchoolMaster System Administrator panel. It is the reference blueprint for the
initial `schoolmaster-frontend` admin shell and for reusable CRUD patterns that
later admin modules should share.

## Product Context

SchoolMaster is a school management SaaS platform with these primary user
roles:

- System Administrator
- School Administrator
- Teacher
- Student

This document is specifically for the System Administrator panel.

## Architecture Requirements

- Use Vue 3 Composition API with `<script setup>`
- Use JavaScript everywhere
- Apply clean architecture principles to frontend organization
- Organize code by domain or feature when that improves clarity and reuse
- Create reusable UI components instead of per-page one-offs
- Separate layout components, feature pages, composables, stores, shared
  contracts, and services by responsibility
- Use Pinia for state management
- Use Vue Router
- Prepare the architecture for long-term scalability
- Prefer production-ready module structure over mock-only examples
- Reuse CRUD patterns for pages, tables, filters, forms, dialogs, and
  pagination across admin modules

## Recommended Package Baseline

Runtime:

- `vue`
- `vue-router`
- `pinia`
- `axios`
- `element-plus`
- `@element-plus/icons-vue`

Styling and tooling:

- `tailwindcss`
- `postcss`
- `autoprefixer`

Build:

- `vite`
- `@vitejs/plugin-vue`

Testing:

- `vitest`
- `@vue/test-utils`
- `jsdom`

Optional build tooling for Element Plus on-demand registration:

- `unplugin-vue-components`
- `unplugin-auto-import`

## Clean Frontend Layers

### Layout Layer

Owns application shell responsibilities:

- responsive sidebar state
- top-level header controls
- route content framing
- layout-level navigation and permission-aware visibility

### Page Layer

Owns route-facing view composition:

- page headers
- page-specific orchestration
- page-level loading, empty, and error coordination
- assembly of reusable CRUD and dashboard primitives

### Component Layer

Owns reusable UI pieces:

- layout widgets
- dashboard cards
- CRUD primitives
- feature-local sections

### Composable Layer

Owns view coordination patterns:

- filter state
- pagination state
- reusable CRUD orchestration
- permission checks used by views and layouts

### Store Layer

Owns shared application state:

- authentication
- user session
- sidebar state
- permissions
- notifications

### Service Layer

Owns transport and contract mapping:

- approved `/api/v1` calls only
- request parameter normalization
- pagination envelope mapping
- consistent API error parsing

### Contract Layer

Owns shared frontend contracts:

- API envelopes
- auth/session contracts
- admin-system feature models
- reusable CRUD and pagination contracts

## Complete Recommended Folder Structure

```text
src/
├── assets/
│   ├── images/
│   ├── icons/
│   └── styles/
│       ├── main.css
│       ├── tailwind.css
│       └── variables.css
│
├── components/
│   ├── ui/
│   │   ├── cards/
│   │   ├── tables/
│   │   ├── forms/
│   │   ├── feedback/
│   │   ├── filters/
│   │   ├── pagination/
│   │   └── dialogs/
│   │
│   └── admin-system/
│       ├── dashboard/
│       ├── schools/
│       ├── users/
│       ├── classes/
│       └── shared/
│
├── layouts/
│   ├── AdminSystemLayout.vue
│   ├── TeacherLayout.vue
│   ├── StudentLayout.vue
│   └── AuthLayout.vue
│
├── pages/
│   ├── auth/
│   │   ├── LoginPage.vue
│   │   └── ForgotPasswordPage.vue
│   │
│   └── admin-system/
│       ├── dashboard/
│       │   └── DashboardPage.vue
│       ├── schools/
│       ├── academic-years/
│       ├── users/
│       ├── guardians/
│       ├── students/
│       ├── classes/
│       ├── teachers/
│       ├── contents/
│       ├── quizzes/
│       ├── reports/
│       └── settings/
│
├── router/
│   ├── index.js
│   └── modules/
│
├── stores/
│   ├── auth.store.js
│   ├── app.store.js
│   └── admin-system/
│
├── composables/
│   ├── useCrud.js
│   ├── usePagination.js
│   ├── useFilters.js
│   └── usePermissions.js
│
├── services/
│   ├── api/
│   ├── auth/
│   └── admin-system/
│
├── contracts/
│   ├── api/
│   ├── auth/
│   └── admin-system/
│
├── constants/
├── utils/
├── App.vue
└── main.js
```

## Folder Responsibilities

### `assets/`

- brand assets
- icon sources that are not covered by Element Plus
- global stylesheet entrypoints
- CSS variables for shared theme tokens

### `components/ui/`

Shared application primitives reusable across modules:

- cards
- tables
- form building blocks
- feedback and status components
- filter bars
- pagination components
- dialogs

### `components/admin-system/`

System Administrator-specific component composition:

- dashboard widgets
- schools feature sections
- user-management sections
- class-management sections
- shared admin-only pieces

### `layouts/`

Reusable route shells split by product area and actor context.

### `pages/`

Route-facing pages only. Pages should orchestrate reusable feature components
and shared primitives instead of containing large monolithic UI logic.

### `router/modules/`

Feature route modules with lazy loading, route meta, breadcrumbs, permission
requirements, and layout selection metadata.

### `stores/`

Global or feature-shared Pinia state. Keep transport out of stores and use
services for API work.

### `composables/`

Reusable view coordination and module orchestration logic that does not belong
inside a single component.

### `services/`

HTTP and contract mapping layer. Organize by concern:

- base API client
- auth/session
- admin-system domains such as schools, users, academic years, guardians, and
  settings

### `contracts/`

Shared frontend contracts and data-shape definitions used across pages,
components, stores, and
services.

## Layout Blueprint

### `AdminSystemLayout.vue`

This is the reusable System Administrator shell.

Structure:

1. Fixed left sidebar
2. Top header
3. Main content area rendered through `RouterView`
4. Responsive behavior:
   - desktop keeps sidebar visible
   - tablet and mobile use a collapsible sidebar with a hamburger toggle

### Sidebar Requirements

- dark theme sidebar
- SchoolMaster logo at the top
- navigation items:
  - Dashboard
  - Schools
  - Academic Years
  - Users
  - Guardians
  - Students
  - Classes
  - Teachers
  - Learning Content
  - Quizzes
  - Reports
  - Settings
- active-route highlighting
- nested children support
- permission-based visibility

### Header Requirements

- white background
- search input
- notification bell icon
- user avatar
- user dropdown menu
- bottom border

### Recommended Layout Composition

- `ElContainer` for overall shell
- `ElAside` for sidebar
- `ElHeader` for top header
- `ElMain` for route content
- `ElMenu`, `ElSubMenu`, and `ElMenuItem` for navigation
- `ElDropdown` and `ElInput` for header actions

Use Tailwind for shell spacing, breakpoints, widths, overflow behavior, and
responsive visibility rules.

## Dashboard Blueprint

### `DashboardPage.vue`

Main content:

1. page title: `Dashboard`
2. subtitle: `Overview of the school management system`

### Metric Cards

- Active Schools
- Users
- Active Classes
- Students
- Learning Contents

### Recent Activity Card

Recent events:

- School created
- Teacher updated
- Class created

### Quick Actions Card

Buttons:

- Create School
- Create User
- Create Academic Year

### Recommended Dashboard Primitives

- `BasePageHeader`
- `StatCard`
- `RecentActivityCard`
- `QuickActionsCard`
- `ElRow`
- `ElCol`
- `ElCard`
- `ElButton`

## CRUD Foundation

Create reusable CRUD patterns for admin modules:

- list page
- search filters
- data table
- pagination
- create modal or create page
- edit modal or edit page
- delete confirmation dialog
- loading states
- empty states
- error states

### Reusable Components

- `BaseCrudPage`
- `BaseDataTable`
- `BaseFilterBar`
- `BasePagination`
- `BaseForm`
- `BaseConfirmDialog`
- `BasePageHeader`
- `StatCard`
- `RecentActivityCard`
- `QuickActionsCard`

### CRUD Composition Expectations

- `BaseCrudPage` coordinates page header, filters, table, pagination, and
  modal or drawer orchestration
- `BaseDataTable` wraps table rendering, row actions, loading skeletons, empty
  states, and column-slot extension points
- `BaseFilterBar` owns search, select filters, reset, and submit actions
- `BasePagination` owns shared pagination UI and event contracts
- `BaseForm` handles common form layout, validation surface, and submit or
  cancel wiring
- `BaseConfirmDialog` standardizes delete and destructive-action confirmation

## Routing Requirements

- use route modules
- support lazy loading
- support route meta for permissions
- support breadcrumbs
- support layout selection by route

### Recommended Route Meta Shape

Route metadata should allow at minimum:

- `layout`
- `requiresAuth`
- `permissions`
- `breadcrumb`
- `sidebar`
- `title`

### Route Organization

Recommended route modules:

- `auth.routes.js`
- `admin-system.routes.js`
- `schools.routes.js`
- `users.routes.js`
- `academic-years.routes.js`
- additional feature modules as the panel expands

## State Management Requirements

Prepare stores for:

- authentication
- user session
- sidebar state
- permissions
- notifications

### Initial Store Baseline

- `auth.store.js`: login state, logout action, token or session hydration,
  current user identity
- `app.store.js`: app shell state, sidebar open or collapsed state, current
  layout helpers
- `permissions.store.js`: resolved permission set and permission helpers
- `notifications.store.js`: unread counts, dropdown state, recent notification
  list

Feature-heavy admin modules may add their own local stores under
`stores/admin-system/` when state must be shared across multiple related pages.

## Shared Frontend Contracts

Initial shared contracts should cover:

- authenticated user/session
- sidebar navigation items
- route breadcrumb items
- permission identifiers
- paginated response envelopes
- CRUD query parameters
- filter models
- table column models where shared rendering behavior is standardized
- module contracts such as `School`, `User`, `AcademicYear`, and `Guardian`

### Recommended Contract Families

- `contracts/api/`: API envelopes, pagination, errors
- `contracts/auth/`: user session, auth payloads, permission claims
- `contracts/admin-system/`: schools, users, academic years, guardians,
  settings,
  dashboard summary models

## Services Baseline

Services should be grouped by concern and return normalized, contract-shaped
results.

Recommended initial service families:

- `services/api/`: base Axios instance, interceptors, envelope helpers
- `services/auth/`: login, logout, forgot-password, current-session
- `services/admin-system/`: schools, users, academic years, guardians,
  dashboard summary, notifications

Services should not mutate UI state directly.

## Example Module: Schools

Recommended structure:

```text
src/
├── components/
│   └── admin-system/
│       └── schools/
│           ├── SchoolForm.vue
│           ├── SchoolFilters.vue
│           ├── SchoolTable.vue
│           └── SchoolStatusTag.vue
│
├── pages/
│   └── admin-system/
│       └── schools/
│           ├── SchoolsListPage.vue
│           ├── CreateSchoolPage.vue
│           └── EditSchoolPage.vue
│
├── stores/
│   └── admin-system/
│       └── schools.store.js
│
├── services/
│   └── admin-system/
│       └── schools.js
│
├── contracts/
│   └── admin-system/
│       └── schools.js
│
└── router/
    └── modules/
        └── schools.routes.js
```

### Schools Module Responsibilities

- list schools with shared filters and pagination
- create schools through reusable form composition
- edit schools through reusable form composition
- delete or deactivate through shared confirmation workflow where approved by
  the contract
- centralize school transport and mapping in `services/admin-system/schools.js`
- keep page orchestration in route pages, not in low-level table or form
  primitives

## Element Plus Usage Rules

Use PascalCase Element Plus component names only.

Use:

- `ElContainer`
- `ElAside`
- `ElHeader`
- `ElMain`
- `ElMenu`
- `ElSubMenu`
- `ElMenuItem`
- `ElCard`
- `ElRow`
- `ElCol`
- `ElButton`
- `ElDropdown`
- `ElInput`
- `ElTable`
- `ElDialog`
- `ElPagination`
- `ElForm`
- `ElFormItem`
- `ElSelect`
- `ElSkeleton`
- `ElEmpty`

Do not use:

- `el-container`
- `el-aside`
- `el-header`
- any kebab-case Element Plus component tag

## Tailwind Usage Rules

Use Tailwind for:

- spacing
- layout
- responsiveness
- utility classes
- custom visual refinements

Do not use Tailwind to replace Element Plus component primitives with ad hoc
duplicates unless a documented exception is needed.

## Color Palette and Theme Rules

### TailAdmin Brand Palette

Primary:

- `50`: `#ECF3FF`
- `100`: `#DDE9FF`
- `200`: `#C2D6FF`
- `300`: `#9CB9FF`
- `400`: `#7592FF`
- `500`: `#465FFF`
- `600`: `#3641F5`
- `700`: `#2A31D8`
- `800`: `#252DAE`
- `900`: `#262E89`

Gray:

- `25`: `#FCFCFD`
- `50`: `#F9FAFB`
- `100`: `#F2F4F7`
- `200`: `#E4E7EC`
- `400`: `#98A2B3`
- `500`: `#667085`
- `700`: `#344054`
- `900`: `#101828`

Semantic:

- `success`: `#12B76A`
- `warning`: `#F79009`
- `danger`: `#F04438`

### Theme Rules

- sidebar background: `#101828`
- header background: `#FFFFFF`
- main page background: `#F9FAFB`
- cards background: `#FFFFFF`
- card border: `#E4E7EC`
- border radius: `16px`

These values should be exposed through shared CSS variables or Tailwind theme
tokens so layouts, cards, and CRUD primitives stay visually consistent.

## Delivery Scope of This Blueprint

This document defines the reference architecture only. It does not by itself
approve frontend product behavior that has not been specified in a feature spec
or backed by approved OpenAPI contracts.
