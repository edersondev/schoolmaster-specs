# Quickstart: Secure User Password Delivery

## Contract and backend gate

Branch check on 2026-08-26: `039-user-password-delivery` is checked out in the
specification, backend, and frontend repositories. The backend contract,
implementation, and PHP verification remain mandatory before any frontend
source change.

From `schoolmaster-specs`:

```bash
rtk npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Document `requestUserPasswordDelivery`, its safe 201 response, tenant context,
and 401/403/404/409/422/429/503 results before backend work. Tenant-context
failures remain `403 tenant_mismatch`; `422` is reserved for request validation.

Contract result on 2026-08-26: both `aggregate@v1` and
`schoolmaster-platform@v1` passed with zero errors. Redocly reported nine
pre-existing unused-component warnings in the platform contract.

From `schoolmaster-backend`:

```bash
rtk docker exec schoolmaster-backend-app-1 php artisan test --compact \
  tests/Feature/AccountLifecycle/UserPasswordDeliveryTest.php
```

Verify scope, tenant-first lookup, eligibility, limit, mail failure, token
suppression before issuance, supersession, safe responses, completion, and
session revocation.

Backend gate result on 2026-08-26:

- `rtk docker exec schoolmaster-backend-app-1 php artisan test
  tests/Feature/AccountLifecycle` passed 69 tests with 375 assertions after the
  final backend changes.
- Focused delivery, mail-failure, and secret-exposure verification passed 18
  tests with 115 assertions after the final audit boundary update.
- `rtk vendor/bin/pint --dirty --format agent` passed.
- `requestUserPasswordDelivery` is present in the Laravel route list, and
  Redocly still passes both configured APIs with zero errors and nine
  pre-existing platform warnings.

Backend gate: **PASS**. Frontend implementation may begin.

## Frontend gate

Only after backend evidence passes, run from `schoolmaster-frontend`:

```bash
rtk npm run test:unit -- tests/unit/account-lifecycle
rtk env CI=1 npm run test:e2e -- e2e/account-lifecycle.spec.js
rtk npm run build
```

Verify safe authorized controls, single-flight/retry/stale-state behavior,
keyboard/focus, and no token/password/private delivery data in DOM, storage,
query, or diagnostics. Record actual results here; accepted mail handoff is not
proof of inbox delivery.

Frontend gate result on 2026-08-26:

- Account-lifecycle Vitest suite passed 32 files and 86 tests.
- Playwright account-lifecycle suite passed all 15 tests across Chromium,
  Firefox, and WebKit, including keyboard focus restoration and route-change
  invalidation.
- The production Vite build passed. Existing dependency annotation and chunk
  size warnings remain non-blocking.

Final exposure review: **PASS**. The OpenAPI response and frontend mapper admit
only `status`, `delivery_channel`, and `delivery_requested_at`. Backend API and
audit assertions reject reusable tokens, URLs, passwords, target email, and
provider diagnostics. Browser assertions confirm those values do not reach the
DOM, local/session storage, URL, or safe diagnostic state. Accepted feedback
states only that email submission was requested; it does not claim inbox
delivery.

Frontend gate: **PASS**. Feature implementation is complete.
