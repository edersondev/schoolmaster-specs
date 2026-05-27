# Backend Guidelines

## Target

Backend implementation targets the `schoolmaster-backend` Laravel API
repository.

## Working Principles

- Keep controllers thin
- Place business logic in services
- Use requests for validation
- Use API resources for response shaping
- Align behavior with the published OpenAPI contract

## Multi-Tenancy

Tenant-aware backend code must follow the tenant-by-column strategy defined in
repository decisions. For v1 school-owned records, the concrete tenant column
is `school_id`; use `tenant_id` only as a generic architecture term unless a
future ADR introduces a different tenant root.

Backend implementation must enforce tenant scope in services, query scopes or
repositories, policies, API resources, and tests. Requests with missing,
mismatched, inactive, or unauthorized school context must fail before
module-specific business logic runs.

## Authentication and Authorization

- Use Laravel-native authentication mechanisms aligned to the published
  OpenAPI contract.
- Keep platform-scope and school-scope authorization paths explicit.
- Do not grant system administrators an implicit bypass for school-scoped
  module actions.
- Expose role and permission information through API resources only as defined
  by the contract.

## Module Boundaries

- Keep controllers thin and route work through feature services.
- Use Form Requests for validation and API Resources for response shaping.
- Use DTOs when request input or service input has multiple coordinated fields.
- Use repositories or explicit query objects for complex tenant-scoped access.

## Backend Test Command

Run the Laravel test suite inside the backend application container:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Do not run `php artisan test` directly on the host for documented backend
verification, because the backend runtime and database drivers are provided by
the Docker container.

## Response and Error Conventions

Backend responses must follow the approved OpenAPI envelopes for successful
responses, validation failures, authorization failures, inactive-record
handling, tenant mismatches, and not-found outcomes.
