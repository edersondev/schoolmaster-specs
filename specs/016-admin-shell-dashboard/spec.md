# Feature Specification: System Administrator Shell and Dashboard Foundation

**Feature Branch**: `016-admin-shell-dashboard`  
**Created**: 2026-06-22  
**Status**: Draft  
**Input**: User description: "Define frontend roadmap item 2: System Administrator Shell and Dashboard Foundation. Specify the reusable System Administrator application shell and dashboard foundation for SchoolMaster. The feature must cover the admin layout frame, sidebar navigation, top header, permission-aware menu visibility, dashboard summary cards, recent activity surface, quick actions, responsive shell behavior, global feedback surfaces, empty and loading states, and the reusable admin frame that later administration, reporting, and support modules will live inside. The feature must consume the completed frontend architecture baseline from specs/015-frontend-architecture-baseline/ and the guidance in docs/frontend-admin-system-architecture.md, docs/frontend-architecture.md, docs/frontend-guidelines.md, and docs/naming-conventions.md. Approve dashboard summary, recent activity, and notification data requirements only when backed by approved contract behavior; otherwise document placeholder behavior for intentionally UI-only surfaces. Preserve API-first delivery: frontend behavior may consume only approved OpenAPI-backed /api/v1 endpoints and documented contract semantics. Do not define authentication screens, concrete CRUD resource pages, backend implementation, undocumented API behavior, reporting module behavior, platform support workflows, or TypeScript requirements in this slice."

## Clarifications

### Session 2026-06-22

- Q: Should dashboard regions be placeholder-only in this slice or require live data contracts before implementation? → A: Placeholder-only dashboard regions for this slice; live data is blocked until later contracts.
- Q: How should navigation items and quick actions behave when the user lacks permission? → A: Hide items and actions when the user lacks permission.
- Q: How should quick actions behave when the target route or workflow is not approved yet? → A: Hide quick actions until the target route or workflow is approved.
- Q: What responsive navigation pattern should the admin shell use? → A: Desktop and tablet use a collapsible sidebar; mobile uses an overlay drawer.
- Q: How should the shell handle unauthorized, forbidden, or expired-session access in this slice? → A: Show shell-level unauthorized, forbidden, and session-expired states; auth screens remain a future slice.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Navigate the Admin Shell (Priority: P1)

A System Administrator needs a consistent authenticated workspace with a
sidebar, header, route content area, breadcrumbs or page context, global
feedback, and permission-aware navigation so future admin pages share one
predictable frame.

**Why this priority**: The shell is the required composition surface for later
administration, reporting, and support modules. Without it, later pages would
invent separate navigation, layout, responsive, and visibility behavior.

**Independent Test**: Review the shell specification and confirm a System
Administrator can identify where navigation, header actions, page content,
global feedback, responsive behavior, and permission-aware visibility belong
without depending on any concrete CRUD resource page.

**Acceptance Scenarios**:

1. **Given** a System Administrator has access to the admin workspace, **When**
   they open any future admin route, **Then** the route is framed by the same
   admin shell with sidebar navigation, top header, page context, and content
   region.
2. **Given** a System Administrator lacks permission for a navigation item,
   **When** the sidebar is rendered, **Then** that item is hidden while backend
   authorization remains authoritative.
3. **Given** a System Administrator uses a small screen, **When** they open
   the admin workspace, **Then** navigation opens through a mobile overlay
   drawer without losing access to page content or global feedback.
4. **Given** admin shell access is unauthorized, forbidden, or session-expired,
   **When** the shell attempts to render protected content, **Then** it shows a
   shell-level denial or expired-session state without defining login or
   recovery screens.

---

### User Story 2 - Review Dashboard Status (Priority: P2)

A System Administrator needs a dashboard landing surface that can show approved
summary cards, recent activity, notification indicators, and operational empty
or loading states so they can quickly understand system status when approved
data exists.

**Why this priority**: The dashboard is the first content surface inside the
admin shell and establishes reusable card, activity, quick-action, and feedback
patterns for later admin modules.

