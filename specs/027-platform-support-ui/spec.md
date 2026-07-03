# Feature Specification: Platform Support Access and Cross-School Oversight UI

**Feature Branch**: `027-platform-support-ui`  
**Created**: 2026-07-03  
**Status**: Ready for Implementation  
**Input**: User description: "Specify feature 027 Platform Support Access and Cross-School Oversight UI. Define platform-only frontend surfaces for minimized cross-school summaries, support drill-down, approval state visibility, redacted details, and support audit review. Consume only approved platform/support routes. Specify redaction, small-count suppression, approval expiry, opt-in, revocation, denied-access behavior, tenant-safe navigation, stale-response handling, diagnostics redaction, empty states, loading states, and authorization boundaries. Keep scope frontend-only for Vue 3 SPA and align contracts with OpenAPI."

## Clarifications

### Session 2026-07-03

- Q: Should this frontend slice include school-admin target-school opt-in create/revoke screens? → A: Platform UI only: display target-school opt-in state; no school-admin opt-in create/revoke screens.
- Q: How should the UI keep support decision and diagnostics state current while approval, revocation, or expiry can change? → A: Auto-refresh visible support decision/diagnostics state, plus manual refresh.
- Q: Which surface should the platform support workspace root open by default? → A: Platform Operational Oversight.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Review Platform Operational Oversight (Priority: P1)

A platform administrator with explicit platform oversight permissions opens a platform support workspace, reviews minimized cross-school school summaries and reporting-health indicators, and can distinguish normal, suppressed, unavailable, denied, and empty states without seeing school-owned detail records.

**Why this priority**: Cross-school oversight is the core platform value, and the UI must make minimized visibility useful without becoming a bypass around school-scoped authorization.

**Independent Test**: Can be fully tested by signing in as a platform administrator with platform-wide operational oversight and reporting overview permissions, opening the platform workspace, loading school summaries and reporting overview data, and verifying only approved minimized fields, suppressed-count indicators, loading states, empty states, denied states, and tenant-safe errors appear.

**Acceptance Scenarios**:

1. **Given** an authenticated platform administrator has explicit platform-wide operational oversight permission, **When** they open the cross-school oversight workspace, **Then** the UI lists only approved minimized school identity, school status, aggregate operational indicators, reporting-health summaries, lifecycle summaries, support diagnostic indicators, and suppression markers.
2. **Given** an authenticated platform user opens the platform support workspace root with operational oversight permission, **When** the workspace resolves its default route, **Then** the UI opens Platform Operational Oversight by default.
3. **Given** platform summary values are suppressed because protected counts are below 5, **When** the summary is rendered, **Then** the UI shows the documented suppressed-count representation without deriving hidden values or replacing suppression with identifying detail.
4. **Given** the actor lacks platform-wide operational oversight or reporting overview permission, **When** they attempt to open platform summaries or reporting overview surfaces, **Then** the UI renders the documented denied state without exposing school counts, hidden school identity, report health, or support activity details.
5. **Given** platform summaries are empty, filtered to no results, temporarily unavailable, or still loading, **When** the workspace renders, **Then** the UI distinguishes these states from authorization denial, tenant mismatch, validation failure, not-found, conflict, and stale-response states.

---

### User Story 2 - Manage Support Access Decisions and Approval States (Priority: P2)

A platform support user requests or reviews target-school support access, sees target-school opt-in and internal platform approval state, understands 24-hour expiry, and cannot drill into diagnostics until both approval gates are valid.

**Why this priority**: Support drill-down is sensitive. Users need clear decision state and approval expiry visibility before any redacted diagnostics are reachable.

**Independent Test**: Can be fully tested by signing in as a support user with support access permission, creating or viewing support access decisions across requested, approved, denied, expired, revoked, stale, and mismatched states, and verifying drill-down controls remain unavailable until active target-school opt-in and internal platform approval are both valid.

**Acceptance Scenarios**:

