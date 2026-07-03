# Implementation Plan: Platform Support Access and Cross-School Oversight UI

**Branch**: `027-platform-support-ui` | **Date**: 2026-07-03 | **Spec**: [spec.md](spec.md)
**Input**: Feature specification from `specs/027-platform-support-ui/spec.md`

## Summary

Specify and prepare roadmap item 12 for `schoolmaster-frontend`: a protected
platform support workspace for authorized platform actors to review minimized
cross-school operational summaries, view cross-school reporting health, manage
platform-side support access decisions, see display-only target-school opt-in
state, open redacted read-only diagnostics for one approved target school, and
review minimized support audit activity.

The implementation extends completed authentication/session, protected shell,
administration, reporting, and platform-support backend contract patterns.
Route views stay thin. Services own Axios/OpenAPI mapping. Composables
coordinate platform permissions, platform summary filters, support access
decision state, display-only opt-in state, internal platform approval state,
automatic/manual refresh for visible support decisions and diagnostics,
redacted diagnostics rendering, audit filters, stale-response protection, and
no-sensitive-data diagnostics.

School-admin target-school opt-in create/revoke screens, generated report
downloads, raw report outputs, private file metadata, emergency access,
unrestricted impersonation, unrestricted search, support writes,
school-owned operational mutations, billing, payroll, accounting, messaging,
notifications, live classroom, video conferencing, permanent purge, legal
hold, anonymization, advanced assessment, and undocumented APIs remain blocked
unless a future specification and OpenAPI contract approve them.

## Technical Context

**Language/Version**: JavaScript with Vue 3 SFCs using Composition API and `<script setup>`; TypeScript remains outside the current frontend baseline.  
**Primary Dependencies**: Vue 3, Vue Router, Pinia session/shell stores, Axios service boundary, Element Plus, `@element-plus/icons-vue`, Vue I18n, Tailwind CSS, approved OpenAPI-backed platform school summary, platform reporting overview, support access decision, support approval/revocation, support diagnostics, support audit, authentication, current-user, permission, and session-context behavior. School-admin support opt-in create/revoke operations are not consumed by this UI slice.  
**Storage**: No backend database change. Route-local state, composable-local state, loaded platform summaries, support access decisions, redacted diagnostics, support audit lists, automatic-refresh timers, selected target school, selected support decision, and Pinia session/shell context only; hidden counts, raw report outputs, generated files, private file metadata, private diagnostic payloads, credentials, token values, role internals, full records, full payloads, and unauthorized school-owned identifiers are not persisted.  
**Testing**: Vitest, Vue Test Utils, service/composable tests, route/component integration tests, redaction-focused diagnostics tests, stale-response tests, and Redocly contract validation because platform/support operation promotion may be required before implementation.  
**Target Platform**: `schoolmaster-frontend` responsive SPA consuming `/api/v1`.  
**Project Type**: Frontend SPA feature with specification and delivery artifacts.  
**Performance Goals**: Mocked service responses render platform summary, support decision, diagnostics, and audit states within 1.5s; protected platform route transitions settle within 2s after session context resolves; support decision submissions show requested or pending state within 2s after service resolution; approval/revocation actions settle within 2s after service resolution; visible support decision/diagnostics state refreshes without page reload; stale responses are ignored or cancelled.  
**Constraints**: No undocumented endpoints, filters, sort keys, include expansions, page sizes, fields, denial details, diagnostic categories, support decision states, opt-in mutations, approval states, audit payloads, or inferred backend state. No direct Axios calls outside services. No client-side tenant inference. No school-admin target-school opt-in create/revoke screens. No generated report downloads, raw output previews, private file metadata, emergency access, unrestricted impersonation, unrestricted search, support writes, school-owned operational mutations, billing, messaging, notifications, purge, legal hold, anonymization, advanced assessment, or undocumented controls. Meaningful automatic-refresh support access changes must be visible without disrupting keyboard focus. WCAG 2.1 AA at 390px, 768px, and 1440px. PascalCase Element Plus components. Centralized display text.  
**Scale/Scope**: Four user-story slices, platform support route module, platform support service family, contract mappers, platform operational oversight, cross-school reporting overview, support access decision workspace, platform approval/revocation controls, display-only opt-in state, redacted diagnostics, support audit review, automatic/manual refresh, empty/denied/unavailable/conflict/expired/stale states, and focused tests.

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- PASS: OpenAPI impact is identified. Platform/support frontend consumption is
  blocked until approved operations are present in `specs/api/openapi.yaml` and the
  platform contract mirror. Any missing operation must start from OpenAPI
  before frontend implementation.
