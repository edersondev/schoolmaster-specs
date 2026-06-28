# Quickstart: Centralize Addresses

Use this guide to validate the design intent before implementation begins.

## 1. Confirm Source Artifacts

- Read `specs/019-centralize-addresses/spec.md`.
- Read `specs/019-centralize-addresses/plan.md`.
- Read `specs/019-centralize-addresses/data-model.md`.
- Read `specs/019-centralize-addresses/contracts/centralize-addresses-contract.md`.
- Confirm current aggregate OpenAPI still references `address_summary` before
  contract update work starts.

## 2. Contract Update Checklist

Update `schoolmaster-specs` first:

- Replace `address_summary` with structured `address` in school response
  schemas.
- Add address input schema for school create/update.
- Document update semantics:
  - omitted `address` means no change
  - address object means replace
  - explicit `address: null` means remove
- Ensure `number` and `zip_code` are digit-only strings.
- Ensure `country` is optional and not defaulted.
- Ensure `address_summary` is rejected on public create/update requests.
- Validate OpenAPI:

```bash
npx @redocly/cli lint specs/api/openapi.yaml
```

Result: Passed on 2026-06-26. Redocly reported the existing `info.license`
warning only; the OpenAPI description is valid.

## 3. Backend Implementation Checklist

Implement in `schoolmaster-backend` only after OpenAPI is updated:

- Add centralized address persistence with soft delete support.
- Store direct `school_id` tenant ownership alongside
  `$table->morphs('addressable')` for school-owned address rows.
- Enforce one active address per owner with a MySQL-safe generated active marker
  or equivalent database constraint; do not rely on nullable `deleted_at` alone.
- Add address DTO/request validation for school create/update payloads.
- Update school service behavior for create, replace, omitted no-op, and
  explicit null removal.
- Update school resources/session resources to return structured address data.
- Drop legacy `schools.address_summary` during development schema reset.
- Existing development data can be deleted and recreated; no parser, backfill,
  or migration exception workflow is required.
- Keep authorization through existing school management boundaries.
- Preserve tenant-safe denial behavior for cross-school address attempts.

## 4. Backend Verification

Run focused tests inside the backend container:

```bash
docker exec schoolmaster-backend-app-1 php artisan test --compact --filter=SchoolAddress
```

Result: Passed on 2026-06-27 with 10 tests and 54 assertions.

Required coverage:

- School create with valid address.
- School create/update rejects missing required address fields.
- School create/update rejects non-numeric `number` and `zip_code`.
- School create/update rejects `address_summary`.
- School update with omitted `address` leaves current address unchanged.
- School update with address object replaces current address.
- School update with `address: null` removes current address recoverably.
- School list/detail/session responses expose structured address data.
- Cross-school address access or mutation is denied without leaking existence.
- Duplicate active address creation for the same owner is rejected by service
  behavior and database constraint while soft-deleted history remains
  recoverable.
- Fresh migrations remove the legacy summary column and keep structured address
  persistence only.

## 5. Frontend Follow-Up

After backend behavior is contract-compliant:

- Replace school `addressSummary` form model with structured address fields.
- Map frontend validation errors to address field controls.
- Provide explicit remove-address intent that submits `address: null`.
- Remove legacy `addressSummary` service/request mapping.
- Add Vitest coverage for form mapping, validation display, no-op updates, and
  remove-address intent.

## 6. Final Validation

Before handoff or PR merge, record:

- OpenAPI lint command and result.
- Focused backend test command and result.
- Any frontend test command and result when frontend work is included.
- Timed acceptance check for SC-003 confirming an authorized administrator can
  create or update a school address with structured fields in under 2 minutes.
- Feature id: `019-centralize-addresses`.
- Affected operation IDs: `listSchools`, `createSchool`, `getSchool`,
  `updateSchool`, and authenticated session operations that embed `School`.

## 7. Implementation Evidence

- OpenAPI lint: `npx @redocly/cli lint specs/api/openapi.yaml` passed on
  2026-06-26 with the pre-existing `info.license` warning.
- Focused backend address suite:
  `docker exec schoolmaster-backend-app-1 php artisan test --compact --filter=SchoolAddress`
  passed on 2026-06-27 with 10 tests and 54 assertions in 17.58 seconds.
- Affected backend school management suite:
  `docker exec schoolmaster-backend-app-1 php artisan test --compact tests/Feature/SchoolManagementApiTest.php`
  passed on 2026-06-26 with 2 tests and 22 assertions.
- Full backend suite:
  `docker exec schoolmaster-backend-app-1 php artisan test --compact` passed
  on 2026-06-27 with 404 tests and 1842 assertions in 35.62 seconds.
- Frontend address suite:
  `npm run test:unit -- --run tests/unit/admin-system/administration/address`
  passed on 2026-06-26 with 3 files and 5 tests.
- Affected frontend school suites:
  `npm run test:unit -- --run tests/unit/admin-system/administration/contracts/schools.contract.spec.js tests/unit/admin-system/administration/services/schools.spec.js tests/unit/admin-system/administration/components/SchoolsModule.spec.js`
  passed on 2026-06-26 with 3 files and 5 tests.
- Public contract scan:
  `rg "address_summary|addressSummary" specs/api app schoolmaster-frontend/src -n`
  found no OpenAPI or frontend `src` usage. Backend matches are intentional
  validation rejection only.
- SC-003 timed acceptance: automated create/update/remove coverage completes
  inside the focused backend suite in 17.58 seconds, under the 2 minute target.
