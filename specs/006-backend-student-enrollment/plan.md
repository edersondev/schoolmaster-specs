# Implementation Plan: Backend Student Profile and Enrollment Management

**Branch**: `006-backend-student-enrollment` | **Date**: 2026-05-21 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/006-backend-student-enrollment/spec.md`

**Note**: This plan defines the backend implementation boundary after `005-backend-student-reporting`. It does not decompose tasks and does not authorize product behavior outside the specification and OpenAPI contract.

## Summary

Implement the backend foundation for school-scoped student profile and enrollment management. The backend must expand the contract before implementation, then expose only approved `/api/v1` operations for listing, creating, viewing, status-changing, and transfer-recording `StudentProfile` records. The design preserves tenant-by-column isolation with `School` as the tenant root and `school_id` for school-owned records, separates platform administration from school-scoped student administration, keeps guardian associations same-school and atomic, records enrollment lifecycle history, prevents cross-tenant movement of historical academic data, and blocks classroom/course/section/roster workflows, guardian self-service, correction workflows, bulk import, report changes, frontend implementation, and undocumented APIs.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, school-admin, teacher-workflow, and student/reporting slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for profile creation/status/transfer inputs, repositories or query objects only for complex tenant-scoped listing and lifecycle history reads, PHPUnit, Redocly CLI for contract validation  
**Storage**: MySQL for `StudentProfile`, student lifecycle status fields, guardian associations, enrollment history, transfer metadata, and audit-relevant actor/effective-date fields  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`  
**Target Platform**: API-only backend service consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before student-owned data access for 100% of protected operations; student profile lists remain bounded by documented pagination and filters; profile creation/status/transfer writes complete atomically without partial profile, guardian association, or history changes  
**Constraints**: OpenAPI expansion required before backend exposure, no undocumented endpoints or fields, no product Blade views, no frontend implementation, no classroom/course/section/roster model, no guardian self-service, no academic-record correction workflow, no report changes, no bulk import, UUID public identifiers, `school_id` as the v1 tenant column, contract-aligned response envelopes only  
**Scale/Scope**: Proposed operation boundary: list student profiles, create student profile, get student profile, update student profile status, and record student transfer; exact operation IDs and payloads must be finalized in OpenAPI before implementation

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The current contract does not publish student profile lifecycle operations, so this plan requires contract expansion before backend implementation.
- **Repository impact**: PASS. `schoolmaster-specs` leads the plan and contract boundary, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized Laravel controllers, Form Requests, Policies, API Resources, Services, UUID identifiers, DTOs for coordinated lifecycle inputs, and repositories/query objects only for complex tenant-scoped reads.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. Student profiles, enrollment history, guardian associations, lifecycle status, and transfer metadata are school-owned records scoped by `school_id`, with explicit cross-school transfer constraints.
- **API compatibility and auth**: PASS. The plan is additive, requires explicit school-scoped student administration permission, and preserves platform/school authorization separation.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/006-backend-student-enrollment/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-student-enrollment.md
└── checklists/
    └── requirements.md
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── StudentProfiles/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── StudentProfiles/
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
└── specs/006-backend-student-enrollment/
```

**Structure Decision**: Keep this as a backend-only implementation slice with specification artifacts in `schoolmaster-specs`. Backend code should follow the existing Laravel API structure used by previous slices, with profile lifecycle rules in services and controllers limited to orchestration. Frontend work is explicitly deferred until the backend behavior is contract-compliant and ready to consume.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- The next backend slice is student profile and enrollment management, not classroom/roster modeling or frontend delivery.
- OpenAPI must be expanded before backend implementation because the current contracts do not publish profile lifecycle operations.
- The initial operation boundary should cover list, create, detail, status update, and transfer recording only.
- Guardian associations during profile creation must be same-school and atomic.
- Enrollment history should be append-only for lifecycle events and retained for audit, reporting, and support review.
- Student transfer preserves source-school history and must not copy academic records, private content, guardian links, or report outputs across tenants.
- Platform administrators do not receive implicit school-scoped enrollment authority without an explicit permitted school context.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, and lifecycle transitions.
- [contracts/backend-student-enrollment.md](./contracts/backend-student-enrollment.md): proposed operation boundary, contract expansion requirements, tenant and authorization rules, response dependencies, blocked contract gaps, and verification expectations.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. The docs identify `schoolmaster-specs` as source of truth and `schoolmaster-backend` as the only implementation target for this slice.
- **Backend architecture**: PASS. Data and contract notes preserve Service Layer, Form Request, Policy, Resource, DTO, Repository/query object, UUID, lifecycle-history, and tenant-scope expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. Every affected school-owned entity is scoped to `school_id`, and transfer rules explicitly prevent cross-tenant data exposure.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, and preserves platform/school authorization separation.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
