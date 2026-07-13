# Quickstart: System Administrator Master Access

## Preconditions

- Feature spec exists at `specs/031-system-admin-master/spec.md`.
- OpenAPI remains source of truth for protected operation authorization notes.
- Backend implementation uses feature identifier `031-system-admin-master`.
- Frontend implementation is deferred and no frontend repository changes are
  part of this run.

## Specification and Contract Validation

1. Review `docs/security.md` and replace contradictory System Administrator
   "no implicit bypass" language with the master-access rule.
2. Review `docs/multi-tenant.md` and document that System Administrator may
   select any active school while school-owned output remains selected-school
   scoped.
3. Review affected existing feature specs and contracts for permission matrices
   or route guard rules that deny System Administrator only because
   feature-specific permissions are missing.
4. Update `api/openapi.yaml` and affected path/component descriptions so
   protected operations document System Administrator master access.
5. Confirm any operation returning cross-school output is documented as
   platform-wide before unscoped output is allowed.
6. Run OpenAPI validation:

```bash
npx @redocly/cli lint api/openapi.yaml
```

Expected result: OpenAPI lint passes and protected operation descriptions use
consistent System Administrator master-access language.

## Backend Validation

1. Add or update centralized authorization behavior so System Administrator
   satisfies feature-specific permission checks.
2. Preserve account/session, tenant-context, actor-ownership, guardian-link, school-state,
   release-state, approval workflow, and safety-gate checks.
3. Verify school-scoped operations require selected active school context and
   return only selected-school data.
4. Verify identity-owned self-service operations retain actor-owned student and
   active guardian-link authorization without impersonation.
5. Record master-access audit evidence for System Administrator writes and
   lifecycle actions.
6. Run backend tests:

```bash
php artisan test
```

Expected result: System Administrator allow cases pass, tenant/subject/safety
gate denials still pass, audit-marker assertions pass, and non-System
Administrator denial behavior remains unchanged.

## Deferred Frontend Validation

1. Make no frontend repository changes in this backend implementation run.
2. Verify the existing authenticated-session roles collection exposes the
   active platform `System Administrator` role without a response schema change.
3. Record route guard, navigation, and action visibility adoption as a separate
   frontend feature.

## Manual Review Scenarios

- Sign in as System Administrator with no extra feature-specific permissions.
- Open a platform-scoped protected route and confirm access.
- Select any active school, open school-owned administration, and confirm only
  selected-school data appears.
- Switch to another active school and confirm stale school-owned data clears
  before the new school's data loads.
- Call student and guardian identity-owned self-service operations as System
  Administrator and confirm existing actor-owned profile or guardian-link rules
  deny access without leaking another subject's data.
- Attempt a workflow blocked by approval, confirmation, support opt-in, file
  scan, or closed-period safety state and confirm the safety gate still blocks
  the action.
- Perform a System Administrator write or lifecycle action and confirm audit
  evidence marks master access.

## Evidence to Capture

- OpenAPI lint output.
- Backend authorization and audit test output.
- Current-session role-context regression output and deferred frontend handoff.
- Short notes showing non-System Administrator denial behavior was not
  broadened.
