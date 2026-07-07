# Quickstart: School Fields Tabs

## Purpose

Use this checklist to validate the contract-first delivery for School create
and edit tabs across `schoolmaster-specs`, `schoolmaster-backend`, and
`schoolmaster-frontend`.

## 1. Specification Repository

1. Confirm the active feature is `specs/029-school-fields-tabs`.
2. Update OpenAPI before implementation:
   - School create/update/read schemas replace `cnpj` with `document`.
   - Runtime payloads using legacy `cnpj` are rejected; no alias is supported.
   - `document` is a 14-digit CNPJ with valid check digits.
   - `inep_code`, `document`, and `email` are unique across all schools,
     including inactive and soft-deleted records.
   - `email` uniqueness is case-insensitive.
   - School status uses numeric `1` and `0`.
   - `address` is required on create and update.
   - Basic, Address, Institutional, and Branding fields are documented.
   - Create/update support JSON without logo and multipart with `logo_file`.
   - Logo validation documents PNG/JPEG/WebP and 2 MB maximum.
   - Branding colors validate as `#RRGGBB` hex values.
   - Institutional lookup/reference operations are published.
   - Institutional lookup/reference option IDs are numeric integers.
   - Institutional lookup/reference operations return full option lists, not
     paginated envelopes.
   - Institutional lookup/reference options are seeded during backend setup.
   - Existing school-owned data migration/backfill is not required.
3. Run contract validation:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Expected result: OpenAPI passes with no undocumented School field, status,
lookup, or logo upload behavior.

## 2. Backend Repository

Implement backend after OpenAPI is updated.

Validation targets:

- Create accepts valid Basic, Address, Institutional, and Branding fields.
- Create rejects missing required Basic, Address, or Institutional fields.
- Create accepts no logo and stores no logo reference.
- Create accepts valid PNG/JPEG/WebP logo under 2 MB and returns logo path or
  reference.
- Create rejects invalid, oversized, SVG, or executable logo content.
- Create and edit reject invalid primary or secondary color values.
- Create rejects document values that are not valid 14-digit CNPJs.
- Create and edit reject duplicate INEP code, document, or email values.
- Duplicate checks include inactive and soft-deleted schools.
- Email duplicate checks reject values that differ only by letter casing.
- Edit displays/preserves document and rejects changed document.
- Edit rejects missing required address fields, even when the user changes only
  unrelated fields.
- Edit preserves existing logo when no new file is supplied.
- Edit replaces existing logo when a valid new file is supplied.
- Successful logo replacement deletes the old stored logo file after the new
  logo is stored and referenced.
- Status accepts only `1` and `0`.
- Institutional lookup/reference operations return documented full option
  lists.
- Institutional field payloads submit numeric integer IDs.
- No runtime UI exists for managing Institutional lookup options.
- Authorization, tenant mismatch, forbidden, validation, not-found, conflict,
  and temporary-unavailable behavior use documented response shapes.
- Simultaneous edits follow existing update behavior; no new optimistic-locking
  conflict requirement is added.

Suggested validation:

```bash
php artisan test --filter=School
```

## 3. Frontend Repository

Implement frontend after backend contract behavior is available or mocked from
the approved OpenAPI shape.

Manual checks:

- Open school create as an authorized administrator.
- Verify tabs: Basic, Address, Institutional, Branding.
- Submit empty form; required errors identify field and tab.
- Fill Basic with 8-digit INEP code, numeric status, document, email, and name.
- Fill required Address fields and verify numeric-only number/zip code.
- Load Institutional options from lookup/reference operations.
- Select required single and multi-value Institutional fields.
- Leave Branding colors blank; verify documented defaults after success.
- Verify Branding color fields use Element Plus `ElColorPicker` controls and
  submit non-alpha `#RRGGBB` values.
- Select valid logo file; verify multipart path and success logo reference.
- Select invalid logo file; verify Branding tab error and preserved values.
- Edit existing school; verify document is read-only.
- Try manipulated changed document; backend rejects and UI preserves values.
- Edit existing incomplete address; save remains blocked until address complete.
- If existing school-owned records conflict with the new contract, delete/reset
  them during pre-rollout setup or seed cleanup instead of adding
  migration/backfill behavior.
- Verify normal runtime create/edit requests do not delete unrelated
  school-owned records.
- Change route or school context while dirty; discard confirmation appears.
- Simulate stale lookup/save responses; current form state is not overwritten.
- Verify simultaneous edit behavior matches existing school update semantics.

Suggested validation:

```bash
npm run test:unit -- school
```

## 4. Cross-Repository Acceptance

Before merge, verify:

- `api/openapi.yaml` and component files define all behavior used by backend
  and frontend.
- Backend request/resource names match OpenAPI field names.
- Frontend services submit only documented fields.
- No code path still submits or expects `cnpj`.
- Legacy `cnpj` payloads are rejected instead of mapped to `document`.
- No code path submits `active` or `inactive` for school status.
- No school create/edit path allows omitted address.
- No school create/edit path allows duplicate INEP code, document, or email
  values from inactive or soft-deleted schools.
