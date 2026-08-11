# Feature Specification: Complete Administrator Account Lifecycle UI

**Feature Branch**: `034-complete-account-lifecycle-ui`  
**Created**: 2026-08-10  
**Status**: Draft  
**Input**: User description: "Complete the administrator account lifecycle UI by enabling the approved `account_lifecycle.manage` permission gate; activating lock, unlock, recover, reactivate, and invitation actions; fixing the create-user invitation workflow; keeping invitation resend excluded until a non-secret API contract exists; adding authorization, tenant-context, service, component, and end-to-end tests; and updating Feature 021 documentation and implementation evidence. Treat the stale hardcoded gate as feature-completion scope, not a small visual bug."

## Clarifications

### Session 2026-08-10

- Q: How should invitation creation start after user creation succeeds? → A: Show an explicit invitation action on the create flow after the user persists.
- Q: What should actors without account lifecycle authority see? → A: Hide invitation, lock-state, and lifecycle-action sections and send no lifecycle requests.
- Q: How should the feature resolve backend blockers that make create-then-invite and scoped lifecycle authorization impossible? → A: Expand Feature 034 across contracts, backend, and frontend; add invitation-ready user creation while preserving the active default, provision scoped permissions, deny self-actions, and enforce tenant-safe lookup before activating the UI.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Manage an eligible account (Priority: P1)

An authorized account administrator can open an eligible user in the proper
platform or school context, review current lock state, and perform the lock,
unlock, recovery, or reactivation actions allowed by that user's current state.

**Why this priority**: Feature 021 delivered the account lifecycle surfaces but
left every administrator action disabled behind a stale hardcoded gate. Removing
that completion blocker provides the main operational value of this feature.

**Independent Test**: Sign in with `account_lifecycle.manage`, open eligible
active, locked, and inactive users in the permitted scope, perform each available
action, and verify state and available actions refresh after every success.

**Acceptance Scenarios**:

1. **Given** an authenticated actor has account lifecycle authority for the
   target's scope and all required tenant context is active, **When** the actor
   opens the target user's detail, **Then** the system loads lock state and shows
   only actions allowed by the target's current state.
2. **Given** an eligible unlocked account, **When** the actor confirms lock with
   a valid reason, **Then** the account becomes locked and unlock and recovery
   replace lock as the available actions.
3. **Given** an eligible administratively locked account, **When** the actor
   unlocks it without a reason or recovers it with an optional reason, **Then**
   the lock is cleared and the visible state is refreshed.
4. **Given** an eligible inactive account, **When** the actor reactivates it with
   or without an optional reason, **Then** the resulting account state is shown
   without implying that unrelated eligibility constraints were bypassed.
5. **Given** an authenticated actor lacks account lifecycle authority, **When**
   the actor opens the same user detail, **Then** invitation, lock-state, and
   lifecycle-action sections are hidden and no lifecycle request is sent.

---

### User Story 2 - Create and invite a user (Priority: P1)

An authorized school administrator can create an invitation-ready user and then
explicitly create that user's account invitation from the same create-user
journey without re-entering identity or role information and without attempting
an invitation before the user exists.

**Why this priority**: The current create-user surface renders an invitation
control with no created user target, so the documented create-and-invite journey
cannot be completed.

**Independent Test**: Create a valid user as an authorized administrator, use
the explicit invitation action shown after persistence, and verify invitation
status, expiry, `delivery_channel`, and `delivery_requested_at` are shown while
the reusable invitation token remains absent.

**Acceptance Scenarios**:

1. **Given** an authorized school administrator is entering a new user, **When**
   the user has not yet been created, **Then** invitation creation cannot run
   against draft form data.
2. **Given** valid user creation uses invitation account setup mode, **When**
   creation succeeds, **Then** the user is persisted in invited state without
   creating an invitation or requesting delivery, and the create flow shows an
   explicit invitation action without requiring re-entry of name, email, role,
   scope, or school information.
3. **Given** invitation creation succeeds, **When** the result is shown, **Then**
   it includes only safe invitation `status`, `expires_at`, `delivery_channel`,
   and `delivery_requested_at` and never exposes or retains the invitation token.
