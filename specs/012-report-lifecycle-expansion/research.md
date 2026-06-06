# Research: Report Lifecycle Expansion

## Decision: Expand OpenAPI before backend exposure

**Decision**: All report lifecycle, catalog, custom definition, custom request, XLSX, expanded list, conflict, and audit response behavior must be added to `api/openapi.yaml` and any affected platform contract before backend routes expose the behavior.

**Rationale**: The constitution requires API-first contract governance, and the current report contract only documents `listReports`, `requestReport`, and `downloadReport` with PDF/CSV output.

**Alternatives considered**:

- Backend-local routes first: rejected because it would create undocumented public API behavior.
- Only feature-local contract notes: rejected because backend and frontend consume the aggregate OpenAPI contract.

## Decision: Separate generation status, soft delete, and output availability

**Decision**: `ReportRun` generation status uses `requested`, `generating`, `generated`, `failed`, and `canceled`; soft delete is a separate marker; generated output availability is tracked per format.

**Rationale**: A single status field cannot accurately represent a generated report that is soft-deleted from lists while one output format is expired and another remains available. Separating these concerns makes state transitions testable and prevents retention side effects.

**Alternatives considered**:

- Single status enum with `deleted` and `expired`: rejected because output expiry is per file format, not a report-run lifecycle state.
- Treat soft delete as generation status: rejected because restore should not mutate generation history.

## Decision: Lifecycle concurrency is first-valid-transition-wins

**Decision**: Concurrent retry, cancel, delete, restore, worker completion, and output-expiry attempts use first-valid-transition-wins semantics. Later conflicting attempts return or record documented conflict outcomes without partial state changes.

**Rationale**: Report generation is asynchronous, so cancellation and worker completion can race. Deterministic conflict semantics keep state auditable and avoid output publication after cancellation.

**Alternatives considered**:

- Idempotent success for repeated actions: rejected because retry/cancel/delete races can materially change output publication.
- Latest action wins: rejected because it can override a valid terminal transition and weaken auditability.
- Queue all lifecycle actions: rejected for v1 because it adds operational complexity without improving product clarity.

## Decision: Add a read-only report catalog

**Decision**: Add a read-only school-scoped report catalog operation that exposes approved launch-scope domains, fields, filter operators, grouping options, sorting options, output formats, and compatibility rules.

**Rationale**: Custom report definitions must be validated against a contract-governed catalog. Exposing the catalog prevents future frontend report-designer clients from hardcoding unsupported combinations.

**Alternatives considered**:

- Static OpenAPI schemas only: rejected because allowed combinations can vary by report domain and format.
- Embed catalog data only in definition responses: rejected because clients need discovery before creating definitions.
- Defer catalog to frontend work: rejected because backend validation needs the same source of truth now.

## Decision: Limit custom report domains to launch scope

**Decision**: Custom report definitions may use only attendance, grades, academic structure, and school activity domains.

**Rationale**: These are the existing launch-scope report domains. Expanding into guardian, teacher content, private files, support, or platform domains would create new visibility and privacy questions outside this slice.

**Alternatives considered**:

- Any school-owned domain except private files: rejected because it would blur authorization boundaries and expose hidden fields.
- Guardian summary data: rejected because guardian self-service explicitly excludes report access.
- Defer all custom definitions: rejected because the roadmap explicitly includes custom report definitions and designer backend behavior.

## Decision: Custom definition lifecycle states

**Decision**: Custom report definitions use `draft`, `active`, `inactive`, and `deleted`. New definitions start as `draft`; only `active` definitions may be requested; restore returns to `inactive`.

**Rationale**: Draft gives administrators a safe preparation state, active controls requestability, inactive supports temporary suspension, and restore-to-inactive prevents accidentally making restored definitions requestable without current validation.

**Alternatives considered**:

- `active`, `inactive`, and `deleted` only: rejected because it lacks a pre-activation authoring state.
- `published`/`archived` terminology: rejected because existing backend lifecycle language uses active/inactive/deleted patterns.
- Immediately active definitions: rejected because it increases risk from invalid or unfinished report definitions.

## Decision: Active definitions allow metadata-only edits

**Decision**: Active custom report definitions may update name and description only. Changes to domain, selected fields, filters, grouping, sorting, or output formats require deactivation first.

**Rationale**: Active definitions are requestable. Blocking structural edits while active avoids request-time races and keeps the report shape stable while still allowing harmless administrative metadata changes.

**Alternatives considered**:

- Allow all edits while active: rejected because future report requests could race against structural changes and produce hard-to-explain snapshots.
- Make active definitions fully read-only: rejected because name and description updates do not affect report shape and can be safely validated.

