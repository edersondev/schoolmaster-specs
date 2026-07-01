# Implementation Plan: Student Self-Service UI

**Branch**: `024-student-self-service-ui` | **Date**: 2026-07-01 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/024-student-self-service-ui/spec.md`

## Summary

Implement roadmap item 9 in `schoolmaster-frontend`: student-facing protected
workspace screens for Assigned Learning Sets, authorized content downloads,
own grades, own attendance, and a limited academic overview assembled only from
approved student self-service data.

The implementation extends completed authentication/session and protected
shell patterns. Assigned Learning Sets is the default landing surface. Route
views stay thin. Services own Axios/OpenAPI mapping. Composables coordinate
active school, current active academic period, active linked student profile,
pagination, loaded-list-backed detail views, unavailable-content feedback,
stale-response protection, and no-sensitive-data diagnostics.

Manual period switching, student questionnaire response submit/review,
standalone student detail endpoints, report runs, report downloads,
transcripts, GPA calculations, attendance-rate calculations, corrections,
imports, teacher/admin/guardian/platform behavior, and undocumented APIs remain
blocked unless a future specification and OpenAPI contract approve them.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript out of scope for this project baseline.
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, approved OpenAPI-backed student learning-set, student content download, student grade, student attendance, authentication, current-user, permission, active-school, and session-context behavior.
**Storage**: No backend database change. Route-local state, composable-local state, loaded list caches, and Pinia session/shell context only; private file data, token values, cross-tenant details, and unauthorized identifiers are not persisted.
**Testing**: Vitest, Vue Test Utils, service/composable tests, route/component integration tests, and Redocly only if OpenAPI contract files change.
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming `/api/v1`.
**Project Type**: Frontend SPA feature with specification and delivery artifacts.
**Performance Goals**: Mocked service responses render within 1.5s; protected student route transitions settle within 2s after session context resolves; authorized download action starts within 2s after service resolution; stale responses are ignored or cancelled.
**Constraints**: No undocumented endpoints, filters, sort keys, include expansions, page sizes, or fields. No direct Axios calls outside services. No manual academic-period switching. No student questionnaire response submit/review. No standalone detail requests for learning sets, grades, or attendance. No calculated GPA, calculated attendance rate, rankings, or trends. No report-run/report-download/transcript UI. No tenant inference. No private file path, storage key, token, role internals, scan internals, or cross-tenant diagnostic persistence. WCAG 2.1 AA at 390px, 768px, and 1440px. PascalCase Element Plus components. Centralized display text.
**Scale/Scope**: Four user-story slices, student route module, student service family, contract mappers, loaded-list-backed detail views, current-active-period scoped lists, content download action, academic overview counts/statuses, empty/denied/unavailable/no-active-school/no-student-profile/no-current-period/not-found states, and focused tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Existing approved student routes are
  consumed; new student report, transcript, period picker, questionnaire
  response UI, standalone detail, or summary fields require future OpenAPI
  work before implementation.
- PASS: Backend, frontend, and specification repository impacts are separated.
  This plan changes specification artifacts first; frontend follows in
  `schoolmaster-frontend`; backend work is separate if contract gaps remain.
- PASS: No Laravel implementation is part of this feature. If backend work is
  later approved, it must follow Service Layer, Requests, Policies, Resources,
  UUID public identifiers, and DTO/Repository rules.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind
  CSS, Element Plus, Axios service modules, feature folders, and
  service-isolated API access.
- PASS: Tenant scoping, active school context, linked student profile, current
  active academic period, cross-tenant denial, and lifecycle/unavailable
  expectations are documented in the spec and design artifacts.
- PASS: API compatibility, authentication/authorization impact, success states,
  validation, denial, unavailable-content, not-found, and stale-response
  expectations are documented for consumed operations.
- PASS: Vitest/service/composable/component coverage and OpenAPI validation
  expectations are defined for changed critical frontend flows and any
  contract updates.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/024-student-self-service-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── student-self-service-ui-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
└── specs/024-student-self-service-ui/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
        └── student-self-service-ui-contract.md

# Frontend repository target layout
schoolmaster-frontend/
├── src/
│   ├── pages/
│   │   └── student/
│   ├── components/
│   │   └── student/
│   ├── composables/
│   │   └── student/
│   ├── services/
│   │   └── student/
│   ├── contracts/
│   │   └── student/
│   ├── stores/
│   └── router/
└── tests/
    └── student-self-service/
```

