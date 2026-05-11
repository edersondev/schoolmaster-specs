# ADR 002: Use Laravel Native Authentication

## Status

Accepted

## Context

The backend target is a Laravel API and requires a maintainable authentication
approach aligned with the framework.

## Decision

Use Laravel-native authentication mechanisms in the backend repository unless a
future decision supersedes this choice.

## Consequences

- Authentication design stays close to framework conventions
- OpenAPI must document auth expectations for clients
- Frontend integration follows the published backend contract
