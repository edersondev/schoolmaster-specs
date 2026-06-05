# Implementation Plan: Backend Guardian Self-Service

**Branch**: `011-guardian-self-service` | **Date**: 2026-06-04 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/011-guardian-self-service/spec.md`

**Note**: This plan defines the backend implementation and contract boundary for guardian self-service. It does not decompose tasks and does not authorize behavior outside the specification and OpenAPI contract.

## Summary

Implement a backend-only guardian self-service slice that lets authenticated guardian users view limited, school-approved information for students they are explicitly linked to in the active school context. The backend must expand OpenAPI before exposing new `/api/v1` guardian behavior for linked student listing, limited student profile/enrollment summary, academic summary-only view, and limited contact view, and must include the minimal same-school school-admin guardian-user-link provisioning lifecycle needed to activate or deactivate guardian self-service access. The design preserves tenant-by-column isolation with `school_id`, requires an explicit school-administrator-created guardian-user link plus an active guardian-student association, requires an explicit same-school academic period for academic summaries, limits academic visibility to current grade summary, attendance totals/status, and learning-set progress/status where documented, limits contact visibility to the authenticated guardian's own contact fields, relationship labels, and the student's primary school-approved contact details, returns the same not-found envelope for missing, unassociated, inactive, or cross-tenant target students, records tenant-safe audit events, and keeps guardian access separate from school administration, student self-view, teacher workflows, reports, platform support, messaging, billing, frontend implementation, self-claiming, consent capture, custody/legal workflows, and undocumented APIs.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by the backend foundation, school-admin, student-reporting, student-enrollment, account-lifecycle, classroom-roster, and teacher-workflow lifecycle slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for guardian self-service query inputs, existing tenant context resolver, existing authentication/account lifecycle behavior, existing role/permission authority, existing Guardian, StudentProfile, GuardianStudentAssociation, AcademicPeriod, GradeRecord, AttendanceRecord, LearningSet/LearningSetAssignment, and audit-event patterns; repositories/query objects only for complex tenant-scoped guardian visibility, summary aggregation, contact visibility, non-enumerating denial, and audit-safe reads  
**Storage**: MySQL for schools, users, guardian records, guardian-user links, guardian-student associations, student profiles, academic periods, grade and attendance records, learning-set summary sources, contact fields, lifecycle state, soft deletes, and audit events  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests should run with `docker exec schoolmaster-backend-app-1 php artisan test` after implementation  
**Target Platform**: API-only Laravel backend consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned data access for 100% of protected operations; guardian student list and target-specific lookups use bounded same-school filters; guardian academic summaries are derived only for one permitted student and one explicit same-school academic period per request; target-specific denied requests avoid protected-record enumeration without extra disclosure  
**Constraints**: OpenAPI expansion required before backend exposure; no undocumented endpoints, fields, filters, includes, sort values, status semantics, denial details, or authorization exceptions; `school_id` remains the tenant column; public identifiers are UUIDs; guardian self-service is read-only; target-specific missing, unassociated, inactive, or cross-tenant students return the same not-found envelope; self-claiming, automatic contact matching, invite-completion linking, guardian profile updates, report access, teacher content download, messaging, billing, frontend implementation, permanent purge, legal hold, and anonymization are out of scope  
**Scale/Scope**: Proposed boundary covers guardian student listing, guardian-visible student profile/enrollment summary, academic summary-only retrieval by explicit same-school academic period, and limited contact retrieval. The feature reuses existing school-owned guardian/student/academic records and introduces or formalizes guardian-user link proof for authenticated guardian self-service access.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The plan requires contract expansion before backend implementation exposes guardian self-service routes, schemas, response envelopes, field visibility, and denial semantics.
- **Repository impact**: PASS. `schoolmaster-specs` leads specification and OpenAPI, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized controllers, Form Requests, Policies, API Resources, Services, DTOs for multi-field query inputs, UUID public identifiers, and repositories/query objects only where guardian visibility, summary aggregation, contact visibility, non-enumerating denial, or audit-safe reads become complex.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. All affected records are school-owned by `school_id`; cross-tenant and platform access remain deny-by-default; support/platform override is outside this slice.
- **API compatibility and auth**: PASS. The plan is additive, requires active permitted school context, explicit guardian-user link proof, active guardian-student association, read-only guardian permission, and non-enumerating target-specific not-found behavior.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges. No frontend implementation is planned, so Vitest is deferred until frontend consumption begins.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/011-guardian-self-service/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-guardian-self-service.md
└── checklists/
    └── requirements.md
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── GuardianSelfService/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── GuardianSelfService/
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
└── specs/011-guardian-self-service/
```

**Structure Decision**: Keep this as a backend-only guardian self-service slice with specification artifacts in `schoolmaster-specs`. Backend implementation should follow the existing Laravel API structure used by prior slices, with domain rules in services, request validation in Form Requests, authorization in Policies, API Resources for response shaping, DTOs for guardian student, academic-summary, and contact-view query inputs, and repositories/query objects only for complex tenant-scoped guardian visibility, summary aggregation, contact visibility, non-enumerating denial, and audit-safe reads. Frontend work is explicitly deferred until contract-compliant backend behavior exists.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- OpenAPI must be expanded before backend implementation exposes guardian self-service routes.
- Guardian self-service proof requires an explicit school-administrator-created guardian-user link plus an active same-school guardian-student association, and this slice includes the minimal school-admin guardian-user-link lifecycle needed to provision that proof.
- Guardian self-service is read-only and does not include guardian self-claiming, automatic contact matching, invite-completion linking, profile updates, association requests, consent capture, custody/legal workflows, messaging, reports, or teacher content downloads.
- Guardian academic visibility is summary-only and requires an explicit same-school academic period.
- Guardian contact visibility is limited to the authenticated guardian's own contact fields, school-approved relationship labels, and the student's primary school-approved contact details.
- Target-specific requests for missing, unassociated, inactive, or cross-tenant students return the same not-found envelope.
- Tenant context is resolved before guardian, student, academic, contact, or audit data access.
- Guardian self-service access grants, denied attempts, blocked cross-tenant attempts, and visibility-sensitive reads are audited with tenant-safe metadata only.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, state transitions, tenant boundaries, guardian-user link proof, guardian-student association rules, academic summaries, contact visibility, denial semantics, and audit requirements.
- [contracts/backend-guardian-self-service.md](./contracts/backend-guardian-self-service.md): proposed operation boundary, required OpenAPI expansion, tenant and authorization rules, response expectations, blocked behavior, and verification requirements.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. `schoolmaster-specs` remains the source of truth, `schoolmaster-backend` is the only implementation target for this slice, and frontend work remains deferred.
- **Backend architecture**: PASS. Design notes preserve Service Layer, Form Request, Policy, Resource, DTO, UUID, audit-event, tenant-scope, non-enumerating denial, contact-visibility, academic-summary, and guardian-link proof expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. All affected records remain school-owned by `school_id`; platform access does not bypass school permissions; private correction, contact, report, file, and audit data stay hidden from guardian responses.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, separates guardian self-service from school administration, student self-view, teacher workflows, reports, and platform support, and defines target-specific not-found behavior for protected student targets.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
