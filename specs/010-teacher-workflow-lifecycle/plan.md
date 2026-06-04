# Implementation Plan: Backend Teacher Workflow Lifecycle and Corrections

**Branch**: `010-teacher-workflow-lifecycle` | **Date**: 2026-06-01 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/specs/010-teacher-workflow-lifecycle/spec.md`

**Note**: This plan defines the backend implementation boundary after `009-classroom-roster-foundation`. It does not decompose tasks and does not authorize product behavior outside the specification and OpenAPI contract.

## Summary

Implement the backend teacher workflow lifecycle slice for school-owned teacher content, questionnaires, learning sets, grades, and attendance, with correction workflows limited to grades and attendance. The backend must expand OpenAPI before exposing new `/api/v1` behavior for detail retrieval, updates, activation/deactivation, soft delete, restore, teacher content download, grade and attendance imports, and grade/attendance correction workflows. The design preserves tenant-by-column isolation with `school_id`, school-administrator override inside the same school, creating/owning teacher authority for owned records, roster-aware learning-set assignment writes, read-only compatibility for legacy direct selected-student assignments, school-administrator-only closed-period grade and attendance corrections with 10-500 character free-text reasons, school-administrator-only create-only JSON payload grade and attendance imports capped at 500 rows, all-or-nothing import validation, shared lifecycle states `active`, `inactive`, and `deleted`, restore-to-inactive behavior, active-only current student visibility with inactive/deleted records hidden except documented history, clean-scan-gated content downloads, rejection of content or questionnaire edits that would change historical student-facing meaning after use, tenant-safe audit events for successful and denied downloads, and explicit separation from guardian self-service, report lifecycle expansion, platform-wide support access, frontend implementation, billing, messaging, advanced assessment types, permanent purge, and undocumented APIs.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, school-admin, teacher-workflow, student-reporting, student-enrollment, administration-lifecycle, account-lifecycle, and classroom-roster slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for correction/import/lifecycle inputs, existing tenant context resolver, existing role/permission authority, existing ClassSection/Roster, RosterMembership, TeacherAssignment, AcademicPeriod, StudentProfile, teacher workflow, private storage, scan-status, and audit-event patterns; repositories/query objects only for complex tenant-scoped reads, ownership checks, roster eligibility, lifecycle dependency checks, correction history, import validation, conflict detection, and audit-safe summaries  
**Storage**: MySQL for schools, users, roles, academic periods, student profiles, class sections/rosters, roster memberships, teacher assignments, teacher content, questionnaires, learning sets, learning-set entries and assignments, grades, attendance, corrections, import runs, lifecycle state, soft deletes, private file metadata, and audit events; private tenant-scoped object/file storage for teacher content files  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests should run with `docker exec schoolmaster-backend-app-1 php artisan test` after implementation  
**Target Platform**: API-only Laravel backend consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned data access for 100% of protected operations; detail and lifecycle operations perform bounded same-school lookups; JSON payload grade and attendance imports validate up to 500 rows before commit; import processing is all-or-nothing; list behavior remains constrained by OpenAPI-approved filters, pagination, and response envelopes  
**Constraints**: OpenAPI expansion required before backend exposure; no undocumented endpoints, fields, filters, status values, lifecycle actions, correction states, import result shapes, or download behaviors; `school_id` remains the tenant column; public identifiers are UUIDs; teacher content downloads require clean scan status; lifecycle states are `active`, `inactive`, and `deleted`; restore returns to `inactive`; permanent purge is out of scope; frontend implementation is out of scope  
**Scale/Scope**: Proposed boundary covers detail, update, activation/deactivation, soft delete, restore, and download behavior for teacher content, questionnaires, and learning sets, plus import and correction behavior for grades and attendance. Grade and attendance imports are create-only, JSON-payload-only, school-administrator-only, and capped at 500 rows. Closed-period grade and attendance corrections are school-administrator-only. New learning-set assignment writes use roster membership context; legacy direct selected-student assignments remain readable only.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The plan requires contract expansion before backend implementation exposes lifecycle, correction, download, import, or detail behavior.
- **Repository impact**: PASS. `schoolmaster-specs` leads specification and OpenAPI, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized controllers, Form Requests, Policies, API Resources, Services, DTOs for multi-field lifecycle/correction/import inputs, UUID identifiers, and repositories/query objects only where tenant-scoped lookup, ownership, dependency, import, correction, and conflict rules become complex.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. All affected records are school-owned by `school_id`; cross-tenant and platform access remain deny-by-default; support/platform override is outside this slice.
- **API compatibility and auth**: PASS. The plan is additive, requires active permitted school context, preserves teacher ownership versus school-admin authority, separates student visibility from private correction/audit data, and documents response expectations before backend exposure.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges. No frontend implementation is planned, so Vitest is deferred until frontend consumption begins.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/specs/010-teacher-workflow-lifecycle/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-teacher-workflow-lifecycle.md
└── checklists/
    └── requirements.md
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── TeacherWorkflow/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── TeacherWorkflow/
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
└── specs/specs/010-teacher-workflow-lifecycle/
```

