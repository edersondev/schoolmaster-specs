# Implementation Plan: School List Filters

**Branch**: `030-school-list-filters` | **Date**: 2026-07-09 | **Spec**: `specs/030-school-list-filters/spec.md`  
**Input**: Feature specification from `specs/030-school-list-filters/spec.md`

## Summary

Add contract-first filtering to the Schools list by status, INEP code,
document/CNPJ, name, email, city, state, administrative type, legal nature,
management type, and pedagogical approach.

OpenAPI leads the additive `/api/v1/schools` query contract. Backend then
validates and applies filters within existing authorization and tenant
visibility. Frontend then adds Schools list filter controls, URL query-string
persistence, institutional option loading, empty-state behavior, and service
tests.

## Technical Context

**Language/Version**: PHP 8.x/Laravel 10+ backend; Vue 3 with TypeScript-capable Composition API frontend; OpenAPI YAML contract.  
**Primary Dependencies**: Laravel Form Requests, Policies, API Resources, Services, Eloquent/MySQL query filtering, DTO optional for normalized filter input; Vue Router query state, Pinia only for existing shell/session state, Axios services, Tailwind CSS, existing school lookup services; Redocly OpenAPI validation.  
**Storage**: Existing MySQL School, Address, and Institutional reference fields. No new persisted data. Frontend stores active filters in the page URL query string and transient route-local list state.  
**Testing**: Redocly/OpenAPI lint; PHPUnit feature/unit tests for school list filter validation, matching behavior, authorization, tenant visibility, pagination, sorting, and response shape; Vitest tests for school list service query serialization, route-query synchronization, filter controls, empty state, and clear behavior.  
**Target Platform**: Laravel JSON API under `/api/v1`; Vue 3 SPA administration UI.  
**Project Type**: Multi-repository web application feature spanning specs, backend API, and frontend SPA.  
**Performance Goals**: Authorized administrators can reduce a Schools list of at least 100 records to the intended matching set with any single requested filter in under 10 seconds during representative testing; filtered list refresh should show loading, results, or validation feedback within 2 seconds after service resolution in frontend tests.  
**Constraints**: OpenAPI changes before implementation; additive `/api/v1/schools` query contract; no new response fields; no undocumented `cnpj` query alias because `document` remains the canonical contract field; status and institutional filters accept one value each; categorical, INEP, and document filters use exact matching after normalization; name, email, city, and state use case-insensitive and accent-insensitive contains matching; multiple filters combine with AND semantics; applying/changing filters resets to page 1 and preserves sort; filters never expand authorized school visibility; no runtime lookup-management UI.  
**Scale/Scope**: One existing list endpoint, eleven new filter criteria, four institutional lookup option sources for frontend controls, one URL query persistence workflow, focused OpenAPI/backend/frontend tests across the specs, backend, and frontend repositories.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. The additive Schools list query filter
  contract starts in `api/paths/schools/index.yaml` and supporting parameter
  components before backend or frontend implementation.
- PASS: Repository impacts are separated. `schoolmaster-specs` owns the spec,
  OpenAPI, and generated design artifacts; `schoolmaster-backend` owns Laravel
  list validation/query behavior/resources/policies/tests; `schoolmaster-frontend`
  owns Vue list controls, route-query state, services, and tests.
- PASS: Backend design uses Laravel feature organization, Form Request
  validation for query parameters, Policies for list
  authorization, a School list Service for normalized filtering, API Resources
  for the existing paginated output, and UUID public identifiers. DTO usage is
  optional but recommended if normalized filters pass through more than one
  layer. Repository abstraction is not planned because filter access remains
  straightforward Eloquent query composition over existing fields and
  relationships.
- PASS: Frontend design uses Vue 3 Composition API, Vue Router query state,
  existing Pinia shell/session state only, Axios service modules, Tailwind CSS,
  existing admin-system list page/filter/query components, and service-isolated
  API access. The active Schools list route currently uses
  `src/pages/admin-system/schools/SchoolsListPage.vue`; implementation must
  extend that live route instead of creating a parallel unused list route.
- PASS: MySQL remains authoritative; School remains the tenant root; filters
  reduce only the already-authorized result set; no new cross-tenant path is
  introduced; soft-delete and lifecycle behavior are unchanged.
