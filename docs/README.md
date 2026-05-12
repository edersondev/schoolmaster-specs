# Documentation Index

This directory contains implementation-oriented guidance for repositories that
consume `schoolmaster-specs`.

## Guides

- [Repository consumption](repository-consumption.md): how backend and frontend
  repositories consume this repository through git submodules.
- [Next actions](next-actions.md): current readiness status, decision gates,
  and recommended sequencing.
- [Architecture overview](architecture.md): repository model, module
  boundaries, release workflow, and non-functional constraints.
- [Backend guidelines](backend-guidelines.md): Laravel API implementation
  expectations.
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
