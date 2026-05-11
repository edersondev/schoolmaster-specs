# ADR 001: Use Separated Repositories

## Status

Accepted

## Context

SchoolMaster needs a stable source of truth for specifications and contracts
while allowing backend and frontend implementation to evolve independently.

## Decision

Use separate repositories for:

- Specifications and contracts
- Laravel API backend
- Vue 3 SPA frontend

## Consequences

- Cross-repository changes require explicit traceability
- OpenAPI becomes the shared contract boundary
- Documentation and implementation lifecycles remain decoupled
