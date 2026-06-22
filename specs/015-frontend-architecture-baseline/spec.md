# Feature Specification: Frontend Architecture Baseline

**Feature Branch**: `015-frontend-architecture-baseline`  
**Created**: 2026-06-21  
**Status**: Draft  
**Input**: User description: "Define frontend roadmap item 1: Frontend Architecture Baseline. Specify the durable Vue 3 SPA architecture for SchoolMaster using JavaScript, Vue 3 Composition API with `<script setup>`, Vue Router, Pinia, Axios, Element Plus, Tailwind CSS, and clean frontend boundaries. Establish the shared application structure for layouts, pages, components, composables, stores, services, contracts, constants, and utilities. Define reusable CRUD-oriented frontend patterns for admin workflows, including list pages, filters, tables, forms, dialogs, pagination, loading states, empty states, and error states. Preserve API-first delivery: frontend behavior may consume only approved OpenAPI-backed `/api/v1` endpoints and documented contract semantics. Include Element Plus PascalCase component usage, Tailwind layout responsibilities, JavaScript file conventions, scalable folder organization, and repository sequencing for `schoolmaster-specs` and `schoolmaster-frontend`. Do not define concrete business module behavior, backend implementation, undocumented API behavior, or TypeScript requirements in this slice."

## Clarifications

### Session 2026-06-21

- Q: What accessibility baseline should the frontend architecture require? → A: WCAG 2.1 AA baseline for reusable layouts, forms, tables, dialogs, navigation, and feedback states.
- Q: What localization baseline should the frontend architecture require? → A: Internationalization-ready with Vue I18n; reusable UI text must be centralized, with one launch language required.
- Q: What observability baseline should the frontend architecture require? → A: Standard client error boundary, service error normalization, and user-action trace points.
- Q: How should JavaScript frontend contract definitions be represented without TypeScript? → A: JSDoc typedefs plus service mapping helpers in `src/contracts/`.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Establish Frontend Architecture Baseline (Priority: P1)

A frontend implementer needs a durable architecture baseline before starting SchoolMaster SPA work so every frontend feature follows the same structure, dependency boundaries, naming conventions, and contract-first behavior.

**Why this priority**: The baseline prevents frontend feature specs and implementation work from inventing incompatible layouts, state ownership, service access, component patterns, or API-consumption rules.

**Independent Test**: Review the architecture documents and confirm they define the approved frontend stack, folder responsibilities, JavaScript conventions, Element Plus usage rules, Tailwind responsibilities, and cross-repository sequencing without approving any concrete business module behavior.

**Acceptance Scenarios**:

1. **Given** a contributor is preparing frontend work, **When** they read the frontend architecture baseline, **Then** they can identify the approved SPA stack, application structure, folder responsibilities, and service/store/component boundaries.
2. **Given** a future frontend feature spec is created, **When** it references the baseline, **Then** it can rely on existing conventions for layouts, pages, components, composables, stores, services, contracts, constants, and utilities instead of redefining them.
3. **Given** a contributor proposes TypeScript, undocumented APIs, or a different UI component system for the baseline, **When** the proposal is reviewed against this feature, **Then** it is rejected unless a future approved architecture change explicitly supersedes this baseline.

---

### User Story 2 - Standardize Reusable Admin Workflow Patterns (Priority: P2)

A frontend implementer needs shared CRUD-oriented patterns for admin workflows so list pages, filters, tables, forms, dialogs, pagination, and operational states remain consistent across admin modules.

**Why this priority**: SchoolMaster administration contains many similar CRUD-heavy workflows. Reusable patterns reduce inconsistent screens, repeated decisions, and expensive rework as modules expand.

**Independent Test**: Review the baseline and verify that reusable admin workflow patterns are documented for list, search/filter, table, form, dialog, pagination, loading, empty, and error states without defining module-specific behavior for schools, users, guardians, classes, or reports.

**Acceptance Scenarios**:

1. **Given** an admin module needs a list page, **When** the module is specified later, **Then** it can reuse the baseline list, filter, table, pagination, loading, empty, and error-state conventions.
2. **Given** an admin module needs create, edit, or delete workflows, **When** the module is specified later, **Then** it can reuse baseline form and confirmation-dialog expectations while leaving resource-specific rules to that module's feature spec.
3. **Given** a module has a workflow shape that does not fit the reusable CRUD baseline, **When** the deviation is proposed, **Then** it must be documented in that module's feature spec or an architecture update before implementation.

---

### User Story 3 - Preserve API-First Frontend Delivery (Priority: P3)