4. **Given** user creation succeeds but invitation creation fails, **When** the
   failure is shown, **Then** the UI distinguishes the persisted invited user
   from the failed invitation and provides a safe retry path for that same user.
5. **Given** user creation fails, **When** validation or service feedback is
   shown, **Then** no invitation request is sent.

---

### User Story 3 - Preserve authorization and tenant isolation (Priority: P2)

Platform and school account administrators can use account lifecycle controls
only within their approved scope, while unauthorized, missing-context,
cross-tenant, inactive-school, and self-target attempts remain safely blocked.

**Why this priority**: Activating previously blocked security-sensitive actions
must preserve the authorization and tenant boundaries already defined by the
account lifecycle contracts.

**Independent Test**: Exercise each administrator action with authorized,
unauthorized, missing-school, mismatched-school, inactive-school, cross-tenant,
platform-versus-school, and self-target combinations and verify only permitted
requests are issued.

**Acceptance Scenarios**:

1. **Given** a school-scoped target and no active permitted school, **When** the
   user detail is opened, **Then** invitation, lock-state, and lifecycle-action
   sections are hidden and no school-owned lifecycle request is sent.
2. **Given** a school administrator and a target from another school, **When**
   any lifecycle entry point is reached, **Then** the system denies the action
   without revealing whether the target exists.
3. **Given** a platform-scoped administrator acting on a platform-scoped target,
   **When** no school context is required by that target scope, **Then** account
   lifecycle authority is evaluated in explicit platform lookup mode without
   inventing a school dependency or falling back to a school-owned lookup.
4. **Given** a System Administrator with documented master access, **When** all
   non-permission prerequisites and target-scope requirements are satisfied,
   **Then** account lifecycle permission checks succeed while tenant, account
   state, self-action, and other business controls remain enforced.
5. **Given** a lifecycle request returns forbidden, tenant mismatch, inactive
   school, not found, conflict, validation, or temporary failure, **When** the
   response is displayed, **Then** feedback is safe, actionable, and does not
   expose hidden user, school, role, permission, or secret data.

---

### User Story 4 - Keep secret-dependent resend unavailable (Priority: P3)

Administrators receive no invitation resend action while the only approved
resend contract requires a reusable invitation token in its path.

**Why this priority**: Completing invitation creation must not reopen the known
secret-handling contract gap or imply that resend is part of this feature.

**Independent Test**: Review create-user and user-detail invitation surfaces for
all supported roles and states and verify no resend request can be initiated,
while invitation creation remains usable for authorized actors.

**Acceptance Scenarios**:

1. **Given** an invitation exists, **When** an administrator views invitation
   controls, **Then** no resend action calls the token-path resend operation.
2. **Given** the published contract still lacks resend by a non-secret
   invitation or user identifier, **When** this feature is completed, **Then**
   resend remains explicitly excluded in behavior, tests, and evidence.
3. **Given** a future non-secret resend contract is approved, **When** resend is
   considered for delivery, **Then** it is handled through separately documented
   scope rather than inferred by this feature.

### Edge Cases

- The active school changes while lock state or an action request is in flight;
  the stale result must not update the new context.
- Permission or authenticated identity changes while a user detail or
  create-and-invite flow is open; controls must be re-evaluated before request.
- User creation is submitted twice while the first request is pending; at most
  one persisted user may become the invitation target for that submission.
- User creation succeeds immediately before navigation, refresh, or retry;
  invitation retry must target the persisted user rather than draft form data.
- The target user is the acting user, soft-deleted, inactive in an inactive
  school, missing active role assignments, or has unresolved invitation or
  password setup state.
- Lock state changes in another session between display and confirmation; the
  system must show conflict feedback and refresh authoritative state.
- An actor has general user-management permissions but lacks
  `account_lifecycle.manage`; general permissions must not unlock lifecycle
  controls.
- An actor has `account_lifecycle.manage` for a different scope or school;
  matching code alone must not bypass scope and tenant checks.
