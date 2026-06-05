# Quickstart: Report Lifecycle Expansion

## Purpose

Use this walkthrough to verify that the report lifecycle expansion plan, contract boundary, and backend implementation remain aligned before implementation proceeds.

## Prerequisites

- Active feature branch: `012-report-lifecycle-expansion`
- Active spec: `specs/012-report-lifecycle-expansion/spec.md`
- Active plan: `specs/012-report-lifecycle-expansion/plan.md`
- OpenAPI changes must be completed before backend routes expose report lifecycle expansion behavior.
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

- operation IDs exist for every approved report lifecycle, catalog, definition, custom request, restore, retry, cancel, delete, and download operation
- all routes remain under `/api/v1`
- existing `listReports`, `requestReport`, and `downloadReport` behavior remains compatible
- report generation status, soft delete, and per-output availability are represented separately
- output availability uses only pending, available, failed, expired, and unsupported states
- default report lists include non-deleted runs and exclude deleted runs unless the include-deleted filter is used
- report catalog exposes only launch-scope report domains and approved field/filter/group/sort/format entries
- custom report definitions reject unsupported catalog entries and over-limit definitions
- XLSX is accepted only where the contract and catalog declare support
- expired-output, conflict, validation, not-found, unauthorized, forbidden, and tenant-mismatch envelopes are documented
- audit expectations include actor, action, outcome, target, school, correlation ID, and tenant-safe reason code
- lifecycle reason inputs are predefined tenant-safe reason codes with no free-text note field
- custom report definition names are unique per school among non-deleted definitions
- active custom report definitions allow name and description updates only; structural edits require deactivation first

## Backend Verification

After backend implementation, run the backend test suite from the backend repository:

```bash
docker exec schoolmaster-backend-app-1 php artisan test
```

Focused backend coverage must include:

- successful report retry from failed report runs
- successful report retry from expired generated report runs with valid original filters and snapshots
- retry rejection for requested, generating, generated-unexpired, canceled, deleted, cross-tenant, unauthorized, and superseded runs
- successful cancellation before output publication
- stale worker completion ignored and audited after cancellation
- report-run delete and restore preserving generation status and output availability
- default report lists including all non-deleted runs and excluding deleted runs
- include-deleted list behavior
- first-valid-transition-wins conflict behavior for concurrent lifecycle actions
- report catalog retrieval with only approved launch-scope entries
- custom definition create, update, activate, deactivate, delete, and restore
- custom definition restore returning to inactive
- duplicate non-deleted custom definition name rejection within the same school
- deleted definition restore returning conflict when another non-deleted same-school definition already uses the restored name
- active custom definition metadata-only update success
- active custom definition structural update rejection for domain, fields, filters, grouping, sorting, and formats
- active-only custom report requests
- historical definition snapshots preserved after definition update/delete/restore
- complexity-limit rejection for more than 25 fields, 10 filters, 2 grouping levels, or 3 sort fields
- unsupported field, filter, grouping, sorting, format, and domain rejection
- XLSX output generation and download only where catalog-approved
- unsupported output format rejection
- per-format output availability and failure behavior
- output availability never exposing a deleted state
- expired output download returning the documented expired-output response without regeneration
- report-run soft delete not deleting, restoring, extending, regenerating, or mutating output files
- tenant-safe audit events for lifecycle, definition, catalog, output, validation, conflict, denied, and cross-tenant outcomes
- predefined reason-code validation for lifecycle inputs and no free-text lifecycle reason persistence
- platform and support users receiving no implicit school-scoped report access

## Backend Architecture Checklist

- Controllers are orchestration-only.
- Business rules live in `App\Services\Reports`.
- Request validation uses Form Requests.
- Authorization uses Policies.
- Responses use API Resources and documented envelopes.
- Multi-field inputs use DTOs where they improve clarity.
- Repositories/query objects are used only for complex catalog, definition validation, lifecycle concurrency, report listing, output availability, and audit-safe reads.
- Public identifiers are UUIDs.
- All school-owned reads and writes are scoped by `school_id`.
- Platform users do not receive implicit report lifecycle, definition, catalog, or output access.
- No frontend behavior is implemented in this slice.

## Out-of-Scope Guardrail

Reject implementation or contract additions for:

- frontend report designer or report UI implementation
- platform-wide reporting or support-user cross-school access
- output delete or output restore actions
- retention override, legal hold, permanent purge, or anonymization
- report domains beyond attendance, grades, academic structure, and school activity
- arbitrary SQL, scripting, uploaded templates, or custom code execution
- output formats beyond PDF, CSV, and catalog-approved XLSX
- automatic expired-output regeneration during download
- guardian report access
- student self-view changes
- teacher workflow changes
- roster changes
- billing, messaging, notification, payroll, or accounting behavior

Any needed behavior from this list requires a separate specification and OpenAPI update.