**Structure Decision**: This feature is specified in `schoolmaster-specs` and
implemented later as an additive student frontend area under the existing
frontend architecture baseline. Backend source changes are not planned in this
UI feature unless a separate OpenAPI or backend feature approves missing
student-facing contracts.

## Component Map

- Student workspace shell: protected student route composition, active school,
  active linked student profile, current active academic period, default
  Assigned Learning Sets landing, and no-active-school/no-student-profile/
  no-current-period states.
- Assigned Learning Sets list: current-period timeline loading, pagination,
  empty state, published status display, read-only questionnaire entry display,
  and selected learning-set detail from loaded timeline data.
- Learning-set detail: ordered entry list, content availability metadata,
  read-only questionnaire entries, unavailable-content feedback, and no
  standalone detail request.
- Content download action: download availability gate, approved binary
  operation call, safe success/failure feedback, and no private path
  persistence.
- Grades screen: current-period student grade list, pagination, status display,
  read-only loaded-record detail, empty state, not-found for stale detail.
- Attendance screen: current-period student attendance list, pagination, status
  display, read-only loaded-record detail, empty state, not-found for stale
  detail.
- Academic overview: counts/statuses from approved student learning-set,
  grade, attendance, and content availability responses only; no GPA,
  attendance-rate, ranking, trend, report-run, or transcript behavior.
- Shared student components: status badges, student empty states, denial
  banners, unavailable-content callouts, no-active-school, no-student-profile,
  no-current-period, not-found, pagination, stale-response guards, and safe
  diagnostics.

## Permission and Capability Gates

- All screens require authenticated access and active permitted school context.
- All student self-service data requires active same-school linked student
  profile; absence shows no-student-profile and blocks data requests.
- Assigned Learning Sets requires current active academic period; absence shows
  no-current-period and blocks the required timeline request.
- Student workspace root opens Assigned Learning Sets by default only after
  authenticated access, active school, active student profile, and current
  active period are confirmed.
- Content download is enabled only when returned student content metadata says
  `download_available` is true.
- Grade and attendance screens may use current active academic-period filter
  where available, but no manual period switch appears in this slice.
- Learning-set, grade, and attendance detail views use loaded list data only.
  Missing loaded records show not-found or contract-unavailable state rather
  than undocumented detail requests.

## Phase 0: Research

Output: [research.md](research.md)

Research decisions cover:

- Consuming approved student OpenAPI contracts without adding backend behavior.
- Using current active academic period only.
- Showing distinct no-student-profile state.
- Making Assigned Learning Sets the default landing surface.
- Keeping questionnaire entries read-only.
- Limiting academic overview to counts/statuses only.
- Using route-local composables and loaded-list-backed detail surfaces.
- Centralizing stale-response and sensitive diagnostics rules.

## Phase 1: Design and Contracts

Outputs:

- [data-model.md](data-model.md)
- [contracts/student-self-service-ui-contract.md](contracts/student-self-service-ui-contract.md)
- [quickstart.md](quickstart.md)

Design decisions cover frontend view models, route/composable/service
boundaries, OpenAPI consumption rules, blocked contract-gap controls,
permission/capability gates, content download behavior, empty states, denial
states, unavailable states, not-found behavior, stale responses, and
implementation verification.

## Post-Design Constitution Check

- PASS: OpenAPI contract usage is explicit and new student-facing report,
  transcript, period picker, questionnaire response, detail, or summary
  behavior remains blocked until contract approval.
- PASS: Specification, frontend, and possible backend sequencing remains clear.
- PASS: Vue 3 Composition API, service-isolated Axios, Pinia/router state, and
  existing frontend patterns are the implementation baseline.
- PASS: Tenant isolation, active school context, student profile context,
  current active period, lifecycle/unavailable display, and denial behavior are
  documented.
- PASS: Test expectations cover successful flows, authentication,
  authorization, tenant denials, no active school, no student profile, no
  current period, unavailable content, not-found, empty states, pagination,
  stale responses, and no-sensitive-data diagnostics.
- PASS: No constitution deviation is introduced.

## Complexity Tracking

No constitution violations.
