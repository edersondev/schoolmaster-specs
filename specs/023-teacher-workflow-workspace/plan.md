# Implementation Plan: Teacher Workflow Workspace

**Branch**: `023-teacher-workflow-workspace` | **Date**: 2026-07-01 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/023-teacher-workflow-workspace/spec.md`

## Summary

Implement roadmap item 8 in `schoolmaster-frontend`: teacher-facing workspace
and limited same-school admin-observed surfaces for teacher content,
questionnaires, learning sets, grades, attendance, correction history,
authorized content downloads, and administrator-only grade/attendance imports.

The implementation extends completed protected shell, admin CRUD foundation,
lifecycle UI, account lifecycle, and student enrollment/roster UI patterns.
Route pages stay thin. Services own Axios/OpenAPI mapping. Composables
coordinate active school, current academic-period default, active teacher roster
scope, route/query period and roster selection, list pagination, form drafts,
upload/download feedback, lifecycle dialogs, correction reasons, JSON import
drafts, stale-response protection, and no-sensitive-data diagnostics.

Learning-set create and scoped learning-set, grade, and attendance lists stay
blocked until OpenAPI adds or confirms roster-aware learning-set create behavior
and period or roster list filters. Student questionnaire response review,
teacher grading of questionnaire responses, student self-service, guardian
self-service, reporting, platform support, CSV/spreadsheet imports, file-upload
imports, and undocumented APIs stay blocked.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript out of scope for this feature.
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, approved OpenAPI-backed teacher content/questionnaire/learning-set/grade/attendance/teacher-assignment/class-section/academic-period operations.
**Storage**: No backend database change. Route query, route-local state, composable-local state, and Pinia session/shell context only; sensitive details are not persisted.
**Testing**: Vitest, Vue Test Utils, service/composable tests, route/component integration tests, and Redocly only if OpenAPI contract files change.
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming `/api/v1`.
**Project Type**: Frontend SPA feature with specification and delivery artifacts.
**Performance Goals**: Mocked service responses render within 1.5s; route transitions settle within 2s; form submissions settle within 2s after service resolution; stale responses are ignored or cancelled.
**Constraints**: No undocumented endpoints, filters, sort keys, include expansions, page sizes, or fields. No direct Axios calls outside services. No response review/grading. No CSV, spreadsheet, archive, or file-upload import UI. No broad client-side filtering for missing scoped lists. No legacy direct-student learning-set create workaround. No tenant inference. No private file path or presigned URL persistence. No sensitive persisted diagnostics. WCAG 2.1 AA at 390px, 768px, and 1440px. PascalCase Element Plus components. Centralized display text.
**Scale/Scope**: Four user-story slices, protected teacher routes, limited admin routes, service families, contract mappers, route/query period and roster selection, shared status/denial/conflict/upload/correction/import/confirmation components, and focused tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Learning-set create and scoped learning-set,
  grade, and attendance list controls are blocked until OpenAPI documents
  roster-aware create behavior and period or roster filters.
- PASS: Backend, frontend, and specification repository impacts are separated.
  This plan changes specification artifacts first; frontend follows in
  `schoolmaster-frontend`; backend work is separate if contract gaps remain.
- PASS: No Laravel implementation is part of this feature. If backend work is
  later approved, it must follow Service Layer, Requests, Policies, Resources,
  UUID public identifiers, and DTO/Repository rules.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind
  CSS, Element Plus, Axios service modules, feature modules, and
  service-isolated API access.
- PASS: Tenant scoping, active school context, same-school administrator
  observation, cross-tenant denial, and soft-delete/lifecycle expectations are
  documented in the spec and design artifacts.
- PASS: API compatibility, authentication/authorization impact, success states,
  validation, denial, conflict, import, download, unsupported filter, and stale
  response expectations are documented.
- PASS: Vitest/service/composable/component coverage and OpenAPI validation
  expectations are defined for changed critical frontend flows and any contract
  updates.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/023-teacher-workflow-workspace/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── teacher-workflow-workspace-ui-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
└── specs/023-teacher-workflow-workspace/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
        └── teacher-workflow-workspace-ui-contract.md

# Frontend repository target layout
schoolmaster-frontend/
├── src/
│   ├── modules/
│   │   └── teacher-workflow/
│   │       ├── components/
│   │       ├── composables/
│   │       ├── routes/
│   │       ├── services/
│   │       └── types/
│   ├── router/
│   ├── services/
│   ├── stores/
│   └── tests/
│       └── teacher-workflow/
└── tests/
```