- PASS: Backend, frontend, and specification repository impacts are separated.
  This plan changes specification artifacts first; frontend follows in
  `schoolmaster-frontend`; backend/API work is separate if platform/support
  contract gaps remain.
- PASS: No Laravel implementation is part of this feature. If backend work is
  later approved, it must follow Service Layer, Requests, Policies, Resources,
  UUID public identifiers, and DTO/Repository rules.
- PASS: Frontend design uses Vue 3 Composition API, Pinia, Vue Router, Tailwind
  CSS, Element Plus, Axios service modules, feature folders, and
  service-isolated API access.
- PASS: Tenant scoping, platform support exceptions, target-school-bound
  diagnostics, read-only support drill-down, approval gates, display-only
  opt-in state, cross-tenant denial, small-count suppression, redaction, and
  soft-delete detail hiding are documented in the spec and design artifacts.
- PASS: API compatibility, authentication/authorization impact, success states,
  validation, conflict, expired/revoked/stale support states, denial,
  unavailable, not-found, stale-response, suppressed-count, and redacted-field
  expectations are documented for consumed operations.
- PASS: Vitest/service/composable/component coverage and OpenAPI validation
  expectations are defined for changed critical frontend flows and any
  contract updates.
- PASS: No constitution deviation is required.

## Project Structure

### Documentation (this feature)

```text
specs/027-platform-support-ui/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── platform-support-ui-contract.md
└── tasks.md
```

### Source Code (target repositories)

```text
# Specification repository
schoolmaster-specs/
└── specs/027-platform-support-ui/
    ├── spec.md
    ├── plan.md
    ├── research.md
    ├── data-model.md
    ├── quickstart.md
    └── contracts/
        └── platform-support-ui-contract.md

# Frontend repository target layout
schoolmaster-frontend/
├── src/
│   ├── pages/
│   │   └── platform-support/
│   ├── components/
│   │   └── platform-support/
│   ├── composables/
│   │   └── platform-support/
│   ├── services/
│   │   └── platform-support/
│   ├── contracts/
│   │   └── platform-support/
│   ├── stores/
│   └── router/
└── tests/
    └── platform-support/
```

**Structure Decision**: This feature is specified in `schoolmaster-specs` and
implemented later as an additive platform-support frontend area under the
existing frontend architecture baseline. Backend source changes are not
planned in this UI feature unless separate OpenAPI or backend work approves
missing platform/support contracts.

## Component Map

- Platform support workspace shell: protected platform route composition,
  platform permission gates, Platform Operational Oversight as the default
  root surface, denied/unavailable/loading state routing, and no school-scoped
  endpoint fallback.
- Platform operational oversight view: minimized school summaries, reporting
  overview indicators, filters/pagination where approved, suppressed-count
  markers, unavailable summary groups, empty states, and denied states.
- Platform summary list and detail preview: school identity/status, approved
  operational indicators, support diagnostic indicators, reporting-health
  snippets, suppression/redaction badges, and no school-owned detail records.
- Support access decision workspace: decision list/detail, request form,
  target school, reason code, purpose, correlation metadata, requested,
  approved, denied, expired, and revoked states, target-school match, active
  approval context, and 24-hour expiry.
- Platform approval controls: approve/revoke actions for actors with explicit
  platform support approval permission, confirmation state, conflict handling,
  returned decision authority, and no school-owned operational writes.
- Support opt-in state panel: display-only target-school opt-in state,
  revocation state, expiry, target-school match, and no school-admin
  opt-in create/revoke controls.
- Support diagnostics view: redacted read-only diagnostics for one target
  school after both approval gates are valid, suppressed-count and redacted
  field indicators, active decision context, expiry, and blocked unsupported
  action messaging.
- Support audit review: paginated minimized audit events, documented filters,
  actor/action/outcome/correlation/reason/timestamp fields, safe target-school
  attribution, empty/no-filter-results states, and denied states.