- PASS: API compatibility, authentication/authorization impact, paginated
  success envelope, validation error response, query persistence, and empty
  result behavior are documented.
- PASS: Verification spans OpenAPI lint, PHPUnit backend coverage, and Vitest
  frontend coverage for filter contract, validation, matching semantics,
  pagination/sorting interaction, tenant safety, URL persistence, empty state,
  and clear behavior.
- PASS: No constitution deviations.

## Project Structure

### Documentation (this feature)

```text
specs/030-school-list-filters/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── school-list-filters-contract.md
└── tasks.md             # Phase 2 output from /speckit-tasks
```

### Source Code (target repositories)

```text
# schoolmaster-backend (Laravel API)
app/
├── DTOs/School/
├── Http/
│   ├── Controllers/Api/V1/
│   ├── Requests/Api/V1/
│   └── Resources/
├── Models/
├── Policies/
└── Services/School/
routes/api.php
tests/
├── Feature/School/
└── Unit/School/

# schoolmaster-frontend (Vue 3 SPA)
src/
├── components/admin-system/schools/
├── composables/admin-system/
├── modules/schools/              # existing create/edit form + lookup helpers
├── pages/admin-system/schools/
├── services/admin-system/
├── router/modules/
└── stores/
tests/
├── unit/admin-system/administration/
└── unit/schools/

# schoolmaster-specs
api/
├── openapi.yaml
├── paths/schools/
└── components/parameters/schools/
specs/030-school-list-filters/
```

**Structure Decision**: Keep the specifications repository as the source of
truth for the query-filter contract and design artifacts. Backend and frontend
implementations must use the shared `030-school-list-filters` feature
identifier and consume only documented query parameter names. Frontend list
implementation extends the current JavaScript admin-system list stack
(`SchoolsListPage.vue`, `SchoolFilters.vue`, `useAdminListQuery.js`, and
`services/admin-system/schools.js`) so filter controls are wired to the route
already registered as `schoolsList`.

## Phase 0: Research

Research is complete in `specs/030-school-list-filters/research.md`.

Key decisions:

- Treat the Schools list filters as an additive contract-first change to
  `GET /api/v1/schools`.
- Use `document` as the API query parameter for the CNPJ filter; the UI may
  label it CNPJ.
- Use exact matching for status, institutional filters, INEP code, and
  document/CNPJ after normalization.
- Use case-insensitive and accent-insensitive contains matching for name,
  email, city, and state.
- Accept only one value per status or institutional filter.
- Combine active filters with AND semantics.
- Persist active frontend filters in the page URL query string.
- Reset to page 1 when filters change and preserve active sort.
- Preserve current paginated response shape and tenant visibility.
- Verify with OpenAPI lint, PHPUnit school list tests, and Vitest school list
  service/composable/component tests.

## Phase 1: Design And Contracts

Design artifacts are complete:

- `specs/030-school-list-filters/data-model.md`
- `specs/030-school-list-filters/contracts/school-list-filters-contract.md`
- `specs/030-school-list-filters/quickstart.md`

Required OpenAPI updates:

- `GET /api/v1/schools` documents new query filters: `status`, `inep_code`,
  `document`, `name`, `email`, `city`, `state`, `administrative_type_id`,
  `legal_nature_id`, `management_type_id`, and `pedagogical_approach_id`.
- `document` is the canonical CNPJ filter field; no `cnpj` query alias is
  introduced.
- Categorical filters accept one value each and validate unsupported values
  with the standard validation error response.
- INEP and document filters compare normalized digits exactly.
- Name, email, city, and state filters are contains searches after trimming and
  case/accent normalization.
- Multiple filters combine with AND semantics.
- Filtered responses preserve the existing paginated Schools collection shape.
- OpenAPI describes pagination/sorting interaction, authentication,
  authorization, and compatibility as additive.

Post-design Constitution Check: PASS. The research, data model, contract, and
quickstart preserve API-first delivery, repository sequencing, Laravel/Vue
boundaries, tenant safety, compatibility expectations, and required OpenAPI,
PHPUnit, and Vitest verification with no deviations.

## Complexity Tracking

No constitution violations.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
