# Implementation Plan: Guardian Self-Service UI

**Branch**: `025-guardian-self-service-ui` | **Date**: 2026-07-02 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/025-guardian-self-service-ui/spec.md`

## Summary

Implement roadmap item 10 in `schoolmaster-frontend`: guardian-facing
protected workspace screens for linked student listing, linked student detail,
summary-only academic views, and approved contact/profile views using only the
approved guardian self-service contracts.

The implementation extends completed authentication/session, protected shell,
administration, teacher, and student frontend patterns. Route views stay thin.
Services own Axios/OpenAPI mapping. Composables coordinate active school,
guardian access, selected student, current active academic period, pagination,
safe not-found non-enumeration, stale-response protection, empty/denied states,
and no-sensitive-data diagnostics.

Manual academic-period switching, guardian writes, guardian profile updates,
association requests, detailed academic rows, teacher content downloads,
questionnaire actions, reports, messaging, notification-center behavior,
school-admin behavior, student self-service behavior, platform support, and
undocumented APIs remain blocked unless a future specification and OpenAPI
contract approve them.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript out of scope for this project baseline.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, approved OpenAPI-backed guardian student list, guardian student detail, guardian academic summary, guardian contact view, authentication, current-user, permission, active-school, and session-context behavior.  
**Storage**: No backend database change. Route-local state, composable-local state, selected student state, loaded list caches, and Pinia session/shell context only; token values, other guardian data, non-primary contacts, school-only notes, unassociated student identifiers, report data, and cross-tenant details are not persisted.  
**Testing**: Vitest, Vue Test Utils, service/composable tests, route/component integration tests, and Redocly only if OpenAPI contract files change.  
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming `/api/v1`.  
**Project Type**: Frontend SPA feature with specification and delivery artifacts.  
**Performance Goals**: Mocked service responses render within 1.5s; protected guardian route transitions settle within 2s after session context resolves; selected-student detail, academic summary, and contact views settle within 2s after service resolution; stale responses are ignored or cancelled.  
**Constraints**: No undocumented endpoints, filters, sort keys, include expansions, page sizes, fields, denial details, or inferred backend state. No direct Axios calls outside services. No manual academic-period switching. No guardian writes. No detailed academic records. No teacher content, questionnaire action, report, messaging, notification-center, school-admin, student self-service, platform support, or undocumented controls. No tenant inference. No unassociated student identifiers, other guardian data, non-primary contacts, school-only notes, correction details, teacher-private data, report data, token values, role internals, or cross-tenant diagnostic persistence. WCAG 2.1 AA at 390px, 768px, and 1440px. PascalCase Element Plus components. Centralized display text.  
**Scale/Scope**: Four user-story slices, guardian route module, guardian service family, contract mappers, linked student list/detail, current-active-period academic summary, contact view, empty/denied/unavailable/no-active-school/no-guardian-link/no-linked-students/no-academic-period/not-found states, and focused tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Existing approved guardian routes are
  consumed; new guardian writes, profile updates, association requests,
  period picker, reports, notifications, messaging, academic detail rows, or
  summary fields require future OpenAPI work before implementation.
- PASS: Backend, frontend, and specification repository impacts are separated.
  This plan changes specification artifacts first; frontend follows in
  `schoolmaster-frontend`; backend work is separate if contract gaps remain.
- PASS: No Laravel implementation is part of this feature. If backend work is
  later approved, it must follow Service Layer, Requests, Policies, Resources,
  UUID public identifiers, and DTO/Repository rules.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind
  CSS, Element Plus, Axios service modules, feature folders, and
  service-isolated API access.
- PASS: Tenant scoping, active school context, guardian-user link proof,
  guardian-student association, current active academic period, cross-tenant
  denial, lifecycle/unavailable expectations, and not-found non-enumeration are
  documented in the spec and design artifacts.
- PASS: API compatibility, authentication/authorization impact, success states,
  validation, denial, unavailable summary, not-found, and stale-response
  expectations are documented for consumed operations.
- PASS: Vitest/service/composable/component coverage and OpenAPI validation
  expectations are defined for changed critical frontend flows and any
  contract updates.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/025-guardian-self-service-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── guardian-self-service-ui-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
└── specs/025-guardian-self-service-ui/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
        └── guardian-self-service-ui-contract.md

# Frontend repository target layout
schoolmaster-frontend/
├── src/
│   ├── pages/
│   │   └── guardian/
│   ├── components/
│   │   └── guardian/
│   ├── composables/
│   │   └── guardian/
│   ├── services/
│   │   └── guardian/
│   ├── contracts/
│   │   └── guardian/
│   ├── stores/
│   └── router/
└── tests/
    └── guardian-self-service/
```

