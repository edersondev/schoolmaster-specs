# System Administrator Master Access Contract

## Scope

This contract defines the required shared authorization behavior for the System
Administrator master user role across released backend operations. Frontend
adoption is deferred to a separate implementation feature.

The feature includes:

- System Administrator satisfies all feature-specific permission checks.
- System Administrator may select any active school for school-scoped work.
- School-owned responses remain scoped to the selected school unless an
  operation is documented as platform-wide.
- Identity-owned self-service operations keep existing actor-owned student and
  active guardian-link authorization; this feature adds no impersonation or
  selected-subject transport.
- Approval workflows and business safety gates remain enforceable.
- System Administrator writes and lifecycle actions require master-access audit
  evidence.
- Non-System Administrator authorization behavior remains unchanged.

The feature excludes:

- Bypassing authentication, account state, lockout, session validity,
  school-state rules, tenant context, identity ownership, guardian-link state, feature release state,
  approval workflows, explicit confirmations, support opt-ins, file safety
  gates, closed-period safety checks, or other business controls.
- Returning unscoped school-owned data from school-scoped operations.
- New resource response fields or success envelope changes.
- Runtime management of roles, permissions, schools, or subjects beyond
  behavior already approved by their own features.

## Shared Authorization Rule

| Actor | Permission check result | Non-permission prerequisites |
|-------|-------------------------|------------------------------|
| System Administrator | Satisfied for every released protected route, action, and operation | Still enforced |
| Any other role | Existing required permission behavior | Still enforced |

For this backend slice, System Administrator means an active platform-scoped
role named exactly `System Administrator`.

Non-permission prerequisites include authentication, active account, unlocked
account, valid session, active school where required, selected school context,
actor-owned student access, active guardian links, feature release state, approval workflow state,
explicit confirmation state, support opt-in state, file safety state,
closed-period safety state, and any other documented business control.

## OpenAPI Contract Requirements

Protected operations must document:

- System Administrator is allowed as the master role for feature-specific
  permission checks.
- School-scoped operations still require selected or resolved school context.
- Identity-owned self-service operations retain existing actor-owned profile or
  active guardian-link rules; no System Administrator subject impersonation is
  defined.
- Platform-wide operations must explicitly say they may return cross-school or
  unscoped platform output.
- Permission-denial responses do not apply to System Administrator when the
  only missing requirement is a feature-specific permission.
- Tenant-context, identity-ownership, guardian-link, account-state, school-state, release-state,
  safety-gate, validation, conflict, and not-found responses remain available
  where applicable.

## Backend Contract

- `User::isSystemAdministrator()` identifies the exact active platform role,
  and `User::hasPermission()` plus `User::hasSchoolPermission()` treat required
  permission codes as satisfied for that actor.
- A global `Gate::before` override must not be used because it would bypass
  policy-level tenant, ownership, lifecycle, approval, and safety gates.
- Policies remain authoritative for protected operations and must preserve
  tenant scope, subject scope, lifecycle state, and business safety checks.
- Requests must continue to validate required input and context identifiers.
- Services must not return school-owned data outside the selected school
  unless the operation is documented as platform-wide.
- Writes and lifecycle actions by System Administrator must record
  `master_access_used: true` through the existing generic or module-specific
  audit pipeline.
- Non-System Administrator actors must continue to be denied when required
  permissions are missing.

## Deferred Frontend Contract

- No frontend repository files are changed by this implementation slice.
- A separate frontend feature must consume the existing authenticated-session
  roles collection and apply the published permission-only override to guards,
  navigation, and actions.
- That follow-up must preserve school-context, identity-ownership, account,
  session, release, approval, and safety states.

## Tenancy and Subject Contract

- System Administrator may select any active school as school context.
- School-owned responses must contain only selected-school data unless the
  operation is documented as platform-wide.
- School context switches must clear stale school-owned data before loading new
  data.
- Existing actor-owned student-profile and active guardian-link rules remain
  authoritative for identity-owned self-service data.
- No request header, session field, or inferred subject selection is introduced.

## Audit Contract

Master-access audit evidence is required for System Administrator:

- Create actions.
- Update actions.
- Delete, restore, activate, deactivate, and other lifecycle actions.
- Imports and state-changing retries or cancellations.
- Approvals, revocations, and other state-changing support or workflow actions.
- Any other operation that changes protected state.

Read-only navigation, list, and detail views do not require new audit evidence
unless the existing operation already requires it.

Audit evidence must identify the actor, action, target, selected school context
where applicable, outcome, timestamp,
and that master access was used.

## Verification Contract

Specification repository:

- OpenAPI lint passes after authorization notes are updated.
- Security and multi-tenant documentation use one consistent master-access
  rule.
- Existing contradictory "no implicit bypass" language is removed or replaced.

Backend repository:

- PHPUnit verifies System Administrator can access every released protected
  platform-scoped, school-scoped, and self-service operation group listed in
  `api/openapi.yaml` without feature-specific permissions.
- PHPUnit verifies selected school context is required and scopes school-owned
  responses.
- PHPUnit verifies actor-owned student-profile and active guardian-link rules
  remain enforced for identity-owned self-service operations.
- PHPUnit verifies approval workflows and safety gates still deny or block
  System Administrator when their non-permission prerequisites are not met.
- PHPUnit verifies master-access audit evidence for writes and lifecycle
  actions.
- PHPUnit verifies non-System Administrator roles without required permissions
  keep documented denial behavior.

Deferred frontend repository:

- No frontend verification is part of this backend implementation run.
- The backend current-session regression test verifies the existing roles
  collection exposes the active platform `System Administrator` role without a
  response schema change.
