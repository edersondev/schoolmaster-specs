# Quickstart: Academic Year List Filters

## Contract Gate

1. Confirm `GET /api/v1/academic-years` documents `name`, `date_from`,
   `date_to`, and all four status values.
2. Confirm paired date validation and inclusive overlap semantics are explicit.
3. Lint aggregate and active platform OpenAPI contracts.

## Backend Gate

1. Create same-school academic years that fall before, inside, across, and after
   a selected date range, plus a matching record in another school.
2. Verify name contains matching is trimmed and case-insensitive.
3. Verify each lifecycle status filters exactly.
4. Verify a range matches fully contained, enclosing, left-overlap,
   right-overlap, and exact-boundary records but excludes disjoint records.
5. Verify incomplete, reversed, malformed, unknown, and array-shaped parameters
   return field-level validation errors.
6. Verify combined filters use AND semantics and pagination metadata reflects
   the filtered set.
7. Verify the other school's record never appears.

## Frontend Gate

1. Confirm Academic Years shows Name, Date range, and Status only.
2. Confirm status offers planned, active, closed, and inactive.
3. Confirm editing draft criteria makes no request until Search is submitted.
4. Confirm submit sends `name`, `date_from`, `date_to`, and `status`, resets page
   to one, and stores values in the route query.
5. Confirm reload, pagination, and page-size changes retain active criteria.
6. Confirm Reset removes all criteria and loads the unfiltered first page.
7. Confirm empty results and validation failures use existing page feedback.

## Commands

```bash
# schoolmaster-specs
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1

# schoolmaster-backend
php artisan test --compact tests/Feature/Api/V1/AcademicYearManagementTest.php
vendor/bin/pint --dirty --format agent

# schoolmaster-frontend
npm run test:unit -- --run tests/unit/admin-system/administration
npm run build
```

## Manual Acceptance

In a resolved school, open Academic Years, submit each filter independently,
combine all filters, reload the filtered URL, paginate, change page size, reset,
and confirm no school-specific criteria are visible. Repeat with a range that
touches an academic-year boundary to confirm inclusive behavior.

## Validation Evidence

Automated validation completed on 2026-08-15:

- Redocly validated `aggregate@v1` and `schoolmaster-platform@v1`; nine existing
  unused nested assessment-component warnings remain unrelated to this feature.
- Backend `AcademicYearManagementTest`: 7 tests passed with 30 assertions in the
  application container.
- Backend Pint formatted all changed PHP files successfully.
- Frontend focused academic administration suite: 16 tests passed across six
  files.
- Frontend production build completed successfully. Existing dependency pure-
  annotation and large-chunk warnings remain unrelated to this feature.
- Manual browser acceptance completed successfully on 2026-08-15, confirmed by the user.
