# Architecture Overview

## Purpose

SchoolMaster uses this repository as the source of truth for specifications,
contracts, architecture decisions, and implementation guidance.

## Repository Model

- `schoolmaster-specs` stores approved requirements, contracts, and decisions
- `schoolmaster-backend` implements the Laravel API
- `schoolmaster-frontend` implements the Vue 3 SPA

## Integration Boundary

The OpenAPI contract is the formal boundary between frontend and backend.
Implementation work should align to published contract definitions.

## Architectural Direction

- API-first delivery
- Laravel API as the backend target
- Vue 3 SPA as the frontend target
- Multi-tenant data isolation by tenant column strategy

## Pending Details

- Module decomposition details
- Cross-repository release workflow
- Non-functional architecture constraints
