# Frontend Guidelines

## Target

Frontend implementation targets the `schoolmaster-frontend` Vue 3 SPA
repository.

## Working Principles

- Use Vue 3 Composition API
- Use Pinia for shared feature state
- Keep API access in a service layer
- Consume only published OpenAPI-backed endpoints
- Keep business rules sourced from approved specifications

## Contract Alignment

The frontend must treat OpenAPI as the backend contract and should not depend
on undocumented payloads or routes.

## State and Module Conventions

- Organize feature code by module under `src/modules/`.
- Keep HTTP access in `src/services/` and keep services free of UI state.
- Use Pinia stores for session state, tenant context, role-aware navigation,
  and feature state that is shared across views.
- Restore tenant context only from authenticated API responses or approved
  persisted session metadata; do not invent tenant scope client-side.

## UI Composition Standards

- Build school administration, teacher, student, and reporting screens from the
  approved product workflows and permissions.
- Treat client-side role checks as navigation and visibility aids only; backend
  authorization remains authoritative.
- Keep reusable composables focused on view coordination, not business rules
  that belong in specifications or backend services.

## Error and Loading Patterns

- Map API error envelopes consistently for validation, unauthorized, forbidden,
  inactive-record, tenant-mismatch, and not-found outcomes.
- Preserve enough error detail for users to correct validation problems without
  exposing tenant or authorization internals.
- Loading states should be scoped to the service/store action being performed
  so one module does not block unrelated tenant-safe workflows.
