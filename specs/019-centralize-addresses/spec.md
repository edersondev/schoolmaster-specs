# Feature Specification: Centralize Addresses

**Feature Branch**: `019-centralize-addresses`  
**Created**: 2026-06-26  
**Status**: Draft  
**Input**: User description: "About the table schools, I want to change the column address_summary. For address I want a dedicated table for that, type polymorphic, here the columns: id, street, number, complement, neighborhood, city, state, zip_code, country, commentable_type, commentable_id, timestamp, softdelete. Other models will have address, so I want to centralize the information."

## Clarifications

### Session 2026-06-26

- Q: How should existing school address summaries be handled while the app is
  still in development? → A: Do not preserve them; development data can be
  deleted and recreated.
- Q: How should omitted country values be handled for school addresses? → A: Leave omitted country absent/null.
- Q: How should submitted address data behave when a school already has an address? → A: Submitted school address data replaces the school's current address.
- Q: How should a school address be removed? → A: Submit an explicit null address in the approved school update flow.
- Q: How should legacy address_summary input be handled after the structured address contract is published? → A: Reject address_summary immediately.
- Q: Were `commentable_type` and `commentable_id` intended as the address table
  column names? → A: No; those were examples from comments. The address table
  must follow Laravel polymorphic naming for an address owner:
  `addressable_type` and `addressable_id`, generated from
  `morphs('addressable')` for the current School owner key type.
- Q: How does the polymorphic address table satisfy SchoolMaster v1 tenant
  rules? → A: School-owned address rows must also carry `school_id` directly.
  The polymorphic owner columns identify the owning record; `school_id` is the
  concrete tenant column used for tenant isolation.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Maintain Structured School Addresses (Priority: P1)

An authorized administrator can create, review, and update a school's address
as structured address information instead of a single free-text summary.

**Why this priority**: Schools are the tenant roots and their address
information appears in administration, support, reporting, and future
communications. Structured fields reduce ambiguity and prepare the platform for
consistent address handling.

**Independent Test**: Create a school with a complete structured address,
retrieve the school, update one address field, and verify the returned school
shows the updated structured address without relying on a free-text address
summary.

**Acceptance Scenarios**:

1. **Given** an authorized administrator creates a school with street, number,
   neighborhood, city, state, and zip code, **When** the school is saved,
   **Then** the school profile includes those address fields as structured
   address information.
2. **Given** an existing school has structured address information, **When**
   an authorized administrator updates the school address, **Then** the change
   applies only to that school's address and the school profile no longer
   depends on a single address summary.
3. **Given** a school has no address yet, **When** the school is retrieved,
   **Then** the response clearly represents the address as absent rather than
   returning stale or synthesized summary text.
4. **Given** an authorized administrator submits letters or symbols in the
   number or zip code fields, **When** the address is validated, **Then** the
   address is rejected and the administrator can correct those fields.

---

### User Story 2 - Reuse Address Handling for Other Records (Priority: P2)

Product teams can attach the same structured address concept to future
addressable records without redefining address fields separately for each
model.

**Why this priority**: Other SchoolMaster records will need addresses. A
central address model avoids duplicated field definitions, inconsistent
validation, and divergent lifecycle behavior.

**Independent Test**: Review the specification and contract changes and verify
that the address concept is defined once, can be owned by one approved record,
and does not introduce new undocumented endpoints for unrelated models.

**Acceptance Scenarios**:

1. **Given** a future approved feature needs an address for a different record
   type, **When** that feature is specified, **Then** it can reuse the same
   address fields and lifecycle rules without inventing a separate address
   summary field.
2. **Given** an address belongs to a specific record, **When** that record is
   accessed, **Then** only authorized users who can access the owning record can
   view or manage its address.

---

### User Story 3 - Drop Legacy Summary During Development Reset (Priority: P3)

Developers can rebuild the development database with the new structured address
schema and no legacy school address summary column.

**Why this priority**: The app is still in development, so preserving legacy
free-text address data adds unnecessary one-time migration complexity.

**Independent Test**: Run migrations from scratch and verify school contracts
use structured `address` fields without `address_summary`.

**Acceptance Scenarios**:

1. **Given** the development database is rebuilt, **When** migrations run,
   **Then** the schema contains centralized addresses and no public
   `schools.address_summary` contract.
2. **Given** existing development data is discarded, **When** seeders or tests
   recreate schools, **Then** address data is supplied only through structured
   fields.
3. **Given** a school is retrieved through approved contracts, **When** the
   response is returned, **Then** consumers see structured address fields and
   not the legacy summary field.

### Edge Cases

- How does the system handle existing free-text address summaries during
  development reset?
- What happens when an address is removed from a school while the school itself
  remains active?
- How does the system prevent users from viewing or changing an address through
  a record they are not authorized to access?
- How does the system handle duplicate address submissions for the same owning
  record when only one primary address is approved for this slice?
- What happens when a client still submits the legacy school address summary
  after the contract has moved to structured addresses?
- What happens when an address number or zip code includes letters, spaces, or
  punctuation?

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST replace the school free-text address summary
  contract with structured address information for school create, detail,
  update, list, and authenticated session school representations where school
  address information is exposed.
- **FR-002**: Structured address information MUST define street, number,
  neighborhood, city, state, and zip code as core fields, and MUST also define
  complement and country as optional fields.
- **FR-003**: Street, number, neighborhood, city, state, and zip code MUST be
  validated as required when an address is submitted for a school.
