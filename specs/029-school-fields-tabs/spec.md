# Feature Specification: School Fields Tabs

**Feature Branch**: `029-school-fields-tabs`  
**Created**: 2026-07-05  
**Status**: Draft  
**Input**: User description: "Refactor school create/edit fields into a tabbed form using docs/school_new_fields.md and current school address fields as source of truth. Add Basic, Address, Institutional, and Branding tabs. Basic includes INEP code, status, name, trade name, legal name, document, email, phone, website, and description; document is required on create, digits only, and read-only on edit. Address includes existing school address fields with current validation and payload behavior preserved. Institutional includes administrative type, legal nature, management type, pedagogical approach, education levels, modalities, timezone, and language, with lookup options from the doc and required rules preserved. Branding includes logo path, primary color, and secondary color with documented defaults. Update school specification and OpenAPI contract so frontend/backend validation, payload shape, create/edit behavior, and tab grouping stay synchronized."

## Clarifications

### Session 2026-07-05

- Q: Should the school document field replace the existing `cnpj` API field or remain a UI/spec label over `cnpj`? -> A: Rename API field from `cnpj` to `document` everywhere.
- Q: Should the Address tab be required or remain optional/null? -> A: Address tab is required for every school create and edit.
- Q: Should school status remain the existing string contract or change to numeric values? -> A: Change school status payload to numeric `1` active and `0` inactive.
- Q: How should Institutional field options be sourced? -> A: Expose Institutional options through documented lookup/reference endpoints.
- Q: How should the Branding tab handle school logos? -> A: Branding tab includes new logo file upload behavior.
- Q: What validation rule should the renamed `document` field use? -> A: `document` must be a 14-digit CNPJ with valid check digits.
- Q: What validation rule should Branding colors use? -> A: Colors must be 7-character hex values in `#RRGGBB` format.
- Q: What identifier type should Institutional lookup options use? -> A: Institutional lookup IDs are numeric integers from `docs/school_new_fields.md`.
- Q: How should existing schools with incomplete addresses behave on edit? -> A: Any edit save requires complete Address, even when editing unrelated fields.
- Q: What happens to the old logo file after a successful replacement? -> A: Delete old logo after successful replacement.
- Q: Must this feature preserve or migrate existing school data? -> A: No migration or backfill is required; existing school data may be deleted or reset if needed.
- Q: What is the allowed scope for deleting or resetting existing data? -> A: Any existing school-owned data may be deleted or reset.
- Q: When may existing school-owned data be deleted or reset? -> A: Only during pre-rollout setup or seed cleanup, not normal runtime use.
- Q: Should the new contract temporarily accept legacy `cnpj` as an alias? -> A: Reject `cnpj` immediately; only `document` is accepted.
- Q: How should Institutional lookup options be created? -> A: Seed Institutional lookup options during backend setup.
- Q: Should Institutional lookup endpoints paginate options? -> A: Lookup endpoints return full option lists.
- Q: Which Basic fields must be unique? -> A: `inep_code`, `document`, and `email` must be unique.
- Q: Should unique school identity fields consider inactive or soft-deleted records? -> A: Uniqueness applies across all schools, including inactive and soft-deleted records.
- Q: Should email uniqueness be case-sensitive? -> A: Email uniqueness is case-insensitive; normalize or compare lowercase.
- Q: Should this refactor add optimistic locking for simultaneous school edits? -> A: Use existing update behavior; no new optimistic-locking requirement.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Create school with grouped fields (Priority: P1)

An authorized administrator creates a school using a form grouped into Basic, Address, Institutional, and Branding tabs, with required fields and validation visible in the relevant tab.

**Why this priority**: School creation is the main workflow affected by the new required fields. Administrators must capture the expanded school profile without losing existing address behavior.

**Independent Test**: Can be fully tested by opening school creation, completing every required tab, submitting a valid school, then repeating with missing or invalid values in each tab and verifying the correct tab and field show validation feedback.

**Acceptance Scenarios**:

1. **Given** an authorized administrator opens school creation, **When** the page loads, **Then** the form shows Basic, Address, Institutional, and Branding tabs with the documented fields assigned to the correct tab.
2. **Given** required Basic, Address, or Institutional values are missing or invalid, **When** the administrator submits the form, **Then** submission is blocked and validation feedback identifies the affected field and tab.
3. **Given** all required values are valid and optional Branding colors are blank, **When** the administrator submits the form without a logo file, **Then** the school is created with documented default branding colors and no logo.
4. **Given** all required values are valid and a permitted logo file is selected, **When** the administrator submits the form, **Then** the school is created with the logo stored through documented upload behavior and returned as the school logo path or reference.

---

### User Story 2 - Edit school without changing immutable document (Priority: P2)

