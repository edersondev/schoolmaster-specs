# Quickstart: Backend Student Profile and Enrollment Management

## Purpose

Use this guide to validate the design intent before implementing the next SchoolMaster backend slice in `schoolmaster-backend`.

## Prerequisites

- Review [spec.md](./spec.md) for scope and acceptance outcomes.
- Review [plan.md](./plan.md) for constitution gates and backend structure.
- Review [research.md](./research.md) for design decisions.
- Review [data-model.md](./data-model.md) for affected entities.
- Review [contracts/backend-student-enrollment.md](./contracts/backend-student-enrollment.md) for the proposed operation boundary and contract expansion requirements.
- Expand `api/openapi.yaml` and `specs/001-schoolmaster-platform/contracts/openapi.yaml` before backend implementation exposes student profile lifecycle behavior.

## Delivery Boundary

Implement only OpenAPI-approved operations for:

- student profile listing
- student profile creation
- student profile detail retrieval
- student profile lifecycle status update
- student transfer recording

Do not implement frontend behavior, classroom/course/section/roster workflows, teacher assignment workflows, guardian self-service, academic-record correction workflows, report changes, bulk import, merge, anonymization, permanent deletion, restore, purge, billing, messaging, notifications, or undocumented APIs until the specification and OpenAPI contracts are expanded.

## Validation Walkthrough

### 1. Contract readiness

- Confirm OpenAPI contains the approved operation IDs for the slice.
- Confirm every implemented route uses the documented `/api/v1` path, request schema, parameters, response status, and content type.
- Confirm profile listing uses documented pagination, filters, sort options, and response envelopes only.
- Confirm create, detail, status, and transfer operations use documented success and error envelopes only.
- Confirm no backend route exposes fields, filters, sorts, status codes, error responses, status values, or transfer modes absent from OpenAPI.

### 2. Tenant and authorization validation

- Confirm all requests resolve an active `school_id` context before student profile, guardian, enrollment history, transfer, or academic data access.
- Confirm missing, inactive, mismatched, and unauthorized tenant contexts fail before service logic runs.
- Confirm platform access does not imply school-scoped student enrollment access.
- Confirm school-scoped student administration permission is required.
- Confirm teachers, students, and guardians cannot use student profile administration operations unless a future contract grants that behavior.

### 3. Student profile creation validation

- Confirm profile creation stores records only in the resolved school.
- Confirm required identity, contact, enrollment, status, and association fields follow OpenAPI.
- Confirm duplicate same-school identifiers are rejected.
- Confirm unsupported statuses, undocumented fields, malformed values, inactive references, and cross-tenant references are rejected.
- Confirm guardian references are all active and same-school before the profile is created.
- Confirm invalid guardian references reject the entire request without partial profile or association writes.

### 4. Student profile list and detail validation

- Confirm listing returns only profiles in the resolved school context.
- Confirm documented filters and sort options work exactly as declared.
- Confirm unsupported filters and sorts are rejected.
- Confirm detail retrieval returns only same-school profiles visible to the requester.
- Confirm not-found behavior does not disclose cross-tenant profile existence.

### 5. Lifecycle status validation

- Confirm status updates allow only documented lifecycle transitions.
- Confirm effective date, reason, and actor context rules are enforced.
- Confirm profile status changes and enrollment history writes are atomic.
- Confirm inactive or transferred profiles no longer participate in new active workflows where prohibited.
- Confirm historical grades, attendance, learning-set assignments, guardian associations, and report references remain retained.

### 6. Transfer validation

- Confirm transfer requires an active source profile in the resolved source school.
- Confirm transfer writes source-school enrollment history and marks the source profile transferred.
- Confirm destination-school behavior is allowed only when documented and the requester has explicit destination permission.
- Confirm transfer does not copy source-school grades, attendance, learning sets, private content, report outputs, or guardian links across tenants.
- Confirm failed transfer validation leaves source profile, destination profile, associations, and history unchanged.

### 7. Response-shape validation

- Confirm successful JSON responses use the documented success or paginated envelopes.
- Confirm validation, unauthorized, forbidden, tenant-mismatch, inactive-record, conflict, and not-found cases use only the error envelopes or codes documented on each affected OpenAPI operation.

## Verification Commands

Run contract validation from `schoolmaster-specs` before backend implementation merges:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Run backend tests from `schoolmaster-backend` after implementation:

```bash
php artisan test
```

Implementation PRs should record the contract validation result, test result, feature id `006-backend-student-enrollment`, and the operation IDs implemented.

## Exit Criteria for Planning

- The implementation boundary maps to approved OpenAPI operation IDs.
- Tenant and authorization rules are explicit for every affected entity.
- Student profile creation, guardian association validation, lifecycle status transitions, transfer behavior, and enrollment history are sufficient for backend design without inventing product behavior.
- Verification covers contract compliance, response shape, tenant isolation, authorization, validation, inactive statuses, duplicate prevention, atomic writes, transfer boundaries, and history preservation.
