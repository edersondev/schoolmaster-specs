# Feature Specification: System Administrator Master Access

**Feature Branch**: `031-system-admin-master`
**Created**: 2026-07-13
**Status**: Draft
**Input**: User description: "Update specs and contracts so the System Administrator role is the master user role. A System Administrator must have access to every frontend route and every backend operation in the app, across platform-scoped and school-scoped areas, without requiring explicit per-feature permissions. For school-scoped resources, System Administrator access must still use a resolved school context when the operation needs a tenant target, but authorization must not be denied only because school-scoped permissions are missing. Update security rules, administration specs, route guard expectations, OpenAPI authorization notes, and test expectations to document and verify this global override behavior. Keep tenant isolation intact: System Administrator may select or operate within any permitted school context, but responses must remain scoped to the selected tenant unless the operation is explicitly platform-wide."

## Clarifications

### Session 2026-07-13

- Q: Which schools may a System Administrator select for school-scoped work? → A: Any active school.
- Q: Which System Administrator actions require explicit master-access audit marking? → A: All writes and lifecycle actions.
- Q: Does master access bypass approval workflows and safety gates? → A: Permission checks only; approval workflows and safety gates still apply.
- Q: How does master access apply to identity-owned self-service routes? → A: Released self-service routes are accessible, but identity-owned pages require selected subject context.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Access Every Authorized Workspace (Priority: P1)

A System Administrator can open any application workspace, page, or action that
is otherwise protected by feature-specific permissions, without needing each
individual permission assigned to their role. Released self-service routes are
included when a required student, guardian, user, or other subject context is
selected.

**Why this priority**: The master user role is intended to unblock platform
operations, emergency administration, onboarding, and support workflows without
duplicating every permission across the role.

**Independent Test**: Sign in as a System Administrator with no extra
feature-specific permissions and verify protected navigation, direct route
access, and protected actions that require feature permissions are available.

**Acceptance Scenarios**:

1. **Given** an authenticated System Administrator has no explicit permission
   for a protected route, **When** they open that route directly, **Then** route
   access is allowed unless the route requires a missing tenant target,
   missing subject context, inactive account, expired session, or other
   non-permission prerequisite.
2. **Given** an authenticated System Administrator opens the application
   navigation, **When** protected destinations are evaluated, **Then** every
   destination is visible except destinations blocked by unfinished feature
   release state, missing tenant target, inactive account, or session failure.
3. **Given** an authenticated System Administrator performs a protected action,
   **When** the action is checked for feature-specific permissions, **Then** the
   System Administrator override satisfies the permission check.
4. **Given** a released self-service route displays identity-owned student,
   guardian, user, or other subject data, **When** a System Administrator opens
   it without a selected subject context, **Then** the route shows the required
   subject-context state instead of loading another user's self-service data.

---

### User Story 2 - Operate Inside School Contexts (Priority: P2)

A System Administrator can access school-scoped resources by selecting any
active school or using a resolved school context, and the results stay limited
to that school unless a platform-wide view is explicitly defined.

**Why this priority**: The master role must not weaken tenant isolation. School
context remains the boundary for school-owned data even when permission checks
are globally overridden.

**Independent Test**: Sign in as a System Administrator, select any active
school, open school-owned resources, confirm only that school's data is visible,
switch to a different active school, and confirm the data changes to the new
school context.

**Acceptance Scenarios**:

1. **Given** a school-scoped route requires a tenant target, **When** a System
   Administrator opens it without a resolved school context, **Then** the
   application requests or shows the required school-context state instead of
   loading school-owned data.
2. **Given** a System Administrator has selected a school context, **When** they
   list, view, create, update, or run lifecycle actions for school-owned
   resources, **Then** authorization is allowed and returned data is scoped to
   the selected school.
3. **Given** a System Administrator switches school context, **When** the
   current page reloads or refreshes data, **Then** previous school-owned data
   is cleared and only the newly selected school's data is shown.

---

### User Story 3 - Audit Master Access Decisions (Priority: P3)

Administrators and reviewers can verify when System Administrator master access
was used for writes and lifecycle actions so privileged operations remain
accountable.

**Why this priority**: A global override is powerful. The product must make the
behavior explicit in documentation, tests, and review evidence so tenant safety
and operational accountability remain clear.

**Independent Test**: Review the updated security, route, contract, and test
expectations and confirm they describe System Administrator override decisions,
master-access audit markers for writes and lifecycle actions, and tenant-scoped
boundaries consistently.

