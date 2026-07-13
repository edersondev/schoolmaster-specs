# System Administrator Master Access Contract

## Scope

This contract defines the required shared authorization behavior for the System
Administrator master user role across released frontend routes and backend
operations.

The feature includes:

- System Administrator satisfies all feature-specific permission checks.
- System Administrator may select any active school for school-scoped work.
- School-owned responses remain scoped to the selected school unless an
  operation is documented as platform-wide.
- Released identity-owned self-service routes require selected subject context.
- Approval workflows and business safety gates remain enforceable.
- System Administrator writes and lifecycle actions require master-access audit
  evidence.
- Non-System Administrator authorization behavior remains unchanged.

The feature excludes:

- Bypassing authentication, account state, lockout, session validity,
  school-state rules, tenant context, subject context, feature release state,
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

Non-permission prerequisites include authentication, active account, unlocked
account, valid session, active school where required, selected school context,
selected subject context, feature release state, approval workflow state,
explicit confirmation state, support opt-in state, file safety state,
closed-period safety state, and any other documented business control.

## OpenAPI Contract Requirements

Protected operations must document:

- System Administrator is allowed as the master role for feature-specific
  permission checks.
- School-scoped operations still require selected or resolved school context.
- Identity-owned self-service operations still require selected subject context
  when used through master access.
- Platform-wide operations must explicitly say they may return cross-school or
  unscoped platform output.
- Permission-denial responses do not apply to System Administrator when the
  only missing requirement is a feature-specific permission.
- Tenant-context, subject-context, account-state, school-state, release-state,
  safety-gate, validation, conflict, and not-found responses remain available
  where applicable.

## Backend Contract

- Central authorization behavior must treat System Administrator as satisfying
  required permission codes.
- Policies remain authoritative for protected operations and must preserve
  tenant scope, subject scope, lifecycle state, and business safety checks.
- Requests must continue to validate required input and context identifiers.
- Services must not return school-owned data outside the selected school
  unless the operation is documented as platform-wide.
- Writes and lifecycle actions by System Administrator must record
  master-access audit evidence.
- Non-System Administrator actors must continue to be denied when required
  permissions are missing.

## Frontend Contract

- Route guards must treat System Administrator as satisfying route permission
  metadata.
- Navigation and action visibility must show released protected destinations
  and actions to System Administrator after required session state resolves.
- School-scoped routes must still require selected active school context before
  loading school-owned data.
- Identity-owned self-service routes must still require selected subject
  context before loading student, guardian, user, or other subject data.
- Denied, missing-school-context, missing-subject-context, inactive-account,
  expired-session, feature-unavailable, and safety-gate states must remain
  distinct.
- Components must not duplicate authorization business rules; shared route,
  store, composable, or service boundaries own the decision.

## Tenancy and Subject Contract

- System Administrator may select any active school as school context.
- School-owned responses must contain only selected-school data unless the
  operation is documented as platform-wide.
- School context switches must clear stale school-owned data before loading new
  data.
- Subject context must be selected before identity-owned self-service data is
  loaded through master access.
- Subject context must be compatible with selected school context when the
  subject is school-owned.

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
where applicable, selected subject context where applicable, outcome, timestamp,
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
- PHPUnit verifies selected subject context is required for identity-owned
  self-service operations.
- PHPUnit verifies approval workflows and safety gates still deny or block
  System Administrator when their non-permission prerequisites are not met.
- PHPUnit verifies master-access audit evidence for writes and lifecycle
  actions.
- PHPUnit verifies non-System Administrator roles without required permissions
  keep documented denial behavior.

Frontend repository:

- Vitest verifies route guards allow System Administrator through protected
  routes after session context resolves.
- Vitest verifies protected navigation and actions are visible to System
  Administrator.
- Vitest verifies school-scoped routes require active selected school context.
- Vitest verifies self-service routes require selected subject context.
- Vitest verifies non-System Administrator route visibility and denial behavior
  remains permission-based.
