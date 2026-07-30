# Quickstart: School Context Selection UI

## Preconditions

- `schoolmaster-specs`, `schoolmaster-backend`, and `schoolmaster-frontend` are
  available as sibling checkouts on matching feature branches.
- Backend and frontend dependencies are already installed.
- A test System Administrator has an active platform role named exactly
  `System Administrator` and no user-owned school.
- Test data contains active, inactive, and duplicate-name schools with INEP,
  city, and state; use at least 100 active schools for pagination/usability
  evidence.

## Delivery Order

1. In `schoolmaster-specs`, require the existing nullable
   `AuthSession.resolved_school` property and lint the aggregate OpenAPI.
2. In `schoolmaster-backend`, retain resolved `TenantContext` for `/auth/me`,
   serialize its school explicitly, and add current-user feature regressions.
3. In `schoolmaster-frontend`, normalize School data, add the constrained
   selection service/composable/UI, then update store persistence, routing,
   shell entry, authoritative response invalidation, and lifecycle invalidation.
4. Add focused frontend unit/browser coverage and run full regressions/build.

## Backend Checklist

1. Change `AuthService::currentUser()` so its resolved `TenantContext` is not
   discarded.
2. Keep `AuthController` thin while passing the resolved school explicitly to
   `AuthSessionResource`.
3. Preserve login serialization for ordinary school-bound users.
4. Test System Administrator with:
   - no header -> `resolved_school: null`;
   - active school header -> exact School with numeric active status;
   - inactive or unknown header -> `403 tenant_mismatch`.
5. Retain existing school-bound user mismatch and token rejection behavior.

Run from `schoolmaster-backend`:

```bash
php artisan test --filter=CurrentUserApiTest
php artisan test
```

## Frontend Checklist

1. Normalize auth/list School status and selector identity fields.
2. Delegate active-only list queries through the existing schools service with
   separate name/INEP fields and explicit `per_page=25`.
3. Add stale-safe search, pagination, refresh, loading, empty, error, and retry
   behavior without storing choices in the session store.
4. Confirm exact active context through `/auth/me` before committing it.
5. Clear previous tenant state before confirmation and reject stale switch
   responses; retain identity for recoverable selection failures.
6. Clear preference on logout, expiry, token rejection, lifecycle cleanup, or
   identity change.
7. Capture missing-context/current route intent, declare safe generic routes,
   and default unsafe routes to the school dashboard.
8. Add persistent current-school presentation as read-only in US1. Enable its
   navigation to the dedicated selector only after US2 reset ordering,
   stale-response rejection, and unsaved-work/lifecycle guards are active.
9. Integrate lifecycle outcomes: refresh after activation; invalidate context
   after current-school deactivation/deletion.
10. Add one injected observer to the shared authenticated administration client.
    Normalize only canonical `response.data.error.code` or documented legacy
    `response.data.code` to a lowercase code or `null`. Invalidate only for
    normalized `tenant_mismatch`/`inactive_school` when failed request header
    and stamped context generation exactly match current school; pass original
    error onward.
11. On invalid persisted-school bootstrap, clear preference/context and retry
    `/auth/me` once without the header. Never retry for generic forbidden,
    token rejection, or 5xx; never poll or retry in a loop.
12. Keep all strings in i18n and all HTTP access in service modules.

Run from `schoolmaster-frontend`:

```bash
npm run test:unit -- --run tests/unit/school-context-selection tests/unit/auth tests/unit/system-admin-master
npm run test:unit -- --run
npm run build
npx playwright test e2e/school-context-selection.spec.js --project=chromium
npx playwright test e2e/school-context-selection.spec.js
```

## OpenAPI Verification

Run from `schoolmaster-specs` after updating `AuthSession.yaml`:

