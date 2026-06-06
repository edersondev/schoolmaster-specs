# Implementation Plan: Platform-Wide Reporting and Support Access

**Branch**: `013-platform-support-access` | **Date**: 2026-06-06 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/013-platform-support-access/spec.md`

**Note**: This plan defines the backend implementation and contract boundary for platform-wide reporting and support access. It does not decompose tasks and does not authorize behavior outside the specification and OpenAPI contract.

## Summary

Implement a backend-only platform oversight and support-access slice for SchoolMaster. The backend must expand OpenAPI before exposing new `/api/v1` platform behavior for minimized school operational summaries, cross-school reporting health summaries, explicit read-only support drill-down, school-scoped target-school opt-in, internal platform approval, 24-hour support approval and opt-in expiration, support access audit review, tenant-safe denial behavior, and small-count suppression for protected aggregate counts below 5.

The design preserves `School` as the v1 tenant root with `school_id` on school-owned records, keeps platform and school authorization separate, rejects implicit platform bypass of existing school-scoped endpoints, excludes frontend implementation, excludes generated report downloads and emergency access, excludes support writes, excludes unrestricted impersonation or raw record search, and requires tenant-safe audit metadata for allowed, denied, approval, revocation, expiration, validation, and conflict outcomes.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, school-admin, account lifecycle, classroom-roster, teacher-workflow lifecycle, guardian self-service, and report lifecycle slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for platform overview/support access/audit inputs; existing tenant context resolver, authentication/account lifecycle behavior, role/permission authority, `School`, `User`, report lifecycle state/output availability/custom definition behavior, audit-event patterns, and response envelope components; repositories/query objects only for complex cross-school summary aggregation, small-count suppression, support access decision lookup, and audit-safe reads  
**Storage**: MySQL for schools, users, roles, permissions, report runs, report outputs, report definitions, platform support access decisions, target-school opt-in records with 24-hour expiration, internal platform approval records with 24-hour expiration, reason codes, correlation IDs, audit events, and diagnostic summary materialization where needed; no generated report output storage changes in this slice  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests should run with `docker exec schoolmaster-backend-app-1 php artisan test` after implementation  
**Target Platform**: API-only Laravel backend consumed by future clients through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Platform school summaries and cross-school reporting overview use documented pagination, filters, sorting, and bounded response fields; tenant and permission checks complete before summary or diagnostic data access for 100% of protected operations; support drill-down decisions are evaluated atomically before diagnostics are returned; protected counts below 5 are suppressed in 100% of platform summary responses  
**Constraints**: OpenAPI expansion required before backend exposure; no undocumented endpoints, fields, filters, sort values, diagnostics, audit payloads, approval states, reason codes, or authorization exceptions; `school_id` remains the school-owned tenant column; public identifiers are UUIDs; platform users do not gain school-scoped endpoint access by role alone; support access is read-only; support drill-down requires both target-school opt-in and internal platform approval; support approvals and target-school opt-ins expire after 24 hours; generated report downloads, emergency access, support writes, unrestricted impersonation, raw report output, private file metadata, and unrestricted record search are out of scope  
**Scale/Scope**: Proposed boundary covers platform school summary list/detail where minimized, cross-school reporting overview, support access decision lifecycle, read-only support diagnostic view, support access audit summary, small-count suppression, 24-hour approval expiration, stale/revoked/mismatched approval denial, tenant-safe audit events, OpenAPI expansion, and backend regression coverage. No frontend implementation is included.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The plan requires contract expansion before backend implementation exposes platform summary, cross-school reporting overview, support drill-down, support access decision, audit summary, small-count suppression, approval, revocation, expiration, denial, or conflict behavior.
- **Repository impact**: PASS. `schoolmaster-specs` leads specification and OpenAPI work, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized controllers, Form Requests, Policies, API Resources, Services, DTOs for multi-field platform/support inputs, UUID public identifiers, and repositories/query objects only where cross-school aggregation, small-count suppression, support access decision lookup, and audit-safe reads become complex.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. School-owned records remain scoped by `school_id`; platform access is deny-by-default except for explicitly contracted platform/support operations; protected counts below 5 are suppressed; support diagnostics are target-school-bound and read-only.
- **API compatibility and auth**: PASS. The plan is additive, preserves existing school-scoped endpoints, requires explicit platform-scoped permissions, and documents tenant-safe validation, authorization, not-found, and conflict responses before backend exposure.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges. No frontend implementation is planned, so Vitest is deferred until frontend consumption begins.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/013-platform-support-access/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-platform-support-access.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Created later by /speckit-tasks
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── PlatformSupport/
│   ├── Http/
│   │   ├── Controllers/Api/V1/Platform/
│   │   ├── Requests/Api/V1/Platform/
│   │   └── Resources/Platform/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── PlatformSupport/
│   └── Repositories/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/PlatformSupport/
    └── Unit/PlatformSupport/

# Frontend repository (Vue 3)
schoolmaster-frontend/
└── No implementation changes in this feature.

# Specification and contract repository
schoolmaster-specs/
├── api/openapi.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
└── specs/013-platform-support-access/
```

**Structure Decision**: Keep this as a backend-only platform support access slice with specification artifacts under `specs/013-platform-support-access/`. Backend implementation should follow existing Laravel API structure used by prior slices, with domain rules in services, request validation in Form Requests, authorization in Policies, API Resources for response shaping, DTOs for platform overview/support access/audit inputs, and repositories/query objects only for complex cross-school summary aggregation, small-count suppression, support access decision lookup, and audit-safe reads. Frontend work is explicitly deferred until contract-compliant backend behavior exists.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- OpenAPI must be expanded before backend implementation exposes any platform/support operation.
- Platform-wide views expose minimized summaries only and do not reuse existing school-scoped endpoints as an implicit bypass.
- Protected aggregate counts below 5 are suppressed.
- Support drill-down is read-only, target-school-bound, and requires both school-scoped target-school opt-in and internal platform approval.
- Support access decisions and target-school opt-ins expire after 24 hours and are denied when stale, revoked, mismatched, expired, or concurrently changed.
- Generated report downloads, raw report outputs, private file metadata, emergency access, support writes, unrestricted impersonation, and unrestricted record search are excluded from v1.
- Platform support audit entries use minimized tenant-safe metadata with actor, action, outcome, target school where applicable, correlation ID, and reason code.
- Backend implementation uses Laravel Services, Form Requests, Policies, API Resources, DTOs, and repositories/query objects only for complex aggregation or audit-safe reads.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): platform summary, reporting overview, support access decision, target-school opt-in, internal platform approval, support diagnostic view, audit event, state transitions, validation rules, tenant boundaries, and redaction rules.
- [contracts/backend-platform-support-access.md](./contracts/backend-platform-support-access.md): proposed operation boundary, required OpenAPI expansion, target-school opt-in operations, tenant and authorization rules, response expectations, blocked behavior, and verification requirements.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

Implementation must update `api/openapi.yaml` before routes, controllers, requests, services, or tests are added in the backend repository.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. `schoolmaster-specs` remains the source of truth, `schoolmaster-backend` is the only implementation target for this slice, and frontend work remains deferred.
- **Backend architecture**: PASS. Design notes preserve Service Layer, Form Request, Policy, Resource, DTO, UUID, audit-event, tenant-scope, explicit platform permission, support approval, small-count suppression, and audit-safe read expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. School-owned records remain scoped by `school_id`; platform/support access is explicit, read-only where school diagnostics are involved, target-school-bound, and denied when missing opt-in or internal approval; protected counts below 5 are suppressed.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, preserves existing school-scoped endpoint behavior, and separates platform/support permissions from school permissions.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
