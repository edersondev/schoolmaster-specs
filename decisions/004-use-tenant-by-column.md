# ADR 004: Use Tenant by Column

## Status

Accepted

## Context

SchoolMaster is a multi-tenant SaaS and needs a default tenancy strategy that
is simple to reason about across specifications, contracts, and code.

## Decision

Use a tenant-by-column strategy for multi-tenancy unless a future ADR replaces
it. For SchoolMaster v1, `School` is the tenant root and school-owned records
use `school_id` as the concrete tenant column. The term `tenant_id` may be used
only as a generic architecture description, not as the required v1 column name.

## Consequences

- Tenant-owned records must carry tenant identity directly or through approved
  ownership rules
- Backend authorization and query logic must enforce tenant isolation
- OpenAPI and documentation should reflect tenant-scoped behavior where relevant
- Specifications, data models, contracts, and implementation tasks should use
  `school_id` consistently for v1 school-scoped records
