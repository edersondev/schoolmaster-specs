# Implementation Plan: SchoolMaster Platform Foundation

**Branch**: `001-schoolmaster-platform` | **Date**: 2026-05-08 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-schoolmaster-platform/spec.md`

**Note**: This template is filled in by the `/speckit-plan` command. See `.specify/templates/plan-template.md` for the execution workflow.

## Summary

Define the v1 implementation design for SchoolMaster as a multi-tenant school
management SaaS delivered across separate specification, backend, and frontend
repositories. The plan establishes the contract-first module boundaries, tenant
data model, and delivery sequencing for the initial modules: authentication and
authorization, school and user administration, academic structure, guardians,
teacher content, questionnaires, learning sets, grades, attendance, and
reports.

## Technical Context

**Language/Version**: PHP 8.x for backend, TypeScript with Vue 3 SPA for
frontend, Markdown/OpenAPI for specification artifacts  
**Primary Dependencies**: Laravel API stack, Vue 3 SPA stack, Pinia, Vue
Router, Axios, Tailwind CSS, OpenAPI 3.1  
**Storage**: MySQL for transactional data, object storage for uploaded teacher
content, specification repository for business rules and API contracts  
**Testing**: PHPUnit for backend, Vitest for frontend services/stores/composables,
OpenAPI contract verification for published endpoints  
**Target Platform**: Browser-based SaaS consumed by school staff and students,
with backend REST services hosted on web infrastructure  
**Project Type**: Multi-repository web application platform with API backend,
SPA frontend, and shared specification/contracts repository  
**Performance Goals**: Core school administration and teaching workflows use
tenant-safe indexed queries on `school_id`, private object storage for teacher
content, and report generation that may run asynchronously when immediate
response time would degrade interactive workflows  
**Constraints**: Strict tenant isolation, `/api/v1` versioning, OpenAPI-first
delivery, UUID identifiers for cross-boundary entities, active/inactive status
tracking, documented business rules before implementation  
**Scale/Scope**: Initial release for multiple schools with platform
administration, school administration, teacher operations, student self-view,
and launch-scope reporting across the listed v1 modules

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- OpenAPI impact is identified, and any API change starts from the contract.
- Backend, frontend, and specification repository impacts are called out
  separately, including delivery sequencing when more than one repo changes.
- Backend design uses Laravel feature organization, Service Layer, Form
  Requests, Policies, API Resources, UUID-based public identifiers, and
  explicit DTO or Repository decisions where applicable.
- Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind CSS,
  Axios-based service modules, feature modules, and service-isolated API access
  where applicable.
- MySQL, tenant-scoping impact, cross-tenant access rules, and soft-delete
  expectations are documented for affected entities and workflows.
- API compatibility, authentication or authorization impact, and success or
  error response expectations are documented for changed endpoints.
- PHPUnit, Vitest, and API contract verification cover all changed critical
  business flows across the affected repositories.
- Any constitution deviation is recorded in Complexity Tracking with approval
  rationale.

**Gate Status**: CONDITIONAL PASS. Constitution intent is covered, but
implementation should not begin until the OpenAPI contract is expanded to
implementation-grade detail for P1, tenancy strategy is explicit, and data
lifecycle decisions are reflected in tasks and validation.

## Tenancy Strategy

- `School` is the tenant root for all school-scoped operations.
- Every tenant-owned table carries `school_id` unless ownership is inherited
  through a strictly school-owned parent record with validated traversal.
- Tenant scope is enforced at service, query or repository, policy, contract,
  and test layers; the default access stance is deny by default.
- Platform administrators use explicit policy paths for cross-tenant
  operations; no implicit override is allowed.
- Teacher content stored outside MySQL uses private object storage paths
  prefixed by tenant identity and guarded by API authorization.

## Contract Strategy

- `schoolmaster-specs` owns the OpenAPI contract as the source of truth for
  payloads, status codes, auth expectations, pagination, filtering, sorting,
  tenancy semantics, and error envelopes.
- P1 endpoints for authentication, schools, users, roles, permissions,
  academic years, academic periods, and guardians must be implementation-ready
  in OpenAPI before backend or frontend coding starts for that slice.
- US2 and US3 extend the published contract additively and must preserve the
  established response envelope and error conventions.
- Cross-repository work uses the shared feature id `001-schoolmaster-platform`
  for traceability across specs, backend, and frontend delivery.

## Data Lifecycle Strategy

- Schools, users, roles, academic structures, guardians, teacher content,
  questionnaires, learning sets, grades, attendance, and report requests use
  explicit business status fields.
- Recoverable tenant-owned operational records use status plus soft deletes
  unless a permanent deletion path is later approved and documented.
- Historical academic records that affect reporting are retained for audit and
  may become operationally immutable after closure of the relevant academic
  period or year according to module business rules.

## Project Structure

### Documentation (this feature)

```text
specs/001-schoolmaster-platform/
├── plan.md              # This file (/speckit-plan command output)
├── research.md          # Phase 0 output (/speckit-plan command)
├── data-model.md        # Phase 1 output (/speckit-plan command)
├── quickstart.md        # Phase 1 output (/speckit-plan command)
├── contracts/
│   └── openapi.yaml     # Initial platform API contract skeleton
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── Exceptions/
│   ├── DTOs/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Repositories/
│   └── Services/
├── database/
│   └── migrations/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/
    └── Unit/

# Frontend repository (Vue 3)
schoolmaster-frontend/
├── src/
│   ├── modules/
│   ├── router/
│   ├── services/
│   └── stores/
└── tests/
    ├── modules/
    └── router/

# Contracts and shared delivery artifacts
schoolmaster-specs/
└── specs/001-schoolmaster-platform/
```

**Structure Decision**: Use `schoolmaster-specs` as the source of truth for
business requirements and OpenAPI contracts, `schoolmaster-backend` for
tenant-aware Laravel API implementation, and `schoolmaster-frontend` for the
Vue 3 SPA that consumes only the published REST endpoints. Backend code is
organized by feature with Service Layer, Requests, Policies, Resources, DTOs,
and selective Repositories. Frontend code is organized by feature modules with
service-based API access and centralized state coordination. Cross-repository
delivery uses the shared feature identifier `001-schoolmaster-platform` in
branching, issue references, and implementation notes.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations currently identified |
