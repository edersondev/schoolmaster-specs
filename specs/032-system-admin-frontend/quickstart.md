# Quickstart: System Administrator Frontend Access

## Preconditions

- Backend feature `031-system-admin-master` is deployed or available locally.
- The existing authenticated-session response contains the active platform role
  collection used by the frontend auth store.
- The frontend checkout is on its matching feature branch and has installed
  dependencies.

## Implementation Checklist

1. Inventory released non-identity-owned protected routes, navigation items,
   and actions, including their feature permissions and school-context needs.
2. Confirm the auth store exposes a resolved active role collection without
   changing the session response contract.
3. Centralize the exact active platform `System Administrator` predicate.
4. Use the predicate only in shared permission evaluation, route guards, and
   navigation/action visibility helpers.
5. Preserve authentication, session, release, school, subject, approval, and
   safety checks around that predicate.
6. Keep student and guardian self-service in their existing actor-owned or
   guardian-link flows; do not add subject selection or impersonation.
7. Clear school-owned state before loading against a changed school context and
   reject stale prior-context responses.

## Focused Verification

Run from `schoolmaster-frontend`:

```bash
npm run test:unit -- --run tests/unit/system-admin-master
npm run build
```

Confirm the focused tests cover:

- System Administrator direct route access and limited-role denial.
- Protected navigation and action visibility.
- Session restoration before master-access evaluation.
- Missing, inactive, and changed school contexts, including stale-data cleanup.
- Existing identity-owned student and guardian self-service restrictions.
- Approval, confirmation, support, file-safety, closed-period, and other
  prerequisite-specific feedback states.

## Manual Scenarios

1. Sign in as a zero-permission user with the active platform `System
   Administrator` role and open each released non-identity-owned protected
   route group directly and from navigation.
2. Select an active school, verify school-owned data is scoped to it, then
   switch schools and verify prior data disappears before new data arrives.
3. Remove the selected school and verify school-owned pages do not load data.
4. Attempt student and guardian self-service without the normal actor-owned
   profile or guardian link and verify those routes remain unavailable.
5. Trigger a representative approval, confirmation, support, file-safety, or
   closed-period block and verify its existing state remains distinct from a
   permission denial.

## Evidence to Record

- Route, navigation, action, context, and feedback test results.
- Production build output.
- Inventory showing all released non-identity-owned protected frontend route
  groups were covered.
- Confirmation that no endpoint, response schema, OpenAPI, or client-side audit
  change was made.
