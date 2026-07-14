# Frontend System Administrator Access Contract

## Purpose

This contract defines how the frontend consumes the completed backend master
access rule. It creates no endpoint, request, response, or audit-contract
change.

## Role Recognition Contract

| Condition | Frontend behavior |
|---|---|
| Resolved active platform role named `System Administrator` | Satisfy released non-identity-owned feature-permission checks. |
| No such role or unresolved session | Use existing permission and session behavior. |
| Inactive, expired, or locked session | Preserve the existing authentication or session-required state. |

The frontend consumes the existing authenticated-session role collection. It
does not require a new boolean, permission, endpoint, or response field.
Role-derived master access is exposed only after the session store reaches its
authenticated state. Explicit permissions retain their existing behavior.

## Route and Visibility Contract

| Surface | System Administrator behavior | Retained requirements |
|---|---|---|
| Released platform route | Allow after resolved active session | Authentication, session, release, and relevant safety states. |
| Released school-owned route | Allow feature-permission evaluation | Active selected school before loading; selected-school scoped response. |
| Released navigation destination or action | Show when applicable prerequisites pass | Release, session, school, approval, and safety states. |
| Student or guardian self-service | Keep existing actor-owned or guardian-link visibility and guards | No master impersonation or subject creation. |
| Unreleased or intentionally hidden surface | Keep hidden | Master access does not change release status. |

## Context-Switch Contract

1. A school-owned view must not start loading without a valid active selected
   school.
2. When the selected school changes, all prior school-owned visible and shared
   state must be invalidated before data for the new school is presented.
3. A stale response associated with the prior school must not repopulate the
   new school context.
4. Platform-wide views remain governed by their published platform-wide scope.

The session store owns a monotonically increasing school-context generation.
Concurrent older selection responses are ignored. The shared
`useSchoolContextSwitch` composable runs registered school-owned state resetters
before asking the existing current-session operation to resolve the selected
school.

## Feedback Contract

The frontend continues to map and present existing backend and guard states:

- missing feature permission for non-System Administrator actors;
- unresolved, expired, or invalid session;
- missing, inactive, or invalid school context;
- missing or invalid subject context;
- approval, confirmation, support, file-safety, closed-period, lifecycle, and
  other business prerequisite states.

System Administrator status removes only the feature-permission cause. It must
not rewrite any other state as a permission denial.

## Verification Contract

- Cover every inventoried released non-identity-owned route group, navigation
  destination, and action with a System Administrator allow case.
- Cover limited-role permission denial for the same shared permission boundary.
- Cover session-resolution ordering, school-context absence, school switching,
  stale response prevention, identity-owned self-service restriction, and
  prerequisite-specific feedback.
- Confirm no new client-side audit record is emitted for state-changing actions.