- No school create/edit path allows duplicate emails that differ only by
  casing.
- No frontend hardcoded Institutional labels replace lookup/reference calls.
- Logo upload failure never clears other form values.
- School lifecycle, tenant selection, role/user/academic/guardian behavior, and
  soft delete behavior remain unchanged.

## 5. Release Gates

- Merge OpenAPI, backend, and frontend changes as one coordinated breaking
  `/api/v1` school contract release.
- Block merge unless Redocly lint, backend Docker school-filtered PHPUnit,
  frontend school Vitest, and frontend production build pass.
- Do not deploy backend create/update contract changes without the matching
  frontend `src/modules/schools` routes and service mapping.
- Seed Institutional lookup options before administrators use create/edit.
- Existing pre-feature school-owned data may be reset during pre-rollout setup
  only; normal runtime create/edit must not delete unrelated records.

## 6. Evidence to Capture

- OpenAPI lint output.
- PHPUnit School feature/unit test output.
- Vitest school service/composable/component test output.
- Manual screenshot or notes for create, edit, validation, lookup, and logo
  upload flows at desktop and mobile widths.

## Evidence

- 2026-07-06 OpenAPI lint: `npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1`
  passed for `api/openapi.yaml`; Redocly reported four pre-existing
  unused-component warnings in `specs/001-schoolmaster-platform/contracts/openapi.yaml`
  for `ReportRunId`, `ReportRun`, `ReportRequest`, and `OutputExpired`.
- 2026-07-06 OpenAPI lint rerun: `npx @redocly/cli lint aggregate@v1
  schoolmaster-platform@v1` passed for `api/openapi.yaml`; the same four
  pre-existing unused-component warnings remained in
  `specs/001-schoolmaster-platform/contracts/openapi.yaml`.
- 2026-07-06 Backend new school tests:
  `docker exec schoolmaster-backend-app-1 php artisan test tests/Feature/School
  tests/Unit/School/SchoolLogoServiceTest.php` passed with 19 tests and 140
  assertions.
- 2026-07-06 Backend Docker school-filtered tests:
  `docker exec schoolmaster-backend-app-1 php artisan test --filter=School`
  passed with 127 tests and 612 assertions.
- 2026-07-06 Frontend school tests: `npm run test:unit -- school` passed with
  18 test files and 30 tests.
- 2026-07-06 Frontend school UI tests: `npm run test:unit -- school` passed
  with 20 test files and 38 tests after adding tab/composable coverage for
  create, edit, inactive-tab errors, logo validation, stale load, and stale
  lookup responses.
- 2026-07-06 Frontend production build: `npm run build` passed. Vite reported
  existing third-party pure-annotation and large-chunk warnings.
- 2026-07-06 Backend Docker school-filtered retest after cleanup:
  `docker exec schoolmaster-backend-app-1 php artisan test --filter=School`
  passed with 127 tests and 612 assertions.
- 2026-07-06 Contract cleanup checks:
  `rg "^\s*cnpj\s*:" api/paths/schools api/components/schemas/schools
  api/openapi.yaml` returned no schema/path property matches; backend active
  API request code only rejects `cnpj` through the v1 prohibited rule while the
  internal legacy DB column remains for compatibility; frontend
  `src/modules/schools` has no `cnpj` submit path and only imports the CNPJ
  validator for `document` check-digit validation.

## Manual Notes

- Desktop review: `SchoolCreatePage.vue` and `SchoolEditPage.vue` compose one
  full-width admin form with Basic, Address, Institutional, and Branding tabs;
  field groups use two-column grids above `sm` and single-column layout below
  `sm`.
- Mobile review: all fixed-format controls are constrained by Element Plus
  inputs/selects and Tailwind grid tracks; no button or label text requires
  horizontal scrolling in the module components.
- Validation review: `useSchoolForm` routes first invalid field to its owning
  tab and `AdminFormPage` exposes the validation summary; tab labels show an
  error marker for inactive-tab discoverability.
- Lookup review: Institutional labels are loaded only through
  `/api/v1/school-lookups/*`; unavailable lookup state disables selectors and
  blocks submit without clearing entered values.
- Logo review: Branding tab keeps current/no-logo state visible, validates
  PNG/JPEG/WebP and 2 MB client-side, uses multipart only when `logo_file` is
  present, and does not clear Basic, Address, or Institutional values after
  validation failure.
- Security review: backend create/update remain under authenticated v1 school
  routes and policies; school detail/update responses omit `cnpj`; logo storage
  uses backend MIME/size validation and UUID filenames; error mappers expose
  safe normalized feedback without raw payloads or private file metadata.
- Timing note: representative dry run over the implemented form fields found
  all required Basic, Address, and Institutional controls visible in three
  tabs, with no dependent dynamic form construction; expected completion is
  under 5 minutes for an administrator with school identity data available.
- Pending UAT: moderated field-location check with multiple administrators is
  still required to claim the 90% participant success target.