- Invitation creation reaches its rate or state limit after user creation;
  user creation remains successful and invitation failure remains explicit.
- A service error contains secret or tenant-private fields; visible feedback
  and diagnostics must not echo them.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Add invitation-ready user creation while
  preserving existing active-user creation by default; provision platform and
  school `account_lifecycle.manage` permissions under scope-aware uniqueness;
  prevent generic lifecycle actions from activating invited users; deny
  self-target account lock and recovery operations through
  `AccountLifecyclePolicy`; make list/detail lookup platform-only or exact-school
  before target resolution; scope lifecycle target lookup after tenant
  authorization; and add focused backend coverage.
- **Frontend repository impact**: Replace the stale administrator lifecycle gate
  with the approved permission and scope-aware eligibility rules; complete the
  create-user-to-invitation handoff; activate existing lock, unlock, recovery,
  reactivation, lock review, and invitation services and components; preserve
  blocked resend; and add focused and end-to-end regression coverage.
- **Specification or contract repository impact**: Add this completion feature;
  update Feature 008 and Feature 021 rules and evidence; extend the user-create
  request with account setup mode; define user-specific invited status; document
  tenant-safe and self-action behavior; and regenerate the aggregate contract.
- **Delivery ownership and sequencing**: `schoolmaster-specs` and OpenAPI lead,
  `schoolmaster-backend` delivers contract conformance and security gates next,
  and `schoolmaster-frontend` activates the UI only after backend verification.
  Feature 021 evidence is updated after full cross-repository validation.

### API Contract Impact

- **OpenAPI update required**: Yes. Add optional `account_setup_mode` with
  `active` default and `invitation` mode to `UserCreateRequest`; define
  user-specific `active`, `inactive`, and `invited` status; document invitation
  mode side effects, user list/detail lookup modes, and account lifecycle
  self-action and tenant boundaries. No new endpoint is added, and token-path
  administrator resend stays excluded.
- **Versioned endpoints affected**:
  - `listUsers` - `GET /api/v1/users`
  - `createUser` - `POST /api/v1/users`
  - `getUser` - `GET /api/v1/users/{userId}`
  - `createAccountInvitation` - `POST /api/v1/account-invitations`
  - `completeAccountInvitation` - `POST /api/v1/account-invitations/{invitationToken}/setup`
  - `getAccountLock` - `GET /api/v1/users/{userId}/account-lock`
  - `lockAccount` - `POST /api/v1/users/{userId}/account-lock`
  - `unlockAccount` - `DELETE /api/v1/users/{userId}/account-lock`
  - `reactivateAccount` - `POST /api/v1/users/{userId}/account-reactivation`
- **JSON response impact**: Existing envelopes remain. `User.status` gains the
  documented `invited` value, and invitation-mode creation returns the normal
  user response without invitation or delivery data. The frontend continues to
  consume documented invitation, lock, lifecycle result, validation, forbidden,
  tenant, not-found, conflict, and transport-failure outcomes.
- **Authentication/authorization impact**: Administrative controls require an
  authenticated active actor with `account_lifecycle.manage` in the applicable
  platform or school scope, or documented System Administrator master access.
  User list/detail bootstrap retains existing read authority: school-scoped
  `users.view` with an exact school header, or platform-scoped `schools.view` (or
  System Administrator master access) with no school header and a
  platform-owned-only query. Lifecycle sections and actions additionally require
  `account_lifecycle.manage` in the target scope or master access. School-owned
  actions also require active permitted school context. All non-permission
  business gates remain enforceable.
- **Compatibility impact**: `account_setup_mode` is additive and defaults to
  existing active-user behavior. The new user-status enum value requires
  contract-first backend delivery and frontend handling before invitation mode
  is used. `resendAccountInvitation` remains unconsumed by administrator UI
  because its path requires a reusable invitation token.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-scoped lifecycle reads and writes require the
  active permitted school and must carry that context. Permission checks,
  target eligibility, and stale-response handling must be recomputed when school
  context changes. Tenant authorization must precede scoped target lookup so
  out-of-scope and unknown targets cannot be distinguished. Platform-scoped
  targets do not acquire an artificial school dependency. User detail lookup
  operates in one route lookup mode selected before the request: a validated
  route/list intent wins; otherwise an active school selects school mode;
  otherwise platform authority selects platform mode; otherwise the request is
  blocked. School mode requires the exact active school and header. Platform mode
  sends no school header, queries platform-owned users only, and never retries in
  the other mode.
