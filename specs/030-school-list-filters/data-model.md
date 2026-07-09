# Data Model: School List Filters

This feature adds filter criteria and frontend filter state for the existing
Schools list. It introduces no new persisted entity and no new tenant root.
`School` remains the v1 tenant root.

## School

**Purpose**: Tenant-root school profile shown in the Schools list.

**Fields Used By Filters**:

- `status`: Numeric lifecycle value; `1` active, `0` inactive.
- `inepCode`: Official 8-digit INEP school code.
- `document`: Canonical API field for CNPJ digits.
- `name`: School display name.
- `email`: School contact email.
- `address.city`: School city from the existing required address profile.
- `address.state`: School state from the existing required address profile.
- `administrativeTypeId`: Institutional administrative type reference.
- `legalNatureId`: Institutional legal nature reference.
- `managementTypeId`: Institutional management type reference.
- `pedagogicalApproachId`: Institutional pedagogical approach reference.

**Relationships**:

- Has one address profile containing `city` and `state`.
- References one administrative type.
- References one legal nature.
- References one management type.
- References one pedagogical approach.

**Validation and State Rules**:

- Filters do not create, update, delete, activate, deactivate, restore, or
  otherwise mutate schools.
- Filters reduce only the schools already visible to the requester through
  existing authorization and tenant visibility rules.
- Existing inactive, soft-deleted, forbidden, restored, and not-found behavior
  remains unchanged.
- CNPJ is represented by the canonical `document` contract field. This feature
  does not add a `cnpj` API query alias.

## SchoolListFilter

**Purpose**: Optional criteria used to narrow the Schools list.

**Fields**:

- `status`: Optional single numeric status value.
- `inepCode`: Optional normalized INEP code filter.
- `document`: Optional normalized CNPJ/document filter.
- `name`: Optional school name text filter.
- `email`: Optional email text filter.
- `city`: Optional city text filter.
- `state`: Optional state text filter.
- `administrativeTypeId`: Optional single administrative type ID.
- `legalNatureId`: Optional single legal nature ID.
- `managementTypeId`: Optional single management type ID.
- `pedagogicalApproachId`: Optional single pedagogical approach ID.

**Rules**:

- Multiple active filters combine with AND semantics.
- `status`, `administrativeTypeId`, `legalNatureId`, `managementTypeId`, and
  `pedagogicalApproachId` accept at most one selected value each.
- `status` accepts only documented school status values.
- Institutional IDs must reference approved option values.
- `inepCode` and `document` match exactly after trimming and removing common
  punctuation from submitted values.
- `name`, `email`, `city`, and `state` match as contains searches after
  trimming submitted values and ignoring casing and accent differences.
- Empty, null, or cleared filter values are not applied.
- Unsupported status or institutional values produce the standard validation
  error response.

## InstitutionalLookup

**Purpose**: Approved option values used by institutional school fields and
institutional list filters.

**Fields Used By Filters**:

- `id`: Stable numeric option identifier.
- `label`: Human-readable option label.
- `group`: One of administrative type, legal nature, management type, or
  pedagogical approach.
- `status`: Optional availability status if supported.

**Rules**:

- Frontend filter controls load option values from the approved lookup sources.
- This feature does not add runtime management of institutional lookup values.
- Missing, inactive, or unavailable lookup values must not create hidden active
  filters; invalid submitted lookup values are rejected through validation.

## SchoolListRouteState

**Purpose**: Frontend route-local state for the Schools list filters.

**Fields**:

- `filters`: Current `SchoolListFilter` values parsed from URL query
  parameters.
- `page`: Current page number.
- `perPage`: Current page size.
- `sort`: Current sort expression.
- `status`: Idle, loading, ready, validation-error, forbidden, unauthorized,
  unavailable, or error.
- `results`: Current paginated School collection.
- `fieldErrors`: Query/filter validation errors keyed by documented filter
  names.
- `lookupOptions`: Institutional option values for filter controls.

**Rules**:

- Active filters are represented in the page URL query string.
- Applying or changing any filter resets `page` to 1.
- Changing filters preserves current `sort` unless the user changes sort.
- Clearing one filter removes that filter from the URL query string and result
  request.
- Clearing all filters returns the list to unfiltered behavior while preserving
  existing authorization and tenant visibility.
- A bookmarked or shared URL with valid filters initializes the filter controls
  and request state.
- A bookmarked or manipulated URL with invalid status or lookup values shows
  standard validation feedback and does not mutate school data.
