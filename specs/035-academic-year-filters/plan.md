# Implementation Plan: Academic Year List Filters

**Branch**: `035-academic-year-filters` | **Date**: 2026-08-15 | **Spec**: `specs/035-academic-year-filters/spec.md`  
**Input**: Feature specification from `specs/035-academic-year-filters/spec.md`

## Summary

Add contract-first academic-year filtering by partial name, inclusive
overlapping date range, and lifecycle status. The shared OpenAPI contract leads,
the Laravel API validates and applies filters inside the resolved school, and
the Vue administration page exposes a dedicated submitted search form with URL
query persistence. Existing pagination, ordering, authorization, response
envelopes, lifecycle visibility, and tenant isolation remain unchanged.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13.8; JavaScript with Vue 3.5; OpenAPI 3.0 YAML.  
**Primary Dependencies**: Laravel Form Requests, Eloquent, existing service and API resource layers; Vue Router, Axios service modules, Element Plus 2.14, Tailwind CSS 4, existing admin list composables.  
**Storage**: Existing MySQL `academic_years` fields only; URL query state for frontend filters; no schema change.  
**Testing**: Redocly contract lint; PHPUnit 12 feature coverage; Vitest 4 component, composable, and service coverage.  
**Target Platform**: Laravel JSON API under `/api/v1` and Vue 3 administration SPA.  
**Project Type**: Multi-repository web feature spanning specification, backend, and frontend repositories.  
**Performance Goals**: An administrator can reduce a representative 100-record academic-year list to the intended set within two interactions; filtered responses retain existing list performance expectations.  
**Constraints**: Contract before implementation; optional additive query parameters only; partial case-insensitive name match; complete inclusive date range required; overlap comparison uses `academic_year.start_date <= date_to` and `academic_year.end_date >= date_from`; all populated filters use AND semantics; invalid input returns the existing validation envelope; filters never widen school scope.  
**Scale/Scope**: One existing endpoint, four query parameters (`name`, `date_from`, `date_to`, `status`), one dedicated filter component, one route-query workflow, and focused cross-repository verification.

## Constitution Check

*GATE: Passed before research and re-checked after design.*

- **PASS — API first**: `api/paths/academic-years/index.yaml`, shared parameter
  components, aggregate OpenAPI, and active platform OpenAPI are updated and
  linted before runtime implementation.
- **PASS — repository sequencing**: `schoolmaster-specs` owns contract/design,
  `schoolmaster-backend` implements validated tenant-scoped behavior, and
  `schoolmaster-frontend` consumes the published parameter names.
- **PASS — backend architecture**: A dedicated list Form Request validates query
  input; the existing controller remains orchestration-only; the existing
  `AcademicYearService` applies straightforward Eloquent predicates after tenant
  resolution and authorization; existing Policy and API Resource behavior is
  preserved. No DTO is required because validated filters cross only controller
  to service and contain four scalar values. No Repository is justified for
  simple predicates over one table. Public UUID behavior is unchanged.
- **PASS — frontend architecture**: The route page remains a composition surface,
  the dedicated filter component owns only draft interaction state, the shared
  query composable owns parse/serialize rules, and the Axios service remains the
  transport boundary. Existing Pinia session state, Vue Router, Element Plus,
  and Tailwind conventions remain intact.
- **PASS — tenancy and data**: MySQL remains authoritative; School remains tenant
  root; every predicate reduces the already resolved school query; no new
  platform or cross-tenant path exists; soft-delete behavior is unchanged.
- **PASS — compatibility and errors**: Query parameters are optional and
  additive. Success shape, pagination, permissions, tenant context, 401/403
  behavior, and 422 validation envelope remain unchanged.
- **PASS — verification**: Redocly validates both published contracts, PHPUnit
  covers filter semantics/validation/isolation, and Vitest covers query mapping,
  service requests, form behavior, persistence, combination, and reset.
- **PASS — deviations**: None.

## Project Structure

### Documentation (this feature)

```text
specs/035-academic-year-filters/
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── academic-year-list-filters-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md
```

### Source Code (target repositories)

```text
# schoolmaster-specs
api/
├── openapi.yaml
├── paths/academic-years/index.yaml
└── components/parameters/academic-years/
specs/001-schoolmaster-platform/contracts/openapi.yaml
specs/035-academic-year-filters/

# schoolmaster-backend
app/
├── Http/Controllers/Api/V1/AcademicYearController.php
├── Http/Requests/Api/V1/AcademicYearListRequest.php
└── Services/AcademicYears/AcademicYearService.php
tests/Feature/Api/V1/AcademicYearManagementTest.php

# schoolmaster-frontend
src/
├── components/admin-system/academic-years/AcademicYearFilters.vue
├── composables/admin-system/useAdminListQuery.js
├── locales/administration.js
├── pages/admin-system/academic-years/AcademicYearsListPage.vue
└── services/admin-system/academic-years.js
tests/unit/admin-system/administration/
├── components/
├── composables/
├── pages/
└── services/
```

**Structure Decision**: Extend the live administration list stack and existing
academic-year endpoint. Add one backend request class and one dedicated frontend
filter component; avoid new stores, repositories, dependencies, routes, response
fields, or database migrations.

## Phase 0: Research

Research decisions are captured in `research.md`: parameter naming, inclusive
overlap semantics, contains matching, status domain, validation boundaries,
query persistence, and component ownership are resolved with no open questions.

## Phase 1: Design and Contracts

- `data-model.md` defines the transient filter set and its relationship to the
  existing Academic Year entity.
- `contracts/academic-year-list-filters-contract.md` defines request parameters,
  combination rules, validation, responses, tenancy, frontend interaction, and
  verification obligations.
- `quickstart.md` defines contract, backend, frontend, and manual acceptance
  gates.
- OpenAPI source and published copies are updated before backend/frontend code.

## Complexity Tracking

No constitution violations or complexity exceptions.