An authorized administrator edits an existing school through the same tabbed structure while the school document remains visible but cannot be changed.

**Why this priority**: Edit behavior must preserve the existing school identity and avoid accidental document changes while allowing other profile fields to be maintained.

**Independent Test**: Can be fully tested by opening an existing school for editing, verifying the document field is read-only, updating valid fields across tabs, and confirming the saved school preserves the original document.

**Acceptance Scenarios**:

1. **Given** an administrator opens school edit, **When** the Basic tab renders, **Then** the document value is shown as read-only and cannot be submitted as a changed value.
2. **Given** the administrator changes valid editable fields across Basic, Address, Institutional, and Branding tabs, **When** the edit is saved, **Then** the updated school reflects those fields and the original document remains unchanged.
3. **Given** an existing school has no complete address, **When** the administrator saves any edit, **Then** the save is blocked until the Address tab contains all required address fields.
4. **Given** the administrator replaces the school logo with a permitted logo file, **When** the edit is saved, **Then** the school shows the new logo path or reference and preserves the original document.
5. **Given** validation fails for a field on a non-active tab, **When** the save attempt returns feedback, **Then** the administrator can identify the tab containing the error without losing entered values.

---

### User Story 3 - Keep contracts and existing school behavior aligned (Priority: P3)

Product, frontend, and backend teams use one synchronized school contract that documents the expanded fields, tab grouping, validation rules, payload shape, and create/edit differences.

**Why this priority**: The feature changes shared school data contracts. Implementation must not proceed from undocumented fields, mismatched names, or hidden frontend-only rules.

**Independent Test**: Can be fully tested by reviewing the school specification and OpenAPI contract to confirm all field groups, required rules, defaults, and create/edit behavior are documented before implementation work begins.

**Acceptance Scenarios**:

1. **Given** implementation planning begins, **When** teams review the school contract, **Then** Basic, Address, Institutional, and Branding fields are all documented with required rules, payload behavior, and lookup/reference sources where applicable.
2. **Given** the frontend and backend consume the school contract, **When** they compare create, update, and read shapes, **Then** the field names, required rules, defaults, and read-only document behavior are consistent.

### Edge Cases

- A required field on an inactive tab is invalid during submit.
- The document contains punctuation, is not exactly 14 digits, or fails CNPJ
  check-digit validation.
- The submitted INEP code, document, or email already belongs to another
  school, including an inactive or soft-deleted school.
- The submitted email differs from another school email only by letter casing.
- An edit attempt includes a changed document value.
- Existing address values are absent, incomplete, or contain invalid numeric-only
  fields; any create or edit save remains blocked until required address fields
  are complete, even when the user edits only unrelated tabs.
- Optional fields are blank, including website, phone, description, timezone, language, primary color, secondary color, country, address complement, and logo file.
- A selected logo file is missing, invalid, too large, unsupported, or rejected by documented upload validation.
- A logo replacement succeeds but old-logo cleanup fails; the school must keep
  the new logo response while cleanup is retried or reported through safe
  diagnostics.
- A primary or secondary color is missing its leading `#`, has invalid hex
  characters, or is not exactly 7 characters.
- Lookup/reference option data for Institutional fields is unavailable or stale while the form is open.
- A user changes school context, authorization, route, or session state while the form has unsaved values.
- Two administrators edit the same school at the same time; this feature follows
  existing update behavior and does not add optimistic-locking conflict checks.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: School create and update validation, logo upload validation/storage behavior, school read output, school persistence, school-related request/resource behavior, Institutional lookup/reference operations, and authenticated CEP address lookup must support the expanded school profile fields, documented defaults, read-only document rule on edit, existing address validation, and school tenant-root constraints.
- **Frontend repository impact**: School create and edit pages must present one tabbed form with Basic, Address, Institutional, and Branding groups; load Institutional options from documented lookup/reference operations; may prefill Address fields from the documented CEP lookup operation; use Element Plus `ElColorPicker` controls for Branding color fields; support documented logo file upload behavior; preserve existing list return, validation feedback, unsaved-change, permission, loading, denied, and not-found behavior; and consume only approved contract fields.
- **Specification or contract repository impact**: This feature must update the school specification and OpenAPI school schemas before implementation begins, including create, update, read, validation, defaults, and tab grouping documentation.
- **Delivery ownership and sequencing**: Specification and OpenAPI changes lead. Backend contract support follows. Frontend tabbed create/edit UI follows once contract behavior is approved and available. Because this is a breaking `/api/v1` contract change, backend and frontend updates MUST be released as one coordinated rollout under the shared `029-school-fields-tabs` feature identifier; no frontend or backend deployment may ship independently while still depending on the old `cnpj`, string status, nullable address, or missing Institutional/Branding contract.

