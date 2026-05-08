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

<!--
  ACTION REQUIRED: Replace the content in this section with the technical details
  for the project. The structure here is presented in advisory capacity to guide
  the iteration process.
-->

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
**Performance Goals**: Core administration and teaching workflows feel
responsive for normal school usage; launch scope should support concurrent use
across multiple schools without tenant leakage or degraded core task completion  
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

**Gate Status**: PASS before research. All constitution constraints are
addressed in this plan, and no deviations require exception handling.

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
<!--
  ACTION REQUIRED: Replace the placeholder tree below with the concrete layout
  for this feature. Delete unused options and expand the chosen structure with
  real paths (e.g., apps/admin, packages/something). The delivered plan must
  not include Option labels.
-->

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Repositories/
│   └── Services/
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
service-based API access and centralized state coordination.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations currently identified |
