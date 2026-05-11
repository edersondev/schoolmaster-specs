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

Tenant-aware backend code must follow the `tenant_id` column strategy defined
in repository decisions unless a future ADR replaces it.

## Pending Details

- Authentication flow details
- Module-specific service boundaries
- Error envelope conventions promoted from approved contracts
