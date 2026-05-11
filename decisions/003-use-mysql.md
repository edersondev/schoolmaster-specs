# ADR 003: Use MySQL

## Status

Accepted

## Context

SchoolMaster requires a relational datastore for tenant-aware transactional
data.

## Decision

Use MySQL as the primary transactional database unless a future ADR changes
this decision.

## Consequences

- Data design should assume relational modeling
- Backend persistence patterns should align with MySQL capabilities
- Performance and indexing guidance will be refined as modules mature