**Acceptance Scenarios**:

1. **Given** a protected permission check is documented, **When** the rule is
   reviewed, **Then** it states that System Administrator satisfies the
   permission requirement without needing the named permission assigned.
2. **Given** a school-scoped operation is documented, **When** the rule is
   reviewed, **Then** it states that school context remains required and output
   remains tenant-scoped.
3. **Given** regression tests cover protected routes and operations, **When**
   they are reviewed, **Then** they include System Administrator allow cases and
   non-System Administrator denial cases.
4. **Given** a System Administrator creates, updates, restores, activates,
   deactivates, deletes, imports, retries, cancels, approves, revokes, or
   otherwise changes protected state, **When** the action is recorded for
   review, **Then** the audit evidence marks that master access was used.

### Edge Cases

- A System Administrator session expires while opening a protected route; the
  route remains denied until re-authentication succeeds.
- A System Administrator account is inactive or locked; master access does not
  bypass account lifecycle restrictions.
- A school-scoped operation has no selected or resolved school context; the
  operation remains blocked until a tenant target is available.
- A self-service route has no selected subject context; the route remains
  blocked until a student, guardian, user, or other required subject target is
  selected.
- A selected school is inactive, missing, or unavailable; school-owned
  operations remain blocked by school state even though permissions are
  satisfied.
- A feature is unreleased or intentionally hidden for product sequencing;
  master access does not expose unfinished workflows unless the feature is
  approved for use.
- An operation requires approval, confirmation, support opt-in, closed-period
  safety checks, file scan clearance, or another business safety gate; master
  access satisfies permission checks only and the safety gate still applies.
- A platform-wide operation intentionally returns cross-school data; the
  contract must identify that operation as platform-wide.
- A school-scoped operation accidentally attempts to return records from more
  than one school; the response must be rejected or corrected to the selected
  tenant scope.
- A non-System Administrator has broad permissions but not the required
  permission for a route; existing denial behavior remains unchanged.
- A System Administrator performs read-only navigation or list/detail viewing;
  normal access evidence remains sufficient unless the operation already
  requires a specific audit record.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: Authorization rules, policy expectations,
  protected route behavior, tenant-context validation, and regression tests must
  recognize System Administrator as the master user role while preserving
  non-permission prerequisites.
- **Frontend repository impact**: Route guards, navigation visibility, action
  visibility, tenant-context gating, denied states, and regression tests must
  treat System Administrator as satisfying all feature-specific permission
  checks.
- **Specification or contract repository impact**: Security documentation,
  multi-tenant rules, affected feature specifications, shared contract notes,
  and operation authorization notes must be updated to make the override
  explicit and remove contradictory "no implicit bypass" language.
- **Delivery ownership and sequencing**: Specification and contract updates lead
  first, followed by backend authorization behavior, then frontend route and
  navigation behavior, with test evidence captured for each affected surface.

### API Contract Impact

- **OpenAPI update required**: Yes. Authorization descriptions for protected
  operations must document System Administrator master access and preserve
  tenant-context requirements for school-owned operations.
- **Versioned endpoints affected**: All protected operations are affected by the
  authorization note; school-scoped operations are affected by the retained
  tenant-context rule.
- **JSON response impact**: No success envelope or resource field changes are
  required. Forbidden or tenant-context responses must remain distinct so users
  can tell permission denial from missing tenant context.
- **Authentication/authorization impact**: System Administrator satisfies every
  feature-specific permission requirement. Authentication, active-account,
  account-lock, session-validity, tenant-context, subject-context,
  school-state, and feature release prerequisites still apply. Approval
  workflows and business safety gates also still apply.
- **Compatibility impact**: Authorization behavior changes for System
  Administrator by expanding access. Other roles keep existing permission-based
  behavior.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-owned records remain scoped to the resolved
  school context. System Administrator does not receive unscoped school-owned
  results unless an operation is explicitly platform-wide.
- **Cross-tenant or platform access impact**: System Administrator may access
  platform-wide routes and may select any available school context for
  school-scoped work. Cross-school output is allowed only for documented
  platform-wide operations.
- **Soft delete impact**: No lifecycle rule changes are introduced. Access to
  soft-deleted or restored records follows each operation's existing lifecycle
  rules, with System Administrator satisfying permission checks only.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST define System Administrator as the master user role
  for authorization decisions.
- **FR-002**: System MUST allow a System Administrator to access protected
  frontend routes without requiring route-specific permissions.
