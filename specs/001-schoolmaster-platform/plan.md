# Implementation Plan: SchoolMaster Platform Foundation

**Branch**: `001-schoolmaster-platform` | **Date**: 2026-05-13 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-schoolmaster-platform/spec.md`

**Planning Focus**: Full v1 platform foundation across tenant onboarding,
teacher workflows, student progress, reporting, and the latest report output
retention/regeneration clarification.

## Summary

Define the v1 implementation design for SchoolMaster as a multi-tenant school
management SaaS delivered across separate specification, backend, and frontend
repositories. The design covers all three launch stories:

- P1 tenant onboarding, academic setup, user administration, roles,
  permissions, and guardian management.
- P2 teacher content, questionnaire, learning set, selected-student grade and
  attendance entry, upload, and malware-scan-gated content workflows.
- P3 student learning timeline, academic self-view, asynchronous reports,
  report output expiry, and report regeneration through a new `ReportRun`.

The implementation remains contract-first: `schoolmaster-specs` records product
rules, data model impact, OpenAPI response semantics, verification expectations,
and repository coordination before backend or frontend implementation begins.

## Technical Context

- **Language/Version**: PHP 8.x for backend, TypeScript with Vue 3 SPA for
  frontend, Markdown/OpenAPI for specification artifacts
- **Primary Dependencies**: Laravel API stack, Vue 3 SPA stack, Pinia, Vue
  Router, Axios, Tailwind CSS, OpenAPI 3.1, Redocly CLI for OpenAPI validation
- **Storage**: MySQL for transactional tenant data, private tenant-scoped object
  storage for teacher content and generated report files, specification
  repository for business rules and API contracts
- **Testing**: PHPUnit for backend feature/unit coverage, Vitest for frontend
  services/stores/composables/routes, Redocly OpenAPI validation and
  response-shape verification for published endpoints
- **Target Platform**: Browser-based SaaS consumed by school staff and students,
  with backend REST services hosted on web infrastructure
- **Project Type**: Multi-repository web application platform with API backend,
  SPA frontend, and shared specification/contracts repository
- **Performance Goals**: Tenant-scoped requests resolve authorization before
  data access; report downloads use generated files when unexpired; expired
  outputs require a new asynchronous `ReportRun` instead of download-time
  regeneration
- **Constraints**: Strict tenant isolation, `/api/v1` versioning,
  OpenAPI-first delivery, UUID identifiers for cross-boundary entities, private
  content/report storage, upload validation and sanitization, malware scanning
  before content availability, 90-day report output retention, metadata retained
  after output expiry, no automatic regeneration during download
- **Scale/Scope**: V1 platform foundation covering system administrators,
  school administrators, teachers, and students across schools, users, roles,
  permissions, academic years, academic periods, guardians, teacher content,
  questionnaires, learning sets, grades, attendance, student views, and reports

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- OpenAPI impact is identified, and all externally visible behavior starts from
  `contracts/openapi.yaml` before backend or frontend implementation.
- Backend, frontend, and specification repository impacts are called out
  separately, including delivery sequencing and feature identifier traceability.
- Backend design uses Laravel feature organization, Service Layer, Form
  Requests, Policies, API Resources, UUID-based public identifiers, DTOs for
  multi-field workflows, and Repositories only for complex tenant-scoped access.
- Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind CSS,
  Axios-based service modules, feature modules, and service-isolated API access.
- MySQL, tenant-scoping impact, cross-tenant access rules, private file storage,
  upload sanitization, and soft-delete expectations are documented for affected
  workflows.
- API compatibility, authentication or authorization impact, success/error
  response expectations, pagination, filtering, sorting, tenancy semantics,
  inactive-record handling, and response envelopes are documented for changed
  endpoints.
- PHPUnit, Vitest, executable OpenAPI validation, and response-shape
  verification cover each changed critical business flow before merge.
- No constitution deviation is introduced by this planning refinement.

**Gate Status**: PASS FOR PLANNING. Implementation remains blocked until the
feature OpenAPI contract and the aggregate `api/openapi.yaml` pass validation
or an explicit promotion/review note documents why aggregate publication is
deferred.

## V1 Scope Design

- `School` is the v1 tenant root. School-owned records use explicit `school_id`
  tenancy and deny-by-default cross-tenant access.
- System administrators may provision and review school tenants through
  platform-scope policies only. They do not gain implicit bypass for
  school-scoped module actions.
- School administrators manage tenant-local users, academic structure,
  guardians, roles available to their school, and reports.
- Teachers manage instructional content, questionnaires, learning sets, grades,
  and attendance only inside their resolved school tenant and active academic
  period rules.
- Students access assigned learning sets, clean authorized content downloads,
  grades, and attendance only through student self-view contracts.

## Academic Recording Boundary

- V1 does not define class, course, section, or enrollment-group entities.
- Teacher grade and attendance entry is recorded directly against selected
  active `StudentProfile` records for the selected academic period.
- Any classroom, course, section, group, or roster-based recording workflow is
  outside v1 until a class/course model is specified in a later feature.
- Backend services must validate that selected students, teachers, records, and
  academic periods belong to the same active tenant context before persistence.
- Frontend teacher workflows may group selected students visually, but cannot
  depend on undocumented class/course APIs or local-only grouping semantics.

## Malware Scanning Strategy

- Uploaded teacher content is persisted in private tenant-scoped storage with
  `scan_status = pending` and is unavailable for teacher download, student
  download, or learning-set consumption until scanning completes successfully.
- The backend owns the scan-status workflow through a `TeacherContentScanService`
  and an asynchronous scan job or adapter boundary.
- V1 may use a pluggable scanner adapter so local, hosted, or third-party
  scanning can be selected by deployment configuration without changing the API
  contract.
- Scan failures set `scan_status = failed`, keep the file unavailable, and
  expose only tenant-safe failure metadata through the API.
- Only authorized backend workflows may transition content to `clean`; clients
  cannot set scan status directly.

## Report Retention and Regeneration Decisions

- Generated report output files are retained for 90 days after generation.
- `ReportRun` metadata remains available after output files expire.
- Expired report output files are not regenerated automatically during download.
- Users must request a new `ReportRun` with the same filters to generate fresh
  output files after expiry.
- Report output expiry is tenant-bound and must not bypass existing report
  authorization or filter rules.

## Tenancy Strategy

- `School` remains the tenant root for users, academic structures, guardians,
  teacher content, questionnaires, learning sets, grades, attendance, reports,
  and generated output files.
- Every authenticated school-scoped request must resolve an active tenant before
  module-specific logic or file access.
- Tenant scope is enforced at service, query/repository, policy, contract, and
  test layers; the default access stance is deny by default.
- Inactive schools and users cannot participate in normal operational workflows.
- Expired report-output handling and malware-scan failure handling must not
  reveal cross-tenant file existence.

## Contract Strategy

- `schoolmaster-specs` owns the OpenAPI contract as the source of truth for
  payloads, status codes, authentication expectations, pagination, filtering,
  sorting, tenancy semantics, and error envelopes.
- Feature work first updates
  `specs/001-schoolmaster-platform/contracts/openapi.yaml`.
- The repository aggregate `api/openapi.yaml` is a publication target and must
  either be updated with approved operations or explicitly documented as
  deferred before dependent implementation begins.
- P1 contracts define schools, users, roles, permissions, academic years,
  academic periods, and guardians.
- P2 contracts define teacher content, upload/download authorization, scan
  status, questionnaires, learning sets, learning set assignment, grades, and
  attendance.
- P3 contracts define student learning timeline, student grades, student
  attendance, content downloads, report requests, report status, output
  availability, and expired-output responses.
- `ReportRun` responses expose output availability metadata so clients can
  distinguish generated, unexpired outputs from expired output files.

## Verification Strategy

- Contract validation must be executable, not only described. The planning
  quickstart records Redocly commands for both the feature contract and the
  aggregate contract.
- Backend work includes PHPUnit coverage for tenant isolation, authorization,
  inactive status handling, upload validation, malware scan availability gates,
  selected-student grade/attendance entry, report generation, output expiry,
  metadata retention, fresh report requests with the same filters, and expired
  output download errors.
- Frontend work includes Vitest coverage for API services, stores, composables,
  route guards, tenant/session mismatch behavior, student self-view, teacher
  workflows, reporting flows, and expired-output handling.
- Response-shape verification must confirm success, validation failure,
  forbidden, not found, inactive-record, tenant-mismatch, scan-pending/failed,
  and expired-output envelopes match OpenAPI.

## Cross-Repository Traceability

- All related backend, frontend, and specification branches, pull requests, and
  issues use the feature identifier `001-schoolmaster-platform`.
- The specification repository leads business rules and contract changes.
- Backend work links to the OpenAPI operation IDs it implements.
- Frontend work links to the OpenAPI operation IDs it consumes.
- Implementation repositories must not introduce undocumented endpoints,
  payload fields, permission semantics, or tenant behavior.

## Data Lifecycle Strategy

- Recoverable tenant-owned operational records use status fields and soft-delete
  support unless a permanent deletion path is explicitly approved.
- Teacher content files remain private and unavailable until upload validation,
  authorization, tenant ownership checks, and malware scanning succeed.
- `ReportRun` metadata remains the durable audit/history record after output
  files expire.
- Generated PDF/CSV files are private tenant-scoped objects retained for 90 days
  after generation.
- Output availability is determined from generation and expiry metadata, not
  from client-side assumptions.
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
│   ├── Jobs/
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
│   ├── modules/admin/
│   ├── modules/teacher/
│   ├── modules/student/
│   ├── modules/reports/
│   ├── router/
│   ├── services/
│   └── stores/
└── tests/
    ├── modules/
    └── router/

# Contracts and shared delivery artifacts
schoolmaster-specs/
├── api/openapi.yaml
└── specs/001-schoolmaster-platform/
```

**Structure Decision**: Use `schoolmaster-specs` as the source of truth for
business rules, OpenAPI contracts, data-model boundaries, and validation
guidance; `schoolmaster-backend` for tenant-aware Laravel services, policies,
resources, persistence, jobs, and tests; and `schoolmaster-frontend` for Vue 3
feature modules that consume only published `/api/v1` contracts.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations currently identified |
