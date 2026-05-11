# Frontend Guidelines

## Target

Frontend implementation targets the `schoolmaster-frontend` Vue 3 SPA
repository.

## Working Principles

- Use Vue 3 Composition API
- Keep API access in a service layer
- Consume only published OpenAPI-backed endpoints
- Keep business rules sourced from approved specifications

## Contract Alignment

The frontend must treat OpenAPI as the backend contract and should not depend
on undocumented payloads or routes.

## Pending Details

- State management conventions by module
- UI composition standards
- Error and loading patterns
