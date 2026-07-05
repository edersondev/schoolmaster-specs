# Implementation Plan: School Fields Tabs

**Branch**: `029-school-fields-tabs` | **Date**: 2026-07-05 | **Spec**: `specs/029-school-fields-tabs/spec.md`  
**Input**: Feature specification from `specs/029-school-fields-tabs/spec.md`

## Summary

Refactor school create/edit into a contract-first tabbed profile with Basic,
Address, Institutional, and Branding groups. The feature replaces `cnpj` with
`document`, changes school status to numeric `1`/`0`, requires Address on
create/edit, exposes Institutional lookup/reference options, enforces unique
school identity fields, and adds logo/color branding behavior.

OpenAPI leads the change, followed by Laravel backend validation/resources and
Vue 3 frontend form/service updates. Existing school-owned data does not need
migration/backfill and may be reset only during pre-rollout setup or seed
cleanup, never by normal runtime create/edit behavior.

## Technical Context

**Language/Version**: PHP 8.x/Laravel 10+ backend; Vue 3 with TypeScript-capable Composition API frontend; OpenAPI YAML contract.  
**Primary Dependencies**: Laravel Form Requests, Policies, API Resources, Services, DTOs where useful, MySQL/Eloquent, file storage; Vue Router, Pinia, Axios services, Tailwind CSS, Element Plus `ElColorPicker`; Redocly OpenAPI validation.  
**Storage**: MySQL school profile/address/institutional/branding fields; seeded Institutional lookup options; backend-managed logo file storage with sanitized filenames and persisted logo path/reference; route-local frontend form state and selected file object before submit.  
**Testing**: Redocly/OpenAPI lint; PHPUnit feature/unit tests for validation, resources, persistence, authorization, tenant boundaries, lookup endpoints, and logo behavior; Vitest/Vue Test Utils for services, composables, tabs, mapping, validation, lookups, upload, stale responses, and accessibility-focused behavior.  
**Target Platform**: Laravel JSON/multipart API under `/api/v1`; Vue 3 SPA administration UI.  
**Project Type**: Multi-repository web application feature spanning specs, backend API, and frontend SPA.  
**Performance Goals**: School create/edit routes render a stable form or denied/loading state within 2 seconds after session context resolves with mocked service latency capped at 1.5 seconds; Institutional lookup selectors show full option lists or recoverable unavailable state within 2 seconds; create/update/logo submissions show success or validation feedback within 2 seconds after service resolution; stale school, lookup, or route responses never overwrite current form state.  
**Constraints**: OpenAPI changes before implementation; no undocumented school fields, endpoints, lookup values, file rules, status values, error shapes, tenant behavior, or authorization exceptions; this breaking `/api/v1` contract change requires coordinated OpenAPI, backend, and frontend merge/release gates or an explicit API versioning plan before merge; no migration/backfill requirement for existing school-owned data and data reset allowed only during pre-rollout setup or seed cleanup; `inep_code`, `document`, and `email` unique across all schools including inactive and soft-deleted records; `email` uniqueness is case-insensitive; `document` replaces `cnpj`, is a 14-digit valid CNPJ, is read-only on edit, and legacy `cnpj` payloads are rejected immediately; status is numeric `1`/`0`; Address is required on create/edit; Institutional lookups use seeded numeric IDs from `docs/school_new_fields.md` and return full option lists; branding colors are non-alpha `#RRGGBB` via Element Plus `ElColorPicker`; logo upload permits PNG/JPEG/WebP up to 2 MB; logo replacement deletes old file only after new file is stored; no new optimistic-locking requirement; no user-facing logo delete action, school lifecycle behavior, tenant selection behavior, role permission behavior, academic behavior, or unrelated school administration refactor.  
**Scale/Scope**: One expanded School contract, one required Address group, six Institutional lookup/reference sets, one Branding/logo upload flow, existing school create/edit frontend pages, school service/form contracts, backend request/resource/service coverage, OpenAPI aggregate validation, and focused tests across specs/backend/frontend repositories.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. School create/update/read schemas,
  validation errors, logo upload, and Institutional lookup/reference changes
  begin in the contract.
- PASS: Repository impacts are separated. `schoolmaster-specs` owns spec,
  OpenAPI, and generated design artifacts; `schoolmaster-backend` owns Laravel
  validation/resources/services/policies/storage; `schoolmaster-frontend` owns
  Vue tabbed form, services, and tests.
