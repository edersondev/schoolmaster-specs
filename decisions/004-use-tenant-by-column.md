# ADR 004: Use Tenant by Column

## Status

Accepted

## Context

SchoolMaster is a multi-tenant SaaS and needs a default tenancy strategy that
is simple to reason about across specifications, contracts, and code.

## Decision

Use a `tenant_id` column strategy for multi-tenancy unless a future ADR
replaces it.

## Consequences

- Tenant-owned records must carry tenant identity directly or through approved
  ownership rules
- Backend authorization and query logic must enforce tenant isolation
- OpenAPI and documentation should reflect tenant-scoped behavior where relevant
