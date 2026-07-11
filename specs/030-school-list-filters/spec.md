# Feature Specification: School List Filters

**Feature Branch**: `030-school-list-filters`  
**Created**: 2026-07-08  
**Status**: Draft  
**Input**: User description: "In the list of Schools I want to be able to filter by: status, inep code, name, cnpj, email, city, state, administrative type, legal nature, management type, pedagogical approach"

## Clarifications

### Session 2026-07-09

- Q: What matching behavior should the school list filters use? → A: Exact match for status/lookups/INEP/CNPJ; contains match for name/email/city/state.
- Q: Should status and institutional filters accept one value or multiple values per field? → A: Status and institutional filters accept one value each.
- Q: How should active Schools list filters persist during navigation and refresh? → A: Persist active filters in the page URL query string.
- Q: Should text filters account for accented characters? → A: Name, email, city, and state text filters ignore accents and casing.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Find schools by identity and contact fields (Priority: P1)

An authorized school administrator filters the Schools list by status, INEP code, school name, CNPJ, email, city, or state so they can quickly locate a specific school or smaller working set.

**Why this priority**: School administration becomes difficult as tenant counts grow. Identity, contact, and location filters cover the most common lookup tasks and directly reduce manual scanning.

**Independent Test**: Can be fully tested by opening the Schools list, applying each identity, contact, and location filter individually and in combination, and confirming that only matching schools remain visible.

**Acceptance Scenarios**:

1. **Given** the Schools list contains active and inactive schools, **When** an administrator filters by status, **Then** the list shows only schools with the selected status.
2. **Given** schools exist with different INEP codes and CNPJ values, **When** an administrator enters a complete code or document search value, **Then** the list shows only schools whose stored INEP code or CNPJ exactly matches the submitted digits.
3. **Given** schools exist with different names, emails, cities, and states, **When** an administrator applies one or more of those filters, **Then** the list shows only schools containing the submitted text in every applied text field.
4. **Given** an administrator clears one or all applied filters, **When** the list refreshes, **Then** the cleared criteria no longer restrict the results.

---

### User Story 2 - Narrow schools by institutional classification (Priority: P2)

An authorized school administrator filters the Schools list by administrative type, legal nature, management type, and pedagogical approach so they can review schools by institutional profile.

**Why this priority**: Institutional classifications were added to the school profile and need matching list discovery so administrators can audit and operate on grouped school types.

**Independent Test**: Can be fully tested by applying each institutional filter with known lookup values and confirming that the list includes only schools assigned to the selected classification values.

**Acceptance Scenarios**:

1. **Given** schools have administrative type, legal nature, management type, and pedagogical approach values, **When** an administrator selects one value for one institutional filter, **Then** the list shows only schools with that selected value.
2. **Given** one value is selected for multiple institutional filters, **When** the administrator applies them together, **Then** the list shows only schools matching every selected institutional value.
3. **Given** an institutional lookup value is unavailable or no longer valid, **When** the administrator opens or refreshes the Schools list, **Then** the list avoids applying an invalid hidden filter and presents a recoverable state.

---

### User Story 3 - Preserve list behavior while filters change (Priority: P3)

An authorized administrator keeps pagination, sorting, tenant visibility, and list actions predictable while applying, changing, sharing, or clearing school filters.

**Why this priority**: Filters must improve discovery without breaking existing list behavior, tenant boundaries, or downstream navigation from a selected school.

**Independent Test**: Can be fully tested by applying filters across paginated and sorted list states, refreshing or reopening the list with filter criteria present, and verifying that authorization and tenant visibility remain unchanged.

**Acceptance Scenarios**:

1. **Given** the administrator is on a later page, **When** they apply or change any filter, **Then** results restart from the first page for the filtered set.
2. **Given** the administrator has an active sort order, **When** they apply filters, **Then** the sort order remains active unless the administrator changes it.
3. **Given** a filter combination returns no schools, **When** the filtered list renders, **Then** the administrator sees an empty result state that preserves the applied filters in the page URL query string and allows clearing them.
4. **Given** a user is not authorized to see a school, **When** they apply broad or exact filters that would otherwise match it, **Then** that school remains hidden.

### Edge Cases

- Filter values contain extra spaces, mixed letter casing, punctuation in CNPJ, or masked/unmasked digit formats.
- INEP code or CNPJ filter values are shorter than the complete stored value and therefore do not match exact-code filters.
- Name, email, city, or state filters contain accents or casing that differ from stored values and still match equivalent normalized text.
- Multiple filters are applied and no school matches all criteria.
- A selected institutional lookup value becomes inactive, missing, or unavailable while the Schools list is open.
- The requester changes school context, session state, or permissions while filtered results are visible.
- Filters are applied while pagination or sorting is active.
- Invalid status or lookup filter values are submitted by a stale UI, bookmarked URL, or manipulated request.
- A bookmarked or shared URL contains active filter criteria when the Schools list opens.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: School list behavior must accept and validate the documented filter criteria, apply filters only within the requester's existing school visibility, and preserve current pagination, sorting, authorization, and response envelope behavior.
- **Frontend repository impact**: The Schools list must expose controls for status, INEP code, name, CNPJ, email, city, state, administrative type, legal nature, management type, and pedagogical approach; load institutional filter options from the approved lookup sources; preserve active filters in the page URL query string during refreshes and navigation; and allow filters to be cleared.
- **Specification or contract repository impact**: The school list specification and OpenAPI contract must document the added query filters before implementation begins.
- **Delivery ownership and sequencing**: Specification and OpenAPI contract updates lead, backend support follows, and frontend list controls follow after the contract is approved. Frontend and backend work should be tracked under the shared `030-school-list-filters` feature identifier.

### API Contract Impact