A frontend implementer needs clear contract-consumption rules so frontend screens do not depend on undocumented backend behavior, hidden response fields, or route assumptions.

**Why this priority**: SchoolMaster uses separate specification, backend, and frontend repositories. The frontend must remain aligned with approved OpenAPI-backed behavior to avoid cross-repository drift.

**Independent Test**: Review the baseline and verify that it requires frontend behavior to consume only documented `/api/v1` contract semantics, isolates HTTP access in services, and blocks undocumented fields, routes, status meanings, or tenant behavior.

**Acceptance Scenarios**:

1. **Given** a frontend feature needs data from the backend, **When** it is specified or implemented, **Then** it must link to approved OpenAPI-backed behavior or document that the required contract is not ready.
2. **Given** a frontend component or page needs remote data, **When** it is designed, **Then** HTTP access must be routed through the approved service boundary and not embedded directly in route views or UI components.
3. **Given** a backend response contains undocumented fields or states, **When** frontend behavior is reviewed, **Then** those fields or states must not be used until the relevant OpenAPI contract and feature specification approve them.

### Edge Cases

- Future requests for TypeScript, a different component library, or a different state model must be handled as separate architecture changes before implementation.
- Client-side permission checks must guide visibility and navigation only; backend authorization remains authoritative.
- Modules that need data not yet documented in OpenAPI must block frontend consumption until the contract is approved.
- The baseline must define shared structure and reusable patterns only; concrete business workflows remain owned by later feature specs.
- Future modules must use the baseline loading, empty, validation, unauthorized, forbidden, tenant-mismatch, not-found, inactive-record, and conflict-state conventions where those states apply.
- Element Plus remains the primary component system, and Tailwind remains responsible for layout, spacing, responsiveness, utility classes, and restrained visual refinement.
- JavaScript file conventions, folder boundaries, and shared contract definitions must remain discoverable through the feature spec, architecture docs, naming conventions, and quickstart.
- Reusable layouts, forms, tables, dialogs, navigation, and feedback states must be reviewed against the WCAG 2.1 AA accessibility baseline.
- Reusable UI text must be centralized through Vue I18n without requiring full multilingual delivery in this slice.
- Frontend failures must remain diagnosable through the client error boundary, service error normalization, and user-action trace points without requiring a full telemetry platform in this slice.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No backend implementation is included. Backend behavior remains governed by existing and future OpenAPI-backed feature specifications.
- **Frontend repository impact**: Establishes the durable SPA architecture baseline that `schoolmaster-frontend` must consume before feature implementation. Future frontend work must follow the approved JavaScript, routing, state, service, component, contract, UI, and CRUD-boundary conventions unless superseded by an approved architecture change.
- **Specification or contract repository impact**: Adds this feature specification as the product-level authorization for the frontend architecture baseline and aligns existing documentation under `docs/frontend-architecture.md`, `docs/frontend-admin-system-architecture.md`, `docs/frontend-guidelines.md`, `docs/naming-conventions.md`, and `docs/frontend-feature-roadmap.md`.
- **Delivery ownership and sequencing**: `schoolmaster-specs` leads the architecture definition first. `schoolmaster-frontend` consumes the approved baseline next. Backend implementation remains unchanged, and future frontend feature specs must link to this baseline plus the OpenAPI operation IDs they consume.

### API Contract Impact

- **OpenAPI update required**: No. This feature defines frontend architecture and contract-consumption rules only; it does not add, change, or remove REST behavior.
- **Versioned endpoints affected**: None directly. Future frontend features may consume `/api/v1` operations only after those operations are documented and approved.
- **JSON response impact**: No response envelope, field, status, or error-contract changes are introduced by this feature.
- **Authentication/authorization impact**: The baseline must preserve existing contract-first authentication, tenant-context, and authorization behavior. Frontend permission checks may control visibility and navigation but must not replace backend authorization.
- **Compatibility impact**: Additive documentation and specification change. No backend or frontend runtime behavior changes are approved by this slice.

### Data & Tenancy Impact

