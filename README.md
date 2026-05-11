# SchoolMaster Specifications

SchoolMaster specifications repository for product scope, architecture
decisions, API contracts, and technical documentation.

This repository is the source of truth for:

- Product specifications and approved feature scope
- Architecture decisions and technical guidelines
- OpenAPI contracts shared across teams
- Cross-repository delivery documentation

SchoolMaster is organized as separate repositories:

- `schoolmaster-specs`: requirements, contracts, and decisions
- `schoolmaster-backend`: Laravel API implementation
- `schoolmaster-frontend`: Vue 3 SPA implementation

The OpenAPI contract is the integration boundary between frontend and backend.
Changes to behavior or payloads should start here before implementation.

## Target Implementations

- Backend: Laravel API
- Frontend: Vue 3 SPA
- Contract: OpenAPI

## Repository Structure

```text
.
├── api/
├── decisions/
├── diagrams/
├── docs/
├── specs/
└── templates/
```

## Core Documentation

- [API contract](api/openapi.yaml)
- [Architecture overview](docs/architecture.md)
- [Backend guidelines](docs/backend-guidelines.md)
- [Frontend guidelines](docs/frontend-guidelines.md)
- [Multi-tenancy](docs/multi-tenant.md)
- [Security](docs/security.md)
- [Naming conventions](docs/naming-conventions.md)

## Spec Kit Artifacts

Existing Spec Kit feature artifacts remain under `specs/`.

Current foundation feature:

- [001-schoolmaster-platform](specs/001-schoolmaster-platform/spec.md)

## Working Rules

1. Update approved requirements in `specs/` first.
2. Update `api/openapi.yaml` when API behavior changes.
3. Keep backend and frontend aligned with the published contract.
4. Record durable technical decisions in `decisions/`.
5. Add pending details as explicit placeholders instead of inventing rules.
