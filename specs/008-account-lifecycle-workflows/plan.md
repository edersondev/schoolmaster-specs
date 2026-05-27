# Implementation Plan: Backend Account Lifecycle Workflows

**Branch**: `008-account-lifecycle-workflows` | **Date**: 2026-05-27 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `specs/008-account-lifecycle-workflows/spec.md`

**Note**: This plan defines the backend implementation boundary after `007-administration-lifecycle`. It does not decompose tasks and does not authorize product behavior outside the specification and OpenAPI contract.

## Summary

Implement the backend account lifecycle slice for invitation creation, invitation completion, initial password setup, password reset request and completion, account lock review or mutation where allowed, account recovery, and account reactivation. The backend must expand the OpenAPI contract before implementation, then expose only approved `/api/v1` operations that preserve tenant-by-column isolation, explicit platform versus school authorization, non-enumerating reset behavior, single-use token semantics, audited email delivery request metadata, reset-request throttling, bearer-token revocation after credential or access-state changes, durable administrative locks until authorized clearance, and inactive-user/school rejection rules. The design keeps platform account administrators limited to platform users and school user administrators limited to same-school users, defers frontend and messaging implementation, and blocks roster, teacher correction, guardian self-service, reporting expansion, billing, and undocumented APIs.

## Technical Context

**Language/Version**: PHP 8.3+ with Laravel API conventions already established by backend foundation, school-admin, teacher-workflow, student-reporting, student-enrollment, and administration-lifecycle slices  
**Primary Dependencies**: Laravel routing, middleware, Eloquent ORM, Form Requests, Policies, API Resources, Services, DTOs for invitation/reset/recovery inputs, repositories or query objects only for complex tenant-scoped account/token lookup and throttling checks, PHPUnit, Redocly CLI for contract validation  
**Storage**: MySQL for users, roles, schools, account invitation state, password setup state, password reset request state, account lock/recovery state, token hashes, token expiry/supersession metadata, reset request throttle metadata, bearer token revocation metadata, email delivery request metadata, audit events, and tenant-scoped actor context  
**Testing**: PHPUnit feature and unit tests in `schoolmaster-backend`; OpenAPI validation through Redocly from `schoolmaster-specs`; backend tests must run with `docker exec schoolmaster-backend-app-1 php artisan test`  
**Target Platform**: API-only backend service consumed by the SchoolMaster Vue SPA through the published `/api/v1` REST contract  
**Project Type**: Laravel API backend slice governed by the shared specification and OpenAPI repository  
**Performance Goals**: Tenant context resolves before school-owned account lifecycle data access for 100% of protected operations; token creation, completion, supersession, bearer-token revocation, invitation revocation, reset request suppression, and audit writes complete atomically; non-enumerating reset request responses remain consistent for eligible, ineligible, and over-limit identifiers  
**Constraints**: OpenAPI expansion required before backend exposure, no undocumented endpoints or fields, no product Blade views, no frontend implementation, no notification or messaging implementation, no provider-specific email behavior, no SMS delivery, no classroom/course/section/roster model, no teacher correction workflow, no guardian self-service, no report lifecycle expansion, no permanent purge, UUID public identifiers, `school_id` as the v1 tenant column, hashed single-use lifecycle tokens only, contract-aligned response envelopes only  
**Scale/Scope**: Proposed boundary covers account invitation creation/completion, first password setup, password reset request/completion, administrative lock/unlock or recovery where approved, and reactivation. Invitation tokens expire after 7 days, reset tokens expire after 30 minutes, new or resent lifecycle tokens supersede prior unused tokens of the same type for the same user and scope, invitation sends/resends are limited to 3 per user and scope per 24 hours, invitations are revoked after 5 failed completion attempts per invitation/account/IP within 15 minutes, password reset requests are limited to 3 per account or IP per 1 hour without changing the non-enumerating accepted response, and failed reset-token completions are limited to 5 per account or IP within 15 minutes before 15-minute new-token suppression

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- **OpenAPI impact**: PASS. The current contract does not publish account lifecycle operations, so this plan requires contract expansion before backend implementation.
- **Repository impact**: PASS. `schoolmaster-specs` leads the plan and contract boundary, `schoolmaster-backend` implements the backend slice, and `schoolmaster-frontend` has no implementation change in this feature.
- **Backend architecture**: PASS. The plan requires feature-organized Laravel controllers, Form Requests, Policies, API Resources, Services, UUID identifiers, DTOs for account lifecycle inputs, and repositories/query objects only for complex tenant-scoped reads, token lookup, throttling, and revocation checks.
- **Frontend architecture**: PASS. No frontend code changes are included; future frontend consumption must remain service-isolated and contract-based.
- **Tenancy and data boundaries**: PASS. School-scoped account lifecycle is governed by `school_id`, platform account lifecycle remains platform-scoped, cross-tenant access remains deny-by-default, and soft-deleted users cannot be recovered through this slice before administration restore rules allow it.
- **API compatibility and auth**: PASS. The plan is additive, requires explicit platform account administrator or school user administrator permission by operation, preserves platform/school authorization separation, and documents token/session response expectations.
- **Verification**: PASS. PHPUnit feature/unit coverage and Redocly contract validation are required before backend implementation merges. No frontend implementation is planned, so Vitest is deferred until frontend consumption begins.
- **Constitution deviations**: PASS. No deviations are introduced.

