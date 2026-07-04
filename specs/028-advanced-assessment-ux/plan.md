# Implementation Plan: Advanced Assessment Frontend UX

**Branch**: `028-advanced-assessment-ux` | **Date**: 2026-07-04 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/028-advanced-assessment-ux/spec.md`

## Summary

Prepare roadmap item 13 for `schoolmaster-frontend`: advanced assessment
frontend surfaces for authorized questionnaire authoring, student
long-text/file-response submission, teacher or school-administrator response
review and manual grading, student result visibility, and approved reporting
summary exposure.

The implementation consumes only approved advanced assessment contracts from
`014-advanced-assessment-content` and the aggregate OpenAPI. Route views stay
thin. Services own Axios/OpenAPI mapping. Composables coordinate active school
context, permission gates, questionnaire authoring state, local-only text
drafts, final-submit file selection, response submission, file scan
visibility, download-only clean file access, grading state, failed-scan
zero/exempt actions, student result state, reporting aggregate fields,
stale-response protection, and safe diagnostics.

Backend behavior, unsupported advanced question types, backend draft attempts,
staged file uploads, inline file preview, resubmission, guardian visibility,
raw answer reporting, generated report file packaging, platform/support detail
access, messaging, notifications, billing, permanent purge, legal hold,
anonymization, and undocumented APIs remain blocked unless a future
specification and OpenAPI contract approve them.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains outside the current frontend baseline.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, approved OpenAPI-backed questionnaire create/update/detail, student questionnaire response submission/detail, teacher/admin response list/detail, grading, clean file download, report catalog/report definition/report request behavior, authentication, current-user, permission, active-school, and session-context behavior.  
**Storage**: No backend database change. Route-local state, composable-local state, local-only unsubmitted text drafts, selected final-submit file references, loaded questionnaire/response/grading/result/report view models, stale request keys, and Pinia session/shell context only; backend draft responses, staged file uploads, raw answer files, private storage paths, hidden answer keys, private grading notes, scanner internals, credentials, full payloads, and unauthorized identifiers are not persisted.  
**Testing**: Vitest, Vue Test Utils, service/composable tests, route/component integration tests, upload validation tests, stale-response tests, safe-diagnostics tests, and Redocly contract validation if OpenAPI files change.  
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming `/api/v1`.  
**Project Type**: Frontend SPA feature with specification and delivery artifacts.  
**Performance Goals**: Mocked service responses render authoring, student response, review queue, grading, student result, and reporting states within 1.5s; protected route transitions settle within 2s after session context resolves; final response submission shows submitted or validation state within 2s after service resolution; grading actions settle within 2s after service resolution; stale responses are ignored or cancelled.  
**Constraints**: No undocumented endpoints, filters, sort keys, include expansions, fields, denial details, question types, answer schemas, grading states, scan states, report fields, or inferred backend state. No direct Axios calls outside services. No client-side tenant inference. No staged file uploads or backend draft response persistence. No inline file preview. Clean answer files are download-only. No resubmission, reopened attempts, guardian advanced assessment visibility, raw answer reporting, file links in reports, generated report file packaging, platform/support detail access, messaging, notifications, billing, purge, legal hold, anonymization, or undocumented controls. WCAG 2.1 AA at 390px, 768px, and 1440px. PascalCase Element Plus components. Centralized display text.  
**Scale/Scope**: Four user-story slices, advanced assessment route module, advanced assessment service family, contract mappers, questionnaire authoring, student response submission, local-only text drafts, final-submit files, review queue, manual grading, failed-scan zero/exempt actions, download-only clean files, student results, reporting summaries, empty/denied/unavailable/conflict/scan/stale states, and focused tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Advanced assessment frontend
  consumption is blocked until the required operations and schemas are present
  in `api/openapi.yaml` and the platform contract mirror.
- PASS: Backend, frontend, and specification repository impacts are separated.
  This plan changes specification artifacts first; frontend follows in
  `schoolmaster-frontend`; backend/API work is separate if contract gaps
  remain.
- PASS: No Laravel implementation is part of this feature. If backend work is
  later approved, it must follow Service Layer, Requests, Policies, Resources,
  UUID public identifiers, and DTO/Repository rules.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router,
  Tailwind CSS, Element Plus, Axios service modules, feature folders, focused
  components, props-down/events-up data flow, and service-isolated API access.
- PASS: Tenant scoping, active school context, student ownership, teacher
  assignment/admin review authority, private file handling, scan states,
  failed-scan grading outcomes, report aggregate boundaries, cross-tenant
  denial, and soft-delete/unavailable behavior are documented in the spec and
  design artifacts.
- PASS: API compatibility, authentication/authorization impact, success
  states, validation, conflict, scan-pending, scan-failed, unavailable-file,
  denial, not-found, and stale-response expectations are documented for
  consumed operations.
- PASS: Vitest/service/composable/component coverage and OpenAPI validation
  expectations are defined for changed critical frontend flows and any
  contract updates.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/028-advanced-assessment-ux/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── advanced-assessment-ux-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
└── specs/028-advanced-assessment-ux/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
        └── advanced-assessment-ux-contract.md

# Frontend repository target layout
schoolmaster-frontend/
├── src/
│   ├── pages/
│   │   └── assessments/
│   ├── components/
│   │   └── assessments/
│   ├── composables/
│   │   └── assessments/
│   ├── services/
│   │   └── assessments/
│   ├── contracts/
│   │   └── assessments/
│   ├── stores/
│   └── router/
└── tests/
    └── advanced-assessment/
```