1. **Given** a support user has support access permission and enters documented target school, reason code, purpose, and correlation metadata, **When** they submit a support access request with active target-school opt-in but no internal platform approval, **Then** the UI shows a requested or pending decision state and does not load school diagnostics.
2. **Given** a support access decision has active target-school opt-in and internal platform approval no older than 24 hours, **When** the support user views the decision, **Then** the UI shows approved status, expiry visibility, target-school match, and drill-down availability for the approved target school only.
3. **Given** a decision or opt-in is expired, revoked, denied, stale, mismatched, older than 24 hours, or concurrently changed, **When** the support user opens decision or drill-down surfaces, **Then** the UI blocks diagnostics, shows the documented conflict or denied state, and avoids implying reusable access.
4. **Given** a platform approver has explicit support approval permission, **When** they approve or revoke a support access decision, **Then** the UI updates the decision state, expiry visibility, and audit-relevant status without exposing protected school-owned operational detail.
5. **Given** target-school opt-in state exists, is revoked, or expires through approved non-platform UI or backend workflow, **When** the platform support user views a related support access decision, **Then** the UI displays the returned opt-in state, revocation state, and 24-hour expiry without exposing school-admin opt-in create or revoke controls.
6. **Given** a visible support access decision or diagnostics view depends on active approval gates, **When** the screen remains open, **Then** the UI automatically refreshes visible state, provides manual refresh, and removes diagnostics access when returned state becomes expired, revoked, denied, stale, mismatched, or unavailable.

---

### User Story 3 - Perform Redacted Support Drill-Down (Priority: P3)

A platform support user with a valid support access decision opens redacted read-only diagnostics for one target school and can review approved troubleshooting indicators without generated report downloads, raw outputs, unrestricted search, impersonation, or operational writes.

**Why this priority**: Diagnostics are the highest-risk surface. They must be useful for support while visibly bounded by approval, redaction, and read-only rules.

**Independent Test**: Can be fully tested by opening diagnostics for an approved target school and verifying the UI renders only approved redacted school diagnostics, support indicators, reporting-health summaries, lifecycle summaries, suppressed counts, and safe errors while blocking generated report downloads, raw outputs, emergency access, unrestricted record search, impersonation, and writes.

**Acceptance Scenarios**:

1. **Given** support drill-down is approved for one target school, **When** the support user opens diagnostics, **Then** the UI renders only approved redacted diagnostic fields for that target school and shows the active decision context and expiry.
2. **Given** diagnostics contain protected counts below 5 or redacted fields, **When** the UI renders the diagnostic view, **Then** it preserves suppression and redaction indicators exactly as documented and does not expose hidden values through labels, filters, filenames, errors, diagnostics, or export-like behavior.
3. **Given** a support user attempts to reach diagnostics for a different school, stale decision, revoked opt-in, expired approval, missing approval, or unsupported diagnostic category, **When** the route or action resolves, **Then** the UI shows safe denied, conflict, unavailable, or not-found feedback without disclosing protected record existence.
4. **Given** the support user attempts generated report downloads, raw report output viewing, private file metadata access, emergency access, unrestricted record search, impersonation, account reactivation, report retry or cancel, school status changes, or other school-owned writes, **When** the action is visible by route, stale state, or browser navigation, **Then** the UI hides or blocks the action and handles any documented denial safely.

---

### User Story 4 - Review Support Audit Activity (Priority: P4)

An authorized platform compliance or support-audit reviewer lists minimized support audit events for platform summaries, support decisions, opt-ins, approvals, revocations, denials, expirations, conflicts, and diagnostics access.

**Why this priority**: Platform support visibility must be reviewable and traceable so cross-school access can be governed after use.

**Independent Test**: Can be fully tested by signing in as an actor with support audit review permission, listing audit events with filters and pagination, and verifying events include only actor, action, outcome, target school where safe, correlation ID, reason code, timestamp, and minimized metadata.

**Acceptance Scenarios**:

1. **Given** an authorized audit reviewer opens support audit review, **When** audit events exist, **Then** the UI lists minimized event summaries for allowed access, denied access, validation rejection, platform overview reads, support decisions, opt-ins, approvals, revocations, expirations, conflicts, and diagnostics access.
2. **Given** audit metadata is redacted or omitted by contract, **When** the UI renders audit review, **Then** it does not reconstruct or expose credentials, bearer tokens, private paths, raw outputs, private content, full records, full request or response payloads, or unauthorized cross-tenant details.
3. **Given** audit filters return no results, are invalid, or are unavailable, **When** audit review loads, **Then** the UI distinguishes no audit results from validation, denied, tenant-safe not-found, conflict, unavailable, and loading states.
4. **Given** an actor lacks support audit review permission, **When** they attempt to open audit review, **Then** the UI renders a denied state without exposing support activity existence or details.