## Decision: Custom definition complexity limits

**Decision**: V1 custom report definitions allow up to 25 selected fields, 10 filters, 2 grouping levels, and 3 sort fields.

**Rationale**: These bounds are large enough for practical school reports while limiting query cost, validation complexity, and output shape.

**Alternatives considered**:

- Smaller limits: rejected because common administrative reports may need more than a handful of fields or filters.
- Larger limits: rejected for v1 because they increase performance and output-shape risk.
- No fixed limits: rejected because implementation and tests need clear rejection criteria.

## Decision: XLSX is the only new output format

**Decision**: Add XLSX as the only output format beyond existing PDF and CSV, and only where OpenAPI and the report catalog declare support.

**Rationale**: XLSX supports operational tabular reporting without introducing document-template, archive, or designer-rendering scope.

**Alternatives considered**:

- Add document formats such as DOCX: rejected because they imply template and document designer behavior.
- Add archive formats: rejected because they complicate file packaging and retention.
- Leave only PDF/CSV: rejected because the roadmap explicitly calls for additional output formats.

## Decision: No explicit report-output delete or restore

**Decision**: This slice does not expose report-output delete or restore actions. Output lifecycle is limited to format support, generated availability, per-format generation failure, and 90-day expiry.

**Rationale**: Report-run soft delete handles operational list visibility, while generated output retention remains predictable and unchanged.

**Alternatives considered**:

- Per-output delete and restore: rejected because it creates retention and recovery rules beyond roadmap needs.
- Output delete only: rejected because it would create irreversible file lifecycle behavior without permanent purge policy.
- Defer all output lifecycle expansion: rejected because per-format availability and XLSX are needed for this slice.

## Decision: Exclude deleted from output availability

**Decision**: Report output availability states are `pending`, `available`, `failed`, `expired`, and `unsupported`; `deleted` is not an output availability state.

**Rationale**: Output delete/restore actions are out of scope. Keeping `deleted` out of output availability prevents report-run soft delete from being mistaken for file lifecycle mutation.

**Alternatives considered**:

- Keep `deleted` as an internal output state: rejected because it invites undocumented output lifecycle behavior.
- Add explicit output deletion semantics: rejected because the feature deliberately excludes output delete/restore and permanent purge.

## Decision: Lifecycle reasons use predefined tenant-safe reason codes

**Decision**: Lifecycle reason inputs use predefined tenant-safe reason codes only. Free-text lifecycle reasons are not accepted or stored.

**Rationale**: Reason-code enums are stable in OpenAPI, simpler to validate, safer for audit records, and avoid accidental storage of sensitive report or tenant data.

**Alternatives considered**:

- Free-text reasons with sanitization: rejected because sanitization is error-prone and creates privacy/audit risk.
- Reason code plus optional note: rejected for v1 because optional notes still create leakage and moderation concerns.

## Decision: Custom report definition names are unique per school among non-deleted definitions

**Decision**: A school cannot have two non-deleted custom report definitions with the same name. Restoring a deleted definition conflicts if another non-deleted definition in the same school already uses that name.

**Rationale**: School administrators need an unambiguous operational list while soft deletion should permit name reuse after cleanup. Restore conflicts are explicit and testable.

**Alternatives considered**:

- Unique forever including deleted definitions: rejected because it blocks practical reuse after soft deletion.
- UUID-only identity with duplicate names allowed: rejected because it degrades administrator experience and complicates list/search behavior.

## Decision: Audit minimum fields

**Decision**: Report audit events include actor, action, outcome, target, school, correlation ID, and tenant-safe reason code, and must not store private payloads, credentials, file paths, generated report contents, full filter payloads, or unauthorized cross-tenant details.

**Rationale**: These fields match the project audit posture while giving enough context to investigate lifecycle, definition, catalog, conflict, validation, and denied-access outcomes.

**Alternatives considered**:

- Minimal actor/action/outcome only: rejected because conflict and stale worker investigation need correlation and target context.
- Full before/after snapshots or request summaries: rejected because report filters and generated contents can contain sensitive school data.

## Decision: Backend implementation structure

**Decision**: Implement through Laravel controllers, Form Requests, Policies, API Resources, Services, DTOs, and repositories/query objects only for complex report catalog, definition validation, lifecycle concurrency, output availability, and audit-safe reads.

**Rationale**: This follows the constitution and prior backend slices while keeping controllers thin and preserving tenant-safe service boundaries.

**Alternatives considered**:

- Controller-level lifecycle logic: rejected by constitution.
- Repository abstraction for every model: rejected because repositories are reserved for complex access patterns.
