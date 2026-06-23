# Admin Shell and Dashboard Contract

This contract defines what `schoolmaster-frontend` may implement for the
System Administrator shell and placeholder dashboard foundation.

## Scope Contract

This feature approves:

- System Administrator shell layout frame.
- Sidebar navigation with hidden unauthorized items.
- Top header with page context and responsive navigation controls.
- Desktop/tablet collapsible sidebar.
- Mobile overlay navigation drawer.
- Placeholder-only dashboard summary, recent activity, and notification regions.
- Quick actions only for approved routes and workflows.
- Shell-level unauthorized, forbidden, session-expired, loading, empty, error,
  tenant-mismatch, and unavailable states.

This feature does not approve:

- Live dashboard summary data.
- Live recent activity data.
- Live notification counts or statuses.
- Authentication screens, logout flow, or account recovery.
- Concrete CRUD resource pages.
- Reporting workflows.
- Platform-support workflows.
- Backend implementation.
- OpenAPI changes.
- TypeScript requirements.

## Route Metadata Contract

Admin shell routes must provide route metadata sufficient for:

- `layout`: selects the System Administrator shell.
- `title`: provides a localized page title key or approved title source.
- `breadcrumb`: provides ordered page context.
- `permissions`: lists visibility and access permission keys.
- `sidebar`: defines sidebar placement, order, icon, and section.
- `requiresAuth`: identifies protected routes.

Routes for future modules may attach to the shell only when their feature spec
approves the route and behavior.

## Navigation Contract

- Sidebar items are derived from approved route metadata or approved static
  shell definitions.
- Sidebar items must be hidden when the user lacks permission.
- Sidebar items must be hidden when the route or workflow is not approved.
- Active state must follow the current route.
- Navigation visibility is a frontend usability aid only; backend
  authorization remains authoritative.
- Navigation labels must use centralized reusable UI text.
- Navigation icons should use Element Plus Icons unless a documented exception
  is approved.

## Responsive Shell Contract

- Desktop and tablet widths use a collapsible sidebar.
- Mobile widths use an overlay navigation drawer.
- The mobile drawer must close after route selection.
- The mobile drawer must not block access to loaded route content after
  selection.
- Header controls must expose sidebar or drawer toggles as appropriate for the
  current viewport.
- Keyboard and focus behavior must satisfy the WCAG 2.1 AA baseline.

## Header Contract

The top header may include:

- current page title or breadcrumb context
- sidebar or drawer toggle
- placeholder or absent-state notification indicator
- future user/session affordance placeholder
- global feedback placement

The header must not define authentication screens, logout behavior, account
recovery behavior, or live notification data in this slice.

## Dashboard Placeholder Contract

Dashboard regions in this slice are placeholder-only:

- summary cards show placeholder, empty, unavailable, loading, or error states
- recent activity shows placeholder, empty, unavailable, loading, or error states
- notification region shows placeholder or absent-state behavior
- no live metric, count, status, timestamp, actor, or resource data may be consumed

Live dashboard data remains blocked until later feature specs and OpenAPI
contracts approve it.

## Quick Action Contract

- Quick actions must link only to approved routes or workflows.
- Quick actions must be hidden when permission is absent.
- Quick actions must be hidden when route or workflow approval is missing.
- Quick action labels must use centralized reusable UI text.
- Quick action icons should use Element Plus Icons unless a documented
  exception is approved.

## Feedback State Contract

The shell and dashboard must support these states where applicable:

- loading
- empty
- error
- forbidden
- unauthorized
- session-expired
- tenant-mismatch
- unavailable

Unauthorized, forbidden, and session-expired states are rendered as shell-level
states in this slice. Login, logout, and recovery screens remain future work.

## State Boundary Contract

Frontend state may own:

- sidebar collapsed state
- mobile drawer open state
- active navigation item
- notification panel placeholder visibility
- shell feedback state

Frontend state must not:

- own remote dashboard data
- bypass services for future API calls
- persist sensitive tenant, permission, or personal data beyond approved
  session behavior

## API Contract

No OpenAPI or `/api/v1` change is approved by this feature.

If future implementation needs live dashboard, recent activity, notification,
or module workflow data, implementation must block until:

- the feature spec approves the behavior
- OpenAPI documents the endpoint, fields, statuses, errors, auth semantics, and
  tenant semantics
- frontend services map the approved contract before pages or components
  consume it

## Accessibility, Localization, and Observability Contract

- Reusable shell, navigation, header, dashboard, quick action, and feedback UI
  must target WCAG 2.1 AA.
- Reusable text must be centralized for Vue I18n.
- Shell and dashboard failures must flow through normalized frontend error
  handling and user-action trace points without exposing sensitive data.