- **Cross-tenant or platform access impact**: Platform and school lifecycle
  authority remain separate. Master access may satisfy permission checks but
  does not authorize unscoped school-owned data or bypass tenant prerequisites.
- **Soft delete impact**: No restoration path is added. Soft-deleted users remain
  blocked from account lifecycle recovery or reactivation unless separately
  restored through approved administration behavior.
- **Permission storage impact**: Permission identity becomes unique by code and
  scope so active platform and school `account_lifecycle.manage` permissions can
  coexist. Existing role-permission relationships remain identifier-based.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST recognize active `account_lifecycle.manage`
  permission in the applicable target scope as the approved administrator
  account lifecycle permission source.
- **FR-002**: The system MUST recognize documented System Administrator master
  access as satisfying the permission check while preserving all tenant,
  target-state, self-action, and other non-permission prerequisites.
- **FR-003**: The system MUST NOT grant account lifecycle authority solely from
  general user view, user management, role view, or unrelated permissions; for
  actors without lifecycle authority, invitation, lock-state, and
  lifecycle-action sections MUST be hidden and no lifecycle request sent.
- **FR-004**: School-scoped account lifecycle state and controls MUST remain
  unavailable until an active permitted school context is resolved, and every
  resulting request MUST use that same context.
- **FR-005**: Platform-scoped account lifecycle eligibility MUST be evaluated by
  matching platform lifecycle authority without requiring an unrelated active
  school context; `listUsers` and `getUser` lookup mode, header, ownership, and
  no-fallback mechanics MUST follow FR-027.
- **FR-006**: Authorized administrators MUST be able to review account lock state
  and initiate only lifecycle actions eligible for the target's current state.
- **FR-007**: Lock MUST require a non-empty reason, unlock MUST submit no reason,
  and recovery and reactivation MUST permit an optional reason within existing
  validation limits.
- **FR-008**: Successful lifecycle actions MUST refresh the displayed lock and
  account state so obsolete actions are removed and newly eligible actions are
  shown.
- **FR-009**: The school create-user workflow MUST support invitation account
  setup mode, MUST use the successfully persisted invited user as the invitation
  target, and MUST NOT issue an invitation for unsaved draft data.
- **FR-010**: After user creation succeeds, the create flow MUST show an explicit
  invitation action to an authorized administrator and MUST allow that action
  without re-entering the persisted user's identity, roles, scope, or school
  context. The flow MUST preserve only the non-secret persisted-user UUID in
  route intent and, after navigation or reload, MUST restore invitation
  eligibility only by re-fetching that UUID under the current authorized tenant
  context; it MUST NOT reconstruct the target from draft or route-supplied user
  details.
- **FR-011**: User creation and invitation creation MUST remain distinguishable
  outcomes; invitation failure MUST NOT report or imply that a successfully
  persisted user was not created.
- **FR-012**: Invitation creation MUST submit only documented scope, school,
  identity, and role fields, MUST omit `delivery_metadata`, and MUST never expose
  or retain a reusable invitation token. Administrator-visible invitation result
  data MUST be limited to `status`, `expires_at`, `delivery_channel`, and
  `delivery_requested_at`; other documented response identifiers remain service
  data and MUST NOT be rendered as delivery diagnostics.
- **FR-013**: Administrator invitation resend MUST remain unavailable and MUST
  NOT consume `resendAccountInvitation` while that operation requires an
  invitation token.
- **FR-014**: Account lifecycle controls MUST be re-evaluated when identity,
  permissions, target, target state, or active school context changes.
- **FR-015**: Responses started under stale identity, target, permission, or
  school context MUST NOT update the current view or trigger a follow-up action.
