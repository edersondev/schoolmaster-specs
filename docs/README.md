# Documentation Index

This directory contains implementation-oriented guidance for repositories that
consume `schoolmaster-specs`.

## Guides

- [Repository consumption](repository-consumption.md): how backend and frontend
  repositories consume this repository through git submodules.
- [Next actions](next-actions.md): current readiness status, decision gates,
  and recommended sequencing.
- [Backend feature roadmap](backend-feature-roadmap.md): backend feature areas
  still blocked or not yet specified, ordered by recommended sequence.
- [Frontend feature roadmap](frontend-feature-roadmap.md): frontend feature
  areas that should be specified before implementation, ordered by recommended
  sequence.
- [Architecture overview](architecture.md): repository model, module
  boundaries, release workflow, and non-functional constraints.
- [Backend guidelines](backend-guidelines.md): Laravel API implementation
  expectations.
- [Frontend architecture](frontend-architecture.md): SPA shell, module,
  routing, state, and service boundaries for `schoolmaster-frontend`.
- [Frontend admin-system architecture](frontend-admin-system-architecture.md):
  detailed System Administrator panel blueprint, folder structure, reusable
  layout, dashboard, CRUD primitives, routing, stores, shared contracts, and
  example module organization.
- [Frontend guidelines](frontend-guidelines.md): Vue 3 SPA implementation
  expectations.
- [Multi-tenancy](multi-tenant.md): tenant-by-column strategy, tenant context,
  and platform access rules.
- [Security](security.md): authentication, authorization, upload, and audit
  expectations.
- [Naming conventions](naming-conventions.md): repository, contract, entity,
  field, and operation naming rules.

## Related Sources

- Product and module scope lives in `specs/`.
- OpenAPI contracts live in `specs/*/contracts/` before promotion into `api/`.
- Durable architecture choices live in `decisions/`.
