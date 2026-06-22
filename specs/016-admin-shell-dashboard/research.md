# Research: System Administrator Shell and Dashboard Foundation

## Decision: Keep dashboard regions placeholder-only in this slice

**Rationale**: The feature must preserve API-first delivery and no live
dashboard summary, recent activity, or notification contract is approved for
this slice. Placeholder-only regions allow the shell and dashboard composition
to be implemented without consuming undocumented backend behavior.

**Alternatives considered**: Requiring live dashboard data before shell
implementation was rejected because it would block the reusable shell on
contracts owned by later features. A hybrid live-notification approach was
rejected because notification semantics are not approved in this slice.

## Decision: Hide unauthorized navigation items and quick actions

**Rationale**: Hiding unauthorized surfaces avoids leaking unavailable modules
and keeps client-side permission checks as usability controls only. Backend
authorization remains authoritative for every protected route or action.

**Alternatives considered**: Disabled unauthorized items were rejected because
they expose unavailable capabilities and add extra messaging requirements.
Showing all items and relying only on route guards was rejected because it
creates poor navigation UX and inconsistent visibility.

## Decision: Hide quick actions until target routes and workflows are approved

**Rationale**: Quick actions should not imply workflows that are not yet
specified. Hiding unapproved actions keeps the dashboard contract-safe and
prevents placeholders from becoming accidental product commitments.

**Alternatives considered**: Disabled "coming soon" actions were rejected
because they introduce product messaging not approved by a feature spec.
Placeholder quick-action cards were rejected because they add no actionable
value in this slice.

## Decision: Use collapsible sidebar on desktop/tablet and overlay drawer on mobile

**Rationale**: A collapsible sidebar preserves admin navigation density on
larger screens while an overlay drawer keeps mobile content usable and avoids a
cramped permanent sidebar. The drawer must close after route selection.

**Alternatives considered**: Always-visible navigation on mobile was rejected
because it competes with route content. Header-only mobile navigation was
rejected because it weakens parity with sidebar sections and permission-aware
menu structure.

## Decision: Represent unauthorized, forbidden, and session-expired access through shell-level states

**Rationale**: Authentication screens are out of scope, but protected shell
routes still need deterministic denial behavior. Shell-level states give later
frontend implementation testable behavior without defining login, logout, or
account recovery screens.

**Alternatives considered**: Redirecting to a future login route was rejected
because that route is not specified in this slice. Blocking shell work until
authentication UI is specified was rejected because this shell can define its
own denial states independently.

## Decision: Use route metadata for layout, permission, title, breadcrumb, and sidebar placement

**Rationale**: Route metadata keeps layout selection and navigation visibility
traceable and lets later modules attach routes to the shell without changing
the shell contract. This follows the completed frontend architecture baseline.

**Alternatives considered**: Hardcoded navigation in the layout was rejected
because it would make later modules modify shell internals. Runtime API-driven
navigation was rejected for this slice because no navigation contract is
approved.

## Decision: Keep shell state in frontend state boundaries, not transport services

**Rationale**: Sidebar collapsed/open state, mobile drawer state, active item,
and notification panel visibility are client UI state. Pinia or composables can
coordinate shared shell behavior, while services remain reserved for approved
API consumption.

**Alternatives considered**: Local-only component state was rejected where
state must survive route changes. Service-owned shell state was rejected
because services must not own presentation state.

## Decision: Preserve baseline accessibility, localization, and observability rules

**Rationale**: Navigation, header controls, dashboard placeholders, quick
actions, and feedback states are reusable shell primitives. They must inherit
WCAG 2.1 AA, centralized reusable text, and lightweight diagnostic boundaries
from the frontend architecture baseline.

**Alternatives considered**: Deferring accessibility, localization, and
observability to later modules was rejected because later modules will reuse
this shell and dashboard foundation.
