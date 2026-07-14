# Feature Specification: System Administrator Frontend Access

**Feature Branch**: `032-system-admin-frontend`  
**Created**: 2026-07-14  
**Status**: Draft  
**Input**: User description: "Implement frontend System Administrator master access."

## Clarifications

### Session 2026-07-14

- Q: How does master access apply to identity-owned student and guardian
  self-service routes? → A: Keep them restricted to their existing actor or
  guardian context; master access does not expose them.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Access Protected Frontend Workspaces (Priority: P1)

A signed-in System Administrator can open every released non-identity-owned
protected workspace and see its protected navigation destinations and available
actions without needing individual feature permissions. Student self-service
and guardian self-service remain available only through their existing actor or
guardian context.

**Why this priority**: The master role is ineffective in the product interface
if its members remain blocked or cannot discover the workspaces already
authorized by the backend.

**Independent Test**: Sign in with an active System Administrator role and no
feature-specific permissions, then open each released protected route group
and confirm its navigation destinations and available actions are presented.

**Acceptance Scenarios**:

1. **Given** a signed-in System Administrator has no named permission for a
   released protected route, **When** they navigate to that route directly or
   through navigation, **Then** the route opens after required session and
   context checks pass.
2. **Given** a signed-in System Administrator views protected navigation,
   **When** its entries or actions normally require feature permissions,
   **Then** the released destinations and actions are visible and available.
3. **Given** a non-System Administrator lacks a required permission, **When**
   they open a protected route or view its navigation, **Then** existing denied
   route and hidden-action behavior remains unchanged.
4. **Given** a System Administrator session expires, is inactive, or is not
   resolved, **When** they open a protected route, **Then** the application
   shows the existing authentication or session-required state instead of
   granting access.
5. **Given** a student or guardian self-service route is identity-owned,
   **When** a System Administrator has no existing actor-owned profile or
   guardian link, **Then** the route remains unavailable and does not expose
   another person's self-service data.

---

### User Story 2 - Keep School and Subject Boundaries (Priority: P2)

A System Administrator can work in any active school context while every
school-owned screen remains scoped to that selected school. Views that require
an existing selected subject context continue to require it.

**Why this priority**: Master access must improve permission usability without
showing stale, unscoped, or identity-owned information outside approved
boundaries.

**Independent Test**: Select an active school, load school-owned screens,
switch to another active school, and confirm prior school data clears before
the new context loads. Open a subject-context route with and without its
required subject context.

**Acceptance Scenarios**:

1. **Given** a school-scoped route has no active selected school, **When** a
   System Administrator attempts to open it, **Then** the application blocks
   school-owned loading and shows the existing school-context state.
2. **Given** a System Administrator selects an active school, **When** they
   open a school-owned route, **Then** the interface uses that school context
   and does not present data retained from another school.
3. **Given** a System Administrator changes the active school, **When** the
   new context is applied, **Then** school-owned data from the prior context is
   cleared before data for the new school is shown.
4. **Given** a route requires an existing selected subject context, **When**
   that context is absent or invalid, **Then** the application preserves the
   existing subject-context state and does not substitute master access for it.

---

### User Story 3 - Preserve Non-Permission States (Priority: P3)

A System Administrator receives clear existing application states when access
is blocked by a business or context prerequisite, rather than a misleading
permission-denied state.

**Why this priority**: Operators need to distinguish a missing school or
subject context, expired session, approval requirement, or safety hold from a
permission problem they cannot solve by changing roles.

**Independent Test**: Exercise representative protected views and actions with
missing school or subject context, expired session, approval, confirmation,
support, file-safety, and closed-period prerequisites, and verify each keeps
its established blocked state.

**Acceptance Scenarios**:

1. **Given** a System Administrator reaches a released protected action with
   an unmet approval or safety prerequisite, **When** the action is evaluated,
   **Then** the application preserves the prerequisite-specific blocked state.
2. **Given** a System Administrator receives a backend response for a missing
   context or business prerequisite, **When** the application renders the
   response, **Then** it does not label the condition as a missing
   feature-specific permission.
3. **Given** a System Administrator performs a state-changing action, **When**
   it succeeds, **Then** the interface relies on the existing backend audit
   behavior and does not create a separate client-side audit record.

### Edge Cases

- A saved direct link targets a released protected route before session
  restoration finishes; route evaluation waits for the existing session state
  instead of treating the role as absent.
- A selected school becomes inactive or unavailable; school-owned loading stays
  blocked until a valid active school is selected.
- A route or action is hidden because the related feature is unreleased; master
  access does not expose unfinished work.
- A platform-wide screen intentionally spans schools; it remains distinct from
  school-owned screens and must not adopt a selected-school filter merely
  because a school is selected.
- A non-System Administrator has unrelated broad permissions; the existing
  route and action permission checks continue to apply.

## Architecture & Contract Impact *(mandatory)*

### Repository Impact

- **Backend repository impact**: None. The completed backend master-access
  behavior, tenant enforcement, audit behavior, and authenticated-session role
  collection are consumed as published.
- **Frontend repository impact**: Protected route evaluation, role-aware
  navigation and action visibility, session role interpretation, school and
  subject context states, error-state mapping, stale school-data cleanup, and
  regression coverage are affected.
- **Specification or contract repository impact**: This feature records the
  frontend adoption contract and must remain aligned with the completed System
  Administrator master-access and tenant-isolation rules. No new service
  contract is introduced.
