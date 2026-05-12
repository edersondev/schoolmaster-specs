# Implementation Plan: SchoolMaster Platform Foundation

**Branch**: `001-schoolmaster-platform` | **Date**: 2026-05-11 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-schoolmaster-platform/spec.md`

**Planning Focus**: Report output retention and regeneration contract update
for the P3 reporting workflow.

## Summary

Define the v1 implementation design for SchoolMaster as a multi-tenant school
management SaaS delivered across separate specification, backend, and frontend
repositories. This planning pass updates the P3 reporting contract after the
approved clarification that generated report output files are retained for 90
days, then expire while `ReportRun` metadata remains available, and that users
must request a new `ReportRun` with the same filters to regenerate fresh
outputs.

The update remains contract-first: `schoolmaster-specs` records the business
rule, data model impact, OpenAPI response semantics, and validation guidance
before backend or frontend implementation begins.

## Technical Context

- **Language/Version**: PHP 8.x for backend, TypeScript with Vue 3 SPA for
  frontend, Markdown/OpenAPI for specification artifacts
- **Primary Dependencies**: Laravel API stack, Vue 3 SPA stack, Pinia, Vue
  Router, Axios, Tailwind CSS, OpenAPI 3.1, Redocly CLI for OpenAPI validation
- **Storage**: MySQL for transactional report metadata, private tenant-scoped
  object storage for generated PDF/CSV report files, specification repository
  for business rules and API contracts
- **Testing**: PHPUnit for backend report retention and authorization behavior,
  Vitest for frontend report services/stores/composables, Redocly OpenAPI
  validation and response-shape verification for published endpoints
- **Target Platform**: Browser-based SaaS consumed by school staff and students,
  with backend REST services hosted on web infrastructure
- **Project Type**: Multi-repository web application platform with API backend,
  SPA frontend, and shared specification/contracts repository
- **Performance Goals**: Report downloads use generated files when unexpired;
  expired outputs require a new asynchronous `ReportRun` instead of download-time
  regeneration
- **Constraints**: Strict tenant isolation, `/api/v1` versioning,
  OpenAPI-first delivery, UUID identifiers for cross-boundary entities,
  private report output storage, 90-day report output retention, metadata
  retained after output expiry, no automatic regeneration during download
- **Scale/Scope**: P3 reporting covers attendance, grades, academic structure,
  and school activity reports with asynchronous PDF/CSV outputs and explicit
  expiry semantics

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- OpenAPI impact is identified, and report expiry behavior starts from the
  contract.
- Backend, frontend, and specification repository impacts are called out
  separately, including delivery sequencing.
- Backend design uses Laravel feature organization, Service Layer, Form
  Requests, Policies, API Resources, UUID-based public identifiers, DTOs for
  report request filters, and query or storage services for tenant-scoped report
  output access.
- Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind CSS,
  Axios-based service modules, feature modules, and service-isolated report API
  access.
- MySQL, tenant-scoping impact, cross-tenant access rules, private report
  output storage, and soft-delete expectations are documented for affected
  workflows.
- API compatibility, authentication or authorization impact, report expiry
  response semantics, and success or error response expectations are documented
  for changed endpoints.
- PHPUnit, Vitest, and OpenAPI validation cover report request, status, output
  download, expiry, and regeneration-entry behavior before backend or frontend
  merge.
- No constitution deviation is introduced by the report retention/regeneration
  contract update.

**Gate Status**: PASS FOR PLANNING. Implementation remains blocked until the
expanded OpenAPI contract validates and backend/frontend work links to the
operation IDs it implements or consumes.

## Report Retention and Regeneration Decisions

- Generated report output files are retained for 90 days after generation.
- `ReportRun` metadata remains available after output files expire.
- Expired report output files are not regenerated automatically during download.
- Users must request a new `ReportRun` with the same filters to generate fresh
  output files after expiry.
- Report output expiry is tenant-bound and must not bypass existing report
  authorization or filter rules.

## Tenancy Strategy

- `School` remains the tenant root for report metadata and generated output
  files.
- Report output downloads must resolve an active tenant before file access.
- Expired output handling must not reveal cross-tenant file existence.
- A new `ReportRun` request after expiry reuses normal report authorization,
  validation, and tenant-bound filter checks.
- Tenant scope is enforced at service, query or repository, policy, contract,
  and test layers; the default access stance is deny by default.

## Contract Strategy

- `schoolmaster-specs` owns the OpenAPI contract as the source of truth for
  payloads, status codes, authentication expectations, pagination, filtering,
  sorting, tenancy semantics, and error envelopes.
- `ReportRun` responses expose output availability metadata so clients can
  distinguish generated, unexpired outputs from expired output files.
- Report output download documents the expired-output error response and keeps
  regeneration as an explicit new `requestReport` action.
- P3 contract changes are additive to the feature contract and preserve the
  established response envelope and standard error responses.
- The repository aggregate `api/openapi.yaml` remains a publication target;
  retention behavior is not promoted there until explicit contract review.

## Cross-Repository Traceability

- All related backend, frontend, and specification branches, pull requests, and
  issues use the feature identifier `001-schoolmaster-platform`.
- The specification repository leads report retention and regeneration
  contract changes.
- Backend work links to `requestReport`, `listReports`, and `downloadReport`
  operation IDs and includes PHPUnit coverage for tenant isolation, output
  expiry, metadata retention, fresh report requests with the same filters, and
  expired output download errors.
- Frontend work links to the same operation IDs and includes Vitest coverage for
  affected report services, stores, composables, route guards, and expired-output
  handling.

## Data Lifecycle Strategy

- `ReportRun` metadata remains the durable audit/history record after output
  files expire.
- Generated PDF/CSV files are private tenant-scoped objects retained for 90 days
  after generation.
- Output availability is determined from generation and expiry metadata, not
  from client-side assumptions.
- Expired outputs are unavailable for download; users create a new asynchronous
  `ReportRun` to generate fresh files.
- Retention cleanup must preserve metadata and must not delete or mutate the
  historical selected filters recorded on the original `ReportRun`.

## Project Structure

### Documentation (this feature)

```text
specs/001-schoolmaster-platform/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── openapi.yaml
└── tasks.md
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
│   ├── Models/
│   ├── Policies/
│   ├── Repositories/
│   └── Services/
├── database/
│   └── migrations/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/
    └── Unit/

# Frontend repository (Vue 3)
schoolmaster-frontend/
├── src/
│   ├── modules/student/
│   ├── modules/reports/
│   ├── router/
│   ├── services/
│   └── stores/
└── tests/
    ├── modules/student/
    ├── modules/reports/
    └── router/

# Contracts and shared delivery artifacts
schoolmaster-specs/
└── specs/001-schoolmaster-platform/
```

**Structure Decision**: Use `schoolmaster-specs` as the source of truth for
report retention and regeneration business rules and OpenAPI contracts,
`schoolmaster-backend` for tenant-aware Laravel report generation, storage, and
expiry behavior, and `schoolmaster-frontend` for Vue 3 reporting screens that
consume only published `/api/v1` endpoints.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations currently identified |
