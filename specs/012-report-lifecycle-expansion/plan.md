# Implementation Plan: Report Lifecycle Expansion

**Branch**: `012-report-lifecycle-expansion` | **Date**: 2026-06-05 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/specs/012-report-lifecycle-expansion/spec.md`

**Note**: This plan defines the backend implementation and contract boundary for report lifecycle expansion. It does not decompose tasks and does not authorize behavior outside the specification and OpenAPI contract.

## Summary

Implement a backend-only report lifecycle expansion slice for school-scoped reporting. The backend must expand OpenAPI before exposing new `/api/v1` behavior for report retry, cancellation, soft delete/restore, read-only report catalog discovery, custom report definitions, custom report requests, XLSX output where catalog-approved, per-format output availability, deterministic lifecycle conflict handling, tenant-safe reason-code validation, per-school custom definition name uniqueness, active-definition edit boundaries, and tenant-safe report audit events.

The design preserves the current asynchronous `ReportRun` foundation, 90-day generated-output retention, expired-output download behavior, and no download-time regeneration. It keeps `School` as the v1 tenant root with `school_id`, separates generation status from soft deletion and per-output availability, keeps output availability free of a deleted state, keeps custom reports limited to launch-scope domains, allows active custom definitions to receive metadata-only edits while requiring deactivation for structural edits, rejects over-complex definitions, and excludes frontend implementation, platform-wide reporting/support access, output delete/restore, retention overrides, permanent purge, legal hold, anonymization, billing, messaging, and undocumented APIs.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, school-admin, student-reporting, classroom-roster, teacher-workflow lifecycle, and guardian self-service slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for report lifecycle, definition, catalog, request, and output inputs; existing tenant context resolver, authentication/account lifecycle behavior, role/permission authority, `ReportRun`, report output, academic period, user, student profile, grade, attendance, academic-structure, school-activity, private storage, asynchronous worker, and audit-event patterns; repositories/query objects only for complex report catalog, definition validation, report-run lifecycle concurrency, output availability, definition-name uniqueness checks, active-definition structural edit checks, and audit-safe reads  
**Storage**: MySQL for schools, users, roles, academic periods, report runs, report outputs, report definitions, definition snapshots, report catalog metadata or configuration, lifecycle state, soft deletes, retry linkage, predefined tenant-safe reason codes, correlation IDs, and audit events; private tenant-scoped file/object storage for generated report outputs  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests should run with `docker exec schoolmaster-backend-app-1 php artisan test` after implementation  
**Target Platform**: API-only Laravel backend consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned report data access for 100% of protected operations; report list and catalog operations use documented pagination or bounded response shapes; custom report definitions are capped at 25 selected fields, 10 filters, 2 grouping levels, and 3 sort fields; lifecycle transitions complete atomically without partial state; stale worker completions do not publish outputs after cancellation  
**Constraints**: OpenAPI expansion required before backend exposure; no undocumented endpoints, fields, filters, sort values, formats, lifecycle states, status mutations, free-text lifecycle reasons, output lifecycle actions, audit payloads, or authorization exceptions; `school_id` remains the tenant column; public identifiers are UUIDs; generation status, soft delete, and per-output availability are separate; report-run delete does not delete outputs; output availability excludes `deleted`; output deletion/restoration and retention override are out of scope; custom report definition names are unique per school among non-deleted definitions; active definitions allow name/description edits only and require deactivation before domain, field, filter, grouping, sorting, or format changes; custom report definitions are limited to attendance, grades, academic structure, and school activity; platform/support override is out of scope  
**Scale/Scope**: Proposed boundary covers report lifecycle operations, report list expansion, read-only report catalog discovery, custom report definition lifecycle, custom report request snapshots, XLSX as the only new format, per-format output availability, lifecycle concurrency, tenant-safe reason-code validation, active-definition edit validation, tenant-safe audit events, and regression coverage. No frontend implementation or platform-wide reporting is included.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The plan requires contract expansion before backend implementation exposes report lifecycle, catalog, definition, custom request, XLSX, output availability, reason-code, active-definition edit, or expanded response behavior.
- **Repository impact**: PASS. `schoolmaster-specs` leads specification and OpenAPI, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized controllers, Form Requests, Policies, API Resources, Services, DTOs for multi-field lifecycle/definition/request/catalog inputs, UUID public identifiers, and repositories/query objects only where report catalog, definition validation, lifecycle concurrency, output availability, definition-name uniqueness, active-definition edit validation, and audit-safe reads become complex.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. All affected records are school-owned by `school_id`; cross-tenant and platform access remain deny-by-default; support/platform override is outside this slice; report definitions enforce per-school non-deleted name uniqueness.
- **API compatibility and auth**: PASS. The plan is additive, preserves existing report request/list/download behavior, requires active permitted school context, separates report lifecycle permission from report definition permission, and documents response expectations before backend exposure.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges. No frontend implementation is planned, so Vitest is deferred until frontend consumption begins.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/specs/012-report-lifecycle-expansion/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-report-lifecycle-expansion.md
└── checklists/
    └── requirements.md
```

