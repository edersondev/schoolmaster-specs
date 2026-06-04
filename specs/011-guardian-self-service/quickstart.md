# Quickstart: Backend Guardian Self-Service

## Purpose

Use this walkthrough to verify that the guardian self-service plan, contract boundary, and backend implementation remain aligned before implementation proceeds.

## Prerequisites

- Active feature branch: `011-guardian-self-service`
- Active spec: `specs/011-guardian-self-service/spec.md`
- Active plan: `specs/011-guardian-self-service/plan.md`
- OpenAPI changes must be completed before backend routes expose guardian self-service behavior.
- Backend implementation must remain API-only and scoped to `schoolmaster-backend`.

## Contract Validation

After OpenAPI is expanded for this feature, validate the aggregate contract:

```bash
npx @redocly/cli lint api/openapi.yaml
```

Validate the platform feature contract if it is updated:

```bash
npx @redocly/cli lint specs/001-schoolmaster-platform/contracts/openapi.yaml
```

Contract review must confirm:

- operation IDs exist for every approved guardian self-service route
- all routes remain under `/api/v1`
- request parameters include explicit academic period for academic summaries
- schemas expose only approved guardian-visible fields
- target-specific missing, unassociated, inactive, and cross-tenant students share the same not-found envelope
- unsupported filters, includes, sorts, and invalid payloads have documented validation behavior
- unauthenticated, forbidden, tenant-mismatch, inactive-record, empty-result, and not-found envelopes are documented

## Backend Verification

After backend implementation, run the backend test suite from the backend repository:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Focused backend coverage must include:

- successful guardian student listing
- successful guardian student detail summary
- successful guardian academic summary with explicit same-school academic period
- successful guardian contact view
- missing, inactive, mismatched, and unauthorized school context rejection
- authenticated user with no active guardian record rejection
- missing or inactive explicit guardian-user link rejection
- inactive guardian-student association rejection
- target-specific missing, unassociated, inactive, and cross-tenant students returning the same not-found envelope
- guardian student listing empty-result behavior
- missing, inactive, unsupported, cross-tenant, or unauthorized academic period rejection
- summary-only academic visibility
- detailed grade and attendance row redaction
- private correction reason and full correction history redaction
- teacher content, questionnaire answer key, report, and private file redaction
- limited contact visibility
- other guardian, non-primary student contact, restricted emergency, and school-only note redaction
- source-school access ending after student transfer unless OpenAPI documents limited historical labels
- destination-school access requiring separate active destination-school association
- guardian read-only boundary for creates, updates, deletes, corrections, imports, report requests, downloads, and student activity submission
- tenant-safe audit events for allowed reads, denied attempts, blocked cross-tenant attempts, and visibility-sensitive reads

## Constitution Gate Checklist

- OpenAPI was updated before backend exposure.
- Backend routes map 100% to approved OpenAPI operation IDs.
- Controllers are orchestration-only.
- Business rules live in Services.
- Request validation uses Form Requests.
- Authorization uses Policies.
- Responses use API Resources and documented envelopes.
- Multi-field query inputs use DTOs where they improve clarity.
- Repositories/query objects are used only for complex tenant-scoped guardian visibility, summary aggregation, contact visibility, non-enumerating denial, or audit-safe reads.
- Public identifiers are UUIDs.
- All school-owned reads are scoped by `school_id`.
- Platform users do not receive implicit guardian self-service or support access.
- No frontend behavior is implemented in this slice.

## Out-of-Scope Guardrail

Reject implementation or contract additions for:

- guardian self-claiming
- automatic contact matching
- invitation completion creating guardian-user links
- guardian profile updates
- guardian association requests
- consent-signature capture
- custody or legal-document workflows
- messaging or notifications
- report lifecycle or report downloads
- teacher content downloads
- detailed grade or attendance rows
- student activity submission
- platform support access
- billing, payment, payroll, or accounting
- permanent purge, anonymization, legal hold, or retention overrides

Any needed behavior from this list requires a separate specification and OpenAPI update.
