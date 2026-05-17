# Quickstart: Backend School Administration Foundation

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-school-admin.md](./contracts/backend-school-admin.md) for the approved operation boundary.
- Review `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` for concrete request and response schemas.

## Delivery Boundary

Implement only these operation IDs in this slice:

- `listUsers`
- `createUser`
- `listRoles`
- `createRole`
- `listPermissions`
- `listAcademicYears`
- `createAcademicYear`
- `listAcademicPeriods`
- `createAcademicPeriod`
- `listGuardians`
- `createGuardian`

Do not implement update, detail, delete, deactivate, invitation, password-management, student self-service, teacher workflow, or reporting behavior until the specification and OpenAPI contracts are expanded.

## Validation Walkthrough

### 1. Contract readiness

- Confirm both aggregate and platform OpenAPI contracts contain the operation IDs listed above.
- Confirm every implemented route uses the documented `/api/v1` path and request schema.
- Confirm list operations use documented pagination, status filters, academic-year filters where present, and response envelopes only.
- Confirm no backend route exposes fields, filters, sorts, or errors absent from OpenAPI.

### 2. Tenant and authorization validation

- Confirm all school-scoped requests resolve an active `school_id` context before data access.
- Confirm missing, inactive, mismatched, and unauthorized tenant contexts fail before service logic runs.
- Confirm platform access does not imply school-scoped access.
- Confirm school administrators cannot list or create records for another school.

### 3. User, role, and permission validation

- Confirm user creation rejects cross-tenant roles, inactive roles, and incompatible role scopes.
- Confirm school-scoped roles require an active resolved school context.
- Confirm school roles cannot include platform-only permissions.
- Confirm permission listing exposes only definitions visible to the requester.
- Confirm direct per-user permission assignment is not available.

### 4. Academic structure validation

- Confirm academic years are created only inside the resolved school.
- Confirm academic year dates are valid.
- Confirm academic periods reference a same-school academic year.
- Confirm academic period dates fit inside the parent academic year.
- Confirm academic period sequence is unique within the academic year.

### 5. Guardian validation

- Confirm guardians are created only inside the resolved school.
- Confirm required relationship fields are validated.
- Confirm optional student profile references are active and same-school.
- Confirm invalid student profile references reject the whole request without partial associations.

### 6. Response-shape validation

- Confirm successful creates use `SuccessEnvelope`.
- Confirm lists use `PaginatedEnvelope`.
- Confirm validation, unauthorized, forbidden, tenant-mismatch, inactive-record, and not-found cases use documented error envelopes.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
php artisan test
```

Implementation PRs should record the contract validation result, test result, feature id `003-backend-school-admin`, and the operation IDs implemented.

## Exit Criteria for Planning

- The implementation boundary maps to the approved OpenAPI operation IDs.
- Tenant and authorization rules are explicit for every affected entity.
- Data model rules are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, tenant isolation, authorization, validation, inactive statuses, academic rules, and guardian associations.