### Source Code (current workspace and target repositories)

```text
# Current backend workspace (Laravel API)
./
├── app/
│   ├── DTOs/
│   │   └── Reports/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── Reports/
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

# Separate frontend repository (Vue 3)
schoolmaster-frontend/
└── No implementation changes in this feature.

# Nested specification workspace available from this repository
specs/
├── api/openapi.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
└── specs/012-report-lifecycle-expansion/
```

**Structure Decision**: Keep this as a backend-only report lifecycle expansion slice with specification artifacts available locally under `specs/`. Backend implementation should follow the existing Laravel API structure used by prior slices, with domain rules in services, request validation in Form Requests, authorization in Policies, API Resources for response shaping, DTOs for lifecycle/definition/catalog/request inputs, and repositories/query objects only for complex report catalog resolution, definition validation, lifecycle concurrency, per-format output availability, report-run list filtering, definition-name uniqueness, active-definition structural edit validation, and audit-safe reads. Frontend work is explicitly deferred until contract-compliant backend behavior exists.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- OpenAPI must be expanded before backend implementation exposes report lifecycle, catalog, custom definition, XLSX, reason-code, active-definition edit, or expanded output behavior.
- Report generation status, report-run soft delete, and per-output availability are separate state concerns.
- Report output availability uses `pending`, `available`, `failed`, `expired`, and `unsupported`; it does not include `deleted`.
- Report lifecycle concurrency uses first-valid-transition-wins semantics, with later conflicts returning or recording documented conflict outcomes.
- Lifecycle reason inputs use predefined tenant-safe reason codes only; free-text lifecycle reasons are not accepted.
- Custom report definitions are limited to launch-scope domains and must be built from a read-only report catalog.
- Custom report definitions use `draft`, `active`, `inactive`, and `deleted`; new definitions start `draft`, and restore returns `inactive`.
- Custom report definition names are unique per school among non-deleted definitions; restore conflicts if another non-deleted definition already uses the name.
- Active custom report definitions allow metadata-only edits to name and description; structural changes to domain, fields, filters, grouping, sorting, or formats require deactivation first.
- Custom report definitions are capped at 25 fields, 10 filters, 2 grouping levels, and 3 sort fields.
- XLSX is the only additional v1 output format, and only where the report catalog and OpenAPI declare support.
- Report outputs do not have explicit delete or restore actions in this slice; report-run soft delete only affects list visibility.
- Report audit events must include actor, action, outcome, target, school, correlation ID, and tenant-safe reason code.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, state transitions, tenant boundaries, retry lineage, catalog constraints, report definition lifecycle and name uniqueness, active-definition edit boundary, output availability, lifecycle reason codes, concurrency, and audit requirements.
- [contracts/backend-report-lifecycle-expansion.md](./contracts/backend-report-lifecycle-expansion.md): proposed operation boundary, required OpenAPI expansion, tenant and authorization rules, response expectations, blocked behavior, and verification requirements.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

Implementation must update `specs/api/openapi.yaml` before routes, controllers, requests, services, or tests are added in the backend repository.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. `schoolmaster-specs` remains the source of truth, `schoolmaster-backend` is the only implementation target for this slice, and frontend work remains deferred.
- **Backend architecture**: PASS. Design notes preserve Service Layer, Form Request, Policy, Resource, DTO, UUID, audit-event, tenant-scope, report catalog, custom definition validation, definition-name uniqueness, active-definition edit validation, lifecycle conflict, retry lineage, output availability, and private output expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. All affected records remain school-owned by `school_id`; platform access does not bypass school permissions; private report contents, filter payloads, storage paths, lifecycle reasons, and audit details stay tenant-safe.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, preserves existing report request/list/download behavior, and separates school report lifecycle, report definition, and platform/support access boundaries.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