## Project Structure

### Documentation (this feature)

```text
specs/008-account-lifecycle-workflows/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── backend-account-lifecycle.md
└── checklists/
    └── requirements.md
```

### Source Code (target repositories)

```text
# Backend repository (Laravel API)
schoolmaster-backend/
├── app/
│   ├── DTOs/
│   │   └── AccountLifecycle/
│   ├── Http/
│   │   ├── Controllers/Api/V1/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   │   └── AccountLifecycle/
│   └── Repositories/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/
    └── Unit/

# Frontend repository (Vue 3)
schoolmaster-frontend/
└── No implementation changes in this feature.

# Contracts and shared delivery artifacts
schoolmaster-specs/
├── api/openapi.yaml
├── specs/001-schoolmaster-platform/contracts/openapi.yaml
└── specs/008-account-lifecycle-workflows/
```

**Structure Decision**: Keep this as a backend-only implementation slice with specification artifacts in `schoolmaster-specs`. Backend code should follow the existing Laravel API structure used by previous slices, with account lifecycle rules in services, request validation in Form Requests, authorization in Policies, API Resources for response shaping, DTOs for coordinated token/recovery inputs, and repositories/query objects only where token lookup, tenant-scoped account lookup, throttle state, or revocation checks become complex. Frontend work is explicitly deferred until the backend behavior is contract-compliant and ready to consume.

## Phase 0 Research Summary

Research is captured in [research.md](./research.md). Key decisions:

- Account lifecycle is the next backend slice after administration lifecycle, not roster, teacher correction, guardian self-service, reporting, or frontend delivery.
- OpenAPI must be expanded before backend implementation because invitation, password setup, reset, recovery, lock, and reactivation operations are not currently published.
- Platform account administrators manage platform users only; school user administrators manage same-school users only through active permitted school context.
- Invitation and reset tokens are hashed, single-use, scoped, and supersede prior unused tokens of the same type for the same user and scope.
- Invitation tokens expire after 7 days; password reset tokens expire after 30 minutes.
- Password setup and reset passwords must be 12 to 128 characters, not common passwords, and compatible with paste/password-manager input.
- Invitation sends/resends are limited to 3 per user and scope per 24 hours; invitations are revoked after 5 failed completion attempts per invitation, account, or IP within 15 minutes.
- Password reset requests are limited to 3 per account or IP per 1 hour; over-limit requests keep the non-enumerating accepted envelope but create no token or email delivery request.
- Password reset completion and account access-state changes revoke all active bearer tokens for the affected user.
- Administrative locks remain active until authorized unlock or recovery clears them.
- Password reset uses non-enumerating request behavior, reset-request throttling, and reset-token failure suppression instead of whole-account lockout.
- Delivery behavior is represented only as auditable email delivery request metadata; messaging implementation, provider-specific behavior, and SMS delivery remain out of scope.

## Phase 1 Design Summary

Design artifacts are captured in:

- [data-model.md](./data-model.md): affected entities, fields, relationships, validation rules, token state, throttle state, and account lifecycle transitions.
- [contracts/backend-account-lifecycle.md](./contracts/backend-account-lifecycle.md): proposed operation boundary, contract expansion requirements, tenant and authorization rules, response dependencies, blocked contract gaps, and verification expectations.
- [quickstart.md](./quickstart.md): validation walkthrough and commands for contract and backend readiness.

## Post-Design Constitution Check

- **OpenAPI impact**: PASS. Design artifacts require contract expansion before implementation and do not approve backend-local behavior outside OpenAPI.
- **Repository impact**: PASS. The docs identify `schoolmaster-specs` as source of truth and `schoolmaster-backend` as the only implementation target for this slice.
- **Backend architecture**: PASS. Data and contract notes preserve Service Layer, Form Request, Policy, Resource, DTO, Repository/query object, UUID, token hashing, throttle-state, audit-event, bearer-token revocation, and tenant-scope expectations.
- **Frontend architecture**: PASS. No frontend source change is planned.
- **Tenancy and data boundaries**: PASS. School-scoped account lifecycle requires `school_id` context, platform account lifecycle remains platform-scoped, and platform access does not bypass school-owned account lifecycle permissions.
- **API compatibility and auth**: PASS. The plan is additive, requires contract revision before exposure, preserves platform/school authorization separation, and defines token/session behavior at the contract boundary.
- **Verification**: PASS. Quickstart defines PHPUnit and OpenAPI validation expectations for all affected critical flows.
- **Constitution deviations**: PASS. No deviations are introduced.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| None | N/A | No constitution violations are introduced by this plan |
