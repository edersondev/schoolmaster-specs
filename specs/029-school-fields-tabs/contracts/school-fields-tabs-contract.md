# School Fields Tabs Contract

## Scope

This contract defines the required OpenAPI, backend, and frontend behavior for
the expanded school create/edit profile. It supersedes current school form
field assumptions from the Administration Foundation UI for this feature.

The feature includes:

- School create/edit/read contract expansion.
- `cnpj` to `document` field rename.
- Numeric school status values.
- Required Address group.
- Authenticated CEP lookup for Address prefill.
- Institutional lookup/reference option operations.
- Branding colors and logo upload/replacement behavior.
- Tabbed frontend form grouping and validation routing.

The feature excludes:

- Migration or backfill of existing school-owned data.
- School lifecycle transition changes.
- Logo deletion/removal action.
- Tenant selection changes.
- Role, user, academic, guardian, billing, notification, messaging, or report
  behavior changes.

## Approved/Required Operations

| Operation | Method and path | Use |
|-----------|-----------------|-----|
| `listSchools` | `GET /api/v1/schools` | Existing school list, response updated with documented School shape where returned |
| `createSchool` | `POST /api/v1/schools` | School create with JSON payload or multipart payload when `logo_file` is present |
| `getSchool` | `GET /api/v1/schools/{school}` | School edit load/detail with expanded School shape |
| `updateSchool` | `PATCH /api/v1/schools/{school}` | School edit with JSON payload or multipart payload when `logo_file` is present; `document` immutable |
| `getAddressLookupByZipCode` | `GET /api/v1/address-lookups/{zipCode}` | Authenticated ViaCEP lookup for Address tab prefill; no data persistence |
| `listSchoolAdministrativeTypes` | `GET /api/v1/school-lookups/administrative-types` | Institutional administrative type options |
| `listSchoolLegalNatures` | `GET /api/v1/school-lookups/legal-natures` | Institutional legal nature options |
| `listSchoolManagementTypes` | `GET /api/v1/school-lookups/management-types` | Institutional management type options |
| `listSchoolPedagogicalApproaches` | `GET /api/v1/school-lookups/pedagogical-approaches` | Institutional pedagogical approach options |
| `listSchoolEducationLevels` | `GET /api/v1/school-lookups/education-levels` | Institutional education level options |
| `listSchoolModalities` | `GET /api/v1/school-lookups/modalities` | Institutional modality options |

OpenAPI may choose equivalent lookup path names if operation IDs remain clear,
versioned under `/api/v1`, and grouped consistently.

## School Request Contract

### Basic Fields

| Field | Create | Update | Rules |
|-------|--------|--------|-------|
| `inep_code` | Required | Optional/updateable | Exactly 8 digits; unique across all schools including inactive and soft-deleted records |
| `status` | Required | Optional/updateable | Numeric `1` active, `0` inactive |
| `name` | Required | Optional/updateable | School display name |
| `trade_name` | Optional | Optional/updateable | Commercial/common name |
| `legal_name` | Optional | Optional/updateable | Registered legal name |
| `document` | Required | Read-only | Unique 14-digit Brazilian CNPJ payload with valid check digits across all schools including inactive and soft-deleted records; replaces `cnpj` |
| `email` | Required | Optional/updateable | Valid email; case-insensitive uniqueness across all schools including inactive and soft-deleted records |
| `phone` | Optional | Optional/updateable | Digits-only payload when supplied |
| `website` | Optional | Optional/updateable | Valid URL when supplied |
| `description` | Optional | Optional/updateable | Free text |

Create requests with a non-CNPJ `document` value and update requests that
include a changed `document` value must be rejected with a documented
validation or immutable-field error and must not update the school.
Create or update requests that duplicate another school's `inep_code`,
`document`, or `email`, including inactive or soft-deleted schools, must be
rejected with documented field-level validation errors. Email duplicate checks
must be case-insensitive.

### Address Fields

`address` is required on create and update.

