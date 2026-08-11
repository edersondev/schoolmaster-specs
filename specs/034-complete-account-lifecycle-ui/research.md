# Research: Complete Administrator Account Lifecycle UI

## Decision 1: Treat the stale gate as cross-repository completion

**Decision**: Deliver OpenAPI/specification, Laravel conformance/security, and
Vue activation in that order.

**Rationale**: Flipping the frontend constant would expose flows that production
cannot authorize consistently and a create-then-invite sequence guaranteed to
return conflict.

**Alternatives considered**: Frontend-only activation was rejected because it
would bless false behavior; removing create/invite and security acceptance was
rejected because it contradicts the requested feature.

## Decision 2: Add invitation-ready user creation mode

**Decision**: Add optional `account_setup_mode` (`active`, `invitation`) to
`createUser`, default `active`. Invitation mode persists `status=invited`, roles,
school, and unusable generated password but creates no invitation/token/delivery.
The administrator then explicitly calls `createAccountInvitation`.

**Rationale**: This preserves current clients, the clarified two-step UX, and
the existing safe invitation endpoint. Invitation failure leaves a retryable
persisted user. The create flow stores only that user's non-secret UUID in route
intent; navigation or reload restores the invitation phase only after an
authorized tenant-scoped re-fetch confirms the same invited user, never from
draft or route-supplied user details.

**Alternatives considered**: Changing every create to invited was rejected as
breaking; adding invitation-by-user endpoint was rejected because existing
creation works once the persisted state is eligible; automatic invite was
rejected because it combines outcomes and can send unintended email.

## Decision 3: Define user-specific invited status

**Decision**: Add `UserStatus` with `active`, `inactive`, `invited`; keep shared
`Status` unchanged. Only invitation completion transitions invited to active.

**Rationale**: Backend already stores invited users in an unconstrained status
column, but OpenAPI currently omits the value. Shared status is reused by
resources where invited is invalid.

**Alternatives considered**: Adding invited globally was rejected; a new user
table column was rejected because existing status already models the state.

## Decision 4: Make permission identity scope-aware

**Decision**: Replace unique permission code with unique `(code, scope)`, update
seeder lookup keys, seed active `account_lifecycle.manage` for platform and
school, and filter ordinary permission queries by code and scope. Verify the
migration, second seeder run, duplicate composite rejection, and rollback guard
against the configured MySQL test database; SQLite-only evidence is insufficient.

**Rationale**: Current policy requires both scoped records, while production can
store only one. Isolated tests hid this collision.

**Alternatives considered**: Separate code names were rejected because they
would diverge from approved policy/contracts; one shared row was rejected
because role and permission scope would become ambiguous.

## Decision 5: Preserve master access but not non-permission bypasses

**Decision**: Frontend recognizes exact active platform role `System
Administrator`; backend enforces its master permission override through
`AccountLifecyclePolicy`. Services invoke that policy before business transition
rules. Tenant, target, self-action, invited-state, lock-state, and validation
gates still apply.

**Rationale**: Auth sessions may expose no permission rows for a master actor;
the role is authoritative. Master access never means unscoped tenant data.

**Alternatives considered**: Requiring `*` or a permission row was rejected
because session contract does not guarantee either as raw permission data.

## Decision 6: Authorize tenant before scoped target lookup

**Decision**: Resolve active tenant and actor authority first, then query target
inside that scope. Unknown and out-of-school UUIDs share the same non-disclosing
outcome. Actor-equals-target lock/recovery requests are denied by
`AccountLifecyclePolicy` before state read. Frontend user detail navigation
selects one lookup mode before requesting the target: validated route/list intent
first, otherwise school mode when an active school exists, otherwise platform
mode when platform authority exists, otherwise no request. School mode uses the
exact active-school header and school `users.view`. Platform mode sends no school
header, requires platform `schools.view` or System Administrator master access,
and queries only platform-owned users. Lifecycle sections separately require
matching `account_lifecycle.manage` or master access. Existing `listUsers` and
`getUser` OpenAPI descriptions and backend tests must state and prove these
modes. No automatic cross-mode fallback is allowed.

**Rationale**: Existing lookup order distinguishes known cross-tenant IDs from
unknown IDs and permits self-lock behavior contrary to Features 008/034.

**Alternatives considered**: UI-only hiding and post-lookup authorization were
rejected because direct API calls remain possible and reveal existence.

## Decision 7: Use raw scoped session data and local composables

**Decision**: Derive authority from active permission objects, role, target
scope, actor ID, and active school. Keep invitation/action request state in
feature composables; use existing Pinia session store only as session source.

**Rationale**: `permissionCodes` loses scope. Request/form state is route-local
and security-sensitive; a new global store would increase stale state risk.

**Alternatives considered**: Flipping the hardcoded boolean, code-only checks,
and a new lifecycle Pinia store were rejected.

## Decision 8: Invalidate every protected async flow by context snapshot

**Decision**: Lock loads, actions, and invitations capture identity,
permissions, target, target scope, and school generation; changes abort or
invalidate the request and prevent follow-up refresh.

**Rationale**: Current lock generation ignores identity/permission and action
submissions have no stale guard, allowing old responses into a new context.

**Alternatives considered**: Target/school-only request counters were rejected;
late-response UI cleanup was rejected because follow-up requests could already
have crossed context.

## Decision 9: Preserve project JavaScript and test layering

**Decision**: Use Vue 3 Composition API and `<script setup>` in existing
JavaScript style. Use PHPUnit for real authorization/tenancy, Vitest for
contracts/services/composables/components, and stateful mocked Playwright for
SPA journeys. Treat the 390/768/1440 responsive and keyboard/dialog matrix as
formal FR-026/SC-011 automated acceptance, while human administrator and
screen-reader review stays separately pending.

**Rationale**: A TypeScript island or new package adds no feature value.
Playwright's normal repository convention mocks `/api/v1`; it cannot replace
backend policy tests or human UAT.

**Alternatives considered**: TypeScript migration, live backend in default E2E,
and source-text-only component tests were rejected.

## Decision 10: Keep admin resend excluded

**Decision**: Do not call `resendAccountInvitation`; retain blocked/absent admin
resend until separately approved non-secret identification exists.

**Rationale**: Current path requires reusable invitation token material that
admin UI must never receive or store.

**Alternatives considered**: Token persistence, response token exposure, or
relabeling create/replace as resend were rejected.
