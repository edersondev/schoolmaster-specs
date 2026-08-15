# Academic Year List Filters Contract

## Operation

`GET /api/v1/academic-years` (`listAcademicYears`)

Existing authentication, `X-School-Id` tenant context, permission checks,
pagination, response envelopes, ordering, and lifecycle visibility remain in
force.

## Optional Query Parameters

| Parameter | Type | Rules | Matching |
|---|---|---|---|
| `name` | string | Trimmed, non-blank, max 255 | Case-insensitive contains |
| `date_from` | date | Required with `date_to`, `YYYY-MM-DD` | Inclusive range start |
| `date_to` | date | Required with `date_from`, `YYYY-MM-DD`, not before `date_from` | Inclusive range end |
| `status` | string | planned, active, closed, inactive | Exact |
| `page` | integer | Existing positive-page rule | Pagination |
| `per_page` | integer | Existing positive maximum-100 rule | Pagination size |

All populated criteria combine with AND semantics. Date matching uses inclusive
overlap:

```text
academic_year.start_date <= date_to
AND academic_year.end_date >= date_from
```

## Success Contract

- Existing paginated success envelope.
- Existing Academic Year resource fields.
- Existing descending start-date ordering.
- No response-field additions.
- No matches return HTTP 200 with an empty `data` collection and valid metadata.

## Validation Contract

Malformed dates, incomplete ranges, reversed ranges, unsupported statuses,
oversized names, invalid pagination, array-shaped scalar parameters, and unknown
query fields return the existing HTTP 422 validation envelope with field errors
keyed by documented parameter name.

## Authorization and Tenancy Contract

- Existing authenticated access and `academic_years.view` permission apply.
- A school context is required exactly as before.
- Filters reduce only the resolved school's collection.
- Cross-school records never match, even when all supplied values match.
- Existing system-administrator master access still requires explicit tenant
  resolution and retains existing audit behavior.

## Frontend Interaction Contract

- Academic Years shows only Name, Date range, and Status search criteria.
- Date range is one paired control producing both boundaries.
- Status offers planned, active, closed, and inactive.
- Search runs only on explicit submit.
- Valid active filters are represented as `name`, `date_from`, `date_to`, and
  `status` in the route query.
- Submit or reset returns to page one.
- Reload, pagination, and page-size changes retain valid active filters.
- Reset clears every academic-year criterion.
- School-only INEP, CNPJ, email, city, state, and institutional fields never
  appear and are never sent by this page.

## Verification Contract

Specification contract lint verifies both aggregate and active platform files.
Backend PHPUnit verifies independent and combined matches, inclusive boundary
overlap, complete-range validation, statuses, empty results, pagination, unknown
fields, and tenant isolation. Frontend Vitest verifies component events, status
options, query parse/serialize/update, service parameter names, URL persistence,
combined submission, and reset.
