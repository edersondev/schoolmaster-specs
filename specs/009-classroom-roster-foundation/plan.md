# Implementation Plan: Backend Classroom Roster Foundation

**Branch**: `009-classroom-roster-foundation` | **Date**: 2026-05-29 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/009-classroom-roster-foundation/spec.md`

**Note**: This plan defines the backend implementation boundary after `008-account-lifecycle-workflows`. It does not decompose tasks and does not authorize product behavior outside the specification and OpenAPI contract.

## Summary

Implement the backend classroom roster foundation for school-owned `ClassSection/Roster` records, roster memberships, and teacher assignments. The backend must expand the OpenAPI contract before implementation, then expose only approved `/api/v1` operations that preserve tenant-by-column isolation with `school_id`, school-administrator-only write authority, teacher read-only visibility into their own active assignments including documented detail retrieval, active academic-period scoping, stable lifecycle states, explicit effective start dates for membership and assignment creation, today-or-past effective dates evaluated in the resolved school's configured timezone with application default fallback, required reasons for lifecycle-ending actions, all-or-nothing batch membership changes capped at 100 requested changes, list pagination with default page size 25 and maximum page size 100, list filters limited to `academicPeriodId` and `status`, no include expansion or sort behavior unless OpenAPI later documents it, transactional conflict handling for duplicate and dependency-sensitive writes, audit events with required tenant-safe field minimums, read-only legacy direct-assignment compatibility, and contract-aligned response envelopes. The design keeps `ClassSection/Roster` as the single v1 teaching structure with structured course/classroom/section/group metadata blocks containing optional `code` and `name` only, blocks separate top-level or internal Course/Classroom/Section/Group resources in v1, requires new `ClassSection/Roster` records to start as `active`, rejects inactive roster reactivation in v1, anchors student membership eligibility to active same-school enrollment covering the membership effective start date, anchors teacher assignment eligibility to an active same-school teacher-compatible role on the assignment effective start date, rejects membership end and teacher deactivation dates before their effective start dates, and defers teacher workflow correction, guardian self-service, report lifecycle expansion, frontend delivery, platform support access, billing, messaging, and undocumented APIs.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, school-admin, teacher-workflow, student-reporting, student-enrollment, administration-lifecycle, and account-lifecycle slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for coordinated roster membership and teacher-assignment inputs, existing school role/permission authority for teacher-compatible role eligibility, repositories or query objects only for complex tenant-scoped class-section, membership, assignment, duplicate, effective-date, timezone, dependency, date-order, role/enrollment eligibility, concurrency-conflict, audit-field, and legacy compatibility reads  
**Storage**: MySQL for schools, school timezone/configuration, academic years, academic periods, student profiles, enrollment history, users, roles, role assignment history, `ClassSection/Roster` records with structured metadata blocks, roster memberships, teacher assignments, lifecycle reasons, lifecycle history, legacy direct assignment references, audit events, and tenant-scoped actor context  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests must run with `docker exec schoolmaster-backend-app-1 php artisan test`  
**Target Platform**: API-only backend service consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned roster data access for 100% of protected operations; list operations use paginated envelopes with default page size 25 and maximum page size 100; all-or-nothing batch roster membership changes up to 100 requested changes, duplicate checks, lifecycle transitions, dependency checks, role/enrollment effective-date checks, transactional conflict handling, and audit writes complete atomically; list operations remain bounded by documented `academicPeriodId` and `status` filters only  
**Constraints**: OpenAPI expansion required before backend exposure, no undocumented endpoints or fields, no separate top-level or internal Course/Classroom/Section/Group resources in v1, no metadata block fields beyond optional `code` and `name`, no inactive `ClassSection/Roster` creation, no inactive `ClassSection/Roster` reactivation, no membership end or teacher deactivation date before the target record's effective start date, no frontend implementation, no teacher workflow correction behavior, no guardian self-service, no report lifecycle expansion, no platform-wide support access, no billing or messaging behavior, no permanent purge, UUID public identifiers, `school_id` as the v1 tenant column, contract-aligned response envelopes only  
**Scale/Scope**: Proposed boundary covers `ClassSection/Roster` list/create/detail/update/status behavior where approved, roster membership list and all-or-nothing batch add/end behavior capped at 100 requested changes, teacher assignment list/detail/create/deactivate behavior where approved, and read-only legacy direct-assignment compatibility. `ClassSection/Roster.code` is required and unique per school and academic period; names may repeat. Course/classroom/section/group data are structured metadata blocks on `ClassSection/Roster` with optional `code` and `name` only. Lifecycle states are `ClassSection/Roster: active/inactive`, `RosterMembership: active/ended`, and `TeacherAssignment: active/inactive`. New `ClassSection/Roster` records start as `active`, `inactive` is reached only through inactivation, and inactive rosters cannot be reactivated in v1. Roster inactivation is rejected while active memberships or active teacher assignments exist. Effective start dates are required for membership and assignment creation, and all effective dates must be today-or-past and within the selected academic period. A compatible academic period is same-school, active, not closed, has usable start and end dates, and contains the relevant lifecycle effective date; membership and assignment compatibility also depends on enrollment or role coverage at the effective start date. Today is evaluated using the resolved school's configured timezone, falling back to the application default timezone only when the school has no configured timezone. Student membership eligibility requires active same-school enrollment covering the membership effective start date. Teacher assignment eligibility requires an active same-school teacher-compatible role from the existing school role/permission authority on the assignment effective start date. Membership end and teacher deactivation effective dates must be on or after their effective start dates. Reasons are required for roster inactivation, roster membership ending, and teacher assignment deactivation.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The current contract does not publish roster foundation operations, so this plan requires contract expansion before backend implementation.
- **Repository impact**: PASS. `schoolmaster-specs` leads the plan and contract boundary, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized Laravel controllers, Form Requests, Policies, API Resources, Services, UUID identifiers, DTOs for batch membership and teacher-assignment inputs, and repositories/query objects only for complex tenant-scoped reads, duplicate checks, effective-date checks, timezone checks, date-order checks, dependency checks, transactional conflict handling, audit-field shaping, role/enrollment eligibility validation, and legacy direct-assignment compatibility.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. `ClassSection/Roster`, roster membership, teacher assignment, and lifecycle history are governed by `school_id`; course/classroom/section/group data are structured fields on the same school-owned record; cross-tenant access remains deny-by-default; platform users do not receive implicit school-owned roster access.
- **API compatibility and auth**: PASS. The plan is additive, requires active permitted school context and school-administrator permissions for management writes, allows teachers to list and retrieve only their own active assignments where OpenAPI documents that visibility, preserves teacher/student/report behavior unless separately specified, and documents response expectations for duplicate, conflict, oversized batch, all-or-nothing batch, unsupported filter/include/sort/page-size, inactive creation, reactivation denial, date-order, and concurrency behavior.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges. No frontend implementation is planned, so Vitest is deferred until frontend consumption begins.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/009-classroom-roster-foundation/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-classroom-roster-foundation.md
└── checklists/
    └── requirements.md
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── ClassroomRoster/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── ClassroomRoster/
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
└── specs/009-classroom-roster-foundation/
```

