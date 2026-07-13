# Feature Specification: System Administrator Master Access

**Feature Branch**: `031-system-admin-master`
**Created**: 2026-07-13
**Status**: Ready for Backend Implementation
**Input**: User description: "Update specs and contracts so the System Administrator role is the master user role. For this implementation slice, update the shared specifications and OpenAPI contract, then implement and verify every protected backend operation. System Administrator access satisfies feature-specific permission checks without bypassing authentication, account state, tenant context, subject ownership, approval workflows, or safety gates. Frontend route and navigation adoption is a separate follow-up implementation."

## Clarifications

### Session 2026-07-13

- Q: Which schools may a System Administrator select for school-scoped work? → A: Any active school.
- Q: Which System Administrator actions require explicit master-access audit marking? → A: All writes and lifecycle actions.
- Q: Does master access bypass approval workflows and safety gates? → A: Permission checks only; approval workflows and safety gates still apply.
- Q: How does this backend slice apply to identity-owned self-service operations? → A: It does not add impersonation or subject selection. Existing actor-owned student access and active guardian-link rules remain enforced; a future contract must define any master-user subject-selection transport before broader access is implemented.
- Q: Which implementation repository is delivered by this feature run? → A: The shared specification/OpenAPI prerequisites and Laravel backend only; frontend route, navigation, and action visibility are deferred to a separate follow-up.
- Q: How is the existing master role identified? → A: An active platform-scoped role whose name is exactly `System Administrator`; no new role identifier or response field is introduced.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Access Protected Backend Reads (Priority: P1)

A System Administrator can call any released read-only backend operation that
is otherwise protected by feature-specific permissions without needing each
permission assigned to the role. State-changing operations are completed with
the audit requirements in User Story 3. Identity-owned self-service operations
remain bound to their existing actor-owned profile or active guardian-link
authorization.

**Why this priority**: The master user role is intended to unblock platform
operations, emergency administration, onboarding, and support workflows without
duplicating every permission across the role.

**Independent Test**: Sign in as a System Administrator with no extra
feature-specific permissions and verify every released protected read operation
group allows the request when its non-permission prerequisites pass.

**Acceptance Scenarios**:

1. **Given** an authenticated System Administrator has no explicit permission
   for a protected operation, **When** they call that operation, **Then** access
   is allowed unless a tenant target, actor-owned profile or guardian link,
   active account, valid session, resource state, approval, or safety
   prerequisite is missing.
2. **Given** an authenticated System Administrator requests the current session,
   **When** the existing session response is returned, **Then** its existing
   role collection identifies the active platform `System Administrator` role
   without adding a response field.
3. **Given** an authenticated System Administrator performs a protected read,
   **When** the action is checked for feature-specific permissions, **Then** the
   System Administrator override satisfies the permission check.
4. **Given** a released self-service operation returns identity-owned student
   or guardian data, **When** a System Administrator calls it without the
   existing actor-owned profile or active guardian link, **Then** access is
   denied without loading another user's self-service data.

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
3. **Given** a System Administrator changes the `X-School-Id` context, **When**
   the next school-scoped request is made, **Then** it returns only the newly
   resolved school's data and never reuses the previous tenant scope.

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
- An identity-owned self-service operation is called by System Administrator;
  existing actor-owned profile or active guardian-link authorization remains
  enforced because this feature defines no impersonation or subject-selection
  transport.
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
- **Frontend repository impact**: Deferred. A separate follow-up must apply the
  published role and authorization rule to route guards, navigation visibility,
  action visibility, and frontend regression tests. This implementation run
  must not modify the frontend repository.
- **Specification or contract repository impact**: Security documentation,
  multi-tenant rules, affected feature specifications, shared contract notes,
  and operation authorization notes must be updated to make the override
  explicit and remove contradictory "no implicit bypass" language.
- **Delivery ownership and sequencing**: Specification and contract updates lead
  first, followed by backend authorization and audit behavior. Frontend adoption
  follows as a separately planned implementation.

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
  account-lock, session-validity, tenant-context, identity-ownership,
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
- **FR-002**: System MUST identify the master role as an active platform-scoped
  role named exactly `System Administrator`.
- **FR-003**: System MUST allow a System Administrator to perform protected
  backend operations without requiring operation-specific permissions.
- **FR-004**: System MUST allow a System Administrator to select any active
  school as the resolved school context before loading or mutating school-owned
  resources that need a tenant target.
- **FR-005**: System MUST keep school-owned responses scoped to the selected or
  resolved school context unless the operation is explicitly platform-wide.
- **FR-006**: System MUST keep authentication, active-account, lockout,
  session-validity, tenant-context, identity-ownership, school-state, and
  feature-release prerequisites enforceable for System Administrator.
- **FR-007**: System MUST preserve existing permission-denial behavior for all
  non-System Administrator roles.
- **FR-008**: System MUST update security rules and tenant rules to state that
  System Administrator master access supersedes feature-specific permission
  checks but does not supersede tenant isolation.
- **FR-009**: System MUST expose the existing System Administrator role context
  through the existing authenticated-session role collection without adding a
  new response field.
- **FR-010**: System MUST update operation authorization notes so protected
  operations identify System Administrator as an allowed master role.
- **FR-011**: System MUST update test expectations to include System
  Administrator allow cases for protected backend operations, plus tenant
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
- **FR-017**: System MUST preserve existing actor-owned student-profile and
  active guardian-link authorization for identity-owned self-service operations;
  this feature MUST NOT introduce impersonation or infer a selected subject.

### Key Entities

- **System Administrator**: Master user role that satisfies all
  feature-specific permission checks while remaining subject to authentication,
  account state, session state, tenant context, and release-state prerequisites.
- **Feature-Specific Permission**: Named capability normally required by a
  route, operation, action, or navigation item for non-System Administrator
  roles.
- **Resolved School Context**: Selected or otherwise confirmed school tenant
  target used to scope school-owned operations.
- **Platform-Wide Operation**: Documented operation that intentionally spans
  multiple schools or platform resources and is not limited to one selected
  school.
- **School-Scoped Operation**: Operation that reads or changes school-owned
  records and must remain scoped to the resolved school context.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In authorization regression checks, 100% of released protected
  backend operation groups include a System Administrator allow case.
- **SC-002**: In tenant isolation checks, 100% of school-scoped System
  Administrator operations return only records for the selected school unless
  documented as platform-wide.
- **SC-003**: In authenticated-session checks, 100% of System Administrator
  sessions expose the active platform master role through the existing role
  collection without a response schema change.
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
- Master access applies to released and approved backend operations, not
  unfinished or intentionally hidden features.
- Frontend route, navigation, and action visibility adoption is deferred to a
  separately planned follow-up and is not part of this implementation run.
- Master access satisfies permission checks only; it does not bypass
  authentication, account status, lockout, session validity, tenant context,
  identity ownership, guardian-link state, or resource lifecycle state.
- Master access does not bypass approval workflows, explicit confirmations,
  support opt-in requirements, file safety gates, closed-period safety checks,
  or other business controls.
- School context may be selected from any active school or otherwise resolved
  through existing approved session behavior before school-owned data is loaded.
- Existing non-System Administrator permission behavior remains unchanged.
- Existing data retention, audit, lifecycle, and soft-delete behavior remains
  unchanged unless a separate feature updates those rules.