- **Tenant scoping impact**: No new tenant-owned data is introduced. The baseline requires frontend tenant context to come only from authenticated API responses or approved persisted session metadata.
- **Cross-tenant or platform access impact**: No new cross-tenant or platform access is approved. Any future platform-wide or support access UI must be specified separately and backed by approved OpenAPI behavior.
- **Soft delete impact**: No soft-delete behavior is introduced. Future modules must render lifecycle, inactive, delete, restore, and conflict states only where feature specs and OpenAPI contracts approve them.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The frontend architecture baseline MUST define SchoolMaster as a JavaScript Vue 3 SPA using Composition API with `<script setup>`, Vue Router, Pinia, Axios, Element Plus, Element Plus Icons, Vue I18n, and Tailwind CSS as the approved frontend foundation.
- **FR-002**: The baseline MUST explicitly exclude TypeScript requirements from this slice and MUST use JavaScript file conventions for frontend services, stores, composables, router modules, contracts, tests, and app entrypoints.
- **FR-003**: The baseline MUST define clear responsibility boundaries for layouts, pages, components, composables, stores, services, contracts, constants, and utilities.
- **FR-004**: The baseline MUST require frontend features to organize code by domain or feature where doing so improves clarity, reuse, and long-term scalability.
- **FR-005**: The baseline MUST define reusable CRUD-oriented admin workflow patterns for list pages, search and filters, data tables, forms, dialogs, pagination, loading states, empty states, error states, and validation surfaces.
- **FR-006**: The baseline MUST require HTTP access to be isolated in frontend services and MUST prohibit route views or UI components from depending directly on transport details.
- **FR-007**: The baseline MUST require stores to coordinate shared client state without becoming transport layers that bypass approved services.
- **FR-008**: The baseline MUST require composables to coordinate reusable view behavior such as filters, pagination, form draft state, permission checks, and CRUD orchestration without becoming hidden global stores.
- **FR-009**: The baseline MUST require frontend behavior to consume only approved OpenAPI-backed `/api/v1` endpoints, fields, filters, status meanings, error envelopes, tenant semantics, and authorization behavior.
- **FR-010**: The baseline MUST require future frontend feature specs to identify which approved contract behavior they consume and to block implementation when needed contract behavior is missing.
- **FR-011**: The baseline MUST define Element Plus as the primary UI component library for common application primitives such as forms, tables, dialogs, navigation, feedback, and data display.
- **FR-012**: The baseline MUST require PascalCase Element Plus component tags in Vue templates, such as `ElButton`, `ElForm`, `ElTable`, and `ElDialog`, and MUST reject kebab-case Element Plus tags.
- **FR-013**: The baseline MUST define Tailwind CSS responsibilities as layout, spacing, responsiveness, utility classes, and restrained visual refinement around Element Plus components.
- **FR-014**: The baseline MUST prevent Tailwind usage from replacing Element Plus with an unapproved competing component system unless a future architecture decision approves the deviation.
- **FR-015**: The baseline MUST require client-side permission and role checks to be treated as navigation and visibility aids only; backend authorization remains authoritative.
- **FR-016**: The baseline MUST require active tenant or school context to come from authenticated backend responses or approved persisted session metadata, not from invented client-side assumptions.
- **FR-017**: The baseline MUST define repository sequencing where `schoolmaster-specs` approves architecture and future feature behavior before `schoolmaster-frontend` implements that behavior.
- **FR-018**: The baseline MUST NOT approve concrete System Administrator layout behavior, authentication screens, business module workflows, backend implementation, undocumented API behavior, reporting behavior, assessment behavior, or platform support behavior in this slice.
- **FR-019**: The baseline MUST require future frontend features to cover loading, empty, validation, unauthorized, forbidden, tenant-mismatch, not-found, inactive-record, and conflict states where those states apply.
- **FR-020**: The baseline MUST remain compatible with existing backend-first specifications and MUST not modify backend API behavior without a separate approved OpenAPI change.
- **FR-021**: The baseline MUST require reusable layouts, forms, tables, dialogs, navigation, and feedback states to meet a WCAG 2.1 AA accessibility baseline.
- **FR-022**: The baseline MUST be internationalization-ready with Vue I18n by requiring reusable UI text to be centralized, while requiring only one launch language in this slice.
- **FR-023**: The baseline MUST define standard frontend observability boundaries through a client error boundary, service error normalization, and user-action trace points.
- **FR-024**: The baseline MUST use Element Plus Icons from `@element-plus/icons-vue` as the default icon source for navigation, actions, feedback, and shared UI primitives, using custom icon assets only when no suitable package icon exists.
- **FR-025**: The baseline MUST represent JavaScript frontend contract definitions with JSDoc typedefs plus service mapping helpers under `src/contracts/`, without introducing TypeScript requirements.

### Key Entities *(include if feature involves data)*