### Edge Cases

- Platform actor is authenticated but inactive, lacks platform workspace permissions, or has only school-scoped roles.
- Platform administrator has school administration permissions but lacks explicit platform-wide operational oversight, platform reporting overview, support approval, support drill-down, or support audit review permissions.
- Target-school opt-in state changes through an approved non-platform UI or backend workflow while a support access decision is open.
- Approved platform/support routes are unavailable, not yet promoted into OpenAPI, return temporary-unavailable responses, or return a shape missing required documented fields.
- Platform summaries or diagnostics contain protected counts below 5, suppressed values, redacted fields, unavailable indicators, or partially available summary groups.
- Support access decision, target-school opt-in, or internal platform approval expires while a decision page or diagnostics page is open.
- A support access decision is revoked, denied, mismatched, stale, concurrently changed, or older than 24 hours while the UI is loading diagnostics.
- Automatic refresh detects support decision expiry, opt-in revocation, approval revocation, diagnostics denial, or unavailable state while keyboard focus remains on a decision action, diagnostics view, filter, or audit review control.
- Active route, selected school summary, filters, support access decision, authorization state, or authenticated user changes while requests are in flight.
- Direct routes target missing, unauthorized, revoked, expired, mismatched, stale, unsupported, or cross-school support decisions, opt-ins, diagnostics, summaries, or audit events.
- Audit events reference denied cross-tenant attempts where target details are intentionally omitted.
- Visible errors, loading labels, diagnostics, filenames, audit metadata, and automated test output must not expose hidden counts, raw report outputs, private file paths, private content, credentials, token values, full records, full payloads, role internals, or unauthorized school-owned record existence.
- Unsupported platform/support actions are attempted by route, stale controls, browser history, or manipulated state, including support writes, generated report downloads, raw output viewing, emergency access, unrestricted impersonation, unrestricted search, billing, payroll, accounting, messaging, notifications, live classroom, video conferencing, permanent purge, legal hold, anonymization, or undocumented APIs.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No new backend behavior is included in this frontend feature. Platform support, oversight, support access decision, opt-in, diagnostics, and audit behavior must already be approved, implemented, and contract-compliant before frontend runtime exposure.
- **Frontend repository impact**: Adds protected platform support workspace routes, platform summary views, cross-school reporting overview views, support access decision views, approval and revocation state controls where approved, target-school opt-in status surfaces where returned by approved platform/support contracts, redacted diagnostics views, support audit review, permission-aware navigation, loading states, empty states, denied states, conflict states, stale-response handling, and diagnostics redaction.
- **Specification or contract repository impact**: This specification defines the frontend consumption boundary for approved platform/support contracts. OpenAPI changes are required before frontend implementation if the platform/support operations from `013-platform-support-access` are not yet promoted into `api/openapi.yaml` and the platform contract mirror.
- **Delivery ownership and sequencing**: `schoolmaster-specs` defines this UI boundary first, `schoolmaster-backend` and OpenAPI contract work must provide approved platform/support operations before runtime consumption, and `schoolmaster-frontend` implements only after plan and tasks confirm route and operation readiness.

### API Contract Impact

