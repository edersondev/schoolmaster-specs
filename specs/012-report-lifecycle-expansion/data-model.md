# Data Model: Report Lifecycle Expansion

## Modeling Principles

- `School` remains the v1 tenant root.
- School-owned reporting records use `school_id` directly.
- Public identifiers crossing API boundaries use UUIDs.
- `ReportRun` generation status, soft delete, and output availability are separate state concerns.
- Report-run delete affects default list visibility only; it does not delete, restore, extend, regenerate, or mutate generated output files.
- Generated report outputs retain the existing 90-day availability window.
- Report output availability excludes `deleted`; delete semantics belong only to report runs and report definitions in this slice.
- Custom report definitions are constrained to launch-scope domains and the read-only report catalog.
- Custom report definition names are unique per school among non-deleted definitions.
- Active custom report definitions allow metadata-only edits; structural changes require deactivation first.
- Lifecycle reason inputs use predefined tenant-safe reason codes only.
- Platform administrators and support users do not receive implicit school-scoped report access.
- Report audit events store tenant-safe metadata only and never store generated report contents, private paths, credentials, full filter payloads, or unauthorized cross-tenant details.

## Entities

### ReportRun

- **Purpose**: Existing school-owned asynchronous report request and durable lifecycle history record.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `requested_by_user_id`
  - `report_type` or custom definition reference
  - `filter_summary`
  - `output_formats`
  - generation status (`requested`, `generating`, `generated`, `failed`, `canceled`)
  - soft-delete marker and timestamp
  - `source_report_run_id` for retry lineage
  - `superseded_by_report_run_id` where retry creates a replacement
  - generated timestamp
  - failure reason code where applicable
  - cancellation reason code where applicable
  - correlation ID
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to requester `User`
  - may reference source `ReportRun`
  - may be superseded by another `ReportRun`
  - may reference `ReportDefinitionSnapshot`
  - has many `ReportOutput`
  - has many `ReportLifecycleEvent`
- **Validation rules**:
  - every lookup requires active permitted school context
  - retry creates a new report run and preserves the original run
  - retry is allowed only for failed runs and expired generated runs whose original filters and definition snapshot remain valid
  - cancellation is allowed only before terminal generation status and before output publication
  - restore clears the soft-delete marker while preserving generation status, output availability, definition snapshots, and audit history
  - default lists include all non-deleted runs; deleted runs require an explicit include-deleted filter
  - concurrent lifecycle actions use first-valid-transition-wins semantics

### ReportOutput

- **Purpose**: Private tenant-scoped generated file for one report run and one output format.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `report_run_id`
  - format (`pdf`, `csv`, `xlsx` where supported)
  - availability (`pending`, `available`, `failed`, `expired`, `unsupported`)
  - generated timestamp
  - expires timestamp
  - failure reason code where applicable
  - private storage reference
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to `ReportRun`
- **Validation rules**:
  - downloadable only when same-school, generated, requested format exists, and unexpired
  - per-format availability is independent
  - availability does not include `deleted`
  - output files expire 90 days after generation
  - expired downloads return the documented expired-output response
  - output delete and restore are not exposed in this slice
  - private paths, storage keys, worker internals, and cross-tenant file existence are never exposed

### ReportDefinition

- **Purpose**: School-owned custom report definition that can be activated and used to request custom report runs.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - name
  - description
  - launch-scope domain (`attendance`, `grades`, `academic_structure`, `school_activity`)
  - selected fields, maximum 25
  - filters, maximum 10
  - grouping levels, maximum 2
  - sort fields, maximum 3
  - output formats
  - lifecycle state (`draft`, `active`, `inactive`, `deleted`)
  - version
  - created_by_user_id
  - updated_by_user_id
  - timestamps
- **Relationships**:
  - belongs to `School`
  - belongs to creator `User`
  - may have many `ReportDefinitionSnapshot`
  - may have many `ReportLifecycleEvent`
- **Validation rules**:
  - definitions belong to exactly one school
  - definitions cannot be shared across schools
  - definition names must be unique per school among non-deleted definitions
  - new definitions start as `draft`
  - only `active` definitions may be used to request report runs
  - active definitions may update only name and description
  - domain, selected fields, filters, grouping, sorting, and output formats are structural fields and require deactivation before update
  - restore returns deleted definitions to `inactive`
  - restore fails with the documented conflict response when another non-deleted definition in the same school already uses the restored definition name
  - fields, filters, grouping, sorting, and output formats must exist in the report catalog
  - arbitrary query text, executable expressions, unsupported joins, hidden fields, private paths, report output references, credentials, school-only notes, correction history, guardian-restricted fields, questionnaire answer keys, non-launch-scope domains, cross-tenant references, and over-limit definitions are rejected

### ReportDefinitionSnapshot

- **Purpose**: Immutable copy of report definition and runtime request shape captured at report-run creation time.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - `report_definition_id`
  - definition version
  - domain
  - selected fields
  - filters
  - grouping
  - sorting
  - output formats
  - runtime filter values
  - created_at
- **Relationships**:
  - belongs to `School`
  - belongs to `ReportDefinition`
  - has many `ReportRun`
- **Validation rules**:
  - immutable after report-run creation
  - existing report runs keep historical meaning after definition update, delete, or restore
  - snapshot values must be tenant-safe and must not include raw private payloads beyond approved report configuration

### ReportCatalog