**Structure Decision**: This feature is specified in `schoolmaster-specs` and
implemented later as an additive `teacher-workflow` frontend module. Backend
source changes are not planned in this UI feature unless a separate OpenAPI or
backend feature closes the identified learning-set and scoped-list contract
gaps.

## Component Map

- Teacher workspace shell: active school, current academic period, selected
  roster, route/query sync, empty/denied/unavailable states, and teacher
  assignment loading.
- Content screens: list, detail, upload form, status/lifecycle actions,
  scan-gated download action, validation, conflict, and safe denial feedback.
- Questionnaire screens: list, detail, create/update form, supported question
  authoring, lifecycle actions, validation, usage conflict, and response/grading
  omission.
- Learning-set screens: list/detail/read-only legacy audience, lifecycle and
  dependency display, plus blocked create/scoped-list states until OpenAPI
  supports roster-aware create and period/roster filters.
- Grade and attendance screens: blocked scoped-list states until OpenAPI
  supports period/roster filters; create/detail/correction/history flows only
  where approved contracts and role authority exist.
- Admin-observed screens: same-school read/detail observation, admin-only JSON
  imports, and closed-period corrections where approved; no teacher-owned
  management takeover unless a contract explicitly requires it.
- Shared components: status badges, denial banners, conflict summaries,
  validation summaries, stale-response guards, correction reason dialog, JSON
  import editor/summary, confirmation dialogs, empty states, pagination, and
  safe diagnostics.

## Permission and Capability Gates

- All screens require authenticated access and active permitted school context.
- Teacher screens require teacher workflow authority plus ownership or approved
  active teacher assignment scope.
- Admin-observed screens require same-school school-administrator authority and
  expose read/detail observation, admin-only imports, and closed-period
  corrections only where approved.
- Learning-set create is unavailable until OpenAPI documents roster-aware create
  request behavior.
- Scoped learning-set, grade, and attendance lists are unavailable until OpenAPI
  documents period or roster filters. The frontend must not send undocumented
  filters or load broad lists for client-side filtering.
- Import routes are unavailable to teachers, students, guardians, platform users
  without school authority, and unauthorized administrators.
- Downloads are available only for clean, authorized content through approved
  download operations; private file paths never enter UI state or diagnostics.

## Phase 0: Research

Output: [research.md](research.md)

Research decisions cover:

- Consuming approved teacher workflow OpenAPI contracts and gating missing
  scope support through contract review.
- Reusing existing admin list/detail/form/status patterns with teacher-specific
  permission gates.
- Defaulting teacher workspace scope to the current active academic period and
  active teacher rosters, with explicit selection in route query.
- Keeping questionnaire response review/grading outside this slice.
- Keeping admin-observed screens to read/detail observation plus admin-only
  imports and closed-period corrections.
- Using structured JSON import payload entry only.
- Centralizing stale-response and sensitive diagnostics rules.

## Phase 1: Design and Contracts

Outputs:

- [data-model.md](data-model.md)
- [contracts/teacher-workflow-workspace-ui-contract.md](contracts/teacher-workflow-workspace-ui-contract.md)
- [quickstart.md](quickstart.md)

Design decisions cover frontend view models, route/composable/service
boundaries, OpenAPI consumption rules, blocked contract-gap controls,
permission/capability gates, upload/download behavior, lifecycle behavior,
correction history, import all-or-nothing feedback, empty states, denial states,
conflicts, and implementation verification.

## Post-Design Constitution Check

- PASS: OpenAPI contract gaps are explicit and block runtime controls rather
  than allowing undocumented requests.
- PASS: Specification, frontend, and possible backend sequencing remains clear.
- PASS: Vue 3 Composition API, service-isolated Axios, Pinia/router state, and
  existing frontend patterns are the implementation baseline.
- PASS: Tenant isolation, active school context, role/capability checks,
  lifecycle/soft-delete display, and denial behavior are documented.
- PASS: Test expectations cover successful flows, validation, authorization,
  tenant denials, upload/scan/download states, lifecycle conflicts,
  dependency conflicts, corrections, imports, empty states, pagination, stale
  responses, and no-sensitive-data diagnostics.
- PASS: No constitution deviation is introduced.

## Complexity Tracking

No constitution violations.