**Structure Decision**: Keep this as a backend-only implementation slice with specification artifacts in `schoolmaster-specs`. Backend code should follow the existing Laravel API structure used by previous slices, with roster rules in services, request validation in Form Requests, authorization in Policies, API Resources for response shaping, DTOs for all-or-nothing membership batches and teacher assignment inputs, and repositories/query objects only where tenant-scoped lookup, duplicate detection, effective-date validation, timezone resolution, date-order validation, dependency validation, transactional conflict handling, audit field shaping, enrollment/role eligibility validation, and legacy direct-assignment compatibility become complex. Frontend work is explicitly deferred until the backend behavior is contract-compliant and ready to consume.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- Classroom roster foundation is the next backend slice after account lifecycle, not teacher correction, guardian self-service, reporting expansion, platform support access, frontend delivery, billing, or messaging.
- OpenAPI must be expanded before backend implementation because `ClassSection/Roster`, roster membership, and teacher assignment operations are not currently published.
- V1 uses one primary `ClassSection/Roster` resource with structured course, classroom, section, and group metadata blocks containing optional `code` and `name` only, not separate top-level lifecycle resources or separate internal tables.
- `ClassSection/Roster.code` is required and unique per school and academic period; display names may repeat.
- New `ClassSection/Roster` records start as `active`, inactive creation is rejected, inactive records cannot be reactivated in v1, and administrators create a new roster when a replacement is needed.
- Management writes are school-administrator-only under an active permitted school context.
- Teachers may list and retrieve only their own active teacher assignments.
- Roster membership batch changes are all-or-nothing and capped at 100 requested changes per request.
- Roster membership and teacher assignment creation require explicit effective start dates.
- Effective-date today-or-past validation uses the resolved school's configured timezone, with application default timezone fallback only when the school has no configured timezone.
- Student membership eligibility requires active same-school enrollment covering the membership effective start date.
- Teacher assignment eligibility requires an active same-school teacher-compatible role on the assignment effective start date.
- Membership end and teacher assignment deactivation effective dates must be on or after their effective start dates.
- Concurrent conflicting roster, membership, teacher-assignment, lifecycle, and dependency-sensitive writes are resolved transactionally so one request succeeds and conflicting requests return the documented conflict response without partial state.
- Audit events include actor user ID, school ID, target type and target ID, action, outcome, lifecycle reason when present, and tenant-safe summary metadata only.
- List operations support pagination with default page size 25 and maximum page size 100 plus `academicPeriodId` and `status` filters only; include expansion and sort behavior are not approved unless OpenAPI later documents them.
- Overlapping active roster memberships for the same student, roster, and academic period are rejected.
- Duplicate active teacher assignments for the same teacher, `ClassSection/Roster`, and academic period are rejected.
- Roster inactivation is rejected while active memberships or active teacher assignments exist.
- Effective dates must fall within the selected academic period and be today-or-past only.
- Lifecycle reasons are required for roster inactivation, roster membership ending, and teacher assignment deactivation.
- Lifecycle states are intentionally narrow: `ClassSection/Roster active/inactive`, `RosterMembership active/ended`, `TeacherAssignment active/inactive`.
- Existing direct learning-set student assignments remain readable as legacy records; new roster-aware writes must use roster memberships once those workflows are approved.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, state transitions, tenant boundaries, metadata block shape, timezone rules, date-order rules, enrollment/role eligibility rules, list constraints, audit field requirements, transactional conflict rules, and legacy compatibility rules.
- [contracts/backend-classroom-roster-foundation.md](./contracts/backend-classroom-roster-foundation.md): proposed operation boundary, contract expansion requirements, tenant and authorization rules, response dependencies, blocked contract gaps, and verification expectations.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. The docs identify `schoolmaster-specs` as source of truth and `schoolmaster-backend` as the only implementation target for this slice.
- **Backend architecture**: PASS. Data and contract notes preserve Service Layer, Form Request, Policy, Resource, DTO, Repository/query object, UUID, audit-event, lifecycle-history, all-or-nothing batch, duplicate-check, effective-date, timezone, date-order, reason-required, dependency-check, transactional-conflict, role/enrollment eligibility, and tenant-scope expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. School-scoped roster foundation records require `school_id` context, structured metadata remains on the primary tenant-owned `ClassSection/Roster` record, platform access does not bypass school-owned roster permissions, teachers are limited to own active assignment reads, and source teacher/student/report behavior remains isolated unless a future spec changes it.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, preserves platform/school authorization separation, and defines school-administrator-only management writes plus teacher own-active-assignment list/detail visibility at the contract boundary.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
