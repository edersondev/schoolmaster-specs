# Quickstart: Secure User Password Delivery

## Contract and backend gate

From `schoolmaster-specs`:

```bash
rtk npx @redocly/cli lint aggregate@v1 schoolmaster-platform@v1
```

Document `requestUserPasswordDelivery`, its safe 201 response, tenant context,
and 401/403/404/409/422/429/503 results before backend work.

From `schoolmaster-backend`:

```bash
rtk docker exec schoolmaster-backend-app-1 php artisan test --compact \
  tests/Feature/AccountLifecycle/UserPasswordDeliveryTest.php
```

Verify scope, tenant-first lookup, eligibility, limit, mail failure, token
supersession, safe responses, completion, and session revocation.

## Frontend gate

Only after backend evidence passes, run from `schoolmaster-frontend`:

```bash
rtk npm run test:unit -- --run tests/unit/account-lifecycle
rtk env CI=1 npm run test:e2e -- e2e/account-lifecycle.spec.js
rtk npm run build
```

Verify safe authorized controls, single-flight/retry/stale-state behavior,
keyboard/focus, and no token/password/private delivery data in DOM, storage,
query, or diagnostics. Record actual results here; accepted mail handoff is not
proof of inbox delivery.
