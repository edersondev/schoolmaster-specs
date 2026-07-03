# Research: Reporting Workspace UI

## Decision: Consume only approved reporting OpenAPI contracts

**Rationale**: The aggregate contract already includes report catalog, report
list/request/download, report retry/cancel/delete/restore, and custom report
definition operations. This UI slice can deliver the workspace without new
backend behavior.

**Alternatives considered**:

- Add frontend-only calls to inferred backend behavior: rejected because
  OpenAPI is the contract source of truth.
- Reuse administration list/report screens for reporting users: rejected
  because reporting has distinct report lifecycle, output, and custom
  definition authority.
- Add missing backend convenience endpoints from the UI plan: rejected until a
  separate specification and OpenAPI update approves them.

## Decision: Keep reporting workspace separate from other actor workspaces

**Rationale**: Reporting access requires explicit same-school report,
report-lifecycle, and report-definition permissions. Separate routes, services,
and display text prevent reporting users from appearing to have
administration, student, guardian, teacher, platform, or support authority.

**Alternatives considered**:

- Add reporting screens inside administration workspace only: rejected because
  authorized reporting users may not be full administrators.
- Add report views into student or guardian self-service: rejected because
  those actors do not receive reporting workspace authority.
- Use one generic analytics workspace: rejected because approved contracts are
  specifically report-run and report-definition oriented.

## Decision: Drive report requests from the approved catalog

**Rationale**: The catalog defines approved domains, fields, filters,
operators, grouping, sorting, output formats, and complexity limits. Using it
as the UI source prevents hardcoded unsupported fields or filters and keeps
custom definitions aligned with backend validation.

**Alternatives considered**:

- Hardcode report fields in the frontend: rejected because catalog changes
  would cause drift.
- Let users submit arbitrary field or filter identifiers: rejected because it
  bypasses contract governance and tenant-safe validation.
- Hide custom definition creation until later: rejected because active custom
  definitions and catalog-driven definition management are approved behavior.

## Decision: Use route-local composables and minimal Pinia coordination

**Rationale**: Reporting screens need selected report, selected definition,
catalog, filters, download, lifecycle, auto-refresh, and stale-response state.
Keeping most state route-local limits persistence of sensitive report metadata
and private filter context. Pinia remains limited to shared session, active
school, shell, navigation, and permission state.

**Alternatives considered**:

- Store all reporting lists, filters, and definitions in Pinia: rejected
  because it persists more report context than needed across routes.
- Put HTTP and mapping logic in route components: rejected by service
  isolation and component-boundary rules.
- Put eligibility rules entirely in templates: rejected because state
  derivation belongs in composables or mapped view models.

## Decision: Auto-refresh visible requested/generating runs and provide manual refresh

**Rationale**: Reporting is asynchronous. Auto-refresh keeps visible pending
runs current without requiring page reloads, while manual refresh gives users
explicit control. Refresh is limited to visible history/detail when runs are
`requested` or `generating` to avoid unnecessary background polling.

**Alternatives considered**:

- Manual refresh only: rejected because pending/generating states would feel
  stale and weaken acceptance criteria.
- Realtime push updates: rejected because no approved realtime contract exists
  for reporting state changes.
- Refresh only on navigation: rejected because users often wait on the report
  history/detail screen.

## Decision: Open Report History by default

**Rationale**: Reporting work is asynchronous, so the default workspace surface
should help users see recent requests, state changes, downloads, retries, and
failed or expired runs immediately. Catalog browsing and new report requests
remain one navigation action away.

**Alternatives considered**:

- Open report catalog by default: rejected because it optimizes discovery over
  monitoring, download, and retry workflows.
- Open new report request by default: rejected because it hides existing
  asynchronous work and expired/failed runs.
- Open last-used tab by default: rejected because it makes acceptance tests and
  first-use behavior less predictable.

## Decision: Render report timestamps in active school timezone

**Rationale**: Reports are tenant-owned operational records. Active school
timezone gives consistent generated, expiry, lifecycle, and custom definition
timestamps regardless of browser/device timezone.

