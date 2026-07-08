# Data Model: Centralize Addresses

## Address

**Purpose**: Reusable structured postal address owned by one approved
addressable record. In this slice, the only approved owner is `School`.

**Fields**:

- `id`: UUID public identifier.
- `street`: Required string.
- `number`: Required digit-only API string that must fit unsigned integer
  storage.
- `complement`: Optional nullable string.
- `neighborhood`: Required string.
- `city`: Required string.
- `state`: Required string, maximum 4 characters.
- `zip_code`: Required string containing digits only, maximum 12 characters.
- `country`: Optional nullable string; remains absent/null when omitted.
- `school_id`: Required tenant column for school-owned address rows in this
  slice. It stores the owning school's internal identifier and is used for
  tenant scoping before owner-specific lookup or mutation.
- `addressable_type`: Internal owner type discriminator created by
  `$table->morphs('addressable')`. Only School is approved in this slice.
- `addressable_id`: Internal owner identifier created by
  `$table->morphs('addressable')`; for the current School owner this follows
  the internal School primary key type.
- `active_owner_marker`: Internal nullable generated marker, or equivalent
  MySQL-safe mechanism, set only for non-deleted rows so a unique index can
  enforce one current address per owner while allowing multiple soft-deleted
  historical rows.
- `created_at`: Creation timestamp.
- `updated_at`: Last update timestamp.
- `deleted_at`: Soft-delete timestamp for recoverable removal.

**Validation Rules**:

- `street`, `number`, `neighborhood`, `city`, `state`, and `zip_code` are
  required when an address object is submitted.
- `number` and `zip_code` accept digits only; letters, spaces, punctuation, and
  symbols are rejected.
- `number` must fit unsigned integer storage; API responses still expose it as
  a string for compatibility.
- `state` is limited to 4 characters and `zip_code` is limited to 12
  characters.
- `complement` and `country` are optional.
- Unknown address fields are rejected.
- Legacy `address_summary` is rejected immediately after the structured
  contract is published.

**Relationships**:

- Belongs to exactly one approved address owner.
- For this slice, belongs to one `School`.
- Carries direct `school_id` tenant ownership for every school-owned row.
- Future owner relationships require future specifications and OpenAPI
  contracts.

**Lifecycle**:

- Create: School create may include a valid address object.
- Read: School list/detail/session representations expose current structured
  address or null/absent address according to OpenAPI.
- Replace: School update with a valid address object replaces the school's
  current address.
- Remove: School update with explicit `address: null` removes the school
  address and soft-deletes the address record.
- Omit: School update without an `address` member leaves the current address
  unchanged.
- Restore: Removal is recoverable through internal administrative data recovery
  of the soft-deleted record. No public address restore endpoint is approved in
  this feature.

**Uniqueness Rules**:

- Only one active address may exist for the same
  `school_id`/`addressable_type`/`addressable_id` owner tuple.
- Soft-deleted address rows must remain recoverable and must not block a new
  current address for the same owner.
- MySQL implementation must not rely on a nullable `deleted_at` column alone for
  uniqueness. Use a generated active marker or an equivalent deterministic
  constraint that distinguishes active rows from deleted history.

## School

**Purpose**: Platform-managed tenant root whose profile may include one current
structured address.

**Address Rules**:

- A school may exist without an address.
- A school may have at most one current address in this slice.
- Submitted school address data replaces the current school address rather than
  creating multiple current addresses.
- School address visibility and mutation are authorized through the existing
  school management boundary.
- Cross-school address access must not disclose another school's existence or
  address state through validation, authorization, or not-found behavior.

## Development Data Reset

**Purpose**: This application is still in development, so legacy
`schools.address_summary` data is not preserved.

**Rules**:

- The schema migration may drop `schools.address_summary` directly.
- Existing development data may be deleted and recreated from migrations and
  seeders.
- No backfill, parser, or migration exception workflow is required for this
  feature slice.