### API Contract Impact

- **OpenAPI update required**: Yes. School create, update, read, address-related schemas, CEP address lookup, logo upload payload/response behavior, and Institutional lookup/reference operations must document the expanded school fields and option sources.
- **Versioned endpoints affected**: `/api/v1/schools`, `/api/v1/schools/{school}`, `/api/v1/address-lookups/{zipCode}`, and documented Institutional lookup/reference endpoints for administrative types, legal natures, management types, pedagogical approaches, education levels, and modalities.
- **JSON response impact**: School responses must include the documented Basic, Address, Institutional, and Branding data needed to render create/edit and detail states without relying on undocumented fields. Logo upload responses must return the documented school logo path or reference. Validation errors must remain keyed to documented field names.
- **Authentication/authorization impact**: Existing school administration authentication, permission, and tenant-root rules remain authoritative. This feature does not introduce new roles or bypass backend authorization.
- **Compatibility impact**: The expanded payload and response shape is a breaking contract change for school create/edit consumers, including replacing the existing school `cnpj` field with `document`, changing school status to numeric `1` active and `0` inactive, making address required for create and edit, and adding documented logo file upload behavior. This feature does not require preserving, migrating, or backfilling existing school-owned data; any existing school-owned data may be deleted or reset during pre-rollout setup or seed cleanup if needed, but normal runtime create/edit behavior must not delete unrelated school-owned records. Rollout requires coordinated OpenAPI, backend, and frontend merge/release gates so dependent repositories switch to the new contract together; if coordinated release is not possible, implementation MUST stop and introduce an explicit API versioning plan before merge.

### Data & Tenancy Impact

- **Tenant scoping impact**: School remains the v1 tenant root. School-owned data must continue to respect `school_id` tenant boundaries where applicable, and school administration must not infer tenant access from client-side tab state.
- **Cross-tenant or platform access impact**: No new cross-tenant access path is introduced. Platform or administrative access remains limited to approved school administration behavior.
- **Soft delete impact**: No lifecycle transition behavior changes. Existing inactive, soft-deleted, not-found, and forbidden school states remain preserved.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST present school create and edit forms with four top-level groups: Basic, Address, Institutional, and Branding.
- **FR-002**: The Basic group MUST include `inep_code`, `status`, `name`, `trade_name`, `legal_name`, `document`, `email`, `phone`, `website`, and `description`.
- **FR-003**: The Basic group MUST require `inep_code`, `status`, `name`, `document` on create, and `email`; `inep_code` must be exactly 8 digits and unique across all schools including inactive and soft-deleted records, `status` must be numeric `1` for active or `0` for inactive, `document` must be a unique 14-digit CNPJ with valid check digits across all schools including inactive and soft-deleted records, and `email` must be a valid unique email address with case-insensitive uniqueness across all schools including inactive and soft-deleted records.
- **FR-004**: The system MUST make `document` editable and required during school creation and read-only during school editing.
- **FR-005**: The Address group MUST be required for every school create and edit and MUST include the existing school address fields `street`, `number`, `complement`, `neighborhood`, `city`, `state`, `zip_code`, and `country`.
- **FR-006**: The Address group MUST require `street`, `number`, `neighborhood`, `city`, `state`, and `zip_code`; `number` and `zip_code` are digits only; `number` must fit unsigned integer storage, `state` is limited to 4 characters, `zip_code` is limited to 12 characters, `complement` and `country` remain optional, and `complement` is limited to 255 characters.
- **FR-006a**: The system MUST expose an authenticated CEP lookup operation that accepts a masked or unmasked `zip_code`, calls ViaCEP, returns normalized address fields for form prefill, caches successful lookups, and does not persist address data.
- **FR-007**: The Institutional group MUST include `administrative_type_id`, `legal_nature_id`, `management_type_id`, `pedagogical_approach_id`, `education_level_ids`, `modality_ids`, `timezone`, and `language`.
- **FR-008**: The Institutional group MUST require administrative type, legal nature, management type, pedagogical approach, at least one education level, and at least one modality.
- **FR-009**: The Institutional group MUST use numeric integer IDs from the documented lookup option sets for administrative type, legal nature, management type, pedagogical approach, education levels, and modalities.
- **FR-010**: The system MUST default blank timezone to `America/Sao_Paulo` and blank language to `pt-BR`.
- **FR-011**: The Branding group MUST include logo file upload behavior, backend-managed `logo_path`, browser-ready `logo_url`, `primary_color`, and `secondary_color`.
- **FR-012**: The Branding group MUST create a school with no logo when no logo file is supplied, store a submitted valid logo through documented upload behavior, delete the old logo file after successful replacement, and default blank primary color to `#1D4ED8` and blank secondary color to `#F59E0B`.
- **FR-012a**: The frontend MUST render `primary_color` and `secondary_color` with Element Plus `ElColorPicker` controls configured for non-alpha `#RRGGBB` hex values.
- **FR-013**: The system MUST preserve entered values across tabs when validation fails, authorization changes, or a recoverable submit error occurs.
- **FR-014**: The system MUST identify validation errors by documented field name and make errors on inactive tabs discoverable from the tabbed form.
- **FR-015**: The system MUST define all changed REST contract behavior in OpenAPI before backend or frontend implementation begins.
- **FR-016**: The system MUST preserve consistent JSON responses for success and failure cases.
- **FR-017**: The system MUST preserve tenant isolation and document any intentional cross-tenant access path.
- **FR-018**: The system MUST identify all affected repositories and the delivery sequence when implementation spans more than one repository.
- **FR-018a**: The system MUST define a coordinated rollout plan for this breaking `/api/v1` school contract change, requiring OpenAPI, backend, and frontend changes to merge and release together or else introducing an explicit API versioning plan before implementation can merge.
- **FR-019**: The system MUST rename the existing school `cnpj` contract field to `document` across create, update, read, validation error, and frontend form contracts.
- **FR-019a**: The system MUST reject runtime create and update payloads that use legacy `cnpj`; no temporary alias is allowed.
- **FR-019b**: The system MUST reject school create and update submissions that would duplicate `inep_code`, `document`, or `email` on another school, including inactive and soft-deleted schools.
- **FR-020**: The system MUST reject school create and edit submissions that omit the address object or leave any required address field incomplete.
- **FR-021**: The system MUST change school status create, update, read, validation error, and frontend form contracts from existing string values to numeric `1` active and `0` inactive values.
- **FR-022**: The system MUST expose Institutional option sets through documented lookup/reference endpoints that return full option lists and MUST NOT require frontend or backend consumers to hardcode option labels from the specification document.
- **FR-022a**: The system MUST create Institutional lookup options through backend setup/seed data; this feature MUST NOT add runtime UI for managing those lookup options.
- **FR-023**: The system MUST define logo file upload validation, failure responses, storage result, and returned logo reference in the school contract before implementation begins.
- **FR-024**: The system MUST preserve existing school update conflict behavior and MUST NOT add a new optimistic-locking requirement for this tabbed-form refactor.

