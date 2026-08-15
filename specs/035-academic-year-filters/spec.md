# Feature Specification: Academic Year List Filters

**Feature Branch**: `035-academic-year-filters`  
**Created**: 2026-08-15  
**Status**: Implemented  
**Input**: User description: "Filter academic years by name, date, and status."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Find an academic year (Priority: P1)

A school administrator can narrow the active school's academic-year list by
name, date, or lifecycle status so they can reach the relevant record without
scanning every page.

**Why this priority**: Finding the correct academic year is the direct user need
and supports later view, edit, and lifecycle actions.

**Independent Test**: Seed academic years with distinct names, dates, and
statuses in one school; apply each filter independently and verify that only
matching records appear.

**Acceptance Scenarios**:

1. **Given** several academic years in the active school, **When** the
   administrator searches using part of an academic-year name, **Then** the list
   contains only same-school records whose names contain that text without
   case sensitivity.
2. **Given** academic years in different lifecycle states, **When** the
   administrator selects a status, **Then** the list contains only records in
   that status.
3. **Given** academic years with different date boundaries, **When** the
   administrator applies the documented date filter, **Then** the list contains
   only records matching the selected date semantics.

---

### User Story 2 - Combine and reset filters (Priority: P2)

A school administrator can combine name, date, and status criteria, share or
reload the filtered page, and reset all criteria.

**Why this priority**: Combined criteria make large lists useful and persistent
page state prevents accidental loss of the administrator's search context.

**Independent Test**: Apply all supported criteria, reload the page, navigate
through results, and reset the filters; verify the criteria, results, and page
number remain consistent with each action.

**Acceptance Scenarios**:

1. **Given** multiple filters are populated, **When** the administrator submits
   the search, **Then** only records matching every populated criterion appear.
2. **Given** a filtered result page, **When** the page is reloaded or its address
   is opened again, **Then** the same valid filters are restored and applied.
3. **Given** active filters, **When** the administrator resets them, **Then** all
   criteria are cleared, pagination returns to the first page, and the
   unfiltered same-school list is loaded.

### Edge Cases

- Blank or whitespace-only name input is treated as no name filter.
- Name matching does not expose records belonging to another school.
- A valid search with no matches shows the filtered empty state rather than a
  loading or failure state.
- Unsupported status values and malformed dates are rejected through the
  existing validation-error contract.
- Changing any filter returns pagination to the first page.
- Filters remain safe when an academic year is soft-deleted or restored; list
  visibility continues to follow existing lifecycle rules.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Extend academic-year list validation and
  tenant-scoped filtering, with focused list-filter tests.
- **Frontend repository impact**: Replace the school-filter reuse with a
  dedicated academic-year search form, persist supported criteria in the page
  address, and cover query, service, and component behavior.
- **Specification or contract repository impact**: Add the approved
  academic-year list parameters to the shared contract and synchronize the
  aggregate contract copy.
- **Delivery ownership and sequencing**: Specification and contract lead;
  backend filtering follows; frontend consumes the completed contract last.

### API Contract Impact

- **OpenAPI update required**: Yes, for academic-year list query parameters.
- **Versioned endpoints affected**: `GET /api/v1/academic-years`.
- **JSON response impact**: None; successful pagination and validation-error
  envelopes remain unchanged.
- **Authentication/authorization impact**: None; existing academic-year view
  permission and active-school resolution remain required.
- **Compatibility impact**: Additive optional query parameters; existing
  unfiltered clients remain compatible.

### Data & Tenancy Impact

- **Tenant scoping impact**: Every criterion is applied only after the active
  school is resolved; no filter may widen the tenant scope.
- **Cross-tenant or platform access impact**: Existing documented system-admin
  access still requires an explicitly resolved school context.
- **Soft delete impact**: No change to current list visibility or lifecycle
  behavior.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The academic-year page MUST provide dedicated name, date, and
  status controls and MUST NOT display school-specific criteria.
- **FR-002**: Name filtering MUST perform case-insensitive contains matching
  within the resolved school.
- **FR-003**: Date filtering MUST accept an inclusive range and return academic
  years whose own date spans overlap any part of that range.
- **FR-004**: Status filtering MUST support every lifecycle status approved for
  academic years: planned, active, closed, and inactive.
- **FR-005**: Populated criteria MUST combine using AND semantics; omitted or
  blank criteria MUST not restrict results.
- **FR-006**: Supported filters MUST persist in the page address and survive
  reload, pagination, and page-size changes.
- **FR-007**: Submitting changed criteria or resetting criteria MUST return the
  list to page one.
- **FR-008**: Reset MUST clear name, date, and status criteria together.
- **FR-009**: Invalid status or date input MUST use the existing validation-error
  response and MUST NOT produce partial results.
- **FR-010**: All filtering MUST preserve active-school tenant isolation,
  authorization, pagination, ordering, response envelopes, and lifecycle
  visibility.
- **FR-011**: The changed academic-year list contract MUST be defined in the
  shared OpenAPI source before backend and frontend implementation begins.
- **FR-012**: Automated coverage MUST verify each criterion independently,
  combined criteria, reset behavior, URL persistence, invalid input, empty
  results, and cross-school isolation.
- **FR-013**: A date range MUST contain both boundaries, and its end boundary
  MUST be the same as or later than its start boundary.

### Key Entities

- **Academic Year**: A school-owned calendar record identified by name, start
  date, end date, and lifecycle status.
- **Academic Year Filter Set**: Optional name, date, and status criteria applied
  together to the active school's academic-year collection.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: An administrator can submit any single supported criterion and see
  the matching first page in no more than two interactions.
- **SC-002**: All accepted filter combinations return only matching records from
  the resolved school in acceptance testing.
- **SC-003**: Valid filters are restored correctly in 100% of reload,
  pagination, and shared-address acceptance cases.
- **SC-004**: School-specific fields no longer appear on the academic-year page
  in desktop or mobile acceptance checks.
- **SC-005**: Invalid criteria produce clear validation feedback in 100% of
  malformed-date and unsupported-status acceptance cases.

## Assumptions

- Existing permissions, tenant resolution, pagination sizes, default ordering,
  and lifecycle visibility remain unchanged.
- Name matching is partial and case-insensitive because administrators may know
  only part of a label such as “2026”.
- Date-range boundaries are inclusive; an academic year matches when its start
  is on or before the selected range end and its end is on or after the selected
  range start.
- Filters execute only when the administrator explicitly submits the search;
  editing a draft value does not issue a request.
- Existing responsive layout and internationalization conventions are reused.
- No database schema change or new dependency is required.
