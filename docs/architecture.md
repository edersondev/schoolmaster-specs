# Architecture Overview

## Purpose

SchoolMaster uses this repository as the source of truth for specifications,
contracts, architecture decisions, and implementation guidance.

See [repository-consumption.md](repository-consumption.md) for how backend and
frontend repositories consume this repository through git submodules.

## Repository Model

- `schoolmaster-specs` stores approved requirements, contracts, and decisions
- `schoolmaster-backend` implements the Laravel API
- `schoolmaster-frontend` implements the Vue 3 SPA

Durable repository and storage decisions are recorded in
[`decisions/`](../decisions/README.md).

## Integration Boundary

The OpenAPI contract is the formal boundary between frontend and backend.
Implementation work should align to published contract definitions.

Feature-level contracts live under `specs/*/contracts/` before approved
behavior is promoted into `api/openapi.yaml`.

## Architectural Direction

- API-first delivery
- Laravel API as the backend target
- Vue 3 SPA as the frontend target
- Multi-tenant data isolation by tenant-by-column strategy, with `School` as
  the v1 tenant root and `school_id` as the concrete tenant column for
  school-owned records

## Module Boundaries

- Platform administration: school tenant provisioning and platform-scoped role
  management.
- School administration: users, roles, permissions, academic years, academic
  periods, guardians, and school profile operations inside one tenant.
- Teacher operations: content folders, uploaded instructional content,
  questionnaires, learning sets, grades, and attendance.
- Student operations: assigned learning sets, own grades, and own attendance.
- Reporting: tenant-bound launch reports for attendance, grades, academic
  structure, and school activity summaries.

## Cross-Repository Release Workflow

1. Update approved requirements, business rules, and OpenAPI contracts in
   `schoolmaster-specs`.
2. Implement contract-compliant behavior in `schoolmaster-backend`.
3. Consume only published `/api/v1` contract behavior in
   `schoolmaster-frontend`.
4. Link backend and frontend work to the relevant specification feature id and
   OpenAPI operation IDs.

## Non-Functional Architecture Constraints

- Tenant isolation is enforced before module-specific data access.
- OpenAPI is the source of truth for payloads, status codes, response envelopes,
  tenant semantics, pagination, filtering, sorting, and error responses.
- Uploaded teacher content uses private tenant-scoped object storage and is
  served only through authorized API access.
- Report generation may run asynchronously when synchronous generation would
  degrade interactive workflows.