**Independent Test**: Review the dashboard specification and confirm it defines
which dashboard regions exist, when they may consume approved data, and what
placeholder behavior is shown when a data contract is not approved.

**Acceptance Scenarios**:

1. **Given** the dashboard is opened in this slice, **When** summary, recent
   activity, or notification regions are rendered, **Then** they show
   documented placeholder, empty, or unavailable behavior without consuming
   live dashboard data.
2. **Given** a later feature approves dashboard data contracts, **When** those
   contracts are consumed, **Then** summary cards display only approved values
   with clear labels and state indicators.
3. **Given** dashboard placeholder regions are loading or unavailable, **When**
   the System Administrator opens the dashboard, **Then** loading, empty,
   error, or forbidden states are visible and consistent with the frontend
   baseline.

---

### User Story 3 - Launch Approved Quick Actions (Priority: P3)

A System Administrator needs quick actions that route to approved admin
destinations so common tasks can be started from the dashboard without exposing
unapproved workflows.

**Why this priority**: Quick actions improve dashboard usefulness but must not
create shortcuts to resource pages, reporting flows, platform-support flows, or
backend behavior that has not been specified.

**Independent Test**: Review the quick-action rules and confirm every action is
linked to an approved destination and hidden when permission, route, or
workflow approval is missing.

**Acceptance Scenarios**:

1. **Given** a quick action targets an approved route, **When** the System
   Administrator selects it, **Then** they are taken to that route with normal
   permission checks applied.
2. **Given** a quick action depends on a future module, **When** the dashboard
   is rendered, **Then** the action is hidden until the target route or
   workflow is approved.
3. **Given** a System Administrator lacks permission for a quick action, **When**
   quick actions are displayed, **Then** that action is hidden.

### Edge Cases

- A System Administrator has no permitted navigation destinations beyond the
  dashboard.
- A System Administrator loses permission for a visible navigation item or
  quick action and the next render must hide the item.
- A dashboard region has approved structure but no approved live data source
  and must render placeholder, empty, or unavailable behavior.
- A summary metric, recent activity list, notification count, or quick action
  depends on live data and must remain blocked until a later contract approves
  it.
- A quick action target route or workflow is not approved and must remain
  hidden.
- A user resizes from desktop to tablet or mobile while the sidebar is open.
- A mobile user opens the overlay drawer, navigates, and the drawer must close
  without obscuring the loaded route content.
- Navigation state, notifications, and dashboard regions must recover from
  refresh without inventing client-side tenant or permission assumptions.
- Backend authorization denies access after a client-side navigation item or
  quick action was visible.
- A protected shell route detects unauthorized, forbidden, or session-expired
  access and must show the corresponding shell-level state without defining an
  authentication screen.
- Dashboard data fails, returns empty, or returns forbidden/not-found states.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: No backend implementation is included in this
  slice. Any data needed for dashboard summaries, recent activity, or
  notifications must already be approved through backend feature specs and
  OpenAPI before the frontend consumes it.
- **Frontend repository impact**: Establishes the first concrete frontend
  shell and dashboard feature for the System Administrator workspace,
  including layout framing, sidebar and header behavior, dashboard regions,
  permission-aware visibility, responsive behavior, and placeholder rules.
- **Specification or contract repository impact**: Adds this feature
  specification only. Live dashboard summary, recent activity, notification,
  or quick-action data contracts are deferred to later feature specs and
  OpenAPI updates; this slice does not update `api/openapi.yaml`.
- **Delivery ownership and sequencing**: `schoolmaster-specs` approves this
  shell and dashboard behavior first. `schoolmaster-frontend` may implement the
  approved shell after this spec and plan are complete. Backend or OpenAPI work
  must precede any live dashboard data consumption.

### API Contract Impact

- **OpenAPI update required**: No. This slice approves shell layout,
  responsive behavior, placeholder-only dashboard regions, permission-aware
  visibility based on existing session permissions, and static quick-action
  definitions. Live dashboard summary, recent activity, or notification data is
  blocked until later contracts approve it.
