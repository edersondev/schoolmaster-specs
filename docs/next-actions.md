# Next Actions

## Current Status

- The active feature branch is `001-schoolmaster-platform`.
- Spec Kit prerequisites pass for the current feature artifacts.
- `specs/001-schoolmaster-platform/contracts/openapi.yaml` is the active
  feature contract.
- `api/openapi.yaml` is the aggregate publication target and is not yet the
  implementation-ready contract.
- OpenAPI validation is standardized on Redocly CLI using `redocly.yaml`.
- P1 foundation areas have baseline contract coverage: authentication, schools,
  users, roles, permissions, academic years, academic periods, and guardians.
- P2 teacher workflow contract detail has been expanded for teacher content,
  questionnaires, learning sets, grades, and attendance.
- P3 student and reporting contract detail has been expanded for student
  learning timelines, student academic records, authorized content downloads,
  asynchronous report requests, report output downloads, 90-day output
  retention, and explicit regeneration through a new `ReportRun`.

## Immediate Sequence

1. Review the `006-backend-student-enrollment` plan and design artifacts.
2. Run `/speckit-analyze` for cross-artifact consistency before task
   generation.
3. Expand OpenAPI for the approved student profile and enrollment operations
   before backend implementation.
4. Use [Backend feature roadmap](backend-feature-roadmap.md) for the remaining
   backend feature sequence after `006`.
5. Run the approved OpenAPI validation command before promoting contract
   behavior.
6. Keep P1 implementation aligned to the active feature contract until a
   deliberate promotion updates `api/openapi.yaml`.
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
- Do not consume `api/openapi.yaml` as implementation-ready while it has no
  promoted paths.
- Do not implement report output retention or regeneration behavior outside the
  90-day retention and explicit new-`ReportRun` contract.
- Link backend and frontend work to feature id `001-schoolmaster-platform` and
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