- **OpenAPI update required**: Yes. The Schools list operation must document query filters for status, INEP code, document/CNPJ, name, email, city, state, administrative type, legal nature, management type, and pedagogical approach.
- **Versioned endpoints affected**: `/api/v1/schools`.
- **JSON response impact**: No new response fields are required. Filtered responses must keep the existing paginated school collection shape and existing success and error envelopes.
- **Authentication/authorization impact**: Existing school administration authentication, authorization, tenant-root visibility, and support-access rules remain authoritative. Filters must never expand the set of schools visible to the requester.
- **Compatibility impact**: Additive contract change. Existing list requests without the new filters continue to behave as they do today.

### Data & Tenancy Impact

- **Tenant scoping impact**: School remains the v1 tenant root. Filters reduce only the already-authorized school result set and do not introduce new tenant ownership rules.
- **Cross-tenant or platform access impact**: No new cross-tenant access path is introduced. Existing platform or support access remains limited to approved school administration behavior.
- **Soft delete impact**: Filtered list results must preserve existing school list behavior for inactive, deleted, restored, forbidden, and not-found schools. This feature does not change lifecycle transitions.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST allow authorized users to filter the Schools list by status.
- **FR-002**: The system MUST allow authorized users to filter the Schools list by INEP code.
- **FR-003**: The system MUST allow authorized users to filter the Schools list by school name.
- **FR-004**: The system MUST allow authorized users to filter the Schools list by CNPJ using the canonical school `document` value while keeping `document` as the contract field name.
- **FR-005**: The system MUST allow authorized users to filter the Schools list by email.
- **FR-006**: The system MUST allow authorized users to filter the Schools list by city.
- **FR-007**: The system MUST allow authorized users to filter the Schools list by state.
- **FR-008**: The system MUST allow authorized users to filter the Schools list by administrative type.
- **FR-009**: The system MUST allow authorized users to filter the Schools list by legal nature.
- **FR-010**: The system MUST allow authorized users to filter the Schools list by management type.
- **FR-011**: The system MUST allow authorized users to filter the Schools list by pedagogical approach.
- **FR-012**: The system MUST combine multiple supplied filters with AND semantics so every returned school matches every active filter.
- **FR-012a**: The system MUST allow at most one selected value for each categorical filter: status, administrative type, legal nature, management type, and pedagogical approach.
- **FR-013**: The system MUST ignore leading and trailing whitespace in text and code filters before matching.
- **FR-014**: The system MUST match name, email, city, and state filters without requiring exact letter casing or exact accent marks.
- **FR-015**: The system MUST match status, administrative type, legal nature, management type, pedagogical approach, INEP code, and CNPJ/document filters exactly after normalizing accepted input formats; INEP code and CNPJ/document matching must compare digits after removing common punctuation from the submitted filter value.
- **FR-015a**: The system MUST match name, email, city, and state filters as contains searches after trimming submitted values and ignoring letter casing and accent differences.
- **FR-016**: The system MUST validate status and institutional filter values against the documented accepted values and return the standard validation response for unsupported values.
- **FR-017**: The system MUST reset filtered list pagination to the first page when filters are applied or changed, while preserving the selected sort order.
- **FR-018**: The system MUST provide a clear empty result state when no school matches the active filters and MUST allow users to clear filters from that state.
- **FR-018a**: The system MUST represent active Schools list filters in the page URL query string so filtered views can survive refreshes and be shared or reopened.
- **FR-019**: The system MUST preserve existing school list pagination, sorting, success envelope, validation error envelope, unauthorized response, forbidden response, and tenant visibility rules.
- **FR-020**: The system MUST define all changed school list contract behavior in OpenAPI before backend or frontend implementation begins.
- **FR-021**: The system MUST identify all affected repositories and delivery sequence when implementation spans more than one repository.
- **FR-022**: The system MUST NOT introduce runtime management of administrative type, legal nature, management type, or pedagogical approach lookup values as part of this feature.

### Key Entities *(include if feature involves data)*

- **School**: Tenant-root school profile shown in the Schools list, including identity, contact, address, institutional classification, branding, and lifecycle status fields.
- **SchoolListFilter**: The set of optional criteria a user applies to narrow the Schools list: status, INEP code, document/CNPJ, name, email, city, state, administrative type, legal nature, management type, and pedagogical approach. Status and institutional criteria accept one selected value per field.
- **InstitutionalLookup**: Approved option values for administrative type, legal nature, management type, and pedagogical approach used by school records and list filters.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Authorized administrators can reduce a Schools list of at least 100 records to the intended matching set using any single requested filter in under 10 seconds during representative task testing.
- **SC-002**: 100% of supported filter combinations return only schools that match every active criterion and remain within the requester's authorized visibility.
- **SC-003**: 100% of invalid status or institutional filter submissions produce the standard validation feedback without changing stored school data.
- **SC-004**: At least 90% of representative administrators can locate and clear active filters without assistance.
- **SC-005**: Existing unfiltered Schools list behavior remains unchanged for pagination, sorting, response shape, and authorization in regression testing.
- **SC-006**: Contract review finds zero undocumented Schools list filters before backend and frontend implementation begins.

## Assumptions

- The current school contract from feature `029-school-fields-tabs` is authoritative: CNPJ is represented by the canonical `document` field and legacy `cnpj` payloads are rejected.
- Status uses the existing numeric school status values: `1` for active and `0` for inactive.
- Administrative type, legal nature, management type, and pedagogical approach options are read from the existing institutional lookup/reference sources.
- This feature adds list filtering only; it does not change create, edit, lifecycle, restore, deletion, branding, address, or lookup-management behavior.
- Existing tenant visibility, school administration permissions, pagination, sorting, and response envelopes remain in force.