- **FR-004**: Number and zip code MUST accept numeric characters only.
- **FR-005**: Complement and country MAY be omitted when an address is submitted
  for a school. Omitted country values MUST remain absent/null and MUST NOT be
  defaulted from locale or deployment context.
- **FR-006**: A school MAY exist without an address, but if any address field
  is submitted the complete required address fields MUST be valid.
- **FR-007**: Each address MUST belong to exactly one approved owning record at
  a time using Laravel polymorphic ownership columns `addressable_type` and
  `addressable_id`, generated from the `addressable` morph name. School-owned
  address rows MUST also carry `school_id` as the concrete v1 tenant column.
- **FR-008**: Submitted school address data MUST replace the school's current
  address through the approved school create or update flow. This slice MUST
  NOT keep multiple current addresses for one school. The persistence layer MUST
  enforce one active address per owner while allowing soft-deleted historical
  address rows to remain recoverable.
- **FR-009**: This feature MUST approve school-owned addresses first and MUST
  define reusable ownership rules that future approved features can apply to
  other addressable records.
- **FR-010**: Address visibility and management MUST be authorized through the
  owning record's existing authorization boundary.
- **FR-011**: School address changes MUST preserve tenant isolation and MUST
  NOT allow a user to infer or modify another school's address through
  validation, authorization, query scope, or not-found behavior.
- **FR-012**: A school address MUST be removed only when the approved school
  update flow submits an explicit null address. Omitted address data MUST NOT
  remove or modify the current school address.
- **FR-013**: Removing an address from a school MUST soft-delete the address so
  the record remains recoverable by internal administrative data recovery
  workflows. This feature MUST NOT expose public address restore endpoints.
- **FR-014**: Existing development `schools.address_summary` data does not need
  to be preserved. The migration MAY drop the legacy column directly because
  development data can be deleted and recreated.
- **FR-015**: The system MUST reject legacy address summary input immediately
  after the structured address contract is published.
- **FR-016**: The system MUST NOT expose standalone address endpoints or allow
  address management for non-school records until those record types are
  approved by future specifications and OpenAPI contracts.
- **FR-017**: Address responses MUST follow the approved success and error
  envelope conventions for school operations and MUST NOT add undocumented
  response fields.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Specifications**: Add this feature specification and update relevant
  downstream plans, data models, and tasks before backend implementation.
- **Backend**: Change school address behavior from a free-text school summary
  to structured reusable address ownership, including validation,
  authorization, development schema reset, recovery, and tests.
- **Frontend**: Update school administration forms and displays to use
  structured address fields instead of address summary once the OpenAPI
  contract is updated and implemented.

### API Contract Impact

- **OpenAPI changes required**: Yes. Update school schemas used by
  `listSchools`, `createSchool`, `getSchool`, `updateSchool`, and any
  authenticated session response that embeds a school representation.
- **Approved API boundary**: Existing `/api/v1/schools` and authenticated
  session surfaces may expose school address data after OpenAPI is updated.
  No standalone address endpoints are approved in this specification.
- **Response and error semantics**: School operations continue to use the
  approved success, paginated, validation, unauthorized, forbidden,
  tenant-mismatch, inactive-record, and not-found envelopes.

### Tenant and Authorization Impact

- **Tenant-owned data**: School addresses are governed by the school that owns
  them. Future addressable school-owned records must preserve school tenant
  isolation through the owning record.
- **Authorization boundary**: Platform-scoped school management permissions
  govern school address management. Future owner types must inherit the
  authorization boundary of the owning record and must not create an implicit
  cross-tenant bypass.

### Key Entities *(include if feature involves data)*

- **Address**: Structured postal address with street, number, complement,
  neighborhood, city, state, zip code, country, lifecycle state, and ownership
  by one approved addressable record through `addressable_type` and
  `addressable_id`. School-owned address rows also carry `school_id` for direct
  tenant isolation. For schools, submitted address data replaces the current
  address rather than creating multiple current addresses.
- **School**: Platform-managed tenant root whose profile may include one
  structured address.
- **Address Owner**: Approved record that owns one address and controls address
  visibility, authorization, tenant isolation, and lifecycle behavior.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of school address fields exposed through approved contracts
  are structured address fields rather than a legacy summary field.
- **SC-002**: 100% of existing schools with address summaries are either
  converted to structured address information or flagged for administrator
  correction without losing the original address text.
- **SC-003**: Authorized administrators can create or update a school address
  using structured fields in under 2 minutes during acceptance testing.
- **SC-004**: Tenant-isolation tests reject 100% of attempts to view or modify
  an address through an unauthorized or mismatched school context.
- **SC-005**: Contract review confirms no standalone address endpoints or
  non-school address owner types are exposed before a future specification
  approves them.
- **SC-006**: Validation tests reject 100% of school address submissions where
  number or zip code contains non-numeric characters.
- **SC-007**: Validation tests reject 100% of public school create or update
  requests that submit the legacy address summary field after structured
  address publication.

## Assumptions

- The first approved owner type for centralized addresses is School.
- Each school has at most one address in this slice.
- Address country values are optional in this slice and are stored only when
  explicitly supplied.
- Existing development data may be deleted and recreated; no legacy
  `address_summary` parser or preservation workflow is required.
- Future address owner types require their own specification and OpenAPI
  updates before backend or frontend behavior is exposed.