- **Purpose**: Read-only school-scoped catalog of approved custom report configuration inputs.
- **Core fields**:
  - domain
  - field identifiers and labels
  - filter operators
  - allowed reference types
  - grouping options
  - sorting options
  - supported output formats
  - format compatibility rules
  - complexity limits
- **Relationships**:
  - resolved in the context of one `School`
  - referenced by `ReportDefinition` validation
- **Validation rules**:
  - exposes only attendance, grades, academic structure, and school activity domains
  - hidden/private fields are absent from the catalog
  - custom definitions using unsupported or absent catalog entries are rejected
  - catalog reads require authenticated school-scoped custom report permission

### ReportLifecycleEvent

- **Purpose**: Tenant-safe audit and history record for report lifecycle, catalog, definition, output, validation, conflict, and denied-access outcomes.
- **Core fields**:
  - `id` (UUID)
  - `school_id`
  - actor user ID where available
  - action
  - outcome
  - target type
  - target ID where safe
  - correlation ID
  - tenant-safe reason code
  - timestamp
  - tenant-safe summary metadata
- **Relationships**:
  - belongs to `School`
  - may reference actor `User`
  - may reference `ReportRun`
  - may reference `ReportDefinition`
- **Validation rules**:
  - required for lifecycle transitions, retry, cancellation, delete, restore, catalog access, definition changes, output expiry, denied access, validation failures, conflicts, blocked cross-tenant attempts, and stale worker completions
  - must not store private report contents, credentials, file paths, full filter payloads, generated output contents, or unauthorized cross-tenant details
  - must not store free-text lifecycle reasons

### AcademicPeriod

- **Purpose**: Existing school-owned period used by built-in and custom report filters.
- **Core fields used in this slice**:
  - `id` (UUID)
  - `school_id`
  - status
  - date range
- **Relationships**:
  - belongs to `School`
  - referenced by report filters and definition snapshots
- **Validation rules**:
  - every reference must resolve to the active permitted school context
  - inactive, missing, unauthorized, or cross-tenant periods are rejected according to OpenAPI

### User

- **Purpose**: Existing actor identity for report lifecycle and definition operations.
- **Validation rules**:
  - inactive users cannot perform report operations
  - lifecycle operations require school-scoped report lifecycle permission
  - definition and catalog operations require school-scoped report definition permission
  - platform roles do not imply school report access

### School

- **Purpose**: Tenant root for report runs, definitions, outputs, catalog resolution, snapshots, and audit events.
- **Validation rules**:
  - every operation requires active permitted school context
  - missing, mismatched, inactive, or unauthorized school context fails before school-owned report data access
  - cross-tenant references are rejected without exposing protected record existence

## State Transitions

### ReportRun Generation Status

| From | To | Trigger | Notes |
|------|----|---------|-------|
| none | `requested` | built-in or custom report request | Creates report run and requested output formats |
| `requested` | `generating` | worker start | Same-school worker context required |
| `requested`/`generating` | `generated` | worker completion | Publishes only requested supported outputs |
| `requested`/`generating` | `failed` | worker failure | Stores tenant-safe failure reason |
| `requested`/`generating` | `canceled` | authorized cancellation | Prevents later output publication |
| `failed`/expired generated | `requested` on new run | retry | New run links to source; source is preserved |

### ReportRun Soft Delete

| Action | Effect |
|--------|--------|
| delete | Sets soft-delete marker and removes run from default lists |
| restore | Clears soft-delete marker and preserves generation status and output availability |

### ReportDefinition Lifecycle

| From | To | Trigger | Notes |
|------|----|---------|-------|
| none | `draft` | create | New definitions are not requestable |
| `draft`/`inactive` | `active` | activate | Validates catalog, limits, and school context |
| `active`/`draft` | `inactive` | deactivate | Future requests blocked |
| `draft`/`active`/`inactive` | `deleted` | delete | Hidden from default definition lists |
| `deleted` | `inactive` | restore | Activation is separate |

Active definition edit boundary:

| Current State | Allowed update |
|---------------|----------------|
| `active` | Name and description only |
| `draft`/`inactive` | Metadata and structural report configuration |
| `deleted` | No update until restore |

### ReportOutput Availability

| State | Meaning |
|-------|---------|
| `pending` | Format was requested but output is not yet available |
| `available` | Format is generated, unexpired, and downloadable by authorized users |
| `failed` | Format generation failed while other formats may still succeed |
| `expired` | 90-day retention window has passed |
| `unsupported` | Format is not supported for the report type or definition |

`deleted` is intentionally absent from output availability because report outputs have no explicit delete or restore API in this slice.

## Validation Summary

- Resolve active permitted school context before report lookup, catalog lookup, definition lookup, filter validation, output lookup, persistence, audit write, or response shaping.
- Reject unsupported fields, filters, grouping, sorting, output formats, and over-limit custom definitions before persistence.
- Reject duplicate custom report definition names within the same school when a non-deleted definition already uses that name.
- Reject structural updates to active custom report definitions until the definition is inactive.
- Reject lifecycle reason inputs that are not predefined tenant-safe reason codes.
- Validate every academic period, user, student profile, status, and date range reference within the resolved school and selected report type or definition.
- Apply first-valid-transition-wins for lifecycle races and stale worker completions.
- Return or record documented conflict outcomes without partial state changes.
- Shape every response through OpenAPI-approved envelopes and API Resources.