### Key Entities *(include if feature involves data)*

- **School**: Tenant-root school profile with identity, contact, institutional, address, branding, and lifecycle status fields used by administration workflows.
- **SchoolAddress**: Existing address value associated with a school, including street, number, complement, neighborhood, city, state, zip code, and country.
- **InstitutionalLookup**: Approved lookup/reference option values for administrative type, legal nature, management type, pedagogical approach, education levels, and modalities, loaded through documented contract operations.
- **SchoolBranding**: Optional visual identity values for logo upload result, logo path or reference, primary color, and secondary color.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: An authorized administrator can complete school creation with all required Basic, Address, and Institutional fields in under 5 minutes during representative task testing.
- **SC-002**: 100% of validation failures in Basic, Address, Institutional, and Branding groups identify the affected field and tab without losing entered values.
- **SC-003**: 100% of school edit attempts preserve the original document value, including attempts to change it through stale form state or manipulated payloads.
- **SC-004**: Contract review finds zero undocumented school create, update, or read fields before backend and frontend implementation begins.
- **SC-005**: At least 90% of representative administrators can locate required school profile fields in the expected tab without assistance.
- **SC-006**: 100% of logo upload success and validation-failure cases use documented contract behavior and return a visible Branding tab outcome without losing other form values.

## Assumptions

- `docs/school_new_fields.md` is the source of truth for new Basic, Institutional, and Branding fields and for the initial Institutional lookup/reference option labels that the contract must expose.
- The current OpenAPI address schemas are the source of truth for Address tab field names and numeric-only address fields; this feature changes address presence from optional to required.
- Existing school administration permission, tenant selection, denied-state, not-found, validation summary, list return, and unsaved-change behaviors remain in force.
- Color values use documented defaults when blank and must be valid
  7-character hex values in `#RRGGBB` format when supplied.
- This feature changes school profile fields and form organization only; it does not change school lifecycle transitions, role permissions, user assignment, academic calendars, or tenant selection.
- This feature is delivered as a coordinated cross-repository release; old-contract frontend/backend deployments are not supported after the new contract is adopted.
- Existing school-owned data preservation is not required for this feature. If
  the implementation cannot safely adapt existing school-owned records, those
  records may be deleted or reset during pre-rollout setup or seed cleanup
  instead of migrated or backfilled. Normal runtime create/edit behavior must
  not delete unrelated school-owned records.
