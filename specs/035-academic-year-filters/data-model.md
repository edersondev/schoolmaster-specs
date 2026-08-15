# Data Model: Academic Year List Filters

## Existing Entity: Academic Year

No persisted field changes.

| Field | Meaning | Filter behavior |
|---|---|---|
| `school_id` | Owning school | Mandatory resolved-school boundary |
| `name` | Administrator-facing label | Case-insensitive contains match |
| `start_date` | First included calendar date | Must be on or before selected range end |
| `end_date` | Last included calendar date | Must be on or after selected range start |
| `status` | Lifecycle state | Exact match: planned, active, closed, inactive |
| `deleted_at` | Soft-delete marker | Existing visibility remains unchanged |

## Transient Entity: Academic Year Filter Set

| Field | Type | Required | Validation and normalization |
|---|---|---:|---|
| `name` | string | No | Trimmed; blank omitted; maximum 255 characters |
| `date_from` | date | With `date_to` | `YYYY-MM-DD`; inclusive lower search boundary |
| `date_to` | date | With `date_from` | `YYYY-MM-DD`; same as or later than `date_from` |
| `status` | enum | No | planned, active, closed, or inactive |
| `page` | positive integer | No | Existing default and validation |
| `per_page` | positive integer | No | Existing maximum and default |

## Query Invariants

1. Resolve school and authorize academic-year view access before returning data.
2. Begin from academic years owned by the resolved school.
3. Apply only populated, validated criteria.
4. Combine criteria with AND semantics.
5. Apply overlap as `start_date <= date_to` and `end_date >= date_from`.
6. Preserve current descending start-date order and pagination.
7. Return no matches as a successful empty page.

## State Transitions

None. Filters read existing records and do not change academic-year lifecycle or
soft-delete state.