**Structure Decision**: This feature is specified in `schoolmaster-specs` and
implemented later as an additive advanced-assessment frontend area under the
existing frontend architecture baseline. Backend source changes are not
planned in this UI feature unless separate OpenAPI or backend work approves
missing advanced assessment contracts.

## Component Map

- Advanced assessment workspace routes: protected route composition, active
  school context, assessment permission gates, denied/unavailable/loading
  state routing, and contract-unavailable handling.
- Questionnaire authoring view: existing question types plus approved
  `long_text` and `file_response` schema controls, answer constraints,
  grading expectations, file rules, lifecycle locks, validation feedback, and
  safe denied states.
- Advanced question editor components: prompt, question type, long-text
  constraints, file-response rules, manual grading metadata, sequence, and
  props-down/events-up field updates.
- Student response view: assigned question rendering, due-date state,
  one-attempt warning, local-only text draft behavior, final-submit file
  selection, submission readiness, validation feedback, and no backend draft
  state.
- File-response selector: one-file rule, allowed category/size display, safe
  filename feedback, no staged upload, and final-submit-only file payload.
- Assessment review queue: authorized same-school response list, filters by
  response/grading/scan state, safe student identity fields, empty states,
  denied states, and stale-response guards.
- Response grading view: long-text answer review, clean file download action,
  manual 0-100 grading, student-visible feedback summary, failed-scan
  zero/exempt controls, conflict handling, and no private note exposure.
- Student result view: own submission status, grading status, score summary,
  student-visible feedback summary, safe file availability metadata, pending
  or unavailable states, and no private grading details.
- Advanced assessment reporting views: report catalog/definition/request/result
  exposure for counts, completion status, grading status, and score summaries
  only.
- Shared advanced assessment components: question-type badges, grading-state
  badges, scan-state badges, due-date callouts, file-rule callouts, validation
  summaries, denied/unavailable/conflict messages, empty states,
  stale-response guards, and safe diagnostics.

## Permission and Capability Gates

- Permission and capability identifiers come from approved current-user,
  permission-definition, and OpenAPI-backed assessment contracts. Frontend
  route meta and access composables must centralize those identifiers and
  render contract-unavailable feedback when a required identifier is absent.
- All screens require authenticated active actor state and active permitted
  school context.
- Questionnaire authoring requires approved same-school teacher or
  school-administrator assessment authoring permission.
- Student response submission requires the authenticated user to be linked to
  the active same-school student profile and assigned learning set.
- Student result view requires the authenticated student to own the response.
- Teacher review and grading require owning or assigned teacher authority, or
  explicit same-school school-administrator assessment review authority.
- Clean answer-file download requires authorized review authority and clean
  scan state; inline file preview remains unavailable.
- Failed-scan file-response answers expose only zero-score and exempt grading
  actions to authorized graders.
- Reporting surfaces require approved reporting/report-definition permission
  and may expose only aggregate assessment fields.
- Guardian, platform support, messaging, notification, billing, public file
  sharing, generated report file packaging, legal hold, anonymization, and
  purge controls are not represented.

## Phase 0: Research

Output: [research.md](research.md)

Research decisions cover:

- Consuming approved advanced assessment OpenAPI contracts without backend
  behavior changes.
- Keeping advanced assessment UX inside school-scoped assessment workflows.
- Modeling questionnaire authoring around approved question schemas and
  lifecycle locks.
- Keeping student unsubmitted text drafts local-only.
- Uploading file-response files only during final assessment submission.
- Blocking inline file preview and using download-only clean file access.
- Representing scan states and failed-scan zero/exempt grading controls.
- Separating review queue, grading view, student result, and reporting
  aggregate surfaces.
- Using route-local composables and minimal Pinia coordination.
- Centralizing stale-response and sensitive diagnostics rules.
- Keeping Vue component boundaries explicit.

## Phase 1: Design and Contracts

Outputs:

- [data-model.md](data-model.md)
- [contracts/advanced-assessment-ux-contract.md](contracts/advanced-assessment-ux-contract.md)
- [quickstart.md](quickstart.md)

Design decisions cover frontend view models, route/composable/service
boundaries, OpenAPI consumption rules, permission/capability gates, local-only
drafts, final-submit file selection, scan-state rendering, download-only clean
file access, failed-scan grading actions, student result visibility, reporting
aggregate boundaries, empty states, denial states, conflict states, stale
responses, and implementation verification.

## Post-Design Constitution Check

- PASS: OpenAPI contract usage is explicit and missing advanced assessment
  operations remain blocked until contract promotion.
- PASS: Specification, frontend, and possible backend/API sequencing remains
  clear.
- PASS: Vue 3 Composition API, focused SFCs, props-down/events-up data flow,
  service-isolated Axios, Pinia/router state, and existing frontend patterns
  are the implementation baseline.
- PASS: Tenant isolation, active school context, student ownership, teacher
  assignment/admin authority, private answer files, scan states, failed-scan
  outcomes, reporting aggregate limits, and denial behavior are documented.
- PASS: Test expectations cover successful flows, authentication,
  authorization, contract unavailability, authoring schemas, lifecycle locks,
  local-only drafts, final-submit files, scan states, failed-scan zero/exempt
  controls, clean file downloads, student results, reporting summaries,
  conflicts, stale responses, and no-sensitive-data diagnostics.
- PASS: No constitution deviation is introduced.

## Complexity Tracking

No constitution violations.
