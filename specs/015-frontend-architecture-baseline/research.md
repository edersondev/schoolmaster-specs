# Research: Frontend Architecture Baseline

## Decision: Use JavaScript Vue 3 with Composition API and `<script setup>`

**Rationale**: The user explicitly removed TypeScript from the frontend baseline. Vue 3 Composition API with `<script setup>` remains the approved component authoring model and matches the project constitution.

**Alternatives considered**: TypeScript was rejected for this slice because the current architecture decision is JavaScript-only. Vue Options API was rejected because the project baseline requires Composition API.

## Decision: Use Vue Router route modules and lazy loading

**Rationale**: Route modules keep feature areas isolated and allow future layout selection, permission metadata, breadcrumb metadata, and lazy-loaded pages without centralizing every route in one file.

**Alternatives considered**: A single router file was rejected because it will not scale across admin, teacher, student, guardian, reporting, and support areas.

## Decision: Use Pinia for shared state boundaries

**Rationale**: Pinia is the approved Vue state library and fits session, permissions, sidebar state, notifications, active tenant or school context, and feature state shared across route views.

**Alternatives considered**: Component-only state was rejected for session and cross-route concerns. Stores as API transport layers were rejected because services must own HTTP access.

## Decision: Use Axios through service modules only

**Rationale**: Axios service modules provide one boundary for approved `/api/v1` calls, request mapping, response mapping, pagination handling, and error normalization. This preserves API-first delivery and prevents components from relying on undocumented backend details.

**Alternatives considered**: Direct Axios usage in pages or components was rejected because it breaks service isolation and makes contract drift harder to review.

## Decision: Use Element Plus as the primary UI component system

**Rationale**: Element Plus provides production-ready Vue 3 components for forms, tables, dialogs, menus, dropdowns, cards, pagination, skeletons, and empty states. The baseline requires PascalCase template tags such as `ElButton`, `ElForm`, `ElTable`, and `ElDialog`.

**Alternatives considered**: Custom Tailwind-only primitives were rejected because they would create a competing component system. Other UI libraries were rejected because Element Plus is the selected project standard.

## Decision: Use Element Plus Icons from `@element-plus/icons-vue`

**Rationale**: The user selected Element Plus Icons. A single package icon source keeps navigation, actions, feedback, and shared UI primitives visually consistent.

**Alternatives considered**: Custom icon assets remain allowed only when no suitable package icon exists. Mixing unrelated icon libraries was rejected for the baseline.

## Decision: Use Vue I18n for centralized reusable UI text

**Rationale**: The user selected Vue I18n. The baseline must be localization-ready while requiring only one launch language in this slice. Centralized reusable text prevents shared labels, validation labels, empty states, and feedback messages from being hardcoded in shared components.

**Alternatives considered**: Hardcoded strings were rejected for shared UI. Full multilingual delivery was deferred because this slice only establishes the architecture baseline.

## Decision: Use Tailwind CSS for layout and responsive refinement

**Rationale**: Tailwind is useful for spacing, layout, responsiveness, utility classes, and restrained visual adjustments around Element Plus components.

**Alternatives considered**: Rebuilding application primitives with Tailwind alone was rejected because Element Plus is the approved component system.

## Decision: Establish reusable CRUD-oriented admin patterns

**Rationale**: SchoolMaster administration will include repeated list, filter, table, form, dialog, pagination, loading, empty, validation, and error-state workflows. A reusable foundation avoids inconsistent module implementations.

**Alternatives considered**: Per-module CRUD patterns were rejected because they would duplicate architecture decisions and weaken consistency.

## Decision: Preserve API-first delivery with no OpenAPI change in this slice

**Rationale**: This feature defines frontend architecture and contract-consumption rules only. No endpoint, schema, status code, tenant rule, or backend behavior changes are introduced.

**Alternatives considered**: Defining temporary API behavior in frontend docs was rejected because frontend behavior may consume only approved OpenAPI-backed `/api/v1` semantics.

## Decision: Represent JavaScript contracts with JSDoc typedefs and service mapping helpers

**Rationale**: JSDoc typedefs keep the JavaScript-only baseline while giving services, stores, composables, and tests concrete data-shape references. Service mapping helpers keep API envelope transformation close to the contract boundary without introducing TypeScript.

**Alternatives considered**: Documentation-only shapes were rejected because they are too weak for implementation review. Runtime validation schemas and generated OpenAPI clients were deferred because this baseline should not introduce additional runtime or generation requirements before the frontend implementation needs them.

## Decision: Require WCAG 2.1 AA baseline for reusable UI

**Rationale**: The clarified accessibility baseline applies to reusable layouts, forms, tables, dialogs, navigation, and feedback states so future features inherit accessible primitives.

**Alternatives considered**: Deferring accessibility to each feature was rejected because shared primitives would be reused before accessibility expectations were stable.

## Decision: Define lightweight frontend observability boundaries

**Rationale**: The clarified baseline requires a client error boundary, service error normalization, and user-action trace points. This makes frontend failures diagnosable without requiring a full telemetry platform in this slice.

**Alternatives considered**: A full analytics or external error-reporting integration was deferred. No observability standard was rejected because it would make cross-feature error handling inconsistent.
