# Research: Academic Year List Filters

## Decision 1: Use four explicit query parameters

- **Decision**: Add `name`, `date_from`, `date_to`, and `status` to
  `GET /api/v1/academic-years`.
- **Rationale**: Names mirror the resource and make the two range boundaries
  unambiguous in URLs, logs, validation errors, and frontend mapping.
- **Alternatives considered**: A single encoded date-range value was rejected
  because field-level validation would be less precise. Reusing `start_date` and
  `end_date` was rejected because those names imply exact entity-field matching.

## Decision 2: Date range uses inclusive overlap semantics

- **Decision**: Match when academic-year start is on or before `date_to` and
  academic-year end is on or after `date_from`.
- **Rationale**: Administrators searching a period should see every academic
  year active during any part of it. Inclusive comparisons handle boundary-day
  matches predictably.
- **Alternatives considered**: Exact-boundary matching was too restrictive.
  Requiring the academic year to be fully contained in the selected range would
  hide partially intersecting years.

## Decision 3: Require a complete valid range

- **Decision**: `date_from` and `date_to` are optional as a pair. Supplying one
  without the other is invalid; both use `YYYY-MM-DD`; `date_to` must be on or
  after `date_from`.
- **Rationale**: Partial ranges would introduce undocumented open-ended behavior
  and make UI/API results diverge.
- **Alternatives considered**: Open-ended ranges were rejected because the user
  requested a date range and the UI will submit a complete range picker value.

## Decision 4: Name uses case-insensitive contains matching

- **Decision**: Trim name input and apply partial matching within the tenant
  query using the database's existing case-insensitive collation behavior.
- **Rationale**: Academic-year labels are short human-entered values; partial
  matching supports inputs such as `2026` without requiring the exact label.
- **Alternatives considered**: Exact matching and prefix-only matching were less
  useful. A new search index was disproportionate for this bounded list.

## Decision 5: Status exposes the full academic-year lifecycle domain

- **Decision**: Accept `planned`, `active`, `closed`, and `inactive`.
- **Rationale**: Backend lifecycle behavior already recognizes all four values;
  the current shared UI incorrectly narrows the list to school boolean values.
- **Alternatives considered**: Active/inactive only was rejected because it
  cannot find planned or closed academic years and does not match backend state.

## Decision 6: Use a dedicated list Form Request

- **Decision**: Add an academic-year list request that normalizes blank text and
  validates pagination and all filter fields before the controller invokes the
  service.
- **Rationale**: This preserves thin controllers, field-level 422 errors, unknown
  query rejection, and the project constitution's request-validation boundary.
- **Alternatives considered**: Extending the generic service validation trait was
  rejected because range-pair validation and name normalization are specific to
  this endpoint.

## Decision 7: Keep service filtering direct

- **Decision**: Apply optional predicates in the existing
  `AcademicYearService` after school resolution and permission checks.
- **Rationale**: Four scalar predicates over one table are clear Eloquent query
  composition and do not justify a DTO or Repository.
- **Alternatives considered**: A new filter repository/service was rejected as
  unnecessary indirection for current scope.

## Decision 8: Frontend uses submitted draft state plus URL query state

- **Decision**: `AcademicYearFilters.vue` owns a local draft; submit emits the
  complete normalized filter set; the route-query composable owns accepted URL
  values and serialization; reset clears every filter.
- **Rationale**: Explicit submit matches existing school-search interaction and
  prevents requests on each keystroke. URL state makes reload and sharing stable.
- **Alternatives considered**: Reusing `SchoolFilters.vue` caused the current
  regression. A new Pinia store was rejected because state is route-local.

## Decision 9: No database migration or dependency

- **Decision**: Use existing `academic_years` columns and indexes; add no package.
- **Rationale**: School-scoped academic-year collections are bounded and current
  fields support all predicates. Performance will be verified with existing
  list expectations before any index change is justified.
- **Alternatives considered**: New composite/name indexes were deferred because
  no measured bottleneck exists.