- **FR-003**: System MUST allow a System Administrator to perform protected
  backend operations without requiring operation-specific permissions.
- **FR-004**: System MUST allow a System Administrator to select any active
  school as the resolved school context before loading or mutating school-owned
  resources that need a tenant target.
- **FR-005**: System MUST keep school-owned responses scoped to the selected or
  resolved school context unless the operation is explicitly platform-wide.
- **FR-006**: System MUST keep authentication, active-account, lockout,
  session-validity, tenant-context, subject-context, school-state, and
  feature-release prerequisites enforceable for System Administrator.
- **FR-007**: System MUST preserve existing permission-denial behavior for all
  non-System Administrator roles.
- **FR-008**: System MUST update security rules and tenant rules to state that
  System Administrator master access supersedes feature-specific permission
  checks but does not supersede tenant isolation.
- **FR-009**: System MUST update route guard expectations so System
  Administrator sees and can open protected navigation destinations while
  preserving tenant-context gates.
- **FR-010**: System MUST update operation authorization notes so protected
  operations identify System Administrator as an allowed master role.
- **FR-011**: System MUST update test expectations to include System
  Administrator allow cases for protected routes and operations, plus tenant
  isolation checks for school-scoped data.
- **FR-012**: System MUST ensure forbidden responses caused only by missing
  feature-specific permissions are not returned to System Administrator.
- **FR-013**: System MUST preserve tenant-context or school-state denial
  responses for System Administrator when those prerequisites are missing or
  invalid.
- **FR-014**: System MUST document any operation that returns cross-school data
  as platform-wide before System Administrator can receive unscoped output from
  it.
- **FR-015**: System MUST mark audit evidence for System Administrator writes
  and lifecycle actions to show that master access was used.
- **FR-016**: System MUST enforce approval workflows, explicit confirmations,
  support opt-in requirements, file safety gates, closed-period safety checks,
  and other business controls for System Administrator.
- **FR-017**: System MUST allow System Administrator to access released
  identity-owned self-service routes only after the required student, guardian,
  user, or other subject context is selected.

### Key Entities

- **System Administrator**: Master user role that satisfies all
  feature-specific permission checks while remaining subject to authentication,
  account state, session state, tenant context, and release-state prerequisites.
- **Feature-Specific Permission**: Named capability normally required by a
  route, operation, action, or navigation item for non-System Administrator
  roles.
- **Resolved School Context**: Selected or otherwise confirmed school tenant
  target used to scope school-owned operations.
- **Selected Subject Context**: Selected student, guardian, user, or other
  person-specific target required before loading identity-owned self-service
  data through master access.
- **Platform-Wide Operation**: Documented operation that intentionally spans
  multiple schools or platform resources and is not limited to one selected
  school.
- **School-Scoped Operation**: Operation that reads or changes school-owned
  records and must remain scoped to the resolved school context.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In authorization regression checks, 100% of protected route and
  operation groups include a System Administrator allow case.
- **SC-002**: In tenant isolation checks, 100% of school-scoped System
  Administrator operations return only records for the selected school unless
  documented as platform-wide.
- **SC-003**: In route visibility checks, a System Administrator can see and
  open 100% of released protected navigation destinations after any required
  school or subject context is resolved.
- **SC-004**: In negative authorization checks, 100% of non-System
  Administrator roles without required permissions continue to receive the
  documented denial behavior.
- **SC-005**: Review confirms 100% of security, tenant, route, operation, and
  test documentation touched by this feature uses one consistent master-access
  rule.
- **SC-006**: Review confirms 100% of System Administrator write and lifecycle
  action test cases verify that master-access audit evidence is present.

## Assumptions

- "System Administrator" refers to the existing platform role intended to act
  as the product master user.
- Master access applies to released and approved application routes and
  operations, not unfinished or intentionally hidden features.
- Master access satisfies permission checks only; it does not bypass
  authentication, account status, lockout, session validity, tenant context,
  subject context, or resource lifecycle state.
- Master access does not bypass approval workflows, explicit confirmations,
  support opt-in requirements, file safety gates, closed-period safety checks,
  or other business controls.
- School context may be selected from any active school or otherwise resolved
  through existing approved session behavior before school-owned data is loaded.
- Existing non-System Administrator permission behavior remains unchanged.
- Existing data retention, audit, lifecycle, and soft-delete behavior remains
  unchanged unless a separate feature updates those rules.
