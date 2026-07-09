# Quickstart: School List Filters

## Purpose

Use this checklist to validate contract-first delivery for Schools list
filtering across `schoolmaster-specs`, `schoolmaster-backend`, and
`schoolmaster-frontend`.

## 1. Specification Repository

1. Confirm active feature is `specs/030-school-list-filters`.
2. Update OpenAPI before implementation:
   - `GET /api/v1/schools` documents query filters for `status`,
     `inep_code`, `document`, `name`, `email`, `city`, `state`,
     `administrative_type_id`, `legal_nature_id`, `management_type_id`, and
     `pedagogical_approach_id`.
   - `document` is the canonical CNPJ filter query parameter; no `cnpj` query
     alias is introduced.
   - Status and institutional filters accept one value each.
   - Status and institutional filters validate unsupported values.
   - INEP and document filters match normalized digits exactly.
   - Name, email, city, and state filters use case-insensitive and
     accent-insensitive contains matching.
   - Multiple active filters combine with AND semantics.
   - Filtered responses preserve the existing paginated Schools response
     shape.
   - Existing authorization, tenant visibility, pagination, sorting, and error
     envelope behavior remains documented.
3. Run contract validation:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Expected result: OpenAPI passes with all Schools list filters documented and no
undocumented query behavior.

## 2. Backend Repository

Implement backend after OpenAPI is updated.

Validation targets:

- List without filters preserves existing behavior.
- `status=1` and `status=0` return only matching schools.
- `inep_code` filters exact normalized digits and does not match partial
  values.
- `document` filters exact normalized CNPJ digits and does not accept/require a
  `cnpj` query alias.
- `name`, `email`, `city`, and `state` filters are contains searches.
- Text filters ignore casing and accent differences.
- `administrative_type_id`, `legal_nature_id`, `management_type_id`, and
  `pedagogical_approach_id` accept one approved value each.
- Multiple filters combine with AND semantics.
- Invalid status, invalid lookup IDs, and malformed single-value categorical
  filters return standard validation errors.
- Filtered responses keep the existing paginated envelope and School resource
  shape.
- Authorization and tenant visibility are preserved for broad and exact
  filters.
- Soft-delete and lifecycle visibility are unchanged.

Suggested validation:

```bash
php artisan test --filter=School
```

## 3. Frontend Repository

Implement frontend after backend contract behavior is available or mocked from
the approved OpenAPI shape.

Manual checks:

- Open Schools list as an authorized administrator.
- Verify filter controls for status, INEP code, CNPJ, email, city, state,
  administrative type, legal nature, management type, and pedagogical approach.
- Apply each filter individually and confirm results narrow correctly.
- Apply several filters together and confirm AND semantics.
- Verify status and institutional controls allow one selected value each.
- Verify CNPJ filter serializes as `document`, not `cnpj`.
- Verify active filters appear in the page URL query string.
- Refresh a filtered URL and confirm controls and results restore from the
  query string.
- Share/open a filtered URL and confirm the same valid filters initialize.
- Change filters while on a later page and confirm page resets to 1.
- Change filters while sorted and confirm sort remains active.
- Trigger no-result filters and confirm empty state preserves filters and
  allows clearing.
- Clear one filter and confirm only that query value is removed.
- Clear all filters and confirm unfiltered list behavior returns.
- Simulate invalid bookmarked status or lookup values and confirm validation
  feedback without data mutation.

Suggested validation:

```bash
npm run test:unit -- school
```

## 4. Cross-Repository Acceptance

Before merge, verify:

- OpenAPI, backend request validation, and frontend service parameter names
  match exactly.
- No code path submits `cnpj` for the list filter contract.
- Frontend CNPJ label maps to `document`.
- Backend filters apply after authorization and tenant visibility constraints.
- Text matching behavior is documented and covered for casing and accent
  differences.
- The paginated Schools response shape remains unchanged.
- Existing create/edit, lifecycle, restore, delete, branding, address, and
  lookup-management behavior remains unchanged.

## 5. Release Gates

- Merge OpenAPI changes before backend and frontend implementation.
- Block backend merge unless OpenAPI lint and School list PHPUnit coverage
  pass.
- Block frontend merge unless school list service/composable/component Vitest
  coverage and production build pass.
- Do not deploy frontend filter controls that submit undocumented parameters.
- Do not deploy backend query parameters that are missing from OpenAPI.

## 6. Evidence to Capture

- OpenAPI lint output.
- PHPUnit School list filter test output.
- Vitest school list service/composable/component test output.
- Manual notes for URL query persistence, no-results state, clear behavior,
  and tenant/authorization checks.
