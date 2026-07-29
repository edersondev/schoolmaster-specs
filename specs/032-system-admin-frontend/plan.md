# Implementation Plan: System Administrator Frontend Access

**Branch**: `032-system-admin-frontend` | **Date**: 2026-07-14 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/032-system-admin-frontend/spec.md`

## Summary

Adopt the completed System Administrator backend authorization rule in the Vue
SPA. A resolved active platform role named `System Administrator` satisfies
frontend feature-permission checks for released non-identity-owned routes,
navigation, and actions. Centralized route and visibility evaluation retains
authentication, session, active-school, selected-subject, release, approval,
and safety checks. School-owned state is cleared on context changes, and the
existing backend error and audit contracts remain authoritative.

## Technical Context

**Language/Version**: JavaScript; Vue 3 SPA with Composition API and `<script setup>`  
**Primary Dependencies**: Vue Router, Pinia, Axios service modules, Element Plus, Vue I18n, Tailwind CSS  
**Storage**: Existing in-memory and approved persisted session and active-school context only; no new backend persistence  
**Testing**: Vitest unit coverage for router guards, authorization composables, navigation/action visibility, school/subject context, and error-state mapping; production build  
**Target Platform**: Modern desktop and mobile browsers consuming the existing `/api/v1` backend  
**Project Type**: Web application frontend  
**Performance Goals**: Protected-route evaluation completes within the existing session-bootstrap flow and never renders protected school-owned data before required context resolves  
**Constraints**: No backend endpoint, OpenAPI, response schema, or client-side audit change; client-side checks are non-authoritative; all school-owned UI state must clear before a new school context loads  
**Scale/Scope**: All released non-identity-owned protected route groups, navigation destinations, and actions in `schoolmaster-frontend`; existing identity-owned student and guardian self-service boundaries remain unchanged

## Constitution Check

- PASS: OpenAPI impact is identified. This frontend-only feature consumes the
  completed published contract and introduces no API change.
- PASS: Repository ownership is explicit. `schoolmaster-frontend` implements
  the behavior; `schoolmaster-specs` owns this plan and contract; the backend
  is a completed prerequisite.
- N/A: No Laravel controller, service, request, policy, resource, DTO, or
  repository change is required.
- PASS: The frontend design uses Vue 3 Composition API, Vue Router, Pinia,
  Tailwind CSS, Axios-based service modules, feature-aligned modules, and no
  direct component HTTP access.
- PASS: Tenant rules remain explicit. School-owned screens wait for an active
  selected school, clear stale state on switch, and do not present cross-school
  data except on already documented platform-wide surfaces.
- PASS: Authentication, authorization, and error compatibility are retained.
  The existing session role collection and error envelopes are consumed without
  response changes.
- PASS: Vitest coverage covers every changed critical frontend flow. PHPUnit
  and OpenAPI verification remain covered by completed feature 031 backend and
  contract work; no additional backend or API behavior changes occur here.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/032-system-admin-frontend/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── frontend-system-admin-access-contract.md
└── tasks.md                     # Created by /speckit-tasks
```

### Source Code (target repositories)

```text
schoolmaster-frontend/
├── src/
│   ├── router/
│   │   ├── authGuards.js
│   │   ├── adminFallbackRoute.js
│   │   └── modules/
│   ├── stores/
│   │   └── auth/sessionStore.js
│   ├── composables/
│   │   ├── admin-system/useAdminShellPermissions.js
│   │   ├── admin-system/useAdminQuickActions.js
│   │   ├── auth/useSchoolContextSwitch.js
│   │   ├── student/useStudentWorkspaceContext.js
│   │   └── guardian/useGuardianWorkspaceContext.js
│   ├── contracts/
│   │   └── auth/authSession.contract.js
│   ├── services/
│   │   ├── auth/authErrorMapper.js
│   │   ├── admin-system/administration-error-mapper.js
│   │   ├── student/studentSelfServiceFeedbackMapper.js
│   │   └── guardian/guardianSelfServiceFeedbackMapper.js
│   └── pages/
└── tests/unit/system-admin-master/
    ├── router/
    ├── navigation/
    ├── schoolContext/
    └── states/
```

**Structure Decision**: Extend existing shared auth, router, navigation,
school-context, and error boundaries rather than adding per-page permission
logic. Tests are grouped under the feature identifier to prove the global
override and retained non-permission gates.

## Implementation Approach

### Phase 0: Inventory and Central Decision

- Inventory released non-identity-owned route metadata, navigation items, and
  actions that consume feature permission requirements.
- Confirm the existing session role collection exposes role name and platform
  scope needed to identify the active System Administrator role.
- Add one reusable master-access predicate and use it from shared permission,
  route-guard, and navigation/action visibility boundaries.
- Keep identity-owned student and guardian self-service routes outside the
  override; their established actor-owned and guardian-link rules continue to
  govern visibility and direct navigation.

### Phase 1: Route, Navigation, and Context Adoption

- Apply the shared predicate to direct route navigation after session
  resolution, avoiding a temporary limited-role denial while roles load.
- Apply the same decision to released navigation destinations and actions.
- Preserve school-context guards and block school-owned loading until an active
  selected school exists.
- Clear school-owned store and view state before loading against a newly
  selected school context.
- Preserve selected-subject guard behavior and non-permission feedback states.

### Phase 2: Verification and Delivery Evidence

- Add focused Vitest coverage for master route access, limited-role denial,
  navigation/action visibility, session-resolution ordering, active-school
  selection, stale-data clearing, subject-context restriction, and
  prerequisite-specific feedback.
- Run the focused suite and the frontend build, then record results in the
  feature quickstart.
- Confirm no frontend component calls Axios directly and no frontend change
  adds a backend endpoint, response field, or audit record.

## Post-Design Constitution Check

- PASS: Research, data model, UI contract, and quickstart use the existing
  frontend architecture and published backend contract.
- PASS: Master access is centralized and client-side only for route/navigation
  experience; backend authorization remains authoritative.
- PASS: The design retains tenant isolation, identity-owned self-service
  boundaries, and all non-permission prerequisite states.
- PASS: The verification plan supplies focused Vitest coverage and build
  validation without implying a backend or OpenAPI change.

## Complexity Tracking

No constitution violations or exceptional complexity are required.