- **OpenAPI update required**: Yes before frontend implementation if approved platform/support operations are absent from `api/openapi.yaml` or the platform contract mirror. No UI may consume a platform/support operation until its route, operation ID, request schema, response schema, authorization behavior, validation behavior, conflict behavior, and redaction semantics are documented.
- **Versioned endpoints affected**: Frontend may consume only approved `/api/v1/platform/schools`, `/api/v1/platform/reporting/overview`, `/api/v1/platform/support-access`, `/api/v1/platform/support-access/{supportAccessId}`, `/api/v1/platform/support-access/{supportAccessId}/approve`, `/api/v1/platform/support-access/{supportAccessId}/revoke`, `/api/v1/platform/support/schools/{schoolId}/diagnostics`, and `/api/v1/platform/support-audit-events` operations, plus already approved authentication, current-user, permission, session, and school context behavior. Dedicated school-admin opt-in create/revoke operations remain outside this frontend slice.
- **JSON response impact**: UI behavior depends only on documented success, accepted, paginated, validation, unauthorized, forbidden, tenant-mismatch, not-found, conflict, temporary-unavailable, suppressed-count, redacted-field, decision-state, opt-in-state, approval-state, diagnostics, and audit-summary semantics. No UI may depend on undocumented fields, status codes, denial reasons, backend internals, hidden actor metadata, private payloads, raw report outputs, or private storage metadata.
- **Authentication/authorization impact**: All platform support workspace screens require authenticated active platform access where applicable. Platform summaries require explicit platform-wide operational oversight permission. Cross-school reporting overview requires explicit platform-wide reporting permission. Support decision and diagnostics surfaces require explicit support access or support drill-down permissions plus valid target-school opt-in and internal platform approval where diagnostics are involved. Approval and revocation controls require explicit support approval or revocation permission. Target-school opt-in state is display-only in this slice. Audit review requires explicit support audit review permission. Navigation and controls are visibility aids only; backend authorization remains authoritative.
- **Permission capability source**: Frontend route meta, navigation gates, service guards, and composables must use the exact permission identifiers exposed by approved current-user, permission-definition, and OpenAPI-backed platform/support contracts from `013-platform-support-access`. If those identifiers are absent or not yet promoted, the UI must treat the affected surface as contract-unavailable instead of inventing local permission aliases.
- **Compatibility impact**: Frontend delivery is additive. It must not change existing authentication, school-scoped administration, teacher workflow, student self-service, guardian self-service, reporting workspace, backend platform support, or OpenAPI behavior unless a separate approved specification changes those areas.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-owned records remain scoped by the target school. Platform UI may render only minimized or redacted cross-school summaries and diagnostics returned by approved platform/support operations. Existing school-scoped endpoints remain inaccessible as an implicit support shortcut.
- **Cross-tenant or platform access impact**: This feature exposes platform-only UI for the approved cross-school oversight and support exceptions defined by platform/support contracts. Support drill-down is read-only, target-school-bound, approval-gated, expires after 24 hours, and cannot be reused across schools.
- **Soft delete impact**: UI may render approved lifecycle summary counts and support audit state where returned by platform/support contracts. It must not expose soft-deleted school-owned detail records or restore/delete controls for school-owned operational records through support access.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The UI MUST expose only approved platform/support behavior documented before frontend implementation begins.
- **FR-002**: The UI MUST require an authenticated active actor before loading platform/support data or enabling protected platform/support actions.
- **FR-003**: The UI MUST keep platform summaries, reporting overview, support access decisions, approvals, revocations, display-only target-school opt-in state, diagnostics, and audit review permission-gated by the approved platform/support permissions.
- **FR-004**: The UI MUST show platform support navigation only to actors with relevant platform/support permission signals and MUST treat navigation visibility as non-authoritative.
- **FR-005**: The UI MUST load cross-school school summaries only through the approved platform school summary operation and render only documented minimized fields.
- **FR-006**: The UI MUST load cross-school reporting overview only through the approved platform reporting overview operation and render only documented aggregate report health, lifecycle, retention, output availability, and failure-summary indicators.
- **FR-007**: The UI MUST preserve documented small-count suppression below 5 and redacted-field indicators without deriving, inferring, or displaying hidden values.
- **FR-008**: The UI MUST distinguish true empty summaries, no filtered results, denied access, validation failure, tenant mismatch, not found, conflict, temporary unavailable, loading, and stale-response states.
- **FR-009**: The platform support workspace root MUST open Platform Operational Oversight by default after authenticated access and operational oversight permission are confirmed.
- **FR-010**: The UI MUST submit support access requests only with documented target school, reason code, purpose, and correlation metadata.
- **FR-011**: The UI MUST render support access decision states `requested`, `approved`, `denied`, `expired`, and `revoked` without inventing additional decision states.
- **FR-012**: The UI MUST show target-school opt-in state, internal platform approval state, target-school match, revocation state, and 24-hour expiry visibility where returned by approved support access contracts.
- **FR-013**: The UI MUST block support diagnostics until both target-school opt-in and internal platform approval are valid for the same target school and no older than 24 hours.
- **FR-014**: The UI MUST hide or block diagnostics when support access is missing, pending, denied, expired, revoked, stale, mismatched, concurrently changed, older than 24 hours, or not authorized.
- **FR-015**: The UI MUST expose support approval and revocation controls only where approved by contract, current decision state, actor permission, and documented target-school context.
- **FR-016**: The UI MUST display target-school opt-in state, revocation state, and expiry where returned by approved platform/support contracts and MUST NOT expose school-admin target-school opt-in create or revoke controls in this slice.
- **FR-017**: The UI MUST load support diagnostics only through the approved target-school diagnostics operation and render only approved redacted read-only diagnostic fields.
- **FR-018**: The UI MUST display active support decision context and expiry while diagnostics are visible.
- **FR-019**: The UI MUST automatically refresh visible support access decision and diagnostics state while approval gates remain active or pending, and MUST also provide a manual refresh action.
- **FR-020**: The UI MUST remove diagnostics access and render documented safe feedback when automatic or manual refresh returns expired, revoked, denied, stale, mismatched, unavailable, or no-longer-authorized decision state.
- **FR-021**: The UI MUST NOT expose generated report downloads, raw report outputs, private file metadata, unrestricted record search, emergency access, unrestricted impersonation, support writes, account reactivation, report retry or cancel, school status changes, or other school-owned operational mutations through support access.
- **FR-022**: The UI MUST list support audit events only through the approved support audit operation and render only minimized actor, action, outcome, target school where safe, correlation ID, reason code, timestamp, and minimized metadata.
- **FR-023**: The UI MUST prevent visible errors, client-side diagnostics, route labels, filenames, audit review, and automated test output from exposing hidden counts, credentials, bearer tokens, private paths, raw outputs, private content, full records, full payloads, role internals, or unauthorized school-owned record existence.
- **FR-024**: The UI MUST ignore or cancel stale platform/support responses when the user changes route, selected school summary, filters, support access decision, target school, authentication state, permission state, or open approval/revocation state before the response is applied.
- **FR-025**: The UI MUST handle documented unauthorized, forbidden, validation, tenant-mismatch, not-found, conflict, stale-response, temporary-unavailable, suppressed-count, redacted-field, expired, revoked, and denied response categories without relying on undocumented response details.
- **FR-026**: The UI MUST reuse existing protected shell, role-aware navigation, list, detail, status, pagination, loading, empty-state, denial, unavailable, conflict, stale-response, and safe-error behavior from completed frontend features.
- **FR-027**: The UI MUST keep platform support workspace routes, page titles, navigation labels, empty states, denied states, and diagnostics distinct from school administration, teacher workflow, student self-service, guardian self-service, and reporting workspace surfaces.
- **FR-028**: The UI MUST include test coverage or equivalent verification for platform summary loading, reporting overview loading, default Platform Operational Oversight route, small-count suppression, support decision states, target-school opt-in state, approval state, 24-hour expiry, revocation, redacted diagnostics, audit review, denied access, conflict, empty states, loading states, stale responses, unsupported actions, and no-sensitive-data diagnostics behavior.

