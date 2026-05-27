# Quickstart: Backend Administration Lifecycle Management

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-administration-lifecycle.md](./contracts/backend-administration-lifecycle.md) for the proposed operation boundary and contract expansion requirements.
- Expand `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` before backend implementation exposes administration lifecycle behavior.

## Delivery Boundary

Implement only OpenAPI-approved operations for:

- school detail, update, activation, deactivation, soft deletion, and restoration
- user detail, update, activation, deactivation, soft deletion, restoration, and selected bulk lifecycle
- role detail, update, activation, deactivation, soft deletion, restoration, and selected bulk lifecycle
- academic year detail, update, activation, deactivation, soft deletion, restoration, and selected bulk lifecycle
- academic period detail, update, activation, deactivation, soft deletion, restoration, and selected bulk lifecycle
- guardian detail, update, activation, deactivation, soft deletion, restoration, and selected bulk lifecycle

Do not implement invitations, password setup/reset, account recovery, classroom/course/section/roster workflows, teacher correction workflows, guardian self-service, student academic correction workflows, report lifecycle expansion, platform support-user school-owned access, frontend behavior, permanent purge, anonymization, billing, messaging, notifications, or undocumented APIs until the specification and OpenAPI contracts are expanded.

## Validation Walkthrough

### 1. Contract readiness

- Confirm OpenAPI contains the approved operation IDs for the slice.
- Confirm every implemented route uses the documented `/api/v1` path, request schema, parameters, response status, and content type.
- Confirm each update operation documents mutable and immutable fields.
- Confirm each lifecycle operation documents allowed transitions, required reason/effective-date fields, dependency conflicts, and response envelopes.
- Confirm selected bulk operations document resource type, action, maximum record count, all-or-nothing behavior, duplicate handling, and result envelope.
- Confirm no backend route exposes fields, filters, sorts, status codes, error responses, status values, lifecycle actions, or bulk modes absent from OpenAPI.

### 2. Tenant and authorization validation

- Confirm all school-owned resource requests resolve an active `school_id` context before detail, update, lifecycle, dependency, or bulk data access.
- Confirm missing, inactive, mismatched, and unauthorized tenant contexts fail before service logic runs.
- Confirm school lifecycle operations are platform-scoped tenant-root operations.
- Confirm platform access does not imply school-owned user, role, academic, guardian, student, teacher, report, or content lifecycle access.
- Confirm teachers, students, and guardians cannot use administration lifecycle operations unless a future contract grants that behavior.

### 3. Detail and update validation

- Confirm detail retrieval returns only records visible in the current permitted scope.
- Confirm update operations accept only documented mutable fields.
- Confirm immutable identifiers, `school_id`, role scope, parent academic ownership, undocumented fields, unsupported relationships, inactive references, and cross-tenant references are rejected.
- Confirm not-found behavior does not disclose cross-tenant record existence.

### 4. Lifecycle transition validation

- Confirm activation, deactivation, soft deletion, and restoration allow only documented transitions.
- Confirm effective date, reason, actor context, and current-state rules are enforced.
- Confirm dependency-blocked transitions return the documented conflict envelope.
- Confirm lifecycle state changes and lifecycle history writes are atomic.
- Confirm deactivated or soft-deleted records no longer participate in new operational workflows where prohibited.
- Confirm historical authorization, academic, guardian, student enrollment, report, and audit references remain retained.

### 5. School lifecycle validation

- Confirm school lifecycle actions require platform-scoped permission.
- Confirm inactive or soft-deleted schools reject protected school-scoped workflows before module-specific logic runs.
- Confirm restoring a school validates uniqueness, parent eligibility where applicable, and documented restored status behavior.
- Confirm school lifecycle actions do not expose school-owned module records to platform users.

### 6. Bulk lifecycle validation

- Confirm bulk lifecycle requests accept one resource type, one action, one scope, and only a documented maximum number of unique identifiers.
- Confirm duplicate identifiers, mixed resource types, mixed scopes, unsupported actions, missing records, unauthorized records, dependency-blocked records, and cross-tenant records are rejected.
- Confirm any invalid selected record leaves every selected record unchanged.
- Confirm successful bulk lifecycle writes history for every affected record.

### 7. Response-shape validation

- Confirm successful JSON responses use the documented success or bulk-result envelopes.
- Confirm validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, and not-found cases use only the error envelopes or codes documented on each affected OpenAPI operation.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Implementation PRs should record the contract validation result, test result, feature id `007-administration-lifecycle`, and the operation IDs implemented.

## Exit Criteria for Planning

- The implementation boundary maps to approved OpenAPI operation IDs.
- Tenant and authorization rules are explicit for every affected entity.
- Detail, update, activation, deactivation, soft delete, restore, dependency conflict, lifecycle history, and selected bulk all-or-nothing behavior are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, tenant isolation, authorization, validation, inactive statuses, immutable fields, dependency conflicts, soft delete, restore, bulk atomicity, and history preservation.
