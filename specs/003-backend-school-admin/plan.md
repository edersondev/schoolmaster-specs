# Implementation Plan: Backend School Administration Foundation

**Branch**: `003-backend-school-admin` | **Date**: 2026-05-17 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/003-backend-school-admin/spec.md`

**Note**: This plan defines the next backend implementation boundary after `002-backend-api-foundation`. It does not decompose tasks and does not authorize product behavior outside the published OpenAPI contract.

## Summary

Implement the remaining P1 school administration backend foundation for users, roles, permissions, academic years, academic periods, and guardians. The backend must consume the existing published `/api/v1` list/create contract operations, enforce tenant-by-column isolation with `School` as the tenant root and `school_id` for school-owned records, preserve platform versus school authorization boundaries, and verify response envelopes, validation, inactive-status handling, and tenant failure behavior.

The plan intentionally blocks update, detail, deactivate, delete, invitation, password-management, student-profile creation, teacher workflow, student self-service, and reporting behavior until `/specs` and OpenAPI document those operations.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by the backend foundation  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, optional DTOs for coordinated create workflows, PHPUnit, Redocly CLI for contract validation  
**Storage**: MySQL for users, role assignments, permission definitions, academic years, academic periods, guardians, guardian-student associations, and soft-delete/status history where recoverability is required  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`  
**Target Platform**: API-only backend service consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned data access for 100% of protected operations in the slice; paginated list operations remain bounded by the documented `page` and `per_page` contract; authorization and validation failures return deterministic envelopes suitable for frontend consumption  
**Constraints**: No undocumented endpoints or fields, no product Blade views, no P2/P3 workflows, no direct per-user permissions, no system-administrator school-scope bypass, no cross-tenant associations, UUID public identifiers, `school_id` as the v1 tenant column, contract-aligned response envelopes only  
**Scale/Scope**: Eleven operation IDs: `listUsers`, `createUser`, `listRoles`, `createRole`, `listPermissions`, `listAcademicYears`, `createAcademicYear`, `listAcademicPeriods`, `createAcademicPeriod`, `listGuardians`, and `createGuardian`

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The slice maps to already published aggregate and platform feature contract operations. Any operation beyond the listed list/create surface requires a contract update first.
- **Repository impact**: PASS. `schoolmaster-specs` leads the plan and contract boundary, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized Laravel controllers, Form Requests, Policies, API Resources, Services, UUID identifiers, DTOs for multi-field create workflows where useful, and repositories/query objects only for complex tenant-scoped access.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. Affected school-owned records use `school_id`; tenant context must resolve before validation, authorization, data access, persistence, or response shaping.
- **API compatibility and auth**: PASS. The plan preserves existing response envelopes, authentication requirements, platform/school authorization separation, and additive behavior only.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/003-backend-school-admin/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-school-admin.md
└── checklists/
    └── requirements.md
```

### Source Code (target repositories)

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
│   ├── Services/
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
└── specs/003-backend-school-admin/
```

**Structure Decision**: Keep this as a backend-only implementation slice with specification artifacts in `schoolmaster-specs`. Backend code should follow the existing Laravel API structure established by the foundation slice. Frontend work is explicitly deferred until the backend behavior is contract-compliant and ready to consume.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- The next backend slice is the remaining P1 school administration foundation, not P2 teacher workflows.
- The implementation boundary is the eleven published OpenAPI operation IDs listed in Technical Context.
- Any maintain/update/deactivate/delete/detail or identity lifecycle behavior must wait for contract expansion.
- Tenant context is resolved once at the request boundary and propagated to services, policies, resources, and tests.
- Role and permission scope integrity is a first-class acceptance concern.
- Academic period validation depends on same-school parent academic year, date containment, and sequence uniqueness.
- Guardian association must be atomic and reject missing, inactive, or cross-tenant student profile references.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, and state transitions.
- [contracts/backend-school-admin.md](./contracts/backend-school-admin.md): consumed operation IDs, tenant and authorization rules, response envelope dependencies, blocked contract gaps, and verification expectations.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts do not add public behavior beyond the current OpenAPI surface and explicitly block undocumented operations.
- **Repository impact**: PASS. The docs identify `schoolmaster-specs` as source of truth and `schoolmaster-backend` as the only implementation target for this slice.
- **Backend architecture**: PASS. Data and contract notes preserve Service Layer, Form Request, Policy, Resource, DTO, Repository/query object, and UUID expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. Every affected school-owned entity is scoped to `school_id`, and cross-tenant association rules are documented.
- **API compatibility and auth**: PASS. The plan keeps additive behavior and requires contract revision before any missing operation is exposed.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