**Structure Decision**: This feature is specified in `schoolmaster-specs` and
implemented later as an additive guardian frontend area under the existing
frontend architecture baseline. Backend source changes are not planned in this
UI feature unless a separate OpenAPI or backend feature approves missing
guardian-facing contracts.

## Component Map

- Guardian workspace shell: protected guardian route composition, active
  school, guardian access state, current active academic period, linked-student
  list landing, and no-active-school/no-guardian-link/no-linked-students/
  no-academic-period states.
- Linked students list: guardian-visible student collection loading,
  pagination, relationship label display, limited student summary, and true
  no-linked-students empty state.
- Linked student detail: approved limited profile and enrollment summary for a
  selected student, safe not-found for direct or stale targets, and no
  restricted profile fields.
- Academic summary screen: current-active-period grade summary, attendance
  summary, learning-set progress/status, unavailable-summary feedback, and no
  detailed rows or report controls.
- Contact view: authenticated guardian-owned contact fields, relationship
  label, student primary school-approved contact details, safe missing-value
  state, and restricted contact redaction.
- Shared guardian components: status badges, summary panels, contact panels,
  guardian empty states, denial banners, no-active-school, no-guardian-link,
  no-linked-students, no-academic-period, unavailable-summary, not-found,
  pagination, stale-response guards, and safe diagnostics.

## Permission and Capability Gates

- All screens require authenticated access and active permitted school context.
- Guardian self-service data requires active same-school guardian-user link and
  active guardian record. Missing guardian-user link shows no-guardian-link
  only when approved session or access behavior safely identifies it; other
  denials follow the documented response envelope without client-side
  inference.
- Student list shows only active same-school students returned by
  `listGuardianStudents`.
- Target-specific student detail, academic summary, and contact views treat
  missing, unassociated, inactive, transferred, deleted, and cross-tenant
  targets as one safe not-found state.
- Academic summary requires the current active same-school academic period; no
  manual period picker or query-driven alternate period source appears in this
  slice.
- Contact view exposes only authenticated guardian contact, relationship label,
  and student primary contact fields returned by contract.
- Guardian screens never expose school-admin, student self-service, teacher,
  platform, report, messaging, notification, lifecycle, correction, import,
  restore, purge, legal, billing, or undocumented controls.

## Phase 0: Research

Output: [research.md](research.md)

Research decisions cover:

- Consuming approved guardian OpenAPI contracts without adding backend
  behavior.
- Keeping guardian workspace separate from student self-service.
- Using current active academic period only.
- Mapping target-specific denial to safe not-found.
- Modeling no-guardian-link, no-linked-students, and unavailable-summary states.
- Using route-local composables and minimal Pinia coordination.
- Centralizing stale-response and sensitive diagnostics rules.
- Keeping Vue component boundaries explicit with service-isolated API access.

## Phase 1: Design and Contracts

Outputs:

- [data-model.md](data-model.md)
- [contracts/guardian-self-service-ui-contract.md](contracts/guardian-self-service-ui-contract.md)
- [quickstart.md](quickstart.md)

Design decisions cover frontend view models, route/composable/service
boundaries, OpenAPI consumption rules, blocked contract-gap controls,
permission/capability gates, empty states, denial states, unavailable states,
not-found non-enumeration, stale responses, and implementation verification.

## Post-Design Constitution Check

- PASS: OpenAPI contract usage is explicit and new guardian-facing write,
  profile update, association request, academic detail, report, period picker,
  notification, messaging, or summary behavior remains blocked until contract
  approval.
- PASS: Specification, frontend, and possible backend sequencing remains clear.
- PASS: Vue 3 Composition API, service-isolated Axios, Pinia/router state, and
  existing frontend patterns are the implementation baseline.
- PASS: Tenant isolation, active school context, guardian access context,
  linked student association, current active period, lifecycle/unavailable
  display, and denial behavior are documented.
- PASS: Test expectations cover successful flows, authentication,
  authorization, tenant denials, no active school, missing guardian link, no
  linked students, no current active period, unavailable summary, not-found
  non-enumeration, empty states, pagination, stale responses, and
  no-sensitive-data diagnostics.
- PASS: No constitution deviation is introduced.

## Complexity Tracking

No constitution violations.
