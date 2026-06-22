# Data Model: System Administrator Shell and Dashboard Foundation

This feature does not introduce database entities. The model below defines
frontend concepts, UI state, and contract boundaries for the System
Administrator shell and placeholder dashboard.

## AdminShell

**Purpose**: Authenticated System Administrator workspace frame.

**Fields**:

- `layoutId`: Stable shell layout identifier for route metadata.
- `contentRegion`: Route content slot for admin pages.
- `pageContext`: Current title, breadcrumb trail, and optional page metadata.
- `globalFeedback`: Loading, empty, error, forbidden, unauthorized,
  session-expired, tenant-mismatch, and unavailable states.
- `desktopSidebarState`: Expanded or collapsed state for desktop/tablet.
- `mobileDrawerState`: Open or closed state for mobile overlay navigation.
- `activeNavigationItem`: Current visible navigation item matched from route metadata.

**Relationships**:

- Owns `NavigationItem`, `HeaderControl`, `ShellFeedbackState`, and dashboard placement.
- Uses `RouteMetadata` to select layout, active item, permissions, title, and breadcrumb.
- Uses `ShellState` for responsive and cross-route UI state.

**Validation rules**:

- Must not define login, logout, account recovery, CRUD resource pages,
  reporting workflows, or platform-support workflows.
- Must keep route content accessible when mobile drawer opens or closes.
- Must show shell-level denial states for unauthorized, forbidden, and
  session-expired access.

## RouteMetadata

**Purpose**: Contract for attaching routes to the admin shell.

**Fields**:

- `layout`: Must select the System Administrator shell for admin routes.
- `title`: Localized page title key or approved title source.
- `breadcrumb`: Ordered breadcrumb metadata for page context.
- `permissions`: Permission keys required for visibility and access checks.
- `sidebar`: Sidebar section, order, icon, and visibility metadata.
- `requiresAuth`: Protected-route flag.

**Relationships**:

- Feeds `NavigationItem`, `AdminShell`, and route guard behavior.
- Must align with later module specs when modules attach routes.

**Validation rules**:

- Must not reference unapproved routes or workflows.
- Must hide unauthorized navigation items derived from metadata.
- Must not invent backend authorization semantics.

## NavigationItem

**Purpose**: Visible sidebar entry or section.

**Fields**:

- `label`: Centralized reusable UI text key.
- `destination`: Approved route name or path.
- `icon`: Element Plus Icon identifier or documented exception.
- `permissionRule`: Permission key or rule used for client-side visibility.
- `order`: Display order within the sidebar.
- `activeState`: Whether the item maps to the current route.
- `hiddenState`: Hidden when permission or route approval is missing.

**Relationships**:

- Derived from `RouteMetadata`.
- Rendered by `AdminShell`.

**Validation rules**:

- Must be hidden when user lacks permission.
- Must be hidden when destination route is not approved.
- Must not be used as authoritative authorization.

## HeaderControl

**Purpose**: Top-header surface for global shell affordances.

**Fields**:

- `pageTitle`: Current localized title.
- `drawerToggle`: Mobile navigation control.
- `sidebarToggle`: Desktop/tablet collapse control.
- `notificationIndicator`: Placeholder or absent-state indicator in this slice.
- `userAffordance`: Non-authentication placeholder for future session UI.
- `feedbackAnchor`: Placement for global feedback messages.

**Relationships**:

- Uses `ShellState` for drawer/sidebar controls.
- Uses `ShellFeedbackState` for global feedback.

**Validation rules**:

- Must not define login, logout, account recovery, or live notification data.
- Icon-only controls must have accessible labels.

## DashboardSummaryCard

**Purpose**: Placeholder-only summary card structure.

**Fields**:

- `slotId`: Stable placeholder region identifier.
- `label`: Centralized reusable UI text key.
- `placeholderState`: Empty, unavailable, loading, or error.
- `futureContractDependency`: Optional later contract dependency note.

**Relationships**:

- Rendered by `AdminDashboard`.
- Must remain placeholder-only until later contract approval.

**Validation rules**:

- Must not consume live metrics, labels, values, or statuses in this slice.
- Must not invent dashboard counts or backend-derived status meanings.

## RecentActivityItem

**Purpose**: Placeholder structure for future operational activity.

**Fields**:

- `slotId`: Stable recent-activity placeholder identifier.
- `placeholderState`: Empty, unavailable, loading, or error.
- `futureContractDependency`: Optional later activity contract dependency note.

**Relationships**:

- Rendered by `AdminDashboard`.

**Validation rules**:

- Must not consume live activity timestamps, actor labels, action labels, or
  resource names in this slice.

## QuickAction

**Purpose**: Dashboard shortcut to an approved destination.

**Fields**:

- `label`: Centralized reusable UI text key.
- `destination`: Approved route name or path.
- `icon`: Element Plus Icon identifier or documented exception.
- `permissionRule`: Permission key or rule used for client-side visibility.
- `order`: Display order in the quick-action region.
- `hiddenState`: Hidden when permission, route approval, or workflow approval is missing.

**Relationships**:

- Uses `RouteMetadata`.
- Rendered by `AdminDashboard`.

**Validation rules**:

- Must link only to approved routes or workflows.
- Must be hidden for unauthorized users or unapproved destinations.

## ShellFeedbackState

**Purpose**: Standard shell or dashboard feedback state.

**Fields**:

- `state`: Loading, empty, error, forbidden, unauthorized, session-expired,
  tenant-mismatch, unavailable.
- `messageKey`: Centralized reusable UI text key.
- `severity`: Informational, warning, or error.
- `action`: Optional approved recovery action, excluding login/recovery screens in this slice.

**Relationships**:

- Used by `AdminShell`, `HeaderControl`, and `AdminDashboard`.

**Validation rules**:

- Must not expose sensitive tenant, permission, or personal data.
- Must not define authentication screens.

## ShellState

**Purpose**: Client UI state for shell behavior across route changes.

**Fields**:

- `sidebarCollapsed`: Desktop/tablet sidebar collapsed state.
- `mobileDrawerOpen`: Mobile drawer state.
- `activeRouteKey`: Current active route key.
- `notificationPanelOpen`: Placeholder-only notification panel visibility.

**Relationships**:

- Coordinates `AdminShell`, `NavigationItem`, and `HeaderControl`.

**Validation rules**:

- Must not own remote data transport.
- Must not persist sensitive session, tenant, or permission data beyond approved session behavior.

## AdminDashboard

**Purpose**: System Administrator landing surface inside the shell.

**Fields**:

- `summaryRegion`: Placeholder-only summary cards.
- `recentActivityRegion`: Placeholder-only recent activity.
- `quickActionRegion`: Approved and permission-visible quick actions only.
- `notificationRegion`: Placeholder or absent-state notification surface.
- `feedbackRegion`: Loading, empty, unavailable, error, or forbidden states.

**Relationships**:

- Uses `DashboardSummaryCard`, `RecentActivityItem`, `QuickAction`, and
  `ShellFeedbackState`.

**Validation rules**:

- Must not consume live dashboard data in this slice.
- Must not expose unapproved quick actions.
