# Repository Consumption Guide

## Purpose

Backend and frontend repositories consume `schoolmaster-specs` as the source of
truth for product scope, API contracts, architecture decisions, and
implementation constraints.

## Submodule Use

When included as a git submodule, this repository should be treated as
read-only by implementation repositories. Changes to requirements, contracts,
or decisions should be proposed in `schoolmaster-specs` first, then consumed by
backend and frontend work after approval.

Recommended submodule path in implementation repositories:

```text
vendor/schoolmaster-specs/
```

## Source Priority

1. `AGENTS.md` defines operational behavior for agents and contributors.
2. `specs/*/spec.md` defines approved product behavior and business rules.
3. `specs/*/contracts/openapi.yaml` defines feature-level API behavior before
   promotion.
4. `api/openapi.yaml` is the repository-level aggregate publication target.
5. `docs/` defines implementation guidance.
6. `decisions/` defines durable architecture decisions.

If these sources conflict, update `schoolmaster-specs` before implementation.
Do not resolve conflicts inside backend or frontend repositories by inventing
behavior.

## Current Active Contract

The active foundation contract is:

```text
specs/001-schoolmaster-platform/contracts/openapi.yaml
```

The aggregate contract is:

```text
api/openapi.yaml
```

Until feature behavior is promoted into the aggregate, backend and frontend
work should link to the feature contract operation IDs it implements or
consumes.

## Contract Validation

Validate OpenAPI contracts from the specification repository root:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

The command uses `redocly.yaml` to validate the aggregate contract and the
active feature contract. Contract behavior should not be promoted or consumed
for implementation until validation passes or an explicit exception is
documented.

## Cross-Repository Traceability

- Use the feature id, such as `001-schoolmaster-platform`, in related branches,
  issues, pull requests, and implementation notes.
- Backend pull requests must link to the OpenAPI operation IDs implemented.
- Frontend pull requests must link to the OpenAPI operation IDs consumed.
- Contract-changing backend or frontend work must first land the matching
  `schoolmaster-specs` change.

## Boundaries

- Do not import source code directly from implementation repositories.
- Do not add implementation-only assumptions to this repository.
- Do not change API behavior without updating the relevant OpenAPI contract.
- Do not change business rules without updating the relevant feature spec.
