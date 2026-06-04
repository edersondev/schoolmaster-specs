# Next Actions

## Current Status

- The active feature branch is `011-guardian-self-service`.
- Spec Kit prerequisites pass for the current feature artifacts.
- `specs/001-schoolmaster-platform/contracts/openapi.yaml` is the active
  feature contract.
- `api/openapi.yaml` is the aggregate publication target and now includes the
  approved guardian self-service surface promoted for this feature.
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

## Immediate Sequence

1. Implement Phase 2 foundational backend work for
   `011-guardian-self-service`.
2. Add school-admin guardian-user-link lifecycle support in the backend before
   enabling guardian self-service reads.
3. Implement guardian student list/detail, then academic summary, then contact
   view with route-to-operation traceability.
4. Run backend PHPUnit coverage for tenant isolation, authorization,
   non-enumeration, redaction, transfer visibility, and audit events.
5. Use [Backend feature roadmap](backend-feature-roadmap.md) for the remaining
   backend feature sequence after `011`.
6. Keep backend and frontend repositories pinned to an approved
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
- Do not implement guardian self-service backend routes that drift from the
  approved OpenAPI operation IDs and response envelopes.
- Do not implement report output retention or regeneration behavior outside the
  90-day retention and explicit new-`ReportRun` contract.
- Link backend and frontend work to feature id `011-guardian-self-service` and
  to the OpenAPI operation IDs implemented or consumed.

## Promotion Gate

Before promoting feature contract behavior into `api/openapi.yaml`:

- the promoted operations must pass OpenAPI validation
- each promoted operation must have an operation ID
- request and response schemas must be concrete enough for backend and frontend
  implementation
- tenant-context and authorization behavior must be documented
- standard success and error envelopes must be documented
- P2 or P3 placeholder operations must not be promoted as implementation-ready
