# Implementation Plan: Backend Teacher Workflow Foundation

**Branch**: `004-backend-teacher-workflows` | **Date**: 2026-05-18 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/004-backend-teacher-workflows/spec.md`

**Note**: This plan defines the next backend implementation boundary after `003-backend-school-admin`. It does not decompose tasks and does not authorize product behavior outside the published OpenAPI contract.

## Summary

Implement the P2 teacher workflow backend foundation for teacher content, questionnaires, learning sets, grades, and attendance. The backend must consume the existing published `/api/v1` list/create contract operations, enforce tenant-by-column isolation with `School` as the tenant root and `school_id` for school-owned records, preserve platform versus school authorization boundaries, validate uploads and academic-period/student references, gate content availability on malware scan status, and verify response envelopes, validation, inactive-status handling, and tenant failure behavior.

The plan intentionally blocks student self-service, reporting, classroom/course/section/roster workflows, public content-folder CRUD, detail, update, deactivate, delete, restore, download, bulk import, and correction behavior until `/specs` and OpenAPI document those operations.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by the backend foundation and school-admin slice  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for coordinated create workflows, private filesystem storage, queued job/service boundary for malware scan status, PHPUnit, Redocly CLI for contract validation  
**Storage**: MySQL for teacher content metadata, optional folder references, questionnaires, questions, learning sets, entries, assignments, grade records, attendance records, and status/soft-delete history where recoverability is required; private tenant-scoped file storage for uploaded instructional files  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`  
**Target Platform**: API-only backend service consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned data access for 100% of protected operations in the slice; uploaded files are rejected before persistence when they violate type, size, or authorization rules; paginated list operations remain bounded by the documented `page` and `per_page` contract  
**Constraints**: No undocumented endpoints or fields, no product Blade views, no frontend implementation, no student self-service, no reports, no class/course/section/roster model, no public folder CRUD, no direct content download route, no automatic scan bypass, UUID public identifiers, `school_id` as the v1 tenant column, contract-aligned response envelopes only  
**Scale/Scope**: Ten operation IDs: `listTeacherContent`, `createTeacherContent`, `listQuestionnaires`, `createQuestionnaire`, `listLearningSets`, `createLearningSet`, `listGrades`, `createGrade`, `listAttendance`, and `createAttendance`

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The slice maps to already published aggregate and platform feature contract operations. Any operation beyond the listed list/create surface requires a contract update first.
- **Repository impact**: PASS. `schoolmaster-specs` leads the plan and contract boundary, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized Laravel controllers, Form Requests, Policies, API Resources, Services, UUID identifiers, DTOs for multi-field create workflows where useful, and repositories/query objects only for complex tenant-scoped access.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. Every affected school-owned entity is scoped to `school_id`, and cross-tenant reference rules are documented for content, questionnaires, academic periods, teachers, and student profiles.
- **API compatibility and auth**: PASS. The plan preserves existing response envelopes, authentication requirements, platform/school authorization separation, and additive behavior only.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/004-backend-teacher-workflows/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-teacher-workflows.md
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
│   ├── Jobs/
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
└── specs/004-backend-teacher-workflows/
```

**Structure Decision**: Keep this as a backend-only implementation slice with specification artifacts in `schoolmaster-specs`. Backend code should follow the existing Laravel API structure established by the foundation and school-admin slices. Frontend work is explicitly deferred until the backend behavior is contract-compliant and ready to consume.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- The next backend slice is P2 teacher workflow foundation, not student self-service or reports.
- The implementation boundary is the ten published OpenAPI operation IDs listed in Technical Context.
- Public folder management, downloads, updates, corrections, student views, reports, and classroom/course models must wait for contract expansion.
- Tenant context is resolved once at the request boundary and propagated to services, policies, storage, resources, and tests.
- Upload validation includes declared type, detected type, file size, rejected executable/archive categories, tenant ownership, authorization, private storage, and scan-status gating.
- Learning sets are assigned directly to selected active same-school `StudentProfile` records for a same-school active academic period.
- Grade and attendance writes require same-school active students, same-school active academic periods, permitted teacher recorder, and documented values only.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, and state transitions.
- [contracts/backend-teacher-workflows.md](./contracts/backend-teacher-workflows.md): consumed operation IDs, tenant and authorization rules, response envelope dependencies, blocked contract gaps, and verification expectations.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts do not add public behavior beyond the current OpenAPI surface and explicitly block undocumented operations.
- **Repository impact**: PASS. The docs identify `schoolmaster-specs` as source of truth and `schoolmaster-backend` as the only implementation target for this slice.
- **Backend architecture**: PASS. Data and contract notes preserve Service Layer, Form Request, Policy, Resource, DTO, Repository/query object, UUID, private storage, and scan-status workflow expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. Every affected school-owned entity is scoped to `school_id`, and cross-tenant reference rules are documented.
- **API compatibility and auth**: PASS. The plan keeps additive behavior and requires contract revision before any missing operation is exposed.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