- **FrontendArchitectureBaseline**: Durable set of frontend architecture rules that define the approved SPA foundation, repository sequencing, and cross-feature boundaries.
- **ApplicationShellBoundary**: Shared application frame responsibilities for route layout, session bootstrap, navigation, and global feedback surfaces; concrete layout implementation remains a later feature.
- **FeatureAreaBoundary**: Rule that future product areas own their pages, components, stores, services, contracts, and composables through feature-aligned folders while depending only on shared boundaries and approved services.
- **ServiceBoundary**: Contract-consumption layer that owns HTTP access, request/response mapping, pagination envelope handling, and error parsing for approved `/api/v1` behavior.
- **StateBoundary**: Pinia-based shared state ownership for session, tenant context, permissions, navigation, notifications, and feature state shared across views.
- **ReusableCrudPattern**: Shared admin workflow pattern covering list, filter, table, form, dialog, pagination, loading, empty, error, and validation behavior.
- **UiPrimitiveSystem**: Approved UI composition approach using Element Plus as the primary component system with Tailwind for layout and restrained visual refinement.
- **FrontendContractDefinition**: Shared frontend data-shape definition using JSDoc typedefs plus service mapping helpers under `src/contracts/` to document consumed API envelopes, pagination models, errors, and module data shapes without introducing TypeScript requirements.
- **FrontendObservabilityBoundary**: Shared frontend diagnostic boundary for client error capture, normalized service errors, and user-action trace points without requiring a full telemetry platform in this slice.
- **IconPrimitiveSystem**: Approved iconography approach using Element Plus Icons from `@element-plus/icons-vue` for navigation, actions, feedback, and shared UI primitives.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of future frontend feature specs can reference a single approved architecture baseline for application structure, state ownership, service boundaries, UI primitive rules, and JavaScript conventions.
- **SC-002**: Architecture review can map 100% of approved frontend folder responsibilities to exactly one documented responsibility area without unresolved overlap between pages, layouts, components, composables, stores, services, contracts, constants, and utilities.
- **SC-003**: Frontend review rejects 100% of direct component-level or route-level HTTP access when the behavior should pass through the approved service boundary.
- **SC-004**: Frontend review rejects 100% of undocumented route, field, filter, status, error, tenant, or authorization assumptions before implementation begins.
- **SC-005**: Frontend review confirms 100% of Element Plus component usage in templates follows PascalCase naming.
- **SC-006**: Future admin module specs can reuse baseline CRUD patterns for list, filter, table, form, dialog, pagination, loading, empty, and error states without redefining the architecture foundation.
- **SC-007**: Frontend planning can identify whether each future frontend slice changes only `schoolmaster-frontend`, only `schoolmaster-specs`, or both, before implementation starts.
- **SC-008**: Architecture review confirms no TypeScript requirement, backend implementation task, concrete business module workflow, or undocumented API behavior is approved by this slice.
- **SC-009**: Accessibility review confirms reusable layouts, forms, tables, dialogs, navigation, and feedback states satisfy WCAG 2.1 AA expectations before dependent frontend features reuse them.
- **SC-010**: Frontend review confirms reusable UI text is centralized through Vue I18n for future localization and no full multilingual delivery is required by this baseline slice.
- **SC-011**: Frontend review confirms reusable error handling, service error mapping, and user-action trace points are defined before dependent frontend features reuse the baseline.
- **SC-012**: Frontend review confirms shared icon usage comes from Element Plus Icons unless a documented exception requires a custom asset.
- **SC-013**: Frontend review confirms JavaScript contract definitions use JSDoc typedefs plus service mapping helpers in `src/contracts/` instead of TypeScript types or documentation-only shapes.

## Assumptions

- Existing frontend architecture documentation under `docs/` is the starting baseline and may be refined by this feature during planning.
- The first concrete frontend delivery after this baseline will be the System Administrator shell and dashboard foundation unless the roadmap changes.
- The frontend repository is separate from this specification repository and consumes this baseline through an approved `schoolmaster-specs` revision.
- Backend API behavior remains contract-first; frontend implementation cannot use routes, fields, filters, status values, or error semantics that are not approved by OpenAPI.
- JavaScript is the frontend language baseline for this project; TypeScript is intentionally out of scope unless a future approved architecture change revisits the decision.
- Element Plus is the primary UI component library, while Tailwind is used for layout, responsive behavior, spacing, and restrained visual refinement.
- Element Plus Icons from `@element-plus/icons-vue` are available as the default icon source for shared UI primitives.
- Tenant and school context remain sensitive and must be resolved from authenticated backend behavior or approved session metadata.
- The launch frontend language is a product decision for later feature delivery; this baseline only requires reusable text to be localization-ready through Vue I18n.
- Full analytics, performance telemetry, and external error-reporting integrations remain future decisions; this baseline only requires diagnostic boundaries that future features can reuse.
