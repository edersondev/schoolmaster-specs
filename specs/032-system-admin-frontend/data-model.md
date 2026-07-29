# Frontend State Model: System Administrator Access

## System Administrator Session

Represents the resolved authenticated session used for client-side visibility
and route decisions.

| Field | Source | Rules |
|---|---|---|
| session status | Existing auth state | Must be resolved and active before protected-route evaluation. |
| roles | Existing authenticated-session role collection | A master role is active, platform-scoped, and named exactly `System Administrator`. |
| permissions | Existing session permission collection | Used unchanged for all non-master role decisions. |

## Authorization Decision

Represents the outcome used by a route, navigation item, or action.

| Field | Meaning | Rules |
|---|---|---|
| feature permission satisfied | Actor may pass the named feature-permission check | True for a resolved System Administrator or an existing permitted non-master actor. |
| prerequisite state | Required non-permission conditions | Authentication, session, school, subject, release, approval, and safety checks remain independent. |
| visibility | Whether a destination or action is shown | Requires both satisfied feature permission and applicable prerequisites. |
| navigation result | Allow, block, redirect, or existing prerequisite state | Must never treat an unresolved session as master access. |

## School Context

Represents the active tenant target for school-owned views.

| Field | Meaning | Rules |
|---|---|---|
| selected school | Current active school identifier | Must be present and active before school-owned loading begins. |
| context generation | Monotonic context-change marker or equivalent invalidation boundary | Prior school-owned state is cleared before the next context loads. |
| loading state | Pending state for the selected school | Cannot display records retained from a prior school. |

## Subject Context

Represents the existing subject target required by applicable identity-owned
routes.

| Field | Meaning | Rules |
|---|---|---|
| selected subject | Existing valid subject context | Required only by routes that already require it. |
| ownership or link state | Existing actor-owned student profile or guardian link condition | System Administrator status does not create, infer, or bypass it. |

## Navigation Surface

Represents a route link, sidebar item, quick action, or page action.

| Field | Meaning | Rules |
|---|---|---|
| release state | Whether the surface belongs to released product behavior | Unreleased surfaces remain hidden for every role. |
| required permissions | Existing feature-permission metadata | Satisfied by master access only for non-identity-owned surfaces. |
| context requirements | School or subject context metadata | Remain required after master permission evaluation. |
| action state | Available, disabled, or blocked with existing feedback | Uses the prerequisite-specific state, not a false permission denial. |
