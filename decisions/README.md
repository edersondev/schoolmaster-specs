# Architecture Decision Records

This directory stores durable architecture decisions for SchoolMaster.

## Index

| ADR | Status | Decision |
| --- | --- | --- |
| [001](001-use-separated-repositories.md) | Accepted | Use separate repositories for specifications, backend, and frontend. |
| [002](002-use-laravel-native-auth.md) | Accepted | Use Laravel-native authentication mechanisms. |
| [003](003-use-mysql.md) | Accepted | Use MySQL as the primary transactional database. |
| [004](004-use-tenant-by-column.md) | Accepted | Use tenant-by-column multi-tenancy with `School` as the v1 tenant root and `school_id` for school-owned records. |

## Rules

- Number ADRs with a three-digit prefix.
- Keep each ADR focused on one durable decision.
- Update or supersede ADRs with a new ADR; do not rewrite accepted history
  unless correcting non-behavioral wording.
- Cross-reference affected specs, docs, and OpenAPI contracts when a decision
  changes implementation behavior.
