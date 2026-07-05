# Data Model: School Fields Tabs

This feature changes the School profile contract and related frontend/backend
view models. It introduces no new tenant root; `School` remains the v1 tenant
root.

## School

**Purpose**: Tenant-root school profile used by administration workflows,
session context, and school-owned records.

**Fields**:

- `id`: UUID public identifier.
- `inepCode`: Required 8-digit official INEP school code.
- `status`: Required numeric lifecycle value; `1` active, `0` inactive.
- `name`: Required school display name.
- `tradeName`: Optional commercial/common name.
- `legalName`: Optional registered legal name.
- `document`: Required on create, immutable on edit, 14-digit Brazilian CNPJ
  payload value with valid check digits.
- `email`: Required valid school contact email.
- `phone`: Optional digits-only contact phone.
- `website`: Optional school website URL.
- `description`: Optional free-text school description.
- `address`: Required `SchoolAddress`.
- `institutionalProfile`: Required `SchoolInstitutionalProfile`.
- `branding`: Required `SchoolBranding` value object with defaults.

**Relationships**:

- Has one required `SchoolAddress`.
- Has one required `SchoolInstitutionalProfile`.
- Has one required `SchoolBranding`.
- References multiple Institutional lookup/reference values.

**Validation and State Rules**:

- `document` replaces the existing `cnpj` contract field everywhere.
- `inepCode`, `document`, and `email` must each be unique across all schools,
  including inactive and soft-deleted records.
- `email` uniqueness is case-insensitive; persistence or validation must
  normalize or compare email values in lowercase for uniqueness checks.
- Runtime payloads using legacy `cnpj` are rejected; no alias is supported.
- `document` must be accepted only during creation, must pass 14-digit CNPJ
  check-digit validation, and must not change during update.
- `status` uses only `1` and `0`; old `active`/`inactive` strings are invalid.
- Create and edit saves fail when required Address or Institutional fields are
  incomplete.
- School lifecycle transitions, soft delete, activation, deactivation, restore,
  and tenant-root behavior remain unchanged.
- Simultaneous edits use existing school update behavior; this feature does not
  add an optimistic-locking state transition or version token.
- Existing school-owned records do not require migration or backfill for this
  feature; any existing school-owned records may be deleted or reset only
  during pre-rollout setup or seed cleanup before the new contract is used.
  Normal runtime create/edit behavior must not delete unrelated school-owned
  records.

## SchoolAddress

**Purpose**: Required physical/address profile for a school.

**Fields**:

- `street`: Required string.
- `number`: Required digits-only string.
- `complement`: Optional string.
- `neighborhood`: Required string.
- `city`: Required string.
- `state`: Required string.
- `zipCode`: Required digits-only string.
- `country`: Optional string.

**Rules**:

- Address object is required on create and edit.
- Existing address field names and numeric-only rules remain the source of
  truth.
- Existing records without complete address cannot be saved through edit until
  required address fields are complete.

## SchoolInstitutionalProfile

**Purpose**: Required institutional classification and operating defaults for
a school.

**Fields**:

- `administrativeTypeId`: Required lookup/reference ID.
- `legalNatureId`: Required lookup/reference ID.
- `managementTypeId`: Required lookup/reference ID.
- `pedagogicalApproachId`: Required lookup/reference ID.
- `educationLevelIds`: Required non-empty array of lookup/reference IDs.
- `modalityIds`: Required non-empty array of lookup/reference IDs.
- `timezone`: Optional string; defaults to `America/Sao_Paulo`.
- `language`: Optional string; defaults to `pt-BR`.

**Relationships**:

- `administrativeTypeId` references `InstitutionalLookupOption` group
  `administrative_type`.
- `legalNatureId` references `InstitutionalLookupOption` group `legal_nature`.
- `managementTypeId` references `InstitutionalLookupOption` group
  `management_type`.
- `pedagogicalApproachId` references `InstitutionalLookupOption` group
  `pedagogical_approach`.
- `educationLevelIds` references one or more `education_level` options.
- `modalityIds` references one or more `modality` options.

**Rules**:

- Lookup/reference options come from documented API operations.
- Frontend and backend consumers must not hardcode labels from
  `docs/school_new_fields.md`.
- Lookup/reference operations return full option lists for each group.

## InstitutionalLookupOption