- PASS: Backend design uses Form Requests for validation, Policies for
  authorization, Services for school profile/logo business logic, API
  Resources for output, UUID public identifiers, and DTOs for the expanded
  multi-field payload. Repository abstraction is not planned because data
  access is simple Eloquent relationship/reference lookup work.
- PASS: Frontend design uses Vue 3 Composition API, Vue Router, Pinia only for
  existing session/shell context, Axios services, Tailwind CSS, feature-local
  components, route-local composables, and service-isolated API access.
- PASS: MySQL remains authoritative; School remains the tenant root; school
  ownership and `school_id` tenant boundaries remain explicit; no new
  cross-tenant access path is introduced; soft-delete lifecycle behavior is
  unchanged.
- PASS: API compatibility, authentication/authorization impact, success
  envelopes, validation errors, logo upload errors, lookup unavailable states,
  optimistic-locking non-scope, and coordinated breaking-change rollout gates
  are documented.
- PASS: Verification spans OpenAPI lint, PHPUnit backend coverage, and Vitest
  frontend coverage for create/edit, uniqueness, required address,
  Institutional lookups, logo upload, validation, authorization, tenant safety,
  and tab state.
- PASS: No constitution deviations.

## Project Structure

### Documentation (this feature)

```text
specs/029-school-fields-tabs/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── school-fields-tabs-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
# schoolmaster-backend (Laravel API)
app/
├── DTOs/School/
├── Http/
│   ├── Controllers/Api/V1/
│   ├── Requests/Api/V1/
│   └── Resources/
├── Models/
├── Policies/
└── Services/School/
database/
├── migrations/
└── seeders/
routes/api.php
tests/
├── Feature/School/
└── Unit/School/

# schoolmaster-frontend (Vue 3 SPA)
src/
├── modules/schools/
│   ├── components/
│   ├── composables/
│   ├── routes/
│   ├── services/
│   └── types/
├── router/
└── stores/
tests/
└── unit/schools/

# schoolmaster-specs
api/
├── openapi.yaml
├── paths/
└── components/schemas/
specs/029-school-fields-tabs/
```

**Structure Decision**: Keep the specifications repository as the source of
truth for OpenAPI and design artifacts. Backend and frontend implementations
must use the same `029-school-fields-tabs` feature identifier and consume only
the documented contract fields.

## Phase 0: Research

Research is complete in `specs/029-school-fields-tabs/research.md`.

Key decisions:

- Treat the school profile change as a breaking contract-first update.
- Coordinate OpenAPI, backend, and frontend release gates for the breaking
  `/api/v1` school contract; if that is not possible, stop and add an explicit
  API versioning plan before merge.
- Use `document` as the canonical field and reject legacy `cnpj`.
- Enforce `inep_code`, `document`, and `email` uniqueness across all schools,
  including inactive and soft-deleted records; compare email case-insensitively.
- Make Address required on create and edit.
- Do not migrate/backfill existing school-owned data.
- Use seeded Institutional lookup/reference endpoints with full option lists.
- Use JSON by default and multipart only when `logo_file` is present.
- Limit logo files to PNG/JPEG/WebP up to 2 MB.
- Delete old logos only after successful replacement.
- Use existing school update behavior with no new optimistic-locking contract.
- Use backend DTO + Service and route-local frontend tabbed form composable.
- Verify through OpenAPI, PHPUnit, and Vitest.

## Phase 1: Design And Contracts

Design artifacts are complete:

- `specs/029-school-fields-tabs/data-model.md`
- `specs/029-school-fields-tabs/contracts/school-fields-tabs-contract.md`
- `specs/029-school-fields-tabs/quickstart.md`

Required OpenAPI updates:

- School create/update/read schemas replace `cnpj` with `document`.
- Breaking school contract changes require coordinated OpenAPI, backend, and
  frontend merge/release gates before implementation can merge.
- School create/update reject legacy `cnpj` payloads.
- `inep_code`, `document`, and `email` uniqueness validation is documented,
  including inactive/soft-deleted records and case-insensitive email checks.
- Status uses numeric `1` active and `0` inactive.
- Address is required on create/update.
- Institutional lookup/reference endpoints publish seeded numeric options and
  return full option lists.
- Branding documents color defaults/validation, Element Plus color-picker
  frontend expectation, JSON vs multipart request behavior, and logo upload
  validation/replacement behavior.
- Error responses remain keyed to documented field paths and must not disclose
  cross-tenant existence or private file storage details.

## Complexity Tracking

No constitution violations.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
