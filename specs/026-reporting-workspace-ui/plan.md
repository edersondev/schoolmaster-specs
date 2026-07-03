# Implementation Plan: Reporting Workspace UI

**Branch**: `026-reporting-workspace-ui` | **Date**: 2026-07-03 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/026-reporting-workspace-ui/spec.md`

## Summary

Implement roadmap item 11 in `schoolmaster-frontend`: a protected reporting
workspace for authorized school reporting users to browse the approved report
catalog, request built-in and active custom-definition reports, review
asynchronous report history, download available outputs, run approved lifecycle
actions, and maintain custom report definitions using only approved reporting
contracts.

The implementation extends completed authentication/session, protected shell,
administration, teacher, student, and guardian frontend patterns. Route views
stay thin. Services own Axios/OpenAPI mapping. Composables coordinate active
school, permission gates, report catalog state, report request forms, report
history filters, list-backed report detail, automatic refresh for
requested/generating runs, active school timezone formatting, download availability, lifecycle
dialogs, custom definition editing, stale-response protection, polite
state-change announcements, and no-sensitive-data diagnostics.

Platform-wide reporting, manual status mutation, output delete/restore,
retention override, permanent purge, legal hold, anonymization, arbitrary query
text, custom code, uploaded templates, unapproved domains, report sharing,
messaging, notifications, billing, and undocumented APIs remain blocked unless
a future specification and OpenAPI contract approve them.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript out of scope for this project baseline.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, approved OpenAPI-backed report catalog, report list/request/download/retry/cancel/delete/restore, report definition list/detail/create/update/activate/deactivate/delete/restore, authentication, current-user, permission, active-school, and session-context behavior. Report-run detail is list-backed because no standalone report-run detail operation is approved in this UI slice.
**Storage**: No backend database change. Route-local state, composable-local state, loaded report/catalog/definition caches, automatic-refresh timers, selected report/definition state, and Pinia session/shell context only; report contents, binary files, private filter payloads, storage paths, storage keys, token values, role internals, unauthorized identifiers, and cross-tenant details are not persisted.  
**Testing**: Vitest, Vue Test Utils, service/composable tests, route/component integration tests, accessibility-oriented state announcement tests, and Redocly only if OpenAPI contract files change.  
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming `/api/v1`.  
**Project Type**: Frontend SPA feature with specification and delivery artifacts.  
**Performance Goals**: Mocked service responses render within 1.5s; protected reporting route transitions settle within 2s after session context resolves; report request submission shows accepted run state within 2s after service resolution; download action starts within 2s after service resolution; visible requested/generating runs refresh to their latest documented state without page reload; stale responses are ignored or cancelled.  
**Constraints**: No undocumented endpoints, filters, sort keys, include expansions, page sizes, fields, denial details, output formats, lifecycle reasons, or inferred backend state. No standalone report-run detail lookup is allowed without a future approved OpenAPI operation. No direct Axios calls outside services. No client-side tenant inference. No report content preview. No free-text lifecycle reasons. No platform-wide reporting, manual status mutation, output delete/restore, retention override, permanent purge, legal hold, anonymization, arbitrary query text, custom code, uploaded templates, unapproved domains, report sharing, messaging, notifications, billing, or undocumented controls. Active school timezone is used for report and definition timestamps. Meaningful automatic-refresh state changes receive visible updates and polite assistive-technology announcements. WCAG 2.1 AA at 390px, 768px, and 1440px. PascalCase Element Plus components. Centralized display text.
**Scale/Scope**: Five user-story slices, reporting route module, reporting service family, contract mappers, catalog browser, request flow, report history/detail, auto-refresh/manual refresh, output download surfaces, lifecycle actions, custom definition workspace, empty/denied/unavailable/conflict/expired/stale states, active school timezone formatting, polite announcements, and focused tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Existing approved reporting routes are
  consumed; new reporting behavior, formats, lifecycle actions, domains,
  fields, filters, platform-wide views, retention controls, sharing,
  notifications, messaging, or billing require future OpenAPI work before
  implementation.
- PASS: Backend, frontend, and specification repository impacts are separated.
  This plan changes specification artifacts first; frontend follows in
  `schoolmaster-frontend`; backend work is separate if contract gaps remain.
- PASS: No Laravel implementation is part of this feature. If backend work is
  later approved, it must follow Service Layer, Requests, Policies, Resources,
  UUID public identifiers, and DTO/Repository rules.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind
  CSS, Element Plus, Axios service modules, feature folders, and
  service-isolated API access.
- PASS: Tenant scoping, active school context, reporting permissions, report
  catalog visibility, report runs, outputs, custom definitions, lifecycle
  actions, cross-tenant denial, soft-delete visibility, and output retention
  expectations are documented in the spec and design artifacts.
- PASS: API compatibility, authentication/authorization impact, success states,
  validation, conflict, expired-output, denial, unavailable, not-found,
  stale-response, and binary download expectations are documented for consumed
  operations.
- PASS: Vitest/service/composable/component coverage and OpenAPI validation
  expectations are defined for changed critical frontend flows and any
  contract updates.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/026-reporting-workspace-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── reporting-workspace-ui-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
└── specs/026-reporting-workspace-ui/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
        └── reporting-workspace-ui-contract.md

# Frontend repository target layout
schoolmaster-frontend/
├── src/
│   ├── pages/
│   │   └── reporting/
│   ├── components/
│   │   └── reporting/
│   ├── composables/
│   │   └── reporting/
│   ├── services/
│   │   └── reporting/
│   ├── contracts/
│   │   └── reporting/
│   ├── stores/
│   └── router/
└── tests/
    └── reporting-workspace/
```