```bash
npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Confirm no new path, field, response envelope, status, or version was added.
The change only guarantees presence of the existing nullable
`resolved_school` key.

## Source Audits

Run from `schoolmaster-frontend`:

```bash
rg "axios" src/pages src/components src/composables src/router
rg "/api/v1/" src/pages src/components src/composables
rg -n "cnpj|document" src/components/auth src/pages/auth
```

Expected results:

- no direct Axios use outside services;
- no hard-coded selector endpoint outside services;
- no CNPJ/document rendering in selection page/components.

## Manual Scenarios

1. Sign in as System Administrator without context, directly request a
   school-owned route, select an active school, and verify the compatible
   requested route resumes only after exact confirmation.
2. Search by school name, then exact INEP. Navigate multiple pages and identify
   duplicate names using name, INEP, city, and state. Verify no first item is
   auto-selected and no CNPJ appears.
3. From a generic school list/workspace, use the shell control to switch School
   A to School B. Verify the route reloads in B and no A data remains.
4. From a detail/edit/action/subject route, switch schools and verify dashboard
   fallback without old identifiers.
5. From a platform-wide school/support route, switch schools and verify the
   route stays platform-wide and unfiltered.
6. With unsaved work or an open lifecycle confirmation, request a switch.
   Cancel once and verify A/data remain; approve once and verify clearing occurs
   only after approval.
7. Force a stale search response and a stale School A confirmation after School
   B. Verify neither replaces the latest state.
8. Test loading, filtered no results, unfiltered no active schools, validation,
   unauthorized, forbidden, inactive, tenant mismatch, expired session,
   conflict, and temporary-unavailable feedback. Verify each state has distinct
   safe presentation and only identity-loss responses clear the session.
9. Activate an inactive school through existing administration lifecycle UI.
   Verify it appears only after deliberate selector refresh and is not selected
   automatically. Deactivate/delete the current school and verify context/data
   clear.
10. Reload with a valid last-confirmed school, then test invalid school, logout,
    expiry, token rejection, and a different identity on the same browser.
11. While School A is current, simulate a matching `tenant_mismatch` and
    `inactive_school` from a school-owned request. Verify context/data/preference
    clear, identity remains, and a school-owned route enters selection without
    polling.
12. Repeat with generic 401/403, headerless platform request, stale School A
    response after School B is current, stale earlier generation of same school,
    `no_active_school`, `inactive_record`, transient failure, and duplicate
    response. Verify none clears valid current context.

## Responsive and Accessibility Evidence

- Run the browser scenario at widths 390px, 768px, and 1440px.
- Verify no document overflow; long duplicate names wrap; search, results,
  feedback, pagination, and shell control remain usable.
- Use Tab/Shift+Tab and Enter/Space; verify visible focus, labeled name/INEP
  fields, informative result accessible names, disabled duplicate submission,
  `aria-busy`, polite progress/results, assertive errors, and reliable retry.
- Complete manual NVDA or VoiceOver review. Record tool/browser and findings.
- Moderate the 100-school selection task and activate-then-select journey using
  the usability protocol below; record timings separately from automation.

## Performance and Usability Protocol

### SC-004 Performance

1. Use production frontend build in Chromium with at least 100 active-school
   fixtures.
2. Apply deterministic 300 ms delays to school-list and current-session
   responses without injecting server failures.
3. Run 100 explicit selection attempts, timing from selection submission to
   exact confirmation or recoverable safe state.
4. Run 100 restoration attempts, timing from authenticated bootstrap request to
   exact confirmation or recoverable safe state.
5. Score flows separately. At least 95 selection attempts and 95 restoration
   attempts must complete within 2 seconds.

### SC-001 and SC-005 Usability

1. Recruit at least 10 intended System Administrator users or approved
   role-representative proxies who did not implement the feature.
2. Provide task goal only; give no feature-specific coaching or UI directions.
3. Use one seeded dataset with at least 100 active schools and one inactive
   activation target.
4. Record completion time, facilitator assistance, success/failure, and whether
   participant correctly explains activation versus selection.
5. Pass SC-001 when at least 9 of 10 participants select intended active school
   within 30 seconds without help.
6. Pass SC-005 when at least 9 of 10 participants correctly distinguish the two
   actions and finish activate-then-select within 2 minutes without help.

## Evidence to Record

- Redocly lint result.
- Focused and full PHPUnit results.
- Focused and full Vitest results.
- Frontend production build.
- Chromium and configured cross-browser Playwright results.
- Source-audit results.
- Evidence that no poll/timer was added and authoritative response handling is
  installed once at the application boundary.
- Manual responsive, keyboard, screen-reader, and usability evidence.
- Separate SC-004 selection/restoration attempt counts and timing percentiles.
- SC-001/SC-005 participant count, role basis, assistance, timings, outcomes,
  and activation-versus-selection answers.
- Confirmation that no database migration, new endpoint, new package, or
  client-side tenant authorization was introduced.

## Implementation Evidence — 2026-07-30

- OpenAPI: Redocly validated `aggregate@v1` and `schoolmaster-platform@v1`.
  Four existing unused reporting-component warnings remain; this feature added
  no lint error or warning.
- Backend formatting: `vendor/bin/pint --dirty --format agent` passed after
  formatting the changed PHP files.
- Backend focused regression: `CurrentUserApiTest` passed 7 tests with 24
  assertions in the application container.
- Backend lifecycle regression: updated `SchoolDetailUpdateTest` to use the
  model's numeric active-status constant; its 3 tests and 9 assertions passed.
- Backend full regression: with the PHPUnit process configured for 512 MB, 476
  tests passed with 2,385 assertions and no feature regression. One unrelated
  baseline failure remains: `StudentProfileTransferDestinationTest` receives
  the existing `403` where it expects `200`.
- Frontend focused regression: school-context, auth, System Administrator, and
  lifecycle suites passed at their checkpoints. The requested-route guard now
  permits a contextless System Administrator to reach the platform School
  administration list from the selector.
- Frontend full regression: 336 files and 657 tests passed.
- Frontend production build: passed. Only existing third-party pure-annotation
  and large-chunk warnings were emitted.
- Frontend targeted Prettier passed. Targeted ESLint reported 0 errors and 4
  Playwright policy warnings for an intentional bounded wait, Chromium-only
  performance skips, and alternating performance-test targets.
- Browser functional coverage: 11 scenarios passed in each of Chromium,
  Firefox, and WebKit. Responsive selector scenarios passed at 390 px, 768 px,
  and 1440 px with 100 active fixtures, duplicate names, pagination, Enter-key
  search, exact confirmation, focus, no CNPJ rendering, and overflow checks.
- US2/US3 browser coverage passed for valid and invalid restoration, safe and
  unsafe route switching, dirty cancellation, stale authoritative responses,
  matching invalidation, platform-route retention, activate-refresh-select,
  activation conflict, and current-school deactivation.
- SC-004 selection: 100 of 100 attempts completed within 2 seconds with
  deterministic 300 ms API delays; p95 was 871 ms. Timing ends only after the
  selector closes and exact confirmed context is visible.
- SC-004 restoration: 100 of 100 attempts completed within 2 seconds with
  deterministic 300 ms API delays; p95 was 1,005 ms. Timing ends only after the
  selector is absent and restored context is visible.
- Source audits found no direct Axios/API use in pages, components, composables,
  or router; no selector CNPJ/document rendering; no poll/timer additions; and
  one observer installation at `main.js` plus its single service definition.
- No database migration, new endpoint, new npm/composer package, or client-side
  tenant authorization was introduced.
- Pending evidence: manual Tab/Shift+Tab/Enter/Space review, NVDA or VoiceOver
  review, and moderated 10-participant SC-001/SC-005 usability work. T067 stays
  open until human evidence is recorded.
