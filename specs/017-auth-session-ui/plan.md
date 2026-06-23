# Implementation Plan: Authentication and Session Foundation UI

**Branch**: `017-auth-session-ui` | **Date**: 2026-06-23 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/017-auth-session-ui/spec.md`

## Summary

Define the frontend authentication and session foundation for
`schoolmaster-frontend`: sign-in, forgot-password entry, authenticated session
bootstrap, current-user hydration, permission loading, active school context
resolution, user-authorized school selection when required, protected-route
preservation, layout selection, and contract-safe denial states for expired
session, unauthorized, forbidden, inactive-user, inactive-school, and
tenant-mismatch outcomes.

The plan consumes existing OpenAPI authentication and account-lifecycle
contracts and blocks school-selection implementation unless an approved
OpenAPI operation returns only schools authorized for the current user. It does
not approve new backend behavior. Frontend implementation must use
service-isolated API access, Pinia session state, Vue Router guards, and the
completed System Administrator shell as the protected admin layout target.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains out of scope.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, existing OpenAPI-backed auth/account lifecycle contracts, and an approved OpenAPI school-selection source if school selection is implemented.  
**Storage**: No backend or database storage change. Frontend session state uses Pinia; any client persistence must follow the approved authentication contract and security guidance, and non-sensitive tenant metadata may be persisted only for last approved active school restoration.  
**Testing**: Specification review in `schoolmaster-specs`; future `schoolmaster-frontend` implementation should use Vitest for auth services, session store, route guards, layout selection, tenant context restoration, denied-state mapping, and password reset request entry. Redocly/OpenAPI validation remains required if the contract is edited later.  
**Target Platform**: `schoolmaster-frontend` Vue 3 SPA consuming published `/api/v1` contracts from `schoolmaster-specs`.  
**Project Type**: Frontend SPA feature specification and cross-repository planning artifact.  
**Performance Goals**: Sign-in and valid session bootstrap should complete within the spec target of 30 seconds for at least 95% of valid active-account tests; password reset request entry should show neutral confirmation within 15 seconds for at least 90% of valid email submissions; protected content must remain hidden for 100% of bootstrap tests until current user, permissions, and tenant context are confirmed.  
**Constraints**: No undocumented API consumption; no direct Axios usage in components, pages, layouts, or route guards; no backend implementation in this slice; no account setup/reset completion, invitations, reactivation, account lock management, or broader account lifecycle screens; no client-side tenant inference; preserve originally requested protected routes only when the renewed/authenticated session remains authorized; block school-selection UI until an approved OpenAPI operation returns only schools authorized for the current user.  
**Scale/Scope**: Applies to unauthenticated auth entry pages, session bootstrap, current-user/role/permission hydration, active school context restoration or selection gating, route guards, denied-state screens, and layout selection for protected frontend surfaces.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. This slice consumes existing documented
  `/api/v1/auth/login`, `/api/v1/auth/me`, `/api/v1/auth/logout`, and
  `/api/v1/auth/password-reset-requests` behavior. School selection requires a
  confirmed operation that returns only schools authorized for the current user.
  Any discovered contract gap must be fixed in OpenAPI before frontend
  consumption.
- PASS: Repository impacts are separated. `schoolmaster-specs` leads this plan,
  `schoolmaster-frontend` consumes it for implementation, and
  `schoolmaster-backend` remains unchanged unless a later contract review finds
  a gap.
- PASS: Backend architecture requirements are N/A for the planned frontend
  slice because no Laravel code, services, requests, policies, resources,
  models, public identifiers, routes, or MySQL schema changes are approved.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind
  CSS, Axios-based service modules, feature-aligned organization, and
  service-isolated API access.
- PASS: MySQL, tenant-scoping, cross-tenant access, and soft-delete impacts are
  documented as unchanged. Active school context must come from approved auth
  responses, approved persisted session metadata, or explicit user selection.
- PASS: API compatibility, authentication or authorization impact, and success
  or error response expectations are documented through existing OpenAPI
  contract consumption and frontend error-state mapping.
- PASS: Verification is scoped to specification review now, with future Vitest
  coverage for auth services, stores, guards, tenant context, denied states, and
  password reset request entry. Redocly validation is required only if OpenAPI
  changes are made later.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/017-auth-session-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── auth-session-ui-contract.md
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
│   ├── paths/auth/
│   └── components/
├── docs/
│   ├── frontend-architecture.md
│   ├── frontend-guidelines.md
│   └── frontend-feature-roadmap.md
├── specs/
│   └── 017-auth-session-ui/
│       ├── spec.md
│       ├── plan.md
│       ├── research.md
│       ├── data-model.md
│       ├── quickstart.md
│       └── contracts/
│           └── auth-session-ui-contract.md
└── AGENTS.md

# Frontend repository target shape consumed later by schoolmaster-frontend
schoolmaster-frontend/
├── src/
│   ├── pages/
│   │   └── auth/
│   │       ├── LoginPage.vue
│   │       ├── ForgotPasswordPage.vue
│   │       └── SchoolSelectionPage.vue
│   ├── components/
│   │   └── auth/
│   │       └── AuthFeedbackState.vue
│   ├── composables/
│   │   └── auth/
│   ├── contracts/
│   │   └── auth/
│   ├── layouts/
│   │   └── auth/
│   ├── locales/
│   │   └── auth.js
│   ├── router/
│   │   ├── index.js
│   │   └── modules/
│   │       └── auth.routes.js
│   ├── services/
│   │   └── auth/
│   └── stores/
│       └── auth/
└── tests/
    └── unit/
```

**Structure Decision**: This planning slice changes specification artifacts in
`schoolmaster-specs` and defines target frontend files for later
`schoolmaster-frontend` implementation. It does not change backend code or
approve new API behavior. Frontend auth implementation must stay inside
auth/session boundaries and coordinate with the existing admin shell through
route metadata and session state instead of coupling the shell to auth pages.

## Phase 0: Research

Research output is captured in [research.md](research.md). All technical
context items are resolved.

## Phase 1: Design and Contracts

Design outputs are captured in:

- [data-model.md](data-model.md)
- [contracts/auth-session-ui-contract.md](contracts/auth-session-ui-contract.md)
- [quickstart.md](quickstart.md)

## Post-Design Constitution Check

- PASS: The design artifacts preserve API-first behavior, identify the existing
  OpenAPI operations consumed by the frontend, and block school-selection
  implementation until a user-authorized school-selection source is confirmed
  or added to OpenAPI.
- PASS: Repository ownership remains clear: specs define the plan and contract,
  frontend implements later, backend changes only if a future OpenAPI review
  discovers a gap.
- PASS: Frontend state, guards, pages, services, and contract mapping stay
  separated by responsibility and preserve service-isolated HTTP access.
- PASS: Tenant context behavior is explicit: restore last approved active
  school only if authorized; otherwise require school selection before
  tenant-owned content renders.
- PASS: Authentication, authorization, session expiration, invalid credentials,
  lockout, inactive-user, inactive-school, and tenant-mismatch outcomes map to
  contract-safe frontend states.
- PASS: Future verification expectations are captured for Vitest and optional
  OpenAPI validation if contracts change.
- PASS: No complexity exception is introduced.

## Complexity Tracking

> **Fill ONLY if Constitution Check has violations that must be justified**

No constitution violations.
