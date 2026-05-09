# schoolmaster-specs

[![GitHub Repo](https://img.shields.io/badge/repo-schoolmaster--specs-1f2937?logo=github)](https://github.com/edersondev/schoolmaster-specs)
[![Branch](https://img.shields.io/badge/branch-main-0f766e)](https://github.com/edersondev/schoolmaster-specs/tree/main)
[![Last Commit](https://img.shields.io/github/last-commit/edersondev/schoolmaster-specs/main)](https://github.com/edersondev/schoolmaster-specs/commits/main)

Product specifications, business rules, architecture decisions, and OpenAPI
contracts for SchoolMaster, a multi-tenant school management SaaS.

Specification and contract repository for the SchoolMaster platform. This repo
defines what the product should do, how modules are bounded, and how backend
and frontend teams integrate through versioned API contracts.

## Getting Started

For contributors working on a new feature:

1. Review the active feature folder under `specs/`.
2. Read `spec.md` to understand scope, actors, and acceptance criteria.
3. Read `plan.md` and `research.md` before changing contracts or structure.
4. Update `contracts/openapi.yaml` before backend or frontend implementation.
5. Keep requirements, contracts, and task breakdown aligned.

Current feature entry point:

- [SchoolMaster Platform Foundation](specs/001-schoolmaster-platform/spec.md)

## Purpose

This repository is the source of truth for:

- Product scope and user stories
- Business rules and domain boundaries
- Cross-repository architecture decisions
- Versioned API contracts used by backend and frontend teams
- Delivery planning artifacts for implementation

SchoolMaster is designed as three separate repositories:

- `schoolmaster-specs`: specifications and contracts
- `schoolmaster-backend`: Laravel API
- `schoolmaster-frontend`: Vue 3 SPA

The backend is API-first, and the frontend is expected to consume only the
published REST API contracts.

## Current Scope

The current feature set defines the SchoolMaster platform foundation for a
multi-tenant school management SaaS with these launch modules:

- Authentication and authorization
- School management
- User management
- Roles and permissions
- Academic years and academic periods
- Guardian management
- Teacher content management
- Questionnaires and learning sets
- Grades and attendance
- Reports

## Repository Structure

```text
.
├── AGENTS.md
└── specs/
    └── 001-schoolmaster-platform/
        ├── checklists/
        │   └── requirements.md
        ├── contracts/
        │   └── openapi.yaml
        ├── data-model.md
        ├── plan.md
        ├── quickstart.md
        ├── research.md
        ├── spec.md
        └── tasks.md
```

## Key Documents

- [Feature specification](specs/001-schoolmaster-platform/spec.md): product
  vision, actors, requirements, non-goals, and success criteria
- [Implementation plan](specs/001-schoolmaster-platform/plan.md): architecture,
  delivery sequencing, tenancy strategy, and contract strategy
- [Research notes](specs/001-schoolmaster-platform/research.md): major planning
  decisions and rationale
- [Data model](specs/001-schoolmaster-platform/data-model.md): domain entities
  and relationship boundaries
- [OpenAPI contract](specs/001-schoolmaster-platform/contracts/openapi.yaml):
  initial versioned REST API definition
- [Tasks](specs/001-schoolmaster-platform/tasks.md): dependency-ordered work
  plan for implementation
- [Quickstart](specs/001-schoolmaster-platform/quickstart.md): validation flow
  before backend and frontend implementation

## Working Model

1. Define or update requirements and business rules in this repository.
2. Publish or refine the OpenAPI contract before implementation.
3. Implement matching behavior in `schoolmaster-backend`.
4. Build frontend flows in `schoolmaster-frontend` against approved endpoints.
5. Keep cross-repository traceability with the shared feature identifier.

## Domain Principles

- Multi-tenant isolation is anchored on `School` as the tenant root.
- School-scoped data must not leak across tenants.
- API routes are versioned under `/api/v1`.
- UUIDs are used for public entity identifiers where applicable.
- Status-driven lifecycle management is preferred over destructive removal.
- Business rules are documented before implementation begins.

## Status

This repository currently contains the initial platform foundation artifacts for
feature `001-schoolmaster-platform`.
