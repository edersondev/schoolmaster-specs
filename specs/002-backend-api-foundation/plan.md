# Implementation Plan: Backend API Foundation

**Branch**: `002-backend-api-foundation` | **Date**: 2026-05-14 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/specs/002-backend-api-foundation/spec.md`

**Note**: This plan defines backend setup and implementation boundaries only. It does not decompose implementation tasks and does not authorize product endpoints outside the published OpenAPI contracts.

## Summary

Prepare the SchoolMaster backend repository for contract-first Laravel API work by defining API-only boundaries, `/specs` consumption rules, environment expectations, base response semantics, tenant-context enforcement, authentication security rules, audit expectations, and the first backend product slice for authentication and school management.

The clarified authentication foundation now requires bearer tokens that expire after 8 hours, logout/access revocation for inactive users or schools, failed-login lockout after 5 failed attempts per email or IP within 15 minutes, and audit events for authentication and school lifecycle changes. These product-visible security behaviors are reflected in `/specs` and OpenAPI before authentication coding exposes them.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel 13.x as currently declared in `composer.json`  
**Primary Dependencies**: Laravel framework, Laravel-native authentication mechanisms, Eloquent ORM, Form Requests, Policies, API Resources, PHPUnit, Redocly CLI for contract validation  
**Storage**: MySQL for transactional tenant data; storage for audit records and token metadata must be durable enough to support token expiry, logout revocation, lockout checks, inactive-status rejection, and school lifecycle review  
**Testing**: PHPUnit via `php artisan test`; contract validation via Redocly against `specs/api/openapi.yaml` and `specs/specs/001-schoolmaster-platform/contracts/openapi.yaml`  
**Target Platform**: API backend service consumed by the SchoolMaster Vue SPA and governed by OpenAPI  
**Project Type**: Single Laravel API backend repository consuming a specification submodule  
**Performance Goals**: Backend readiness checks complete locally without private project knowledge; tenant context is resolved before school-owned data access for all protected school-scoped operations; authentication lockout decisions are deterministic for email and IP attempts within a 15-minute window  
**Constraints**: API-only product behavior, `/api/v1` route prefix for product APIs, no product Blade views, MySQL primary datastore, strict `school_id` v1 tenancy, no undocumented endpoints or fields, no frontend implementation, no package additions unless explicitly approved, no storage of plaintext credentials or bearer token values in audit events  
**Scale/Scope**: Foundation for v1 platform implementation; first product slice is limited to `login`, `getCurrentUser`, `logout`, `listSchools`, `createSchool`, `getSchool`, and `updateSchool`, with contract updates in place for auth token expiry, logout revocation, lockout response, token rejection, and audit semantics

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

The local constitution is now aligned with backend `AGENTS.md` and `/specs`;
the effective gates for this repository are:

- **Source-of-truth gate**: PASS. The plan uses `/specs/AGENTS.md`, active feature specs, aggregate OpenAPI, docs, and ADRs as authoritative inputs.
- **Contract-first gate**: PASS. The first product slice maps only to approved OpenAPI operations; token expiry, logout revocation, inactive-context revocation, failed-login lockout, and audit semantics are reflected in `/specs` and OpenAPI before authentication implementation merges.
- **API-only gate**: PASS. Product behavior remains under `/api/v1`; frontend code and product Blade views are excluded.
- **Tenancy gate**: PASS. The plan follows ADR 004: tenant-by-column with `School` as v1 tenant root and `school_id` as the concrete school-owned column.
- **Authorization gate**: PASS. Platform-scope school provisioning remains separate from school-scoped module authorization; system administrators do not receive implicit school-scope bypass.
- **Security gate**: PASS. Clarified auth security rules are specific, testable, and documented in OpenAPI before coding.
- **Testing gate**: PASS. The plan requires feature tests for API behavior, authorization, tenant isolation, validation, inactive status, response shape, token expiry/revocation, login lockout, and audit creation, plus unit tests for isolated service behavior.
- **Documentation gate**: PASS. Backend setup guidance and quickstart validation are planned before coding begins.

## Project Structure

### Documentation (this feature)

```text
specs/specs/002-backend-api-foundation/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-readiness.md
└── checklists/
    └── requirements.md
```

### Source Code (repository root)

```text
app/
├── DTOs/
├── Http/
│   ├── Controllers/Api/V1/
│   ├── Requests/
│   └── Resources/
├── Models/
├── Policies/
└── Services/

bootstrap/
└── app.php

config/
├── auth.php
└── database.php

database/
├── factories/
├── migrations/
└── seeders/

routes/
└── api.php

tests/
├── Feature/
└── Unit/

AGENTS.md
README.md
.env.example
phpunit.xml
```

**Structure Decision**: Keep a single Laravel API backend repository. Add only backend foundation structure needed by the approved architecture: API controllers under `App\Http\Controllers\Api\V1`, Form Requests, API Resources, Policies, Services, models, migrations, seeders, and tests. Do not add frontend source trees or product Blade views.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- `/specs` remains the source of truth for business rules, contracts, and architecture decisions.
- API-only readiness is a prerequisite before product endpoint implementation.
- V1 tenancy uses `school_id`, not a literal `tenant_id`, for school-owned records.
- Authentication uses Laravel-native mechanisms aligned to OpenAPI and must support 8-hour bearer-token expiry, logout revocation, inactive user/school revocation, and 5-failure-per-15-minute email/IP lockout.
- Audit coverage for the first product slice includes login success, login failure, logout, token rejection, and school create/update/status changes.
- First backend implementation boundary is authentication, logout revocation, and school management only.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): readiness, response, auth session, login attempt control, audit event, school, tenant, role, and permission entities.
- [contracts/backend-readiness.md](./contracts/backend-readiness.md): consumed OpenAPI operations, response envelope dependency, authentication security contract deltas, audit boundary, and non-goals.
- [quickstart.md](./quickstart.md): validation path for repository readiness before implementation.

## Post-Design Constitution Check

- **Source-of-truth gate**: PASS. Design artifacts reference `/specs` and do not introduce independent product rules.
- **Contract-first gate**: PASS. OpenAPI and the active feature contract document the logout operation, token expiry, token rejection, lockout, and audit-relevant response semantics before authentication implementation.
- **API-only gate**: PASS. Quickstart checks route exposure and product Blade exclusion.
- **Tenancy gate**: PASS. Data model and contracts consistently use `school_id` for v1 school-owned records.
- **Authorization gate**: PASS. Platform and school scopes remain separate in data model and contracts.
- **Security gate**: PASS. Clarified token, lockout, inactive-context rejection, and audit rules are present and testable.
- **Testing gate**: PASS. Quickstart defines backend tests and contract validation expectations.
- **Documentation gate**: PASS. The plan produces repository setup documentation expectations without coding product behavior.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution or `/specs` violations are introduced by this plan |
