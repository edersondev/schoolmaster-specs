# Research: Centralize Addresses

## Decision: Update OpenAPI Before Backend or Frontend Changes

Replace `address_summary` with structured `address` schemas in the aggregate
OpenAPI contract before implementation exposes or consumes the behavior.

**Rationale**: The current published school schemas still expose
`address_summary`. Contract-first delivery requires request fields, response
fields, nullable removal behavior, validation errors, and embedded session
school shape to be approved before backend or frontend code changes.

**Alternatives considered**:

- Backend-first migration with later OpenAPI update: rejected because it creates
  undocumented external behavior.
- Add standalone address endpoints first: rejected because the feature approves
  only school create/update/read surfaces and no standalone address API.

## Decision: Model Address as Reusable Owned Data, School First

Introduce a reusable address concept with `School` as the only approved owner
type in this slice. Other owner types require future specifications and
contract changes.

**Rationale**: The user requirement is to centralize address information for
future models, but the only currently approved behavior is school address
replacement. Planning the reusable ownership boundary now prevents duplicated
address fields while avoiding undocumented non-school behavior.

**Alternatives considered**:

- Keep address fields directly on `schools`: rejected because it does not
  centralize future addressable records.
- Approve all future addressable models now: rejected because no business rules
  or authorization semantics are specified for those owners.

## Decision: Keep `school_id` on School-Owned Address Rows

Use Laravel polymorphic ownership columns generated from `morphs('addressable')`
to identify the owning record and also store `school_id` directly on address
rows owned by schools.

**Rationale**: The polymorphic columns support the reusable address model, while
`school_id` satisfies SchoolMaster v1 tenant-by-column rules and keeps tenant
scope explicit before owner lookup, validation, authorization, mutation, audit,
or response shaping.

**Alternatives considered**:

- Rely only on `addressable_type`/`addressable_id`: rejected because it makes
  tenant scope indirect and conflicts with v1 `school_id` guidance for
  school-owned records.
- Add non-school owners now to justify a generic tenant strategy: rejected
  because future owner types require their own specifications and contracts.

## Decision: Enforce One Active Address with a MySQL-Safe Constraint

The addresses table must enforce one active address for the same
`school_id`/`addressable_type`/`addressable_id` owner while allowing
soft-deleted historical rows to remain recoverable.

**Rationale**: MySQL unique indexes that include nullable `deleted_at` do not
reliably enforce one active row because multiple `NULL` values are allowed. A
generated active marker, or equivalent deterministic constraint, makes the rule
database-enforced without losing soft-deleted history.

**Alternatives considered**:

- Unique index on owner columns plus `deleted_at`: rejected because it can allow
  duplicate active rows.
- Service-only duplicate prevention: rejected because concurrent updates need a
  database-backed invariant.

## Decision: Use Existing School Routes, Not Standalone Address Routes

School address create, read, replace, and remove behavior is exposed through
the existing school create/detail/update/list/session surfaces after OpenAPI
changes.

**Rationale**: The specification explicitly blocks standalone address endpoints
and keeps authorization tied to the owning record. Reusing school operations
preserves current school management permissions and response envelopes.

**Alternatives considered**:

- `POST /api/v1/addresses`: rejected as an unapproved standalone address
  endpoint.
- Nested `/api/v1/schools/{schoolId}/address`: rejected for this slice because
  it expands the API surface beyond the clarified requirement.

## Decision: Represent Number and Zip Code as Digit-Only Strings

The API contract should represent `number` and `zip_code` as strings with a
digits-only constraint. Backend storage may use narrower physical types when
the response remains compatible with the documented API string shape.

**Rationale**: The user requires these values to accept only numbers. Strings
with numeric patterns preserve leading zeros, avoid integer range issues, and
still allow validation to reject letters, spaces, and punctuation.

**Alternatives considered**:

- Integer API fields: rejected because zip codes and address numbers can
  contain leading zeros and should not be used for arithmetic by consumers.
- Free-text strings: rejected because they do not enforce the clarified
  numeric-only requirement.

## Decision: Explicit Null Removes Address on Update

Omitting `address` on school update means no address change. Submitting
`address: null` removes the current school address and must follow the same
recoverability/audit expectations as administrative data.

**Rationale**: This separates "not changing address" from "remove address" and
fits the one-current-address school scope without a new endpoint.

**Alternatives considered**:

- Empty address object removes address: rejected because it conflicts with
  required-field validation.
- Disallow address removal: rejected because the specification requires removal
  to be recoverable.

## Decision: Drop Legacy Summary Data During Development Reset

Legacy `address_summary` values do not need to be preserved because the
application is still in development and existing data can be deleted and
recreated.

**Rationale**: Avoiding a one-time parser and exception workflow keeps the
feature focused on the final schema and public contract.

**Alternatives considered**:

- Preserve unparseable summaries as migration exceptions: rejected as
  unnecessary for a development-only dataset.
- Parse summaries into structured addresses: rejected because the data can be
  recreated cleanly.

## Decision: Reject Legacy `address_summary` Immediately After Publication

Public school create/update requests that submit `address_summary` after the
structured contract is published must fail validation.

**Rationale**: Immediate rejection avoids ambiguous compatibility behavior and
forces clients to adopt the authoritative contract.

**Alternatives considered**:

- One-release compatibility window: rejected because it allows stale clients to
  believe legacy input still works.
- Migration-tool-only `address_summary`: rejected for public API behavior; any
  internal migration handling must remain outside public request contracts.
