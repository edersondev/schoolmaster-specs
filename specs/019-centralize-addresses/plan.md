# Implementation Plan: Centralize Addresses

**Branch**: `019-centralize-addresses` | **Date**: 2026-06-26 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/019-centralize-addresses/spec.md`

## Summary

Replace the legacy school `address_summary` field with a reusable structured
address model. The first approved owner type is `School`; future owner types
must be specified and contracted separately. OpenAPI must change before
backend or frontend implementation: school create/detail/update/list/session
school schemas expose `address`, reject `address_summary`, validate required
structured fields, preserve digit-only `number` and `zip_code`, and support
explicit `address: null` removal on school update.

Backend implementation in `schoolmaster-backend` will add tenant-safe
address ownership, development schema reset for the legacy summary column,
request validation, authorization through the owning school boundary, API
resources, services, and tests. Frontend implementation in `schoolmaster-frontend` will
replace school address-summary form/display behavior after the contract and
backend are complete.

## Technical Context

**Language/Version**: PHP 8.3 with Laravel 13 backend conventions; JavaScript
Vue 3 frontend consumption after backend contract compliance.
**Primary Dependencies**: Laravel routing, Eloquent ORM, migrations, Form
Requests, Policies, API Resources, Services, DTOs for coordinated address
input, existing school administration controllers/services/resources,
OpenAPI/Redocly contract validation, Vue 3 administration forms/services after
backend delivery.
**Storage**: MySQL. Add centralized `addresses` storage with UUID public
identifier, structured address fields, `addressable_type`/`addressable_id`
ownership metadata, direct `school_id` tenant column for school-owned rows,
timestamps, soft delete support, and a MySQL-safe active-owner uniqueness
strategy; drop `schools.address_summary` during development schema reset and
contract publication.
**Testing**: PHPUnit feature/unit tests in `schoolmaster-backend`; Redocly
OpenAPI validation in `schoolmaster-specs`; Vitest/service/form tests in
`schoolmaster-frontend` once frontend consumption is implemented.
**Target Platform**: API-only Laravel backend consumed by the SchoolMaster Vue
SPA through published `/api/v1` REST contracts.
**Project Type**: Cross-repository contract-first feature: specification and
OpenAPI update, Laravel API backend change, and frontend follow-up.
**Performance Goals**: School list/detail/session responses include at most one
current address per school without unbounded joins; validation rejects invalid
address payloads before persistence; tenant and authorization checks complete
before address lookup or mutation for 100% of school-owned address operations.
**Constraints**: OpenAPI update required before backend exposure; no standalone
address endpoints; no non-school address owner behavior in this slice; one
current address per school; `address_summary` rejected immediately after
structured-address publication; `number` and `zip_code` are digit-only strings
to preserve leading zeros; omitted `address` on update means no address change;
explicit `address: null` on update removes the school address; omitted country
remains absent/null; address rows must carry `school_id` for tenant isolation
and use a generated active marker or equivalent MySQL-safe constraint to prevent
multiple active addresses for the same owner while permitting soft-deleted
history.
**Scale/Scope**: School address create/read/update/remove, development reset
of legacy summaries, reusable ownership foundation for future owner types,
contract updates for school schemas and authenticated session embedding,
backend regression tests, and frontend school form/display follow-up.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **Source-of-truth alignment**: PASS. Read `specs/AGENTS.md`, the active
  feature spec, current school OpenAPI schemas, multi-tenant documentation,
  backend guidelines, and ADR 004 before planning.
- **OpenAPI impact**: PASS. School schemas and authenticated session school
  embedding must be updated before backend or frontend implementation consumes
  structured address behavior.
- **Repository boundary**: PASS. `schoolmaster-specs` owns this plan and
  contract changes; `schoolmaster-backend` implements API/storage behavior;
  `schoolmaster-frontend` updates school forms/displays after contract and
  backend readiness.
- **Tenant and authorization isolation**: PASS. School is the first approved
  address owner; school address access is authorized through school management
  permissions and must not leak cross-school existence.
- **Architecture fit**: PASS. Backend work uses Laravel Form Requests,
  Policies, API Resources, Services, DTOs, migrations, factories, and tests;
  repositories/query objects are not required unless existing school queries
  become complex.
- **Verification**: PASS. Requires Redocly/OpenAPI validation, PHPUnit feature
  and unit tests for validation, development schema reset, authorization, tenant isolation,
  response shape, soft delete/removal, and frontend tests when implemented.
- **Constitution deviations**: PASS. No deviations are required.

## Project Structure

### Documentation (this feature)

```text
specs/019-centralize-addresses/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── centralize-addresses-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (target repositories)

```text
# Specification and contract repository
schoolmaster-specs/
├── api/
│   ├── openapi.yaml
│   └── components/schemas/schools/
├── specs/019-centralize-addresses/
└── AGENTS.md

# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── Addresses/
│   ├── Http/
│   │   ├── Requests/Schools/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   └── Services/
│       ├── Addresses/
│       └── SchoolService.php
├── database/
│   ├── factories/
│   └── migrations/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/Schools/
    └── Unit/Addresses/

# Frontend repository (Vue 3 SPA)
schoolmaster-frontend/
├── src/
│   ├── components/admin-system/schools/
│   ├── contracts/admin-system/schools.js
│   ├── pages/admin-system/schools/
│   └── services/admin-system/schools.js
└── tests/unit/admin-system/administration/
```

**Structure Decision**: Treat centralized addresses as a contract-first
cross-repository feature. The specs repository updates OpenAPI and design
artifacts first. The backend implements address ownership through existing
school routes rather than standalone address routes. The frontend changes only
the school administration address form/display after the contract and backend
behavior are available.

## Phase 0: Research

Research output is captured in [research.md](research.md). No unresolved
technical clarifications remain.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/centralize-addresses-contract.md](contracts/centralize-addresses-contract.md)
- [quickstart.md](quickstart.md)

Implementation must update `api/openapi.yaml` and referenced schema components
before backend routes, request validation, resources, services, or frontend
consumption rely on structured address behavior.

## Post-Design Constitution Check

- **Source-of-truth alignment**: PASS. Design artifacts preserve the clarified
  specification and identify current contract fields requiring replacement.
- **OpenAPI impact**: PASS. Contract artifact defines the exact schema areas to
  replace and blocks backend/frontend use before OpenAPI validation passes.
- **Repository boundary**: PASS. Specs, backend, and frontend responsibilities
  remain separated; no backend-only product behavior is approved.
- **Tenant and authorization isolation**: PASS. Address access follows the
  owning school authorization path and denies cross-school leakage.
- **Architecture fit**: PASS. Backend design keeps controllers thin and routes
  coordinated address input through Requests, DTOs, Services, Policies, and
  Resources.
- **Verification**: PASS. Quickstart requires OpenAPI validation and focused
  backend tests for validation, replacement, explicit removal, development reset,
  authorization, tenant isolation, and response shape; frontend tests are
  deferred to frontend implementation.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
