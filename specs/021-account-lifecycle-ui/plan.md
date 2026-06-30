# Implementation Plan: Account Lifecycle Workflows UI

**Branch**: `021-account-lifecycle-ui` | **Date**: 2026-06-28 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/021-account-lifecycle-ui/spec.md`

## Summary

Implement roadmap item 6 in `schoolmaster-frontend`: account invitation entry
from existing user create/detail flows, token-proven invitation setup, email
only password reset request, token-proven password reset completion, account
lock review, administrative lock, unlock, recovery, reactivation, safe
invalid-token feedback, safe denial states, and auth/session integration.

The implementation extends the completed auth/session and administration user
management foundations. Route pages remain thin composition surfaces. Services
own Axios calls and OpenAPI envelope mapping. Composables coordinate token
flows, account lifecycle action state, stale-response protection, password
validation feedback, tenant clearing, and blocked admin resend behavior. Admin
resend stays hidden or blocked because the current `resendAccountInvitation`
operation requires an invitation token; a non-secret resend operation by
invitation or user identifier is required before exposing admin resend.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains out of scope.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, and approved OpenAPI-backed account lifecycle operations.  
**Storage**: No backend or database change. Token-flow form state, lock dialogs, and lifecycle confirmations remain route-local/in-memory. Existing Pinia auth/session state owns current user, permissions, token/session recovery, and active school context. Lifecycle token values and plaintext passwords must not be persisted in reusable frontend state.  
**Testing**: Vitest and Vue Test Utils for services, contract mappers, composables, forms, route pages, route metadata, invitation setup, password reset request/completion, lock state, lock/unlock/recovery/reactivation actions, stale response handling, denial mapping, invalid-token mapping, permission/capability gates, and no-secret diagnostics. Redocly/OpenAPI validation is required only if contract files change.  
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming published `/api/v1` contracts from `schoolmaster-specs`.  
**Project Type**: Frontend SPA feature with specification and cross-repository delivery artifacts.  
**Performance Goals**: With mocked account lifecycle services settling within 1.5 seconds, invitation setup, password reset completion, and account lifecycle action routes render stable success, invalid-token, validation, denied, conflict, or recoverable error states within 2 seconds; neutral password reset request confirmation renders within 15 seconds; stale requests are cancelled or ignored.  
**Constraints**: No undocumented endpoint, field, status, token mode, permission code, capability flag, lifecycle action, or error dependency; no direct Axios in pages/components/router; no `delivery_metadata` submission for invitation creation; no public school selector or `school_id` submission for password reset request; no admin resend UI while only token-based resend exists; no automatic sign-in after setup/reset unless OpenAPI adds it; no persisted lifecycle tokens, plaintext passwords, bearer tokens, reasons, tenant-private details, role internals, or permission payloads; no client-side tenant inference; WCAG 2.1 AA target at 390px, 768px, and 1440px; PascalCase Element Plus tags; centralized reusable text.  
**Scale/Scope**: Four user-story slices, three guest auth routes, existing user create/detail invitation entry points, account lock/action controls on user detail, one account lifecycle service family, one auth/service extension for reset completion, shared feedback primitives, contract mapping, route metadata, and focused frontend tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. This frontend feature consumes existing
  account lifecycle operation IDs and blocks admin resend until a non-secret
  resend contract is added. No OpenAPI change is planned by this slice.
- PASS: Repository impacts are separated. `schoolmaster-specs` defines this
  plan; `schoolmaster-frontend` implements it; `schoolmaster-backend` remains
  unchanged unless contract verification finds an account lifecycle gap.
- PASS: Backend architecture requirements are N/A because no Laravel route,
  Request, Policy, Resource, Service, DTO, Repository, schema, or public
  identifier change is approved.
- PASS: Frontend design uses Vue 3 Composition API, Pinia only for existing
  shared session/shell state, Vue Router, Tailwind CSS, Element Plus, Axios
  service modules, feature folders, and service-isolated API access.
- PASS: MySQL and database soft-delete behavior are unchanged. School-scoped
  account lifecycle UI uses authenticated active school context only; platform
  account lifecycle remains separate; soft-deleted users stay blocked unless
  administration lifecycle restoration exposes them first.
- PASS: API compatibility, authentication, authorization, success envelopes,
  validation, conflict, forbidden, tenant, not-found, invalid-token, and
  non-enumerating reset outcomes are documented.
- PASS: Vitest covers changed critical frontend flows. OpenAPI validation is
  required only if contract review creates a separate contract change.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/021-account-lifecycle-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── account-lifecycle-ui-contract.md
├── checklists/
│   └── requirements.md
└── tasks.md             # Phase 2 output (/speckit-tasks command - NOT created by /speckit-plan)
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
├── api/
│   ├── openapi.yaml
│   ├── paths/account-lifecycle/
│   ├── paths/auth/
│   ├── paths/users/
│   └── components/schemas/account-lifecycle/
├── docs/
│   ├── frontend-architecture.md
│   ├── frontend-admin-system-architecture.md
│   ├── frontend-guidelines.md
│   └── frontend-feature-roadmap.md
├── specs/021-account-lifecycle-ui/
└── AGENTS.md

# Frontend repository target shape
schoolmaster-frontend/
├── src/
│   ├── components/
│   │   ├── auth/
│   │   │   ├── PasswordSetupForm.vue
│   │   │   ├── PasswordResetCompletionForm.vue
│   │   │   ├── AccountLifecycleTokenState.vue
│   │   │   └── AccountLifecycleSuccessState.vue
│   │   ├── admin-system/
│   │   │   └── users/
│   │   │       ├── UserInvitationPanel.vue
│   │   │       ├── AccountLockPanel.vue
│   │   │       └── AccountLifecycleActions.vue
│   │   └── ui/admin/
│   │       └── AdminAccountLifecycleDialog.vue
│   ├── composables/
│   │   └── auth/
│   │       ├── useInvitationSetup.js
│   │       ├── usePasswordResetCompletion.js
│   │       └── usePasswordResetRequest.js
│   ├── composables/admin-system/
│   │   └── useAccountLifecycleActions.js
│   ├── contracts/
│   │   ├── auth/
│   │   │   └── account-lifecycle.js
│   │   └── admin-system/
│   │       └── account-lifecycle.js
│   ├── locales/
│   │   └── account-lifecycle.js
│   ├── pages/
│   │   ├── auth/
│   │   │   ├── InvitationSetupPage.vue
│   │   │   ├── PasswordResetRequestPage.vue
│   │   │   └── PasswordResetCompletionPage.vue
│   │   └── admin-system/
│   │       └── users/
│   │           └── UserDetailPage.vue
│   ├── router/modules/
│   │   ├── auth.routes.js
│   │   └── access-administration.routes.js
│   ├── services/
│   │   ├── auth/
│   │   │   └── accountLifecycle.js
│   │   └── admin-system/
│   │       └── accountLifecycle.js
│   └── stores/
│       └── auth/
│           └── sessionStore.js
└── tests/unit/account-lifecycle/
    ├── contracts/
    ├── services/
    ├── composables/
    ├── components/
    ├── pages/
    └── routes/
```

