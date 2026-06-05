# Contract Boundary: Report Lifecycle Expansion

## Purpose

This feature expands the public API surface for school-scoped report lifecycle, report catalog, custom report definitions, custom report requests, and per-format output behavior. Backend implementation must wait until OpenAPI documents exact operations, parameters, request schemas, response schemas, errors, tenant behavior, authorization behavior, output formats, conflict semantics, audit expectations, and operation IDs.

## Authoritative Contracts

- Aggregate contract: `api/openapi.yaml`
- Platform feature contract: `specs/001-schoolmaster-platform/contracts/openapi.yaml`
- Source-of-truth guide: `AGENTS.md`
- Backend implementation guidance: `docs/backend-guidelines.md`
- Multi-tenant guidance: `docs/multi-tenant.md`
- Security guidance: `docs/security.md`
- Tenant decision: `decisions/004-use-tenant-by-column.md`

## Current Report Baseline

Existing OpenAPI currently documents:

| Operation ID | Method and Path | Boundary |
|--------------|-----------------|----------|
| `listReports` | `GET /api/v1/reports` | List report runs for the resolved school |
| `requestReport` | `POST /api/v1/reports` | Request an asynchronous launch-scope report |
| `downloadReport` | `GET /api/v1/reports/{reportRunId}/download` | Download an authorized generated PDF or CSV output while unexpired |

This feature is additive and must preserve existing report request, list, download, PDF/CSV, 90-day retention, and expired-output behavior unless OpenAPI documents a compatible expansion.

## Approved Operation Boundary

OpenAPI must define operation IDs and routes for this backend slice before implementation. Proposed operation mapping:

| Method | Route | Proposed Operation ID | Boundary |
|--------|-------|-----------------------|----------|
| `GET` | `/api/v1/reports` | `listReports` | Expand filters to include generation status, custom/built-in type, and explicit include-deleted behavior while preserving default non-deleted listing |
| `POST` | `/api/v1/reports` | `requestReport` | Expand request behavior to support catalog-approved custom definitions, snapshots, and approved output formats |
| `POST` | `/api/v1/reports/{reportRunId}/retry` | `retryReport` | Create a new report run from failed or expired generated source runs |
| `POST` | `/api/v1/reports/{reportRunId}/cancel` | `cancelReport` | Cancel requested or generating runs before output publication |
| `DELETE` | `/api/v1/reports/{reportRunId}` | `deleteReport` | Soft-delete report runs from default lists without deleting outputs |
| `POST` | `/api/v1/reports/{reportRunId}/restore` | `restoreReport` | Clear soft-delete marker while preserving generation and output state |
| `GET` | `/api/v1/reports/{reportRunId}/download` | `downloadReport` | Expand format support to catalog-approved XLSX while preserving existing PDF/CSV behavior |
| `GET` | `/api/v1/report-catalog` | `getReportCatalog` | Return approved launch-scope domains, fields, filters, operators, grouping, sorting, output formats, and compatibility rules |
| `GET` | `/api/v1/report-definitions` | `listReportDefinitions` | List same-school report definitions with documented lifecycle filters |
| `POST` | `/api/v1/report-definitions` | `createReportDefinition` | Create draft same-school custom report definition |
| `GET` | `/api/v1/report-definitions/{reportDefinitionId}` | `getReportDefinition` | Retrieve same-school custom report definition |
| `PATCH` | `/api/v1/report-definitions/{reportDefinitionId}` | `updateReportDefinition` | Update allowed fields without changing historical snapshots; active definitions allow metadata-only edits |
| `POST` | `/api/v1/report-definitions/{reportDefinitionId}/activate` | `activateReportDefinition` | Activate a valid draft or inactive definition |
| `POST` | `/api/v1/report-definitions/{reportDefinitionId}/deactivate` | `deactivateReportDefinition` | Move active definition to inactive |
| `DELETE` | `/api/v1/report-definitions/{reportDefinitionId}` | `deleteReportDefinition` | Soft-delete definition |
| `POST` | `/api/v1/report-definitions/{reportDefinitionId}/restore` | `restoreReportDefinition` | Restore definition to inactive |

## Required Contract Expansion

OpenAPI must define, at minimum:

- operation IDs and versioned `/api/v1` paths for every approved operation
- required `X-School-Id` tenant context behavior where applicable
- school-scoped report lifecycle permission
- school-scoped report definition/catalog permission
- report generation statuses `requested`, `generating`, `generated`, `failed`, and `canceled`
- separate soft-delete metadata and include-deleted list behavior
- per-output availability states and response fields
- output availability states excluding `deleted`
- retry lineage fields where exposed
- predefined tenant-safe lifecycle reason-code request schemas
- report definition lifecycle states `draft`, `active`, `inactive`, and `deleted`
- per-school custom report definition name uniqueness among non-deleted definitions
- update schemas that distinguish active metadata-only edits from inactive/draft structural edits
- custom definition create/update schemas and complexity limits
- report catalog response schemas
- XLSX as an approved format only where supported
- conflict responses for invalid concurrent lifecycle transitions and stale worker completions
- validation errors for unsupported catalog entries, hidden fields, unsupported formats, over-limit definitions, invalid references, and cross-tenant filters
- audit expectations with actor, action, outcome, target, school, correlation ID, and tenant-safe reason code

