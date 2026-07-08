# Contract Boundary: Centralize Addresses

## Purpose

This feature replaces the legacy school `address_summary` contract with a
structured `address` object on approved school surfaces. Backend and frontend
implementation must wait until OpenAPI documents the updated schemas,
validation behavior, nullable removal behavior, response shape, and rejection
of legacy input.

## Authoritative Contract Files

- Aggregate OpenAPI: `api/openapi.yaml`
- School schema components:
  - `api/components/schemas/schools/School.yaml`
  - `api/components/schemas/schools/SchoolCreateRequest.yaml`
  - `api/components/schemas/schools/SchoolUpdateRequest.yaml`
  - `api/components/schemas/auth/AuthSession.yaml`
- School paths:
  - `api/paths/schools/index.yaml`
  - `api/paths/schools/school.yaml`

## Affected Operation Boundary

| Operation ID | Route | Required Contract Change |
|--------------|-------|--------------------------|
| `listSchools` | `GET /api/v1/schools` | Return structured `address` in each school representation where address data is exposed. |
| `createSchool` | `POST /api/v1/schools` | Accept optional structured `address`; reject `address_summary`. |
| `getSchool` | `GET /api/v1/schools/{schoolId}` | Return structured `address` or no address according to schema. |
| `updateSchool` | `PATCH /api/v1/schools/{schoolId}` | Accept omitted `address` as no change, address object as replacement, and explicit `address: null` as removal; reject `address_summary`. |
| Authenticated session school embedding | `GET /api/v1/auth/me` and login/session responses where applicable | Embedded `resolved_school` uses the updated `School` schema. |

No standalone address endpoints are approved.

## Address Schema Requirements

OpenAPI must add an address component equivalent to:

```yaml
Address:
  type: object
  required:
    - id
    - street
    - number
    - neighborhood
    - city
    - state
    - zip_code
  properties:
    id:
      type: string
      format: uuid
    street:
      type: string
    number:
      type: string
      pattern: '^[0-9]+$'
      maxLength: 10
    complement:
      type:
        - string
        - 'null'
    neighborhood:
      type: string
    city:
      type: string
    state:
      type: string
      maxLength: 4
    zip_code:
      type: string
      pattern: '^[0-9]+$'
      maxLength: 12
    country:
      type:
        - string
        - 'null'
```

Request schemas may use an input variant without `id` but must preserve the
same required and optional address fields. The public API keeps `number` and
`zip_code` as digit-only strings; backend storage may use narrower physical
types as long as responses preserve the documented string shape.

## School Schema Changes

- Remove `address_summary` from `School`, `SchoolCreateRequest`, and
  `SchoolUpdateRequest`.
- Add `address` to `School`; it may be an `Address` object or null/absent
  according to the aggregate schema convention selected during OpenAPI update.
- Add optional `address` object to `SchoolCreateRequest`; if present, required
  address fields must be valid.
- Add optional `address` to `SchoolUpdateRequest`:
  - omitted `address`: no address change
  - address object: replace current school address
  - explicit `null`: remove current school address
- `additionalProperties: false` must reject unknown fields, including
  `address_summary`, wherever request schemas currently enforce strict input.

## Validation Behavior

- Missing `street`, `number`, `neighborhood`, `city`, `state`, or `zip_code`
  in a submitted address object returns the standard validation error envelope.
- Non-numeric `number` or `zip_code` values return the standard validation
  error envelope.
- `country` is optional and must not be inferred from locale, deployment, or
  tenant context.
- `address_summary` in public school create/update requests returns the
  standard validation error envelope immediately after publication.
- Cross-school address access must use the same tenant-safe denial behavior as
  school access and must not disclose another school's address existence.

## Authorization and Tenant Behavior

- School address management uses existing school management authorization.
- School-owned address persistence carries direct `school_id` tenant ownership
  in addition to polymorphic owner columns.
- Platform-scoped school management remains separate from school-scoped module
  administration.
- No user may access or mutate an address through an owner they cannot access.
- Future school-owned address owners must inherit tenant isolation through the
  owning record and require future specifications before exposure.
- Backend queries must resolve permitted school context before address lookup or
  mutation and must scope address persistence by `school_id`.

## Migration Contract Boundary

- The application is still in development; legacy data preservation is not
  required.
- The schema migration may drop `schools.address_summary` directly.
- Existing development data may be deleted and recreated with fresh migrations
  and seeders.
- No public or internal migration exception workflow is approved for this
  feature slice.

## Blocked Until Future Specification

- Standalone address endpoints.
- Non-school address owner types.
- Multiple current addresses for one school.
- Address search, bulk address import, geocoding, normalization, validation
  against postal databases, coordinates, maps, or external address services.
- Frontend behavior that consumes undocumented address fields.

## Concrete Contract Assertions

- `Address` and `AddressInput` schemas require `street`, `number`,
  `neighborhood`, `city`, `state`, and `zip_code`.
- `number` and `zip_code` are strings constrained by `^[0-9]+$`.
- `School` responses expose `address` as a structured object or `null`; public
  school response contracts do not expose `address_summary`.
- `SchoolCreateRequest` accepts omitted `address` or a complete structured
  address object and rejects legacy or unknown fields through strict request
  validation.
- `SchoolUpdateRequest` preserves the three update states: omitted `address`
  is no-op, structured object replaces, and explicit `null` removes.
- Future address owners must be added to the backend owner registry and specs
  before public exposure; direct standalone address routes remain blocked.

## Verification Boundary

Implementation PRs must include evidence for:

- Redocly/OpenAPI validation after schema changes.
- Backend feature tests for school create with address, school update address
  replacement, omitted address no-op, explicit null removal, and school read
  response shape.
- Backend validation tests for required fields, digit-only `number` and
  `zip_code`, unknown fields, and rejected `address_summary`.
- Backend tenant/authorization tests for cross-school address denial and
  platform/school permission separation.
- Migration tests or full-suite migration coverage verifying the legacy
  `schools.address_summary` column is removed during development reset.
- Frontend tests, when implemented, for structured address form mapping,
  validation display, null removal intent, and rejection of legacy field usage.