**Alternatives considered**:

- Browser/device timezone: rejected because it can show different expiry times
  for users viewing the same school-owned report.
- User profile timezone: rejected because reporting contracts and tenant
  operations are school-scoped.
- UTC only: rejected because it is less usable for school operations and
  retention review.

## Decision: Use visible updates plus polite assistive announcements

**Rationale**: Meaningful state changes from automatic refresh, lifecycle
actions, and download availability changes must be clear to visual and
assistive-technology users without interrupting focus or announcing every
refresh cycle.

**Alternatives considered**:

- Visual-only updates: rejected because automatic status changes can be missed
  by screen-reader users.
- Assertive announcement for every refresh result: rejected because it is noisy
  and can disrupt forms, dialogs, and keyboard workflows.
- No behavior beyond existing shell patterns: rejected because asynchronous
  report state changes are central to this feature.

## Decision: Treat output availability as per-format UI state

**Rationale**: Approved outputs have independent availability states:
`pending`, `available`, `failed`, `expired`, and `unsupported`. Rendering per
format prevents the UI from treating a generated report as wholly available
when only one format is available.

**Alternatives considered**:

- One run-level download status: rejected because output formats can differ.
- Hide unavailable formats entirely: rejected because users need clear pending,
  failed, expired, or unsupported messaging.
- Retry automatically on expired download: rejected because download never
  regenerates outputs; users must request a new run or approved retry.

## Decision: Gate lifecycle controls by permission and state

**Rationale**: Retry, cancellation, delete, and restore mutate report history
or visibility. Controls must be displayed only when the actor has the required
school-scoped permission and the current report state supports the transition.

**Alternatives considered**:

- Show all lifecycle buttons and let the backend reject: rejected because it
  creates confusing workflows and avoidable invalid submissions.
- Hide lifecycle history after delete: rejected because soft-delete affects
  list visibility, not audit meaning or output retention.
- Use free-text lifecycle reasons: rejected because contracts approve only
  predefined tenant-safe reason codes.

## Decision: Keep custom definition editing catalog-bound

**Rationale**: Custom report definitions are safe only when built from the
approved catalog and v1 complexity limits. The UI must enforce visible limits,
active metadata-only edits, deactivation before structural edits, restore to
inactive, and activation before report request.

**Alternatives considered**:

- Let active definitions be structurally edited: rejected because active edit
  boundary preserves historical run meaning.
- Allow arbitrary query text or uploaded templates: rejected because this is
  explicitly unsupported and unsafe for v1.
- Activate restored definitions automatically: rejected because restore returns
  definitions to inactive by approved lifecycle rules.

## Decision: Verify safe diagnostics as part of feature acceptance

**Rationale**: Reporting screens can expose private filter values, report
contents, storage details, hidden fields, and cross-tenant identifiers if
diagnostics are careless. Diagnostics may include safe operation IDs, generic
state kinds, field labels, route names, and safe correlation IDs only.

**Alternatives considered**:

- Rely on existing logging conventions only: rejected because reporting
  introduces high-value output and filter privacy risk.
- Check only failed downloads: rejected because success, validation, catalog,
  lifecycle, and custom definition states also carry sensitive context.

## Decision: Keep Vue component boundaries explicit

**Rationale**: Vue 3 Composition API, `<script setup>`, props-down/events-up,
service-isolated HTTP access, focused components, and composables are the
approved frontend baseline. Reporting route views should compose smaller
components instead of owning service calls, mapping logic, timers, dialogs, and
multiple UI sections directly.

**Alternatives considered**:

- One large route component for the whole reporting workspace: rejected
  because it mixes orchestration, catalog, forms, history, downloads,
  lifecycle, definitions, and feedback responsibilities.
- Direct Axios in components: rejected by frontend architecture rules.
- Add TypeScript-only contracts: rejected because the current frontend
  baseline uses JavaScript with JSDoc typedefs and mapping helpers.