**Structure Decision**: This feature is specified in `schoolmaster-specs` and
implemented later as an additive reporting frontend area under the existing
frontend architecture baseline. Backend source changes are not planned in this
UI feature unless a separate OpenAPI or backend feature approves missing
reporting contracts.

## Component Map

- Reporting workspace shell: protected reporting route composition, active
  school, reporting permission gates, Report History as the default root
  surface, no-active-school, inactive-school, denied, and
  contract-unavailable states.
- Report catalog browser: approved report domains, fields, filters,
  operators, grouping, sorting, output formats, complexity limits, and
  unavailable-catalog state.
- Report request form: built-in report selection, active custom definition
  selection, same-school filter controls, output format selection, visible
  validation, accepted asynchronous run feedback, and no unsupported options.
- Report history list: paginated report runs, documented filters,
  generation-status rendering, soft-delete visibility, source/custom markers,
  retry lineage summary, empty/no-filter-results states, and manual refresh.
- Report run detail: selected run metadata, active school timezone timestamps,
  output availability by format, retention messaging, retry/cancel/delete/
  restore action surface, and safe denial/conflict/expired states.
- Report output download action: format availability gate, approved binary
  operation call, expired-output feedback, conflict/validation/not-found
  handling, and no private storage persistence.
- Report lifecycle dialogs: retry, cancellation, soft-delete, and restore
  controls with approved reason-code choices, conflict handling, and no
  optimistic state that contradicts returned run state.
- Custom definition workspace: definition list/detail/editor, draft/active/
  inactive/deleted lifecycle display, active metadata-only edit boundary,
  deactivation before structural edits, activation, delete, restore to
  inactive, duplicate-name and complexity-limit feedback.
- Shared reporting components: status badges, output availability badges,
  retention callouts, catalog field selectors, filter builders,
  definition-state badges, empty states, denial banners, conflict messages,
  expired-output callouts, stale-response guards, polite announcement region,
  and safe diagnostics.

## Permission and Capability Gates

- All screens require authenticated access and active permitted school context.
- Reporting workspace root opens Report History by default after authenticated
  access, active school, and report permission are confirmed.
- Report catalog use for report request workflows, report requests, report
  history, and downloads require explicit same-school report permission.
- Report catalog use for custom definition workflows requires explicit
  same-school report-definition permission.
- Retry, cancellation, delete, and restore controls require explicit
  same-school report lifecycle permission and state eligibility from the
  approved contract.
- Custom report definition screens require explicit same-school
  report-definition permission.
- Platform administrator or support roles do not unlock reporting workspace
  controls without explicit same-school reporting permission.
- No reporting request is submitted until catalog, active school, permissions,
  filters, references, and output formats are valid for the selected report.
- Download controls are enabled only for available generated outputs in
  documented formats.
- Automatic refresh runs only while visible report history or list-backed
  detail contains requested or generating runs, and stops or ignores responses
  when route, active school, authentication, filters, selected report, selected
  definition, catalog, or dialog state changes.
- Custom report requests are enabled only for active custom definitions.
- Active custom definitions allow metadata-only edits; structural edits
  require deactivation.

## Phase 0: Research

Output: [research.md](research.md)

Research decisions cover:

- Consuming approved reporting OpenAPI contracts without backend changes.
- Keeping reporting workspace separate from administration, student, guardian,
  teacher, and platform views.
- Modeling report catalog-driven request forms.
- Using route-local composables and minimal Pinia coordination.
- Auto-refreshing visible requested/generating report runs with manual refresh.
- Opening Report History as the workspace root default.
- Rendering timestamps in active school timezone.
- Using polite announcements for meaningful state changes.
- Mapping output availability and expired-output states.
- Gating lifecycle actions and custom definitions by permission and state.
- Centralizing stale-response and sensitive diagnostics rules.

## Phase 1: Design and Contracts

Outputs:

- [data-model.md](data-model.md)
- [contracts/reporting-workspace-ui-contract.md](contracts/reporting-workspace-ui-contract.md)
- [quickstart.md](quickstart.md)

Design decisions cover frontend view models, route/composable/service
boundaries, OpenAPI consumption rules, blocked contract-gap controls,
permission/capability gates, automatic refresh, active school timezone
formatting, polite announcements, output availability, lifecycle actions,
custom definition editing rules, empty states, denial states, conflict states,
expired-output states, stale responses, and implementation verification.

## Post-Design Constitution Check

- PASS: OpenAPI contract usage is explicit and new reporting behavior, output
  formats, fields, filters, lifecycle actions, platform-wide reporting,
  sharing, messaging, notifications, billing, or retention controls remain
  blocked until contract approval.
- PASS: Specification, frontend, and possible backend sequencing remains clear.
- PASS: Vue 3 Composition API, service-isolated Axios, Pinia/router state, and
  existing frontend patterns are the implementation baseline.
- PASS: Tenant isolation, active school context, reporting access context,
  report catalog visibility, report run history, output availability,
  lifecycle eligibility, custom definition lifecycle, soft-delete visibility,
  active school timezone, and denial behavior are documented.
- PASS: Test expectations cover successful flows, authentication,
  authorization, tenant denials, no active school, unavailable catalog, report
  request, default Report History root, report history, auto-refresh, manual
  refresh, output downloads, expired outputs, lifecycle actions, custom
  definitions, conflicts, stale responses, polite announcements, and
  no-sensitive-data diagnostics.
- PASS: No constitution deviation is introduced.

## Complexity Tracking

No constitution violations.
