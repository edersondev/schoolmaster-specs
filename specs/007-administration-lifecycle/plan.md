# Implementation Plan: Backend Administration Lifecycle Management

**Branch**: `007-administration-lifecycle` | **Date**: 2026-05-25 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/007-administration-lifecycle/spec.md`

**Note**: This plan defines the backend implementation boundary after `006-backend-student-enrollment`. It does not decompose tasks and does not authorize product behavior outside the specification and OpenAPI contract.

## Summary

Implement the backend administration lifecycle slice for existing platform and school-administration resources. The backend must expand the contract before implementation, then expose only approved `/api/v1` operations for individual detail, update, activation, deactivation, soft delete, and restore actions for schools, users, roles, academic years, academic periods, and guardians where allowed. Selected all-or-nothing bulk lifecycle actions are limited to school-owned users, roles, academic years, academic periods, and guardians. The design preserves tenant-by-column isolation with `School` as the tenant root and `school_id` for school-owned records, separates platform-scoped school lifecycle from school-owned administration lifecycle, keeps soft-deleted records recoverable for history and dependency checks, requires documented conflict behavior, and blocks account lifecycle, roster, teacher correction, guardian self-service, reporting expansion, frontend implementation, permanent purge, and undocumented APIs.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, school-admin, teacher-workflow, student-reporting, and student-enrollment slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for lifecycle and bulk inputs, repositories or query objects only for complex tenant-scoped lookup/dependency checks, PHPUnit, Redocly CLI for contract validation  
**Storage**: MySQL for school, user, role, academic year, academic period, guardian, status fields, soft-delete metadata, lifecycle history, role assignments, guardian associations, academic dependencies, and audit-relevant actor/effective-date fields  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests must run with `docker exec schoolmaster-backend-app-1 php artisan test`  
**Target Platform**: API-only backend service consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned data access for 100% of protected lifecycle operations; detail/update/lifecycle writes complete atomically with lifecycle history; bulk lifecycle requests are bounded by documented limits and either change every selected record or no records  
**Constraints**: OpenAPI expansion required before backend exposure, no undocumented endpoints or fields, no product Blade views, no frontend implementation, no account invitation/password recovery behavior, no classroom/course/section/roster model, no teacher correction workflow, no guardian self-service, no report lifecycle expansion, no permanent purge, UUID public identifiers, `school_id` as the v1 tenant column, contract-aligned response envelopes only  
**Scale/Scope**: Proposed boundary covers individual detail, update, activate, deactivate, soft delete, and restore operations for schools, users, roles, academic years, academic periods, and guardians; selected bulk lifecycle operations are limited to school-owned users, roles, academic years, academic periods, and guardians. Exact operation IDs, paths, payloads, response envelopes, dependency conflicts, and bulk limits must be finalized in OpenAPI before implementation

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The current contract does not publish most administration lifecycle operations, so this plan requires contract expansion before backend implementation.
- **Repository impact**: PASS. `schoolmaster-specs` leads the plan and contract boundary, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized Laravel controllers, Form Requests, Policies, API Resources, Services, UUID identifiers, DTOs for coordinated lifecycle/bulk inputs, and repositories/query objects only for complex tenant-scoped reads and dependency checks.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. School-owned resources remain scoped by `school_id`, school lifecycle is platform-scoped, cross-tenant access remains deny-by-default, and recoverable business records use soft deletes.
- **API compatibility and auth**: PASS. The plan is additive, requires explicit platform or school administration permission by operation, and preserves platform/school authorization separation.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/007-administration-lifecycle/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-administration-lifecycle.md
└── checklists/
    └── requirements.md
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── AdministrationLifecycle/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── AdministrationLifecycle/
│   └── Repositories/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/
    └── Unit/

# Frontend repository (Vue 3)
schoolmaster-frontend/
└── No implementation changes in this feature.

# Contracts and shared delivery artifacts
schoolmaster-specs/
├── api/openapi.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
└── specs/007-administration-lifecycle/
```

**Structure Decision**: Keep this as a backend-only implementation slice with specification artifacts in `schoolmaster-specs`. Backend code should follow the existing Laravel API structure used by previous slices, with lifecycle rules in services, request validation in Form Requests, authorization in Policies, and controllers limited to orchestration. Frontend work is explicitly deferred until the backend behavior is contract-compliant and ready to consume.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- The next backend slice is administration lifecycle management, not account lifecycle, roster, teacher correction, guardian self-service, reporting, or frontend delivery.
- OpenAPI must be expanded before backend implementation because most lifecycle operations are not currently published.
- School lifecycle operations are platform-scoped; school-owned resource lifecycle operations are school-scoped and require active permitted school context.
- Delete behavior is recoverable soft delete only; permanent purge remains out of scope.
- Lifecycle changes must record history and preserve dependent academic, authorization, guardian, student enrollment, report, and audit references.
- Bulk lifecycle actions are narrow, bounded, one resource type, one action, one scope, and all-or-nothing.
- Dependency conflicts must be contract-visible and block invalid activation, deactivation, delete, and restore transitions.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, dependency checks, and lifecycle transitions.
- [contracts/backend-administration-lifecycle.md](./contracts/backend-administration-lifecycle.md): proposed operation boundary, contract expansion requirements, tenant and authorization rules, response dependencies, blocked contract gaps, and verification expectations.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. The docs identify `schoolmaster-specs` as source of truth and `schoolmaster-backend` as the only implementation target for this slice.
- **Backend architecture**: PASS. Data and contract notes preserve Service Layer, Form Request, Policy, Resource, DTO, Repository/query object, UUID, lifecycle-history, soft-delete, and tenant-scope expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. Every affected school-owned entity is scoped to `school_id`, school lifecycle remains platform-scoped, and platform access does not bypass school-owned lifecycle permissions.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, and preserves platform/school authorization separation.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