- **Delivery ownership and sequencing**: The backend implementation is the
  prerequisite and is complete. The frontend repository implements this feature
  against the existing session and authorization contract; no backend delivery
  is required for the frontend change.

### API Contract Impact

- **OpenAPI update required**: No. This feature consumes already documented
  authorization behavior and does not add or change operations.
- **Versioned endpoints affected**: No endpoint behavior changes are required;
  protected routes continue to use their existing published operations.
- **JSON response impact**: No response envelope, field, or error-contract
  change is required. The existing role collection remains the source for
  identifying System Administrator.
- **Authentication/authorization impact**: Client-side permission evaluation
  treats an active platform-scoped `System Administrator` role as satisfying
  released feature-specific route and action permissions. Authentication,
  session state, school context, subject context, release state, approval, and
  safety prerequisites remain enforceable.
- **Compatibility impact**: Additive access visibility and route behavior for
  System Administrator only. Existing non-System Administrator behavior is
  unchanged.

### Data & Tenancy Impact

- **Tenant scoping impact**: School-owned screens require an active selected
  school and display only data from the current school context.
- **Cross-tenant or platform access impact**: System Administrator may select
  any active school through the existing context mechanism. Only already
  documented platform-wide screens may present cross-school data.
- **Soft delete impact**: No lifecycle or soft-delete behavior changes are
  introduced; existing screen and backend rules continue to apply.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The application MUST identify System Administrator from the
  existing authenticated-session role collection as an active platform-scoped
  role named exactly `System Administrator`.
- **FR-002**: The application MUST allow a resolved System Administrator to
  pass every released non-identity-owned protected frontend route permission
  check without requiring feature-specific permissions.
- **FR-003**: The application MUST show every released non-identity-owned
  protected navigation destination and action to a resolved System
  Administrator when its non-permission prerequisites are met.
- **FR-004**: The application MUST preserve existing protected-route and
  navigation denial behavior for non-System Administrator users who lack a
  required permission.
- **FR-005**: The application MUST preserve authentication, active-account,
  session-resolution, and release-state checks for System Administrator.
- **FR-006**: The application MUST require a valid active selected school
  before loading any school-owned screen for System Administrator.
- **FR-007**: The application MUST clear school-owned data associated with the
  prior school before presenting data for a newly selected school.
- **FR-008**: The application MUST preserve required selected-subject context
  checks for identity-owned routes, MUST retain existing actor-owned student
  and guardian-link access rules, and MUST NOT infer or create subject context
  from System Administrator status.
- **FR-009**: The application MUST preserve approval, confirmation, support,
  file-safety, closed-period, and other non-permission blocked states for
  System Administrator.
- **FR-010**: The application MUST distinguish missing permission, missing
  school context, missing subject context, expired session, and business or
  safety prerequisite states wherever existing feedback presents those states.
- **FR-011**: The application MUST not create a new client-side audit record
  for master access; state-changing audit evidence remains the backend's
  responsibility.
- **FR-012**: The application MUST use platform-wide presentation only for
  screens already documented as platform-wide and MUST NOT expose cross-school
  data from school-owned screens.
- **FR-013**: Automated frontend regression coverage MUST verify System
  Administrator route access, navigation/action visibility, tenant and subject
  context gating, non-permission states, and limited-role denial behavior.

### Key Entities

- **System Administrator Session**: Authenticated session whose existing role
  collection contains the active platform master role.
- **Protected Frontend Route**: Released non-identity-owned application
  destination requiring a feature-specific permission for users who are not
  System Administrator.
- **Selected School Context**: Active school target used by school-owned views
  and requests.
- **Selected Subject Context**: Existing subject target required by applicable
  identity-owned routes; it is independent of the actor's role.
- **Non-Permission Prerequisite**: Authentication, session, tenant, subject,
  release, approval, confirmation, support, safety, or lifecycle condition
  that master access does not bypass.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Automated route checks show that 100% of released
  non-identity-owned protected route groups allow a resolved System
  Administrator without feature-specific permissions when non-permission
  prerequisites pass.
- **SC-002**: Automated navigation and action checks show that 100% of
  inventoried released non-identity-owned protected destinations and actions
  are available to a resolved System Administrator and retain existing denial
  behavior for limited users.
- **SC-003**: Automated school-context checks show that 100% of covered
  school-owned views block loading without an active school and clear prior
  school data before presenting a newly selected school's data.
- **SC-004**: Automated context and state checks show that 100% of covered
  selected-subject, session, approval, and safety prerequisites remain blocked
  for System Administrator when unmet.
- **SC-005**: The frontend change introduces no backend endpoint, response
  field, or response-envelope change.

## Assumptions

- The completed backend feature exposes the active platform role through the
  existing authenticated-session role collection.
- The backend remains authoritative for every authorization and tenant-scoping
  decision; frontend checks control experience, visibility, and loading only.
- Released protected routes, navigation entries, and actions can be inventoried
  from existing frontend route and navigation definitions.
- Existing school and subject context mechanisms remain the only approved ways
  to establish their respective context.
- Student self-service remains actor-owned and guardian self-service remains
  guardian-link-bound; this feature does not add System Administrator
  impersonation or subject-selection transport.
- Master access covers released product behavior only and does not expose
  unfinished, intentionally hidden, or disabled features.
