# Research: Platform Support Access and Cross-School Oversight UI

## Decision: Block frontend implementation until platform/support operations are approved in OpenAPI

**Rationale**: Feature `013-platform-support-access` defines the backend
operation boundary, but aggregate `api/openapi.yaml` and the platform contract
mirror may not yet expose the platform/support operations. The frontend must
not consume inferred routes, response fields, filters, denial semantics, or
diagnostic categories.

**Alternatives considered**:

- Build against proposed routes from the backend spec only: rejected because
  OpenAPI is the cross-repository source of truth for frontend consumption.
- Stub platform/support calls in the frontend until backend catches up:
  rejected because it risks locking in undocumented payloads.
- Hide this feature until backend implementation completes: rejected because
  planning can define the frontend boundary while preserving the contract gate.

## Decision: Keep platform support workspace separate from school-scoped workspaces

**Rationale**: Platform support access is a documented cross-school exception,
not school administration. Dedicated route, service, contract, and display
boundaries prevent platform roles from appearing to inherit school-scoped
student, guardian, teacher, administration, or reporting authority.

**Alternatives considered**:

- Add support screens inside school administration: rejected because support
  actors are platform-scoped and support drill-down must remain narrower than
  school administrator access.
- Add platform support controls to reporting workspace: rejected because
  platform support cannot download generated reports or run report lifecycle
  actions.
- Use a generic admin/support dashboard: rejected because support access has
  unique opt-in, approval, redaction, audit, and expiration rules.

## Decision: Open Platform Operational Oversight by default

**Rationale**: The safest first surface is minimized cross-school operational
oversight. It provides platform value while avoiding immediate entry into
sensitive support decision or diagnostics workflows.

**Alternatives considered**:

- Open Support Access Decisions by default: rejected because it centers
  exception handling before overview context.
- Open Support Audit Review by default: rejected because audit review is for
  governance after activity, not the primary operating surface.
- Open last visited tab by default: rejected because it makes first-use and
  acceptance tests less predictable.

## Decision: Keep target-school opt-in state display-only

**Rationale**: The feature prompt requests a platform-only UI. Showing
target-school opt-in state is necessary for support decisions, but
school-admin opt-in create/revoke screens belong to a separate school-scoped
surface or future feature.

**Alternatives considered**:

- Include school-admin opt-in screens in this feature: rejected because it
  breaks the platform-only scope and adds a school-scoped workflow.
- Let platform actors create target-school opt-ins: rejected because opt-in is
  a school-side approval gate.
- Hide opt-in state entirely: rejected because support users need to
  understand why diagnostics are blocked or available.

## Decision: Model support decisions around approval gates and expiry

**Rationale**: Support diagnostics are valid only for one target school when
target-school opt-in and internal platform approval are valid, matched, active,
and no older than 24 hours. UI state must represent requested, approved,
denied, expired, revoked, stale, and mismatched outcomes without implying
reusable support context.

**Alternatives considered**:

- Treat support access as a binary enabled/disabled flag: rejected because it
  hides why access is blocked and weakens test coverage.
- Cache approved access as long as the route remains open: rejected because
  revocation, expiry, and concurrent changes must remove diagnostics access.
- Allow one decision to select multiple schools: rejected by the target-school
  bound backend contract.

## Decision: Auto-refresh visible support decision and diagnostics state

**Rationale**: Approval, revocation, expiration, and diagnostics denial can
change while a page is open. Automatic refresh keeps sensitive access current,
and manual refresh provides explicit user control. Responses must be ignored
when route, selected decision, target school, auth, permission, approval,
revocation, or opt-in context changes.

**Alternatives considered**:

- Manual refresh only: rejected because expired or revoked diagnostics could
  appear current until user action.
- Realtime push updates: rejected because no approved realtime contract exists.
- Local expiry countdown only: rejected because server-side revocation,
  denial, and mismatched approval require authoritative refresh.

## Decision: Preserve redaction and small-count suppression as contract state

**Rationale**: Suppressed counts below 5 and redacted fields are privacy
controls. The frontend must display the returned suppression/redaction state
without deriving hidden values from totals, labels, filters, tooltips,
diagnostics, exports, logs, or filenames.

**Alternatives considered**:

- Hide suppressed rows or fields entirely: rejected because users need to know
  data is intentionally suppressed.
- Replace suppressed values with estimated ranges: rejected because it can
  disclose small counts.
- Show raw values to platform administrators: rejected because platform roles
  do not bypass minimization rules.

## Decision: Use route-local composables and minimal Pinia coordination

**Rationale**: Platform support screens include sensitive target-school,
decision, diagnostic, and audit context. Keeping most state route-local limits
unnecessary persistence. Pinia remains for session, shell, navigation, and
permission state.

**Alternatives considered**:

- Store all platform support lists and diagnostics in Pinia: rejected because
  it persists more sensitive state than necessary across routes.
- Put HTTP, mapping, and refresh logic in route components: rejected by
  service isolation and component-boundary rules.
- Use component templates for eligibility derivation: rejected because
  derived access state belongs in composables or mapped view models.

## Decision: Gate support audit review with minimized metadata only

**Rationale**: Support audit review is required for governance but must not
become a secondary disclosure channel. Audit event UI may show actor, action,
outcome, safe target-school attribution, correlation ID, reason code,
timestamp, and minimized metadata only.

**Alternatives considered**:

- Show raw audit metadata to platform auditors: rejected because audit metadata
  can contain denied target references or private payload fragments.
- Hide denied events: rejected because denied and validation outcomes are part
  of support governance.
- Merge audit review into diagnostics: rejected because diagnostics and audit
  review have different permissions and privacy expectations.

## Decision: Verify safe diagnostics as part of feature acceptance

**Rationale**: Platform support UI can expose hidden counts, raw report
outputs, private file metadata, full records, full payloads, credentials,
tokens, role internals, and unauthorized school-owned record existence if
errors or diagnostics are careless. Verification must cover visible UI,
client diagnostics, filenames, and automated test output.

**Alternatives considered**:

- Rely only on shared error handling tests: rejected because platform support
  introduces cross-school visibility and approval-gated diagnostics.
- Check diagnostics only for failed requests: rejected because success,
  conflict, audit, and refresh states can also leak data.

## Decision: Keep Vue component boundaries explicit

**Rationale**: Vue 3 Composition API, `<script setup>`, props-down/events-up,
service-isolated HTTP access, focused components, and composables are the
approved frontend baseline. Platform support route views should compose
smaller components instead of owning service calls, timers, dialogs, audit
filters, diagnostics, and multiple UI sections directly.

**Alternatives considered**:

- One large route component for the whole platform support workspace: rejected
  because it mixes orchestration, summaries, decisions, approvals,
  diagnostics, audit review, refresh, and feedback responsibilities.
- Direct Axios in components: rejected by frontend architecture rules.
- Add TypeScript-only contracts: rejected because the current frontend
  baseline uses JavaScript with JSDoc typedefs and mapping helpers.