- Shared platform support components: status badges, decision-state badges,
  approval-gate summary, expiry callouts, suppression badges, redaction
  callouts, denied/unavailable/conflict messages, stale-response guards,
  refresh controls, empty states, safe diagnostics, and platform navigation
  entries.

## Permission and Capability Gates

- Permission and capability identifiers come from approved current-user,
  permission-definition, and OpenAPI-backed platform/support contracts from
  `013-platform-support-access`. Frontend route meta and access composables
  must centralize those contract-derived identifiers and must render
  contract-unavailable feedback when a required identifier is not promoted,
  undocumented, or missing from the authenticated permission context.
- All screens require authenticated active actor state.
- Platform support workspace root opens Platform Operational Oversight by
  default after authenticated access and operational oversight permission are
  confirmed.
- Platform school summaries require explicit platform-wide operational
  oversight permission.
- Cross-school reporting overview requires explicit platform-wide reporting
  permission.
- Support access decision request and detail surfaces require explicit support
  access permission.
- Support diagnostics require support drill-down permission, target-school
  opt-in state, internal platform approval, target-school match, and both gates
  no older than 24 hours.
- Platform approval and revocation controls require explicit support approval
  or support revocation permission.
- Target-school opt-in state is display-only in this UI slice; dedicated
  school-admin opt-in create/revoke screens remain out of scope.
- Support audit review requires explicit support audit review permission.
- Platform administrator status alone is not sufficient without the documented
  permission.
- Existing school-scoped endpoints are not consumed as support shortcuts.
- Automatic refresh runs only while visible support decisions or diagnostics
  depend on active or pending approval gates, and stops or ignores responses
  when route, selected school, filters, decision, target school,
  authentication, permission, approval, revocation, or opt-in state changes.

## Phase 0: Research

Output: [research.md](research.md)

Research decisions cover:

- Blocking implementation until platform/support OpenAPI operations are
  promoted and contract-compliant.
- Keeping platform support workspace separate from school-scoped
  administration, reporting, teacher, student, and guardian workspaces.
- Opening Platform Operational Oversight as the default root surface.
- Keeping target-school opt-in state display-only in this UI slice.
- Modeling support decisions, approval gates, and 24-hour expiry.
- Auto-refreshing visible support decision/diagnostics state with manual
  refresh.
- Rendering redacted diagnostics and suppressed-count indicators without
  derived disclosure.
- Using route-local composables and minimal Pinia coordination.
- Gating support audit review and audit metadata redaction.
- Centralizing stale-response and sensitive diagnostics rules.
- Keeping Vue component boundaries explicit.

## Phase 1: Design and Contracts

Outputs:

- [data-model.md](data-model.md)
- [contracts/platform-support-ui-contract.md](contracts/platform-support-ui-contract.md)
- [quickstart.md](quickstart.md)

Design decisions cover frontend view models, route/composable/service
boundaries, OpenAPI consumption rules, blocked contract-gap controls,
permission/capability gates, support decision state, display-only opt-in state,
automatic refresh, redacted diagnostics, small-count suppression, audit review,
empty states, denial states, conflict states, stale responses, and
implementation verification.

## Post-Design Constitution Check

- PASS: OpenAPI contract usage is explicit and missing platform/support
  operations remain blocked until contract promotion.
- PASS: Specification, frontend, and possible backend/API sequencing remains
  clear.
- PASS: Vue 3 Composition API, service-isolated Axios, Pinia/router state, and
  existing frontend patterns are the implementation baseline.
- PASS: Tenant isolation, cross-school support exceptions, platform access
  context, support decision state, opt-in state, approval state, diagnostics
  redaction, small-count suppression, audit review, and denial behavior are
  documented.
- PASS: Test expectations cover successful flows, authentication,
  authorization, platform denials, contract unavailability, default Platform
  Operational Oversight root, platform summaries, reporting overview, support
  decisions, approval/revocation, display-only opt-in state, diagnostics,
  auto-refresh, manual refresh, conflicts, stale responses, audit review, and
  no-sensitive-data diagnostics.
- PASS: No constitution deviation is introduced.

## Complexity Tracking

No constitution violations.