- **FR-016**: Forbidden, tenant-mismatch, inactive-school, not-found, conflict,
  validation, and temporary-failure feedback MUST remain safe and MUST NOT
  disclose hidden account, role, school, permission, or secret data.
- **FR-017**: Automated verification MUST cover permission authorization,
  System Administrator access, missing and mismatched tenant context,
  cross-tenant denial, service request/response behavior, action eligibility,
  create-and-invite component behavior, stale responses, blocked resend, and
  complete administrator journeys in a browser.
- **FR-018**: Feature 021 documentation and evidence MUST be updated to replace
  the obsolete hardcoded-gate limitation with the verified permission source,
  tests executed, remaining resend exclusion, and any pending manual evidence.
- **FR-019**: This feature MUST NOT add or consume undocumented endpoints,
  response fields, lifecycle actions, authorization exceptions, or resend
  behavior.
- **FR-020**: `createUser` MUST preserve active-user creation when account setup
  mode is omitted or active, and invitation mode MUST persist the user in invited
  state without creating an invitation, issuing a lifecycle token, or requesting
  delivery.
- **FR-021**: Only successful invitation completion MAY transition an invited
  user to active; generic user activation, update, recovery, or reactivation
  behavior MUST NOT bypass first password setup.
- **FR-022**: The system MUST store and seed active
  `account_lifecycle.manage` permissions independently for platform and school
  scopes and MUST evaluate the requested scope when checking ordinary actors.
- **FR-023**: Account lock review, lock, unlock, recovery, and reactivation MUST
  deny actor-equals-target attempts before reading protected lock state or
  applying any mutation, and backend authorization for those operations MUST be
  enforced through `AccountLifecyclePolicy` before service business rules run.
- **FR-024**: School tenant context and actor authority MUST be resolved before
  target lookup, and target lookup MUST remain scoped so unknown and
  out-of-school identifiers produce the same non-disclosing outcome.
- **FR-025**: Backend verification MUST cover invitation-mode creation and retry,
  active-default compatibility, invited-to-active setup, both permission scopes
  coexisting, System Administrator master access, ordinary scoped actors,
  self-action denial, tenant-safe lookup ordering, and lifecycle transition
  conflicts.
- **FR-026**: Create-user, user-detail, invitation-result/error, lock-state, and
  lifecycle-dialog surfaces MUST avoid horizontal overflow at 390, 768, and 1440
  pixel widths; expose semantic headings, named inputs and controls, and live
  status/error feedback; support keyboard reachability, dialog focus containment
  and return, Escape/cancel, and pending-state submission protection.
- **FR-027**: `listUsers` and `getUser` MUST select exactly one lookup mode before
  target resolution. School mode MUST require school-scoped `users.view`, the
  exact active school header, and a same-school query. Platform mode MUST require
  platform-scoped `schools.view` or System Administrator master access, omit the
  school header, and query only platform-owned users. Lifecycle sections MUST
  still require matching `account_lifecycle.manage` or master access, and neither
  mode may fall back to the other.

### Key Entities

- **Administrator Session**: Authenticated actor identity, active permissions,
  roles, master-access state, and selected school context used to decide whether
  administrator lifecycle controls may be used.
- **Target User**: Persisted platform- or school-scoped account whose lifecycle
  state, role dependencies, tenant ownership, and relationship to the actor
  determine eligible actions.
- **Account Setup Mode**: User-creation choice of `active` or `invitation`;
  omission preserves active behavior, while invitation mode persists an invited
  user and requires a separate explicit invitation action.
- **Account Lock**: Current lock status and safe display metadata that determine
  whether lock, unlock, or recovery is available.
- **Account Lifecycle Action**: Lock, unlock, recovery, or reactivation request
  with its target, optional or required reason, eligibility, pending state, and
  outcome.
- **Account Invitation**: Safe administrator view of invitation status, expiry,
  `delivery_channel`, and `delivery_requested_at` for a persisted target user;
  reusable token secrets and request `delivery_metadata` are excluded.
