# Data Model: Frontend Architecture Baseline

This feature does not introduce database entities. The model below defines architecture concepts and relationships that future frontend feature specs must consume.

## FrontendArchitectureBaseline

**Purpose**: Durable frontend rule set for the SchoolMaster Vue 3 SPA.

**Fields**:

- `approvedStack`: JavaScript, Vue 3 Composition API with `<script setup>`, Vue Router, Pinia, Axios, Element Plus, Element Plus Icons, Vue I18n, Tailwind CSS.
- `repositorySequence`: `schoolmaster-specs` approves architecture and behavior before `schoolmaster-frontend` implements it.
- `excludedScope`: TypeScript requirements, backend implementation, undocumented API behavior, concrete business module behavior.
- `qualityBaseline`: WCAG 2.1 AA for reusable UI, API-first consumption, centralized reusable UI text, service error normalization.

**Relationships**:

- Owns `ApplicationShellBoundary`, `FeatureAreaBoundary`, `ServiceBoundary`, `StateBoundary`, `ReusableCrudPattern`, `UiPrimitiveSystem`, `FrontendContractDefinition`, `FrontendObservabilityBoundary`, and `IconPrimitiveSystem`.

**Validation rules**:

- Must not approve behavior that consumes undocumented `/api/v1` routes, fields, filters, status meanings, error envelopes, tenant semantics, or authorization rules.
- Must reject TypeScript requirements unless a later architecture change supersedes this baseline.

## ApplicationShellBoundary

**Purpose**: Shared application frame responsibilities that future layouts consume.

**Fields**:

- `routeLayoutSelection`: Route metadata selects the layout.
- `sessionBootstrap`: Authenticated session restoration and current-user hydration.
- `navigationState`: Role-aware navigation, active route, sidebar state.
- `globalFeedback`: Session expired, maintenance, contract-safe error states.

**Relationships**:

- Uses `StateBoundary` for shared shell state.
- Uses `ServiceBoundary` only through approved session or navigation services.

**Validation rules**:

- Must not define concrete System Administrator layout implementation in this slice.
- Must not treat client-side permission checks as authoritative authorization.

## FeatureAreaBoundary

**Purpose**: Feature-aligned organization for future product areas.

**Fields**:

- `pages`: Route-facing views.
- `components`: Feature-local visual components.
- `composables`: Feature-local view coordination.
- `stores`: Shared feature state.
- `services`: Feature-specific API access through approved contracts.
- `contracts`: JSDoc typedefs and service mapping helpers for JavaScript data shapes.

**Relationships**:

- Depends on shared UI primitives and approved services.
- Must not depend on another feature area's private files.

**Validation rules**:

- Feature behavior belongs in the feature spec, not in the architecture baseline.

## ServiceBoundary

**Purpose**: Contract-consumption layer for HTTP access.

**Fields**:

- `apiBase`: Approved `/api/v1` behavior only.
- `requestMapping`: Maps UI query/form state to documented contract fields.
- `responseMapping`: Maps documented response envelopes to frontend contract shapes.
- `errorNormalization`: Converts API errors into standard frontend error states.
- `paginationMapping`: Handles approved pagination envelope semantics.

**Relationships**:

- Uses Axios.
- Called by stores or composables, not directly by presentational components.

**Validation rules**:

- Must not call undocumented endpoints.
- Must not expose hidden backend fields as UI behavior.
- Must not contain presentation state.

## StateBoundary

**Purpose**: Pinia ownership for shared client state.

**Fields**:

- `authSession`: Authenticated user and session state.
- `tenantContext`: Active school or tenant context from approved sources.
- `permissions`: Visibility and navigation permissions.
- `sidebar`: Responsive sidebar state.
- `notifications`: Global user-facing notifications.
- `featureState`: State shared across related feature views.

**Relationships**:

- Calls services for remote data.
- Feeds layouts, pages, and composables.

**Validation rules**:

- Must not bypass services for HTTP access.
- Must not become a hidden global dump for local component state.

## ReusableCrudPattern

**Purpose**: Shared admin workflow foundation for repeated CRUD-heavy modules.

**Fields**:

- `listPage`: Page header, filters, table, pagination, action surfaces.
- `filters`: Search and filter state, route query synchronization where approved.
- `table`: Data table, loading, empty, error, selection, row actions.
- `form`: Create/edit form draft state and validation display.
- `dialog`: Confirmation, create/edit modal where the feature approves modal UX.
- `pagination`: Page, per-page, total, and approved pagination semantics.
- `states`: Loading, empty, validation, unauthorized, forbidden, tenant mismatch, not found, inactive record, conflict.

**Relationships**:

- Uses `ServiceBoundary` for remote behavior.
- Uses `UiPrimitiveSystem` for Element Plus components and Tailwind layout.

**Validation rules**:

- Must not define resource-specific business rules.
- Deviations must be documented in the consuming feature spec or architecture update.

## UiPrimitiveSystem

**Purpose**: Approved UI composition approach.

**Fields**:

- `componentLibrary`: Element Plus.
- `componentNaming`: PascalCase Element Plus tags only.
- `layoutStyling`: Tailwind CSS for spacing, layout, responsiveness, and restrained refinement.
- `textLocalization`: Vue I18n for reusable UI text.
- `accessibility`: WCAG 2.1 AA baseline for reusable UI.

**Relationships**:

- Uses `IconPrimitiveSystem` for icons.
- Used by layouts, reusable components, and feature components.

**Validation rules**:

- Must reject kebab-case Element Plus tags.
- Must not rebuild a competing Tailwind-only component system.

## FrontendContractDefinition

**Purpose**: Shared frontend representation of consumed API shapes using JSDoc typedefs plus service mapping helpers without TypeScript requirements.

**Fields**:

- `apiEnvelope`: Documented success and error structures.
- `paginationModel`: Documented pagination shape.
- `errorModel`: Machine-readable code, message, field validation details where applicable.
- `moduleShapes`: Resource data shapes consumed by future modules.
- `mappingHelpers`: Service-level helpers that convert documented API envelopes into frontend contract shapes.

**Relationships**:

- Mirrors OpenAPI-backed behavior.
- Used by services, stores, composables, and tests.

**Validation rules**:

- Must not introduce fields or status meanings absent from OpenAPI.
- Must not rely on TypeScript types or documentation-only shapes for frontend contract definitions.

## FrontendObservabilityBoundary

**Purpose**: Standard frontend diagnostics boundary.

**Fields**:

- `clientErrorBoundary`: Captures render or route-level failures where applicable.
- `serviceErrorNormalization`: Maps transport and API errors consistently.
- `userActionTracePoints`: Records meaningful UI action context for diagnosis.

**Relationships**:

- Used by services, stores, route workflows, and global feedback surfaces.

**Validation rules**:

- Must not require a full telemetry platform in this slice.
- Must not expose sensitive tenant, authorization, or personal data in traces.

## IconPrimitiveSystem

**Purpose**: Shared iconography standard.

**Fields**:

- `package`: `@element-plus/icons-vue`.
- `usage`: Navigation, actions, feedback, empty states, and shared UI primitives.
- `exceptions`: Custom assets only when no suitable package icon exists.

**Relationships**:

- Used by `UiPrimitiveSystem`, navigation, buttons, feedback, and reusable components.

**Validation rules**:

- Must avoid mixing unrelated icon libraries without an approved architecture change.