| Field | Required | Rules |
|-------|----------|-------|
| `address.street` | Yes | String |
| `address.number` | Yes | Digits-only string that fits unsigned integer storage |
| `address.complement` | No | String or null, maximum 255 characters |
| `address.neighborhood` | Yes | String |
| `address.city` | Yes | String |
| `address.state` | Yes | String, maximum 4 characters |
| `address.zip_code` | Yes | Digits-only string, maximum 12 characters |
| `address.country` | No | String or null |

### Institutional Fields

| Field | Required | Rules |
|-------|----------|-------|
| `administrative_type_id` | Yes | Must reference approved administrative type |
| `legal_nature_id` | Yes | Must reference approved legal nature |
| `management_type_id` | Yes | Must reference approved management type |
| `pedagogical_approach_id` | Yes | Must reference approved pedagogical approach |
| `education_level_ids` | Yes | Non-empty array of approved education level IDs |
| `modality_ids` | Yes | Non-empty array of approved modality IDs |
| `timezone` | No | Defaults to `America/Sao_Paulo` |
| `language` | No | Defaults to `pt-BR` |

### Address Lookup Contract

`GET /api/v1/address-lookups/{zipCode}` accepts masked or unmasked Brazilian
CEP values, requires authentication but not resolved school context, and
returns normalized ViaCEP data in the standard success envelope. Successful
lookups may be cached for 24 hours. Invalid CEP format returns field-level
validation for `zip_code`; a valid missing CEP returns not found; ViaCEP
timeout, connection failure, invalid upstream response, or upstream server
failure returns `temporary_unavailable`. The operation must not create or
update persisted address records.

### Branding Fields

| Field | Required | Rules |
|-------|----------|-------|
| `logo_file` | No | PNG, JPEG, or WebP image; max 2 MB; create/update multipart only |
| `primary_color` | No | `#RRGGBB` hex value; defaults to `#1D4ED8` |
| `secondary_color` | No | `#RRGGBB` hex value; defaults to `#F59E0B` |

No `logo_file` on create creates the school with no logo. No `logo_file` on
update preserves the current logo. A successful replacement deletes the old
logo file after the new logo is stored and referenced. User-facing logo
deletion is not approved in this feature. Supplied colors must be exactly 7
characters in `#RRGGBB` format.

## Content Types

- `application/json`: Accepted for create/update when no `logo_file` is
  submitted.
- `multipart/form-data`: Required for create/update when `logo_file` is
  submitted. Multipart field names must match the documented request contract,
  using documented nested/array encoding in OpenAPI.

## School Response Contract

School create, detail, update, and list responses must expose documented fields
needed by the frontend:

- `id`
- `inep_code`
- `status`
- `name`
- `trade_name`
- `legal_name`
- `document`
- `email`
- `phone`
- `website`
- `description`
- `address`
- `administrative_type_id`
- `legal_nature_id`
- `management_type_id`
- `pedagogical_approach_id`
- `education_level_ids`
- `modality_ids`
- `timezone`
- `language`
- `logo_path`
- `logo_url`
- `primary_color`
- `secondary_color`

Responses must not include `cnpj` after this contract is adopted.
Runtime create/update payloads that include legacy `cnpj` must be rejected; no
temporary alias is supported.
Existing school-owned records do not require compatibility support; any
existing school-owned records may be deleted or reset during pre-rollout setup
or seed cleanup instead of migrated or backfilled. Normal runtime create/edit
behavior must not delete unrelated school-owned records.

## Institutional Lookup Contract

Lookup/reference responses return full option lists for each group. Responses
are not paginated in this feature. Each option object contains:

| Field | Rules |
|-------|-------|
| `id` | Stable numeric integer option identifier from `docs/school_new_fields.md` |
| `label` | Display label |
| `status` | Optional numeric availability status if supported |
| `sort_order` | Optional display order |

Initial labels come from `docs/school_new_fields.md`:

- Administrative types: Public, Private.
- Legal natures: For Profit, Non Profit.
- Management types: Traditional, Confessional, Community, Philanthropic,
  Military, Corporate.
- Pedagogical approaches: Traditional, Constructivist, Montessori, Waldorf,
  Bilingual.
- Education levels: Early Childhood Education, Elementary School, High School,
  Technical Education, Higher Education.
- Modalities: On-site, Distance Learning, Hybrid.

