# Implementation Plan: System Administrator Master Access

**Branch**: `031-system-admin-master` | **Date**: 2026-07-13 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/031-system-admin-master/spec.md`

## Summary

Define System Administrator as the master user role across released protected
backend operations. This backend implementation slice is contract-first:
specification, security/tenancy documentation, and all affected OpenAPI
authorization notes lead; Laravel authorization, tenant isolation, audit
behavior, and PHPUnit coverage follow. Frontend adoption is deferred.

The override satisfies feature-specific permission checks only. Authentication,
account lifecycle, session validity, tenant context, identity ownership and guardian links,
school state, feature release state, approval workflows, explicit
confirmations, support opt-in requirements, file safety gates, closed-period
safety checks, and other business controls remain enforceable.

## Technical Context

**Language/Version**: OpenAPI YAML and Markdown specifications; PHP 8.3/Laravel 13 backend.
**Primary Dependencies**: Laravel authentication/session behavior, `User` role and permission helpers, Policies, Services, Form Requests where protected operations validate input, `AuthSessionResource`, `TenantContextResolver`, module tenant guards, `AuditEventService`, module audit services/stores, and OpenAPI authorization notes.
**Storage**: Existing user, role, permission, school, session, and audit/event records. No new business entity is required; audit evidence may need to record a master-access marker for System Administrator writes and lifecycle actions.
**Testing**: Redocly/OpenAPI lint; PHPUnit feature/unit tests for master-role detection, permission override, tenant context, identity ownership, guardian links, non-System Administrator denial, safety gates, and audit markers.
**Target Platform**: Laravel JSON API under `/api/v1`.
**Project Type**: Shared contract plus Laravel backend authorization change. Frontend implementation is outside this run.
**Performance Goals**: No new dedicated performance target is introduced. Existing authorization request regression checks must continue to pass.
**Constraints**: Contract-first delivery; no undocumented protected operation access; System Administrator may select any active school for school-scoped work; school-owned data remains scoped to the selected school; identity-owned self-service keeps existing actor ownership and guardian-link rules; master access bypasses only permission checks; writes and lifecycle actions require master-access audit evidence; non-System Administrator behavior remains unchanged; no response schema or success envelope changes.
**Scale/Scope**: Global backend permission rule touching all released protected operation groups, school-context selection, identity-ownership gates, existing audit pipelines for writes/lifecycle actions, OpenAPI authorization documentation, PHPUnit coverage, and implementation evidence.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Protected operation authorization notes
  must document System Administrator master access before backend
  implementation changes consume the new rule.
- PASS: Repository impacts are separated. `schoolmaster-specs` owns this spec,
  design artifacts, documentation, and OpenAPI contract updates;
  `schoolmaster-backend` owns Laravel authorization/audit behavior and tests;
  frontend adoption is a deferred follow-up.
- PASS: Backend design uses Laravel feature organization, the existing `User`
  permission methods for the permission-only master decision, Policies for
  authorization, Services for business rules, Form Requests and
  API Resources where existing operations already require them, UUID public
  identifiers, and existing audit/event boundaries. DTO and Repository changes
  are not required unless an affected operation already uses them.
- PASS: Frontend implementation is explicitly deferred and no frontend files
  are changed by this implementation run.
- PASS: MySQL remains authoritative; tenant-by-column rules remain in force;
  System Administrator is a documented platform override for permission checks
  only; school-owned output remains selected-school scoped; soft-delete and
  lifecycle behavior are unchanged.
- PASS: API compatibility, authentication/authorization impact, success/error
  envelope expectations, permission-denial vs tenant/identity-ownership denial,
  and audit-marker expectations are documented.
- PASS: Verification spans OpenAPI lint and PHPUnit backend coverage for the
  changed critical authorization, tenancy, ownership, safety, and audit flows.
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

```

**Structure Decision**: Keep `schoolmaster-specs` as the source of truth for
the master-access rule and contract language. Backend implementation uses
feature identifier `031-system-admin-master`, updates the
existing centralized Laravel permission helpers instead of duplicating
per-feature exceptions, and preserves current feature modules. Frontend work is
tracked as a separate follow-up.

## Phase 0: Research

Research is complete in [research.md](research.md).

Key decisions:

- Treat System Administrator as a global permission-check override documented
  in security, multi-tenant, feature, and OpenAPI contract notes.
- Let System Administrator select any active school for school-scoped work.
- Keep school-owned responses scoped to selected school unless an operation is
  explicitly platform-wide.
- Preserve actor-owned student profile and active guardian-link rules for
  identity-owned self-service operations. No impersonation or subject-selection
  transport is introduced.
- Apply master access to permission checks only; approval workflows and safety
  gates still apply.
- Mark audit evidence for System Administrator writes and lifecycle actions.
- Preserve all non-System Administrator permission-denial behavior.
- Keep success response envelopes unchanged and distinguish permission denial
  from tenant/identity-ownership denial.
- Verify with OpenAPI lint and PHPUnit authorization, tenancy, ownership,
  safety-gate, and audit tests.

## Phase 1: Design And Contracts

Design artifacts are complete:

- [data-model.md](data-model.md)
- [contracts/system-admin-master-contract.md](contracts/system-admin-master-contract.md)
- [quickstart.md](quickstart.md)

Required contract and documentation updates:

- Replace contradictory "System administrator access does not bypass
  school-scoped authorization" language with the clarified rule: System
  Administrator bypasses feature-specific permission checks but not tenant
  isolation, identity ownership, guardian links, account/session state, release state, approval
  workflows, or safety gates.
- Document System Administrator master access in OpenAPI authorization notes
  for protected operations.
- Document any cross-school response as platform-wide before unscoped output is
  returned.
- Preserve tenant-context, identity-ownership, and guardian-link error behavior
  separately from permission-denial behavior.
- Require audit evidence for System Administrator writes and lifecycle actions.
- Identify System Administrator through the existing active platform role named
  exactly `System Administrator`; add no role field to the session contract.
- Implement the permission-only override through `User::hasPermission()` and
  `User::hasSchoolPermission()`, then correct policy/service preconditions that
  assume all school-authorized users have a fixed `users.school_id`.
- Reuse `TenantContextResolver`, `TenantContextService`, and existing module
  tenant guards instead of introducing fictional global school-context helpers.
- Carry `master_access_used` through the generic and module-specific existing
  audit pipelines; do not introduce a second audit framework.

Post-design Constitution Check: PASS. Research, data model, contract, and
quickstart preserve API-first delivery, repository sequencing, Laravel
boundaries, tenant safety, compatibility expectations, and required OpenAPI and
PHPUnit verification with no deviations.

## Complexity Tracking

No constitution violations.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