### Key Entities *(include if feature involves data)*

- **PlatformSupportWorkspace**: Protected platform-only UI surface for cross-school oversight, support access decisions, redacted diagnostics, and support audit review.
- **PlatformOversightSummaryView**: Minimized cross-school presentation of school identity/status, operational indicators, reporting-health summaries, lifecycle summaries, diagnostic indicators, and suppressed-count markers.
- **PlatformReportingOverviewView**: Aggregate cross-school reporting-health presentation without raw report outputs, generated files, private payloads, or school-owned detail records.
- **SupportAccessDecisionView**: UI representation of a target-school support decision, reason code, correlation metadata, target-school opt-in state, internal platform approval state, lifecycle state, expiry, revocation, and target-school match.
- **SupportOptInView**: Display-only UI representation of school-side opt-in state for support access, visible only through approved platform/support contracts and target-school context.
- **SupportDiagnosticView**: Redacted read-only target-school diagnostic presentation available only when support drill-down approval gates are valid.
- **SupportAuditReviewView**: Minimized audit event list for platform/support access activity, denials, decisions, opt-ins, approvals, revocations, expirations, conflicts, and diagnostics access.
- **PlatformSupportSafeState**: UI state category for denied, unavailable, not-found, empty, suppressed, redacted, expired, revoked, conflict, unsupported, stale, or missing-context behavior that avoids tenant and record enumeration.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of platform summary, reporting overview, support decision, approval, revocation, display-only opt-in state, diagnostics, and audit UI behavior can be traced to approved platform/support contract behavior before frontend implementation begins.
- **SC-002**: In usability checks, an authorized platform administrator can open the platform support workspace, identify school operational status, reporting-health status, suppressed counts, and unavailable states in under 3 minutes without assistance.
- **SC-003**: In usability checks, an authorized support user can identify whether a support access decision is requested, approved, denied, expired, revoked, stale, or mismatched with at least 90% accuracy.
- **SC-004**: In usability checks, an authorized support user can identify target-school opt-in state, internal platform approval state, revocation state, and 24-hour expiry timing in under 2 minutes without assistance and without school-admin opt-in controls.
- **SC-005**: 100% of tested diagnostics views remain unavailable until target-school opt-in and internal platform approval are both valid for the same target school and no older than 24 hours.
- **SC-006**: 100% of tested platform summary and diagnostics cases preserve suppressed-count and redacted-field indicators without exposing hidden values in visible UI, errors, route labels, filenames, diagnostics, or test output.
- **SC-007**: 100% of tested unsupported actions, including support writes, generated report downloads, raw output viewing, emergency access, unrestricted impersonation, unrestricted search, and school-owned operational mutations, are hidden or blocked before submission when the UI has enough approved state to decide.
- **SC-008**: 100% of tested unauthorized, forbidden, tenant-mismatch, validation, not-found, conflict, expired, revoked, stale, unavailable, empty, and temporary-unavailable responses show safe feedback without exposing protected school-owned record existence.
- **SC-009**: 100% of tested stale responses caused by route, selected school, filter, decision, target school, authentication, permission, approval, revocation, or opt-in changes do not overwrite the current visible screen state.
- **SC-010**: An authorized audit reviewer can find a known support access event by documented filters and confirm action, outcome, safe target-school attribution, correlation ID, reason code, and timestamp in under 2 minutes without seeing private payloads.
- **SC-011**: Review confirms no UI surface consumes undocumented platform/support APIs or exposes school-admin target-school opt-in create/revoke screens, school-scoped administration, reporting lifecycle, teacher workflow, student, guardian, billing, messaging, notification, purge, legal hold, anonymization, or advanced assessment behavior through support access.
- **SC-012**: 100% of tested visible support decision or diagnostics views refresh active approval state without page reload and remove diagnostics access after returned expiry, revocation, denial, stale, mismatched, unavailable, or no-longer-authorized state.
- **SC-013**: 100% of tested platform support workspace root visits land on Platform Operational Oversight after authenticated access and operational oversight permission are confirmed.