- **Versioned endpoints affected**: None directly in this specification. Future
  live data consumption must identify approved `/api/v1/...` operation IDs
  before implementation.
- **JSON response impact**: No response envelope, field, status, or error
  contract change is introduced by this placeholder-only shell and dashboard
  behavior. Live dashboard data requires a later approved response shape before
  use.
- **Authentication/authorization impact**: The shell assumes an already
  authenticated System Administrator context from the future authentication and
  session foundation. Client-side permission checks control navigation and quick
  action visibility only; backend authorization remains authoritative.
  Unauthorized, forbidden, and session-expired access is represented through
  shell-level denial states in this slice, while login and recovery screens
  remain future work.
- **Compatibility impact**: Additive frontend specification. It does not break
  existing backend contracts or require migration.

### Data & Tenancy Impact

- **Tenant scoping impact**: No new tenant-owned data is introduced. Any tenant
  or school context displayed in the shell must come from approved
  authenticated session or tenant-context behavior.
- **Cross-tenant or platform access impact**: No new platform-support or
  cross-school oversight workflow is approved. Platform-sensitive dashboard
  data must remain blocked until a separate approved contract exists.
- **Soft delete impact**: No soft-delete behavior is introduced. Inactive,
  deleted, restored, or conflict states may be displayed only when later module
  specs and contracts approve them.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST define a reusable System Administrator shell with
  sidebar navigation, top header, route content region, page context, and
  global feedback surfaces.
- **FR-002**: The shell MUST support permission-aware navigation visibility for
  menu sections, menu items, and quick actions by hiding unauthorized surfaces
  while treating backend authorization as authoritative.
- **FR-003**: The shell MUST define responsive behavior for desktop, tablet,
  and mobile widths, using a collapsible sidebar on desktop and tablet and an
  overlay navigation drawer on mobile.
- **FR-003a**: Mobile overlay navigation MUST close after route selection and
  MUST not block access to route content, global feedback, or header controls.
- **FR-004**: The shell MUST define global loading, empty, error, forbidden,
  unauthorized, and tenant-mismatch feedback placement for shell-level
  failures.
- **FR-004a**: The shell MUST show shell-level unauthorized, forbidden, and
  session-expired states for protected-route failures without defining login,
  logout, or account recovery screens.
- **FR-005**: The dashboard MUST define a landing surface with page context,
  summary card region, recent activity region, quick actions region, and
  notification or alert region.
- **FR-006**: Dashboard summary cards in this slice MUST use placeholder,
  empty, or unavailable behavior only and MUST NOT consume live metrics until a
  later contract approves labels, values, and status indicators.
- **FR-007**: The recent activity surface in this slice MUST use placeholder,
  empty, or unavailable behavior only and MUST NOT consume live activity data
  until a later contract approves it.
- **FR-008**: Quick actions MUST link only to approved routes or workflows;
  actions for future modules, unapproved workflows, or unauthorized users MUST
  be hidden.
- **FR-009**: Notification indicators in this slice MUST use placeholder or
  absent-state behavior only and MUST NOT invent live counts or statuses until
  a later notification contract approves them.
- **FR-010**: The feature MUST define how route metadata, breadcrumbs or page
  titles, permission requirements, and layout selection are represented for
  shell and dashboard routes.
- **FR-011**: The feature MUST define how shell state such as sidebar open,
  sidebar collapsed, active navigation item, and notification panel visibility
  behaves across route changes and refresh.
- **FR-012**: The feature MUST preserve the completed frontend architecture
  baseline, including JavaScript-only frontend conventions, reusable frontend
  boundaries, approved UI primitives, localization readiness, accessibility,
  and diagnostic boundaries.
- **FR-013**: The feature MUST require reusable shell, navigation, dashboard,
  card, activity, action, and feedback text to be centralized for future
  localization.
- **FR-014**: The feature MUST require reusable shell, navigation, dashboard,
  card, activity, action, and feedback states to meet the WCAG 2.1 AA
  accessibility baseline.
