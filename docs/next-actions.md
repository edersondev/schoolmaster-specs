# Next Actions

## Current Status

- The active feature branch is `013-platform-support-access`.
- Spec Kit prerequisites pass for the current feature artifacts.
- `specs/001-schoolmaster-platform/contracts/openapi.yaml` is the active
  feature contract.
- `api/openapi.yaml` is the aggregate publication target and includes the
  approved reporting foundation from `005-backend-student-reporting`, the
  approved guardian self-service surface from `011-guardian-self-service`, and
  the implemented report lifecycle expansion from
  `012-report-lifecycle-expansion`.
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
- Platform-wide reporting and support access planning is now in place for
  minimized platform school summaries, cross-school reporting overview,
  read-only support drill-down, target-school opt-in, internal platform
  approval, 24-hour approval gates, support audit review, redaction, and
  small-count suppression.

## Immediate Sequence

1. Complete Phase 1 and Phase 2 foundational contract work for
   `013-platform-support-access`.
2. Expand `api/openapi.yaml` and
   `specs/001-schoolmaster-platform/contracts/openapi.yaml` for all platform
   support operations before backend routes expose new behavior.
3. Run the contract-first Redocly validation gate before backend route
   exposure.
4. Implement minimized platform school summaries and cross-school reporting
   overview as the MVP.
5. Implement read-only support drill-down only after target-school opt-in,
   internal platform approval, expiration, revocation, mismatch, and
   concurrency gates are covered.
6. Implement support audit review with minimized metadata, tenant-safe reason
   codes, and escalation event coverage.
7. Run backend PHPUnit coverage for platform authorization, tenant-safe
   denials, small-count suppression, approval gates, blocked support behavior,
   and audit redaction.
8. Keep backend and frontend repositories pinned to an approved
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
- Do not implement platform-wide reporting or support access backend routes
  that drift from the approved OpenAPI operation IDs and response envelopes.
- Do not expose support writes, generated report downloads, emergency access,
  unrestricted impersonation, private file metadata, raw report output, or
  unrestricted record search in this slice.
- Do not expose school-scoped student, teacher, guardian, administration, or
  reporting endpoint access to platform/support actors unless an
  operation-specific contract grants access.
- Link backend and frontend work to feature id
  `013-platform-support-access` and to the OpenAPI operation IDs
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
