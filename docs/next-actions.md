# Next Actions

## Current Status

- The active feature branch is `012-report-lifecycle-expansion`.
- Spec Kit prerequisites pass for the current feature artifacts.
- `specs/001-schoolmaster-platform/contracts/openapi.yaml` is the active
  feature contract.
- `api/openapi.yaml` is the aggregate publication target and now includes the
  approved reporting foundation from `005-backend-student-reporting` and the
  approved guardian self-service surface from `011-guardian-self-service`.
- OpenAPI validation is standardized on Redocly CLI using `redocly.yaml`.
- P1 foundation areas have baseline contract coverage: authentication, schools,
  users, roles, permissions, academic years, academic periods, and guardians.
- P2 teacher workflow contract detail has been expanded for teacher content,
  questionnaires, learning sets, grades, and attendance.
- P3 student and reporting contract detail has been expanded for student
  learning timelines, student academic records, authorized content downloads,
  asynchronous report requests, report output downloads, 90-day output
  retention, and explicit regeneration through a new `ReportRun`.
- Guardian self-service contract coverage is now in place for guardian student
  list/detail, academic summary, contact view, and school-admin
  guardian-user-link provisioning.
- Report lifecycle expansion planning is now in place for retry, cancellation,
  soft delete/restore, report catalog discovery, custom report definitions,
  XLSX output, and per-format output availability.

## Immediate Sequence

1. Implement Phase 2 foundational backend work for
   `012-report-lifecycle-expansion`.
2. Expand `api/openapi.yaml` for report lifecycle, report catalog, report
   definitions, custom report requests, and XLSX output before backend routes
   expose new behavior.
3. Remove backend assumptions from the reporting slice that retry, cancel,
   soft delete/restore, and report-definition endpoints must remain
   unexposed.
4. Implement report-run lifecycle foundations first: state separation,
   soft-delete visibility, per-format output availability, and audit support.
5. Implement report catalog and custom definition workflows after lifecycle
   foundations pass route-to-operation traceability checks.
6. Run backend PHPUnit coverage for tenant isolation, authorization,
   conflict handling, snapshot preservation, retention, and audit events.
7. Keep backend and frontend repositories pinned to an approved
   `schoolmaster-specs` commit or submodule revision.

## Resolved Decisions

- **OpenAPI validation**: use Redocly CLI with the repository `redocly.yaml`
  configuration.
- **Validation command**: run
  `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1`.
- **Promotion timing**: do not promote P1 paths into `api/openapi.yaml`
  automatically. Promote only after P1 contract review passes and the aggregate
  publication change is intentional.

## Product Clarifications Required

- No high-impact P1 through P3 product clarifications are currently open for
  contract planning.

## Readiness Gates

- Do not implement undocumented API behavior in backend or frontend
  repositories.
- Do not implement report lifecycle expansion backend routes that drift from
  the approved OpenAPI operation IDs and response envelopes.
- Do not implement report output retention or regeneration behavior outside the
  90-day retention and explicit new-`ReportRun` contract.
- Do not expose manual report status mutation, output delete/restore,
  platform-wide reporting, or support-user cross-school overrides in this
  slice.
- Link backend and frontend work to feature id
  `012-report-lifecycle-expansion` and to the OpenAPI operation IDs
  implemented or consumed.

## Promotion Gate

Before promoting feature contract behavior into `api/openapi.yaml`:

- the promoted operations must pass OpenAPI validation
- each promoted operation must have an operation ID
- request and response schemas must be concrete enough for backend and frontend
  implementation
- tenant-context and authorization behavior must be documented
- standard success and error envelopes must be documented
- P2 or P3 placeholder operations must not be promoted as implementation-ready
