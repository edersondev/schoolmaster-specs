# Multi-Tenant Strategy

## Current Decision

SchoolMaster multi-tenancy uses a tenant-by-column strategy. For v1, `School`
is the tenant root and school-owned records use `school_id` as the tenant
column unless ownership is inherited through a strictly school-owned parent
record with documented traversal rules.

This follows [ADR 004](../decisions/004-use-tenant-by-column.md).

## Scope

- Tenant-owned data is isolated by school identity.
- Backend queries, repositories, services, policies, and tests must enforce
  tenant boundaries.
- Frontend behavior must respect tenant-scoped data flows exposed by the API
  and must not infer authorization from client-side state alone.
- OpenAPI documents tenant-context behavior for school-scoped endpoints.

## Repository Impact

- Specifications define tenant rules and constraints
- OpenAPI documents tenant-aware contract behavior
- Laravel API enforces tenant isolation
- Vue 3 SPA consumes tenant-safe endpoints

## Tenant Context Resolution

- School-scoped requests resolve an active tenant before module-specific
  business logic runs.
- The API contract uses the `X-School-Id` header for explicit tenant context
  where a request is not already bound to exactly one active school.
- Requests with missing, mismatched, inactive, or unauthorized tenant context
  are rejected before data access.

## Platform Access

- System administrators operate at platform scope for provisioning and
  reviewing school tenants.
- Platform access is not an implicit bypass for school-scoped module actions.
  Any cross-tenant override must be explicitly documented in the specification,
  OpenAPI contract, authorization policy, and regression tests.

## Shared and Tenant-Owned Resources

- `Permission` records are shared capability definitions controlled at platform
  scope and filtered by role scope.
- School-scoped roles, users, academic structures, guardians, content,
  questionnaires, learning sets, grades, attendance, and report requests are
  tenant-owned.
- Tenant-owned records use status fields and soft-delete support where recovery
  or audit history is relevant.