**Purpose**: Contract-published option used by the Institutional tab.

**Fields**:

- `id`: Stable numeric integer option identifier from `docs/school_new_fields.md`.
- `group`: One of `administrative_type`, `legal_nature`, `management_type`,
  `pedagogical_approach`, `education_level`, `modality`.
- `label`: Human-readable option label.
- `status`: Optional numeric active/inactive indicator if the lookup supports
  inactive options.
- `sortOrder`: Optional display order.

**Initial Option Sets**:

- Administrative type: Public, Private.
- Legal nature: For Profit, Non Profit.
- Management type: Traditional, Confessional, Community, Philanthropic,
  Military, Corporate.
- Pedagogical approach: Traditional, Constructivist, Montessori, Waldorf,
  Bilingual.
- Education levels: Early Childhood Education, Elementary School, High School,
  Technical Education, Higher Education.
- Modalities: On-site, Distance Learning, Hybrid.

**Rules**:

- Options are read-only numeric reference data for this feature.
- Options are seeded during backend setup from `docs/school_new_fields.md`.
- Runtime management UI for lookup options is outside this feature.
- Missing or unavailable lookup data renders a recoverable Institutional tab
  state and blocks submit only when required selections cannot be made.

## SchoolBranding

**Purpose**: Visual school identity values shown by administration and later
school-facing surfaces.

**Fields**:

- `logoPath`: Nullable returned path/reference to the stored school logo.
- `logoFile`: Frontend-only selected file before create/update submission.
- `primaryColor`: Optional `#RRGGBB` hex color value, default `#1D4ED8`.
- `secondaryColor`: Optional `#RRGGBB` hex color value, default `#F59E0B`.

**Rules**:

- No selected logo file on create means no logo.
- No selected logo file on edit preserves the existing logo.
- Selected logo file replaces the existing logo after successful update.
- Old logo file is deleted only after the new logo is successfully stored and
  referenced by the school.
- Logo files must be PNG, JPEG, or WebP and no larger than 2 MB.
- SVG, executable content, invalid images, and oversized files are rejected
  with documented validation errors.
- Supplied color values must be exactly 7 characters in `#RRGGBB` format.
- Frontend color editing uses Element Plus `ElColorPicker` without alpha
  channel support for `primaryColor` and `secondaryColor`.
- User-facing logo delete/removal action is outside this feature.

## SchoolFormState

**Purpose**: Frontend route-local model for the create/edit tabbed form.

**Fields**:

- `mode`: `create` or `edit`.
- `initialValues`: Loaded or empty normalized School form values.
- `values`: Current Basic, Address, Institutional, and Branding values.
- `activeTab`: Basic, Address, Institutional, or Branding.
- `fieldErrors`: Contract field path to localized messages.
- `tabErrors`: Derived map of tabs with validation errors.
- `logoFile`: Selected file object, if any.
- `lookupState`: Institutional lookup/reference loading and selected options.
- `status`: Idle, loading, ready, submitting, succeeded, validation-error,
  forbidden, tenant-mismatch, not-found, unavailable, or error.
- `isDirty`: Derived comparison with `initialValues` and selected logo file.
- `requestKey`: Current route, mode, school identifier, and session generation.

**Rules**:

- Values persist across tab changes and validation failures.
- Validation errors on inactive tabs are discoverable from tab labels or
  summary.
- Duplicate submit is blocked while submitting.
- Route or school-context changes confirm discard when dirty.
- Stale load/save/lookup responses cannot overwrite current form state.

## SchoolFeedback

**Purpose**: Normalized user-facing and diagnostic feedback for school create,
edit, lookup, and upload flows.

**Fields**:

- `type`: Validation, unauthorized, forbidden, tenant-mismatch, not-found,
  inactive-context, conflict, upload-invalid, unavailable, stale-response, or
  unknown.
- `messageKey`: Centralized display key.
- `fieldErrors`: Optional field-level errors.
- `tabErrors`: Optional affected tabs.
- `operationId`: Safe operation identifier.
- `requestId`: Optional safe correlation identifier.

**Rules**:

- Feedback must not disclose cross-tenant school existence or private file
  storage paths.
- Validation feedback preserves all entered values and selected options.
- Logo upload errors are shown on the Branding tab and do not clear other form
  values.
