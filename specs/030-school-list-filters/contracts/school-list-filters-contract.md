# School List Filters Contract

## Scope

This contract defines the required OpenAPI, backend, and frontend behavior for
filtering the Schools list.

The feature includes:

- New query filters on `GET /api/v1/schools`.
- Exact matching for status, institutional references, INEP code, and
  document/CNPJ.
- Contains matching for name, email, city, and state.
- Case-insensitive and accent-insensitive matching for text filters.
- Single selected value for each categorical filter.
- URL query-string persistence for frontend filter state.
- Existing pagination, sorting, authorization, tenant visibility, and response
  envelope preservation.

The feature excludes:

- New response fields.
- School create/edit, lifecycle, restore, delete, branding, address, or lookup
  management changes.
- New cross-tenant or platform support access paths.
- Runtime management UI for institutional lookup values.

## Approved/Required Operation

| Operation | Method and path | Use |
|-----------|-----------------|-----|
| `listSchools` | `GET /api/v1/schools` | Existing paginated school list narrowed by optional filters |

## Query Parameters

Existing pagination and sorting parameters remain supported.

| Query parameter | Required | Accepted values | Matching behavior |
|-----------------|----------|-----------------|-------------------|
| `page` | No | Integer >= 1 | Existing pagination behavior |
| `per_page` | No | Existing page-size contract | Existing pagination behavior |
| `sort` | No | Existing school list sort contract | Existing sorting behavior |
| `status` | No | One numeric school status value, `1` or `0` | Exact |
| `inep_code` | No | INEP code digits, punctuation ignored | Exact digits after normalization |
| `document` | No | CNPJ/document digits, punctuation ignored | Exact digits after normalization |
| `name` | No | Text | Contains after trim, case folding, and accent normalization |
| `email` | No | Text | Contains after trim, case folding, and accent normalization |
| `city` | No | Text | Contains after trim, case folding, and accent normalization |
| `state` | No | Text | Contains after trim, case folding, and accent normalization |
| `administrative_type_id` | No | One approved administrative type ID | Exact |
| `legal_nature_id` | No | One approved legal nature ID | Exact |
| `management_type_id` | No | One approved management type ID | Exact |
| `pedagogical_approach_id` | No | One approved pedagogical approach ID | Exact |

`document` is the canonical API query parameter for CNPJ filtering. The
frontend may label the control as CNPJ, but this feature does not introduce a
`cnpj` query parameter.

## Filter Semantics

- Multiple active filters combine with AND semantics.
- Empty or cleared filters are not applied.
- Leading and trailing whitespace is ignored for all submitted filters.
- INEP code and document filters remove common punctuation before matching
  stored digits.
- Status and institutional filters accept one value per field.
- Name, email, city, and state are contains searches that ignore casing and
  accent differences.
- Applying or changing any filter resets pagination to page 1.
- Applying or changing filters preserves active sort unless the user changes
  sort.
- Broad, exact, or manipulated filters must never return schools outside the
  requester's authorized visibility.

## Response Contract

Successful filtered responses use the existing paginated Schools collection
shape:

- Standard paginated success envelope.
- `data` array of documented School resources.
- Existing pagination metadata.
- No new response fields required for active filters.

No-result responses are successful paginated responses with an empty `data`
array and existing pagination metadata. The frontend must render a clear empty
state and allow filters to be cleared.

## Validation and Error Contract

Invalid query values use the existing standard validation error envelope with
field-level details keyed to documented query parameter names.

Validation applies to:

- Unsupported `status` values.
- Unsupported institutional lookup IDs.
- Query shapes that attempt multiple values for a single-value categorical
  filter.
- INEP or document values that cannot be normalized into an acceptable filter
  value.

Validation errors must not mutate school data and must not disclose
cross-tenant school existence.

## Authorization and Tenancy Contract

- Existing authenticated school administration authorization applies.
- Backend Policies remain authoritative for the list operation.
- School remains the v1 tenant root.
- Filters reduce only the already-authorized result set.
- No new cross-tenant access path is introduced.
- Platform or support access remains limited to previously approved school
  administration behavior.
- Soft-delete and lifecycle visibility rules remain unchanged.

## Frontend Contract

- The Schools list exposes controls for status, INEP code, CNPJ/document, name,
  email, city, state, administrative type, legal nature, management type, and
  pedagogical approach.
- Status and institutional controls allow one selected value per field.
- Institutional option controls load labels from approved lookup sources.
- Active filters are represented in the page URL query string.
- Opening a bookmarked or shared filtered URL initializes the filter controls
  and list request from the query string.
- Clearing one filter removes only that filter from the query string and next
  request.
- Clearing all filters returns to existing unfiltered list behavior.
- Invalid filters from a bookmarked or manipulated URL show validation feedback
  without clearing unrelated valid filter controls.
- The empty result state preserves active filters and provides a clear action
  to clear filters.

## Verification Contract

Specification repository:

- OpenAPI lint passes after adding all query parameters and validation response
  references.

Backend repository:

- PHPUnit verifies each filter individually.
- PHPUnit verifies combined filters use AND semantics.
- PHPUnit verifies exact matching for status, institutional references, INEP,
  and document.
- PHPUnit verifies contains matching for name, email, city, and state.
- PHPUnit verifies case-insensitive and accent-insensitive text matching.
- PHPUnit verifies invalid status and lookup values return validation errors.
- PHPUnit verifies filters preserve authorization and tenant visibility.
- PHPUnit verifies pagination resets are driven by request parameters and
  response shape remains unchanged.

Frontend repository:

- Vitest verifies service query serialization uses documented parameter names.
- Vitest verifies no `cnpj` query parameter is submitted.
- Vitest verifies URL query synchronization, refresh initialization, clear-one,
  and clear-all behavior.
- Vitest verifies status and institutional controls accept one selected value.
- Vitest verifies empty result state and validation feedback behavior.
