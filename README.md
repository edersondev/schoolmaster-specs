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

- [Repository consumption guide](docs/repository-consumption.md)
- [Next actions](docs/next-actions.md)
- [Architecture overview](docs/architecture.md)
- [Backend guidelines](docs/backend-guidelines.md)
- [Frontend guidelines](docs/frontend-guidelines.md)
- [Multi-tenancy](docs/multi-tenant.md)
- [Security](docs/security.md)
- [Naming conventions](docs/naming-conventions.md)
- [Documentation index](docs/README.md)
- [ADR index](decisions/README.md)

## OpenAPI Contracts

- Active feature contract:
  [specs/001-schoolmaster-platform/contracts/openapi.yaml](specs/001-schoolmaster-platform/contracts/openapi.yaml)
- Repository aggregate contract:
  [api/openapi.yaml](api/openapi.yaml)

Feature contracts are authored under `specs/*/contracts/`. The repository-level
aggregate in `api/openapi.yaml` is the stable publication target for promoted
contract behavior.

## Spec Kit Artifacts

Existing Spec Kit feature artifacts remain under `specs/`.

Current foundation feature:

- [001-schoolmaster-platform](specs/001-schoolmaster-platform/spec.md)

## Consuming This Repository

Backend and frontend repositories may include `schoolmaster-specs` as a git
submodule. Consuming repositories must read requirements from `specs/`, API
behavior from the active OpenAPI contract, implementation guidance from
`docs/`, and durable architecture decisions from `decisions/`.

## Working Rules

1. Update approved requirements in `specs/` first.
2. Update the relevant feature OpenAPI contract in `specs/*/contracts/` when
   API behavior changes.
3. Promote approved contract behavior into `api/openapi.yaml` when maintaining
   a repository-level aggregate contract.
4. Keep backend and frontend aligned with the published contract.
5. Record durable technical decisions in `decisions/`.
6. Add pending details as explicit placeholders instead of inventing rules.