**Structure Decision**: Extend existing auth/session and admin-system user
folders instead of creating a detached account module. Guest token flows live
under auth pages, auth components, auth composables, and auth services.
Administrative lock, invitation, recovery, and reactivation controls extend
existing admin-system user detail/create surfaces. Services stay split by
auth guest flows versus admin authenticated flows while sharing contract
mapping helpers. No new Pinia account lifecycle store is planned; existing
session state and route-local composables are sufficient.

## Component Map

- `InvitationSetupPage.vue`: Guest route composition surface for invitation
  token extraction, setup form, invalid-token state, and sign-in recovery.
- `PasswordResetRequestPage.vue`: Guest route composition surface that submits
  email only and renders neutral confirmation.
- `PasswordResetCompletionPage.vue`: Guest route composition surface for reset
  token extraction, password form, invalid-token state, and sign-in recovery.
- `PasswordSetupForm.vue`: Password entry and validation presentation for
  invitation setup; emits submit payload without owning service calls.
- `PasswordResetCompletionForm.vue`: Password entry and validation presentation
  for reset completion; emits submit payload without owning service calls.
- `AccountLifecycleTokenState.vue`: Shared invalid-token, expired/reused,
  conflict, temporary failure, and recovery action display for token flows.
- `AccountLifecycleSuccessState.vue`: Shared success feedback for setup/reset
  completion and return-to-sign-in guidance.
- `UserInvitationPanel.vue`: Admin user create/detail panel for invitation
  creation status and blocked admin resend state.
- `AccountLockPanel.vue`: Admin user detail panel for approved lock state
  display without exposing hidden account or tenant details.
- `AccountLifecycleActions.vue`: Admin user detail actions for lock, unlock,
  recovery, and reactivation eligibility from approved permissions or
  capability flags.
- `AdminAccountLifecycleDialog.vue`: Shared confirmation dialog for lock
  reason, optional recovery/reactivation reason, no-reason unlock, validation
  summary, and pending state.
- Route pages: Compose feature components with composables and services; pages
  contain no direct HTTP logic.

## Permission and Capability Gate

| Surface | Required confirmation before enabling |
|---------|---------------------------------------|
| Invitation creation | Existing user create/detail route access plus approved platform or school account lifecycle permission/capability confirmed before enabling |
| Admin invitation resend | Blocked until OpenAPI provides non-secret resend by invitation or user identifier |
| Account lock review | Approved platform or school account lifecycle permission/capability |
| Lock account | Approved platform or school account lifecycle permission/capability and eligible target account state |
| Unlock account | Approved platform or school account lifecycle permission/capability and eligible lock state |
| Recover account | Approved platform or school account lifecycle permission/capability and eligible recovery state |
| Reactivate account | Approved platform or school account lifecycle permission/capability and eligible inactive account state |
| Invitation setup | Token-proven guest flow; no authenticated permission check |
| Password reset request | Guest flow; email only; no school selector |
| Password reset completion | Token-proven guest flow; no authenticated permission check |

Backend authorization remains authoritative. Client-side visibility improves
usability only after implementation confirms approved permission codes or
session capability flags.

## Phase 0: Research

Research output is captured in [research.md](research.md). No unresolved
technical clarifications remain.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/account-lifecycle-ui-contract.md](contracts/account-lifecycle-ui-contract.md)
- [quickstart.md](quickstart.md)

## Post-Design Constitution Check

- PASS: Design preserves API-first consumption, names every consumed account
  lifecycle operation, and records blocked admin resend until a non-secret
  contract exists.
- PASS: Repository ownership remains explicit; no backend implementation or
  OpenAPI mutation is hidden in frontend work.
- PASS: Pages/components remain presentation and composition boundaries;
  services own transport; composables own token, form, lifecycle action,
  stale-response, and tenant coordination; existing Pinia stores own shared
  session only.
- PASS: Tenant behavior remains explicit: school-scoped admin operations use
  authenticated active school context; public reset request submits email only;
  token-proven guest flows do not infer tenant scope.
- PASS: Authentication, authorization, invalid-token, neutral reset,
  validation, forbidden, tenant-mismatch, not-found, conflict, and temporary
  failure outcomes map to contract-safe frontend states.
- PASS: Future verification expectations are captured for Vitest and optional
  OpenAPI validation if contracts change.
- PASS: No complexity exception is introduced.

## Complexity Tracking

No constitution violations.
