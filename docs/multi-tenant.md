# Multi-Tenant Strategy

## Current Decision

SchoolMaster multi-tenancy must use the `tenant_id` column strategy unless a
future ADR changes that decision.

## Scope

- Tenant-owned data is isolated by tenant identifier
- Backend queries and authorization must enforce tenant boundaries
- Frontend behavior must respect tenant-scoped data flows exposed by the API

## Repository Impact

- Specifications define tenant rules and constraints
- OpenAPI documents tenant-aware contract behavior
- Laravel API enforces tenant isolation
- Vue 3 SPA consumes tenant-safe endpoints

## Pending Details

- Tenant context resolution
- Platform administrator cross-tenant rules
- Shared versus tenant-owned resources
