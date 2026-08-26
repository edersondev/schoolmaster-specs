# Implementation Plan: Secure User Password Delivery

**Branch**: `039-user-password-delivery` | **Date**: 2026-08-26 | **Spec**: [spec.md](spec.md)

## Summary

Add an explicit, authorized email password-link action for an eligible active
user after creation. Backend owns contract, scope, tenant-first lookup, limit,
token lifecycle, mail handoff, and verification. Frontend follows only after
that gate with safe post-create/detail controls and existing guest completion.

## Technical Context

**Language/Version**: PHP 8.3/Laravel 13; JavaScript/Vue 3
**Primary Dependencies**: Laravel mail/Sanctum/PHPUnit; Vue Router/Pinia/Axios/
Element Plus/Vitest/Playwright
**Storage**: MySQL `school_id`; existing reset-token storage plus safe audit data
**Testing**: Redocly, PHPUnit, Vitest, Playwright, production build
**Target Platform**: Laravel `/api/v1` and responsive SPA
**Project Type**: Specs, backend, frontend
**Constraints**: Email only; no auto-send; no secrets; 3/user/scope/24h;
locked/invited/inactive/deleted/cross-tenant denial; backend before frontend

## Constitution Check

*GATE: PASS before Phase 0; re-check after design.*

- PASS: OpenAPI leads additive `requestUserPasswordDelivery` behavior.
- PASS: All affected repositories use `039-user-password-delivery`.
- PASS: Backend uses Form Request, Policy, service, Resource, UUID target, and
tenant-first access; frontend uses service/composable isolation.
- PASS: Backend contract, implementation, and verification block frontend.
- PASS: Redocly, PHPUnit, Vitest, browser, privacy, and build gates apply.

## Project Structure

```text
schoolmaster-specs/specs/039-user-password-delivery/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
└── contracts/user-password-delivery.md

schoolmaster-backend/app/{Http,Policies,Services}/
schoolmaster-backend/tests/Feature/AccountLifecycle/
schoolmaster-frontend/src/{services,composables,components,pages}/
schoolmaster-frontend/tests/unit/account-lifecycle/
```

## Implementation Approach

### Phase 0: Contract

Add the new authenticated user-scoped OpenAPI operation, safe 201 shape, tenant
header, and documented 401/403/404/409/422/429/503 outcomes. Reuse existing
`completePasswordReset`; do not alter public reset non-enumeration.

### Phase 1: Backend gate

Implement tenant-first lookup, scoped lifecycle authority, active/unlocked
eligibility, 3-per-user/scope/24h limiting, single-use reset-token issuance,
supersession only after accepted mail handoff, safe audit data, and 503 with no
usable new token on mail failure. Add full PHP feature/unit and contract tests.

### Phase 2: Frontend after backend verification

Add safe post-create/detail controls, API service mapping, and route-local
single-flight/retry/stale-context composable. Reuse guest completion. Expose
only status/channel/time; keep tokens/passwords/private data out of UI, storage,
query, logs, and diagnostics. Add Vitest, browser, accessibility, and build
evidence.

## Post-Design Constitution Check

- PASS: Existing reset completion is the only recipient completion flow.
- PASS: No unresolved decisions; backend gate is explicit.

## Complexity Tracking

No constitution deviations.