- **FR-015**: The feature MUST require shell and dashboard failures to be
  diagnosable through documented error normalization and user-action trace
  points.
- **FR-016**: The feature MUST NOT define authentication screens, account
  recovery flows, concrete CRUD resource pages, reporting workflows,
  platform-support workflows, backend implementation, undocumented API
  behavior, or TypeScript requirements.
- **FR-017**: Any live dashboard summary, recent activity, notification, or
  quick-action data consumption MUST be blocked until the required OpenAPI
  contract behavior is approved.
- **FR-018**: The feature MUST identify all follow-up frontend and contract
  dependencies needed before later administration, reporting, or support
  modules can attach to the shell.

### Key Entities *(include if feature involves data)*

- **AdminShell**: The authenticated System Administrator workspace frame,
  including navigation, header, content region, page context, global feedback,
  responsive state, desktop/tablet collapsed state, and mobile drawer state.
- **NavigationItem**: A sidebar entry or section with label, destination,
  permission visibility rule, active state, ordering, and hidden state for
  unauthorized users.
- **HeaderControl**: A top-header surface for page context, global actions,
  notification indicators, user/session affordances, and responsive controls.
- **DashboardSummaryCard**: A dashboard card representing placeholder-only
  structure in this slice, with live labels, values, status, and descriptions
  blocked until later contract approval.
- **RecentActivityItem**: A dashboard activity placeholder representing the
  future structure for operational activity, with live timestamp,
  actor/context label, and action label blocked until later contract approval.
- **QuickAction**: A dashboard shortcut to an approved destination, hidden when
  permission, route approval, or workflow approval is missing.
- **ShellFeedbackState**: A standard shell or dashboard state for loading,
  empty, error, forbidden, unauthorized, session-expired, tenant-mismatch, or
  unavailable conditions.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of future System Administrator pages can identify the
  approved shell region they render inside without redefining layout,
  navigation, or header behavior.
- **SC-002**: 100% of sidebar menu items and quick actions in this slice map to
  a documented permission rule and are hidden when that permission is absent.
- **SC-003**: 100% of dashboard summary, recent activity, and notification
  surfaces display documented placeholder, empty, unavailable, or absent-state
  behavior in this slice with no live dashboard data consumption.
- **SC-004**: A reviewer can validate desktop/tablet collapsible sidebar
  behavior and mobile overlay drawer behavior from the specification without
  requiring another feature spec.
- **SC-005**: A System Administrator can reach the dashboard content area and
  available navigation destinations in no more than two interactions after
  entering the authenticated admin workspace.
- **SC-006**: Accessibility review confirms the shell, navigation, header,
  dashboard cards, quick actions, activity list, and feedback states satisfy
  WCAG 2.1 AA expectations before frontend implementation is accepted.
- **SC-007**: Frontend review rejects 100% of undocumented dashboard data,
  notification counts, recent activity items, or quick-action destinations.
- **SC-008**: Later administration, reporting, and support module specs can
  attach routes to the shell by referencing this feature and defining only
  their module-specific navigation and content requirements.
- **SC-009**: Unauthorized, forbidden, and session-expired shell access can be
  validated through shell-level states without requiring login, logout, or
  account recovery screen behavior.

## Assumptions

- The frontend architecture baseline from
  `specs/015-frontend-architecture-baseline/` is complete and remains the
  governing baseline for this feature.
- The System Administrator using this shell is already authenticated through a
  future authentication and session foundation; login and recovery screens are
  outside this slice.
- Initial dashboard summary cards, recent activity, and notification indicators
  are placeholder-only in this slice; live data consumption requires later
  contract approval.
- Client-side permission visibility is a usability control and does not replace
  backend authorization.
- Later feature specs own concrete CRUD resources, reporting workflows,
  platform-support workflows, and their route entries.
- One launch language is sufficient for this slice, but reusable text must be
  ready for future localization.
- Desktop, tablet, and mobile browser widths are in scope for responsive shell
  behavior; mobile-native application work remains out of scope.
