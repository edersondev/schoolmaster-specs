# Research: School List Filters

## Decision: Treat filters as an additive contract-first Schools list change

**Rationale**: The feature changes externally visible query behavior for
`GET /api/v1/schools`. The response shape remains the existing paginated School
collection, so this is additive within `/api/v1`, but OpenAPI must still define
accepted query parameters, validation, matching semantics, and pagination/sort
interaction before backend or frontend work begins.

**Alternatives considered**: Frontend-only filtering was rejected because it
would filter only the current page and could expose inconsistent pagination.
Backend-only undocumented filters were rejected by API-first governance.

## Decision: Use `document` as the CNPJ filter query parameter

**Rationale**: Feature `029-school-fields-tabs` made `document` the canonical
school contract field and rejected legacy `cnpj` payloads. Keeping the API
query parameter as `document` avoids reintroducing deprecated terminology. The
frontend can label the control as CNPJ while submitting `document`.

**Alternatives considered**: A `cnpj` query alias was rejected because it would
extend a deprecated field name. Supporting both `cnpj` and `document` was
rejected because duplicate query fields create conflict-resolution rules not
needed for this additive feature.

## Decision: Use exact matching for structured filters

**Rationale**: Status, administrative type, legal nature, management type,
pedagogical approach, INEP code, and document/CNPJ are structured identifiers
or enumerated values. Exact matching after input normalization prevents false
positive results and makes acceptance tests deterministic. INEP and document
filters compare digits after trimming and removing common punctuation.

**Alternatives considered**: Contains matching for INEP and document was
rejected because partial identifiers can return unrelated schools. Prefix
matching was rejected because it still creates ambiguous identity lookup
results.

## Decision: Use contains matching for text filters with case and accent normalization

**Rationale**: Name, email, city, and state are human-entered text fields.
Contains matching supports normal list search behavior, while case-insensitive
and accent-insensitive comparison avoids misses for common Portuguese names and
locations.

**Alternatives considered**: Exact text matching was rejected because users
would need full stored values. Case-only normalization was rejected because it
still misses accent variants such as `São` and `Sao`.

## Decision: Accept one value per categorical filter

**Rationale**: The user asked to filter by each field, not to build multi-value
facets. Single-value status and institutional filters keep the query contract,
validation, frontend controls, and tests simpler while still allowing
combination across fields through AND semantics.

**Alternatives considered**: Multi-value filters were rejected because they
increase parameter encoding and result semantics without a stated use case.
Mixed single/multiple behavior was rejected because it would make the UI and
contract harder to learn.

## Decision: Combine active filters with AND semantics

**Rationale**: Administrators use multiple filters to narrow the Schools list.
AND semantics makes each additional filter reduce the result set and matches
the feature's acceptance scenarios.

**Alternatives considered**: OR semantics was rejected because it expands
results and makes mixed identity/location/institutional filters less useful.
Per-filter custom semantics were rejected because they complicate validation
and testing.

## Decision: Persist active filters in the page URL query string

**Rationale**: URL query persistence supports refresh, browser navigation,
bookmarks, and shared filtered views. It also gives backend and frontend tests a
clear serialized contract for active filters.

**Alternatives considered**: Local component state was rejected because filters
would disappear on refresh. Session-only persistence was rejected because
filtered views would not be shareable.

## Decision: Reset to page 1 when filters change and preserve sorting

**Rationale**: Changing filters can reduce the available page count, so staying
on a later page can show an empty or invalid view. Preserving sort keeps user
intent stable while the result set changes.

**Alternatives considered**: Preserving the current page was rejected because
it can create confusing empty pages. Resetting sort was rejected because filter
changes do not imply a new sort preference.

## Decision: Preserve tenant visibility and current response envelopes

**Rationale**: Filters must narrow only the set of schools already visible to
the requester. Keeping the existing paginated success envelope and standard
validation/authorization errors avoids unnecessary contract churn.

**Alternatives considered**: Adding filter metadata to the response was
rejected because the frontend can derive active filters from the URL query.
Changing authorization behavior was rejected because filters are not a new
access path.

## Decision: Validate through OpenAPI, PHPUnit, and Vitest

**Rationale**: This feature spans the shared API contract, backend query
behavior, and frontend list workflow. Redocly validates contract publication,
PHPUnit verifies backend filtering and tenant safety, and Vitest verifies query
serialization, route synchronization, UI controls, empty state, and clear
behavior.

**Alternatives considered**: Manual-only testing was rejected because matching
semantics and tenant safety are regression-prone. Contract-only testing was
rejected because implementation can still drift from documented behavior.