## Required Response Shapes

Backend implementation must follow only the response statuses, content types, and components declared on each approved OpenAPI operation. Expected response categories include:

- success envelopes for report lifecycle actions, report definitions, and report catalog reads
- accepted envelopes for asynchronous report requests and retries where applicable
- paginated envelopes for report and definition lists
- validation error envelope
- unauthorized response for unauthenticated or inactive actor access
- forbidden response for authenticated actors without school-scoped permission
- tenant mismatch or inactive tenant response for school-scoped operations
- not-found response for missing, unauthorized, deleted where not included, or cross-tenant targets according to contract
- conflict response for first-valid-transition-wins lifecycle conflicts and stale worker completions where client-visible
- expired-output response for expired downloads
- binary response for authorized generated output downloads

No backend-local product envelope, ad hoc error response, undocumented status code, undocumented field, undocumented filter, undocumented include expansion, undocumented sort behavior, undocumented lifecycle action, undocumented output format, undocumented output lifecycle action, undocumented audit payload, or authorization exception is approved in this slice.

## Tenant Behavior

- Operations use documented `X-School-Id` tenant context behavior when the authenticated actor is not already bound to exactly one active school.
- V1 school-owned report records use `school_id`.
- Missing, mismatched, inactive, or unauthorized tenant context fails before report lookup, definition lookup, catalog lookup, filter validation, output lookup, persistence, or response shaping.
- Platform users do not receive implicit permission to perform school-scoped report lifecycle, catalog, definition, output, or custom request workflows.

## Authorization Behavior

- All operations require authenticated access and active actor status.
- Report lifecycle operations require explicit same-school report lifecycle permission.
- Report definition and catalog operations require explicit same-school report definition permission.
- Report download requires same-school generated output access in the requested supported format.
- Support-user or platform-wide reporting access is outside this slice.

## Validation Behavior

- Report retry is allowed only for failed runs and expired generated runs whose original filters and definition snapshot remain valid.
- Cancellation is allowed only for requested or generating runs before output publication.
- Report-run delete is soft delete only and must not delete generated outputs.
- Report-run restore clears the soft-delete marker and preserves generation/output state.
- Default report lists include all non-deleted report runs.
- Deleted report runs require an explicit include-deleted filter.
- Custom definitions must use only catalog-approved launch-scope fields, filters, grouping, sorting, and formats.
- Custom definitions must not exceed 25 fields, 10 filters, 2 grouping levels, or 3 sort fields.
- Custom definition names must be unique per school among non-deleted definitions.
- Restoring a deleted custom definition must return the documented conflict response if another non-deleted same-school definition already uses the restored name.
- Active custom definitions may update only name and description.
- Changes to domain, selected fields, filters, grouping, sorting, or output formats require the definition to be inactive or draft.
- Custom definitions must reject hidden/private fields, arbitrary query text, executable expressions, unsupported joins, cross-tenant references, and non-launch-scope domains.
- Lifecycle reason inputs must be predefined tenant-safe reason codes; free-text lifecycle reasons are rejected.
- Stale worker completions after cancellation, delete, restore, supersession, or terminal state must be ignored and audited.
- Expired downloads must not regenerate, enqueue retry, mutate runs, or change retention timestamps.

## Blocked Until Future Specification

These behaviors are outside this implementation boundary until a future spec and OpenAPI update approve them:

- frontend report designer or reporting UI implementation
- platform-wide reporting or support-user cross-school visibility
- report-output delete or restore
- retention override, legal hold, permanent purge, or anonymization
- report domains beyond attendance, grades, academic structure, and school activity
- arbitrary SQL, scripting, uploaded report templates, or custom code execution
- output formats beyond PDF, CSV, and catalog-approved XLSX
- automatic expired-output regeneration during download
- guardian report access
- student self-view, teacher workflow, roster, billing, messaging, notification, payroll, or accounting behavior

## Verification Boundary

Backend implementation PRs for this slice must link to every operation ID implemented and include evidence for:

- OpenAPI validation of aggregate and platform contracts
- PHPUnit feature coverage for every approved operation
- response-shape checks for success and standard error envelopes
- tenant-isolation failures for missing, mismatched, inactive, and unauthorized school context
- report lifecycle permission checks
- report definition/catalog permission checks
- retry eligibility and lineage checks
- cancellation eligibility and stale worker completion checks
- delete/restore soft-delete behavior and default list behavior
- first-valid-transition-wins conflict behavior
- report catalog discovery and hidden-field exclusion
- custom definition lifecycle checks
- custom definition duplicate-name and restore-name-conflict checks
- active-definition metadata-only edit checks and structural-edit rejection checks
- custom definition complexity-limit rejection
- historical definition snapshot preservation
- XLSX support and unsupported format rejection
- per-format output availability and expired-output behavior
- absence of `deleted` from output availability
- report-run delete/restore not mutating output files or retention timestamps
- tenant-safe audit events with actor, action, outcome, target, school, correlation ID, and reason code
- invalid lifecycle reason-code rejection and no free-text lifecycle reason persistence