**Structure Decision**: Keep this as a backend-only lifecycle and correction slice with specification artifacts in `schoolmaster-specs`. Backend implementation should follow the existing Laravel API structure used by prior slices, with domain rules in services, request validation in Form Requests, authorization in Policies, API Resources for response shaping, DTOs for lifecycle/correction/import inputs, and repositories/query objects only for complex tenant-scoped lookup, ownership, roster, dependency, correction-history, import-validation, conflict, and audit-summary reads. Frontend work is explicitly deferred until contract-compliant backend behavior exists.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- OpenAPI must be expanded before backend implementation exposes lifecycle, correction, download, import, or detail behavior.
- Teacher management is owner-scoped for creating/owning teachers, with school-administrator same-school override.
- Used content and questionnaires reject historical-meaning edits in v1 rather than creating versions.
- New learning-set assignment writes use roster memberships; legacy direct selected-student assignments remain read-only.
- Shared lifecycle states are `active`, `inactive`, and `deleted`; restore returns records to `inactive`.
- Closed-period grade and attendance corrections are school-administrator-only and require a 10-500 character free-text reason.
- Grade and attendance imports are create-only, JSON-payload-only, school-administrator-only, capped at 500 rows, and all-or-nothing.
- Teacher content download requires clean scan status, same-school authority, active allowed lifecycle state, and tenant-safe private delivery.
- Successful and denied teacher content download attempts are audited with tenant-safe metadata only.
- Permanent purge, legal hold, anonymization, report lifecycle expansion, guardian self-service, platform support access, frontend implementation, messaging, billing, and advanced assessment types remain outside this slice.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, state transitions, tenant boundaries, ownership rules, correction records, import runs, student visibility rules, download audit requirements, and roster-aware compatibility.
- [contracts/backend-teacher-workflow-lifecycle.md](./contracts/backend-teacher-workflow-lifecycle.md): proposed operation boundary, required OpenAPI expansion, tenant and authorization rules, response expectations, blocked behavior, and verification requirements.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. `schoolmaster-specs` remains the source of truth, `schoolmaster-backend` is the only implementation target for this slice, and frontend work remains deferred.
- **Backend architecture**: PASS. Design notes preserve Service Layer, Form Request, Policy, Resource, DTO, UUID, audit-event, lifecycle-history, correction-history, import-run, all-or-nothing import, ownership, tenant-scope, and conflict-handling expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. All affected records remain school-owned by `school_id`; platform access does not bypass school permissions; private correction and audit data stay tenant-safe.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, preserves existing list/create operations, preserves legacy direct assignment reads, and defines teacher-owner, school-admin, student visibility, and import authority boundaries.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
