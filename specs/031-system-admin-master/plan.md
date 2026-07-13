# Implementation Plan: System Administrator Master Access

**Branch**: `031-system-admin-master` | **Date**: 2026-07-13 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/031-system-admin-master/spec.md`

## Summary

Define System Administrator as the master user role across released frontend
routes and protected backend operations. The implementation is contract-first:
specification and OpenAPI authorization notes lead, then backend authorization
rules and tests, then frontend route guards, navigation/action visibility, and
tests.

The override satisfies feature-specific permission checks only. Authentication,
account lifecycle, session validity, tenant context, selected subject context,
school state, feature release state, approval workflows, explicit
confirmations, support opt-in requirements, file safety gates, closed-period
safety checks, and other business controls remain enforceable.

## Technical Context

**Language/Version**: OpenAPI YAML and Markdown specifications; PHP 8.x/Laravel 10+ backend; Vue 3 JavaScript SPA using Composition API.
**Primary Dependencies**: Laravel authentication/session behavior, role/permission checks, Policies, Services, Form Requests where protected operations validate input, API Resources, audit/event recording; Vue Router guards, Pinia session/shell state, Axios service modules, existing admin/system route metadata, and OpenAPI authorization notes.
**Storage**: Existing user, role, permission, school, session, and audit/event records. No new business entity is required; audit evidence may need to record a master-access marker for System Administrator writes and lifecycle actions.
**Testing**: Redocly/OpenAPI lint; PHPUnit feature/unit tests for authorization override, tenant context, subject context, non-System Administrator denial, safety gates, and audit markers; Vitest tests for frontend route guards, navigation visibility, tenant/subject gating, and denied states.
**Target Platform**: Laravel JSON API under `/api/v1`; Vue 3 SPA consuming the published API contract.
**Project Type**: Multi-repository web application authorization change spanning specifications, backend API, and frontend SPA.
**Performance Goals**: No new dedicated performance target is introduced. Existing authorization request and route-evaluation regression checks must continue to pass, and System Administrator route visibility must settle in the same session-resolution flow as existing permission-based visibility.
**Constraints**: Contract-first delivery; no undocumented protected route or operation access; System Administrator may select any active school for school-scoped work; school-owned data remains scoped to the selected school; identity-owned self-service routes require selected subject context; master access bypasses only permission checks; writes and lifecycle actions require master-access audit evidence; non-System Administrator behavior remains unchanged; no success response envelope changes.
**Scale/Scope**: Global authorization rule touching all released protected route and operation groups, school-context selection, self-service subject-context gating, audit evidence for writes/lifecycle actions, OpenAPI authorization documentation, backend authorization tests, frontend guard/navigation tests, and documentation updates.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Protected operation authorization notes
  must document System Administrator master access before backend/frontend
  implementation changes consume the new rule.
- PASS: Repository impacts are separated. `schoolmaster-specs` owns this spec,
  design artifacts, documentation, and OpenAPI contract updates;
  `schoolmaster-backend` owns Laravel authorization/audit behavior and tests;
  `schoolmaster-frontend` owns route guard/navigation/action visibility and
  tests.
- PASS: Backend design uses Laravel feature organization, Policies for
  authorization, Services for shared master-access decisions, Form Requests and
  API Resources where existing operations already require them, UUID public
  identifiers, and existing audit/event boundaries. DTO and Repository changes
  are not required unless an affected operation already uses them.
- PASS: Frontend design uses Vue 3 Composition API, Vue Router route metadata
  and guards, existing Pinia session/shell state, Axios service modules,
  feature modules, Tailwind CSS, and service-isolated API access. Components do
  not own authorization business rules.
- PASS: MySQL remains authoritative; tenant-by-column rules remain in force;
  System Administrator is a documented platform override for permission checks
  only; school-owned output remains selected-school scoped; soft-delete and
  lifecycle behavior are unchanged.
- PASS: API compatibility, authentication/authorization impact, success/error
  envelope expectations, permission-denial vs tenant/subject-context denial,
  and audit-marker expectations are documented.
- PASS: Verification spans OpenAPI lint, PHPUnit backend coverage, and Vitest
  frontend coverage for the changed critical authorization flows.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/031-system-admin-master/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── system-admin-master-contract.md
└── tasks.md             # Phase 2 output from /speckit-tasks
```

### Source Code (target repositories)

```text
# schoolmaster-specs
api/
├── openapi.yaml
├── paths/
└── components/responses/
docs/
├── security.md
└── multi-tenant.md
specs/031-system-admin-master/

# schoolmaster-backend (Laravel API)
app/
├── Http/
│   ├── Controllers/Api/V1/
│   ├── Requests/Api/V1/
│   └── Resources/
├── Models/
├── Policies/
└── Services/
routes/api.php
tests/
├── Feature/
└── Unit/

# schoolmaster-frontend (Vue 3 SPA)
src/
├── router/
├── stores/
├── services/
├── composables/
├── pages/
└── components/
tests/
└── unit/
```

**Structure Decision**: Keep `schoolmaster-specs` as the source of truth for
the master-access rule and contract language. Backend and frontend
implementations must share feature identifier `031-system-admin-master`, update
their existing centralized authorization/route guard boundaries instead of
duplicating per-feature exceptions, and preserve current feature modules.

## Phase 0: Research

Research is complete in [research.md](research.md).

Key decisions:

- Treat System Administrator as a global permission-check override documented
  in security, multi-tenant, feature, and OpenAPI contract notes.
- Let System Administrator select any active school for school-scoped work.
- Keep school-owned responses scoped to selected school unless an operation is
  explicitly platform-wide.
- Require selected subject context before identity-owned self-service routes
  load student, guardian, user, or similar person-specific data.
- Apply master access to permission checks only; approval workflows and safety
  gates still apply.
- Mark audit evidence for System Administrator writes and lifecycle actions.
- Preserve all non-System Administrator permission-denial behavior.
- Keep success response envelopes unchanged and distinguish permission denial
  from tenant/subject-context denial.
- Verify with OpenAPI lint, PHPUnit authorization/audit tests, and Vitest
  route/navigation tests.

## Phase 1: Design And Contracts

Design artifacts are complete:

- [data-model.md](data-model.md)
- [contracts/system-admin-master-contract.md](contracts/system-admin-master-contract.md)
- [quickstart.md](quickstart.md)

Required contract and documentation updates:

- Replace contradictory "System administrator access does not bypass
  school-scoped authorization" language with the clarified rule: System
  Administrator bypasses feature-specific permission checks but not tenant
  isolation, subject context, account/session state, release state, approval
  workflows, or safety gates.
- Document System Administrator master access in OpenAPI authorization notes
  for protected operations.
- Document any cross-school response as platform-wide before unscoped output is
  returned.
- Preserve tenant-context and subject-context error behavior separately from
  permission-denial behavior.
- Require audit evidence for System Administrator writes and lifecycle actions.

Post-design Constitution Check: PASS. Research, data model, contract, and
quickstart preserve API-first delivery, repository sequencing, Laravel/Vue
boundaries, tenant safety, compatibility expectations, and required OpenAPI,
PHPUnit, and Vitest verification with no deviations.

## Complexity Tracking

No constitution violations.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
