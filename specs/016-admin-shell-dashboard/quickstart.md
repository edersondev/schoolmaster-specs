# Quickstart: System Administrator Shell and Dashboard Foundation

Use this checklist when planning or reviewing `schoolmaster-frontend`
implementation for roadmap item 2.

## 1. Confirm Scope

- Start from `specs/016-admin-shell-dashboard/spec.md`.
- Use `specs/015-frontend-architecture-baseline/` as the governing frontend
  architecture baseline.
- Use `docs/frontend-admin-system-architecture.md`,
  `docs/frontend-architecture.md`, `docs/frontend-guidelines.md`, and
  `docs/naming-conventions.md` as supporting guidance.
- Do not add authentication screens, CRUD resource pages, reporting workflows,
  platform-support workflows, backend code, OpenAPI changes, or TypeScript.

## 2. Verify Shell Behavior

The implementation should provide:

- System Administrator layout frame.
- Sidebar navigation.
- Top header.
- Route content region.
- Page title or breadcrumb context.
- Global feedback placement.
- Desktop/tablet collapsible sidebar.
- Mobile overlay drawer.
- Shell-level unauthorized, forbidden, and session-expired states.

The mobile drawer must close after route selection.

## 3. Verify Permission and Route Visibility

- Hide sidebar items when permission is absent.
- Hide quick actions when permission is absent.
- Hide quick actions when the target route or workflow is not approved.
- Treat client-side visibility as UX only; backend authorization remains
  authoritative.
- Do not show disabled or "coming soon" controls for unapproved workflows in
  this slice.

## 4. Verify Dashboard Placeholder Rules

Dashboard summary, recent activity, and notification regions are
placeholder-only in this slice.

Allowed states:

- placeholder
- empty
- unavailable
- loading
- error
- absent notification state

Blocked behavior:

- live summary metrics
- live recent activity items
- live notification counts
- undocumented statuses
- invented backend-derived labels or values

## 5. Verify Route Metadata

Admin shell routes should define:

- layout selection
- title or page context
- breadcrumbs where applicable
- permissions
- sidebar placement
- auth requirement

Future modules attach to the shell only after their feature specs approve route
and workflow behavior.

## 6. Verify UI Baseline

- Use Element Plus primitives for layout, menu, card, button, feedback,
  skeleton, and empty states.
- Use PascalCase Element Plus tags.
- Use Element Plus Icons by default.
- Use Tailwind for layout, spacing, responsiveness, and restrained visual
  refinement.
- Centralize reusable UI text for Vue I18n.
- Target WCAG 2.1 AA for shell, navigation, header, dashboard, quick actions,
  and feedback states.

## 7. Verify API-First Boundary

No live dashboard API consumption is approved.

Before any later feature consumes dashboard, activity, notification, or module
data, confirm:

- behavior is approved by a feature spec
- endpoint is documented under `/api/v1`
- request and response fields are documented
- error, auth, authorization, and tenant semantics are documented
- frontend service mapping exists before pages/components consume the data

## 8. Suggested Review Commands

After implementation exists in `schoolmaster-frontend`, use checks like:

```bash
rg "axios" src/layouts src/pages src/components
rg "<el-" src
rg "Coming soon|coming soon" src
rg "notificationCount|recentActivity|summaryMetric" src
```

Expected review result:

- no direct Axios usage in layouts/pages/components
- no kebab-case Element Plus tags
- no "coming soon" quick actions for unapproved workflows
- no live dashboard data names unless a later contract approves them

Affected frontend routes, stores, composables, and shell/dashboard behavior
should have Vitest coverage when implemented in `schoolmaster-frontend`.