- **Tenant Context**: Active school selected for school-owned requests; it must
  match both actor authority and target ownership.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In automated authorization coverage, 100% of actors with valid
  account lifecycle authority and context can reach eligible controls, and 100%
  of actors without that authority issue no lifecycle request.
- **SC-002**: Automated tenant coverage rejects 100% of missing, inactive,
  mismatched, and cross-tenant school contexts without exposing protected target
  state or applying stale results.
- **SC-003**: Authorized browser journeys successfully complete lock, unlock,
  recovery, reactivation, and invitation creation, with visible state refreshed
  after every successful operation.
- **SC-004**: A newly created eligible user can be invited in the same
  create-user journey without re-entering user data, and invitation failure is
  reported separately from user creation in 100% of covered failure cases.
  Automated reload coverage proves UUID route intent restores the same target
  only after an authorized tenant-scoped re-fetch and never creates a second
  user or falls back to draft data.
- **SC-005**: Security verification finds zero administrator resend requests,
  reusable invitation tokens, permission payloads, tenant-private details, or
  submitted plaintext reasons in post-submit feedback, diagnostics, logs, or
  client persistence. A reason may exist only in the active reason input and the
  documented outgoing lock, recovery, or reactivation request.
- **SC-006**: Focused service, authorization, tenant-context, component, and
  browser-level test suites pass with no regression in existing Feature 021
  guest invitation setup or password-reset journeys.
- **SC-007**: Feature 021 evidence names the active permission source, records
  executed verification and results, preserves the resend exclusion, and lists
  any genuinely pending manual review without stale completion blockers.
- **SC-008**: Contract and backend verification confirms 100% of invitation-mode
  creates have no invitation/token/delivery side effect, active-default creates
  remain compatible, and only completed invitation setup activates invited
  users.
- **SC-009**: Backend authorization coverage rejects 100% of self-target,
  missing-context, mismatched-context, inactive-school, and cross-tenant cases in
  the defined matrix without revealing target existence or changing state.
- **SC-010**: Verification against the configured MySQL test database applies the
  permission-index migration, runs the seeder twice, proves platform and school
  `account_lifecycle.manage` rows coexist under `(code, scope)`, rejects duplicate
  composite rows, authorizes only matching scopes, preserves System Administrator
  non-permission gates, and proves rollback refuses or reconciles duplicate codes
  before restoring code-only uniqueness.
- **SC-011**: Automated Chromium, Firefox, and WebKit coverage passes the defined
  responsive and keyboard/dialog matrix at 390, 768, and 1440 pixel widths with
  zero horizontal overflow, named controls, correct live feedback semantics,
  contained/restored focus, Escape/cancel behavior, and duplicate-submit
  prevention. Human administrator and screen-reader review remains pending unless
  separately recorded as performed.
- **SC-012**: OpenAPI and backend feature verification prove 100% of covered
  `listUsers` and `getUser` requests use exactly one authorized lookup mode,
  return only platform-owned or exact-school users for that mode, perform no
  cross-mode fallback, and give unknown and opposite-mode identifiers the same
  non-disclosing result.

## Assumptions

- Existing backend policy establishes `account_lifecycle.manage` as the approved
  code, but this feature must correct production provisioning and scope-aware
  uniqueness before frontend activation.
- Current authenticated-session permission data and documented System
  Administrator master-access behavior remain the authoritative frontend inputs.
- Existing account lifecycle route names, response envelopes, password rules,
  rate limits, and audit behavior remain unless this feature explicitly tightens
  self-action, tenant lookup, invited-user, or permission-scope behavior.
- Recovery continues to use the approved recovery action supported by the
  existing account reactivation contract; no new recovery endpoint is added.
- User storage already supports invited state; no user-table migration is
  required. A created invited user remains persisted if subsequent invitation
  creation fails, and the administrator can retry for that same user.
- Admin resend remains out of scope until a separately approved non-secret
  contract identifies an invitation or user without requiring reusable token
  material.
- Guest invitation setup and password-reset request/completion behavior from
  Feature 021 are regression scope only, not new product behavior.