## Assumptions

- Backend feature `013-platform-support-access` is the source of product truth for platform summaries, cross-school reporting overview, support decisions, target-school opt-in, internal platform approval, redacted diagnostics, audit review, small-count suppression below 5, 24-hour approval windows, and blocked support behavior.
- Approved platform/support OpenAPI operations may need promotion into `api/openapi.yaml` and the platform contract mirror before frontend implementation can consume them.
- Existing authentication and session UI behavior from `017-auth-session-ui` provides protected-route handling, current-user hydration, session-expired handling, unauthorized handling, forbidden handling, inactive-user handling, and safe permission loading.
- Existing frontend features provide reusable protected-shell, role-aware navigation, list, detail, status, pagination, loading, empty-state, denial, unavailable, conflict, stale-response, download-blocking, and not-found patterns that this feature must reuse rather than redefine.
- Platform support workspace users are platform administrators, platform support users, platform approvers, and support audit reviewers. School-admin target-school opt-in create/revoke screens are outside this frontend slice.
- Support drill-down is read-only, target-school-bound, approval-gated, and expires after 24 hours. One support decision cannot be reused across schools.
- Generated report downloads, raw report outputs, private file metadata, emergency access, unrestricted impersonation, unrestricted record search, support writes, school-owned operational mutations, billing, payroll, accounting, messaging, notifications, live classroom, video conferencing, permanent purge, legal hold, and anonymization remain outside this frontend slice.