Frontend must load these options through the documented operations, not
hardcode labels. Backend setup seeds these options from
`docs/school_new_fields.md`; runtime management UI for lookup options is not
approved in this feature. Request payloads submit numeric integer lookup IDs.

## Authorization and Tenancy Contract

- Existing authenticated school administration authorization applies.
- Backend Policies remain authoritative for list, create, detail, update, logo
  upload, and lookup operations.
- School remains the v1 tenant root.
- No new cross-tenant or platform support access path is introduced.
- Platform access never implies unauthorized school-owned access.
- Direct-route access, stale permissions, manipulated payloads, and changed
  school context must be safely denied by backend authorization.
- Simultaneous school edits use existing update behavior; this feature does not
  add a version token, `updated_at` precondition, or optimistic-locking
  conflict requirement.

## Frontend Route and Form Contract

- School create/edit uses one form with Basic, Address, Institutional, and
  Branding tabs.
- Required field errors on inactive tabs are discoverable from the tab labels
  or validation summary.
- Entered values, selected lookup options, and selected logo file remain
  visible after validation errors.
- Edit mode shows `document` as read-only and never submits a changed
  document.
- Existing school records with incomplete address cannot be saved until
  required Address fields are complete, even when the user edits only unrelated
  tabs.
- Implementations do not need to preserve pre-feature school-owned records; if
  those records conflict with the new contract, they may be deleted or reset
  during pre-rollout setup or seed cleanup before users access this workflow.
  Runtime create/edit requests must not delete unrelated school-owned records.
- Institutional lookup unavailable states are recoverable and must not submit
  incomplete required Institutional values.
- Logo upload validation errors appear on Branding and must not clear Basic,
  Address, or Institutional values.
- Branding color controls use Element Plus `ElColorPicker` for `primary_color`
  and `secondary_color`, bound to non-alpha `#RRGGBB` string values.
- Successful logo replacement returns the new logo path or reference and must
  not continue exposing the old logo as current.
- Stale load, lookup, and save responses cannot overwrite current route/form
  state.

## Error and Feedback Contract

Supported normalized outcomes:

- loading
- validation
- upload-invalid
- unauthorized/session expired
- forbidden
- tenant mismatch
- inactive context
- not found
- conflict
- lookup unavailable
- temporary unavailable
- stale response
- unknown error

Validation errors must be keyed to documented field paths such as
`document`, `address.zip_code`, `education_level_ids`, or `logo_file`.
Errors must not expose cross-tenant existence, private storage paths, hidden
file metadata, bearer tokens, or raw payloads.

## Verification Contract

OpenAPI validation must verify:

- no `cnpj` field remains in School create/update/read contracts;
- `document`, numeric `status`, required `address`, Institutional fields,
  Branding colors, and `logo_file` multipart behavior are documented;
- lookup/reference endpoints and schemas are published;
- success and validation error responses use documented field names.

Backend PHPUnit coverage must verify:

- successful create/update with all tabs;
- duplicate `inep_code`, `document`, and `email` rejection, including inactive
  and soft-deleted schools;
- case-insensitive duplicate `email` rejection;
- existing simultaneous-edit/update behavior remains unchanged with no new
  optimistic-locking requirement;
- create rejects missing required Basic, Address, or Institutional fields;
- edit rejects changed `document`;
- status accepts only `1` or `0`;
- address object is required on create/update;
- lookup/reference operations return approved labels;
- logo upload accepts PNG/JPEG/WebP under 2 MB and rejects invalid files;
- no-logo create and no-new-logo edit behavior;
- old-logo cleanup after successful replacement;
- authorization, tenant, not-found, and validation response shapes.

Frontend Vitest coverage must verify:

- tab rendering and field grouping;
- create and edit request mapping, including JSON and multipart paths;
- document read-only behavior on edit;
- numeric status values;
- required Address tab validation;
- Institutional full option-list loading and unavailable state;
- logo file selection, replacement, validation failure, and value preservation;
- Element Plus `ElColorPicker` usage for primary and secondary brand colors;
- validation summary and inactive-tab error discovery;
- stale load/lookup/save response ignore behavior;
- permission/denied/not-found states;
- responsive and keyboard-operable critical controls.
