# Implementation Plan: Backend Student and Reporting Foundation

**Branch**: `005-backend-student-reporting` | **Date**: 2026-05-20 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/005-backend-student-reporting/spec.md`

**Note**: This plan defines the next backend implementation boundary after `004-backend-teacher-workflows`. It does not decompose tasks and does not authorize product behavior outside the published OpenAPI contract.

## Summary

Implement the P3 backend foundation for student self-view and school-scoped reporting. The backend must consume the existing published `/api/v1` operations for student learning timelines, student teacher-content downloads, student grades, student attendance, report listing, report requests, and report downloads. The design preserves tenant-by-column isolation with `School` as the tenant root and `school_id` for school-owned records, separates platform administration from school-scoped student/report access, enforces student ownership and private file authorization, supports asynchronous `ReportRun` generation, keeps report outputs available for 90 days, and requires explicit regeneration through a new report run after output expiry.

The plan intentionally blocks frontend implementation, teacher workflow writes, report designer/custom report definitions, platform-wide reporting, report deletion, report retry, automatic expired-output regeneration during download, classroom/course/section/roster workflows, correction workflows, and any undocumented API behavior until `/specs` and OpenAPI document those operations.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by the backend foundation, school-admin slice, and teacher-workflow slice  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for report requests where useful, private filesystem storage, queued job/service boundary for report generation, PHPUnit, Redocly CLI for contract validation  
**Storage**: MySQL for student profile links, learning-set assignments, grades, attendance, report runs, report filters, report statuses, and output expiry metadata; private tenant-scoped file storage for authorized teacher content and generated report outputs  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`  
**Target Platform**: API-only backend service consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned data access for 100% of protected operations in the slice; student self-view queries are bounded by documented pagination and academic-period filters; report requests return a `ReportRun` without waiting for generated files; report downloads use generated private files when available and unexpired  
**Constraints**: No undocumented endpoints or fields, no product Blade views, no frontend implementation, no teacher writes, no custom report designer, no platform-wide reports, no class/course/section/roster model, no automatic download-time regeneration, UUID public identifiers, `school_id` as the v1 tenant column, contract-aligned response envelopes and binary responses only  
**Scale/Scope**: Seven operation IDs: `listStudentLearningSets`, `downloadStudentTeacherContent`, `listStudentGrades`, `listStudentAttendance`, `listReports`, `requestReport`, and `downloadReport`

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The slice maps to already published aggregate and platform feature contract operations. Any operation beyond the listed student/reporting surface requires a contract update first.
- **Repository impact**: PASS. `schoolmaster-specs` leads the plan and contract boundary, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized Laravel controllers, Form Requests, Policies, API Resources, Services, UUID identifiers, DTOs for report-request filters where useful, and repositories/query objects only for complex tenant-scoped reads.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. Every affected school-owned entity is scoped to `school_id`, and cross-tenant reference rules are documented for learning sets, assignments, content, grades, attendance, academic periods, student profiles, users, report runs, and files.
- **API compatibility and auth**: PASS. The plan preserves existing response envelopes, binary download behavior, authentication requirements, platform/school authorization separation, and additive behavior only.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/005-backend-student-reporting/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-student-reporting.md
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
└── specs/005-backend-student-reporting/
```

**Structure Decision**: Keep this as a backend-only implementation slice with specification artifacts in `schoolmaster-specs`. Backend code should follow the existing Laravel API structure established by the foundation, school-admin, and teacher-workflow slices. Frontend work is explicitly deferred until the backend behavior is contract-compliant and ready to consume.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- The next backend slice is P3 student self-view and school reporting, not frontend delivery or new teacher write workflows.
- The implementation boundary is the seven published OpenAPI operation IDs listed in Technical Context.
- Student self-view is authenticated-user-owned and requires an active same-school `StudentProfile`.
- Student content downloads require assignment through a same-school learning set and `clean` scan status.
- Reports remain school-scoped, asynchronous, and limited to launch-scope report types.
- Generated report outputs are private tenant-scoped PDF/CSV files available for 90 days after generation.
- Expired report outputs are not regenerated during download; regeneration requires a new `ReportRun` with the same filters.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, and state transitions.
- [contracts/backend-student-reporting.md](./contracts/backend-student-reporting.md): consumed operation IDs, tenant and authorization rules, response and binary-output dependencies, blocked contract gaps, and verification expectations.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts do not add public behavior beyond the current OpenAPI surface and explicitly block undocumented operations.
- **Repository impact**: PASS. The docs identify `schoolmaster-specs` as source of truth and `schoolmaster-backend` as the only implementation target for this slice.
- **Backend architecture**: PASS. Data and contract notes preserve Service Layer, Form Request, Policy, Resource, DTO, Repository/query object, UUID, private storage, queue/job boundary, retention, and expiry expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. Every affected school-owned entity is scoped to `school_id`, and cross-tenant reference rules are documented.
- **API compatibility and auth**: PASS. The plan keeps additive behavior and requires contract revision before any missing operation is exposed.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
